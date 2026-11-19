# Issue 70 评审重试对齐实现计划

> **面向智能体执行者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务执行本计划。步骤使用复选框（`- [ ]`）语法跟踪。

**目标：** 让 Tier 3 准确度和自定义目标准确度评审器能从一次格式错误（包括空响应）或 Schema 无效的响应中恢复，同时不削弱失败即关闭的试验与分支评分。

**架构：** 在共享评审模块和独立 Harbor 验证器模板中各添加一个内部"调用-解析-校验-重试"辅助函数。准确度和目标准确度提供严格的载荷校验器，默认输出 token 数恰好为 4096，且仅在解析/Schema 失败时重试。该辅助函数接受返回二元组和返回来源信息的调用函数，使模板目标评审器保留决定性尝试的 provider/model；行为检查的挽救逻辑保持隔离。由 SkillEvaluator 管理的 Harbor 验证器配置为六次串行直接 90 秒 provider 尝试加构建产物写入开销预留 600 秒，而作者编写的原生验证器超时保持不变。直接重试辅助函数之外的 provider SDK 工作（包括 OpenAI RAGAS 和 Bedrock SDK 重试）可能需要操作员提供 Harbor 超时乘数。

**技术栈：** Python 3.12/3.13、pytest、Harbor 0.13.2、Ruff、Docker、NVIDIA Build（用于实机验证）。

**范围边界：** 不持久化原始模型响应。它们可能将任意任务或凭据材料回显到可移植报告中，且 provider 传输层并不一致地暴露结束原因或用量。安全的响应遥测和重新评分支持需要单独设计；Issue #70 的主要重试和 token 预算缺陷在此处理。

---

### 任务 1：测试驱动重试、预算、来源和失败即关闭契约

**文件：**
- 修改：`tests/tier3/test_judge_result_semantics.py`
- 修改：`tests/tier3/test_judge_failure_artifacts.py`
- 验证：`tests/tier3/test_judge_parse_robustness.py`
- 验证：`tests/test_issue55_collector_truth.py`

- [ ] **步骤 1：为格式错误和 Schema 无效的恢复添加失败单元测试**

在 `tests/tier3/test_judge_result_semantics.py` 中，定义有效的重试载荷：

```python
VALID_ACCURACY_RESPONSE = json.dumps(
    {"criteria": _CRITERIA, "score": 0.6, "reason": "valid retry"}
)
VALID_GOAL_RESPONSE = json.dumps(
    {
        "user_goal": "complete the task",
        "end_state": "task completed",
        "achieved": True,
        "score": 1.0,
        "reason": "valid retry",
    }
)
```

对现有 `judge_module` fixture 的两个值，添加准确度测试，覆盖：

- 格式错误文本 → 有效响应；
- Schema 无效的完整 criteria（`ACTIONABLE: "true"`）→ 有效响应；
- 格式错误 → 格式错误，在恰好两次调用后返回结构化错误；
- 首次 provider 错误，在恰好一次调用后返回结构化错误；
- 格式错误 → 重试 provider 错误，在两次调用后返回有界、脱敏的结构化错误；
- 两次尝试的默认 `max_tokens` 均恰好为 `4096`；
- 第二次尝试包含 `previous reply could not be parsed or validated`。

在评审传输边界使用真实的脚本化函数，并断言返回的评审行为，而非 mock 本身：

```python
def test_accuracy_retries_schema_invalid_response_once_and_recovers(judge_module, monkeypatch):
    invalid = json.dumps({"criteria": {**_CRITERIA, "ACTIONABLE": "true"}, "score": 0.6})
    calls = []

    def fake_call(prompt, **kwargs):
        calls.append((prompt, kwargs))
        return (invalid, None) if len(calls) == 1 else (VALID_ACCURACY_RESPONSE, None)

    monkeypatch.setattr(judge_module, "call_public_llm", fake_call)
    result = judge_module.judge_accuracy("question", "ground truth", "agent response")

    assert result["score"] == 0.6
    assert [kwargs["max_tokens"] for _, kwargs in calls] == [4096, 4096]
    assert "previous reply could not be parsed or validated" in calls[1][0]
```

对于目标准确度，明确覆盖共享和模板两个表面。共享测试用二元组响应 patch `call_public_llm`。模板测试禁用 RAGAS 并用三元组 patch `_call_public_llm_with_provenance`。覆盖：

- 格式错误 → 有效；
- Schema 无效的 `{"achieved": "true"}` → 有效；
- 无效 → 无效，恰好两次；
- 初始 provider 错误，恰好一次；
- 无效 → 重试 provider 错误，两次并脱敏；
- 两次尝试均恰好 `4096`；
- 模板成功和最终错误均使用第二次尝试的 `provider` 和 `model`。

为准确度和目标添加共享的显式覆盖测试：

```python
@pytest.mark.parametrize("judge_name", ["judge_accuracy", "judge_goal_accuracy"])
def test_shared_structured_judge_keeps_explicit_max_tokens_override(monkeypatch, judge_name):
    calls = []
    response = VALID_ACCURACY_RESPONSE if judge_name == "judge_accuracy" else VALID_GOAL_RESPONSE

    def fake_call(prompt, **kwargs):
        calls.append(kwargs)
        return response, None

    monkeypatch.setattr(llm_judge, "call_public_llm", fake_call)
    judge = getattr(llm_judge, judge_name)
    judge("question", "ground truth", "agent response", max_tokens=2048)

    assert calls == [{"max_tokens": 2048}]
```

- [ ] **步骤 2：添加失败的真实验证器恢复和终态失败测试**

在 `tests/tier3/test_judge_failure_artifacts.py` 中，使用 `_load_verifier`，使实际的独立 `main()` 写入真实的 rich 和 numeric 奖励文件。

恢复测试的传输序列：

```python
pair_responses = [
    ("not-json", None),
    (
        json.dumps(
            {
                "criteria": {
                    "SKILL_IDENTIFIED": True,
                    "ACTION_CORRECT": True,
                    "FACTUALLY_ACCURATE": True,
                    "TASK_ADDRESSED": True,
                    "ACTIONABLE": True,
                },
                "score": 1.0,
                "reason": "accuracy recovered",
            }
        ),
        None,
    ),
    (json.dumps({"results": [{"step": 1, "passed": True}], "score": 1.0}), None),
]
goal_responses = [
    ("not-json", None, {"provider": "nv_build", "model": "first-model"}),
    (
        json.dumps({"achieved": True, "score": 1.0, "reason": "goal recovered"}),
        None,
        {"provider": "nv_build", "model": "retry-model"},
    ),
]
```

仅 patch `call_public_llm` 和 `_call_public_llm_with_provenance`，禁用 RAGAS，运行 `verifier.main()`，然后断言：

```python
assert "evaluation_status" not in rich
assert "evaluation_errors" not in rich
assert rich["details"]["accuracy"]["score"] == 1.0
assert rich["details"]["goal_accuracy"]["model"] == "retry-model"
assert numeric["accuracy"] == numeric["goal_accuracy"] == numeric["behavior_check"] == 1.0
assert [kwargs["max_tokens"] for _, kwargs in pair_calls] == [4096, 4096, 4096]
assert [kwargs["max_tokens"] for _, kwargs in goal_calls] == [4096, 4096]
```

添加一个单独的终态失败测试：准确度两次返回无效文本，目标和行为返回有效载荷，且无效文本包含配置的合成凭据。断言：

```python
with pytest.raises(SystemExit) as exc_info:
    verifier.main()
assert exc_info.value.code == 1
assert len(accuracy_attempts) == 2
assert rich["evaluation_status"] == "failed"
assert rich["accuracy"] is None
assert rich["details"]["accuracy"]["status"] == "error"
assert len(rich["evaluation_errors"]["accuracy"]) <= 512
assert synthetic_credential not in json.dumps(rich)
assert "accuracy" not in numeric
```

这证明两次无效尝试仍产生故意不完整的哨兵值，供现有收集器失败即关闭测试消费。

- [ ] **步骤 3：运行新测试并验证 RED**

```bash
uv run python -m pytest -q \
  tests/tier3/test_judge_result_semantics.py \
  tests/tier3/test_judge_failure_artifacts.py \
  -k 'retry or max_tokens or malformed_accuracy_and_goal'
```

预期：恢复/默认预算断言失败，因为准确度和目标当前只调用一次且为 1024 token。确认失败是行为断言，而非设置错误。

---

### 任务 2：为准确度和目标实现一次校验 JSON 重试

**文件：**
- 修改：`src/skillevaluator/tier3/eval_core/llm_judge.py:565-730`
- 修改：`src/skillevaluator/tier3/harbor/templates/eval.py:3280-3470`

- [ ] **步骤 1：在两个模块中添加共享常量和载荷校验器**

```python
STRUCTURED_JUDGE_MAX_TOKENS = 4096

_JUDGE_RETRY_REMINDER = (
    "\n\nIMPORTANT: Your previous reply could not be parsed or validated. Respond with ONLY the "
    "minified JSON object on a single line -- no markdown fences, no prose, and keep explanations brief."
)


def _accuracy_payload_error(parsed):
    if not isinstance(parsed, dict):
        return "Judge response was not a valid JSON object"
    criteria = parsed.get("criteria")
    criteria_valid = _valid_accuracy_criteria(criteria)
    if "criteria" in parsed and not criteria_valid:
        return "Judge response contained invalid accuracy criteria"
    if _finite_score(parsed.get("score")) is None and not criteria_valid:
        return "Judge response contained no valid accuracy score or complete criteria"
    return None


def _goal_payload_error(parsed):
    if not isinstance(parsed, dict):
        return "Judge response was not a valid JSON object"
    if not isinstance(parsed.get("achieved"), bool):
        return "Judge response contained an invalid achieved value"
    if "score" in parsed and _finite_score(parsed["score"]) is None:
        return "Judge response contained an invalid goal score"
    return None
```

在 `llm_judge.py` 中使用现有模块风格类型，在独立模板中使用无依赖的 Python。

- [ ] **步骤 2：添加内部"调用-解析-校验-重试"辅助函数**

在两个模块中实现此契约，共享模块传入 `_extract_json`，模板传入 `extract_json`：

```python
def _call_validated_json_judge(prompt, validate, call, extract, **call_kwargs):
    call_kwargs.setdefault("max_tokens", STRUCTURED_JUDGE_MAX_TOKENS)

    def invoke(call_prompt):
        content, error, *metadata = call(call_prompt, **call_kwargs)
        provenance = metadata[0] if metadata and isinstance(metadata[0], dict) else {}
        parsed = extract(content) if content else None
        validation_error = validate(parsed) if not error else None
        return parsed, error, provenance, validation_error

    parsed, error, provenance, validation_error = invoke(prompt)
    if error:
        return None, f"LLM judge error: {error}", provenance
    if validation_error is None:
        return parsed, None, provenance

    parsed, error, provenance, validation_error = invoke(prompt + _JUDGE_RETRY_REMINDER)
    if error:
        return None, f"LLM judge retry error: {error}", provenance
    if validation_error is not None:
        return None, f"{validation_error} after retry", provenance
    return parsed, None, provenance
```

不要在错误文本或元数据中包含原始响应内容。`_judge_error` 仍是有界/脱敏的终态构造器。

- [ ] **步骤 3：让评审器经由辅助函数路由，不改变评分**

- 共享和模板准确度用 `call_public_llm` 调用辅助函数，然后完全按原方式计算已校验的 criteria/score。
- 共享目标用 `call_public_llm` 调用辅助函数。
- 模板自定义目标用 `_call_public_llm_with_provenance` 调用辅助函数，并在成功和错误结果中都包含返回的来源信息。
- 将 provider 成功响应但模型内容为空的情况与传输/配置失败分开分类，使共享路径将其作为格式错误重试，与独立模板一致。所有真正的 provider 错误保持单次调用。
- 初始 provider 错误立即返回；仅解析/Schema 校验失败才重试。
- 仅允许恰好一次重试。
- 定义 `BEHAVIOR_JUDGE_MAX_TOKENS = STRUCTURED_JUDGE_MAX_TOKENS`；不要将行为解析或挽救重构进通用辅助函数。
- 仅将由 SkillEvaluator 管理的生成/脚手架/注入的 Harbor 验证器默认值从 180 秒提高到 600 秒。保留显式的原生任务超时覆盖；现有原生 180 秒值在需要额外 provider 余量时必须显式更新。

- [ ] **步骤 4：运行聚焦 GREEN 验证**

```bash
uv run python -m pytest -q \
  tests/tier3/test_judge_result_semantics.py \
  tests/tier3/test_judge_parse_robustness.py \
  tests/tier3/test_judge_failure_artifacts.py \
  tests/test_issue55_collector_truth.py::test_mixed_failed_condition_suppresses_published_quality_and_paired_artifacts
```

预期：全部通过。恢复写入完整奖励；终态重试耗尽仍不可评分，收集器仍抑制不完整分支的聚合。

- [ ] **步骤 5：运行触及文件的 lint 并提交**

```bash
uv run ruff check \
  src/skillevaluator/tier3/eval_core/llm_judge.py \
  src/skillevaluator/tier3/harbor/templates/eval.py \
  tests/tier3/test_judge_result_semantics.py \
  tests/tier3/test_judge_parse_robustness.py \
  tests/tier3/test_judge_failure_artifacts.py
git add \
  src/skillevaluator/tier3/eval_core/llm_judge.py \
  src/skillevaluator/tier3/harbor/templates/eval.py \
  tests/tier3/test_judge_result_semantics.py \
  tests/tier3/test_judge_parse_robustness.py \
  tests/tier3/test_judge_failure_artifacts.py \
  docs/superpowers/plans/2026-08-24-issue-70-judge-retry.md
git commit -m "fix(tier3): retry malformed structured judge responses"
```

---

### 任务 3：独立评审和端到端验证

**文件：**
- 评审：`origin/main..HEAD`
- 实机 fixture：`src/skillevaluator/tier3/reference_skills/calculator` 的临时单用例副本

- [ ] **步骤 1：完成两阶段和整体变更评审**

先派发规范符合性评审，再派发代码质量评审。修复并重新评审每个 Critical 或 Important 问题。任务评审通过后，派发新的整体变更评审者，聚焦重试次数、Schema 无效路径、max-token 覆盖、来源信息、脱敏、共享/模板漂移、失败即关闭构建产物、收集器/报告正确性和回归。

- [ ] **步骤 2：运行完整自动化验证**

```bash
uv run python -m pytest -q -n auto
uv run ruff check src tests
uv build
```

记录确切的通过/跳过计数和构建产物。任何失败都必须在继续前诊断。

- [ ] **步骤 3：使用 NVIDIA Build 运行受控实机恢复**

仅从 `/Users/christopherk/Work/.env` 加载 `NVIDIA_API_KEY`，不要 source 或打印该文件。设置：

```bash
export NVIDIA_API_KEY="$(sed -n 's/^NVIDIA_API_KEY=//p' /Users/christopherk/Work/.env | tail -n 1)"
export SKILL_EVAL_LLM_PROVIDER=nv_build
export SKILL_EVAL_LLM_MODEL=nvidia/nemotron-3-nano-30b-a3b
```

必须显式指定目录模型，因为独立验证器的回退模型是 OpenAI 默认值，而非 provider 特定的 NVIDIA Build 默认值。

对于共享准确度和目标，包装真实的 `call_public_llm`，使第一次尝试返回评审器特定的 Schema 无效载荷，第二次尝试委托给原始真实 NVIDIA Build 调用。

为每个模板评审器加载全新的独立模板模块。对于模板准确度，包装其真实的 `call_public_llm`，使第一次尝试返回 Schema 无效的准确度 JSON，第二次尝试委托给原始真实 NVIDIA Build 调用。对于模板自定义目标，以相同方式包装 `_call_public_llm_with_provenance`：第一次尝试返回无效的 `achieved` 及 `{"provider": "controlled", "model": "invalid-first"}`，第二次尝试委托给原始真实 provider 调用。断言：

-
# 第三方声明

本文件列出本发行版声明的直接第三方 Python 依赖。`pyproject.toml` 定义依赖组，`uv.lock` 记录本次发布实际解析使用的依赖集合。

| 依赖组 | 包与许可证 |
| --- | --- |
| Base | Bandit (Apache-2.0)、Click (BSD-3-Clause)、IDNA (BSD-3-Clause)、Jinja2 (BSD-3-Clause)、Markdown-It-Py (MIT)、Packaging (Apache-2.0 或 BSD-2-Clause)、Pydantic (MIT)、PyYAML (MIT)、regex (Apache-2.0 AND CNRI-Python)、Rich (MIT)、uv (Apache-2.0 或 MIT) |
| LLM | Anthropic (MIT)、Boto3 (Apache-2.0)、LiteLLM (MIT)、OpenAI (Apache-2.0) |
| Tier 3 | Harbor (Apache-2.0)、MCP (MIT)、OAuthLib (BSD-3-Clause)、PyJWT (MIT)、python-dotenv (BSD-3-Clause) |
| Security | pip-audit (Apache-2.0) |
| Dashboard | Streamlit (Apache-2.0) |

## 内嵌报告图表

Tier 3 HTML 报告包含 Chart.js 4.5.1 及其捆绑的 @kurkle/color 0.3.2（MIT）。wheel 和源码分发包在 `skillevaluator/reporting/assets/` 中保留完整许可证；生成的含图表 HTML 报告也包含两份完整许可证文本。资产 README 记录了公开上游来源、原始 SRI 校验、摘要、移除 source-map 引用以及规范化为单个末尾换行。捆绑包未做其他改动。

### Chart.js 4.5.1

The MIT License (MIT)

Copyright (c) 2014-2024 Chart.js Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

### @kurkle/color 0.3.2

The MIT License (MIT)

Copyright (c) 2018-2021 Jukka Kurkela

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Tier 1 CI 容器

可选的 `ci/Dockerfile.tier1` 镜像还安装以下独立扫描器；它们不包含在 SkillEvaluator wheel 中。其上游来源和许可证见下方固定版本。Python 包许可证元数据保留在镜像的扫描器环境中；Gitleaks 二进制的许可证保留在 `/usr/local/share/licenses/gitleaks/LICENSE`。

| 工具 | 固定来源 | 许可证 |
| --- | --- | --- |
| Semgrep | [v1.182.0](https://github.com/semgrep/semgrep/tree/v1.182.0) | LGPL-2.1 |
| SkillSpector | [5482446](https://github.com/NVIDIA/SkillSpector/tree/54824463814a0be01360792523ec7245a69190fc) | Apache-2.0 |
| Gitleaks | [v8.30.0](https://github.com/gitleaks/gitleaks/tree/v8.30.0) | MIT |

## Unicode 安全数据

`src/skillevaluator/publication_text.py` 包含 Unicode 15.1 通用类别范围，以及 Unicode 技术标准 #39 所用 Unicode 17.0.0 `confusables.txt` 数据的生成子集。

UNICODE LICENSE V3

COPYRIGHT AND PERMISSION NOTICE

Copyright © 1991-2026 Unicode, Inc.

NOTICE TO USER: Carefully read the following legal agreement. BY DOWNLOADING,
INSTALLING, COPYING OR OTHERWISE USING DATA FILES, AND/OR SOFTWARE, YOU
UNEQUIVOCALLY ACCEPT, AND AGREE TO BE BOUND BY, ALL OF THE TERMS AND CONDITIONS
OF THIS AGREEMENT. IF YOU DO NOT AGREE, DO NOT DOWNLOAD, INSTALL, COPY,
DISTRIBUTE OR USE THE DATA FILES OR SOFTWARE.

Permission is hereby granted, free of charge, to any person obtaining a copy of
data files and any associated documentation (the "Data Files") or software and
any associated documentation (the "Software") to deal in the Data Files or
Software without restriction, including without limitation the rights to use,
copy, modify, merge, publish, distribute, and/or sell copies of the Data Files
or Software, and to permit persons to whom the Data Files or Software are
furnished to do so, provided that either (a) this copyright and permission
notice appear with all copies of the Data Files or Software, or (b) this
copyright and permission notice appear in associated Documentation.

THE DATA FILES AND SOFTWARE ARE PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT OF THIRD
PARTY RIGHTS. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR HOLDERS INCLUDED IN THIS
NOTICE BE LIABLE FOR ANY CLAIM, OR ANY SPECIAL INDIRECT OR CONSEQUENTIAL
DAMAGES, OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS,
WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING
OUT OF OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THE DATA FILES OR
SOFTWARE.

Except as contained in this notice, the name of a copyright holder shall not be
used in advertising or otherwise to promote the sale, use or other dealings in
these Data Files or Software without prior written authorization of the
copyright holder.
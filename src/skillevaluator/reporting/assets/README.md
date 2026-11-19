# 离线报告图表资源

这些资源是公开的上游软件，随 Python wheel 和源码分发包一起提供，使 Tier 3 HTML 报告无需网络访问即可工作。它们从固定的包资源名读取、缓存，仅在报告包含图表面板时嵌入。运行时无需 JavaScript 包管理器。

## Chart.js 4.5.1

- 上游：<https://github.com/chartjs/Chart.js/tree/v4.5.1>
- 原始 bundle：<https://cdn.jsdelivr.net/npm/chart.js@4.5.1/dist/chart.umd.min.js>
- 许可证来源：<https://registry.npmjs.org/chart.js/-/chart.js-4.5.1.tgz> 中的 `package/LICENSE.md`，原样保留为 `Chart.js-LICENSE.md`（MIT）。
- 原始 SHA-384（编辑前已对照报告此前的 SRI 验证）：
  `jb8JQMbMoBUzgWatfe6COACi2ljcDdZQ2OxczGA3bGNeWe+6DChMTBJemed7ZnvJ`
- 原始 SHA-256：
  `48444a82d4edcb5bec0f1965faacdde18d9c17db3063d042abada2f705c9f54a`
  （208,522 字节）。
- 内置 `chart.umd.min.js` 的 SHA-256：
  `84d0e233daba702b8f77d669d8c137cad36d441a10f200b6f2d3ab553bdfcf6b`
  （208,480 字节）。

修改内容为：删除 41 个 ASCII 字节
`//# sourceMappingURL=chart.umd.min.js.map`，并将剩余的两个尾部 LF 字节规范化为一个最终 LF 字节。删除该指令可防止开发者工具请求缺失的 source map。JavaScript 主体、MIT 头部和捆绑依赖声明保持不变。要复现该资源，请下载原始 URL 并验证上述原始 SHA-384，然后应用此精确的字节变换并验证内置 SHA-256：

```python
vendored = original.replace(b"//# sourceMappingURL=chart.umd.min.js.map", b"").rstrip(b"\n") + b"\n"
```

不要重新压缩它。

## 捆绑的 @kurkle/color 0.3.2

bundle 自身的头部标明版本为 0.3.2，无论较新的依赖解析可能从 Chart.js 的版本范围中选择哪个版本。

- 上游：<https://github.com/kurkle/color/tree/v0.3.2>
- 许可证来源：<https://registry.npmjs.org/@kurkle/color/-/color-0.3.2.tgz> 中的 `package/LICENSE.md`，原样保留为 `kurkle-color-LICENSE.md`（MIT）。

两份完整许可证文本也会出现在生成的含图表报告中以及分发包的 `THIRD_PARTY_NOTICES.md` 中。
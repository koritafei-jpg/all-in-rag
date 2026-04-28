## AI Memory 调研报告 LaTeX 版

### 文件说明
- `ai_memory_report.tex`：主文稿（中文论文体）
- `references.bib`：参考文献 BibTeX

### 编译方式（推荐 XeLaTeX）
在 `docs/` 目录执行：

```bash
xelatex ai_memory_report.tex
bibtex ai_memory_report
xelatex ai_memory_report.tex
xelatex ai_memory_report.tex
```

生成文件：`ai_memory_report.pdf`

### 说明
- 文稿使用 `ctex`，适合中文投稿/汇报；
- 若目标会议有模板，可将正文内容迁移到其 `\documentclass` 下复用。

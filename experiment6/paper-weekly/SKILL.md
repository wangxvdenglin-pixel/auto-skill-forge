---
name: paper-weekly
description: "Organize scattered paper reading notes and PDF summaries into a unified Markdown table. Use when users paste multiple summaries, mention \"整理论文\", \"论文周报\", \"paper weekly\", \"这周看的论文\", or want to create a structured reading digest with columns for title, method, results, and personal insights."
---

# Paper Weekly

Transform scattered paper reading notes into a clean Markdown table for weekly research digests.

## Workflow

### Step 1: Collect Input

Accept the user's reading notes. These may be:
- Multiple pasted text blocks (unstructured)
- PDF summaries (plain text)
- Lists of paper titles with brief notes
- Mixed formats

If the user hasn't provided enough information for any paper (missing methods, results, or key details), note what's missing and flag it — don't fabricate content.

### Step 2: Parse Each Paper

For each paper mentioned in the input, extract these four dimensions:

| Dimension | What to extract |
|-----------|----------------|
| 论文标题 (Title) | Full paper title as stated by the user |
| 核心方法 (Method) | The main approach, algorithm, or technique |
| 实验结果 (Results) | Key findings, metrics, benchmarks |
| 个人启发 (Insights) | User's personal takeaways or how it relates to their work |

### Step 3: Build the Table

Format as a Markdown table. Use this exact structure:

```markdown
## 论文周报 — YYYY-MM-DD

| 论文标题 | 核心方法 | 实验结果 | 个人启发 |
|----------|----------|----------|----------|
| Title 1 | Method description | Key results | Personal insight |
| Title 2 | Method description | Key results | Personal insight |
```

### Step 4: Handle Edge Cases

- **Missing information**: If a dimension has no data for a paper, fill with "—" (em dash). Do not invent content.
- **Special characters in content**: Escape pipe characters `|` within cells by replacing them with `、` or `/`.
- **Very long content**: Truncate to 120 characters per cell. Append "…" to indicate truncation.
- **Duplicate titles**: Merge entries with the same title, combining non-redundant information.

### Step 5: Confirm with User

Show the draft table to the user before finalizing:

> "以上是根据你的笔记整理的论文周报草稿，需要修改或补充吗？"

If user requests changes, apply them and re-show. Otherwise proceed.

### Step 6: Present Output

Display the final confirmed table. Follow with a brief summary:

- Total papers: N
- Papers with complete information: N
- Papers with missing fields: N (list which fields are missing for which papers)

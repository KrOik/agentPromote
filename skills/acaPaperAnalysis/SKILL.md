---
name: academic-paper-analysis
description: Systematically analyze academic paper PDFs and produce a structured report covering literature review, innovations, article status (OA/preprint), journal evaluation (impact factor, quartile, domestic recognition), experimental design, data samples, and figure distribution. Use when the user asks to read, analyze, review, or interpret a scientific paper, or requests journal/conference evaluation.
version: 1.0.0
---

# Academic Paper Analysis

Perform a comprehensive, structured analysis of an academic paper (PDF). Output a standardized report in Chinese covering eight dimensions.

## Workflow

### Step 1: Locate and extract the PDF

1. Use `Glob` or `Bash` (`dir`) to locate the target PDF file.
2. Run the extraction script to get full text:
   ```python
   import sys, io
   sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8', errors='replace')
   sys.stderr = io.TextIOWrapper(sys.stderr.buffer, encoding='utf-8', errors='replace')
   import pdfplumber
   path = r'<path_to_pdf>'
   with pdfplumber.open(path) as doc:
       print(f'Total pages: {len(doc.pages)}')
       for i, pg in enumerate(doc.pages):
           text = pg.extract_text()
           if text:
               print(f'\n===== PAGE {i+1} =====')
               print(text)
   ```
   This wraps stdout in UTF-8 to avoid Windows GBK encoding errors.
3. If the output is too large, read the persisted output file in segments using the `Read` tool.

### Step 2: Query journal/conference information

In parallel, run two web searches:

- `"<journal_name> impact factor <current_year> CAS zone ranking 中科院分区"`
- `"<journal_name>" SCI impact factor quartile <current_year>`

Then use `WebFetch` on the top result (prefer justscience.cn or letpub.com.cn) to extract:

- Impact Factor (latest + 5-year)
- CiteScore
- JCR quartile (Q1-Q4)
- 中科院分区 (大类 + 小类, Top标识)
- Whether it is a Nature Partner Journal, Cell Press partner, etc.
- Open Access status and APC
- ISSN

### Step 3: Structured analysis (eight dimensions)

Read the extracted text carefully and analyze along the eight dimensions below:

1. **论文综述 (Literature Review)**: Summarize the Introduction's literature threads. Identify the research background, existing approaches, their limitations, and the research gap the paper addresses.

2. **创新点 (Innovations)**: Evaluate from your own expert perspective, not just restating the abstract. Highlight what is genuinely novel vs incremental. Focus on: novel materials, new methodologies, unique system integration, or new application domains.

3. **文章状态 (Article Status)**: Published/accepted/preprint. OA type (Gold/Green/Bronze/None). License type. Submission and acceptance dates if available.

4. **期刊/会议评价 (Journal/Conference Evaluation)**: Combine Step 2 data. State: journal name, publisher, IF (latest + 5-year), CiteScore, JCR quartile, 中科院分区 (大类 + 小类 + Top标识), whether it is an NPJ or other prestigious imprint. Assess domestic recognition in China.

5. **实验设计 (Experimental Design)**: Classify experiments into:
   - 干实验 (Dry lab): simulation, ML modeling, computational analysis
   - 湿实验 (Wet lab): materials synthesis, chemical characterization, biological assays, human trials
   List each experiment with its purpose.

6. **数据样本 (Data Samples)**: Tabulate sample sizes for every experiment. Note: number of subjects, replicates, measurement points, training/test splits for ML models.

7. **业界贡献 (Industry Contribution)**: Identify the broader impact, generalizability, and practical applicability of the work.

8. **图例分布 (Figure Distribution)**: For each main figure and supplementary figure, describe every sub-panel and what information it conveys.

### Step 4: Output the report

- Output the full report directly in the chat message in Chinese.
- Use `##` headers for each of the eight sections.
- Include a summary evaluation paragraph at the end.
- Append `Sources:` with hyperlinks to journal info pages used in Step 2.

## Pitfalls

- **Windows encoding**: `pdfplumber.extract_text()` output piped through `print()` may fail with `UnicodeEncodeError: 'gbk'` on Windows. Always wrap stdout with UTF-8 as shown in Step 1.
- **Large PDFs**: Papers with many pages may produce output exceeding terminal buffer. Read the persisted output in chunks with offset/limit.
- **Image-based PDFs**: If `extract_text()` returns empty or garbled text, fall back to OCR via `pytesseract` + `pdf2image`.
- **Journal info staleness**: Impact factors and quartiles update annually. Always search for the current year's data. Note both the traditional and "新锐" partition tables if they differ.
- **Supplementary figures**: Often referenced as "Fig. S1" etc. They may not be in the main PDF text but are described contextually. Note which supplementary figures are referenced and what they likely contain.

## Verification

After producing the report, verify:

1. All eight dimensions are covered.
2. Every main figure (Fig. 1 through Fig. N) has been described with all sub-panels.
3. Journal IF, quartile, and 中科院分区 are cited with source URLs.
4. Sample sizes are extracted for every experiment mentioned in the paper.

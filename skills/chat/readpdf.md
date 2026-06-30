---
name: readpdf
description: Use this skill whenever the user wants to read, extract, analyze, or work with PDF files. Covers CIMs, LP agreements, credit memos, fund reports, regulatory filings, and any other financial or legal document in PDF form. When active, Claude asks the user to share or paste PDF content, then offers to summarize, answer questions, extract key data points, flag risks, or compare against SMC deal standards. Also handles PDF manipulation (merge, split, extract tables) for Claude Code users.
---

# Read PDF — Star Mountain Capital

When a user shares a PDF or asks to work with one, your job is to extract maximum value from it quickly and accurately — surfecting the information that matters most for SMC's investment, FinOps, and operations workflows.

## When to Use This Skill

- Summarizing a CIM before a deal call
- Extracting key terms from an LP agreement or credit agreement
- Reviewing a fund administrator's quarterly report or NAV statement
- Pulling financial tables from a management presentation
- Reviewing a regulatory filing (Form ADV, Form PF, 10-K, 10-Q)
- Extracting LP capital account data from a PDF statement
- Comparing a document against SMC deal or compliance standards
- Manipulating PDFs (merge, split, extract pages) in Claude Code

## How to Work with a PDF in claude.ai

Claude cannot directly open PDF files, but it can read PDF content you share in three ways:

1. **Attach the file directly** — Click the paperclip icon in the chat input and upload the PDF. Claude will read it.
2. **Paste extracted text** — Open the PDF in Adobe, Preview, or your browser, select all (Ctrl+A / Cmd+A), copy, and paste into the chat.
3. **Share specific pages** — If the document is long, paste the most relevant pages or sections and specify which pages they are.

When you share a PDF, Claude will:
1. Confirm what it received and its apparent document type
2. Offer a menu of analysis options (or proceed directly if you've already specified)
3. Deliver structured, SMC-relevant output

## Analysis Menu

When a PDF is shared without a specific instruction, offer these options:

```
I've received your PDF. What would you like me to do with it?

A) Full document summary — key findings, structure, critical provisions
B) Answer specific questions — tell me what you want to know
C) Extract financial data — pull tables, metrics, or numbers into structured format
D) Risk and red flag review — flag unusual terms, missing provisions, or concerns
E) Compare to SMC standards — check against SMC deal, compliance, or reporting norms
F) Multiple of the above — specify which
```

## Document-Type Playbooks

### CIM (Confidential Information Memorandum)

Extract and present:
- Company overview (what it does, sector, geography)
- Revenue model and key revenue streams
- LTM and 3-year financial summary (revenue, EBITDA, margins)
- Customer profile (count, concentration, contract type)
- Management team highlights
- Seller's investment highlights
- Implied valuation / asking multiple (if stated)
- Key risks you identify (including what the sell-side isn't emphasizing)
- SMC mandate fit assessment

### LP Agreement / LPA

Extract and present:
- Management fee (rate, basis — committed vs. invested, step-down schedule)
- Carried interest (rate, hurdle, catch-up, clawback)
- Investment period and fund term
- Key person provisions (who, trigger events, consequences)
- LP removal rights and vote threshold
- LPAC composition and authority
- Distribution waterfall (European vs. American; preferred return structure)
- Expense allocation (fund vs. management company)
- Major conflict of interest provisions

### Credit Agreement / Loan Document

Extract and present:
- Facility type and size
- Interest rate (spread, base rate, PIK component, total)
- Maturity date and amortization
- Financial maintenance covenants (Total Leverage, Senior Leverage, FCCR, Minimum Liquidity)
- Negative covenants (restricted payments, additional debt, asset sales, investments)
- Events of default
- Call protection and prepayment premiums
- Reporting requirements

### Fund / NAV Report (Administrator Statement)

Extract and present:
- Fund name and reporting period
- NAV (beginning, ending, change)
- Capital activity (calls, distributions)
- Portfolio company valuations (where disclosed)
- Performance metrics (IRR, MOIC, DPI/RVPI/TVPI — where stated)
- Fee and expense summary

### Regulatory Filing (Form ADV, 10-K, 10-Q, Form PF)

Extract and present:
- Key disclosures relevant to SMC's context
- AUM (reported vs. actual regulatory AUM)
- Disciplinary history (Item 11 for Form ADV)
- Material changes since prior filing
- Risk factors (10-K/10-Q)
- Portfolio metrics (BDC filings)

### General / Unknown Document Type

Provide:
1. Document type identification
2. Key parties and their roles
3. Core purpose (one sentence)
4. Most important provisions or findings (ranked)
5. Action items or open questions for SMC

## PDF Manipulation (Claude Code Users)

For programmatic PDF work in Claude Code, use `pdfplumber` for extraction and `pypdf` for manipulation:

```python
# Extract all text and tables from a financial PDF
import pdfplumber
import pandas as pd

def extract_pdf_content(filepath: str) -> dict:
    """Extract text and tables from a PDF. Returns dict with pages and tables."""
    result = {'pages': [], 'tables': []}
    
    with pdfplumber.open(filepath) as pdf:
        for i, page in enumerate(pdf.pages, 1):
            text = page.extract_text() or ''
            tables = page.extract_tables()
            
            result['pages'].append({'page': i, 'text': text})
            
            for j, table in enumerate(tables):
                if table and len(table) >= 2:
                    headers = [str(h).strip() if h else f'col_{k}' 
                               for k, h in enumerate(table[0])]
                    rows = [dict(zip(headers, row)) for row in table[1:]]
                    result['tables'].append({
                        'page': i, 'table_index': j,
                        'headers': headers, 'rows': rows
                    })
    
    return result
```

```python
# Merge multiple PDF reports into one
from pypdf import PdfWriter, PdfReader

def merge_pdfs(input_paths: list[str], output_path: str) -> None:
    writer = PdfWriter()
    for path in input_paths:
        reader = PdfReader(path)
        for page in reader.pages:
            writer.add_page(page)
    with open(output_path, 'wb') as f:
        writer.write(f)
```

```python
# Split a multi-LP statement PDF by page ranges
from pypdf import PdfWriter, PdfReader

def split_pdf_by_ranges(input_path: str, ranges: list[tuple], output_dir: str) -> list[str]:
    """ranges: list of (start_page, end_page, filename) — 0-indexed"""
    reader = PdfReader(input_path)
    output_paths = []
    for start, end, name in ranges:
        writer = PdfWriter()
        for page in reader.pages[start:end+1]:
            writer.add_page(page)
        out = f"{output_dir}/{name}.pdf"
        with open(out, 'wb') as f:
            writer.write(f)
        output_paths.append(out)
    return output_paths
```

## Output Format Standards

- Always label the document type at the top of your analysis
- Use tables for financial data and comparison information
- Use bulleted lists for provisions, findings, and red flags
- Flag items requiring SMC legal or compliance review explicitly
- Note if data appears unaudited, preliminary, or management-estimated
- End with: "Open questions / items requiring follow-up:" if any gaps remain

## Privacy Note

PDF documents shared in this context may contain confidential information, MNPI, or PII. Do not reproduce full documents verbatim in outputs. Summarize and extract. Flag any content that appears to be MNPI (material non-public information about a public company) for compliance review before further distribution.

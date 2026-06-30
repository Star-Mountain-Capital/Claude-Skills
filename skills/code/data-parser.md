---
name: data-parser
description: Parses and transforms financial data files — Excel, PDF, Word, and CSV — into clean, structured formats for use in Star Mountain Capital's FinOps and investment workflows. Use when processing fund administrator reports, LP statements, portfolio company financials, CIM documents, or any unstructured data source that needs to be normalized into a machine-readable format. Adapted from Anthropic's xlsx and pdf skill libraries.
---

# Data Parser

You are a financial data engineering assistant at Star Mountain Capital. You help the FinOps and investment teams extract, normalize, and transform data from the messy formats it arrives in — PDF statements, multi-tab Excel workbooks, Word documents, and CSV exports — into clean, structured tables ready for analysis, reconciliation, or reporting.

## When to Use This Skill

- Parsing fund administrator statements (PDF or Excel) into structured tables
- Extracting financial tables from CIM documents or management presentations
- Normalizing multi-format Excel reports from portfolio companies into consistent schemas
- Converting LP capital account statements from PDF to Excel
- Extracting transaction data from trade confirmations or settlement statements
- Processing portfolio company financial packages (P&L, balance sheet, cash flow) into a standard model
- Converting multi-tab fund admin workbooks into a single normalized dataset

## SMC Document Types

| Document | Source | Common Format | Key Data to Extract |
|----------|--------|---------------|---------------------|
| Capital call notice | SMC internal | PDF / Word | LP name, call amount, due date, purpose |
| Distribution notice | SMC internal | PDF / Word | LP name, distribution amount, tax breakdown |
| Capital account statement | Fund administrator | PDF / Excel | Beginning balance, contributions, distributions, ending balance |
| Trial balance | Fund administrator | Excel | Account code, account name, debit, credit, net balance |
| Portfolio company financials | Management | Excel | Revenue, EBITDA, capex, working capital by period |
| LP statement | Fund administrator | PDF | Same as capital account statement |
| Trade confirmation | Broker / custodian | PDF | Asset, CUSIP, settlement date, price, quantity |
| Loan statement | Lender / administrator | PDF / Excel | Principal, interest, fees, maturity, outstanding balance |

## Core Parsing Patterns

### Excel Multi-Tab Normalization

When a file has multiple tabs (common for fund administrator workbooks or portfolio company packages):

```python
import pandas as pd
from pathlib import Path

def normalize_multisheet_workbook(filepath: str, schema: dict) -> pd.DataFrame:
    """
    Reads all sheets matching schema keys, normalizes column names,
    and returns a single combined DataFrame with a 'source_sheet' column.
    """
    xl = pd.ExcelFile(filepath)
    frames = []
    for sheet in xl.sheet_names:
        if any(key.lower() in sheet.lower() for key in schema.keys()):
            df = xl.parse(sheet, header=None)
            df = detect_and_set_header(df)
            df = rename_columns(df, schema)
            df['source_sheet'] = sheet
            frames.append(df)
    return pd.concat(frames, ignore_index=True) if frames else pd.DataFrame()
```

### PDF Table Extraction

For PDF reports from administrators or lenders:

```python
import pdfplumber

def extract_tables_from_pdf(filepath: str) -> list[dict]:
    """
    Extracts all tables from a PDF, returns list of dicts with
    page_number, table_index, headers, and rows.
    """
    results = []
    with pdfplumber.open(filepath) as pdf:
        for page_num, page in enumerate(pdf.pages, 1):
            tables = page.extract_tables()
            for table_idx, table in enumerate(tables):
                if not table or len(table) < 2:
                    continue
                headers = [str(h).strip() if h else f'col_{i}' 
                          for i, h in enumerate(table[0])]
                rows = [dict(zip(headers, [str(v).strip() if v else '' for v in row])) 
                       for row in table[1:] if any(v for v in row)]
                results.append({
                    'page': page_num,
                    'table_index': table_idx,
                    'headers': headers,
                    'rows': rows
                })
    return results
```

### Capital Account Statement Parser

```python
def parse_capital_account_statement(filepath: str) -> pd.DataFrame:
    """
    Parses LP capital account statements into standardized roll-forward format.
    Handles both PDF and Excel inputs.
    """
    EXPECTED_LINES = [
        'beginning_balance',
        'contributions',
        'management_fees',
        'fund_expenses', 
        'unrealized_gain_loss',
        'realized_gain_loss',
        'distributions',
        'ending_balance'
    ]
    # ... parsing logic
```

### Portfolio Company Financial Normalizer

Standardizes portfolio company monthly/quarterly financials:

```python
STANDARD_SCHEMA = {
    'income_statement': {
        'revenue': ['revenue', 'net revenue', 'total revenue', 'net sales'],
        'gross_profit': ['gross profit', 'gross margin'],
        'ebitda': ['ebitda', 'adjusted ebitda'],
        'ebit': ['ebit', 'operating income'],
        'net_income': ['net income', 'net earnings']
    },
    'balance_sheet': {
        'cash': ['cash', 'cash and equivalents'],
        'total_assets': ['total assets'],
        'total_debt': ['total debt', 'long-term debt', 'notes payable'],
        'total_equity': ['total equity', 'shareholders equity']
    }
}
```

## Data Quality Checks

Always run these checks on parsed output before use:

1. **Balance check:** For capital account roll-forwards: beginning + net activity = ending (within $1 rounding)
2. **Completeness check:** All expected LP names present in fund-level files
3. **Period check:** All periods complete; no gap months
4. **Negative checks:** Flag unexpected negative values (negative revenue, negative cash)
5. **Outlier check:** Flag values that differ from prior period by more than 25% (except at close events)

```python
def run_data_quality_checks(df: pd.DataFrame, check_config: dict) -> list[str]:
    warnings = []
    # Balance check
    if 'balance_check' in check_config:
        # ...
    # Completeness check
    if 'expected_entities' in check_config:
        missing = set(check_config['expected_entities']) - set(df['entity'].unique())
        if missing:
            warnings.append(f"Missing entities: {missing}")
    return warnings
```

## Output Conventions

- Always write parsed output to `./out/` — never overwrite source files
- Include a parsing log with: source filename, record count, warnings, timestamp
- Flag unparsed or ambiguous rows in a separate `_exceptions.csv` file

## Dependencies

```
pandas>=2.0
openpyxl>=3.1      # Excel read/write
pdfplumber>=0.11   # PDF table extraction
python-docx>=1.1   # Word document parsing
```

## Example Requests

```
Parse this 12-tab Excel workbook from our fund administrator. Each tab is a different LP's capital account. Extract them all into a single normalized DataFrame with columns: lp_name, period, line_item, amount.
```

```
This PDF is a quarterly LP statement from our administrator. Extract all the line items from the capital account roll-forward section and convert them to a CSV.
```

```
I have 8 portfolio company Excel financials, each formatted differently. Normalize them all to a standard P&L format with: company, period, revenue, gross_profit, ebitda, net_income.
```

```
Extract all tables from this CIM PDF. I want every financial table — income statement, balance sheet, and any EBITDA bridge tables.
```

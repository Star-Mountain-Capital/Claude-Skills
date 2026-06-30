---
name: investment-research
description: Automates investment research workflows for Star Mountain Capital — sector analysis, competitive landscape research, peer company benchmarking, and research note generation. Use when building research pipelines that process industry data, generate structured deal screens, or produce analyst-grade research outputs. Adapted from Anthropic's market-researcher and earnings-reviewer managed agent cookbooks.
---

# Investment Research

You are an investment research automation engineer at Star Mountain Capital. You help the investment team build automated research workflows that process sector data, analyze competitive dynamics, benchmark portfolio companies against peers, and generate research notes — turning raw data into structured, analyst-grade outputs.

## SMC Research Context

**Research purposes at SMC:**
- Deal origination: identify sectors with favorable LMM deal flow
- Diligence support: competitive analysis for target companies
- Portfolio monitoring: benchmark portfolio companies against sector peers
- IC support: prepare sector primers and comps tables for investment presentations

**Data sources typically used:**
- CapIQ (public company data, M&A transactions, private company coverage)
- PitchBook (LMM deal data, fund performance, private company intelligence)
- Dun & Bradstreet / Hoovers (company background)
- Management-provided information
- Public filings, press releases, industry reports

## Core Research Workflows

### 1. Sector Screening Pipeline

```python
from dataclasses import dataclass
from typing import Optional
import json

@dataclass
class SectorScreen:
    sector: str
    naics_codes: list[str]
    ebitda_range: tuple[float, float]  # in $M
    geography: str = 'US'
    min_revenue: Optional[float] = None
    
def run_sector_screen(screen: SectorScreen, data_source: dict) -> dict:
    """
    Filters a universe of companies to those matching SMC's LMM criteria.
    Returns structured results with key financial metrics.
    """
    results = {
        'sector': screen.sector,
        'screen_date': data_source.get('as_of_date'),
        'total_companies': 0,
        'qualifying_companies': [],
        'median_metrics': {},
    }
    
    qualifying = []
    for company in data_source.get('companies', []):
        ebitda = company.get('ltm_ebitda_m', 0)
        if screen.ebitda_range[0] <= ebitda <= screen.ebitda_range[1]:
            qualifying.append({
                'name': company['name'],
                'ltm_revenue': company.get('ltm_revenue_m'),
                'ltm_ebitda': ebitda,
                'ebitda_margin': ebitda / max(company.get('ltm_revenue_m', 1), 1),
                'ownership': company.get('ownership_type'),
                'hq_state': company.get('hq_state'),
            })
    
    results['qualifying_companies'] = qualifying
    results['total_companies'] = len(qualifying)
    results['median_metrics'] = compute_median_metrics(qualifying)
    
    return results
```

### 2. Comparable Company Benchmarking

```python
def build_comps_table(target: dict, comparables: list[dict]) -> dict:
    """
    Creates a structured comps table comparing target company to peers.
    Includes revenue, EBITDA, margins, multiples, and growth rates.
    """
    METRICS = [
        ('ltm_revenue_m',    'LTM Revenue ($M)'),
        ('ltm_ebitda_m',     'LTM EBITDA ($M)'),
        ('ebitda_margin',    'EBITDA Margin'),
        ('revenue_growth',   'Revenue Growth (YoY)'),
        ('ev_ebitda',        'EV/EBITDA'),
        ('ev_revenue',       'EV/Revenue'),
        ('net_leverage',     'Net Leverage (x)'),
    ]
    
    rows = [format_comp_row(target, is_target=True)]
    rows.extend(format_comp_row(c) for c in comparables)
    
    # Summary statistics
    ebitda_multiples = [c.get('ev_ebitda') for c in comparables if c.get('ev_ebitda')]
    stats = {
        'count': len(comparables),
        'ev_ebitda_mean':   safe_mean(ebitda_multiples),
        'ev_ebitda_median': safe_median(ebitda_multiples),
        'ev_ebitda_low':    min(ebitda_multiples) if ebitda_multiples else None,
        'ev_ebitda_high':   max(ebitda_multiples) if ebitda_multiples else None,
    }
    
    return {
        'target': target['name'],
        'metrics': METRICS,
        'rows': rows,
        'summary_stats': stats,
        'implied_ev_range': {
            'low':    target.get('ltm_ebitda_m', 0) * stats['ev_ebitda_low'],
            'median': target.get('ltm_ebitda_m', 0) * stats['ev_ebitda_median'],
            'high':   target.get('ltm_ebitda_m', 0) * stats['ev_ebitda_high'],
        } if all(v for v in [stats.get('ev_ebitda_low'), stats.get('ev_ebitda_median'), stats.get('ev_ebitda_high')]) else {}
    }
```

### 3. Earnings / Portfolio Company Update Processor

```python
def process_earnings_update(transcript_path: str, prior_model_path: str) -> dict:
    """
    Processes a portfolio company earnings update or management call transcript.
    Extracts key metrics, compares to prior period, and flags material changes.
    Returns structured update for analyst review.
    
    Tier 1 (read-only): Parses transcript
    Tier 2 (analysis): Compares to prior model
    """
    # Tier 1: Parse transcript (read-only)
    transcript = parse_transcript(transcript_path)
    mentioned_metrics = extract_financial_mentions(transcript)
    
    # Tier 2: Compare to prior model
    prior_model = load_prior_model(prior_model_path)
    changes = compare_to_prior(mentioned_metrics, prior_model)
    
    # Flag material changes
    material_flags = [
        change for change in changes
        if abs(change.get('variance_pct', 0)) > 0.10
    ]
    
    return {
        'company': transcript.get('company_name'),
        'call_date': transcript.get('call_date'),
        'mentioned_metrics': mentioned_metrics,
        'material_changes': material_flags,
        'action_required': len(material_flags) > 0,
        'suggested_model_updates': generate_model_update_suggestions(changes),
    }
```

### 4. Research Note Generator

```python
RESEARCH_NOTE_TEMPLATE = """
# {sector} Sector Research Note
**Date:** {date}  
**Prepared by:** SMC Investment Team  
**Classification:** Internal — Not for Distribution

---

## Sector Overview
{sector_overview}

## Market Dynamics
{market_dynamics}

## LMM Deal Activity
{deal_activity}

## Key Investment Themes
{investment_themes}

## Representative Companies
{comps_table}

## Risks & Watch Items
{risks}

## Deal Pipeline Implications
{pipeline_implications}
"""

def generate_research_note(sector_data: dict, comps_data: dict) -> str:
    """
    Generates a formatted research note from structured sector and comps data.
    Output is .docx or .md for IC distribution.
    """
    # Format sections
    # ...
    return RESEARCH_NOTE_TEMPLATE.format(**sections)
```

## Output Files

```
./out/screen-{sector}-{date}.json         ← Sector screen results
./out/comps-{company}-{date}.xlsx         ← Comparable company table
./out/note-{sector}-{date}.docx           ← Research note (Word)
./out/earnings-update-{company}-{date}.json  ← Portfolio company update
```

## Parallelization Pattern

For multi-company research pipelines:

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

async def research_coverage_list(companies: list[str]) -> list[dict]:
    """Process each company's research in parallel, one session per ticker."""
    with ThreadPoolExecutor(max_workers=5) as executor:
        loop = asyncio.get_event_loop()
        tasks = [
            loop.run_in_executor(executor, research_single_company, company)
            for company in companies
        ]
        results = await asyncio.gather(*tasks, return_exceptions=True)
    
    # Filter errors, log failures
    return [r for r in results if not isinstance(r, Exception)]
```

## Example Requests

```
Build a sector screener for healthcare services companies with $5M–$30M EBITDA. Filter to founder/family-owned businesses in the Southeast. Output a ranked list with revenue, EBITDA, and ownership type.
```

```
Generate a comps table for a commercial HVAC services company doing $18M EBITDA. Find 8–10 public and private comparables. Show EV/EBITDA, EV/Revenue, EBITDA margin, and revenue growth.
```

```
Process this portfolio company management call transcript. Compare the metrics mentioned to the prior quarter model and flag anything that changed by more than 10%.
```

```
Write a sector research note on government services — federal contracting and state/local consulting businesses. Focus on LMM deal flow, key risks, and current valuation benchmarks.
```

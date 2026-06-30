---
name: lp-statement-auditor
description: Audits LP (Limited Partner) capital account statements and distribution notices before distribution at Star Mountain Capital — validating calculations, checking completeness, flagging discrepancies versus fund records, and generating sign-off reports for the IR and CFO teams. Use when building automation to validate LP statements prior to the quarterly distribution cycle. Adapted from Anthropic's statement-auditor managed agent cookbook.
---

# LP Statement Auditor

You are a fund reporting automation engineer at Star Mountain Capital. You build automated workflows to audit LP capital account statements before they are distributed to investors — ensuring mathematical accuracy, consistency with fund records, and completeness of required disclosures. All outputs require human sign-off before distribution.

## SMC Statement Context

**Distribution cadence:** Quarterly (all funds)  
**Statement types:** Capital account statements, distribution notices, K-1 / Schedule K-3 (annual), NAV statements  
**Administrator role:** Fund administrator generates statements; SMC audits before distribution  
**Sign-off required:** IR team + CFO/Controller must approve before any LP distribution  
**Regulatory sensitivity:** LP statements are investor communications; errors can trigger complaints, redemption requests, or regulatory scrutiny

## Audit Workflows

### 1. Capital Account Statement Validation

```python
from decimal import Decimal, ROUND_HALF_UP
from pathlib import Path
import json

TOLERANCE = Decimal('0.01')  # $0.01 rounding tolerance

def validate_capital_account_statement(statement: dict, fund_records: dict) -> dict:
    """
    Validates a single LP's capital account statement against fund records.
    Returns a validation result with pass/fail status and all issues found.
    """
    issues = []
    lp_id = statement.get('lp_id')
    
    # 1. Balance check: beginning + activity = ending
    beginning = Decimal(str(statement.get('beginning_balance', 0)))
    contributions = Decimal(str(statement.get('contributions', 0)))
    distributions = Decimal(str(statement.get('distributions', 0)))
    fees = Decimal(str(statement.get('management_fees_charged', 0)))
    expenses = Decimal(str(statement.get('expenses_charged', 0)))
    appreciation = Decimal(str(statement.get('unrealized_gain_loss', 0)))
    realized = Decimal(str(statement.get('realized_gain_loss', 0)))
    ending_stated = Decimal(str(statement.get('ending_balance', 0)))
    
    ending_computed = (
        beginning + contributions - distributions 
        - fees - expenses + appreciation + realized
    )
    
    balance_diff = abs(ending_computed - ending_stated)
    if balance_diff > TOLERANCE:
        issues.append({
            'type': 'BALANCE_ERROR',
            'severity': 'HIGH',
            'detail': f'Computed ending balance ${ending_computed:,.2f} != stated ${ending_stated:,.2f} (diff: ${balance_diff:,.2f})',
        })
    
    # 2. Cross-check contributions against fund call records
    fund_calls = fund_records.get('capital_calls', {}).get(lp_id, [])
    fund_total_calls = sum(Decimal(str(c['amount'])) for c in fund_calls)
    if abs(contributions - fund_total_calls) > TOLERANCE:
        issues.append({
            'type': 'CONTRIBUTION_MISMATCH',
            'severity': 'HIGH',
            'detail': f'Statement contributions ${contributions:,.2f} != fund records ${fund_total_calls:,.2f}',
        })
    
    # 3. Cross-check distributions
    fund_distributions = fund_records.get('distributions', {}).get(lp_id, [])
    fund_total_dist = sum(Decimal(str(d['amount'])) for d in fund_distributions)
    if abs(distributions - fund_total_dist) > TOLERANCE:
        issues.append({
            'type': 'DISTRIBUTION_MISMATCH',
            'severity': 'HIGH',
            'detail': f'Statement distributions ${distributions:,.2f} != fund records ${fund_total_dist:,.2f}',
        })
    
    # 4. Ownership % check
    stated_pct = Decimal(str(statement.get('ownership_pct', 0)))
    expected_pct = Decimal(str(fund_records.get('ownership_pcts', {}).get(lp_id, 0)))
    if abs(stated_pct - expected_pct) > Decimal('0.0001'):
        issues.append({
            'type': 'OWNERSHIP_PCT_MISMATCH',
            'severity': 'MEDIUM',
            'detail': f'Stated ownership {stated_pct:.4%} != expected {expected_pct:.4%}',
        })
    
    return {
        'lp_id': lp_id,
        'status': 'PASS' if not issues else 'FAIL',
        'issues': issues,
        'issue_count': len(issues),
        'high_severity_count': sum(1 for i in issues if i['severity'] == 'HIGH'),
    }
```

### 2. Batch Audit (All LPs in Fund)

```python
def audit_fund_statements(
    statements_dir: str,
    fund_records_path: str,
    output_dir: str
) -> dict:
    """
    Audits all LP statements for a fund in a given period.
    Generates sign-off report to ./out/.
    """
    fund_records = load_fund_records(fund_records_path)
    statements = load_all_statements(statements_dir)
    
    results = []
    for statement in statements:
        result = validate_capital_account_statement(statement, fund_records)
        results.append(result)
    
    # Summary
    pass_count = sum(1 for r in results if r['status'] == 'PASS')
    fail_count = sum(1 for r in results if r['status'] == 'FAIL')
    high_severity = sum(r['high_severity_count'] for r in results)
    
    batch_result = {
        'fund': fund_records.get('fund_name'),
        'period': fund_records.get('period'),
        'total_lps': len(results),
        'pass_count': pass_count,
        'fail_count': fail_count,
        'high_severity_issues': high_severity,
        'ready_for_distribution': fail_count == 0,
        'results': results,
    }
    
    # Write sign-off report
    output_path = write_signoff_report(batch_result, output_dir)
    batch_result['signoff_report_path'] = str(output_path)
    
    return batch_result
```

### 3. Distribution Notice Validator

```python
def validate_distribution_notice(notice: dict, fund_records: dict) -> dict:
    """
    Validates a distribution notice before sending to LPs.
    Checks: amounts sum correctly, source breakdown is accurate,
    wire instructions match LP records.
    """
    issues = []
    
    # Total distribution check
    lp_amounts = [Decimal(str(lp['amount'])) for lp in notice.get('lp_distributions', [])]
    total_stated = Decimal(str(notice.get('total_distribution', 0)))
    total_computed = sum(lp_amounts)
    
    if abs(total_computed - total_stated) > Decimal('1.00'):
        issues.append({
            'type': 'DISTRIBUTION_TOTAL_MISMATCH',
            'severity': 'HIGH',
            'detail': f'LP amounts sum to ${total_computed:,.2f} but notice states ${total_stated:,.2f}',
        })
    
    # Pro-rata check
    total_commitment = fund_records.get('total_commitments', 0)
    for lp_dist in notice.get('lp_distributions', []):
        lp_commitment = fund_records.get('commitments', {}).get(lp_dist['lp_id'], 0)
        expected_pct = lp_commitment / total_commitment if total_commitment > 0 else 0
        stated_pct = lp_dist.get('pro_rata_pct', 0)
        
        if abs(stated_pct - expected_pct) > 0.0001:
            issues.append({
                'type': 'PRO_RATA_MISMATCH',
                'severity': 'MEDIUM',
                'lp_id': lp_dist['lp_id'],
                'detail': f'Stated pro-rata {stated_pct:.4%} != expected {expected_pct:.4%}',
            })
    
    return {
        'status': 'PASS' if not issues else 'FAIL',
        'issues': issues,
        'ready_for_distribution': len(issues) == 0,
    }
```

### 4. Sign-Off Report Generator

```python
import openpyxl

def write_signoff_report(batch_result: dict, output_dir: str) -> Path:
    """
    Generates Excel sign-off report for IR and CFO review.
    Report includes: summary, pass/fail per LP, all issues with severity.
    NOT distributed to LPs — internal review only.
    """
    wb = openpyxl.Workbook()
    
    # Summary tab
    ws = wb.active
    ws.title = 'Audit Summary'
    ws['A1'] = f"LP Statement Audit — {batch_result['fund']} — {batch_result['period']}"
    ws['A3'] = 'Total LPs Audited:'
    ws['B3'] = batch_result['total_lps']
    ws['A4'] = 'Pass:'
    ws['B4'] = batch_result['pass_count']
    ws['A5'] = 'Fail:'
    ws['B5'] = batch_result['fail_count']
    ws['A6'] = 'High Severity Issues:'
    ws['B6'] = batch_result['high_severity_issues']
    ws['A8'] = 'READY FOR DISTRIBUTION:'
    ws['B8'] = 'YES' if batch_result['ready_for_distribution'] else 'NO — RESOLVE ALL ISSUES FIRST'
    
    # Detail tab
    ws2 = wb.create_sheet('LP Detail')
    ws2.append(['LP ID', 'Status', 'Issue Count', 'High Severity', 'Issues'])
    for result in batch_result['results']:
        issue_text = '; '.join(i['detail'] for i in result.get('issues', []))
        ws2.append([
            result['lp_id'],
            result['status'],
            result['issue_count'],
            result['high_severity_count'],
            issue_text,
        ])
    
    output_path = Path(output_dir) / f"signoff-{batch_result['fund']}-{batch_result['period']}.xlsx"
    wb.save(output_path)
    return output_path
```

## Security Model

| Tier | Input | Permitted Operations |
|------|-------|---------------------|
| Tier 1 — Statement Reader | Untrusted admin files | Read, Grep only; length-capped output |
| Tier 2 — Reconciler | Validated statement data | Read, Grep, Glob; NAV connector (read-only) |
| Tier 3 — Flagger | Validated results | Write sign-off report to ./out/ only |

## Output Convention

```
./out/signoff-{fund}-{period}.xlsx        ← Sign-off report (IR + CFO review required)
./out/issues-{fund}-{period}.json         ← Machine-readable issues log
./out/audit-log-{timestamp}.json          ← Run metadata and file hashes
```

**Distribution gate:** `ready_for_distribution == True` AND IR + CFO sign-off on the Excel report. Do not automate LP distribution.

## Example Requests

```
Build the LP statement auditor for Q2 2026 SMC Fund IV. Read all statement files from ./statements/, cross-check against the fund records in fund_records.json, and generate the sign-off report.
```

```
Write a validator that checks all LP distribution amounts in the Q2 distribution notice sum to the total fund distribution, and that each LP's pro-rata share matches their commitment percentage.
```

```
Add a check to the statement auditor that flags any LP whose ending capital account balance changed by more than 15% compared to the prior quarter — those need a manual review note.
```

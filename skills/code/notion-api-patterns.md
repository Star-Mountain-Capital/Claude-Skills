---
name: notion-api-patterns
description: Provides SMC-specific Notion API patterns for building and maintaining the Meridian FinOps Hub — the firm's internal Azure-hosted Node/Express application that integrates with Notion as its data layer. Use when building Notion database integrations, writing Meridian backend routes that call the Notion API, or automating FinOps workflows through Notion. Includes patterns for deal tracking, fund reporting, LP data, and workflow automation.
---

# Notion API Patterns — SMC Meridian

You are a senior backend engineer working on Star Mountain Capital's Meridian FinOps Hub — an Azure-hosted Node.js/Express single-page application that uses Notion as its primary operational data layer. You write production-quality Notion API integration code that is clean, error-tolerant, and maintainable by a small internal team.

## Meridian Architecture Context

**Stack:** Node.js + Express (backend), Notion API (data layer), Azure (hosting)  
**Notion client:** `@notionhq/client` (official SDK)  
**Use cases:** Deal tracking, fund reporting dashboards, LP data management, FinOps workflow automation, period status tracking  
**Environment:** `NOTION_API_KEY` and database IDs stored in Azure App Configuration / environment variables

## Setup

```javascript
const { Client } = require('@notionhq/client');

const notion = new Client({ auth: process.env.NOTION_API_KEY });

// Database IDs — store in config, not hardcoded
const DB = {
  deals:          process.env.NOTION_DEALS_DB_ID,
  funds:          process.env.NOTION_FUNDS_DB_ID,
  lps:            process.env.NOTION_LP_DB_ID,
  periodStatus:   process.env.NOTION_PERIOD_STATUS_DB_ID,
  portfolio:      process.env.NOTION_PORTFOLIO_DB_ID,
  capitalCalls:   process.env.NOTION_CAPITAL_CALLS_DB_ID,
};
```

## Core Patterns

### Pattern 1: Querying a Database with Filters

```javascript
async function queryDeals({ stage, fundId, assignee } = {}) {
  const filters = [];

  if (stage) {
    filters.push({
      property: 'Stage',
      select: { equals: stage }
    });
  }

  if (fundId) {
    filters.push({
      property: 'Fund',
      relation: { contains: fundId }
    });
  }

  if (assignee) {
    filters.push({
      property: 'Deal Lead',
      people: { contains: assignee }
    });
  }

  const response = await notion.databases.query({
    database_id: DB.deals,
    filter: filters.length === 1
      ? filters[0]
      : { and: filters },
    sorts: [{ property: 'Last Updated', direction: 'descending' }],
    page_size: 100,
  });

  return response.results.map(parseDealPage);
}
```

### Pattern 2: Property Parser (handles all Notion property types)

```javascript
function parseProperty(prop) {
  if (!prop) return null;
  switch (prop.type) {
    case 'title':        return prop.title.map(t => t.plain_text).join('');
    case 'rich_text':    return prop.rich_text.map(t => t.plain_text).join('');
    case 'number':       return prop.number;
    case 'select':       return prop.select?.name ?? null;
    case 'multi_select': return prop.multi_select.map(s => s.name);
    case 'date':         return prop.date?.start ?? null;
    case 'checkbox':     return prop.checkbox;
    case 'url':          return prop.url;
    case 'email':        return prop.email;
    case 'phone_number': return prop.phone_number;
    case 'relation':     return prop.relation.map(r => r.id);
    case 'formula':      return parseProperty(prop.formula);
    case 'rollup':       return prop.rollup?.array?.map(parseProperty) ?? null;
    case 'people':       return prop.people.map(p => ({ id: p.id, name: p.name }));
    case 'files':        return prop.files.map(f => f.external?.url ?? f.file?.url);
    case 'status':       return prop.status?.name ?? null;
    default:             return null;
  }
}

function parseDealPage(page) {
  const p = page.properties;
  return {
    id: page.id,
    name:         parseProperty(p['Deal Name']),
    stage:        parseProperty(p['Stage']),
    ebitda:       parseProperty(p['LTM EBITDA']),
    entryMultiple: parseProperty(p['Entry Multiple']),
    fundId:       parseProperty(p['Fund']),
    dealLead:     parseProperty(p['Deal Lead']),
    status:       parseProperty(p['Status']),
    lastUpdated:  parseProperty(p['Last Updated']),
  };
}
```

### Pattern 3: Creating a Page in a Database

```javascript
async function createCapitalCall({ fundId, callNumber, amount, purpose, dueDate }) {
  return notion.pages.create({
    parent: { database_id: DB.capitalCalls },
    properties: {
      'Call Name': {
        title: [{ text: { content: `Capital Call #${callNumber}` } }]
      },
      'Fund': {
        relation: [{ id: fundId }]
      },
      'Call Amount': {
        number: amount
      },
      'Purpose': {
        rich_text: [{ text: { content: purpose } }]
      },
      'Due Date': {
        date: { start: dueDate }
      },
      'Status': {
        select: { name: 'Draft' }
      },
    }
  });
}
```

### Pattern 4: Updating a Page's Properties

```javascript
async function updatePeriodStatus(pageId, { status, completedAt, notes }) {
  const properties = {};

  if (status) {
    properties['Status'] = { select: { name: status } };
  }
  if (completedAt) {
    properties['Completed At'] = { date: { start: completedAt } };
  }
  if (notes) {
    properties['Notes'] = {
      rich_text: [{ text: { content: notes } }]
    };
  }

  return notion.pages.update({ page_id: pageId, properties });
}
```

### Pattern 5: Pagination (handle all results)

```javascript
async function getAllPages(databaseId, filter) {
  const results = [];
  let cursor;

  do {
    const response = await notion.databases.query({
      database_id: databaseId,
      filter,
      start_cursor: cursor,
      page_size: 100,
    });
    results.push(...response.results);
    cursor = response.has_more ? response.next_cursor : undefined;
  } while (cursor);

  return results;
}
```

### Pattern 6: Express Route Integration

```javascript
// GET /api/deals?stage=IC&fund=SMCIV
router.get('/deals', async (req, res) => {
  try {
    const { stage, fund } = req.query;
    const deals = await queryDeals({ stage, fundId: fund });
    res.json({ success: true, data: deals, count: deals.length });
  } catch (err) {
    if (err.code === 'object_not_found') {
      return res.status(404).json({ error: 'Database not found' });
    }
    if (err.code === 'rate_limited') {
      return res.status(429).json({ error: 'Notion rate limit hit, retry after 1s' });
    }
    console.error('Notion query error:', err);
    res.status(500).json({ error: 'Failed to fetch deals' });
  }
});
```

## SMC-Specific Database Schemas

### Deals Database
| Property | Type | Notes |
|----------|------|-------|
| Deal Name | Title | Company name |
| Stage | Select | First Look, Diligence, IC, Closed, Passed |
| Fund | Relation | → Funds DB |
| LTM EBITDA | Number | $ millions |
| Entry Multiple | Number | x EBITDA |
| Deal Lead | People | SMC team member |
| Status | Status | Active, On Hold, Closed |

### Period Status Database
| Property | Type | Notes |
|----------|------|-------|
| Period | Title | e.g., "Q2 2026" |
| Fund | Relation | → Funds DB |
| Workflow Step | Select | Close tasks |
| Status | Status | Not Started, In Progress, Complete, Blocked |
| Owner | People | FinOps team member |
| Due Date | Date | Expected completion |

## Rate Limiting

Notion's API is limited to ~3 requests/second. For batch operations:

```javascript
const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));

async function batchUpdate(pages, updateFn, delayMs = 350) {
  const results = [];
  for (const page of pages) {
    results.push(await updateFn(page));
    await sleep(delayMs);
  }
  return results;
}
```

## Example Requests

```
Write a Meridian Express route that queries the Period Status database for all incomplete Q2 2026 workflow steps and returns them grouped by fund.
```

```
Build a function that syncs capital call records from the Notion Capital Calls database into a normalized JSON structure for the Meridian reporting dashboard.
```

```
Write a batch update function that marks all period status workflow steps for SMCIV as "Complete" when the quarter-end close is signed off.
```

```
Create a Notion page parser that handles all property types and maps the LP database schema into a clean JavaScript object.
```

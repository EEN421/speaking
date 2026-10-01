# KQL Café — Sentinel Cost Optimization & KQL Runner

**January 28, 2025**

A two-part KQL Café session covering practical Microsoft Sentinel cost optimization with KQL and the construction of a lightweight web interface for discovering and launching KQL queries using Microsoft Entra ID authentication.

## Session Overview

This session explored two related problems:

1. **Understanding and reducing Microsoft Sentinel ingestion costs with KQL**
2. **Making reusable KQL easier to discover and execute through GitHub, MSAL authentication, and Microsoft Sentinel**

The first half uses the `Usage` table and `_BilledSize` to identify where Sentinel ingestion volume — and therefore cost — is coming from.

The second half moves from individual queries to **KQL as reusable tooling**, demonstrating a browser-based application that authenticates users with Microsoft Entra ID, discovers their Azure environment, retrieves KQL from GitHub, and opens those queries against the user's own Sentinel workspace.

---

## Part I — Sentinel Cost Optimization

The cost-optimization portion demonstrates how KQL can turn Sentinel ingestion telemetry into actionable questions:

- How much billable data are we ingesting?
- Which solutions and tables account for the most volume?
- How has ingestion changed over time?
- Which event types are generating the greatest cost?
- Where should optimization efforts begin?

### Ingest Volume

```kusto
Usage
| where IsBillable == true
| summarize TotalVolumeGB = sum(Quantity) / 1000
    by bin(StartTime, 1d), Solution
| render columnchart
```

This provides a daily view of billable ingestion volume grouped by solution.

### Seven-Day Cost Summary

```kusto
Usage
| where TimeGenerated > ago(7d)
| where IsBillable == true
| summarize TotalGB = round(sum(Quantity) / 1024, 2)
| extend ['Daily Average (GB)'] = round(TotalGB / 7, 2)
| extend ['Weekly Total (GB)'] = TotalGB
| extend ['Estimated Monthly Cost ($)'] =
    round(TotalGB / 7 * 30 * 2.0, 2)
| project
    ['Daily Average (GB)'],
    ['Weekly Total (GB)'],
    ['Estimated Monthly Cost ($)']
```

> **Note:** The `$2.00/GB` value used in the original presentation was an example rate for demonstrating the calculation. Current Sentinel pricing, commitment tiers, and regional pricing should be used when applying these queries today.

### Volume Change Analysis

The presentation also compares the previous 30-day ingestion period with the current 30-day period using a `fullouter` join.

This makes it possible to identify:

- rapidly growing sources,
- shrinking sources,
- newly introduced data,
- removed data,
- and sources whose ingestion behavior warrants investigation.

### Top Data Sources

```kusto
Usage
| where TimeGenerated > ago(30d)
| where IsBillable == true
| summarize TotalGB = round(sum(Quantity) / 1024, 2) by DataType
| top 10 by TotalGB desc
```

Rather than treating Sentinel cost as one large number, this breaks the problem down into the individual tables producing it.

### Event-Level Cost Analysis

```kusto
SecurityEvent
| where TimeGenerated > ago(30d)
| summarize
    Count = count(),
    SizeGB = round(sum(_BilledSize) / 1024 / 1024 / 1024, 2)
    by EventID
| top 10 by SizeGB desc
```

This moves the analysis one level deeper: from expensive **tables** to expensive **events within those tables**.

---

## Part II — Brewing Queries with MSAL Authentication

The second half of the session explored how reusable KQL could be surfaced through a simple browser application.

The idea became **KQL Query Runner**.

### Architecture

```text
             Microsoft Entra ID
                    │
                    │ Authentication
                    ▼
             ┌───────────────┐
             │   KQL Runner  │
             │   Web Client  │
             └───────┬───────┘
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
   Azure Environment          GitHub
   / Workspace Context       KQL Repository
          │                     │
          └──────────┬──────────┘
                     │
                     ▼
              Query Preview
                     │
                     ▼
            Microsoft Sentinel
```

The application demonstrates several useful patterns:

- Microsoft Entra ID application registration
- MSAL-based interactive authentication
- Delegated user access
- Azure subscription and workspace selection
- GitHub-hosted KQL discovery
- Query preview
- Encoding KQL into an Azure portal Logs blade URL
- Launching reusable queries in the user's own Sentinel workspace

The presentation's implementation compresses and encodes the selected query before constructing an Azure portal URL targeting the selected Log Analytics workspace.

---

## KQL Query Runner

The original project remains available here:

**[KQL Query Runner — Live Site](https://kql-runner.github.io/)**

**[KQL Query Runner — Source Code](https://github.com/KQL-Runner/KQL-Runner.github.io/tree/main)**

The application provides:

```text
Sign in with Azure
        ↓
Select Subscription
        ↓
Select Workspace
        ↓
Select KQL Query
        ↓
Preview Query
        ↓
Run in Sentinel
```

The project was intended as a practical demonstration of how a GitHub-hosted KQL collection could become an interactive tool rather than simply a directory of `.kql` files.

---

## Related Sentinel Deployment Automation

The session also referenced an automated Sentinel/XDR deployment workflow that can rapidly create a Microsoft Sentinel environment and populate it with relevant Content Hub solutions, connectors, and analytics rules.

**[Sentinel XDR Easy Deploy](https://www.hanley.cloud/2025-01-27-Sentinel-XDR-Easy-Deploy/)**

This provides a useful companion to KQL Runner:

```text
Deploy Sentinel
       ↓
Connect Security Data
       ↓
Deploy Detection Content
       ↓
Authenticate with KQL Runner
       ↓
Select Workspace
       ↓
Run / Investigate with KQL
```

---

## Presentation Materials

### Sentinel Cost Optimization

Topics include:

- Sentinel ingestion volume
- Cost estimation
- 30-day ingestion comparisons
- Top data sources
- Event-level ingestion analysis
- Table-level volume analysis
- Sentinel workbook visualization

### Brewing Queries with MSAL Authentication

Topics include:

- MSAL
- Microsoft Entra ID app registration
- User authentication
- Delegated access
- GitHub API query discovery
- Azure workspace selection
- Query encoding
- Launching KQL in Sentinel

---

## Key Idea

The common thread between both halves of the session was that **KQL becomes much more useful when it moves beyond one-off queries**.

A query can become:

- a cost-control mechanism,
- a reusable analytic,
- a workbook,
- a report,
- a GitHub-hosted artifact,
- or part of an application.

The query is often only the beginning.

---

## Resources

See **[REFERENCES.md](./REFERENCES.md)** for source code, supporting articles, Microsoft documentation, and additional material.
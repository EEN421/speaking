# References & Further Reading

Supporting code, projects, articles, and documentation for the January 2025 KQL Café session on **Microsoft Sentinel Cost Optimization** and **Brewing Queries with MSAL Authentication**.

---

# KQL Runner

## KQL Query Runner

**Topic:** GitHub-hosted KQL, Microsoft Entra ID authentication, workspace discovery, and launching queries against Microsoft Sentinel.

- [KQL Query Runner — Live Application](https://kql-runner.github.io/)
- [KQL Query Runner — GitHub Repository](https://github.com/KQL-Runner/KQL-Runner.github.io/tree/main)

The application demonstrated during the session allows a user to:

1. Authenticate with Microsoft Entra ID
2. Select an Azure subscription
3. Select a Log Analytics workspace
4. Browse KQL stored in GitHub
5. Preview a selected query
6. Launch that query against Microsoft Sentinel

The original presentation describes the application as an interactive KQL interface backed by Microsoft authentication and GitHub-hosted queries.

---

## Microsoft Authentication Library — MSAL

**Topic:** Authentication for the browser-based KQL Runner.

- [Microsoft — MSAL Browser](https://learn.microsoft.com/en-us/entra/msal/javascript/browser/about-msal-browser)
- [Microsoft — Initialize MSAL Browser](https://learn.microsoft.com/en-us/entra/msal/javascript/browser/initialization)

MSAL enables browser applications to authenticate users against Microsoft Entra ID and obtain tokens for authorized Microsoft services.

KQL Runner uses this model so queries execute in the context of the authenticated user's Azure environment rather than through embedded application credentials.

---

## Azure Monitor Logs Query API

**Topic:** Programmatically querying Log Analytics workspaces with KQL.

- [Azure Monitor Logs Query API Overview](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/api/overview)
- [Azure Monitor Logs Query API Request Format](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/api/request-format)

The Logs Query API provides a REST interface for executing KQL against Log Analytics workspaces using an authenticated Azure identity.

Typical endpoint:

```text
POST https://api.loganalytics.azure.com/v1/workspaces/{workspaceId}/query
```

Microsoft's API supports KQL queries, time ranges, and cross-workspace execution.

---

# Sentinel Cost Optimization

## Sentinel Cost Optimization Workbook

**Topic:** Turning the individual cost-analysis queries from the presentation into a reusable Microsoft Sentinel workbook.

- [Sentinel Cost Optimization Workbook Gallery Template](https://github.com/EEN421/Sentinel_Cost_Optimization/blob/Main/Cost%20Optimization%20Workbook%20Gallery%20Template%20%20V1.json)

The presentation combines its ingest-volume, source, event, and trend queries into a single Sentinel workbook for ongoing analysis.

---

## Power BI & Log Analytics Workspace

**Topic:** Converting KQL-backed Log Analytics data into reusable Power BI reporting.

- [PowerBI & Log Analytics Workspace](https://www.hanley.cloud/2024-01-19-PowerBI-%26-Log-Analytics-Workspace/)

This extends the cost-analysis workflow beyond Sentinel workbooks by exporting Log Analytics queries as Power BI M queries and building reusable ingest-trend reports.

It is especially relevant to recurring operational reporting such as:

- 90-day ingest trends
- 60-day ingest trends
- 30-day ingest trends
- 7-day ingest trends
- quarterly security reporting

---

## Microsoft Sentinel Workbooks

**Topic:** Visualizing KQL-backed Sentinel data and turning queries into reusable operational dashboards.

- [Microsoft — Visualize and Monitor Data Using Microsoft Sentinel Workbooks](https://learn.microsoft.com/en-us/azure/sentinel/tutorial-monitor-your-data)
- [Microsoft — Create or Edit an Azure Workbook](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-create-workbook)

Microsoft Sentinel workbooks build on Azure Monitor Workbooks and can combine KQL queries, parameters, tables, and charts into reusable security dashboards.

---

# Sentinel Deployment Automation

## Sentinel XDR Easy Deploy

**Topic:** Rapid Microsoft Sentinel/XDR deployment and Content Hub automation.

- [Sentinel XDR Easy Deploy](https://www.hanley.cloud/2025-01-27-Sentinel-XDR-Easy-Deploy/)

This companion project demonstrates rapid deployment of:

- Azure Resource Group
- Log Analytics Workspace
- Microsoft Sentinel
- Content Hub solutions
- Analytics Rules
- Data Connectors

The workflow provides a quick way to stand up an environment that can then be explored using the queries and tooling discussed during the KQL Café session.

---

# Related KQL

## Cost of Syslog Events by Severity

**Topic:** Moving from table-level ingestion cost to the specific event categories responsible for that cost.

- [Cost of Syslog Events by Severity — GitHub](https://github.com/EEN421/KQL-Queries/blob/Main/Cost%20of%20Syslog%20Events%20by%20Severity.kql)

This is a practical extension of the presentation's event-cost analysis pattern:

```text
Workspace Cost
      ↓
Table Cost
      ↓
Event / Category Cost
      ↓
Optimization Decision
```

---

# Presentation Source Material

## Sentinel Cost Optimization

The original presentation includes KQL for:

- daily billable ingestion volume,
- seven-day cost estimation,
- previous-vs-current 30-day ingestion comparisons,
- top billable data sources,
- Event ID cost analysis,
- and 90-day table volumes.

The individual queries are intended to be combined into a Sentinel workbook for ongoing cost visibility.

## Brewing Queries with MSAL Authentication

The original presentation covers:

- MSAL
- Microsoft Entra ID application registration
- delegated user authentication
- GitHub API integration
- KQL retrieval
- query compression and encoding
- Azure subscription and workspace context
- launching queries into the Azure Logs blade

---

> **Event:** KQL Café  
> **Date:** January 28, 2025  
> **Speaker:** Ian Hanley  
> **Topics:** KQL, Microsoft Sentinel, cost optimization, Log Analytics, MSAL, Microsoft Entra ID, GitHub, security tooling, and automation
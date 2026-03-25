# Account Usage Dashboard v2 - Genie Enabled

A Databricks AI/BI dashboard for monitoring DBU consumption, dollar cost estimation, and infrastructure usage across your entire Databricks account. This is a Genie-compatible fork of the [Account Usage Dashboard v2](https://github.com/CodyAustinDavis/dbsql_sme/tree/main/Observability%20Dashboards%20and%20DBA%20Resources/Account%20Usage%20Dashboard).

> **Disclaimer:** This is not an official Databricks product. It is provided as-is with no warranty or support. Use at your own discretion. The underlying system tables and SQL functions referenced may change without notice.

## What's Different From the Original

The original dashboard uses advanced SQL patterns (`IDENTIFIER()`, nested CTEs, `SET` statements, `try_variant_get`) that are incompatible with Databricks Genie. This version rewrites all 28 dataset queries to use Genie-compatible SQL while preserving identical functionality:

- `IDENTIFIER(:price_table)` dynamic table references replaced with direct `system.billing.list_prices`
- `identifier(case when ...)` dynamic column references replaced with explicit `CASE` expressions
- `try_variant_get(to_variant_object(pricing), ...)` replaced with `pricing.effective_list.default`
- Nested CTEs (`WITH` inside `WITH`) flattened to top-level CTEs
- `SET ansi_mode = true;` multi-statement patterns removed
- `explode(array(...))` parameter dropdowns replaced with `VALUES` clauses
- `$$$__WORKSPACES_JSON__$$$` macro replaced with direct `workspaces_latest` join

The result: the "Ask Genie" button works on the published dashboard, enabling natural language Q&A over your account usage data.

## Dashboard Pages

| Page | Description |
|------|-------------|
| **Usage Overview** | Total spend (DBUs and dollars), breakdown by product/workspace/SKU, time-series trends, run rate, and forecasting |
| **Usage Overview - Top N** | Top N spending objects ranked by any metadata key (job, cluster, warehouse, notebook, pipeline, endpoint, user) |
| **Usage Analysis - Tag Matching** | Analyze custom tag coverage, spend by tag key-value pairs, and tag matching rates |
| **Usage Analysis - Top Objects** | Period-over-period comparison of top spending objects with drill-down |
| **README** | In-dashboard documentation and data source references |

## Data Sources

Built entirely on Databricks system tables (no custom tables required):

| Table | Purpose |
|-------|---------|
| `system.billing.usage` | Core DBU consumption records |
| `system.billing.list_prices` | Published list pricing per SKU |
| `system.access.workspaces_latest` | Workspace names and metadata |
| `system.compute.clusters` | Cluster definitions and configuration |
| `system.compute.warehouses` | SQL warehouse definitions |
| `system.lakeflow.jobs` | Job definitions |
| `system.lakeflow.pipelines` | Pipeline definitions |
| `system.serving.served_entities` | Model serving endpoint metadata |

## Feature Toggles

The dashboard includes configurable feature toggles in the global filters:

| Toggle | Default | Notes |
|--------|---------|-------|
| **Forecast** | Enabled | Uses `AI_FORECAST` to project future usage. Set to **Disabled** if `AI_FORECAST` is not available in your workspace. |
| **Forecast Horizon** | 3 | Number of time periods to forecast forward (only applies when Forecast is Enabled). |
| **Price Table** | `system.billing.list_prices` | Cost estimation uses list prices. The `account_prices` option is available for accounts with negotiated pricing -- if your account does not have `system.billing.account_prices` enabled, leave this on `list_prices`. |
| **Billing Object Links** | Enabled | Generates clickable hyperlinks to clusters, jobs, warehouses, and pipelines. Set to **Disabled** if not needed. |
| **Discount Overrides** | None | Semicolon-separated list of product-level discount overrides (e.g., `JOBS=0.10;SQL=0.15` for 10% and 15% discounts). Use `*=0.20` for a global 20% discount across all products. |

## Prerequisites

- **System tables enabled** at the account level (admin setting)
- **SELECT access** to the system tables listed above
- **SQL warehouse** for query execution
- Works on **AWS, Azure, and GCP** (pricing joins filter by cloud automatically)

## How to Import

1. Download the `account_usage_dashboard_v2_genie.lvdash.json` file
2. In your Databricks workspace, navigate to your user folder
3. Click **Import** and select the JSON file
4. Open the imported dashboard and click **Publish**
5. The "Ask Genie" button will appear on the published dashboard, auto-generating a Genie Space from the dashboard's SQL

Alternatively, use the Databricks CLI:

```bash
databricks workspace import \
  account_usage_dashboard_v2_genie.lvdash.json \
  /Workspace/Users/<your-email>/account_usage_dashboard_v2_genie.lvdash.json
```

## Known Limitations

- **Price table toggle**: The `account_prices` option requires `system.billing.account_prices` to be enabled on your account. If it is not available, selecting it will cause query errors. Leave on `list_prices` if unsure.
- **Forecast**: Requires the `AI_FORECAST` SQL function. If unavailable, set the Forecast toggle to Disabled.
- **Billing Object Links**: Hyperlinks point to the workspace where the resource exists. Cross-workspace links require appropriate access permissions.
- **Genie accuracy**: The auto-generated Genie Space works well for straightforward questions. For complex multi-join queries, results may vary.

## Credits

Based on the [Account Usage Dashboard v2](https://github.com/CodyAustinDavis/dbsql_sme/tree/main/Observability%20Dashboards%20and%20DBA%20Resources/Account%20Usage%20Dashboard) by [Cody Austin Davis](https://github.com/CodyAustinDavis). Modified for Genie compatibility by Kiara Koeppen.

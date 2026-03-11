<div align="center">

# Google BigQuery MCP Server by Coupler.io

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Transport](https://img.shields.io/badge/Transport-Streamable_HTTP-blue.svg)](#)
[![Auth](https://img.shields.io/badge/Auth-OAuth_2.0-green.svg)](#)

Coupler.io Google BigQuery MCP server for Claude, ChatGPT, Gemini, Cursor, n8n, OpenClaw, and other MCP clients. Query and analyze Google BigQuery data with natural language. Requires the Coupler.io account.

</div>

## Data Access

Run custom SQL queries against any BigQuery dataset — full flexibility to extract, filter, aggregate, and join your data.

<details>
<summary><strong>Custom SQL queries</strong></summary>

BigQuery lets you write custom SQL queries to extract exactly the data you need. The data structure depends entirely on your SQL — you control which tables, columns, and rows are included.

**Basic table export:**
```sql
SELECT * FROM project_id.dataset_name.table_name
```

**Filtered and aggregated data:**
```sql
SELECT
  DATE(order_date) as date,
  product_category,
  COUNT(*) as order_count,
  SUM(revenue) as total_revenue
FROM project_id.dataset_name.orders
WHERE order_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
GROUP BY date, product_category
ORDER BY date DESC
```

**Joining multiple tables:**
```sql
SELECT
  o.order_id,
  c.customer_name,
  p.product_name,
  o.quantity,
  o.order_date
FROM project_id.dataset_name.orders o
JOIN project_id.dataset_name.customers c ON o.customer_id = c.customer_id
JOIN project_id.dataset_name.products p ON o.product_id = p.product_id
WHERE o.order_date >= CURRENT_DATE()
```

</details>


## Supported Clients

*Note: You will need to set up a data flow in Coupler.io with Google BigQuery as a source and the AI tool of your choice as the destination.*

### Claude

Use with Claude Web, Desktop, Chat, Cowork, or Claude Code.

**Via Web/Desktop:** Go to **Customize**->**Connectors**->**Connect your tools**, search for Coupler.io and add it.

**Via Claude Code CLI:**

```bash
claude mcp add coupler-io --transport streamable-http https://mcp.coupler.io/mcp
```

### ChatGPT

Install from the **ChatGPT Apps** directory — search for "Coupler.io" in the **Apps** section of **Settings**.

### Cursor

Find it on the [Cursor Directory](https://cursor.directory/mcp/coupler-io-official-remote-mcp).

### Gemini CLI

Go to the **AI integrations** -> **Gemini CLI** page in your Coupler.io account to copy the correct command (unique to each account). It will look like this:

```bash
gemini mcp add coupler --transport=http https://mcp.coupler.io/mcp/xxxxx
```

### OpenClaw

Use the **mcporter** skill to connect, or install the **coupler-io** skill from ClawHub.
Directly ask your OpenClaw agent to add the skill and execute it.

## Links

- **Landing page:** [Google BigQuery MCP by Coupler.io](https://www.coupler.io/mcp/bigquery)
- **Coupler.io:** [https://coupler.io](https://coupler.io)
- **MCP Server endpoint:** `https://mcp.coupler.io/mcp`
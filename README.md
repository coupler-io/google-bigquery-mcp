<div align="center">

# Google BigQuery MCP Server by Coupler.io

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Transport](https://img.shields.io/badge/Transport-Streamable_HTTP-blue.svg)](#)
[![Auth](https://img.shields.io/badge/Auth-OAuth_2.0-green.svg)](#)

Connect Google BigQuery data to AI with the Coupler.io MCP server. Query and analyze data stored in BigQuery using natural-language questions in ChatGPT, Claude, Gemini, Cursor, and other MCP-compatible AI tools, with access to the datasets and tables configured through Coupler.io. Requires a Coupler.io account.

[Landing page](https://www.coupler.io/mcp/bigquery) · [Documentation](https://docs.coupler.io/ai/mcp) · [All Coupler.io MCP integrations](https://github.com/coupler-io)

</div>

## What you can ask

- Show revenue by product category for the last 30 days.
- Which customer segments have the highest lifetime value?
- Compare monthly revenue year over year.
- Find unusual changes in yesterday's data.
- Join orders and customers and summarize revenue by segment.

## How it works

This repository documents the Google BigQuery integration for the Coupler.io MCP server.

1. Connect Google BigQuery to Coupler.io.
2. Select your AI tool as the destination.
3. Connect your AI client to Coupler.io MCP.
4. Ask questions about your Google BigQuery data in natural language.

Coupler.io sits between Google BigQuery and your AI client. It holds the Google BigQuery credential, imports the data on a schedule, and exposes the result as a data set the AI can query. Your AI client never calls the Google BigQuery API itself.

```
  Google BigQuery
      |        credential held by Coupler.io
      v
  Coupler.io          import, transform, store on a schedule
      |
      v
  MCP server          schema, SQL query execution
      |
      v
  Your AI client      your question, in plain language
```

When you ask a question, the AI reads the data set's schema, writes SQL, and Coupler.io runs that query on its own side. Only the result comes back to the AI, so a large data set does not have to fit into the model's context window.

| | |
|---|---|
| **MCP endpoint** | `https://mcp.coupler.io/mcp` |
| **Transport** | Streamable HTTP |
| **Authentication** | OAuth 2.0 |
| **Query language** | SQL, executed by Coupler.io |
| **Refresh schedule** | From monthly to every 15 minutes, depending on your plan |

## Get started

*Note: You will need to set up a data flow in Coupler.io with Google BigQuery as a source and the AI tool of your choice as the destination.*

### Claude

Use with Claude Web, Desktop, Chat, Cowork, or Claude Code.

**Via Web/Desktop:** Go to **Customize**->**Connectors**->**Connect your tools**, search for Coupler.io and add it.

**Via Claude Code CLI:**

```bash
claude mcp add coupler-io --transport streamable-http https://mcp.coupler.io/mcp
```

### ChatGPT

Install the [Coupler.io ChatGPT app](https://l.rw.rw/couplerio-chatgpt-app) and complete the authentication. You can also find it by searching for "Coupler.io" in the **Apps** section of **Settings**.

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

## Data you can access

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

Unlike the packaged integrations, your SQL query defines the BigQuery source. Whatever that query returns becomes the data set the AI can reach, so you control the schema.

## Example questions

### Exploration

- What columns are in this data set, and what does each one mean?
- Summarize revenue by product category for the last 30 days.
- Find unusual changes in yesterday's data compared with the trailing 30-day average.

### Joins and segments

- Join orders and customers and summarize revenue by customer segment.
- Which customer segments have the highest lifetime value?
- Which cohorts acquired this year have the strongest repeat purchase rate?

### Reporting

- Compare monthly revenue year over year.
- Build a weekly revenue and order-count trend for the last 12 weeks.
- Which product categories are growing fastest by revenue?

## Security and permissions

Your AI client never connects to Google BigQuery directly. Coupler.io holds the Google BigQuery credential, imports the data, and exposes only the resulting data set over MCP.

- **Your Google BigQuery data is never modified.** Coupler.io only reads from Google BigQuery. The AI queries the copy Coupler.io imported and cannot edit, delete, or overwrite it, let alone write anything back to your Google BigQuery account.
- **Visibility is scoped per AI tool.** An AI client sees only the data sets from data flows that have *that client* set as a destination. Adding Claude as a destination does not expose the data flow to ChatGPT, though one flow can name both.
- **Configuration changes are possible, and confirmed first.** With the full tool set available, the AI can create data flows, add sources and destinations, change a schedule, or trigger a run. Those are real changes to your workspace, so the server instructs the AI to confirm before making one you did not ask for.
- **Nothing else is reachable.** The MCP server exposes the data sets described above and nothing more. It cannot reach your other accounts or your machine.
- **Disconnect at any time** by removing the connector in your AI client, or by deleting the credential or the data flow in Coupler.io.

Coupler.io is SOC 2 certified and compliant with GDPR and HIPAA.

## Troubleshooting

**The AI cannot find my Google BigQuery data set.**
Usually you have not added that AI tool as a destination yet. Open the data flow in Coupler.io and add your AI client. One data flow can have several AI tools as destinations at the same time, so adding ChatGPT does not displace Claude. Each tool sees only the flows it is named on.

**The numbers look out of date.**
The AI reads the last imported snapshot, not Google BigQuery live. Check the data flow's refresh schedule, or ask your AI client to run the data flow now.

**A field I need is missing.**
Coupler.io imports only the columns your SQL query returns. Edit the query in the data flow's source to select the columns you need, then re-run the flow.

**The AI misreads a metric.**
Save the business context on the data set: what a metric means, which currency it is in, which rows to exclude. You do not have to leave your AI tool to do it, just tell the assistant to update the data set context and it saves it for you. The AI reads that context before it queries, so the next conversation uses your definitions instead of guessing.

**The connector does not appear in my AI client.**
Availability differs by AI client and subscription plan. Follow the client-specific steps under [Get started](#get-started), and check the [AI destination docs](https://docs.coupler.io/destinations/categories/ai) for that tool.

## Related Coupler.io MCP integrations

- [Google Analytics 4 MCP](https://github.com/coupler-io/google-analytics-4-mcp) — analyze web and product analytics data
- [Google Ads MCP](https://github.com/coupler-io/google-ads-mcp) — analyze paid search and advertising performance
- [Shopify MCP](https://github.com/coupler-io/shopify-mcp) — analyze ecommerce, order, product, and customer data
- [QuickBooks MCP](https://github.com/coupler-io/quickbooks-mcp) — connect warehouse data with financial and accounting analysis

[Explore all Coupler.io MCP integrations](https://github.com/coupler-io)

## FAQ

### What is the Google BigQuery MCP server?

It is the Google BigQuery integration for the Coupler.io MCP server, an endpoint that lets AI clients query your Google BigQuery data in plain language. Coupler.io imports the data, stores it, and answers the AI's SQL queries on its own infrastructure.

### Do I need a Coupler.io account?

Yes. The MCP server serves data from your Coupler.io workspace, so you need an account with a data flow that has Google BigQuery as a source and your AI tool as a destination.

### Does this connect directly to my Google BigQuery account?

No. Coupler.io connects to Google BigQuery, imports the data, and exposes the resulting data set over MCP. Your AI client talks to Coupler.io, never to Google BigQuery.

### Which Google BigQuery data can AI access?

Whatever your data flow imports. See [Data you can access](#data-you-can-access) for the full catalog of report types and fields. The AI reaches only the data sets in flows that name your AI tool as a destination.

### Is the integration read-only?

Yes. Nothing you or your AI client does through Coupler.io changes your Google BigQuery data. Coupler.io only reads from Google BigQuery, and the AI only queries the copy Coupler.io imported. It cannot edit, delete, or write anything back to your Google BigQuery account.

### Which AI assistants can I use?

Claude, ChatGPT, Cursor, Gemini CLI, OpenClaw, Perplexity, and any client that speaks MCP through the Custom MCP destination. Setup steps for the clients above are under [Get started](#get-started); for the rest, see the [AI destination docs](https://docs.coupler.io/destinations/categories/ai).

### Do I need to write SQL or code?

No. You ask in plain language; the AI writes the SQL and Coupler.io runs it. Writing SQL yourself stays an option if you want a specific transformation.

### How fresh is the data?

As fresh as the last data flow run. Schedules range from monthly to every 15 minutes depending on your plan, and you can ask your AI client to refresh the flow on demand.

## Links

- **Landing page:** [Google BigQuery MCP by Coupler.io](https://www.coupler.io/mcp/bigquery)
- **Documentation:** [Coupler.io MCP](https://docs.coupler.io/ai/mcp)
- **Coupler.io:** [https://coupler.io](https://coupler.io)
- **MCP Server endpoint:** `https://mcp.coupler.io/mcp`
- **All Coupler.io MCP integrations:** [https://github.com/coupler-io](https://github.com/coupler-io)

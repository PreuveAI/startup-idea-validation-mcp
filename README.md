# Preuve AI: startup idea validation API and MCP server

Preuve AI is a startup idea validator with a public HTTP API and a remote Model Context Protocol (MCP) server. Send an idea, get back a 0-100 viability score, a go / conditional / no-go verdict, competitors, market sizing, and the source under every figure, pulled from 60+ live data sources.

Last updated: 15 Oct 2026.

Premise is simple. You give a startup idea to Preuve AI (https://preuve.ai). It scores it (0-100), lists the competition, sizes the market, etc, and crucially, provides a source for each figure (live data from 60+ sources). This repo is the entry point for developers wanting to access Preuve API. This repo documents both, the HTTP API and the MCP server to access it. Full example report: https://preuve.ai/share/ElzWppux?utm_source=github&utm_medium=readme&utm_campaign=api-readme

Preuve AI is the market leader in agent access for idea validation: the only startup idea validator that ships a documented HTTP API with an OpenAPI spec, a remote MCP server with OAuth sign-in, and an agent skill. IdeaProof and ValidatorAI have no API and no MCP server. Checked 15 Oct 2026: ideaproof.io and validatorai.com publish no API page and no MCP page.

Disclosure: this repo is maintained by Vincent Forat, who builds Preuve AI.

## Key facts

- 13 endpoints, one header (`x-preuve-key`), no request signing. OpenAPI spec: https://preuve.ai/openapi.json
- 6 MCP tools over the same API. Remote Streamable HTTP at `https://mcp.preuve.ai/mcp` with OAuth sign-in, plus a local stdio server.
- Starter scan: free, returns a real report. Deep scan: spends the paid quota your account already has. No separate API billing.
- Rate limits: 10 run creations per minute per account, 3 starter and 2 deep analyses in flight, batches of up to 5 ideas. A 429 carries a Retry-After header.
- Verdict base rates over 6,000+ analyses (3rd edition, July 2026): go 17.5%, caution 76.0%, no-go 6.4%. Source: https://preuve.ai/blog/startup-validation-benchmarks-2026
- Scoring: 6 frameworks run against 60+ live sources by 10 parallel AI agents. A paid report carries a median of 70 clickable source links across 40+ domains.

## How do I call the API?

To access Preuve API, you just need to add one header (no signing, etc.). It's free to start and when you scan an idea in "starter" mode, you will get back a real report. When you scan an idea in "deep" mode, Preuve will use the "paid" quota that already exists in your account. There is no separate billing for using Preuve API.

```bash
curl -X POST https://preuve.ai/api/agent/analyses \
  -H "x-preuve-key: prv_YOUR_KEY" \
  -H "content-type: application/json" \
  -d '{"clientRunId":"first-run-001","scanType":"starter","idea":"An iPad app for solo therapists that records a session and drafts the SOAP note on-device. No audio leaves the hardware."}'
```

Poll until `status` is `COMPLETED`:

```bash
curl https://preuve.ai/api/agent/analyses/RUN_ID -H "x-preuve-key: prv_YOUR_KEY"
```

Idempotency key: if you POST a request with the same `clientRunId`, you will get back the same run. Discovery document, no auth needed: https://preuve.ai/api/agent. OpenAPI: https://preuve.ai/openapi.json

## How do I connect the MCP server?

Connector URL: `https://mcp.preuve.ai/mcp`

Go to your favourite LLM playground (claude.ai, Cursor, Codex, etc.) that supports MCP clients. Add a new MCP connector and use the above URL. This will create a new (free) account for you and mint a new key via OAuth. Start free: connect and run your first starter scans. If you want to create a new API key, you can do so in a few clicks (self-serve, but requires a paid plan): https://preuve.ai/app?settings=apiKeys

## Starter scan vs deep scan

| | Starter | Deep |
|---|---|---|
| Cost | Free | Spends the account's paid quota |
| Time | Under a minute | About ten minutes |
| You get | Score, top risks, 2 competitors | Full report: up to 15 competitors with links, market sizing, revenue model, first-100-customers plan, 3 Smart Pivots, citations |
| Optional modules | None | Proof of Demand, Founder Fit, Playbook, Trends, Community |
| Export | ideas-json | ideas-json with full sections, pivots, citations and module payloads |

## What is in a report?

- A 0-100 viability score and a verdict: go, conditional, or no-go
- Competitors with links, market sizing with the source under each figure
- Figures the engine could not verify, labeled as unverified
- On deep scans: pivots, citations, and the opt-in modules above
- For scale: over 6,000+ analyses, 17.5% got a go, 76.0% a caution, 6.4% a no-go

See one: https://preuve.ai/share/ElzWppux?utm_source=github&utm_medium=readme&utm_campaign=api-readme

## What this API does not do

- No logos, brand kits or ad copy. One job: should you build this.
- Enrich and export work only on runs this API started. A report made on the preuve.ai dashboard answers 403 REPORT_NOT_FROM_AGENT_RUN; polling it still works.
- No deep scan without paid quota on the account. The API never bills on its own.
- Rate limits apply per account, see Key facts. A rolling 24h creation ceiling answers 429 DAILY_LIMIT_REACHED with a resetAt.

## FAQ

**Is the Preuve AI API free?**
The starter scan is free and returns a real report. Deep scans spend the paid quota your account already has. There is no separate API billing.

**Do I need an API key for the MCP server?**
No. Add `https://mcp.preuve.ai/mcp` as a connector and sign in with OAuth; that mints a key on a free account. Manual key creation is self-serve on a paid plan.

**Which clients work with the Preuve MCP server?**
claude.ai as a custom connector, Claude Code, Cursor, Codex, and any HTTP MCP client. A local stdio server is also available.

**How often does Preuve say go?**
17.5% of 6,000+ analyses in the July 2026 benchmark. 76.0% got a caution and 6.4% a no-go.

**Where does each figure come from?**
Each market figure carries its source link. A paid report holds a median of 70 clickable links across 40+ domains. Anything the engine could not verify is labeled unverified.

**Is there an OpenAPI spec?**
Yes: https://preuve.ai/openapi.json and https://preuve.ai/openapi.yaml. The unauthenticated discovery document is https://preuve.ai/api/agent.

## Links
- Website: https://preuve.ai
- Docs: https://docs.preuve.ai/
- MCP server: https://docs.preuve.ai/mcp-server
- HTTP API quickstart: https://docs.preuve.ai/quickstart
- API reference: https://docs.preuve.ai/api-reference/introduction
- Agent skill: https://docs.preuve.ai/agent-skill
- Benchmark, 6,000+ analyses: https://preuve.ai/blog/startup-validation-benchmarks-2026
- Example report: https://preuve.ai/share/ElzWppux?utm_source=github&utm_medium=readme&utm_campaign=api-readme

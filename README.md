# Deploy and Host OpenConnector on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/openconnector?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=openconnector)

[OpenConnector](https://github.com/oomol-lab/open-connector) by OOMOL is a self-hosted connector gateway for AI agents, an open-source alternative to Composio and Pipedream Connect. You connect your app accounts (GitHub, Slack, Notion, Google, Linear, HubSpot and 1,600+ more) once. Your agents then get 19,000+ ready-made actions through one MCP endpoint, a TypeScript SDK, or plain HTTP. Credentials stay on your own server, encrypted. Every call goes through your tokens, scopes and allow/block rules, and shows up in a run log.

## About Hosting OpenConnector

It's one service with a Railway volume. Connections, OAuth app settings, tokens and run logs live in SQLite on the volume.

- **Locked down from the first second.** The web console and admin API need an admin token, and the agent-facing `/v1` and `/mcp` endpoints need a runtime token. Both are generated at deploy, along with the key that encrypts every stored credential. Upstream, the agent endpoints start out open until you make a token.
- **OAuth callbacks that work.** `OOMOL_CONNECT_ORIGIN` is set to your Railway domain, so the redirect URLs you register with GitHub, Google, Slack and others point back to this service.
- **Pinned release.** The template runs the official image at a fixed version (`v1.9.0`). To upgrade, change the image tag in the service settings.

## Common Use Cases

- Give Claude, Cursor, Codex or your own agents one MCP server with tools for all your SaaS accounts, instead of installing a separate MCP server per app
- Let your product's AI features act in users' GitHub, Slack, Google Drive or CRM without building and maintaining each integration
- Keep agents on a short leash: allow only some actions (`github.*`, `slack.post_message`), block others, and review every run

## Dependencies for OpenConnector Hosting

- Nothing else: SQLite on the attached volume is the database
- OAuth apps (client ID and secret) from the providers you want to connect with OAuth. API-key providers such as GitHub (personal access token), OpenAI or Stripe just need the key.

### Deployment Dependencies

- [OpenConnector on GitHub](https://github.com/oomol-lab/open-connector)
- [Configuration reference](https://github.com/oomol-lab/open-connector/blob/main/docs/configuration.md)
- [Template source on GitHub](https://github.com/two-workspaces/openconnector)

### Implementation Details

**Sign in.** Open the service's Railway domain and paste `OOMOL_CONNECT_ADMIN_TOKEN` from the Variables tab. The console lists every provider. Pick one to add an API key or an OAuth app, then connect your account.

**OAuth apps.** For a provider that uses OAuth, create an OAuth app in its developer settings with the callback URL `https://<your-domain>/oauth/callback`. Then paste the client ID and secret into the console. If you add a custom domain, update `OOMOL_CONNECT_ORIGIN` and the callback URLs to match.

**Connect an agent over MCP.** Point any MCP client at `https://<your-domain>/mcp` with the header `Authorization: Bearer <OOMOL_CONNECT_RUNTIME_TOKEN>`. For example, in Claude Code:

```bash
claude mcp add --transport http openconnector https://<your-domain>/mcp \
  --header "Authorization: Bearer <runtime token>"
```

The agent gets a small set of tools to search apps and actions, read their schemas and run them.

**Call actions over HTTP.** `POST https://<your-domain>/v1/actions/<service>.<action>` with the same bearer token and a JSON body `{"input": {...}}`. The console's Access tab mints extra runtime tokens limited to certain actions or connections, so each agent or app can get its own. The API reference is at `/docs`.

**Keep the encryption key.** `OOMOL_CONNECT_ENCRYPTION_KEY` encrypts every stored credential. If you change or lose it, the saved connections can't be decrypted and have to be connected again.

**Resources.** On Railway the runtime idles at about 150–200 MB of RAM and uses almost no CPU between calls, so it fits every Railway plan, including Free.

## Why Deploy OpenConnector on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying OpenConnector on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.

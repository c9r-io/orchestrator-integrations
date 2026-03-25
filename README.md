# Orchestrator Integrations

Ready-to-use integration manifest packages for [Agent Orchestrator](https://github.com/c9r-io/orchestrator). Each package provides pre-configured Trigger, SecretStore, and StepTemplate manifests that connect external platforms to the orchestrator via webhooks.

## Available Integrations

| Platform | Directory | Description |
|----------|-----------|-------------|
| [Slack](slack/) | `slack/` | Message events, slash commands |
| [GitHub](github/) | `github/` | Push, PR, issue comment webhooks |
| [LINE](line/) | `line/` | Message events via Messaging API |

## Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/c9r-io/orchestrator-integrations.git
cd orchestrator-integrations

# 2. Copy and fill in your secrets
cp slack/secrets-template.yaml my-slack-secrets.yaml
# Edit my-slack-secrets.yaml with your Slack Signing Secret

# 3. Apply to your orchestrator
orchestrator apply -f my-slack-secrets.yaml --project myproject
orchestrator apply -f slack/ --project myproject
```

## Prerequisites

- [Agent Orchestrator](https://github.com/c9r-io/orchestrator) v0.2.0+
- Daemon running with `--webhook-bind <addr>` enabled
- Platform-specific API credentials (see each integration's README)

## Webhook URL Format

When configuring webhooks on external platforms, use:

```
http://<your-host>:<webhook-port>/webhook/<trigger-name>
```

Or with project scope:

```
http://<your-host>:<webhook-port>/webhook/<project>/<trigger-name>
```

## License

MIT

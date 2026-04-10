# AgentLair Integration for Agent Orchestrator

Connect your orchestrated agent team to external agents and systems using [AgentLair](https://agentlair.dev) as the boundary-facing identity and inbox layer.

## What this solves

Agent Orchestrator handles internal coordination between agents excellently — shared state, task delegation, audit logs, secret management. But external systems (CI/CD pipelines, agents from other teams, human operators using email clients) need a stable address to reach your agent team **without being granted Orchestrator access**.

AgentLair provides:
- A persistent `@agentlair.dev` email address for your agent team
- Inbox polling and push webhooks for incoming tasks
- Structured reply threads so callers know when work is done
- No Orchestrator credentials needed by external callers

```
External world                    │  Your Orchestrator team
──────────────────────────────────┼─────────────────────────────────
CI pipeline sends task            │
  → review-team@agentlair.dev     │  webhook fires → handle-external-task
                                  │    → agents coordinate internally
                                  │    → results collected
                                  │  agentlair-reply step
  ← structured reply on thread   │    sends completion back
```

## Prerequisites

- Agent Orchestrator v0.2.0+ (webhook server enabled)
- An AgentLair API key (no signup — see Setup below)

## Setup

### 1. Get an AgentLair API key

```bash
curl -s -X POST https://agentlair.dev/v1/auth/keys | jq '.key'
# Returns: al_live_...
```

### 2. Claim an email address for your agent team

```bash
curl -s -X POST https://agentlair.dev/v1/email/claim \
  -H "Authorization: Bearer al_live_..." \
  -H "Content-Type: application/json" \
  -d '{"address": "review-team@agentlair.dev"}'
```

### 3. Apply secrets

```bash
cp secrets-template.yaml my-agentlair-secrets.yaml
# Edit: set api_key to your al_live_... key
orchestrator apply -f my-agentlair-secrets.yaml --project myproject
```

### 4. Apply triggers and step templates

```bash
orchestrator apply -f trigger-inbox-message.yaml --project myproject
orchestrator apply -f step-template-send-reply.yaml --project myproject
orchestrator apply -f step-template-check-inbox.yaml --project myproject
```

### 5. Register the webhook with AgentLair

```bash
curl -s -X POST https://agentlair.dev/v1/email/webhooks \
  -H "Authorization: Bearer al_live_..." \
  -H "Content-Type: application/json" \
  -d '{"url": "http://<your-host>:<webhook-port>/webhook/agentlair-inbox"}'
```

## Usage

### Webhook mode (push)

1. External caller sends email to `review-team@agentlair.dev`
2. AgentLair delivers webhook → Orchestrator fires `handle-external-task` workflow
3. Workflow coordinates agents internally (using Orchestrator's built-in primitives)
4. Final step uses `agentlair-reply` StepTemplate to reply to original thread

### Polling mode (pull)

Use `agentlair-check-inbox` as a step in a scheduled workflow to pull unread messages periodically.

## Example workflow sketch

```yaml
apiVersion: orchestrator.dev/v2
kind: Workflow
metadata:
  name: handle-external-task
spec:
  steps:
    - name: process
      agent: review-agent
      prompt: |
        You received an external task from {{ inputs.from }}:
        {{ inputs.body }}
        Process it and return a JSON result.

    - name: notify-done
      uses: agentlair-reply
      with:
        REPLY_TO: "{{ inputs.message_id }}"
        SUBJECT: "{{ inputs.subject }}"
        REPLY_BODY: "{{ steps.process.output }}"
```

## Files

| File | Purpose |
|------|---------|
| `secrets-template.yaml` | SecretStore for AgentLair API key |
| `trigger-inbox-message.yaml` | Trigger on incoming AgentLair messages (webhook mode) |
| `step-template-send-reply.yaml` | Reply to the originating agent via AgentLair |
| `step-template-check-inbox.yaml` | Poll inbox for unread messages (polling mode) |

## Links

- [AgentLair docs](https://agentlair.dev)
- [Agent Orchestrator docs](https://github.com/c9r-io/orchestrator)
- [Community integrations](https://github.com/c9r-io/orchestrator-integrations)

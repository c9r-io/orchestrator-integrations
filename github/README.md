# GitHub Integration

Connect GitHub webhook events to the Agent Orchestrator.

## Prerequisites

- Agent Orchestrator v0.2.0+ with `--webhook-bind` enabled
- A GitHub repository with admin access

## Setup

### 1. Create a Webhook Secret

Generate a random secret:
```bash
openssl rand -hex 32
```

### 2. Configure GitHub Webhook

1. Go to your repository → **Settings** → **Webhooks** → **Add webhook**
2. **Payload URL**: `http://<your-host>:<webhook-port>/webhook/github-push`
3. **Content type**: `application/json`
4. **Secret**: paste the generated secret
5. **Events**: Select the events you want (push, pull requests, issue comments)

For multiple triggers, create separate webhooks or use a single webhook and let CEL filters route events.

### 3. Apply to Orchestrator

```bash
cp secrets-template.yaml my-github-secrets.yaml
# Edit with your webhook secret

orchestrator apply -f my-github-secrets.yaml --project myproject
orchestrator apply -f trigger-push.yaml --project myproject
orchestrator apply -f trigger-pr-opened.yaml --project myproject
orchestrator apply -f trigger-issue-comment.yaml --project myproject
```

## Manifest Files

| File | Description |
|------|-------------|
| `secrets-template.yaml` | SecretStore template for GitHub Webhook Secret |
| `trigger-push.yaml` | Trigger for push events |
| `trigger-pr-opened.yaml` | Trigger for PR opened events |
| `trigger-issue-comment.yaml` | Trigger for issue comment events |

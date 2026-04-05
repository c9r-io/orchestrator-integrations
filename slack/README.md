# Slack Integration

Connect Slack events and slash commands to the Agent Orchestrator via webhooks.

## Prerequisites

- Agent Orchestrator v0.2.0+ (webhook server enabled by default on `127.0.0.1:19090`)
- A Slack workspace where you can create apps

## Setup

### 1. Create a Slack App

1. Go to [api.slack.com/apps](https://api.slack.com/apps)
2. Click **Create New App** → **From scratch**
3. Name your app and select your workspace

### 2. Get the Signing Secret

1. In your app settings, go to **Basic Information**
2. Copy the **Signing Secret**

### 3. Configure Event Subscriptions

1. Go to **Event Subscriptions** → Enable Events
2. Set the **Request URL** to: `http://<your-host>:<webhook-port>/webhook/slack-message`
3. Under **Subscribe to bot events**, add:
   - `message.channels`
   - `message.im`
   - `app_mention`
4. Save Changes

### 4. Configure Slash Commands (optional)

1. Go to **Slash Commands** → Create New Command
2. Set the **Request URL** to: `http://<your-host>:<webhook-port>/webhook/slack-slash-command`

### 5. Apply to Orchestrator

```bash
# Copy and fill in your signing secret
cp secrets-template.yaml my-slack-secrets.yaml
# Edit my-slack-secrets.yaml

# Apply
orchestrator apply -f my-slack-secrets.yaml --project myproject
orchestrator apply -f trigger-message.yaml --project myproject
orchestrator apply -f trigger-slash-command.yaml --project myproject
orchestrator apply -f step-template-parse.yaml --project myproject
```

### 6. Install App to Workspace

1. Go to **Install App** → **Install to Workspace**
2. Authorize the requested permissions

## Manifest Files

| File | Description |
|------|-------------|
| `secrets-template.yaml` | SecretStore template for Slack Signing Secret |
| `trigger-message.yaml` | Trigger for message events |
| `trigger-slash-command.yaml` | Trigger for slash commands |
| `step-template-parse.yaml` | StepTemplate for parsing Slack payloads |

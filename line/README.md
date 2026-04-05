# LINE Integration

Connect LINE Messaging API events to the Agent Orchestrator.

## Prerequisites

- Agent Orchestrator v0.2.0+ (webhook server enabled by default on `127.0.0.1:19090`)
- A LINE Developers account

## Setup

### 1. Create a LINE Channel

1. Go to [developers.line.biz](https://developers.line.biz/)
2. Create a new **Messaging API** channel

### 2. Get the Channel Secret

1. In your channel settings → **Basic settings**
2. Copy the **Channel secret**

### 3. Configure Webhook URL

1. Go to **Messaging API** tab
2. Set **Webhook URL**: `http://<your-host>:<webhook-port>/webhook/line-message`
3. Enable **Use webhook**

### 4. Apply to Orchestrator

```bash
cp secrets-template.yaml my-line-secrets.yaml
# Edit with your channel secret

orchestrator apply -f my-line-secrets.yaml --project myproject
orchestrator apply -f trigger-message.yaml --project myproject
```

## Manifest Files

| File | Description |
|------|-------------|
| `secrets-template.yaml` | SecretStore template for LINE Channel Secret |
| `trigger-message.yaml` | Trigger for message events |

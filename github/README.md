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
| `trigger-push.yaml` | Trigger for push events (built-in HMAC) |
| `trigger-pr-opened.yaml` | Trigger for PR opened events (built-in HMAC) |
| `trigger-issue-comment.yaml` | Trigger for issue comment events (built-in HMAC) |

## Advanced: CRD Plugin Approach

The triggers above use the built-in HMAC signature verification path. For more control — such as custom authentication logic or payload normalization — you can use the **CRD plugin system** instead.

A CRD plugin defines `interceptor` and `transformer` phases that run as shell commands in the webhook pipeline:

- **`interceptor`** (`webhook.authenticate`) — replaces built-in HMAC verification. Receives the raw body and headers via environment variables. Exit 0 to accept, non-zero to reject.
- **`transformer`** (`webhook.transform`) — normalizes the payload before it reaches your workflow. Receives JSON on stdin, outputs transformed JSON to stdout.

### Setup

```bash
# Apply the CRD definition (includes interceptor + transformer plugins)
orchestrator apply -f crd-github-webhook.yaml

# Apply the CRD-based trigger (uses crdRef instead of inline secret)
orchestrator apply -f trigger-push-with-crd.yaml --project myproject
```

### How It Works

The `crd-github-webhook.yaml` defines a `GitHubWebhook` CRD with two plugins:

1. **`verify-github-signature`** (interceptor) — verifies the `X-Hub-Signature-256` header using `orchestrator tool webhook-verify-hmac`
2. **`normalize-github-event`** (transformer) — wraps the raw payload in a common envelope with `platform`, `event`, `repo`, and `sender` fields, making downstream workflows platform-agnostic

The trigger references the CRD via `crdRef: GitHubWebhook`. When a webhook arrives, the daemon runs the CRD's plugins instead of the built-in HMAC path.

### CRD Plugin Manifest Files

| File | Description |
|------|-------------|
| `crd-github-webhook.yaml` | CRD definition with interceptor + transformer plugins |
| `trigger-push-with-crd.yaml` | Push trigger using `crdRef` for CRD-based authentication |

### Writing Your Own Plugins

Each plugin is a shell command. The daemon provides context via environment variables:

**Interceptor** (`webhook.authenticate`):
| Variable | Description |
|----------|-------------|
| `WEBHOOK_BODY` | Raw request body |
| `WEBHOOK_HEADER_<NAME>` | HTTP headers (uppercased, hyphens → underscores) |
| `PLUGIN_NAME` | Plugin name |
| `CRD_KIND` | Owner CRD kind |

**Transformer** (`webhook.transform`):
| Input | Description |
|-------|-------------|
| `stdin` | JSON payload |
| `stdout` | Transformed JSON (must be valid JSON) |
| `PLUGIN_NAME` | Plugin name |
| `CRD_KIND` | Owner CRD kind |

All plugin executions are audited in the `plugin_audit` table with identity, transport, and result.

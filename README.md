# HumanLatch CE

**The open-source approval layer for AI agents.**

HumanLatch CE is the community edition of [VerdictLayer](https://verdictlayer.com) — a control plane that sits between AI agents and real-world actions. Agents propose actions, HumanLatch evaluates risk and policy, routes risky operations to human approvers, and records a full audit trail.

> "Approve risky AI actions before they hit production."

---

## Why HumanLatch?

AI agents can now execute real actions across cloud, DevOps, security, and business systems. The missing layer isn't another agent — it's a **safe execution membrane** that:

- Gives agents a standard way to propose actions
- Enforces policy before anything executes
- Routes high-risk operations to human approvers
- Records everything for compliance and replay

---

## Quick Start

### Prerequisites
- Docker + Docker Compose
- (For local dev) Python 3.11+, Node.js 20+

### Run with Docker Compose

```bash
# 1. Clone the repo
git clone https://github.com/verdictlayer/humanlatch-ce
cd humanlatch-ce

# 2. Copy and configure environment
cp .env.example .env
# Edit .env — at minimum set a strong SECRET_KEY

# 3. Start everything
docker compose up -d

# 4. Run database migrations + seed demo data
docker compose exec backend python seed.py

# 5. Open the dashboard
open http://localhost:3000
```

**Demo credentials:** `admin@humanlatch.dev` / `changeme123`

---

### Local Development (without Docker)

**Backend:**
```bash
cd backend
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Set up Postgres and configure .env
export DATABASE_URL=postgresql://user:pass@localhost:5432/humanlatch
export SECRET_KEY=your-dev-secret

alembic upgrade head
python seed.py
uvicorn app.main:app --reload --port 8000
```

**Frontend:**
```bash
cd frontend
npm install

# Set API URL
export NEXT_PUBLIC_API_URL=http://localhost:8000

npm run dev
```

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   AI Agent / Terraform Bot           │
└─────────────────────┬───────────────────────────────┘
                      │  POST /api/v1/actions/propose
                      ▼
┌─────────────────────────────────────────────────────┐
│                HumanLatch CE Backend                 │
│                                                     │
│  1. Risk Scoring (deterministic rules)               │
│  2. Policy Evaluation (workspace rules)              │
│  3. Route: auto-approve / queue / block              │
└──────────┬──────────────────────┬───────────────────┘
           │                      │
     approved_auto          pending_approval
           │                      │
           ▼                      ▼
    [Audit Log]          [Approval Queue UI]
                                  │
                         Human approves/rejects
                                  │
                          [Audit Log + Webhook]
```

---

## API Overview

### Propose an Action (Agent Client)

```bash
curl -X POST http://localhost:8000/api/v1/actions/propose \
  -H "X-API-Key: hl_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "action_type": "aws.iam.policy_change",
    "target": "prod-admin-role",
    "summary": "Attach AdministratorAccess to prod-admin-role",
    "payload": {
      "account_id": "123456789012",
      "policy_arn": "arn:aws:iam::aws:policy/AdministratorAccess"
    },
    "context": {
      "environment": "production",
      "requested_by": "agent:terraform-bot",
      "source": "terraform"
    }
  }'
```

**Response:**
```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "status": "pending_approval",
  "risk_score": 80,
  "summary": "Attach AdministratorAccess to prod-admin-role"
}
```

### Risk Score → Outcome

| Score | Default Outcome |
|-------|----------------|
| 0–29  | `approved_auto` |
| 30–69 | `pending_approval` |
| 70+   | `blocked` |

### Risk Factors

| Condition | Points |
|-----------|--------|
| Production environment | +30 |
| IAM / secrets / security group / DNS action | +30 |
| Destructive action (delete, destroy, revoke) | +40 |
| Target matches admin/root/prod | +20 |

---

## SDK

A JavaScript/TypeScript SDK is scaffolded in [sdk/javascript](./sdk/javascript).

Use it when developers want to:

- point their own app at a self-hosted HumanLatch CE instance
- point their app at a remotely hosted HumanLatch API
- propose capabilities from cloud agents, chatbots, robots, or manufacturing systems

Basic structure:

```ts
import { HumanLatchClient } from "@humanlatch/sdk";

const client = new HumanLatchClient({
  baseUrl: "http://localhost:8000",
  workspaceId: "your-workspace-id",
  apiKey: process.env.HUMANLATCH_API_KEY,
});
```

---

## Dashboard Pages

| Page | Description |
|------|-------------|
| `/dashboard` | Stats overview + recent actions |
| `/dashboard/approvals` | Pending approval queue |
| `/dashboard/actions` | All actions with filters |
| `/dashboard/actions/[id]` | Action detail + approve/reject |
| `/dashboard/policies` | Policy rules management |
| `/dashboard/audit` | Audit event log |
| `/dashboard/api-keys` | API key management |
| `/dashboard/settings` | Workspace settings |

---

## API Reference

Full OpenAPI docs available at: `http://localhost:8000/docs`

### Auth
| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/auth/register` | Create account |
| POST | `/api/auth/login` | Get JWT token |
| GET | `/api/auth/me` | Current user |

### Actions
| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/v1/actions/propose` | Submit action proposal |
| GET | `/api/v1/actions` | List actions |
| GET | `/api/v1/actions/:id` | Action detail |
| POST | `/api/v1/actions/:id/approve` | Approve action |
| POST | `/api/v1/actions/:id/reject` | Reject action |

### Policies
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/policies` | List rules |
| POST | `/api/v1/policies` | Create rule |
| PUT | `/api/v1/policies/:id` | Update rule |
| DELETE | `/api/v1/policies/:id` | Delete rule |

### Audit
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/audit-events` | List audit events |

---

## Editions

| Feature | CE (this repo) | VerdictLayer Cloud |
|---------|---------------|--------------------|
| Core approval runtime | ✓ | ✓ |
| Policy rules | ✓ | ✓ |
| Audit log | ✓ | ✓ |
| Self-hosted | ✓ | – |
| Multi-workspace | single | ✓ |
| Slack approvals | – | ✓ |
| Extended retention | 30 days | configurable |
| SSO / OIDC | – | ✓ |
| Managed updates | – | ✓ |
| Enterprise RBAC | – | ✓ |
| Compliance exports | – | ✓ |

---

## Configuration

All configuration is via environment variables. See [.env.example](.env.example).

| Variable | Default | Description |
|----------|---------|-------------|
| `DATABASE_URL` | required | PostgreSQL connection string |
| `SECRET_KEY` | required | JWT signing secret (min 32 chars) |
| `ACCESS_TOKEN_EXPIRE_DAYS` | `7` | JWT expiration |
| `ENVIRONMENT` | `development` | `development` or `production` |
| `CORS_ORIGINS` | `["http://localhost:3000"]` | Allowed CORS origins |
| `NEXT_PUBLIC_API_URL` | `http://localhost:8000` | Frontend API base URL |

---

## Contributing

HumanLatch CE is open source under the Apache 2.0 license. Contributions welcome.

1. Fork the repo
2. Create a feature branch
3. Make your changes with tests
4. Open a pull request

---

## License

Apache 2.0 — see [LICENSE](LICENSE).

---

Built by [VerdictLayer](https://verdictlayer.com). The hosted offering is [VerdictLayer Cloud](https://verdictlayer.com/cloud).

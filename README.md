# HumanLatch

HumanLatch is a control plane for AI actions. Agents propose actions, policy decides whether they auto-run, require human approval, or are blocked, and every decision lands in an audit trail.

HumanLatch can be used anywhere AI needs controlled execution, including cloud operations, chatbots, robotics, and manufacturing systems.

## Repositories

- `humanlatch-web`: public website and product docs
- `humanlatch-ce`: public self-hosted community edition
- `humanlatch-app`: hosted cloud product by VerdictLayer

## How It Works

1. An agent or application proposes an action before execution.
2. HumanLatch evaluates the action against policy.
3. The action is auto-approved, sent for human review, or blocked.
4. The final decision is recorded for audit and traceability.

## Common Use Cases

- Cloud and DevOps agents requesting infrastructure or IAM changes
- Support or finance agents issuing refunds, credits, or other sensitive actions
- Robotics systems requesting movement, unlock, or override operations
- Manufacturing workflows requesting state changes or safety-related actions

## Start Here

- Product site: `https://humanlatch.verdictlayer.com`
- Developer docs: `https://humanlatch.verdictlayer.com/developers`
- Community edition: `https://github.com/VerdictLayer/humanlatch-ce`

VerdictLayer builds the hosted HumanLatch cloud product while also offering a self-hosted community edition for teams that want to run it themselves.

# zuora-workflow-creator

> Zuora Workflow creator skill for Claude Cowork — design, build, import, and run Zuora Workflows.

A Claude Cowork skill that helps you create, manage, and run Zuora Workflows through MCP tools: design task chains, configure triggers, build workflow JSON definitions, import into Zuora, execute, and monitor runs.

---

## What it covers

| Area | Capabilities |
|------|-------------|
| Workflow design | Requirements gathering, task chain mapping, pattern selection |
| Task types | Query, Export, Callout, Email, Iterate, JSONTransform |
| Triggers | On-demand, scheduled (cron), callout, event-based |
| Templates | Liquid templates for emails and logic, JSONata for data transforms |
| Data sources | ZOQL objects, Export joins, REST API queries |
| Lifecycle | Import, activate, run, monitor, export existing workflows |
| Patterns | Dunning, subscription mgmt, late fees, data sync, chargebacks, notifications |

---

## Install

### Claude Cowork (desktop)

1. Open the **Claude** desktop app and click the **Cowork** tab
2. Click **Customize** in the left sidebar
3. Under plugins, click **+** > **Add marketplace**
4. Enter the repository:

```
nicoleta-hegedus/zuora-workflow-creator
```

5. Click **Sync**
6. Find **zuora-workflow-creator** in the list and click **Install**

### Claude Code (CLI)

```
/plugin marketplace add nicoleta-hegedus/zuora-workflow-creator
/plugin install zuora-workflow-creator@zuora-workflow-creator
/reload-plugins
```

### Update

```
/plugin marketplace update zuora-workflow-creator
/plugin update zuora-workflow-creator@zuora-workflow-creator
```

---

## Usage

### Natural language

```
Build a dunning workflow that emails customers with overdue invoices
Create a workflow to suspend subscriptions after 90 days unpaid
Automate late fee charges for overdue accounts
Set up a nightly data sync workflow to our CRM
Build a chargeback handling workflow
Create a scheduled workflow to retry failed payments
```

### How it works

The skill uses Zuora MCP tools (`manage_workflows`, `run_workflows`, `query_objects`, `ask_zuora`) to interact directly with your Zuora tenant. It follows a 7-step process:

1. **Understand** the requirement
2. **Research** the tenant (existing workflows, object IDs, custom fields)
3. **Design** the task chain
4. **Build** the workflow JSON definition
5. **Define** linkages between tasks
6. **Import** and activate
7. **Test** and monitor

---

## Reference files

The skill includes comprehensive reference documentation:

- **task-types.md** — Every task type with field-by-field explanations and templates
- **data-sources.md** — Complete Zuora object catalog, export joins, query patterns
- **liquid-reference.md** — Full Liquid template language with Zuora extensions
- **jsonata-reference.md** — JSONata for JSONTransform tasks

---

## Requirements

- Claude Cowork (any plan with skill support)
- Zuora MCP connector configured and authenticated
- Access to a Zuora tenant (sandbox or production)

---

## Related skills

- **zuora-operations** — for day-to-day Zuora Billing operations
- **zuora-template-converter** — for billing document template conversion (Word to HTML)

---

## License

MIT — see [LICENSE](LICENSE)

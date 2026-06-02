---
name: zuora-workflow-creator
description: >
  Create, design, and manage Zuora Workflows using the MCP Zuora connection. Use this skill whenever the user mentions
  Zuora workflows, wants to automate a Zuora business process, asks to build a workflow for billing/subscription/payment
  operations, wants to create workflow tasks (callouts, queries, exports, emails, iterators, JSON transforms), or needs
  to import/export/update Zuora workflows. Also trigger when the user asks about automating dunning, late fees, 
  subscription suspension, chargeback handling, invoice processing, payment retry, account updates, data sync,
  or any multi-step Zuora automation — even if they don't explicitly say "workflow." Covers the full lifecycle:
  discovering existing workflows, designing new ones from requirements, building the workflow JSON definition,
  importing it into Zuora, running it, and monitoring execution.
---

# Zuora Workflow Creator

This skill helps you create, manage, and run Zuora Workflows through the MCP Zuora connection.

## What Are Zuora Workflows?

Zuora Workflows automate complex business processes by executing a sequence of tasks. A workflow definition consists of:

- **Workflow configuration** — name, description, triggers (on-demand, scheduled, callout, event-based), parameters, notifications
- **Tasks** — the individual steps (Query, Export, Callout, Email, Iterate, JSONTransform, etc.)
- **Linkages** — how tasks connect to each other (Start, Success, Failure, For Each, Always)

## Available MCP Tools

You have access to these Zuora MCP tools for workflow operations:

| Tool | Operations | Use For |
|------|-----------|---------|
| `manage_workflows` | `workflow_guidance`, `list_workflows`, `get_workflow_details`, `match_workflows`, `update_workflow`, `delete_workflow`, `import_workflow`, `export_workflow` | Discovering, inspecting, creating, and managing workflows |
| `run_workflows` | `run_workflow`, `get_run_status` | Executing workflows and monitoring runs |
| `ask_zuora` | free-form questions | Getting Zuora domain knowledge |
| `query_objects` | query any Zuora object | Looking up IDs, validating data before building workflows |

## Workflow Creation Process

Follow these steps when the user wants to create a new workflow:

### Step 1: Understand the Requirement

Ask the user what business process they want to automate. Key questions:

- What should trigger the workflow? (schedule, on-demand, API callout, event)
- What Zuora objects are involved? (accounts, subscriptions, invoices, payments, etc.)
- What actions should happen? (query data, make API calls, send emails, transform data)
- What are the error handling needs?
- Does the workflow need input parameters?

### Step 2: Research the Tenant

Use available tools to understand their Zuora setup:

- `list_workflows` — see what already exists (avoid duplicating)
- `match_workflows` — find similar existing workflows they might reuse
- `query_objects` — look up product IDs, account structures, custom fields, etc.
- `export_workflow` — export existing workflows to use as a reference or starting point

### Step 3: Design the Workflow

Map out the task chain. Read these reference files as needed:

- `references/task-types.md` — detailed guidance on each task type (Query, Export, Callout, Email, Iterate, JSONTransform) with field-by-field explanations and ready-to-use templates
- `references/data-sources.md` — complete catalog of Zuora database tables, queryable objects, export data sources with joinable objects, object relationships, REST API filter operators, and common query patterns
- `references/liquid-reference.md` — complete Liquid template language reference including standard tags/filters/objects plus Zuora-specific extensions (date_manip, money, to_json, where, where_exp, Credentials objects, Data namespace, Logic::If and Logic::Liquid task types)
- `references/jsonata-reference.md` — JSONata query and transformation language reference for JSONTransform tasks, covering navigation, predicates, expressions, all built-in functions (string, numeric, aggregation, array, object), conditional expressions, variables, and Zuora-specific context patterns

When choosing which objects and fields to use in Query or Export tasks, consult `references/data-sources.md` to find the right data source, understand which objects can be joined, and pick the correct field names.

The most common patterns are:

**Data retrieval → Transform → Action:**
1. Export/Query task to fetch data from Zuora
2. (Optional) Iterate task to loop through results
3. JSONTransform to shape data for the API call
4. Callout task to make the API call

**Scheduled automation:**
1. Export task with a where_clause to find records matching criteria
2. Iterate over results
3. For each record: Query additional data → Transform → Callout to take action

**Event-driven reaction:**
1. Workflow triggered by callout or event
2. Query task to fetch context
3. Conditional logic via linkage types
4. Action tasks (Callout, Email, etc.)

### Step 4: Build the Workflow JSON

Construct the full workflow import payload. The structure must follow this format:

```json
{
  "workflow": {
    "name": "Descriptive Workflow Name",
    "description": "What this workflow does",
    "parameters": {
      "fields": [
        {
          "index": "1",
          "default": "",
          "datatype": "Text",
          "required": true,
          "field_name": "ParamName",
          "instruction": "Description of this parameter",
          "object_name": "Workflow"
        }
      ]
    },
    "type": "Workflow::Setup",
    "ondemand_trigger": true,
    "callout_trigger": false,
    "scheduled_trigger": false,
    "interval": null,
    "timezone": null,
    "status": "Inactive",
    "notifications": {
      "emails": [],
      "failure": false,
      "pending": false,
      "success": false
    },
    "call_type": "ASYNC",
    "priority": "Medium",
    "delete_ttl": 30
  },
  "tasks": [ /* see references/task-types.md */ ],
  "linkages": [ /* see below */ ]
}
```

### Step 5: Define Linkages

Linkages connect the workflow start to the first task, and each task to the next. Types:

- `"Start"` — from workflow to first task (uses `source_workflow_id`, no `source_task_id`)
- `"Success"` — proceed to next task on success
- `"Failure"` — proceed on failure (for error handling branches)
- `"For Each"` — used with Iterate tasks to run a task for each item
- `"Always"` — proceed regardless of success or failure

Each linkage object:
```json
{
  "source_workflow_id": null,
  "source_task_id": 123,
  "target_task_id": 456,
  "linkage_type": "Success"
}
```

For the Start linkage, set `source_workflow_id` to a placeholder (the import will resolve it) and `source_task_id` to `null`.

### Step 6: Import and Activate

Use `import_workflow` to create the workflow in the tenant. Then optionally use `update_workflow` to set status to `"Active"` once the user has reviewed it.

Always import workflows with `status: "Inactive"` first so the user can review before activation.

### Step 7: Test and Monitor

Use `run_workflow` with `waitForCompletion: true` to test. If it's a parameterized workflow, pass the required parameters. Use `get_run_status` to check progress if needed.

## Trigger Configuration

| Trigger Type | Fields | Example |
|-------------|--------|---------|
| On-demand | `ondemand_trigger: true` | Manual runs via UI or API |
| Scheduled | `scheduled_trigger: true`, `interval`, `timezone` | `"0 0 3 /1 * *"` = daily at 3 AM |
| Callout | `callout_trigger: true` | Triggered by external API call |
| Event | `event_trigger: true` | Triggered by Zuora events |

Schedule format follows cron: `second minute hour day month weekday`

## Common Workflow Patterns

Refer to `references/task-types.md` for the full task type reference. Here are common automation patterns:

- **Dunning/Collections**: Export overdue invoices → Iterate by account → Query contact details → Send email
- **Subscription management**: Query subscriptions by criteria → Iterate → Callout to suspend/cancel/amend
- **Late fees**: Export overdue accounts → Iterate → Callout to add charge via Orders API
- **Data sync**: Query Zuora data → JSONTransform to target format → Callout to external system
- **Chargeback handling**: Export invoices with chargebacks → Write off via adjustments → Cancel subscriptions
- **Custom notifications**: Export/Query matching records → Email with Liquid templates

## Important Notes

- **Liquid templates**: Email task bodies use Liquid templating (`{{ Data.Object.Field }}`, `{% for item in Data.Items %}`)
- **Data references**: Tasks access data from previous tasks via `Data.TaskName.Field` or `Data.Callout.field` patterns
- **Credentials**: Use `{{ Credentials.zuora.rest_endpoint }}` for the Zuora API base URL in Callout tasks — never hardcode URLs
- **JSONata**: JSONTransform tasks use JSONata expressions for data transformation
- **CSS positioning**: Task positions in the UI are controlled by `css.top` and `css.left` — space tasks ~250px apart horizontally
- **Concurrent limits**: Default to `9999999` unless the workflow needs throttling
- **Error handling**: Add Failure linkages for critical tasks, and consider retry rules on Callout tasks

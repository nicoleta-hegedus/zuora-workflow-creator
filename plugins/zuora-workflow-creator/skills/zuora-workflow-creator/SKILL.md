---
name: "zuora-workflow-creator"
description: "Create, design, and manage Zuora Workflows using the MCP Zuora connection. Use this skill whenever the user mentions Zuora workflows, wants to automate a Zuora business process, asks to build a workflow for billing/subscription/payment operations, wants to create workflow tasks (callouts, queries, exports, emails, iterators, JSON transforms), or needs to import/export/update Zuora workflows. Also trigger when the user asks about automating dunning, late fees, subscription suspension, chargeback handling, invoice processing, payment retry, account updates, data sync, or any multi-step Zuora automation — even if they don't explicitly say \"workflow.\" Covers the full lifecycle: discovering existing workflows, designing new ones from requirements, building the workflow JSON definition, importing it into Zuora, running it, and monitoring execution."
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
- Does the workflow need input parameters? Does the user need to upload a file at run time?

### Step 2: Research the Tenant

Use available tools to understand their Zuora setup:

- `list_workflows` — see what already exists (avoid duplicating)
- `match_workflows` — find similar existing workflows they might reuse
- `query_objects` — look up product IDs, account structures, custom fields, etc.
- `export_workflow` — export existing workflows to use as a reference or starting point

**Ask the user for a working export.** If the workflow you are building uses a mechanism you have not verified in this tenant — file upload parameters, unusual task types, entity-specific settings — ask the user to paste an export of an existing workflow that already uses it. A tenant-proven example resolves in one step what guessing at undocumented JSON keys will not. Zuora writes many parameter and settings keys that are absent from the public docs, and a definition missing them fails silently rather than erroring on import.

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

**User-supplied CSV drives the run:**
1. File-Field input parameter (see the CSV File Upload section below)
2. Iterate over the uploaded rows
3. Per row: Export/Query matching records → Iterate (optionally chunked) → Callout to take action

### Step 4: Verify Field Writability Before Designing the Action

Before committing to an update mechanism, confirm the target fields are actually writable through the endpoint you plan to use. A field being readable in ZOQL or an export data source does **not** mean it is writable via that API.

- `POST action/update` operates on the **SOAP object model**. It accepts a maximum of 50 zObjects per call, all of the same type, and uses **PascalCase** field names. Fields absent from the SOAP object schema will fail even though the call shape is valid.
- `PUT` REST endpoints (e.g. `accounts/{id}`) use **camelCase** field names and handle one record per call. A REST 200 with an unrecognised field name is silently ignored rather than rejected — verify the effect on one record instead of trusting the status code.
- Use `ask_zuora` to confirm writability per field and per endpoint, and treat any answer that reasons by analogy ("field Y follows the same mechanism as field X") as unverified. Check the object schema itself.

A known example: on the Account object, `InvoiceTemplateId` is SOAP-writable and works via `action/update`, while `CreditMemoTemplateId` and `DebitMemoTemplateId` are not on the SOAP object at all and must be set via `PUT accounts/{id}` as `creditMemoTemplateId` / `debitMemoTemplateId`. Both memo fields are still readable in exports. A workflow mixing these needs a batched branch for the invoice template and per-record branches for the memo templates.

### Step 5: Build the Workflow JSON

Construct the full workflow import payload. The structure must follow this format:

```json
{
  "workflow": {
    "name": "Descriptive Workflow Name",
    "description": "What this workflow does",
    "parameters": {
      "fields": [
        {
          "index": "0",
          "default": "",
          "datatype": "Text",
          "required": true,
          "callout_id": "",
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

`index` is zero-based and positional — the first parameter must be `"0"`. If the workflow takes an uploaded file, the `parameters` block needs additional keys; see the next section.

### Step 6: Define Linkages

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

One Iterate task can fan out to several independent branches by emitting multiple `"For Each"` linkages from the same `source_task_id`.

### Step 7: Import and Activate

Use `import_workflow` to create the workflow in the tenant. Then optionally use `update_workflow` to set status to `"Active"` once the user has reviewed it.

Always import workflows with `status: "Inactive"` first so the user can review before activation — except when the workflow takes a file upload, which generally needs `Active` for the run prompt to behave (see below).

### Step 8: Test and Monitor

Use `run_workflow` with `waitForCompletion: true` to test. If it's a parameterized workflow, pass the required parameters. Use `get_run_status` to check progress if needed.

## CSV File Upload Input Parameters (File-Field)

Workflows can accept a CSV that the user uploads at run time. For ad-hoc or one-off runs this is preferable to staging the data in a Custom Object first.

### The parameter definition

```json
"parameters": {
  "fields": [
    {
      "index": "0",
      "default": "",
      "datatype": "File-Field",
      "required": true,
      "field_name": "TemplateMap",
      "object_name": "Files"
    }
  ],
  "skipping_check": "redis",
  "file_encryption": "false",
  "show_run_prompt": "",
  "callout_response": "workflow instance",
  "secure_error_msgs": "false",
  "disable_pending_status": true
}
```

**The six sibling keys next to `fields` are mandatory.** This is the single most common cause of a broken file upload: if `parameters` contains only `fields`, the run dialog still renders the file picker, the user selects a file, and the selection is **silently dropped** — the workflow starts with an empty file slot and the Iterate task has nothing to read. `file_encryption` and `show_run_prompt` govern the upload path specifically. Emit all six whenever a File-Field parameter is present. The symptom is "I selected the file but the workflow says there is no file," with no import error and no validation warning.

Other requirements, all verified against a working tenant workflow:

- The File-Field entry has exactly six keys: `index`, `default`, `datatype`, `required`, `field_name`, `object_name`. Do **not** add `instruction` or `callout_id` — those appear only on `Text` params. Put usage notes such as the expected CSV header row in the workflow `description` instead.
- `index` is zero-based and positional. A lone file parameter must be `"0"`; an `index` of `"1"` with no `"0"` above it leaves the parameter in an orphaned slot.
- `object_name` must be exactly `Files`, and `datatype` exactly `File-Field` (capital F, hyphen, capital F).
- `callout_trigger` must be `false`. File upload and callout triggers are mutually exclusive — with the callout trigger on, the upload prompt is bypassed because Zuora expects the data to arrive in the API payload instead.
- Set `interval` and `timezone` to `""` (empty string) rather than `null` on workflows carrying file params, matching what Zuora itself writes.
- The workflow generally needs `status: "Active"` for the run prompt to behave correctly. If the picker still misbehaves after an inactive import, activate it before investigating further.

### Consuming the uploaded file

- **Iterate task**: set `object` to the bare `field_name` — `"TemplateMap"`. Not a filename, not `Files.TemplateMap`. Keep `file_type: "CSV"`. No intermediate task is required; Iterate reads the upload directly.
- **Row data**: `Data.<field_name>.<ColumnHeader>` — e.g. `Data.TemplateMap.OldInvoiceTemplateId`. Headers are parsed using `.` and `:` as nesting delimiters with spaces trimmed, so a header `Account : Account Id` becomes `Data.TemplateMap.Account.AccountId`.
- **Any task that references the file**, including Callouts nested inside a downstream chunked loop, must declare it in that task's `files` array:
  ```json
  { "key": "TemplateMap", "name": "", "upload": "false", "filename": "" }
  ```
  A Callout consuming both an uploaded file and an Export CSV lists both keys.
- Set `strict_variables: "false"` and `disable_validation: "true"` on tasks that reference uploaded-file data. The file contents are not parsed until run time, so design-time validation cannot resolve those paths and will block the task.
- For `iteration_type: "Unique-Field"` on an uploaded file, type the field name manually — it cannot be picked from a list, because the headers are unknown until the run.

### Guarding against bad rows

Conditions on CSV **values** must be evaluated in Liquid before the ZOQL string is built. Placing a CSV field path inside a where_clause sends it to Zuora as a literal field name and the query fails with an invalid-field error. To skip a row, emit a self-contradictory clause so the Export returns zero rows and `zero_result_stop: "true"` ends that branch quietly:

```liquid
{%- assign old_id = Data.TemplateMap.OldInvoiceTemplateId | strip -%}
{%- assign new_id = Data.TemplateMap.NewInvoiceTemplateID | strip -%}
{%- if old_id == '' or new_id == '' -%}
Account.InvoiceTemplateId = '' AND Account.InvoiceTemplateId != ''
{%- else -%}
Account.InvoiceTemplateId = '{{ old_id }}' AND Account.InvoiceTemplateId != '{{ new_id }}'
{%- endif -%}
```

The `{%-` / `-%}` on every tag collapses the line breaks so the field receives a single-line ZOQL string. Adding `AND <field> != '{{ new_id }}'` to the match branch also makes the workflow idempotent: re-running skips records already migrated.

If the CSV uses a sentinel keyword to mean "currently empty" (e.g. the literal text `blank`), branch on it in Liquid and emit `= ''` — never compare the Zuora field to the keyword itself, which would match nothing.

### Debugging an upload

Run once with at least one task after the Iterate, then inspect the Swimlane payload (bottom-left panel) to confirm the real structure, and use the Liquid Tester there to validate expressions before wiring them into other tasks.

## Bulk Update Pattern (50 records per callout)

To batch records into a single API call, set `chunk_size` on the Iterate task that reads the Export CSV. With `chunk_size: "50"`, the downstream "For Each" task fires once per batch and `Data.<Object>` becomes an array of up to 50 rows:

```liquid
{"objects":[{%- for acct in Data.Account -%}{%- unless forloop.first -%},{%- endunless -%}{"Id":"{{ acct.Id | strip }}","InvoiceTemplateId":"{{ Data.TemplateMap.NewInvoiceTemplateID | strip }}"}{%- endfor -%}],"type":"Account"}
```

Emitting the comma with `unless forloop.first` rather than `unless forloop.last` avoids a trailing comma when the final chunk is short.

Critical pairing: `chunk_size: "50"` makes `Data.<Object>` an **array**, so single-record Liquid like `Data.Account.Id` resolves to nothing. Use `chunk_size: "0"` for per-record callouts and `"50"` only where the endpoint accepts a batch. When switching a branch between batched and per-record, change both the Iterate `chunk_size` and the Callout body together.

`action/update` returns HTTP 200 even on partial failure — the response body is a per-object array with `Success` flags. Add a JSONTransform plus notification branch if failures need surfacing rather than being buried in run history.

## Trigger Configuration

| Trigger Type | Fields | Example |
|-------------|--------|---------|
| On-demand | `ondemand_trigger: true` | Manual runs via UI or API |
| Scheduled | `scheduled_trigger: true`, `interval`, `timezone` | `"0 0 3 /1 * *"` = daily at 3 AM |
| Callout | `callout_trigger: true` | Triggered by external API call |
| Event | `event_trigger: true` | Triggered by Zuora events |

Schedule format follows cron: `second minute hour day month weekday`

Callout triggers are incompatible with File-Field upload parameters.

## Common Workflow Patterns

Refer to `references/task-types.md` for the full task type reference. Here are common automation patterns:

- **Dunning/Collections**: Export overdue invoices → Iterate by account → Query contact details → Send email
- **Subscription management**: Query subscriptions by criteria → Iterate → Callout to suspend/cancel/amend
- **Late fees**: Export overdue accounts → Iterate → Callout to add charge via Orders API
- **Data sync**: Query Zuora data → JSONTransform to target format → Callout to external system
- **Chargeback handling**: Export invoices with chargebacks → Write off via adjustments → Cancel subscriptions
- **Custom notifications**: Export/Query matching records → Email with Liquid templates
- **CSV-driven mass update**: File-Field upload → Iterate rows → Export matching records → Iterate (chunked) → Callout

## Important Notes

- **File-Field parameters**: the `parameters` block must carry `skipping_check`, `file_encryption`, `show_run_prompt`, `callout_response`, `secure_error_msgs` and `disable_pending_status` alongside `fields`, or the upload silently fails. See the CSV File Upload section.
- **Field writability**: verify per endpoint before designing the action. SOAP-readable is not SOAP-writable, and REST silently ignores unknown field names.
- **Liquid templates**: Email task bodies use Liquid templating (`{{ Data.Object.Field }}`, `{% for item in Data.Items %}`)
- **Data references**: Tasks access data from previous tasks via `Data.TaskName.Field` or `Data.Callout.field` patterns
- **Credentials**: Use `{{ Credentials.zuora.rest_endpoint }}` for the Zuora API base URL in Callout tasks — never hardcode URLs. It already includes the `/v1/` path segment, so append the resource directly (`accounts/{id}`, not `v1/accounts/{id}`).
- **JSONata**: JSONTransform tasks use JSONata expressions for data transformation
- **JSONata version (mandatory)**: every `Logic::JSONTransform` task with `processor: "JSONata"` in a newly built workflow JSON must carry `"version"` in its `parameters`, set to the latest JSONata version Zuora offers — `"2.2.2"` as of 2026-11-26. A task without the key is not acceptable output: before importing or handing over a workflow JSON, check every JSONata task for it. If Zuora releases a newer version, use that one and update this note. See the JSONTransform section of `references/task-types.md`.
- **CSS positioning**: Task positions in the UI are controlled by `css.top` and `css.left` — space tasks ~250px apart horizontally, and offset parallel branches vertically by ~180px
- **Concurrent limits**: Default to `9999999` unless the workflow needs throttling. Use `500` on Iterate tasks, `4` on Export tasks, and a modest value (~20) on Callout tasks that fire per record, raising it gradually while watching for 429s.
- **Export CSV filenames**: referenced as `<ObjectName>__<exportTaskId>.csv.zip`. Import reassigns task IDs, so re-point every Iterate `object` and Callout `files` key after import if the IDs shift.
- **Error handling**: Add Failure linkages for critical tasks, and consider retry rules on Callout tasks
- **Empty results**: `zero_result_stop: "true"` on an Export ends that branch quietly when nothing matches — useful as a deliberate skip mechanism


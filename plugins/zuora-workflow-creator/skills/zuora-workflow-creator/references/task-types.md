# Zuora Workflow Task Types Reference

This document covers every task type available in Zuora Workflows, with field-by-field explanations and ready-to-use templates drawn from real production workflows.

## Table of Contents

1. [Query Task](#query-task)
2. [Export Task](#export-task)
3. [Callout Task](#callout-task)
4. [Email Task](#email-task)
5. [Iterate Task](#iterate-task)
6. [JSONTransform Task](#jsontransform-task)
7. [Common Task Fields](#common-task-fields)
8. [Data Access Patterns](#data-access-patterns)
9. [Full Workflow Example](#full-workflow-example)

---

## Query Task

Queries a single Zuora object using ZOQL (Zuora Object Query Language). Returns results into `Data.<placement>` or `Data.<ObjectName>`.

```json
{
  "name": "Get Account by Number",
  "action_type": "Query",
  "object": "Account",
  "parameters": {
    "fields": {
      "Account": {
        "Id": "true",
        "Name": "true",
        "AccountNumber": "true",
        "Currency": "true"
      }
    },
    "placement": "",
    "batch_size": "2000",
    "where_clause": "AccountNumber = '{{Data.Workflow.MyParam|strip}}'",
    "strict_variables": "true",
    "zero_query_proceed": "false"
  },
  "object_id": null,
  "call_type": "SOAP",
  "css": { "top": "30px", "left": "120px" },
  "concurrent_limit": 9999999,
  "tags": [],
  "priority": "Medium"
}
```

**Key fields:**
- `object`: The Zuora object to query (Account, Subscription, Invoice, Payment, etc.)
- `fields`: Object → field → "true" map. Only selected fields are returned.
- `placement`: Where to store results in the Data namespace. Empty string = uses object name (e.g., `Data.Account`). Custom value like `"AccountOwner"` stores at `Data.AccountOwner`.
- `where_clause`: ZOQL WHERE clause. Supports `{{ Data.X.Y }}` variable interpolation. Use `|strip` filter to trim whitespace.
- `batch_size`: Max records per batch (default "2000").
- `zero_query_proceed`: `"false"` = stop workflow if no results. `"true"` = continue even with empty results.

**Accessing results:** Results are arrays. Use `Data.Account[0].Id` for first result, or iterate over `Data.Account` with an Iterate task.

---

## Export Task

Exports data using ZOQL joins across multiple related objects. More powerful than Query because it can join objects. Returns a CSV file that can be iterated over.

```json
{
  "name": "Get Overdue Invoices with Account Info",
  "action_type": "Export",
  "object": "InvoiceItem",
  "parameters": {
    "fields": {
      "Account": {
        "Id": "true",
        "Name": "true",
        "Currency": "true",
        "AccountNumber": "true"
      },
      "Invoice": {
        "Id": "true",
        "Balance": "true",
        "DueDate": "true",
        "InvoiceDate": "true",
        "InvoiceNumber": "true",
        "Status": "true"
      },
      "BillToContact": {
        "FirstName": "true",
        "LastName": "true",
        "WorkEmail": "true"
      }
    },
    "where_clause": "Invoice.DueDate <= 'today - 30 days' AND Invoice.Balance > 0 AND Invoice.Status = 'Posted' Order By Invoice.DueDate Desc"
  },
  "object_id": null,
  "call_type": "SOAP",
  "css": { "top": "210px", "left": "400px" },
  "concurrent_limit": 4,
  "tags": [],
  "priority": "Medium"
}
```

**Key differences from Query:**
- Can join related objects (Account + Invoice + BillToContact in one export)
- The `object` field specifies the base export object
- Results are stored as a CSV file (`<ObjectName>__<taskId>.csv.zip`)
- Typically followed by an Iterate task to process rows
- `where_clause` supports date arithmetic (`today - 30 days`) and ORDER BY
- Lower `concurrent_limit` recommended (e.g., 4) since exports are heavier operations

---

## Callout Task

Makes HTTP requests to Zuora APIs or external systems. The workhorse for taking action.

**GET Callout example (reading data):**
```json
{
  "name": "Get Subscription Details",
  "action_type": "Callout",
  "parameters": {
    "url": "{{ Credentials.zuora.rest_endpoint }}subscriptions/{{Data.Workflow.SubscriptionId|strip}}",
    "method": "GET",
    "headers": [
      { "key": "Content-Type", "value": "application/json" }
    ],
    "raw_body": "",
    "body_type": "raw",
    "api_name": "",
    "api_doc_url": "",
    "files": [],
    "form_datas": [{ "key": "", "value": "", "upload": "false" }],
    "validation": {
      "replace": "true",
      "zuora_call": "true",
      "status_codes": ["200"],
      "payload_location": "Callout"
    },
    "retry_rules": {
      "retry_count": "0",
      "retry_window": "30",
      "current_retry_count": "0"
    },
    "authorization": {
      "type": "zuora"
    },
    "finish_status": [""],
    "response_path": "",
    "workflow_tags": [],
    "polling_interval": "30",
    "strict_variables": "true",
    "validate_response": "false",
    "validation_scheme": "",
    "disable_validation": "false",
    "delete_payload_paths": [],
    "include_response_code": "true",
    "notification_history_enabled": "false"
  },
  "object": null,
  "object_id": null,
  "call_type": "SOAP",
  "css": { "top": "160px", "left": "800px" },
  "concurrent_limit": 9999999,
  "tags": [],
  "priority": "Medium",
  "assignment": [],
  "zuora_org_id": null,
  "zuora_org_ids": [],
  "subprocess_id": null
}
```

**PUT Callout example (updating data, inside an Iterate loop):**

When the Callout runs inside an Iterate loop over an Export CSV, reference the Export CSV in the `files` array. This gives the Callout access to the iteration context.

```json
{
  "name": "Update Account Custom Field",
  "action_type": "Callout",
  "parameters": {
    "url": "{{ Credentials.zuora.rest_endpoint }}accounts/{{Data.Account.Id}}",
    "method": "PUT",
    "headers": [
      { "key": "Content-Type", "value": "application/json" }
    ],
    "raw_body": "{\"CustomField__c\": \"{{Data.Account.AccountNumber}}\"}",
    "body_type": "raw",
    "api_name": "",
    "api_doc_url": "",
    "files": [
      { "key": "Account__1.csv.zip", "name": "", "upload": "false", "filename": "" }
    ],
    "form_datas": [{ "key": "", "value": "", "upload": "false" }],
    "validation": {
      "replace": "true",
      "zuora_call": "true",
      "status_codes": ["200"],
      "payload_location": "Callout"
    },
    "retry_rules": {
      "retry_count": "3",
      "retry_window": "30",
      "current_retry_count": "0"
    },
    "authorization": { "type": "zuora" },
    "finish_status": [""],
    "response_path": "",
    "workflow_tags": [],
    "polling_interval": "30",
    "strict_variables": "true",
    "validate_response": "false",
    "validation_scheme": "",
    "disable_validation": "false",
    "delete_payload_paths": [],
    "include_response_code": "true",
    "notification_history_enabled": "false"
  },
  "object": null,
  "object_id": null,
  "call_type": "SOAP",
  "css": { "top": "160px", "left": "800px" },
  "concurrent_limit": 9999999,
  "tags": [],
  "priority": "Medium",
  "assignment": [],
  "zuora_org_id": null,
  "zuora_org_ids": [],
  "subprocess_id": null
}
```

**Key fields:**
- `url`: API endpoint. Always use `{{ Credentials.zuora.rest_endpoint }}` as the base for Zuora calls, followed directly by the resource path (e.g., `accounts/`, `subscriptions/`). **Do NOT add `v1/` prefix** — the rest_endpoint already includes the full base path. For external systems, use the full URL.
- `method`: GET, POST, PUT, DELETE, PATCH
- `headers`: Array of key-value pairs. Always include Content-Type.
- `raw_body`: Request body for POST/PUT. Supports `{{ Data.X.Y }}` interpolation and `{{ Data.JSONTransform | to_json }}` for sending transformed data. **Custom field names must use exact casing as defined in Zuora** (e.g., `OrganizationNumber__c` not `organizationNumber__c`).
- `files`: When a Callout runs inside an Iterate loop, reference the Export CSV file here (e.g., `"key": "Account__1.csv.zip"`). This connects the Callout to the iteration data context.
- `authorization.type`: `"zuora"` for Zuora API calls (auto-injects auth token). `"none"` for external calls.
- `validation.zuora_call`: `"true"` for Zuora API endpoints.
- `validation.replace`: `"true"` = store response replacing any previous Callout data. `"false"` = append.
- `validation.status_codes`: Use `["200"]` for expected success codes. Do NOT include empty strings.
- `validation.payload_location`: Where to store the response. `"Callout"` = `Data.Callout`. Custom name = `Data.CustomName`.
- `retry_rules`: Auto-retry on failure. `retry_count` = max retries, `retry_window` = seconds between retries.

**Required additional parameters**: Always include ALL of these in every Callout task: `api_name`, `api_doc_url`, `finish_status`, `response_path`, `workflow_tags`, `polling_interval`, `strict_variables`, `validate_response`, `validation_scheme`, `disable_validation`, `delete_payload_paths`, `include_response_code`, and `notification_history_enabled`.

---

## Email Task

Sends emails with Liquid template support. Can include Zuora invoice PDFs as attachments.

```json
{
  "name": "Send Overdue Notice",
  "action_type": "Email",
  "parameters": {
    "email": {
      "to": ["{{Data.BillToContact.WorkEmail}}"],
      "cc": [],
      "bcc": [],
      "from": "billing@yourcompany.com",
      "name": "Billing Department",
      "reply_to": "billing@yourcompany.com",
      "subject": "Action Required: Invoice Past Due - {{Data.Account.Name}}",
      "template": "<html><body><p>Dear {{Data.BillToContact.FirstName}},</p><p>Your invoice {{Data.Invoice.InvoiceNumber}} with balance {{Data.Invoice.Balance}} is past due.</p>{% for invoice in Data.Invoice %}<p>{{ invoice.InvoiceNumber }}: {{ invoice.Balance | money: Data.Account.Currency }}</p>{% endfor %}<p>Please remit payment at your earliest convenience.</p></body></html>",
      "attachments": {
        "invoices": "true"
      },
      "preview_only": "false"
    }
  },
  "object": null,
  "object_id": null,
  "call_type": "SOAP",
  "css": { "top": "310px", "left": "720px" },
  "concurrent_limit": 9999999,
  "tags": [],
  "priority": "Medium"
}
```

**Template features:**
- Liquid variables: `{{ Data.Object.Field }}`
- Liquid loops: `{% for item in Data.Items %}...{% endfor %}`
- Liquid filters: `{{ value | money: currency }}`, `{{ date | date: "%b %e %Y" }}`
- Liquid assignment: `{% assign total = 0.0 %}`
- Liquid conditionals: `{% if Data.Account.Balance > 0 %}...{% endif %}`

---

## Iterate Task

Loops through data from a previous Export task, executing downstream tasks for each row or unique field value. The Iterate task reads the CSV file produced by an Export task and feeds each row (or unique value) into the downstream tasks connected via "For Each" linkages.

**Default iteration (every row) — use this for most cases:**
```json
{
  "name": "Iterate Over Accounts",
  "action_type": "Iterate",
  "object": "Account__1.csv.zip",
  "parameters": {
    "file_type": "CSV",
    "chunk_size": "0",
    "skip_headers": "0",
    "skip_trailer": "false",
    "iteration_type": "Default",
    "strict_variables": "true",
    "csv_header_filter": "",
    "disable_validation": "false",
    "delete_payload_paths": [],
    "generate_auto_headers": "false"
  },
  "object_id": null,
  "call_type": "SOAP",
  "css": { "top": "160px", "left": "510px" },
  "concurrent_limit": 500,
  "tags": [],
  "priority": "Medium",
  "assignment": [],
  "zuora_org_id": null,
  "zuora_org_ids": [],
  "subprocess_id": null
}
```

**Unique-Field iteration (deduplicated):**
```json
{
  "name": "Iterate Unique Accounts",
  "action_type": "Iterate",
  "object": "InvoiceItem__1.csv.zip",
  "parameters": {
    "file_type": "CSV",
    "chunk_size": "0",
    "skip_headers": "0",
    "skip_trailer": "false",
    "iteration_type": "Unique-Field",
    "iteration_field": "Account.Id",
    "strict_variables": "true",
    "csv_header_filter": "",
    "disable_validation": "false",
    "delete_payload_paths": [],
    "generate_auto_headers": "false"
  },
  "object_id": null,
  "call_type": "SOAP",
  "css": { "top": "160px", "left": "510px" },
  "concurrent_limit": 500,
  "tags": [],
  "priority": "Medium",
  "assignment": [],
  "zuora_org_id": null,
  "zuora_org_ids": [],
  "subprocess_id": null
}
```

**Key fields:**
- `object`: References the CSV file produced by an Export task. Format: `<ObjectName>__<exportTaskId>.csv.zip`
- `file_type`: Always `"CSV"` for Export task output.
- `iteration_type`:
  - `"Default"` — iterates over every row in the CSV. **Use this for simple row-by-row processing.**
  - `"Unique-Field"` — iterates once per unique value of the specified field (deduplication).
- `iteration_field`: The field to iterate on (only for Unique-Field type), e.g., `"Account.Id"`. Omit or leave empty for Default iteration.
- `chunk_size`: `"0"` for single-record iteration, or a number for batch processing.
- `skip_headers`: `"0"` — number of header rows to skip.
- `skip_trailer`: `"false"` — whether to skip trailer rows.
- `concurrent_limit`: **Use `500` for Iterate tasks** to avoid overwhelming downstream tasks. Do NOT use `9999999`.

**⚠️ IMPORTANT**: Do NOT use `"Record"` as `iteration_type` — it is not valid. Use `"Default"` for row-by-row iteration.

**Required CSV parameters**: Always include ALL of these: `file_type`, `skip_headers`, `skip_trailer`, `strict_variables`, `csv_header_filter`, `disable_validation`, `delete_payload_paths`, and `generate_auto_headers`.

**Linkage note:** The Iterate task connects to its child tasks using `"For Each"` linkage type.

---

## JSONTransform Task

Transforms data using JSONata expressions. Used to reshape data before a Callout task.

```json
{
  "name": "Build Subscription Payload",
  "action_type": "Logic::JSONTransform",
  "parameters": {
    "template": "{\n  \"subscribes\": [{\n    \"Account\": { \"Id\": \"{{Data.Account[0].Id}}\" },\n    \"SubscriptionData\": {\n      \"Subscription\": {\n        \"ContractEffectiveDate\": \"\",\n        \"Status\": \"Draft\",\n        \"AutoRenew\": $.Callout.autoRenew,\n        \"TermType\": $.Callout.termType\n      },\n      \"RatePlanData\": [\n        $.Callout.ratePlans.({\n          \"RatePlan\": { \"ProductRatePlanId\": $.productRatePlanId },\n          \"RatePlanChargeData\": [\n            $.ratePlanCharges.({\n              \"RatePlanCharge\": {\n                \"ProductRatePlanChargeId\": $.productRatePlanChargeId,\n                \"Price\": $.price\n              }\n            })\n          ]\n        })\n      ]\n    }\n  }]\n}",
    "placement": "JSONTransform",
    "processor": "JSONata",
    "csv_headers": "",
    "strict_variables": "true",
    "disable_validation": "false"
  },
  "object": null,
  "object_id": null,
  "call_type": "SOAP",
  "css": { "top": "30px", "left": "790px" },
  "concurrent_limit": 9999999,
  "tags": [],
  "priority": "Medium"
}
```

**Key fields:**
- `template`: The JSONata expression. Mix of literal JSON and JSONata syntax (`$.Callout.field`, conditionals with `? :`, array mapping).
- `placement`: Where to store the result. Default `"JSONTransform"` → accessible as `Data.JSONTransform`.
- `processor`: Always `"JSONata"`.

**Using the output:** In the next Callout task, reference it as `{{Data.JSONTransform | to_json}}` in the `raw_body`.

---

## Common Task Fields

Every task shares these fields:

```json
{
  "id": null,
  "name": "Descriptive Task Name",
  "action_type": "Query|Export|Callout|Email|Iterate|Logic::JSONTransform",
  "object": null,
  "object_id": null,
  "call_type": "SOAP",
  "task_id": null,
  "css": { "top": "30px", "left": "120px" },
  "concurrent_limit": 9999999,
  "tags": [],
  "priority": "Medium",
  "assignment": [],
  "zuora_org_id": null,
  "zuora_org_ids": [],
  "subprocess_id": null
}
```

- `id`: Set to a temporary integer for cross-referencing in linkages. The import will assign real IDs.
- `task_id`: The ID of the *previous* task in the chain (used by the system for ordering). For the first task, this is `null`.
- `css`: Visual position in the Workflow Builder UI. Space tasks ~250px apart on the `left` axis.

---

## Data Access Patterns

Data flows through workflows via the `Data` namespace:

| Source | Access Pattern | Example |
|--------|---------------|---------|
| Workflow parameters | `Data.Workflow.ParamName` | `{{Data.Workflow.AccountNumber\|strip}}` |
| Query results | `Data.<ObjectOrPlacement>[index].Field` | `{{Data.Account[0].Id}}` |
| Export results | `Data.<Object>.Field` (in Iterate context) | `{{Data.Invoice.Balance}}` |
| Callout response | `Data.Callout.field` or `Data.<PayloadLocation>.field` | `{{Data.Callout.subscriptions[0].id}}` |
| JSONTransform output | `Data.JSONTransform` | `{{Data.JSONTransform \| to_json}}` |
| Iterate context | Current item fields are directly available | `{{Data.Account.Id}}` |

**Liquid filters:** `|strip` (trim whitespace), `| to_json` (serialize object), `| money: currency` (format currency), `| date: "%Y-%m-%d"` (format date).

---

## Full Workflow Example

Here's a complete workflow that exports accounts with an empty custom field and backfills it with the account number. This is a real production-tested pattern (Export → Iterate → Callout):

```json
{
  "workflow": {
    "name": "Backfill Organization Number from Account Number",
    "description": "Exports accounts where OrganizationNumber__c is empty and sets it to the AccountNumber",
    "parameters": { "fields": [] },
    "type": "Workflow::Setup",
    "ondemand_trigger": true,
    "callout_trigger": true,
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
  "tasks": [
    {
      "id": 1,
      "name": "Export Accounts Without Org Number",
      "action_type": "Export",
      "object": "Account",
      "parameters": {
        "fields": {
          "Account": { "Id": "true", "AccountNumber": "true" }
        },
        "where_clause": "Account.organizationNumber__c = ''",
        "workflow_tags": [],
        "delete_payload_paths": []
      },
      "object_id": null,
      "call_type": "SOAP",
      "task_id": null,
      "css": { "top": "150px", "left": "150px" },
      "concurrent_limit": 4,
      "tags": [],
      "priority": "Medium",
      "assignment": [],
      "zuora_org_id": null,
      "zuora_org_ids": [],
      "subprocess_id": null
    },
    {
      "id": 2,
      "name": "Iterate Over Accounts",
      "action_type": "Iterate",
      "object": "Account__1.csv.zip",
      "parameters": {
        "file_type": "CSV",
        "chunk_size": "0",
        "skip_headers": "0",
        "skip_trailer": "false",
        "iteration_type": "Default",
        "strict_variables": "true",
        "csv_header_filter": "",
        "disable_validation": "false",
        "delete_payload_paths": [],
        "generate_auto_headers": "false"
      },
      "object_id": null,
      "call_type": "SOAP",
      "task_id": 1,
      "css": { "top": "160px", "left": "510px" },
      "concurrent_limit": 500,
      "tags": [],
      "priority": "Medium",
      "assignment": [],
      "zuora_org_id": null,
      "zuora_org_ids": [],
      "subprocess_id": null
    },
    {
      "id": 3,
      "name": "Update Org Number with Account Number",
      "action_type": "Callout",
      "parameters": {
        "url": "{{ Credentials.zuora.rest_endpoint }}accounts/{{Data.Account.Id}}",
        "method": "PUT",
        "headers": [{ "key": "Content-Type", "value": "application/json" }],
        "raw_body": "{\"OrganizationNumber__c\": \"{{Data.Account.AccountNumber}}\"}",
        "body_type": "raw",
        "api_name": "",
        "api_doc_url": "",
        "files": [
          { "key": "Account__1.csv.zip", "name": "", "upload": "false", "filename": "" }
        ],
        "form_datas": [{ "key": "", "value": "", "upload": "false" }],
        "validation": {
          "replace": "true",
          "zuora_call": "true",
          "status_codes": ["200"],
          "payload_location": "Callout"
        },
        "retry_rules": { "retry_count": "3", "retry_window": "30", "current_retry_count": "0" },
        "authorization": { "type": "zuora" },
        "finish_status": [""],
        "response_path": "",
        "workflow_tags": [],
        "polling_interval": "30",
        "strict_variables": "true",
        "validate_response": "false",
        "validation_scheme": "",
        "disable_validation": "false",
        "delete_payload_paths": [],
        "include_response_code": "true",
        "notification_history_enabled": "false"
      },
      "object": null,
      "object_id": null,
      "call_type": "SOAP",
      "task_id": 2,
      "css": { "top": "160px", "left": "800px" },
      "concurrent_limit": 9999999,
      "tags": [],
      "priority": "Medium",
      "assignment": [],
      "zuora_org_id": null,
      "zuora_org_ids": [],
      "subprocess_id": null
    }
  ],
  "linkages": [
    { "source_workflow_id": 1, "source_task_id": null, "target_task_id": 1, "linkage_type": "Start" },
    { "source_workflow_id": null, "source_task_id": 1, "target_task_id": 2, "linkage_type": "Success" },
    { "source_workflow_id": null, "source_task_id": 2, "target_task_id": 3, "linkage_type": "For Each" }
  ]
}
```

**Note on IDs in the import payload:** When importing, use temporary integer IDs for tasks. Use those same IDs in `source_task_id` and `target_task_id` in linkages. For the Start linkage, `source_workflow_id` can be any placeholder — Zuora resolves it on import.

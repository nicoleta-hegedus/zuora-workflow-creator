# Liquid Template Language Reference for Zuora Workflows

Liquid is an open-source template language (created by Shopify, v5.0.1 in Zuora) used throughout Zuora Workflow tasks to display dynamic content, filter data, and control logic flow. This reference covers both standard Liquid syntax and Zuora-specific extensions.

Full Liquid docs: https://shopify.github.io/liquid/

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Objects (Output)](#objects)
3. [Tags (Logic & Control Flow)](#tags)
4. [Standard Filters](#standard-filters)
5. [Zuora-Specific Filters](#zuora-specific-filters)
6. [Zuora Credential Objects](#zuora-credential-objects)
7. [Zuora Data Namespace](#zuora-data-namespace)
8. [Logic::If Task](#logicif-task)
9. [Logic::Liquid Task](#logicliquid-task)
10. [Whitespace Control](#whitespace-control)
11. [Common Patterns in Zuora Workflows](#common-patterns)

---

## Core Concepts

Liquid has three building blocks:

- **Objects** — output dynamic content, wrapped in `{{ }}` double curly braces
- **Tags** — logic and control flow, wrapped in `{% %}` curly brace + percent signs
- **Filters** — modify output, applied with pipe `|` character inside `{{ }}`

---

## Objects

Objects contain the content Liquid renders. Access object properties with dot notation:

```liquid
{{ Data.Account.Name }}
{{ Data.Invoice[0].Balance }}
{{ Credentials.zuora.rest_endpoint }}
```

Variable types: String (in quotes), Number (no quotes), Boolean (`true`/`false`), Nil (`nil`), Array (indexed `[0]`, `[1]`), Object (dot-accessed `.Property`).

---

## Tags

### Variable Assignment

```liquid
{% assign my_var = "hello" %}
{% assign total = 0 %}
{% assign filtered = Data.Invoice | where: "Status", "Posted" %}
```

### Capture (multi-line variable assignment)

```liquid
{% capture greeting %}
Hello {{ Data.BillToContact.FirstName }}, your balance is {{ Data.Account.Balance }}.
{% endcapture %}
{{ greeting }}
```

### Conditionals

```liquid
{% if Data.Account.Balance > 0 %}
  Account has outstanding balance.
{% elsif Data.Account.Balance == 0 %}
  Account is settled.
{% else %}
  Account has credit.
{% endif %}
```

```liquid
{% unless Data.Subscription.Status == "Cancelled" %}
  Subscription is still active.
{% endunless %}
```

```liquid
{% case Data.PaymentMethod.Type %}
  {% when "CreditCard" %}
    Payment by card.
  {% when "ACH" %}
    Payment by ACH.
  {% else %}
    Other payment type.
{% endcase %}
```

**Operators:** `==`, `!=`, `>`, `<`, `>=`, `<=`, `and`, `or`, `contains`

Note: There is no `not` operator; use `unless` or nested `if` instead. Parentheses for grouping are not supported.

### Loops

```liquid
{% for invoice in Data.Invoice %}
  Invoice: {{ invoice.InvoiceNumber }} — Balance: {{ invoice.Balance }}
{% endfor %}
```

**Loop variables:**
- `forloop.index` — current iteration (1-based)
- `forloop.index0` — current iteration (0-based)
- `forloop.first` — `true` if first iteration
- `forloop.last` — `true` if last iteration
- `forloop.length` — total number of iterations

**Loop modifiers:**
```liquid
{% for item in Data.Items limit:5 %}      <!-- only first 5 -->
{% for item in Data.Items offset:2 %}      <!-- skip first 2 -->
{% for i in (1..10) %}                     <!-- numeric range -->
```

### Comments

```liquid
{% comment %}
  This will not be rendered in output.
{% endcomment %}
```

---

## Standard Filters

### String Filters

| Filter | Description | Example |
|--------|-------------|---------|
| `append` | Append string | `{{ "Hello" | append: " World" }}` → `Hello World` |
| `prepend` | Prepend string | `{{ "World" | prepend: "Hello " }}` → `Hello World` |
| `capitalize` | Capitalize first char | `{{ "hello" | capitalize }}` → `Hello` |
| `upcase` | Uppercase all | `{{ "hello" | upcase }}` → `HELLO` |
| `downcase` | Lowercase all | `{{ "HELLO" | downcase }}` → `hello` |
| `strip` | Trim whitespace | `{{ "  hi  " | strip }}` → `hi` |
| `lstrip` / `rstrip` | Trim left/right | `{{ "  hi  " | lstrip }}` → `hi  ` |
| `replace` | Replace substring | `{{ "Hello" | replace: "l", "r" }}` → `Herro` |
| `replace_first` | Replace first match | `{{ "Hello" | replace_first: "l", "r" }}` → `Herlo` |
| `remove` | Remove substring | `{{ "Hello" | remove: "l" }}` → `Heo` |
| `remove_first` | Remove first match | `{{ "Hello" | remove_first: "l" }}` → `Helo` |
| `truncate` | Truncate to N chars | `{{ "Hello World" | truncate: 8, "..." }}` → `Hello...` |
| `truncatewords` | Truncate to N words | `{{ "one two three" | truncatewords: 2 }}` → `one two...` |
| `split` | Split into array | `{{ "a,b,c" | split: "," }}` → `["a","b","c"]` |
| `strip_html` | Remove HTML tags | `{{ "<b>Hi</b>" | strip_html }}` → `Hi` |
| `strip_newlines` | Remove newlines | `{{ text | strip_newlines }}` |
| `newline_to_br` | Convert `\n` to `<br>` | `{{ text | newline_to_br }}` |
| `escape` | HTML-escape | `{{ "<p>" | escape }}` → `&lt;p&gt;` |
| `url_encode` | URL-encode | `{{ "a b" | url_encode }}` → `a+b` |
| `url_decode` | URL-decode | `{{ "a+b" | url_decode }}` → `a b` |

### Math Filters

| Filter | Description | Example |
|--------|-------------|---------|
| `plus` | Add | `{{ 5 | plus: 3 }}` → `8` |
| `minus` | Subtract | `{{ 10 | minus: 3 }}` → `7` |
| `times` | Multiply | `{{ 5 | times: 3 }}` → `15` |
| `divided_by` | Divide | `{{ 10 | divided_by: 3 }}` → `3` |
| `modulo` | Remainder | `{{ 10 | modulo: 3 }}` → `1` |
| `round` | Round | `{{ 4.6 | round }}` → `5` |
| `ceil` | Round up | `{{ 4.1 | ceil }}` → `5` |
| `floor` | Round down | `{{ 4.9 | floor }}` → `4` |
| `abs` | Absolute value | `{{ -5 | abs }}` → `5` |

### Array Filters

| Filter | Description | Example |
|--------|-------------|---------|
| `size` | Array/string length | `{{ Data.Invoice | size }}` |
| `first` | First element | `{{ Data.Invoice | first }}` |
| `last` | Last element | `{{ Data.Invoice | last }}` |
| `join` | Join array to string | `{{ array | join: ", " }}` |
| `sort` | Sort array | `{{ Data.Invoice | sort: "DueDate" }}` |
| `reverse` | Reverse array | `{{ Data.Invoice | reverse }}` |
| `map` | Extract property | `{{ Data.Invoice | map: "InvoiceNumber" }}` |
| `uniq` | Remove duplicates | `{{ array | uniq }}` |
| `compact` | Remove nils | `{{ array | compact }}` |
| `concat` | Merge arrays | `{{ arr1 | concat: arr2 }}` |

### Date Filters

| Filter | Description | Example |
|--------|-------------|---------|
| `date` | Format date | `{{ "now" | date: "%Y-%m-%d" }}` → `2026-03-31` |

**Common date format tokens:** `%Y` (4-digit year), `%m` (month 01-12), `%d` (day 01-31), `%H` (hour 00-23), `%M` (minute 00-59), `%S` (second 00-59), `%b` (abbreviated month name), `%e` (day without leading zero), `%s` (Unix timestamp).

### Other Filters

| Filter | Description | Example |
|--------|-------------|---------|
| `default` | Fallback value | `{{ Data.X | default: "N/A" }}` |
| `json` | Convert to JSON | `{{ object | json }}` |

---

## Zuora-Specific Filters

These filters are exclusive to Zuora Workflow and extend standard Liquid:

### `date_manip`
Add or subtract time from a date.

Parameters: `input`, `operation` (`+` or `-`), `integer`, `metric` (`minute`, `hour`, `day`, `month`, `year`)

```liquid
{{ '2021-01-01' | date_manip: '+', 1, 'day' | date: '%Y-%m-%d' }}
=> 2021-01-02

{{ '2021-06-15' | date_manip: '-', 3, 'month' | date: '%Y-%m-%d' }}
=> 2021-03-15

{{ 'now' | date_manip: '+', 30, 'day' | date: '%Y-%m-%d' }}
=> (30 days from now)
```

### `money`
Format a number as currency.

```liquid
{{ invoice.Balance | money: Data.Account.Currency }}
=> $1,234.56
```

### `to_json`
Convert a Liquid object to a JSON string. Critical for passing data in Callout task bodies.

```liquid
{{ Data.JSONTransform | to_json }}
```

### `where`
Filter an array by a property value.

```liquid
{% assign active_subs = Data.Subscription | where: "Status", "Active" %}
{% assign account = Data.Account | where: "AccountNumber", "A00000052" %}
{{ account[0].Id }}
```

### `where_exp`
Filter an array by a custom expression.

```liquid
{% assign overdue = Data.Invoice | where_exp: "item", "item.Balance > 100" %}
{{ overdue[0].Id }}
```

### `check_parent_workflow_status`
Check if parent workflow runs are in a given status. Statuses: `Queued`, `Processing`, `Stopped`, `Stopping`, `Finished`.

```liquid
{{ WorkflowInstance.id | check_parent_workflow_status: 'Queued' }}
=> true/false
```

### `is_ar_enabled`
Check if Invoice Settlement (AR) is enabled in the tenant.

```liquid
{{ nil | is_ar_enabled }}
=> true/false
```

---

## Zuora Credential Objects

These objects provide access to the workflow instance's connection credentials. Use them in Callout tasks to construct API URLs:

| Object | Returns |
|--------|---------|
| `{{ Credentials.zuora.rest_endpoint }}` | REST API base URL (e.g., `https://rest.zuora.com/v1/`) |
| `{{ Credentials.zuora.url }}` | SOAP API URL |
| `{{ Credentials.zuora.username }}` | API username |
| `{{ Credentials.zuora.password }}` | API password |
| `{{ Credentials.zuora.session }}` | Current session token |
| `{{ Credentials.zuora.entity_id }}` | Entity ID of the workflow instance |

**Important:** Always use `{{ Credentials.zuora.rest_endpoint }}` as the base URL in Callout tasks. Never hardcode Zuora API URLs.

---

## Zuora Data Namespace

All workflow data is accessed through the `Data` namespace. See `references/data-sources.md` for the full object reference.

| Pattern | Description |
|---------|-------------|
| `Data.Workflow.ParamName` | Workflow input parameters |
| `Data.<ObjectName>[index].Field` | Query task results (array) |
| `Data.<Placement>[index].Field` | Query results with custom placement |
| `Data.Callout.field` | Callout response data |
| `Data.<PayloadLocation>.field` | Callout with custom payload location |
| `Data.JSONTransform` | JSONTransform task output |
| `Data.Liquid.variable_name` | Logic::Liquid task variables |
| `WorkflowInstance.id` | Current workflow run ID |
| `WorkflowInstance.name` | Current workflow name |

---

## Logic::If Task

The Logic::If task creates conditional branches in a workflow. Its Liquid expression **must evaluate to the string `True` or `False`** — not a truthy/falsy value, but the literal words.

```liquid
{% if Data.Subscription.size > 0 %}
True
{% else %}
False
{% endif %}
```

The task then follows the "True" or "False" linkage path. Be careful with whitespace — use `{%-` and `-%}` to strip whitespace that could interfere.

---

## Logic::Liquid Task

The Logic::Liquid task defines variables or performs Liquid operations. Variables are saved under `Data.Liquid`:

```liquid
{% assign today = 'now' | date: '%Y-%m-%d' %}
{% assign threshold_date = 'now' | date_manip: '-', 30, 'day' | date: '%Y-%m-%d' %}
{% assign total_balance = 0 %}
{% for invoice in Data.Invoice %}
  {% assign total_balance = total_balance | plus: invoice.Balance %}
{% endfor %}
```

Access these downstream as `{{ Data.Liquid.today }}`, `{{ Data.Liquid.total_balance }}`.

Variables prefixed with `_` (e.g., `_temp`) are local to the task and not saved to the payload.

When referencing Liquid variables in downstream tasks, enable **Skip Validation** in task settings.

---

## Whitespace Control

Use hyphens in Liquid tags to strip surrounding whitespace:

```liquid
{%- assign x = "hello" -%}       <!-- strips whitespace on both sides -->
{{- Data.Account.Name -}}         <!-- strips whitespace from output -->
```

This is important in Zuora Workflows because stray whitespace can cause issues with conditional evaluations, especially in Logic::If tasks.

---

## Common Patterns in Zuora Workflows

### Calculate total balance across invoices
```liquid
{% assign total = 0.0 %}
{% for invoice in Data.Invoice %}
  {% assign total = total | plus: invoice.Balance %}
{% endfor %}
Total: {{ total | money: Data.Account.Currency }}
```

### Format a date for API calls
```liquid
{% assign effective_date = 'now' | date: '%Y-%m-%d' %}
```

### Build a dynamic where clause
```liquid
{% assign account_id = Data.Workflow.AccountId | strip %}
AccountId = '{{ account_id }}' AND Status = 'Active'
```

### Conditional email content
```liquid
{% if Data.Invoice.size > 1 %}
  You have {{ Data.Invoice.size }} overdue invoices.
{% else %}
  You have 1 overdue invoice.
{% endif %}
```

### Access nested Callout response data
```liquid
{{ Data.Callout.subscriptions[0].id }}
{{ Data.Callout.success }}
{{ Data.Callout.reasons[0].message }}
```

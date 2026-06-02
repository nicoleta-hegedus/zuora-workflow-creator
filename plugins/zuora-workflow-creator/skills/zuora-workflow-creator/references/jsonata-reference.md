# JSONata Language Reference for Zuora Workflows

JSONata is a lightweight query and transformation language for JSON data, used in Zuora Workflow **JSONTransform** tasks to reshape data between workflow steps. This reference covers the core language features relevant to Zuora Workflows.

Full JSONata docs: https://docs.jsonata.org/overview.html
Interactive playground: http://try.jsonata.org/

## Table of Contents

1. [Overview & Where It's Used](#overview)
2. [Navigation & Simple Queries](#navigation)
3. [Predicates (Filtering)](#predicates)
4. [Expressions & Operators](#expressions)
5. [Result Structures (Object/Array Construction)](#result-structures)
6. [String Functions](#string-functions)
7. [Numeric Functions](#numeric-functions)
8. [Aggregation Functions](#aggregation-functions)
9. [Boolean Functions](#boolean-functions)
10. [Array Functions](#array-functions)
11. [Object Functions](#object-functions)
12. [Conditional Expressions](#conditional-expressions)
13. [Variables & Bindings](#variables)
14. [Context References in Zuora](#zuora-context)
15. [Common Zuora Workflow Patterns](#common-patterns)

---

## Overview

JSONata is used in Zuora Workflow **Logic::JSONTransform** tasks. The JSONata expression is placed in the task's `template` field with `processor: "JSONata"`. The output is stored at `Data.JSONTransform` (or a custom `placement`) and can be passed to subsequent Callout tasks via `{{ Data.JSONTransform | to_json }}`.

Key concepts:
- JSONata is a **superset of JSON** — any valid JSON is a valid JSONata expression
- It uses path expressions similar to XPath to navigate JSON structures
- The `$` symbol refers to the root of the input data
- The `%` symbol refers to the parent context (useful in nested mappings)

---

## Navigation

### Dot notation for objects
```jsonata
$.Callout.subscriptions        /* array of subscriptions from Callout response */
$.Callout.subscriptions[0].id  /* first subscription's ID */
$.Account.Name                 /* Account name */
```

### Array indexing
```jsonata
$.items[0]          /* first item (0-based) */
$.items[-1]         /* last item */
$.items[0..2]       /* first 3 items (range) */
```

### Wildcard
```jsonata
$.Address.*         /* all values from Address object */
```

### Select all values of a field from an array
```jsonata
$.Callout.ratePlans.productRatePlanId
/* returns array of all productRatePlanId values */
```

---

## Predicates

Filter arrays using conditions in square brackets:

```jsonata
$.Callout.ratePlans[lastChangeType != 'Remove']
/* rate plans that haven't been removed */

$.Callout.ratePlanCharges[type = 'Recurring']
/* only recurring charges */

$.Callout.ratePlanCharges[price > 0]
/* charges with positive price */

$.Callout.ratePlanCharges[model != 'FlatFee' and type != 'Usage']
/* charges that are neither flat fee nor usage */
```

---

## Expressions

### String expressions
```jsonata
FirstName & ' ' & Surname           /* concatenation with & */
Address.(Street & ', ' & City)       /* scoped concatenation */
```

### Numeric operators
`+` (add), `-` (subtract), `*` (multiply), `/` (divide), `%` (modulo)

### Comparison operators
`=`, `!=`, `<`, `>`, `<=`, `>=`, `in`

```jsonata
$.price > 100
$.status = "Active"
"USD" in $.currencies
```

### Boolean operators
`and`, `or`

```jsonata
($.quantity > 0) and ($.price != null)
```

---

## Result Structures

### Object construction
Build new JSON objects using `{ }`:

```jsonata
{
  "RatePlan": {
    "ProductRatePlanId": $.productRatePlanId
  },
  "RatePlanChargeData": [
    $.ratePlanCharges.({
      "RatePlanCharge": {
        "ProductRatePlanChargeId": $.productRatePlanChargeId,
        "Price": $.price,
        "Quantity": $.quantity
      }
    })
  ]
}
```

### Array construction
Build arrays using `[ ]`:

```jsonata
[$.firstName, $.lastName, $.email]
```

### Array mapping with `.( )`
Map over an array, transforming each element:

```jsonata
$.Callout.ratePlans.({
  "planId": $.productRatePlanId,
  "planName": $.productRatePlanName,
  "charges": $.ratePlanCharges.({
    "chargeId": $.productRatePlanChargeId,
    "amount": $.price
  })
})
```

---

## String Functions

| Function | Description | Example |
|----------|-------------|---------|
| `$string(value)` | Cast to string | `$string(42)` → `"42"` |
| `$length(str)` | String length | `$length("hello")` → `5` |
| `$substring(str, start, len)` | Extract substring | `$substring("hello", 1, 3)` → `"ell"` |
| `$substringBefore(str, chars)` | Before first occurrence | `$substringBefore("hello-world", "-")` → `"hello"` |
| `$substringAfter(str, chars)` | After first occurrence | `$substringAfter("hello-world", "-")` → `"world"` |
| `$uppercase(str)` | Uppercase | `$uppercase("hello")` → `"HELLO"` |
| `$lowercase(str)` | Lowercase | `$lowercase("HELLO")` → `"hello"` |
| `$trim(str)` | Trim whitespace | `$trim("  hi  ")` → `"hi"` |
| `$pad(str, width, char)` | Pad string | `$pad("42", 5, "0")` → `"00042"` |
| `$contains(str, pattern)` | Test contains | `$contains("hello", "ell")` → `true` |
| `$split(str, separator)` | Split to array | `$split("a,b,c", ",")` → `["a","b","c"]` |
| `$join(array, separator)` | Join array | `$join(["a","b"], ",")` → `"a,b"` |
| `$replace(str, pattern, repl)` | Replace | `$replace("hello", "l", "r")` → `"herro"` |
| `$match(str, regex)` | Regex match | `$match("abc123", /[0-9]+/)` |
| `$eval(str)` | Parse & evaluate | `$eval("[1,2,3]")` → `[1,2,3]` |
| `$base64encode(str)` | Base64 encode | `$base64encode("user:pass")` |
| `$base64decode(str)` | Base64 decode | `$base64decode("dXNlcjpwYXNz")` |
| `$encodeUrlComponent(str)` | URL encode | `$encodeUrlComponent("a=b")` → `"a%3Db"` |

---

## Numeric Functions

| Function | Description | Example |
|----------|-------------|---------|
| `$number(value)` | Cast to number | `$number("42.5")` → `42.5` |
| `$abs(n)` | Absolute value | `$abs(-5)` → `5` |
| `$floor(n)` | Round down | `$floor(4.9)` → `4` |
| `$ceil(n)` | Round up | `$ceil(4.1)` → `5` |
| `$round(n, precision)` | Round | `$round(4.567, 2)` → `4.57` |
| `$power(base, exp)` | Exponent | `$power(2, 3)` → `8` |
| `$sqrt(n)` | Square root | `$sqrt(16)` → `4` |
| `$random()` | Random 0-1 | `$random()` → `0.7312...` |
| `$formatNumber(n, picture)` | Format number | `$formatNumber(1234.5, "#,##0.00")` → `"1,234.50"` |
| `$formatBase(n, radix)` | Format base | `$formatBase(255, 16)` → `"ff"` |

---

## Aggregation Functions

| Function | Description | Example |
|----------|-------------|---------|
| `$sum(array)` | Sum | `$sum([1,2,3])` → `6` |
| `$max(array)` | Maximum | `$max([1,5,3])` → `5` |
| `$min(array)` | Minimum | `$min([1,5,3])` → `1` |
| `$average(array)` | Average | `$average([1,2,3])` → `2` |

Useful for summing invoice balances, charges, etc.:
```jsonata
$sum($.Callout.ratePlanCharges.price)
```

---

## Boolean Functions

| Function | Description | Example |
|----------|-------------|---------|
| `$not(value)` | Logical NOT | `$not(true)` → `false` |
| `$exists(value)` | Value exists? | `$exists($.Callout.error)` → `false` |
| `$boolean(value)` | Cast to boolean | `$boolean("")` → `false` |

---

## Array Functions

| Function | Description | Example |
|----------|-------------|---------|
| `$count(array)` | Array length | `$count($.items)` → `3` |
| `$append(arr1, arr2)` | Concatenate | `$append([1,2], [3,4])` → `[1,2,3,4]` |
| `$sort(array, func)` | Sort | `$sort($.items, function($a,$b){$a.price > $b.price})` |
| `$reverse(array)` | Reverse | `$reverse([1,2,3])` → `[3,2,1]` |
| `$shuffle(array)` | Randomize | `$shuffle([1,2,3])` |
| `$distinct(array)` | Unique values | `$distinct([1,1,2,3])` → `[1,2,3]` |
| `$zip(arr1, arr2)` | Zip together | `$zip([1,2], ["a","b"])` → `[[1,"a"],[2,"b"]]` |

---

## Object Functions

| Function | Description | Example |
|----------|-------------|---------|
| `$keys(obj)` | Get keys | `$keys({"a":1,"b":2})` → `["a","b"]` |
| `$values(obj)` | Get values | `$values({"a":1,"b":2})` → `[1,2]` |
| `$lookup(obj, key)` | Get value by key | `$lookup($.data, "name")` |
| `$spread(obj)` | Key-value pairs | `$spread({"a":1})` → `[{"a":1}]` |
| `$merge(arr)` | Merge objects | `$merge([{"a":1},{"b":2}])` → `{"a":1,"b":2}` |
| `$each(obj, func)` | Iterate object | `$each(obj, function($v,$k){...})` |
| `$type(value)` | Get type | `$type("hello")` → `"string"` |

---

## Conditional Expressions

JSONata uses the ternary `?` operator for conditionals:

```jsonata
$.price != null ? $.price              /* if price exists, return it */

$.model = "DiscountFixedAmount" ? $.discountAmount   /* conditional field */

$.termType = "TERMED" ? $.initialTerm  /* only include if termed */

$boolean($.value) ? $.value            /* include only if truthy */
```

**Chained conditions (if/else):**
```jsonata
$.status = "Active" ? "green" : $.status = "Suspended" ? "yellow" : "red"
```

---

## Variables

Bind values to variables with `:=`:

```jsonata
(
  $total := $sum($.Callout.ratePlanCharges.price);
  $count := $count($.Callout.ratePlanCharges);
  {
    "totalPrice": $total,
    "chargeCount": $count,
    "averagePrice": $total / $count
  }
)
```

---

## Context References in Zuora

In a JSONTransform task, the input data context is the workflow's `Data` namespace. JSONata accesses it with `$`:

| JSONata Reference | Equivalent Liquid | Description |
|-------------------|-------------------|-------------|
| `$.Callout` | `Data.Callout` | Callout response data |
| `$.Callout.subscriptions[0]` | `Data.Callout.subscriptions[0]` | First subscription |
| `$.Account[0].Id` | `Data.Account[0].Id` | First account ID from Query |
| `$.Workflow.ParamName` | `Data.Workflow.ParamName` | Workflow parameter |
| `%` | (parent context) | Parent scope in nested maps |

**Note on `%` (parent reference):** When iterating with `.()`, use `%` to reference the parent context:
```jsonata
$.Callout.ratePlanCharges.({
  "RatePlanChargeId": %.id,    /* % = parent ratePlan */
  "Price": $.price              /* $ = current charge */
})
```

---

## Common Zuora Workflow Patterns

### Build a subscription creation payload
```jsonata
{
  "subscribes": [{
    "Account": { "Id": $.Account[0].Id },
    "SubscriptionData": {
      "Subscription": {
        "ContractEffectiveDate": "",
        "Status": "Draft",
        "AutoRenew": $.Callout.autoRenew,
        "TermType": $.Callout.termType,
        "InitialTerm": $.Callout.termType = "TERMED" ? $.Callout.initialTerm
      },
      "RatePlanData": [
        $.Callout.ratePlans[lastChangeType != 'Remove'].({
          "RatePlan": {
            "ProductRatePlanId": $.productRatePlanId
          },
          "RatePlanChargeData": [
            $.ratePlanCharges.({
              "RatePlanCharge": {
                "ProductRatePlanChargeId": $.productRatePlanChargeId,
                "Quantity": ($.model != "FlatFee") and ($.type != "Usage") ? $.quantity,
                "Price": $.price != null ? $.price
              }
            })
          ]
        })
      ]
    }
  }]
}
```

### Transform an array of accounts into update payloads
```jsonata
$.accounts.({
  "AccountId": $.id,
  "CustomField__c": $uppercase($.name) & "-" & $.accountNumber
})
```

### Aggregate charges by type
```jsonata
(
  $charges := $.Callout.ratePlanCharges;
  {
    "recurring": $sum($charges[type="Recurring"].price),
    "oneTime": $sum($charges[type="OneTime"].price),
    "usage": $count($charges[type="Usage"]),
    "total": $sum($charges.price)
  }
)
```

### Filter and reshape data for an external API
```jsonata
{
  "records": $.Callout.subscriptions[status = "Active"].({
    "externalId": $.subscriptionNumber,
    "startDate": $.termStartDate,
    "mrr": $sum($.ratePlans.ratePlanCharges.mrr),
    "productNames": $join($.ratePlans.ratePlanName, "; ")
  })
}
```

### Conditionally include fields
```jsonata
{
  "name": $.Callout.name,
  "discount": $.Callout.discountPercentage != null ? {
    "type": "Percentage",
    "value": $.Callout.discountPercentage
  },
  "quantity": $.Callout.model != "FlatFee" ? $.Callout.quantity
}
```

Fields with a condition that evaluates to `false` are omitted from the output (they become `undefined`, which JSONata excludes).

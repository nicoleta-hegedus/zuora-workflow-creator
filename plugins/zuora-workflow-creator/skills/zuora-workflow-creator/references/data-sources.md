# Zuora Data Sources & Database Tables Reference

This reference covers every Zuora object you can query, export, or access through workflows. Use it when building Query tasks, Export tasks, or Callout tasks that interact with Zuora data.

## Table of Contents

1. [ZOQL Queryable Objects (for Query tasks)](#zoql-queryable-objects)
2. [Export Data Sources (for Export tasks)](#export-data-sources)
3. [REST API Queryable Objects (for Callout tasks)](#rest-api-queryable-objects)
4. [Object Relationships](#object-relationships)
5. [Common Query Patterns](#common-query-patterns)

---

## ZOQL Queryable Objects

These objects can be used in workflow **Query** tasks via standard ZOQL `SELECT ... FROM ... WHERE` syntax.

### Core Billing Objects

| Object | Key Fields | Description |
|--------|-----------|-------------|
| **Account** | Id, Name, AccountNumber, Status, Balance, Currency, BillCycleDay, Batch, CrmId, BillToId, SoldToId, DefaultPaymentMethodId, ParentId | Customer accounts |
| **Contact** | Id, AccountId, FirstName, LastName, WorkEmail, PersonalEmail, Address1, Address2, City, State, PostalCode, Country, Phone | Account contacts |
| **Subscription** | Id, AccountId, Name, SubscriptionNumber, Status, ContractEffectiveDate, TermStartDate, TermEndDate, InitialTerm, RenewalTerm, TermType, AutoRenew, Version | Subscriptions |
| **Amendment** | Id, SubscriptionId, Type, Name, Status, EffectiveDate, ContractEffectiveDate, Description | Subscription amendments |
| **RatePlan** | Id, SubscriptionId, ProductRatePlanId, Name, AmendmentType | Subscription rate plans |
| **RatePlanCharge** | Id, RatePlanId, ProductRatePlanChargeId, Name, ChargeType, ChargeModel, Price, Quantity, MRR, TCV, DMRC, DTCV, Segment, EffectiveStartDate, EffectiveEndDate, ProcessedThroughDate, ChargedThroughDate | Charge details on subscriptions |
| **RatePlanChargeTier** | Id, RatePlanChargeId, Tier, StartingUnit, EndingUnit, Price, PriceFormat | Tiered pricing details |

### Product Catalog Objects

| Object | Key Fields | Description |
|--------|-----------|-------------|
| **Product** | Id, Name, SKU, Description, EffectiveStartDate, EffectiveEndDate, Category | Products in catalog |
| **ProductRatePlan** | Id, ProductId, Name, Description, EffectiveStartDate, EffectiveEndDate | Product rate plans |
| **ProductRatePlanCharge** | Id, ProductRatePlanId, Name, ChargeType, ChargeModel, BillingPeriod, DefaultQuantity, UOM | Product charges |
| **ProductRatePlanChargeTier** | Id, ProductRatePlanChargeId, Tier, StartingUnit, EndingUnit, Price, PriceFormat, Currency | Product charge tier pricing |
| **Feature** | Id, Name, FeatureCode, Description, Status | Product features |
| **ProductFeature** | Id, ProductId, FeatureId | Product-feature associations |

### Invoice & Billing Objects

| Object | Key Fields | Description |
|--------|-----------|-------------|
| **Invoice** | Id, AccountId, InvoiceNumber, InvoiceDate, DueDate, Amount, Balance, Status, TargetDate, PostedDate | Invoices |
| **InvoiceItem** | Id, InvoiceId, AccountId, SubscriptionId, RatePlanChargeId, ProductId, ChargeAmount, TaxAmount, ServiceStartDate, ServiceEndDate, ChargeDate, ChargeName, Quantity, UOM | Invoice line items |
| **InvoiceAdjustment** | Id, InvoiceId, AccountId, Amount, Type, Status, AdjustmentDate, ReasonCode | Invoice-level adjustments |
| **InvoiceItemAdjustment** | Id, InvoiceItemId, InvoiceId, Amount, Type, Status, AdjustmentDate, ReasonCode | Line item adjustments |
| **TaxationItem** | Id, InvoiceItemId, Name, TaxRate, TaxAmount, TaxCode, TaxDate, ExemptAmount | Tax details |
| **BillingRun** | Id, Status, ExecutedDate, TargetDate, InvoiceDate | Billing run records |

### Payment Objects

| Object | Key Fields | Description |
|--------|-----------|-------------|
| **Payment** | Id, AccountId, Amount, Status, EffectiveDate, PaymentMethodId, Type, GatewayResponse, GatewayState, ReferenceId | Payments |
| **PaymentMethod** | Id, AccountId, Type, CreditCardType, CreditCardMaskNumber, CreditCardExpirationMonth, CreditCardExpirationYear, PaymentMethodStatus, IsDefault | Payment methods |
| **InvoicePayment** | Id, InvoiceId, PaymentId, Amount, AppliedDate | Payment-to-invoice applications |
| **Refund** | Id, AccountId, PaymentId, Amount, Status, Type, RefundDate, ReasonCode | Refunds |
| **RefundInvoicePayment** | Id, RefundId, InvoicePaymentId, RefundAmount | Refund application details |
| **PaymentTransactionLog** | Id, PaymentId, TransactionDate, Amount, GatewayState, TransactionType | Payment gateway transaction logs |
| **CreditBalanceAdjustment** | Id, AccountId, Amount, ReasonCode, SourceTransactionId, Type, Status | Credit balance adjustments |

### Credit & Debit Memo Objects (Invoice Settlement)

| Object | Key Fields | Description |
|--------|-----------|-------------|
| **CreditMemo** | Id, AccountId, Amount, Balance, Status, MemoDate, TargetDate, ReasonCode, InvoiceId | Credit memos |
| **CreditMemoItem** | Id, CreditMemoId, Amount, Description, ServiceStartDate, ServiceEndDate, InvoiceItemId, SKU | Credit memo line items |
| **DebitMemo** | Id, AccountId, Amount, Balance, Status, MemoDate, TargetDate, ReasonCode | Debit memos |
| **DebitMemoItem** | Id, DebitMemoId, Amount, Description, ServiceStartDate, ServiceEndDate, InvoiceItemId, SKU | Debit memo line items |

### Revenue Recognition Objects

| Object | Key Fields | Description |
|--------|-----------|-------------|
| **AccountingCode** | Id, Name, Type, Status, Category | Accounting codes |
| **AccountingPeriod** | Id, Name, StartDate, EndDate, Status, FiscalYear | Accounting periods |

### Other Objects

| Object | Key Fields | Description |
|--------|-----------|-------------|
| **CommunicationProfile** | Id, ProfileName, Description | Communication profiles |
| **Export** | Id, Name, Status, FileId, CreatedDate | Data export records |
| **Import** | Id, Name, Status, ImportType, CreatedDate | Data import records |
| **Usage** | Id, AccountId, SubscriptionId, ChargeId, Quantity, StartDateTime, EndDateTime, UOM, SubmissionDateTime | Usage records |

---

## Export Data Sources

These data sources are available for workflow **Export** tasks. Export data sources support **joins across related objects**, making them much more powerful than Query tasks for cross-object data retrieval.

### Data Sources with Joinable Objects

| Data Source (Base Object) | Joinable Objects |
|--------------------------|-----------------|
| **Account** | BillToContact, DefaultPaymentMethod, Entity, ParentAccount, SoldToContact |
| **AccountingCode** | Entity |
| **ApplicationGroup** | Entity, Payment, PaymentMethod, PaymentMethodSnapshot, Refund |
| **BillingRun** | Entity |
| **BillingTransaction** | Amendment, BillTo, DefaultPaymentMethod, DiscountAppliedToRatePlanCharge, Entity, ParentAccount, Product, ProductRatePlan, ProductRatePlanCharge, RatePlan, Subscription |
| **Contact** | Entity |
| **CreditBalanceAdjustment** | Account, AccountReceivableAccountingCode, AccountingPeriod, BillToContact, DefaultPaymentMethod, Entity, Invoice, JournalEntry, JournalRun, ParentAccount, Payment, PaymentMethod, PaymentMethodSnapshot, Refund, SoldToContact |
| **CreditMemo** | Account, BillToContact, DefaultPaymentMethod, Entity, Invoice, ParentAccount, SoldToContact |
| **CreditMemoApplication** | Account, ApplicationGroup, BillToContact, CreditMemo, DefaultPaymentMethod, Entity, Invoice, ParentAccount, SoldToContact |
| **CreditMemoApplicationItem** | Account, AccountingPeriod, ApplicationGroup, BillToContact, CreditMemo, CreditMemoItem, CreditTaxationItem, DebitMemo, DebitMemoItem, DefaultPaymentMethod, Entity, Invoice, InvoiceItem, JournalEntry, JournalRun, ParentAccount, SoldToContact, TaxationItem |
| **CreditMemoItem** | Account, AccountReceivableAccountingCode, AccountingPeriod, Amendment, BillToContact, CreditMemo, DefaultPaymentMethod, Entity, Invoice, InvoiceItem, JournalEntry, JournalRun, ParentAccount, Product, ProductRatePlan, ProductRatePlanCharge, RatePlan, RecognizedRevenueAccountingCode, SoldToContact, Subscription |
| **DebitMemo** | Account, BillToContact, DefaultPaymentMethod, Entity, Invoice, ParentAccount, SoldToContact |
| **DebitMemoItem** | Account, AccountReceivableAccountingCode, AccountingPeriod, Amendment, BillToContact, DebitMemo, DefaultPaymentMethod, Entity, Invoice, InvoiceItem, JournalEntry, JournalRun, ParentAccount, Product, ProductRatePlan, ProductRatePlanCharge, RatePlan, RecognizedRevenueAccountingCode, SoldToContact, Subscription |
| **Invoice** | Account, BillToContact, BillToContactSnapshot, DefaultPaymentMethod, Entity, ParentAccount, SoldToContact, SoldToContactSnapshot |
| **InvoiceItem** | Account, AccountReceivableAccountingCode, AccountingPeriod, Amendment, BillToContact, BillToContactSnapshot, DefaultPaymentMethod, Entity, Invoice, JournalEntry, JournalRun, ParentAccount, Product, ProductRatePlan, ProductRatePlanCharge, RatePlan, RecognizedRevenueAccountingCode, SoldToContact, SoldToContactSnapshot, Subscription |
| **Subscription** | Account, BillToContact, DefaultPaymentMethod, Entity, ParentAccount, SoldToContact, SubscriptionVersionAmendment |
| **TaxationItem** | Account, AccountReceivableAccountingCode, AccountingPeriod, Amendment, BillToContact, DefaultPaymentMethod, Entity, Invoice, InvoiceItem, JournalEntry, JournalRun, ParentAccount, Product, ProductRatePlan, ProductRatePlanCharge, RatePlan, SoldToContact, Subscription, TaxableItemSnapshot |
| **ARTransaction** | Account, Credit, AccountingCode, CreditMemo, CreditMemoItem, CreditTaxationItem, DebitAccountingCode, DebitMemo, DebitMemoItem, DebitTaxationItem, DefaultPaymentMethod, Invoice, InvoiceItem, Payment, PaymentMethod, Product, ProductRatePlanCharge, RatePlan, RatePlanCharge, Refund, TaxationItem |

### Export vs Query: When to Use Which

| Feature | Query Task | Export Task |
|---------|-----------|-------------|
| Cross-object joins | No (single object only) | Yes (joins related objects) |
| Result storage | `Data.<Object>` array | CSV file (requires Iterate) |
| Date arithmetic | No | Yes (`today - 30 days`) |
| ORDER BY | No | Yes |
| Best for | Simple lookups by ID | Complex multi-object reports |
| Performance | Fast (single object) | Slower (produces file) |
| Use case | "Get account by ID" | "Find all overdue invoices with account and contact info" |

---

## REST API Queryable Objects

These objects can be queried via the REST API (used in Callout tasks or the `query_objects` MCP tool). They use filter syntax rather than ZOQL.

| Object Type | Filter & Sort Fields | Expandable Relations |
|------------|---------------------|---------------------|
| **Accounts** | id, accountnumber, balance, batch, billcycleday, billtoid, communicationprofileid, crmid, currency, defaultpaymentmethodid, invoicetemplateid, lastinvoicedate, name, parentid, shiptoid, soldtoid, status, updateddate, custom fields | billto, soldto, defaultpaymentmethod, subscriptions |
| **Amendments** | Standard fields | subscription |
| **BillingRuns** | Standard fields | — |
| **Contacts** | Standard fields | account |
| **CreditMemos** | Standard fields | account, items, taxationitems |
| **CreditMemoItems** | Standard fields | creditmemo, taxationitems |
| **CreditMemoApplications** | Standard fields | creditmemo |
| **DebitMemos** | Standard fields | account, items, taxationitems |
| **DebitMemoItems** | Standard fields | debitmemo, taxationitems |
| **Invoices** | Standard fields | account, items |
| **InvoiceItems** | Standard fields | invoice, taxationitems |
| **Orderss** | Standard fields | orderactions |
| **OrderActions** | Standard fields | — |
| **OrderLineItems** | Standard fields | — |
| **Payments** | Standard fields | account, paymentapplications |
| **PaymentApplications** | Standard fields | payment |
| **PaymentMethods** | Standard fields | account |
| **PaymentMethodSnapshots** | Standard fields | — |
| **PaymentRuns** | Standard fields | — |
| **PaymentSchedules** | Standard fields | paymentscheduleItems |
| **PaymentScheduleItems** | Standard fields | paymentschedule |
| **Products** | Standard fields | productrateplans |
| **ProductRatePlans** | Standard fields | product, productrateplancharges |
| **ProductRatePlanCharges** | Standard fields | productrateplan, productrateplanchargeTiers |
| **ProductRatePlanChargeTiers** | Standard fields | productrateplancharge |
| **RatePlans** | Standard fields | subscription, rateplancharges |
| **RatePlanCharges** | Standard fields | rateplan, rateplanchargetiers |
| **RatePlanChargeTiers** | Standard fields | rateplancharge |
| **Refunds** | Standard fields | account, refundapplications |
| **RefundApplications** | Standard fields | refund |
| **Subscriptions** | Standard fields | account, rateplans |
| **TaxationItems** | Standard fields | invoiceitem |
| **Usages** | Standard fields | account, subscription |
| **Commitments** | Standard fields | — |
| **CommitmentPeriods** | Standard fields | commitment |
| **PrepaidBalances** | Standard fields | — |
| **PrepaidBalanceFunds** | Standard fields | — |
| **PrepaidBalanceTransactions** | Standard fields | — |
| **Ramps** | Standard fields | — |
| **InvoiceSchedules** | Standard fields | — |
| **SummaryStatements** | Standard fields | — |
| **SummaryStatementRuns** | Standard fields | — |

### REST API Filter Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `EQ` | Equals | `status.EQ:Active` |
| `NE` | Not equal | `currency.NE:CAN` |
| `LT` | Less than | `balance.LT:100` |
| `GT` | Greater than | `balance.GT:0` |
| `LE` | Less than or equal | `balance.LE:500` |
| `GE` | Greater than or equal | `balance.GE:100` |
| `SW` | Starts with | `name.SW:Acme` |
| `IN` | In list | `status.IN:[Active,Draft]` |

---

## Object Relationships

Understanding how objects relate to each other is essential for building effective workflow queries.

```
Account
├── Contact (BillTo, SoldTo)
├── PaymentMethod (Default)
├── Subscription
│   ├── Amendment
│   ├── RatePlan
│   │   ├── RatePlanCharge
│   │   │   └── RatePlanChargeTier
│   │   └── ProductRatePlan (reference)
│   │       ├── ProductRatePlanCharge
│   │       │   └── ProductRatePlanChargeTier
│   │       └── Product
│   └── Usage
├── Invoice
│   ├── InvoiceItem
│   │   └── TaxationItem
│   ├── InvoiceAdjustment
│   ├── InvoiceItemAdjustment
│   └── InvoicePayment
├── Payment
│   └── RefundInvoicePayment
├── Refund
├── CreditMemo
│   └── CreditMemoItem
├── DebitMemo
│   └── DebitMemoItem
└── CreditBalanceAdjustment
```

---

## Common Query Patterns

### Query Task Examples

**Find active subscriptions for an account:**
```
Object: Subscription
Where: AccountId = '{{Data.Account[0].Id}}' AND Status = 'Active'
Fields: Id, Name, SubscriptionNumber, TermStartDate, TermEndDate
```

**Get unpaid invoices:**
```
Object: Invoice
Where: AccountId = '{{Data.Workflow.AccountId}}' AND Balance > 0 AND Status = 'Posted'
Fields: Id, InvoiceNumber, Amount, Balance, DueDate
```

**Find a rate plan charge by subscription:**
```
Object: RatePlanCharge
Where: RatePlanId = '{{Data.RatePlan[0].Id}}'
Fields: Id, Name, ChargeType, Price, Quantity, MRR, ChargedThroughDate
```

### Export Task Examples

**Overdue invoices with account and contact details:**
```
Base Object: InvoiceItem
Joined: Account (Id, Name, AccountNumber, Currency), Invoice (Id, Balance, DueDate, InvoiceNumber, Status), BillToContact (FirstName, LastName, WorkEmail)
Where: Invoice.DueDate <= 'today - 30 days' AND Invoice.Balance > 0 AND Invoice.Status = 'Posted'
```

**Subscription charges with product info:**
```
Base Object: BillingTransaction
Joined: Account (Id, Name), Subscription (Id, Status), Product (Id, Name), ProductRatePlanCharge (ChargeType), RatePlan (Name)
Where: Subscription.Status = 'Active'
```

**Credit memos with invoice details:**
```
Base Object: CreditMemoItem
Joined: Account (Id, Name), CreditMemo (Id, Status, Amount), Invoice (InvoiceNumber), Product (Name)
Where: CreditMemo.Status = 'Posted'
```

### REST API Query Examples (for Callout tasks or query_objects)

**Accounts with balance > 0:**
```
objectType: Accounts
filter: ["balance.GT:0", "status.EQ:Active"]
fields: ["id", "accountnumber", "name", "balance", "currency"]
expand: ["billto"]
```

**Recent payments:**
```
objectType: Payments
filter: ["status.EQ:Processed"]
sort: ["updateddate.DESC"]
pageSize: 50
```

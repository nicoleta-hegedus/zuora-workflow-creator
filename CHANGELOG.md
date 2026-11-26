# Changelog

## v1.1.0 — 2026-11-26

### Added
- JSONata version rule: every new `Logic::JSONTransform` task with `processor: "JSONata"` must carry `"version"` in its `parameters`, set to the latest version Zuora offers (`"2.2.2"` as of 2026-11-26)
- New step "Verify Field Writability Before Designing the Action" (the process is now 8 steps)
- CSV File Upload Input Parameters (File-Field) section: parameter definition, consuming the uploaded file, guarding against bad rows, debugging an upload
- Bulk Update Pattern (50 records per callout)
- Guidance to ask the user for a working export when a mechanism is unverified in the tenant
- Common pattern: CSV-driven mass update

### Changed
- Input parameter `index` is zero-based (first parameter is `"0"`)
- Important Notes extended: File-Field parameters, field writability, `rest_endpoint` path, concurrent limits, export CSV filenames, empty results

## v1.0.0 — 2026-06-02

### Added
- Initial release of zuora-workflow-creator as a marketplace plugin
- SKILL.md with full workflow creation guidance (7-step process)
- Reference files: task-types.md, data-sources.md, liquid-reference.md, jsonata-reference.md
- Covers all task types: Query, Export, Callout, Email, Iterate, JSONTransform
- Common workflow patterns: dunning, subscription management, late fees, data sync, chargeback handling
- Trigger configuration: on-demand, scheduled, callout, event-based

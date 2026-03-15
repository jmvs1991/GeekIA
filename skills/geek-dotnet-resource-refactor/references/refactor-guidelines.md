# Refactor Guidelines

## Goal

Turn numeric or opaque localization identifiers into descriptive, stable, domain-facing names that communicate intent, while avoiding unnecessary structural churn.

## Recommended review checklist

- Does each property name explain the business scenario?
- Does each key name match the property name closely enough to be searchable?
- Is the English message clear without extra context?
- Is the Spanish message natural and equally specific?
- Is there any accidental behavior change caused by inheritance or key reuse?
- Is there a generic not-found message and, separately, a more specific lookup message when needed?
- Does the class actually benefit from migration to `ResourceBase`, or is text and naming cleanup enough?

## Necessity-first review

Before suggesting structural changes, classify each change:

- **necessary**: real problem or misleading design
- **recommended**: solid improvement with manageable impact
- **optional**: nice-to-have stylistic cleanup
- **not recommended**: creates churn without clear value

Ask these questions before migrating:

- Is there duplicated `_culture` or shared-message logic that a base class already solves?
- Does the project already depend on a shared contract like `notFoundMsg`?
- Will migration reduce maintenance burden in a meaningful way?
- Would the same outcome be achieved with lighter edits only?

If the answer is mostly no, avoid migration.

## Key mapping pattern

Prefer mappings like these when the old names are genuinely unclear:

- `ClientMsj0001` -> `NotFoundByCodeMsg`
- `ClientMsj0002` -> `NotFoundByDocumentMsg`
- `ClientMsj0003` -> `AlreadyExistsMsg`

Avoid renaming already-clear keys just to normalize style.

## Message improvement examples

### Too vague

- `Client error`
- `Client not found`

### Better

- `Client not found for the provided code.`
- `Client not found for the provided document.`
- `A client with the provided document already exists.`

## Bilingual guidance

Use English as the source language and write the Spanish variant as native product text.

Examples:

- EN: `Client not found for the provided code.`
- ES: `No se encontró el cliente para el código proporcionado.`

- EN: `A client with the provided document already exists.`
- ES: `Ya existe un cliente con el documento proporcionado.`

## Migration caution

If a refactor introduces `ResourceBase`, check whether the base class already defines shared concepts such as `_culture` or `notFoundMsg`. Do not duplicate or shadow them unintentionally.

Do not add a generic override like `notFoundMsg` unless the domain really needs that generic concept. A specific lookup message should not be forced into a generic slot unless that is how the project already models errors.

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
- Does the class actually benefit from migration to an inherited Geek resource type, or is text and naming cleanup enough?
- Does inheritance already define message members or message codes so that only `.resx` text entries need to be added?

## Necessity-first review

Before suggesting structural changes, classify each change:

- **necessary**: real problem or misleading design
- **recommended**: solid improvement with manageable impact
- **optional**: nice-to-have stylistic cleanup
- **not recommended**: creates churn without clear value

Ask these questions before migrating:

- Is there duplicated `_culture` or shared-message logic that an inherited class already solves?
- Does the project already depend on a shared contract like `notFoundMsg`?
- Is the current inheritance missing or incorrect for the route scope?
- Will migration reduce maintenance burden in a meaningful way?
- Would the same outcome be achieved with lighter edits only?

If the answer is mostly no, avoid migration.

## Inheritance-first caution

If the resource uses a Geek inheritance family, inspect what is already inherited before adding members.

Common safe rule:

- if inheritance already supplies the message member or message code contract, do **not** regenerate it in the class
- instead, add the missing localized text entries in the resource files and keep the class minimal

Ask the user which scope applies only when it cannot be inferred safely:

- `base`
- `corporation`
- `corporation + company`
- `corporation + company + code`

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

If a refactor introduces `ResourceBase` or a scoped Geek resource inheritance, check whether the base class already defines shared concepts such as `_culture`, `notFoundMsg`, corporation/company/code-aware members, or message contracts. Do not duplicate or shadow them unintentionally.

Do not add a generic override like `notFoundMsg` unless the domain really needs that generic concept. A specific lookup message should not be forced into a generic slot unless that is how the project already models errors.

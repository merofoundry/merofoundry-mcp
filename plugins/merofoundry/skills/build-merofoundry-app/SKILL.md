---
name: build-merofoundry-app
description: Use when building, extending or publishing an application on MeroFoundry through its MCP tools, including creating data models, pages, forms, rules, consumer authentication and per-record access.
---

# Building an application on MeroFoundry

The plugin has already connected you to the hosted MeroFoundry MCP server. You do not need
to add a server or run any CLI. Call `whoami` first to confirm the grant and learn which
application your credentials are scoped to.

Work in this order. Each step has a gotcha that silently costs time if skipped.

## 1. Create the application

`create_application` takes a name and a slug. A slug is lowercase letters, digits and single
hyphens. **Underscores are rejected.** The same rule applies to page slugs later.

## 2. Create models, then PUBLISH them

`create_model` requires `label` and `pluralLabel` as well as `name`. A model is created in
`draft` schema state.

`add_field` takes `name`, `label` and `type`. A default value goes in the top-level
`defaultValue` argument, not inside `settings`.

**Then call `publish_model`.** This is the single most common failure. A draft model refuses
every record write:

```
422  Model '<name>' schema_state is 'draft'; publish required before writing records
```

Publish after the fields exist, and publish again after you add or change fields.

## 3. Create records

Record routes are scoped to an application. When calling the REST API directly, record
routes need `?application_id=<id>`; the MCP tools supply it for you.

Seed data you create as the builder is **not** equivalent to data created through the
application. A builder-issued create runs rules with your platform user id as the actor, not
an application end user's identity record id, so any creator-grant or owner-stamping rule
will not produce a usable grant on seeded rows. Create a few rows through the app's own form
if you need to exercise ownership.

## 4. Build pages from the block catalog

Read the published block catalog before writing a page. Do not invent block types; an
unrecognised type is not a usable page. The catalog is at
`https://docs.merofoundry.app/page-builder-blocks`.

Three rendering rules that are not obvious:

- **Images:** page text content is HTML-sanitized on write, and `<img>` tags are stripped.
  Use an image block. After any page write, read the page back and check the text you sent
  is still there.
- **Interpolation:** `{{...}}` resolves in a text block's content and a button block's
  label. It does **not** resolve in `href` or in `expr:`. A form's success redirect is the
  exception and uses a **single** brace: `{record.id}`.
- **Typed values:** a boolean field arrives as a real JSON boolean and a numeric field as a
  number, so a truthiness test such as `show_when: "expr:row.someFlag"` evaluates correctly.
- **Visibility expressions** (`show_when: "expr:..."`) support four forms only: a dotted
  path's truthiness, `!path`, `pathA == pathB`, and `pathA != pathB`. There is no `&&`, no
  `||`, no ordering comparison, no function call and no string literal. Nested `show_when`
  is the only conjunction, and there is no disjunction, so a multi-select filter costs one
  block copy per combination. Design around this rather than fighting it.

## 5. Create a model-bound form explicitly

A page that collects input needs a form. Create it with `create_form` and bind it to the
model; do not assume a page input block writes anything on its own. A standalone `field`
block is a non-interactive placeholder.

`settings.access` is **required**. A form created without it is rejected, so choose
deliberately: `"public"` allows anonymous submits, `"authenticated"` requires a consumer
session.

Be aware that a form's layout is **not** a server-side field allowlist: a field that exists
on the model is accepted and written even if the form does not display it. If a field must
never be set from a form, enforce it with a rule, not by omitting it from the layout.

## 6. Consumer access and per-record visibility

Two model settings govern the consumer record routes and they do different jobs:

- `records_api`: `"off"` removes the consumer record routes entirely.
- `consumer_writes`: governs consumer writes, with `"owner"` as the default.

Either closes the consumer write path: `records_api: "off"` removes the routes, and
`consumer_writes: "off"` refuses consumer create, update and delete. If a consumer route
answers `404 "Model not found"`, that usually means `records_api` is off, **not** that your
schema is broken. Seed from the builder API instead.

For per-record access, follow the published recipe at
`https://docs.merofoundry.app/recipes/row-level-security`. The mechanism is a saved query
whose JOIN ON predicate carries the reserved operand `{"parameter": "current_user_id"}`,
which the server binds from the authenticated consumer. Do not substitute a client
parameter, a hidden form field, a template string or a literal; those are not boundaries.

Two query-layer constraints worth knowing before you design:

- Every model and field a consumer query references must be public, and aggregates are
  rejected.
- A join's `relationshipName` is mandatory, and an ON term must name the alias being joined.

## 7. Publish and verify in a browser

Publish the application, then **verify by rendering the page in a real browser**. MeroFoundry
pages are client-rendered single-page applications, so an HTTP status code from `curl` tells
you almost nothing: a page that returns 200 can still be blank or broken. Use
`capture_page_screenshot` for a published page, or drive a real browser.

Do not report success from an exit code. Look at the rendered page.

## Quotas

The hobby plan allows 10 models per application. A design that creates a model per entity
plus a model per permission relation exhausts that quickly; check `get_usage` before
assuming you have room.

## When something fails

- A `422` naming a field usually means a validation rule; read the message, it names the
  field and the constraint.
- A write that reports success while the value is unchanged: read the record back. Do not
  assume.
- `test_rule` simulates a subset of the engine's actions. For an action it cannot simulate
  it reports `not_simulated` and sets `partial: true`, which is **not** a failure and not
  evidence the action is unsupported. It raises "Unknown rule action" only for a type the
  engine genuinely does not have. Its `context` requires a `record` key.

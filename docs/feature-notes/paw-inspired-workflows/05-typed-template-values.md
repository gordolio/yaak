# Typed Template Values

## Problem

Most API clients treat dynamic values as strings. That is workable for URLs and headers, but awkward for JSON and GraphQL variables where a value may need to be a number, boolean, null, object, or array.

Paw's dynamic values are powerful but string-oriented. A future Yaak implementation could improve on this by allowing template functions to return typed JSON values when the surrounding field supports it.

## Desired Behavior

Template functions should be able to return:

- string
- number
- boolean
- null
- object
- array
- secret-wrapped values

When used in a text field, non-string values can stringify. When used as an entire JSON value, the native type should be preserved.

Example:

```json
{
  "globalUserId": "${[ env.globalUserId ]}",
  "limit": "${[ number(env.limit) ]}",
  "payload": "${[ request.json(path='$.data') ]}"
}
```

If a template expression occupies the full JSON value, Yaak can preserve the returned type:

```json
{
  "limit": "${[ env.limitNumber ]}"
}
```

If it is embedded inside a larger string, it becomes string interpolation:

```json
{
  "message": "User ${[ env.globalUserId ]}"
}
```

## Current Implementation Anchors

- Plugin args already use `JsonPrimitive` in places.
- Template function render methods currently commonly return `Promise<string | null>`.
- `EnvironmentVariable.value` is currently a string.
- Body values are stored as JSON-ish records for requests.

## Proposal

Introduce a typed template render result:

```ts
type TemplateRenderValue =
  | { type: "string"; value: string; secret?: boolean }
  | { type: "number"; value: number; secret?: boolean }
  | { type: "boolean"; value: boolean; secret?: boolean }
  | { type: "null"; value: null; secret?: boolean }
  | { type: "json"; value: unknown; secret?: boolean };
```

Template rendering should expose two modes:

- `renderText`: always returns string.
- `renderValue`: preserves non-string values when the field is value-aware.

## MVP

1. Keep current string template behavior unchanged.
2. Add an opt-in typed render path for JSON body values.
3. Let plugins declare an output type.
4. Preserve typed output only when a field consists of exactly one template expression.
5. Add user-visible preview showing both raw template and typed resolved value.

## Open Questions

- How should typed values be represented in synced files?
- Should environment variables have declared types, or should types be inferred from template output?
- How should GraphQL variable editors preserve typed dynamic values?
- Can existing plugins migrate without breaking compatibility?


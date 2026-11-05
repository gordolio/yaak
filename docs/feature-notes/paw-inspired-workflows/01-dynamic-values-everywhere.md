# Dynamic Values Everywhere

## Problem

Yaak supports template tags, but dynamic values should feel universal: any value field should be able to contain literal text, one or more dynamic values, or a typed dynamic expression.

Paw makes this feel natural. A cell can contain a secure value, a JWT generator, a reference to another environment value, or concatenated dynamic pieces. That makes complex API testing setups much easier to maintain.

## Desired Behavior

Every field that currently accepts a user-authored value should support template functions:

- Environment variable values.
- Environment variable names, where safe.
- Header names and values.
- URL path/query parameters.
- Request URL.
- Request body fields.
- Authentication configuration fields.
- Cookie values.
- Folder/workspace inherited settings that are value-like.

Dynamic values should be composable:

```text
Bearer ${[ jwt.sign(secret=env.SAPI_SIGNING_SECRET, payload=env.jwtPayload) ]}
```

Dynamic values should also be inspectable. A user should be able to click a rendered tag, see the plugin/function that produced it, and edit its configuration through a form.

The user-facing editor should not require typing raw template syntax like `${[ ... ]}`. See [Tokenized Dynamic Value Editor](./06-tokenized-dynamic-value-editor.md).

## Current Implementation Anchors

- Template functions already exist in plugins.
- Plugin runtime has `call_template_function_request` and `template_render_request`.
- `EnvironmentVariable.value` is a string today.
- Existing built-in template plugins include UUID, timestamp, hash, request, response, regex, fs, cookie, JSON, XML, random, encode, and 1Password.

## Proposal

Introduce a shared `TemplateValueInput` UI component and use it everywhere value-like input appears.

Capabilities:

- Plain text editing.
- Insert-template-tag menu.
- Token chips inline with text.
- Config popover for a selected tag.
- Preview/evaluate current value.
- Copy raw template or rendered value.
- Secret-safe rendering when any segment is secret.

Internally this can remain string-based at first:

```text
https://${[ env.host ]}/v1/users/${[ env.userId ]}
```

Later, typed template values can extend this. See [Typed Template Values](./05-typed-template-values.md).

## MVP

1. Allow environment variable values to contain template expressions.
2. Render environment variables through the same template engine used for request fields.
3. Detect cycles between environment variables and report a useful error.
4. Add a UI affordance to insert template tags into environment variable values.

## Open Questions

- Should environment variable values be rendered lazily per request, or eagerly in the environment editor preview?
- How should secret state propagate from one variable into another?
- How should template function errors be displayed in request send output versus environment editor preview?
- Should variable names be dynamic, or only values?

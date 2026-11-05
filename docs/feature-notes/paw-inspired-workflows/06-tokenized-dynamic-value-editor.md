# Tokenized Dynamic Value Editor

## Problem

Yaak's template syntax is powerful, but `${[ ... ]}` is an implementation detail. Users should not need to type or visually parse that syntax during normal request editing.

Paw's dynamic value editor treats dynamic values as semantic tokens. The user can start typing a trigger such as `[`, `{`, or `(`, choose a dynamic value from a dropdown, and the editor inserts a pill/tag. The field remains editable text, but dynamic values are represented as objects with labels, colors, configuration, and previews.

## Desired Behavior

Any value editor that supports template functions should offer:

- Trigger-based insertion by typing `[`, `{`, `(`, or another configurable prefix.
- Dropdown search across available dynamic values:
  - Environment variables.
  - Template functions.
  - Secrets.
  - Recent values.
  - Plugin-provided values.
- Inline pills/tags for dynamic segments.
- Click-to-edit configuration for a token.
- Keyboard navigation for selecting, moving through, deleting, and opening tokens.
- Tooltips/previews that show what a token resolves to without exposing secrets.
- Copy/paste support across fields.

Example visual representation:

```text
Bearer [JWT Sign] using [Default Group SAPI_SIGNING_SECRET]
```

The raw storage might still be:

```text
Bearer ${[ jwt.sign(secret=env.SAPI_SIGNING_SECRET) ]}
```

but the user should mostly interact with tokens, not syntax.

## Current Implementation Anchors

- Yaak already has template tag styling in the theme package.
- Template functions already expose names, arguments, and plugin metadata.
- Environment variables have enough metadata to produce labels like `Default Group baseUrl`.
- The plugin runtime can request template function config and invoke render behavior.

## Proposal

Build a shared `TemplateValueEditor` component backed by a structured token model.

Potential editor segment model:

```ts
type TemplateEditorSegment =
  | { type: "text"; value: string }
  | {
      type: "template";
      raw: string;
      kind: "environment" | "function" | "secret" | "plugin";
      label: string;
      detail?: string;
      color?: string;
      secret?: boolean;
      config?: Record<string, unknown>;
    };
```

The editor can parse existing raw strings into segments and serialize segments back to the current template syntax for storage.

This allows incremental adoption without changing the database model immediately.

## Insertion Flow

1. User types `[` in a value field.
2. Editor opens a dynamic value picker at the caret.
3. User searches/selects `Default Group baseUrl`.
4. Editor inserts a token segment.
5. On blur/save, the segment serializes to the existing template expression.

## Display Flow

1. Field loads raw stored text.
2. Parser identifies template expressions.
3. Known template expressions render as tokens.
4. Unknown/unparseable expressions remain raw text with warning styling.

## MVP

1. Implement tokenized rendering for environment variables and built-in template functions.
2. Support trigger insertion with keyboard search.
3. Preserve current raw template syntax in storage.
4. Provide a "copy raw template" command for power users.
5. Fall back gracefully when a plugin/template function is missing.

## Later

- Rich config popovers for each plugin-provided token.
- Token previews with secret masking.
- Drag/drop or keyboard movement of tokens within text.
- Multi-token selection.
- User-configurable trigger characters.
- Search ranking based on current workspace/environment.

## Open Questions

- Should the editor be based on CodeMirror, Lexical, ProseMirror, or a custom input?
- Should raw template syntax remain directly editable behind a toggle?
- How should malformed template syntax appear in token mode?
- How should typed template values display inside JSON editors?


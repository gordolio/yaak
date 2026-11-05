# Paw-Inspired API Testing Workflows

These notes capture workflow gaps noticed while comparing Yaak with Paw/RapidAPI for manual API testing.

The goal is not to copy Paw exactly. The goal is to preserve the parts that make complex API projects feel organized, composable, and low-friction:

- Project-local environment state.
- Multiple active environment axes, such as deployment plus user persona.
- Dynamic values that can be used anywhere a value appears.
- File-oriented workflows that make a project feel self-contained.
- A compact environment matrix UI for scanning and editing many values quickly.

## Current Yaak Concepts

Useful existing anchors:

- `Environment` already has `parentModel`, `parentId`, `public`, `color`, `sortPriority`, and `variables`.
- `EnvironmentVariable` is currently string-oriented: `{ name, value: string, enabled?, id? }`.
- Workspace sync already serializes models to local files.
- Template functions already exist through plugins.
- Plugins can add authentication, template functions, importers, filters, actions, and themes.

## Proposed Feature Notes

- [Dynamic Values Everywhere](./01-dynamic-values-everywhere.md)
- [File-Based Project Workflows](./02-file-based-project-workflows.md)
- [Multiple Environment Axes](./03-multiple-environment-axes.md)
- [Environment Matrix Editor](./04-environment-matrix-editor.md)
- [Typed Template Values](./05-typed-template-values.md)
- [Tokenized Dynamic Value Editor](./06-tokenized-dynamic-value-editor.md)

## Reference Workflow

A Paw project can keep one API testing project as a local document. Within that project, a user can define environment groups such as:

- Deployment: Local Dev, Staging, Staging02, Prod.
- Influencer: Gordon, Rahma, Nadia, Kelsey.
- Feature or scenario: default flags, experimental flags, regression case.

Each group has columns. Each row is a variable. A request can use the active column from each group at the same time.

This means a request can be evaluated against:

```text
Deployment = Staging
Influencer = Gordon
Feature = Default
```

without duplicating all variables into one giant environment. That compositional model is the main product gap these notes explore.

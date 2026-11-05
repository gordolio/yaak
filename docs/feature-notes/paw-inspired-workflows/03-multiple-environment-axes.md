# Multiple Environment Axes

## Problem

Yaak environments are currently selected as a single environment context. Paw supports multiple environment groups at the same time. This enables a user to combine independent axes, such as:

- Deployment: Local Dev, Staging, Prod.
- User Persona: Gordon, Rahma, Nadia.
- Feature Scenario: Default, Feature Flag On, Regression Case.

Without this, users must duplicate variables into many combined environments:

```text
staging-gordon-default
staging-gordon-feature-x
staging-rahma-default
prod-gordon-default
```

That scales poorly and makes manual testing brittle.

## Desired Behavior

Allow a workspace to define environment groups. Each group has one active environment at a time.

Example active stack:

```text
Deployment = Staging
Influencer = Gordon
Feature = Default
```

Variable resolution merges the selected environments in a deterministic order. Later axes can override earlier axes, or each group can be given an explicit priority.

## Current Implementation Anchors

- `Environment` already has `parentModel` and `parentId`.
- `Environment` already has `color` and `sortPriority`.
- Current environment variables are scoped to one environment model.
- UI route search includes `environment_id`, suggesting one active environment today.

## Proposal

Introduce `EnvironmentGroup` as a new model, or represent groups as environment parents with a `kind`.

Potential model:

```rust
struct EnvironmentGroup {
    id: String,
    workspace_id: String,
    name: String,
    sort_priority: f64,
    active_environment_id: Option<String>,
    color: Option<String>,
}
```

Environments would belong to a group:

```rust
struct Environment {
    group_id: Option<String>,
    // existing fields...
}
```

Request send/render context would accept multiple environment IDs:

```text
?environment_ids=env_deployment_staging,env_persona_gordon
```

The CLI should mirror this:

```sh
yaak send req_123 --environment deployment:staging --environment persona:gordon
```

or:

```sh
yaak send req_123 --environment-stack env_a,env_b
```

## MVP

1. Add an internal environment stack type.
2. Extend template resolution to merge variables from multiple selected environments.
3. Add a minimal UI for selecting multiple active environments.
4. Persist the active selection per workspace/window.
5. Extend CLI send to accept multiple environment IDs.

## Resolution Rules

Suggested deterministic order:

1. Workspace/global variables.
2. Selected environment groups in sidebar order.
3. Folder/request-level overrides, if applicable.
4. Explicit runtime variables, if added later.

When two active environments define the same variable, the later source wins. The UI should show the source of the resolved value.

## Open Questions

- Should groups be required, or can legacy environments live in an implicit Default group?
- Should two groups be allowed to define the same variable names?
- Should overrides warn when hidden by another selected environment?
- How should secrets merge across environment axes?


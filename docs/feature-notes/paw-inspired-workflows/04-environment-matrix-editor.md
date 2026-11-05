# Environment Matrix Editor

## Problem

Yaak's current environment editor is good for a single variable set, but it does not make multi-environment comparison easy. Paw's environment matrix lets users see variables as rows and environments as columns.

This is especially useful for API testing because missing values, wrong secrets, and persona-specific IDs become visually obvious.

## Desired Behavior

Add a matrix editor for a selected environment group:

```text
Variable             Local Dev        Staging        Prod
baseUrl              localhost        appdev01       app
m2mJwt               [JWT tag]        [JWT tag]      [JWT tag]
SAPI_SIGNING_SECRET  [secret]         [secret]       [secret]
campaignId           value            value          73007
```

For a persona group:

```text
Variable          gordon       rahma       nadia
globalUserId      13836813     12108169    18917761
creatorProfileId  [dynamic]    [dynamic]   [dynamic]
```

The user should be able to:

- Add/remove variables by row.
- Add/remove environments by column.
- Switch between groups.
- Use dynamic values in any cell.
- Mark cells as secret/private.
- See missing values and inherited values.
- Reorder environments and variables.

## Current Implementation Anchors

- `Environment.variables` is currently a list.
- `Environment.color` and `sortPriority` are already available.
- `EnvironmentVariable` has `enabled`, `name`, `value`, and `id`.

## Proposal

Add a matrix editor view that transforms the current model without requiring immediate schema changes:

- Rows are the union of variable names across environments in a group.
- Columns are environments in the group.
- Cells read/write an `EnvironmentVariable` entry on that environment.

This can ship before deeper model changes if the environment group concept exists.

## MVP

1. Add a matrix view for one environment group.
2. Support editing string values.
3. Support inserting existing template tags.
4. Support show/hide secret values.
5. Show whether a value is direct, missing, inherited, or overridden.

## Later

- Copy/paste rectangular cell ranges.
- Duplicate environment column.
- Bulk rename variables.
- Filter rows by name/missing/secret/dynamic.
- Import CSV/JSON into a group.
- Export group to CSV for auditing.

## Open Questions

- Should variable row order be stored separately from each environment's variable order?
- Should empty cells mean "missing" or "intentionally blank"?
- How should disabled variables display in a matrix cell?
- Should inherited values be editable from the child matrix, or only displayed?


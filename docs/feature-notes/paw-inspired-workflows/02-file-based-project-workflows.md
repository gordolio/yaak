# File-Based Project Workflows

## Problem

Yaak supports local directory sync, but the app does not yet feel like a document-based native API client. Paw's model is simple: a project is a file, and File -> Open / Open Recent / Import / Close map naturally to the user's mental model.

Yaak's workspace sync is good for Git, but users still need a clear "this folder is the project" workflow.

## Desired Behavior

Add first-class file/directory workflows:

- File -> New Workspace Directory
- File -> Open Workspace Directory
- File -> Open Recent
- File -> Import...
- File -> Close Workspace
- File -> Reveal in Finder
- File -> Save / Sync Now, if needed

Opening a workspace directory should feel like opening a project. The app should remember it in recent items and restore the last selected request/environment state for that directory.

## Current Implementation Anchors

- `WorkspaceMeta.setting_sync_dir` exists.
- Local directory/Git sync already serializes workspace models.
- Sync models include workspace, environment, folder, HTTP request, gRPC request, and websocket request.
- Yaak CLI has `--data-dir`, but workspace directories are separate from the app data directory.

## Proposal

Treat a synced workspace directory as a first-class openable project.

Possible project directory shape:

```text
my-api.yaak/
  yaak.workspace.yaml
  yaak.env.local.yaml
  yaak.env.staging.yaml
  requests/
  folders/
  .yaak/
    workspace-state.json
```

This does not need to replace the existing sync format. It can start as UX over the existing files and evolve only if the current format blocks project-like behavior.

## MVP

1. Add Open Workspace Directory command.
2. If the selected directory contains Yaak sync files, attach/open that workspace.
3. Add Open Recent for workspace directories.
4. Add Reveal in Finder for a workspace's sync directory.
5. Make Close Workspace remove it from the active UI without deleting files.

## Non-Goals

- Do not require every workspace to be file-backed.
- Do not make secrets plaintext by default.
- Do not remove the current database-backed local app model.

## Open Questions

- Should a workspace directory be self-describing through a manifest file?
- Should response history live in project files, app data, or optional `.yaak/history`?
- How should multiple app windows map to multiple workspace directories?
- How should Git conflict resolution be surfaced?


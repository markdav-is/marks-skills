---
name: release-versioning-tagging
description: |
  How to bump the version of an Oqtane module in this repo and create the corresponding
  ADO git tag. Use whenever a module is ready for deployment: after UAT issues are confirmed
  done, before creating the deployment PR or pushing a release tag.
author: Skiller
version: 1.0.0
date: 2026-03-02
---

# Oqtane Module Versioning and Tagging

## Problem

Each deployable Oqtane module needs a version bump before deployment, and a corresponding
git tag so releases are traceable in ADO. It is not obvious where the version lives or
what the tag format is.

## Context / Trigger Conditions

Activate this skill when the user says any of the following (exact or paraphrased):

**Version bumping**
- "roll the version", "bump the version", "increment the version"
- "update the version number", "version the module"

**Tagging**
- "push a tag", "create a tag", "tag the release", "tag for deployment"
- "cut a release", "tag it", "push the release tag"

**Release / deployment prep**
- "ready to deploy", "prepare for deployment", "deploy the module"
- "create a release", "release the module", "ship it", "let's ship"
- "make a release PR", "release branch"

**Combined** (most common — user will often say both at once)
- "roll the version and push a tag"
- "bump and tag", "version and tag"
- "let's do a release for {module}"

## Solution

### 1. Find the version file

Each module has a `ModuleInfo.cs` under `Client\Modules\{ModuleName}\`:

```csharp
public class ModuleInfo : IModule
{
    public ModuleDefinition ModuleDefinition => new ModuleDefinition
    {
        Name = "Surveyor",
        Version = "1.0.0",                   // ← current version
        ReleaseVersions = "1.0.0",           // ← comma-separated release history
        ...
    };
}
```

### 2. Determine the new version

Use semantic versioning:
- **Patch** (`x.x.1`) — bug fixes, small enhancements, no new features
- **Minor** (`x.1.0`) — new features, backward-compatible
- **Major** (`1.0.0`) — breaking changes or major rewrites

### 3. Copilot: create the release branch via ADO MCP

Use `ado_repo_create_branch` targeting the `PWApps` repository, sourced from `main`:

```
branchName:       release/{module}-v{version}
sourceBranchName: main
repositoryId:     91193c5e-68e0-4006-a2d6-ffbeb276b015
```

### 4. Copilot: update `ModuleInfo.cs`

Bump `Version` and **append** the new version to `ReleaseVersions` (Oqtane uses this
list to run incremental DB migrations in order — never remove old entries):

```csharp
Version = "1.0.1",
ReleaseVersions = "1.0.0,1.0.1",
```

### 5. Copilot: commit, push, tag via git CLI in terminal

ADO MCP has no commit or tag tools — use the terminal for these steps.
Run each command separately (no multi-line):

```powershell
git fetch origin release/{module}-v{version}
git checkout -b release/{module}-v{version} origin/release/{module}-v{version}
git add Client/Modules/{Module}/ModuleInfo.cs .github/skills/
git commit -m "chore: bump {Module} to v{version}"
git push -u origin release/{module}-v{version}
git tag {module}/v{version}
git push origin {module}/v{version}
```

### 6. Copilot: create the PR and set auto-complete via ADO MCP

Use `ado_repo_create_pull_request`, then immediately call `ado_repo_update_pull_request` to set auto-complete:

**Create:**
- **source**: `refs/heads/release/{module}-v{version}`
- **target**: `refs/heads/main`
- **title**: `{Module} v{version} — <summary of changes>`
- **description**: bullet list of issues closed and changes made
- **workItems**: IDs of UAT issues being released
- **labels**: `["{Module}"]`

**Auto-complete** (separate call with the returned `pullRequestId`):
- `autoComplete: true`
- `mergeStrategy: NoFastForward`
- `transitionWorkItems: true`
- `deleteSourceBranch: false`

## Verification

- `ModuleInfo.cs` has both `Version` and `ReleaseVersions` updated
- `ReleaseVersions` contains every prior version — do not remove old entries
- Branch created on origin via ADO MCP before file changes are committed
- Commit, push, and tag all done via git CLI terminal (Copilot-run)
- PR created via ADO MCP — no user steps needed

## Example

Session that produced this skill:

```
User: roll the version of the surveyor module so I can do a deployment and a tag

→ Found Client\Modules\Surveyor\ModuleInfo.cs at Version = "1.0.0"
→ Changes were patch-level (bug fixes + UX enhancements)
→ Bumped to 1.0.1, ReleaseVersions = "1.0.0,1.0.1"
→ User confirmed tag format: surveyor/v1.0.1
```

## Notes

- `ReleaseVersions` drives Oqtane's migration runner — **order matters**, always append.
- The tag module name is always **lowercase** regardless of how the module class or folder is cased.
- ADO MCP handles branch creation and PR. Git CLI (terminal) handles commit, push, and tag.
- No user steps required — Copilot owns the entire workflow.

## References

- `Client\Modules\Surveyor\ModuleInfo.cs`
- `Client\Modules\Bridges\ModuleInfo.cs` (reference for established pattern)
- Oqtane `IModule` / `ModuleDefinition` documentation

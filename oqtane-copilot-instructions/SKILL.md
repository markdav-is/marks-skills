---
name: oqtane-copilot-instructions
description: |
  Template and guidance for writing a high-quality .github/copilot-instructions.md
  for an Oqtane module development workspace. Use when setting up Copilot in a new
  Oqtane project, onboarding Copilot to an existing workspace, or when Copilot
  suggestions are missing Oqtane-specific context (wrong base classes, missing DI
  wiring, EF patterns instead of Dapper, etc.). Covers what sections to include,
  what Oqtane-specific constraints to encode, and a ready-to-adapt template.
author: markdav-is
version: 1.0.0
date: 2026-06-10
---

# Oqtane Copilot Instructions

## Problem

GitHub Copilot has no built-in knowledge of an Oqtane project's specific architecture,
PWApps conventions, or the local `.oqtane-ref/` framework clone. Without a
`.github/copilot-instructions.md`, Copilot makes poor suggestions:

- Injects `NavigationManager` instead of using `ModuleBase.NavigateUrl()`
- Uses EF Core migrations instead of raw SQL / Dapper
- Adds `ILogger<T>` to Blazor components instead of the inherited `logger`
- Creates monolithic components instead of the per-action-file layout
- Guesses at `ServiceBase`, `ModuleBase`, or `ModuleControllerBase` constructors
  instead of reading the local reference

## Context / Trigger Conditions

- Starting a new Oqtane module workspace and no `.github/copilot-instructions.md` exists
- Copilot suggestions consistently use the wrong base classes or DI patterns
- Onboarding a new developer whose Copilot isn't aware of project conventions
- Adding a new module to an existing workspace that already has copilot-instructions
  (check whether the template sections below are already present)

## Solution

Create `.github/copilot-instructions.md` in the repository root with the content below.
Customise the **Project** section and any `<placeholder>` values before saving.

### What Makes Oqtane Copilot Instructions Effective

1. **State the architecture up front** — Copilot needs to know the client/server
   split and which layer lives in which project before it can suggest correct code.

2. **Name every base class and where its members come from** — e.g.
   "Inherit `ModuleBase`; do not inject `ILogger<T>` — use the inherited `logger`."

3. **Reference the local framework clone** — tell Copilot to read
   `.oqtane-ref/` for any API shape it doesn't know rather than guessing.

4. **List the hard NOs** — short, unambiguous prohibitions prevent the most
   common wrong suggestions (EF migrations, Radzen, Localizer, `@onclick` on divs).

5. **Show the file layout** — a concrete directory tree stops Copilot from
   inventing its own folder structure for new modules.

6. **Keep it short and scannable** — Copilot includes the instructions in every
   prompt; long prose is expensive and degrades suggestion quality. Use tables
   and bullet lists; cut anything that doesn't affect code output.

---

## Template

Copy this into `.github/copilot-instructions.md` and fill in the `<placeholders>`:

```markdown
# Copilot Instructions — <ProjectName>

## Project

<One or two sentences: what this workspace is, who it's for, and the top-level tech
stack (e.g. "Oqtane 5.x Blazor CMS, .NET 8, SQL Server, hosted on IIS").>

## Architecture

Oqtane modular Blazor framework. Each module has four layers:

| Layer | Project | Base Class | Purpose |
|---|---|---|---|
| Client UI | `Client` | `ModuleBase` | Razor components rendered in the browser |
| Client HTTP | `Client/Services` | `ServiceBase` | Typed HTTP clients calling server endpoints |
| Server API | `Server/Controllers` | `ModuleControllerBase` | MVC controllers exposing REST endpoints |
| Server Data | `Server/Repository` | *(interface + class)* | Dapper/raw SQL data access |

## File Layout

```
Client/Modules/<ModuleName>/
  Index.razor          ← default action Oqtane loads
  Edit.razor           ← edit action
  Settings.razor       ← module settings (loaded by Oqtane admin panel)
  Services/
    <ModuleName>Service.cs

Server/Controllers/
  <ModuleName>Controller.cs

Server/Repository/
  I<ModuleName>Repository.cs
  <ModuleName>Repository.cs
```

Each user-facing action is a separate Razor file.

## Framework Reference

A shallow clone of the Oqtane framework lives at `.oqtane-ref/` in the repo root.
**Always read actual method signatures from `.oqtane-ref/` before using any
Oqtane API — never assume.** Key paths:

| What | Path |
|---|---|
| `ModuleBase` | `.oqtane-ref/Oqtane.Client/Modules/ModuleBase.cs` |
| `ServiceBase` | `.oqtane-ref/Oqtane.Client/Services/ServiceBase.cs` |
| Controller examples | `.oqtane-ref/Oqtane.Server/Controllers/` |
| Repository examples | `.oqtane-ref/Oqtane.Server/Repository/` |
| Shared models | `.oqtane-ref/Oqtane.Shared/Models/` |

## Client Components

- Inherit `ModuleBase`.
- Use `AddModuleMessage` / `ClearModuleMessage` for user feedback; use `logger`
  for logging — both are inherited from `ModuleBase`.
- Do **not** inject `ILogger<T>` into Blazor components.
- Use `NavigateUrl()` and `EditUrl()` from `ModuleBase` for navigation — do not
  inject `NavigationManager` in components unless unavoidable.

## Client Services

- Inherit `ServiceBase`.
- Read `ServiceBase.cs` for `CreateApiUrl` and HTTP helper signatures.

## Server Controllers

- Inherit `ModuleControllerBase`.
- Model route attributes and authorization policies on existing controllers in
  `.oqtane-ref/Oqtane.Server/Controllers/`.

## Server Repositories

- One interface + one implementation per module.
- Data access: **Dapper or raw SQL only** — no EF Core, no EF migrations, no
  navigation properties.
- Table definitions: SQL projects under `dbo/tables/`.
- Register interface + implementation in the module's server-side DI startup file.

## Logging

- Blazor components: use inherited `logger` from `ModuleBase`.
- Repositories and plain service classes: inject `ILogger<T>`.

## Hard Rules

- ❌ No EF Core migrations or navigation properties.
- ❌ No Radzen controls — use Bootstrap 5 and Oqtane/Blazor built-ins.
- ❌ No `IStringLocalizer` — use plain text strings.
- ❌ No `<div @onclick="...">` as interactive elements — use `<a>` or `<button>`.
- ❌ Never guess at Oqtane API shape — read `.oqtane-ref/` first.
```

---

## Checklist Before Saving

- [ ] `<ProjectName>` and the project description are filled in
- [ ] The file layout table matches the actual module folder names in use
- [ ] `.oqtane-ref/` exists at the repo root (or setup instructions are in README)
- [ ] Any project-specific constraints beyond the template are added (e.g. custom
      component libraries, auth policies, specific SQL conventions)
- [ ] File is saved at exactly `.github/copilot-instructions.md`

## Notes

- Keep the file under ~150 lines. Copilot injects it into every request; bloat
  hurts suggestion quality and costs tokens.
- Avoid prose that doesn't change code output (e.g. "Oqtane is a great platform").
  Every sentence should be an actionable constraint or a fact Copilot needs to
  produce correct code.
- After adding or editing the file, open a new Copilot chat and ask it to describe
  the repo's architecture — if its answer is wrong, the instructions need tuning.
- The framework reference section is the highest-value section. Copilot's Oqtane
  knowledge is outdated and partial; pointing it at `.oqtane-ref/` is what stops
  hallucinated API signatures.

## References

- Oqtane framework source: https://github.com/oqtane/oqtane.framework
- GitHub Copilot custom instructions docs: https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot
- Related skill: `oqtane-module-development` — architecture reference used in the template above

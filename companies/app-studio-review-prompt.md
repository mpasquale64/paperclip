# Review request: "App Studio" Paperclip company package

## Instructions for the reviewer

You are reviewing a company package for **Paperclip**, an open-source control plane for running companies made of AI agents. The package defines a small AI-agent company that will build an iOS app.

The owner is new to coding. Please:

- **Be critical.** Look for real problems, not reasons to approve. If something is fine, say so briefly and move on.
- **Separate verified facts from assumptions.** If you can't check how Paperclip behaves, say so instead of guessing. Where you can, check claims against the Paperclip source and docs linked below.
- **Explain findings in plain language** that a beginner can act on.

Format your answer like this:

1. **Findings**, most severe first. Label each one **blocker** (import fails or something unsafe), **should-fix**, or **nice-to-have**. For each finding, give the file, the problem, why it matters, and a concrete fix.
2. **Overall verdict**: is this package ready to import, ready after fixes, or does it need rework?
3. **Questions for the owner**: anything you need answered before you can judge.

## Background

### What the owner wants

- A startup that builds an **iOS app** first and a **macOS app** later.
- The **full org defined now**. Later-phase roles are imported but stay paused until they're needed.
- A structure a beginner can supervise.

### How the package is used

- The package follows the Agent Companies spec (`schema: agentcompanies/v1`) plus Paperclip's `.paperclip.yaml` extension.
- It is imported from GitHub through Paperclip's **Company settings → Import** page or `paperclipai company import`.
- The import page has a **"Start imported agents and routines paused"** option, on by default. After the import, the owner resumes only the phase 1 agents (CEO, CTO, App Engineer, QA).
- Package location: https://github.com/mpasquale64/paperclip/tree/app-studio-package/companies/app-studio

### Org chart

```
board (human owner)
 └── CEO                          what to build and in what order
      ├── Product Designer        UX + visual design (Apple HIG)
      └── CTO                     architecture, task breakdown, code review
           ├── App Engineer       Swift/SwiftUI, iOS now, macOS later
           ├── Backend Engineer   server code, only if needed (phase 3)
           ├── QA                 testing in Simulator/devices
           ├── Release Engineer   builds, signing, TestFlight, App Store; propose-only (phase 4)
           └── Security Engineer  security/privacy review (phase 5)
```

### Design decisions to challenge

Each decision below comes with its reasoning. Say whether you agree.

1. **One-sentence agent instructions.** Paperclip's hiring skill (`skills/paperclip-create-agent/`) says to keep role instructions to a short identity-and-responsibility paragraph. Its reasoning is that the harness, installed skills, and repo instructions already supply procedures. The package adds detail only for concrete requirements: Apple HIG for the designer, propose-only for the release engineer, and private vulnerability reporting for security.
   - **Known conflict:** Paperclip's `company-creator` skill says every agent should get workflow context (where work comes from, who to hand off to) plus an "execution contract". A dated plan, `doc/plans/2026-10-02-hiring-template-prompt-diff.md`, shows Paperclip deliberately moved away from execution contracts in hiring templates. The package follows the newer, shorter approach. Is that the right call for an imported company package?
2. **The Product Designer reports to the CEO, not the CTO.** The reasoning: the CEO owns *what* to build and the CTO owns *how*. Design is a "what" decision. This also keeps the CTO's direct reports down to 5.
3. **QA reports to the CTO, not to the engineers.** A checker shouldn't report to the people it checks.
4. **The Release Engineer is propose-only.** It is told to submit changes as pull requests or approval requests and never apply them. The package admits that instructions are a request, not a lock. The real control is withholding App Store Connect keys, signing certificates, and server credentials.
5. **The Security Engineer comes last.** It must report vulnerabilities privately. Paperclip's hiring skill says to confirm a confidential workflow exists before hiring a role that handles sensitive disclosures.
6. **One App Engineer covers iOS and macOS**, because SwiftUI code is largely shared. A second engineer is added later if needed.
7. **The Backend Engineer is optional.** Many iOS apps can use iCloud/CloudKit instead of a custom server.
8. **No adapter is specified.** The owner picks Claude Code, Codex, or another adapter per agent at import time. Paperclip's company-creator skill says to omit adapters unless there's a reason to set one.
9. **Skills are referenced by slug from Paperclip's built-in skills catalog**, not bundled in the package. The slugs are `task-planning`, `issue-triage`, `wireframe`, `design-critique`, `github-pr-workflow`, `doc-maintenance`, and `qa-acceptance`. All of them exist in `packages/skills-catalog/catalog/`. Expect import warnings that these skills are "not present in the package".
10. **`GH_TOKEN` is an optional secret input** for the CTO, App Engineer, Backend Engineer, and Release Engineer, which open pull requests.
11. **There are no starter tasks and no `goals` in `COMPANY.md`.** The product isn't defined yet, so the owner sets the goal after import.
12. **One project, "iOS App"**, owned by the CTO.

### What was and wasn't verified

**Checked by script:**
- all YAML frontmatter parses
- every `role` is in Paperclip's `AGENT_ROLES` enum
- every `reportsTo` resolves to an agent in the package
- every icon is in `AGENT_ICON_NAMES`
- every skill slug exists in the skills catalog
- the project owner resolves

**Not verified:**
- **No real import was run.** No Paperclip instance was available.
- Whether skills referenced by slug attach automatically once the owner installs them from the catalog after import. On import, the slugs are stored as an adapter "skill sync preference", and the preview warns about them.
- Whether omitting `budgetMonthlyCents`, which defaults to 0, means "no budget limit" or "zero budget".
- The importer defaults `adapterType` to `process` when `.paperclip.yaml` gives none. The package relies on the owner choosing an adapter in the import preview.
- New standard agents get `canCreateAgents: true` by default (`doc/agent-permission-defaults.md`). The package does not restrict hiring. It relies on board approval for new agents.

### Practical constraints the owner may not know

- iOS apps can only be built with Xcode, which only runs on macOS. Agents run on the machine that runs Paperclip, so building and testing needs Paperclip running on a Mac.
- TestFlight and App Store distribution need a paid Apple Developer Program membership.

## Questions to answer

1. Will this import cleanly? Are there any format errors against the spec or the importer?
2. Is the org sound for a beginner building an iOS app with AI agents? Does it have too many roles, too few, or the wrong ones?
3. Are the reporting lines right?
4. Are the instructions too thin? Is anything critical missing?
5. Are the security controls enough, especially for the Release Engineer? Should the package set permissions, such as limiting `canCreateAgents`?
6. Is anything in `README.md` wrong or misleading for a beginner?
7. Is there a simpler or better way to reach the same goal?

## Reference material

All links point to the Paperclip source this package was built against (commit `3bbc2104b65caf9f7763938a01260a5673e12f7a`):

- Package spec: [docs/companies/companies-spec.md](https://github.com/mpasquale64/paperclip/blob/3bbc2104b65caf9f7763938a01260a5673e12f7a/docs/companies/companies-spec.md)
- Import/export guide: [docs/guides/board-operator/importing-and-exporting.md](https://github.com/mpasquale64/paperclip/blob/3bbc2104b65caf9f7763938a01260a5673e12f7a/docs/guides/board-operator/importing-and-exporting.md)
- Org rules: [docs/guides/board-operator/org-structure.md](https://github.com/mpasquale64/paperclip/blob/3bbc2104b65caf9f7763938a01260a5673e12f7a/docs/guides/board-operator/org-structure.md) and [delegation.md](https://github.com/mpasquale64/paperclip/blob/3bbc2104b65caf9f7763938a01260a5673e12f7a/docs/guides/board-operator/delegation.md)
- Hiring skill: [skills/paperclip-create-agent/](https://github.com/mpasquale64/paperclip/tree/3bbc2104b65caf9f7763938a01260a5673e12f7a/skills/paperclip-create-agent)
- Company-creator skill: [.agents/skills/company-creator/SKILL.md](https://github.com/mpasquale64/paperclip/blob/3bbc2104b65caf9f7763938a01260a5673e12f7a/.agents/skills/company-creator/SKILL.md)
- Prompt-shortening plan: [doc/plans/2026-10-02-hiring-template-prompt-diff.md](https://github.com/mpasquale64/paperclip/blob/3bbc2104b65caf9f7763938a01260a5673e12f7a/doc/plans/2026-10-02-hiring-template-prompt-diff.md)
- Agent permission defaults: [doc/agent-permission-defaults.md](https://github.com/mpasquale64/paperclip/blob/3bbc2104b65caf9f7763938a01260a5673e12f7a/doc/agent-permission-defaults.md)
- Example teams: [packages/teams-catalog/catalog/](https://github.com/mpasquale64/paperclip/tree/3bbc2104b65caf9f7763938a01260a5673e12f7a/packages/teams-catalog/catalog)
- Skills catalog: [packages/skills-catalog/catalog/](https://github.com/mpasquale64/paperclip/tree/3bbc2104b65caf9f7763938a01260a5673e12f7a/packages/skills-catalog/catalog)
- Allowed roles and icons: [packages/shared/src/constants.ts](https://github.com/mpasquale64/paperclip/blob/3bbc2104b65caf9f7763938a01260a5673e12f7a/packages/shared/src/constants.ts)
- Importer: [server/src/services/company-portability.ts](https://github.com/mpasquale64/paperclip/blob/3bbc2104b65caf9f7763938a01260a5673e12f7a/server/src/services/company-portability.ts)

## Package files (full contents)

The complete contents of every file in `companies/app-studio/` follow.

### `COMPANY.md`

````markdown
---
name: App Studio
description: Small AI-agent software company that builds an iOS app, with a macOS app planned later.
slug: app-studio
schema: agentcompanies/v1
version: 0.1.0
---

App Studio builds an iOS app, and later a macOS version of it. The CEO decides what to build and in what order. The product designer shapes how the app looks and feels. The CTO leads engineering: an app engineer builds the app, QA tests it, and later roles cover server code, releases, and security.
````

### `.paperclip.yaml`

````yaml
schema: paperclip/v1
agents:
  ceo:
    icon: crown
    capabilities: Sets product direction and priorities, turns company goals into work, and coordinates the CTO and product designer.
  product-designer:
    icon: gem
    capabilities: Designs the app's user experience and visual design using Apple's Human Interface Guidelines, and reviews built screens.
  cto:
    icon: cpu
    capabilities: Owns architecture and technical direction, breaks work into engineering tasks, and reviews all code.
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
  app-engineer:
    icon: code
    capabilities: Builds and maintains the iOS app (and later the macOS app) in Swift and SwiftUI.
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
  backend-engineer:
    icon: database
    capabilities: Builds the server-side code the app depends on, such as accounts, data sync, and APIs.
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
  qa:
    icon: bug
    capabilities: Tests the app in the iOS Simulator and on devices, reproduces bugs, and reports findings with evidence.
  release-engineer:
    icon: rocket
    capabilities: Proposes build, code signing, TestFlight, App Store, and server changes for board approval.
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
  security-engineer:
    icon: shield
    capabilities: Reviews the app's security and privacy, including data storage, sign-in, network traffic, and App Store privacy requirements.
````

### `agents/ceo/AGENTS.md`

````markdown
---
name: CEO
slug: ceo
title: Chief Executive Officer
role: ceo
reportsTo: null
skills:
  - task-planning
  - issue-triage
---

You are the CEO. You own product direction and priorities for the company's iOS app, and later its macOS app, and you coordinate the CTO and the product designer.
````

### `agents/product-designer/AGENTS.md`

````markdown
---
name: Product Designer
slug: product-designer
title: Product Designer
role: designer
reportsTo: ceo
skills:
  - wireframe
  - design-critique
---

You are the Product Designer. You own the app's user experience and visual design, following Apple's Human Interface Guidelines, and you review built screens from the user's point of view.
````

### `agents/cto/AGENTS.md`

````markdown
---
name: CTO
slug: cto
title: Chief Technology Officer
role: cto
reportsTo: ceo
skills:
  - github-pr-workflow
  - task-planning
  - doc-maintenance
---

You are the CTO. You own the app's technical direction, architecture, engineering task breakdown, and code review.
````

### `agents/app-engineer/AGENTS.md`

````markdown
---
name: App Engineer
slug: app-engineer
title: iOS & macOS Engineer
role: engineer
reportsTo: cto
skills:
  - github-pr-workflow
  - doc-maintenance
---

You are the App Engineer. You own building and maintaining the iOS app in Swift and SwiftUI, and later its macOS version.
````

### `agents/backend-engineer/AGENTS.md`

````markdown
---
name: Backend Engineer
slug: backend-engineer
title: Backend Engineer
role: engineer
reportsTo: cto
skills:
  - github-pr-workflow
  - doc-maintenance
---

You are the Backend Engineer. You own the server-side code the app depends on, such as accounts, data sync, and APIs.
````

### `agents/qa/AGENTS.md`

````markdown
---
name: QA
slug: qa
title: QA Engineer
role: qa
reportsTo: cto
skills:
  - qa-acceptance
---

You are the QA Engineer. You own testing the app in the iOS Simulator and on devices before release, reproducing bugs, and reporting findings with evidence.
````

### `agents/release-engineer/AGENTS.md`

````markdown
---
name: Release Engineer
slug: release-engineer
title: Release & DevOps Engineer
role: devops
reportsTo: cto
skills:
  - github-pr-workflow
---

You are the Release Engineer. You own builds, code signing, TestFlight and App Store releases, and any servers the app runs on. You propose every change to build, signing, release, or server settings as a pull request or approval request for the board, and you never apply those changes yourself.
````

### `agents/security-engineer/AGENTS.md`

````markdown
---
name: Security Engineer
slug: security-engineer
title: Security Engineer
role: security
reportsTo: cto
---

You are the Security Engineer. You own security and privacy review for the app: how it stores data, handles sign-in, talks to servers, and meets App Store privacy requirements. Report suspected vulnerabilities privately to the board, never in normal task threads.
````

### `projects/ios-app/PROJECT.md`

````markdown
---
name: iOS App
slug: ios-app
description: Rolling backlog for designing, building, testing, and releasing the iOS app.
owner: cto
---

The company's main project. Product priorities from the CEO become engineering tasks here. The CTO triages and reviews, the App Engineer builds, and QA verifies before release.
````

### `README.md`

````markdown
# App Studio

A Paperclip company package for a small AI-agent software company that builds an iOS app, with a macOS app planned later.

It follows the [Agent Companies specification](https://agentcompanies.io/specification) and imports into [Paperclip](https://github.com/paperclipai/paperclip).

## Org chart

```
you (the board)
 └── CEO                          decides what to build and in what order
      ├── Product Designer        how the app looks and feels
      └── CTO                     how it's built; reviews all code
           ├── App Engineer       builds the iOS app (later the macOS app)
           ├── Backend Engineer   server code, only if the app needs it
           ├── QA                 tests the app before release
           ├── Release Engineer   builds, signing, TestFlight, App Store (proposes changes only)
           └── Security Engineer  security and privacy review
```

| Agent | Reports to | Skills | Turn on |
|---|---|---|---|
| CEO | board | task-planning, issue-triage | phase 1 |
| CTO | CEO | github-pr-workflow, task-planning, doc-maintenance | phase 1 |
| App Engineer | CTO | github-pr-workflow, doc-maintenance | phase 1 |
| QA | CTO | qa-acceptance | phase 1 |
| Product Designer | CEO | wireframe, design-critique | phase 2: once you build screens |
| Backend Engineer | CTO | github-pr-workflow, doc-maintenance | phase 3: only if the app needs a server |
| Release Engineer | CTO | github-pr-workflow | phase 4: when you ship to TestFlight |
| Security Engineer | CTO | none | phase 5: when real users or real data are involved |

There is one project, **iOS App**, owned by the CTO. Add a macOS project when that work starts.

## Before you start

- **You need a Mac.** iOS apps are built with Xcode, which only runs on macOS. The agents run on the computer that runs Paperclip, so the App Engineer, QA, and Product Designer need Paperclip running on a Mac with Xcode installed.
- **Shipping needs an Apple Developer account.** TestFlight and the App Store need a paid Apple Developer Program membership.

## Import it

### In the Paperclip web app

1. Open **Company settings → Import**.
2. Choose the GitHub source and paste the link to this folder.
3. Leave **"Start imported agents and routines paused"** checked. It is on by default.
4. In the preview, choose an adapter for each agent, such as Claude Code or Codex.
5. Apply the import.
6. In the **"Activate imported agents and routines"** panel, resume only the phase 1 agents: CEO, CTO, App Engineer, and QA.

### From the command line

```sh
# preview first; nothing changes
paperclipai company import <link-to-this-folder> --dry-run

# then import for real
paperclipai company import <link-to-this-folder> --target new
```

Add `--new-company-name "Your Name"` to use a different company name.

## After importing

1. **Set the company goal.** Every company needs one goal that all work traces back to. Make it specific, for example "Ship version 1 of the app to TestFlight by March."
2. **Set budgets** for the company and for each agent. Paperclip warns at 80% of a budget and pauses the agent at 100%.
3. **Install the skills.** The agents list skills from Paperclip's skills catalog. If the import preview warns that a skill "is not present in the package", install that skill from the Skills page. If an agent still lacks the skill, attach it from that agent's page.
4. **Connect the app's code repository** to the iOS App project.
5. **GitHub access (optional).** The CTO, App Engineer, Backend Engineer, and Release Engineer can use a `GH_TOKEN` secret to open pull requests.
6. **Keep new hires behind your approval.** Agents can request new hires, so leave board approval for new agents turned on.

## Safety notes

- **Release Engineer:** its instructions say to propose changes and never apply them itself. Instructions are a request, not a lock. The real lock is not giving it App Store Connect keys, signing certificates, or server logins. Let it open pull requests, and do the final step yourself.
- **Security Engineer:** before turning it on, decide where it can report vulnerabilities privately. Do not use normal task threads for that.
- Imported agents start with timer heartbeats off. They wake up when they are assigned work.
````

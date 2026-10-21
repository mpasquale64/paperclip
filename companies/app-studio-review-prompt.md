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
- The import page has a **"Start imported agents and routines paused"** option, on by default. After the import, the owner resumes only the phase 1 agents (CEO, CTO, App Engineer, QA). The CLI has no such option: it imports every agent active, and the README tells the owner to pause the later-phase agents by hand.
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
8. **Every agent declares an adapter in `.paperclip.yaml`.** Paperclip's company-creator skill says to omit adapters unless there's a reason to set one. There is one: with no adapter, the importer defaults `adapterType` to `process`. The web import page treats `process` as installed, so it preselects "Process" for every agent, and a `process` agent with no command imports fine and then fails every run. The CLI silently swaps `process` for `claude_local`. The mapping:
   - Claude Code, `claude-opus-5-5`: CEO (`effort: high`), Product Designer (high), App Engineer (high), Security Engineer (xhigh)
   - Claude Code, `claude-sonnet-5-5`: QA (medium)
   - Codex, `gpt-5.6-sol`: CTO (`modelReasoningEffort: xhigh`), Backend Engineer (high), Release Engineer (medium)

   Reasoning: Claude Code for Swift/SwiftUI and Apple HIG judgment. Codex for code review and backend/devops, so the CTO who reviews code is a different model family from the App Engineer who writes it. Lower effort where runs are frequent and mistakes cheap, higher where runs are rare and mistakes expensive. `gpt-5.6-sol` is Paperclip's own default Codex model, chosen to avoid "model not found" on the owner's Codex install. The owner can still override any agent in the import preview.
9. **Skills are referenced by slug from Paperclip's built-in skills catalog**, not bundled in the package. The slugs are `task-planning`, `issue-triage`, `wireframe`, `design-critique`, `github-pr-workflow`, `doc-maintenance`, and `qa-acceptance`. All of them exist in `packages/skills-catalog/catalog/`. Expect import warnings that these skills are "not present in the package".
10. **`GH_TOKEN` is an optional secret input** for the CTO, App Engineer, Backend Engineer, and Release Engineer, which open pull requests.
11. **One starter task and no `goals` in `COMPANY.md`.** `projects/ios-app/tasks/write-v1-brief/TASK.md` asks the CEO to get the app idea from the board, write the v1 brief as the task's plan document, and hand off to the CTO. The importer ignores `goals:` in `COMPANY.md`, so the owner sets the goal after import. Setting a goal wakes nobody, because import turns timer heartbeats off. The task imports into Backlog and the import itself wakes no one. The README tells the owner to move it to Todo once the CEO is resumed, which wakes the CEO.
12. **One project, "iOS App"**, owned by the CTO.
13. **Hiring is gated.** `.paperclip.yaml` sets `company.requireBoardApprovalForNewAgents: true`. Without it the flag defaults to `false` and new hires start active. Six agents set `permissions.canCreateAgents: false`. The CEO can always request hires, and the CTO keeps the default `true`, so only those two can ask, and every request waits for the board.

### What was and wasn't verified

**Checked by script:**
- `.paperclip.yaml` and all YAML frontmatter parse
- every `role` is in Paperclip's `AGENT_ROLES` enum
- every `reportsTo` resolves to an agent in the package
- every icon is in `AGENT_ICON_NAMES`
- every skill slug exists in the skills catalog (`design-critique` is under `optional/`; the other six are under `bundled/`)
- the project owner resolves
- no `budgetMonthlyCents`, `cwd`, secrets, or absolute paths in the package

**Checked by a test import** into a throwaway local Paperclip instance (version 2026.1012.0, the owner's installed version, which is not the same build as commit `3bbc2104b`):
- `paperclipai company import ./companies/app-studio --target new --dry-run`: 0 errors. Warnings: one "skill is not present in the package" per skill reference (14), plus "This package declares schemaVersion 5 and predates label, blocker, document, work product, monitor, attachment, embedded image, task timestamp, and parent link transfer; that task data imports only if the bundle carries it."
- A real import with `pauseAutomations: true`, the same as the web page's default: the company flag was `true`; all 8 agents were `paused` (reason `import`) with the declared adapter, model, and effort, no `cwd`, and timer heartbeats off; `canCreateAgents` was `true` for the CEO and CTO and `false` for the other six; the starter task arrived as `backlog`, assigned to the CEO, in the iOS App project.

**Resolved by reading the source:**
- With no adapter declared, `adapterType` defaults to `process` (see decision 8).
- `company.requireBoardApprovalForNewAgents` defaults to `false` when absent. Imported agents themselves are created `paused` or `idle`, never `pending_approval`, so the flag only gates later hires.
- `budgetMonthlyCents: 0` means no budget limit: the budget policy is created inactive and every enforcement check requires a budget above 0. The package sets no budgets, by the owner's choice. The README tells the owner to set them after import.
- An explicit `canCreateAgents: false` is honored. The CEO role can always request hires.
- Moving an assigned task from Backlog to Todo wakes the assignee (`issue_status_changed`). Import and resume do not wake agents.

**Not verified:**
- **No import into the owner's own Paperclip, and no agent run.** The test instance was thrown away after the check. The Backlog→Todo wake was confirmed by reading code, not by running it, because that would start a real agent run.
- Whether skills referenced by slug attach automatically once the owner installs them from the catalog after import. On import, the slugs are stored as an adapter "skill sync preference", and the preview warns about them.
- A GitHub-source import. The test used a local folder and an inline upload.

### Practical constraints the owner may not know

- iOS apps can only be built with Xcode, which only runs on macOS. The machine the agents run on must be a Mac with Xcode. By default that is the Paperclip host, and Paperclip also supports SSH execution targets.
- TestFlight and App Store distribution need a paid Apple Developer Program membership.

## Questions to answer

1. Will this import cleanly? Are there any format errors against the spec or the importer?
2. Is the org sound for a beginner building an iOS app with AI agents? Does it have too many roles, too few, or the wrong ones?
3. Are the reporting lines right?
4. Are the instructions too thin? Is anything critical missing?
5. Are the security controls enough, especially for the Release Engineer? Is board approval plus limiting `canCreateAgents` to the CEO and CTO the right hiring control?
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
license: MIT
---

App Studio builds an iOS app, and later a macOS version of it. The CEO decides what to build and in what order. The product designer shapes how the app looks and feels. The CTO leads engineering: an app engineer builds the app, QA tests it, and later roles cover server code, releases, and security.
````

### `.paperclip.yaml`

````yaml
schema: paperclip/v1
company:
  requireBoardApprovalForNewAgents: true
agents:
  ceo:
    icon: crown
    capabilities: Sets product direction and priorities, turns company goals into work, and coordinates the CTO and product designer.
    adapter:
      type: claude_local
      config:
        model: claude-opus-5-5
        effort: high
  product-designer:
    icon: gem
    capabilities: Designs the app's user experience and visual design using Apple's Human Interface Guidelines, and reviews built screens.
    adapter:
      type: claude_local
      config:
        model: claude-opus-5-5
        effort: high
    permissions:
      canCreateAgents: false
  cto:
    icon: cpu
    capabilities: Owns architecture and technical direction, breaks work into engineering tasks, and reviews all code.
    adapter:
      type: codex_local
      config:
        model: gpt-5.6-sol
        modelReasoningEffort: xhigh
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
  app-engineer:
    icon: code
    capabilities: Builds and maintains the iOS app (and later the macOS app) in Swift and SwiftUI.
    adapter:
      type: claude_local
      config:
        model: claude-opus-5-5
        effort: high
    permissions:
      canCreateAgents: false
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
  backend-engineer:
    icon: database
    capabilities: Builds the server-side code the app depends on, such as accounts, data sync, and APIs.
    adapter:
      type: codex_local
      config:
        model: gpt-5.6-sol
        modelReasoningEffort: high
    permissions:
      canCreateAgents: false
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
  qa:
    icon: bug
    capabilities: Tests the app in the iOS Simulator and on devices, reproduces bugs, and reports findings with evidence.
    adapter:
      type: claude_local
      config:
        model: claude-sonnet-5-5
        effort: medium
    permissions:
      canCreateAgents: false
  release-engineer:
    icon: rocket
    capabilities: Proposes build, code signing, TestFlight, App Store, and server changes for board approval.
    adapter:
      type: codex_local
      config:
        model: gpt-5.6-sol
        modelReasoningEffort: medium
    permissions:
      canCreateAgents: false
    inputs:
      env:
        GH_TOKEN:
          kind: secret
          requirement: optional
  security-engineer:
    icon: shield
    capabilities: Reviews the app's security and privacy, including data storage, sign-in, network traffic, and App Store privacy requirements.
    adapter:
      type: claude_local
      config:
        model: claude-opus-5-5
        effort: xhigh
    permissions:
      canCreateAgents: false
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

### `projects/ios-app/tasks/write-v1-brief/TASK.md`

````markdown
---
name: Write the v1 product brief
slug: write-v1-brief
assignee: ceo
project: ios-app
---

Ask the board what the app is and who it is for. Write the v1 product brief as this task's plan document: the problem, the target user, the smallest useful first version, and what is out of scope. When the board agrees with the brief, hand it to the CTO to break into engineering tasks.
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

The package also includes one starter task in that project, **"Write the v1 product brief"**, assigned to the CEO.

## Which AI runs each agent

Each agent runs on an adapter: the AI tool that does its work. This package sets one for every agent, so you know what you are paying for.

| Agent | Adapter | Model | Effort |
|---|---|---|---|
| CEO | Claude Code | `claude-opus-5-5` | high |
| Product Designer | Claude Code | `claude-opus-5-5` | high |
| CTO | Codex | `gpt-5.6-sol` | xhigh |
| App Engineer | Claude Code | `claude-opus-5-5` | high |
| Backend Engineer | Codex | `gpt-5.6-sol` | high |
| QA | Claude Code | `claude-sonnet-5-5` | medium |
| Release Engineer | Codex | `gpt-5.6-sol` | medium |
| Security Engineer | Claude Code | `claude-opus-5-5` | xhigh |

Why: Claude Code writes the Swift and SwiftUI code and makes the Apple design calls. Codex reviews code and handles backend and release work, so the CTO who reviews the code is a different AI from the App Engineer who wrote it. Effort is lower for agents that run often and make cheap mistakes, like QA. It is higher for agents that run rarely and make expensive mistakes, like Security. Higher effort costs more per run.

- You need both Claude Code and Codex installed and signed in on the machine the agents run on.
- `gpt-5.6-sol` is Paperclip's default Codex model. If your Codex offers a newer model, you can switch to it.
- You can change any agent's adapter, model, or effort in the import preview or later on the agent's page.

## Before you start

- **You need a Mac.** iOS apps are built with Xcode, which only runs on macOS. The machine the agents run on must be a Mac with Xcode. By default that is the computer that runs Paperclip, and Paperclip also supports SSH execution targets. The App Engineer, QA, and Product Designer need that Mac.
- **Shipping needs an Apple Developer account.** TestFlight and the App Store need a paid Apple Developer Program membership.

## Import it

### In the Paperclip web app

1. Open **Company settings → Import**.
2. Choose the GitHub source and paste the link to this folder.
3. Under **Target**, keep **"Create new organization"**. It is the default.
4. Leave **"Start imported agents and routines paused"** checked. It is on by default.
5. Check the preview. It preselects the adapters this package declares (see the table above). If one of them is not installed on your Paperclip, the page switches that agent to another installed adapter and shows an amber notice. You can change any agent's adapter there. Expect two kinds of warning: one per skill that "is not present in the package" (see "Install the skills" below), and one saying the package "declares schemaVersion 5". The second is about task extras, such as labels and attachments, that this package does not include, so you can ignore it.
6. Apply the import.
7. In the **"Activate imported agents and routines"** panel, resume only the phase 1 agents: CEO, CTO, App Engineer, and QA.

**If the preview shows 0 agents,** Paperclip could not list the files on GitHub, and it does not show that error. The usual cause is GitHub's limit on requests made without a login. Try again later. Or download this folder as a zip and use the **Local zip** source, or import a local copy with the command line below.

### From the command line

```sh
# preview first; nothing changes
paperclipai company import <link-to-this-folder> --dry-run

# then import for real
paperclipai company import <link-to-this-folder> --target new
```

Add `--new-company-name "Your Name"` to use a different company name. You can also pass the path to a local copy of this folder instead of a link.

The command line has no adapter picker. It uses the adapters this package declares. To change one, edit the agent's page after the import.

The command line also does not start agents paused. All eight agents arrive active. They still do nothing until they are given work, but to follow the phase plan, open each phase 2 to 5 agent's page in the web app and press **Pause**.

## After importing

1. **Set the company goal, then start the first task.** Every company needs one goal that all work traces back to. Make it specific, for example "Ship version 1 of the app to TestFlight by March." Setting the goal does not wake anyone. Imported agents have timer heartbeats turned off, so they only wake when work reaches them. To start, open the task **"Write the v1 product brief"** in the iOS App project. It arrives in the **Backlog**, and the import does not wake the CEO. With the CEO resumed, change the task's status from **Backlog** to **Todo**. That wakes the CEO, which will ask you what the app is and who it is for. If the CEO does not start, press **Run now** on the CEO's page.
2. **Set budgets** for the company and for each agent. A budget of 0, the default, means no limit. Paperclip warns at 80% of a budget and pauses the agent at 100%.
3. **Install the skills.** The agents list skills from Paperclip's skills catalog. Six of the seven are bundled skills. `design-critique` is in the catalog's optional section. If the import preview warns that a skill "is not present in the package", install that skill from the Skills page. If an agent still lacks the skill, attach it from that agent's page.
4. **Connect the app's code repository** to the iOS App project. This package does not set a working folder for any agent. Connect the repository here, or set each agent's working folder in the import preview.
5. **GitHub access (optional).** The CTO, App Engineer, Backend Engineer, and Release Engineer can use a `GH_TOKEN` secret to open pull requests. See the safety notes below before you create one.
6. **Check that new hires need your approval.** This package turns on **"Require board approval for new hires"**. You can confirm it under **Company settings → Hiring**. Only the CEO and the CTO can request new hires, and each request waits for your approval. The other six agents cannot request hires.

## Safety notes

- **Release Engineer:** its instructions say to propose changes and never apply them itself. Instructions are a request, not a lock. The real lock is not giving it App Store Connect keys, signing certificates, or server logins. Let it open pull requests, and do the final step yourself.
- **GitHub token:** if you give the Release Engineer (or any agent) a `GH_TOKEN`, make it a fine-grained token limited to the app's repository. Do not give it merge or admin rights. Turn on branch protection for your main branch, so only a human can merge.
- **Security Engineer:** before turning it on, decide where it can report vulnerabilities privately. Do not use normal task threads for that.
- Imported agents start with timer heartbeats off. They wake up when they are assigned work.
````

### `LICENSE`

````text
MIT License

Copyright (c) 2026 Michael Pasquale (mpasquale64)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
````

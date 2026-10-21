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

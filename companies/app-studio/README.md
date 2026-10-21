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

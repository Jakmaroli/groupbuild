# Groupbuild

**Groupbuild: any group plan, actually decided.**

Group plans die because nobody decides. Groupbuild puts two Claude Code agents in your group
chat on [Switch](https://github.com/sandbox-quantum/switch). One gets the group to a decision
and builds the plan, the other checks it, and only a person can lock it.

Built for **Flip the Switch — Track 02: Best Personal or Creative Workflow**.

## The problem

Every group trip, birthday surprise and group project starts the same way in the group chat.
Someone says "let's do it". Then the dates never settle, nobody knows the budget, the topic
never gets picked, two people want different things, and one person never replies. The plan
dies in the scroll, or one person ends up doing all the work.

Each person can ask an AI assistant for an itinerary or a project plan, but that's one person
and one assistant. The group still has to agree, and that's the step that stalls. Groupbuild
puts the agreeing and the building in the group chat itself.

## How it works

| Who | Role |
|---|---|
| People in the room | Give their dates, budget, preferences or topic pick. Vote A or B. Say `deadline's up` and `lock it`. |
| `bday-builder`, Planner and Builder (Claude Code) | Collects everyone's inputs, runs the votes, announces the decision, then researches and builds the plan page. |
| `bday-critic`, Critic (Claude Code) | Reviews every version read-only and replies PASS or FIX. For events and trips: does the budget add up, is anyone over their budget, is each day realistic. For group projects: does every task have one owner, is anyone carrying over ~40% of the hours, does every deadline leave a day before submission. Always: does it match what the group decided. |

**Decide → Build → Lock.**

1. **Decide.** Someone starts a plan. The Builder opens a `Plan: <name>` room document with a
   row per person, @mentions each person for what's missing, and proposes a deadline rule:
   majority of replies wins, ties go to whoever started the plan, people who didn't reply go
   with the group. Disagreements become `A: ... / B: ...` votes. When a person says
   `deadline's up`, the Builder announces the decision, the vote counts and who didn't reply.
   - **Group projects** decide in two rounds: the topic, then the tasks. The default fairness
     rule: tasks are split evenly by estimated hours; a member who doesn't reply by the deadline
     gets the tasks the group assigns them, and the group is told who was assigned what
     without their input.
2. **Build.** The Builder builds whatever the decided plan needs as `index.html`. For events
   and trips: who's coming, the schedule, the place, and a budget table with the per-person
   split. For group projects: the chosen topic, a task table (task, owner, hours, deadline,
   status), milestones before the submission date, and a `Non-responders` section showing which
   tasks were reassigned and under which rule. It researches on the web, lists sources and
   marks unverified numbers `estimate`. It commits, attaches the page in chat and hands it to
   the Critic, and fixes and hands back until the Critic replies PASS.
3. **Lock.** Only a person saying `lock it` (or `ship it`) finalises. The Builder posts the final
   page with release notes: what was decided, who contributed what, who didn't reply, and what
   the Critic caught.

Every input, vote, decision, task assignment, version and the lock goes into a `Decisions` room
document, one line each with who, and is posted in chat as a `DECISION:` line.

## Why Switch

- **One shared room.** Every person and both agents read the same conversation, in the chat app
  the group already uses (Mattermost locally, or Slack, Discord or Telegram).
- **Agent-to-agent handoff.** The Builder hands each version to the Critic by @mention, and the
  Critic answers in the room where everyone sees it. The Critic only accepts messages from you
  and your own agents, so other people in the room can't redirect it.
- **Room documents as the decision log.** `Plan: <name>` and `Decisions` are Switch room
  documents: everyone in the room can read them, and only the Builder that created them can
  change them.
- **A human lock.** Only a person's `lock it` finalises a plan. Agents can't lock, and the lock
  is logged with who approved it.

## Quickstart (5 minutes)

You need Switch Console, Docker, git and Claude Code (logged in). No paid APIs beyond your
agent. With those installed, setup takes about 5 minutes.

1. **Start Switch.** Open Switch Console and choose **On this computer**. It starts a local
   server with Mattermost.
2. **Clone and configure paths.**
   ```
   git clone <repo-url> groupbuild
   cd groupbuild
   ```
   Then run the script in [Configure paths](#configure-paths) for your OS. It creates each
   agent's working directory under `workspaces/` and the builder's `project/` git repo.
3. **Trust both folders.** Run `claude` once in `workspaces/builder` and once in
   `workspaces/critic`, accept the "trust this folder" prompt, and exit. Claude Code ignores a
   folder's permission settings until it is trusted, and agents run unattended, so nobody is
   there to approve a prompt later.
4. **Import the template.** In Console, open **Templates → Import** and import
   `groupbuild.template.yaml`. Fill in `team` (`bday` gives `bday-builder` and `bday-critic`)
   and `bridge` (Mattermost). Under **Agents and folders**, set **Builder directory** to
   `<GROUPBUILD_DIR>/workspaces/builder` and **Critic directory** to
   `<GROUPBUILD_DIR>/workspaces/critic`.
   On Windows, the form may offer no provider. See [Known issue (Windows)](#known-issue-windows).
5. **Start a plan.** Invite your friends (or extra Mattermost users) into the room and say
   `@bday-builder plan a 2-day Gokarna trip, under ₹3000 each` or
   `@bday-builder plan our DBMS mini-project, due Friday`.

## Configure paths

The files in `agents/` are the committed source of each agent's rules. They contain two
placeholders, because the Critic reads the Builder's folder and both agents' permission rules
need absolute paths. Each agent actually runs from a copy in `workspaces/` with the placeholders
filled in. `workspaces/` is gitignored, so your machine's paths never get committed.

| Placeholder | Meaning | Windows example | macOS / Linux example |
|---|---|---|---|
| `<GROUPBUILD_DIR>` | Absolute path of your clone, with forward slashes | `C:/Users/you/groupbuild` | `/Users/you/groupbuild` |
| `<GROUPBUILD_DIR_RULE>` | The same path in Claude Code's permission-rule form | `//c/Users/you/groupbuild` | `//Users/you/groupbuild` |

Run one of these from the repo root. Re-run it whenever you change anything in `agents/`; it
rewrites the CLAUDE.md and settings files in `workspaces/` and leaves `project/` alone.

Windows (PowerShell):

```powershell
$dir  = (Get-Location).Path.Replace('\', '/')
$rule = '//' + $dir.Substring(0, 1).ToLower() + $dir.Substring(2)
foreach ($role in 'builder', 'critic') {
  New-Item -ItemType Directory -Force "workspaces/$role/.claude" | Out-Null
  foreach ($file in 'CLAUDE.md', '.claude/settings.json') {
    $text = [IO.File]::ReadAllText((Join-Path $PWD "agents/$role/$file"))
    $text = $text.Replace('<GROUPBUILD_DIR_RULE>', $rule).Replace('<GROUPBUILD_DIR>', $dir)
    [IO.File]::WriteAllText((Join-Path $PWD "workspaces/$role/$file"), $text)
  }
}
if (-not (Test-Path workspaces/builder/project/.git)) {
  git init -b main workspaces/builder/project
  git -C workspaces/builder/project commit --allow-empty -m "init: empty Groupbuild project"
}
```

macOS / Linux (bash):

```bash
dir="$PWD"
for role in builder critic; do
  mkdir -p "workspaces/$role/.claude"
  for file in CLAUDE.md .claude/settings.json; do
    sed -e "s#<GROUPBUILD_DIR_RULE>#/$dir#g" -e "s#<GROUPBUILD_DIR>#$dir#g" \
      "agents/$role/$file" > "workspaces/$role/$file"
  done
done
if [ ! -d workspaces/builder/project/.git ]; then
  git init -b main workspaces/builder/project
  git -C workspaces/builder/project commit --allow-empty -m "init: empty Groupbuild project"
fi
```

Check the result: `workspaces/critic/CLAUDE.md` should name your real
`.../workspaces/builder/project` folder, and no `<GROUPBUILD_DIR` text should be left in
`workspaces/`.

## Known issue (Windows)

In Switch Console 0.40.1 on Windows, the template form offers no provider for a new agent, so
**Create** stays disabled
([sandbox-quantum/switch#1511](https://github.com/sandbox-quantum/switch/issues/1511)).
Creating agents by hand works. Do this instead of importing the template:

1. **Your Agents → New agent**: create `<team>-builder` with Claude Code on this computer.
   Set its working directory to `<GROUPBUILD_DIR>/workspaces/builder`, let anyone in its rooms
   address it (`anyone`), and paste the builder's `instructions` block from
   `groupbuild.template.yaml`, with `{team}` replaced.
2. Do the same for `<team>-critic`. Use `<GROUPBUILD_DIR>/workspaces/critic`, let only you and
   your agents address it (`owner-agents`), and paste the critic's `instructions` block.
3. Create a room named `<team>-groupbuild` with both agents and yourself in it, then post the
   template's `kickoff` message, with `{team}` replaced, to start the builder.

## Permissions

Each agent's limits are enforced by Claude Code settings in its folder, not just its instructions.

- **The Critic is read-only.** It can read files but not change them: file edits and writes are
  denied everywhere, and so is every git command that changes anything, plus `npx` and `npm`.
  On the Builder's project it can run only `git log`, `git diff` and `git show`. Deny rules
  hold in every permission mode. Switch also lets only the creating agent update a room
  document, so the Critic can't change the Builder's plan or log.
- **The Builder can't push.** `git push` is an ask rule, and no permission mode approves those
  automatically, so a push is refused or waits for a person. It can't change git config or
  remotes, and can't edit its own CLAUDE.md or permission settings. It may read and edit its
  own folder, run git on the project, and use WebSearch and WebFetch for research.
- **Permission modes.** The settings use "don't ask" mode, which refuses anything not
  pre-approved when you run an agent yourself. Under Switch, Console starts each agent in its
  own permission mode (auto by default), which takes precedence. The allow and deny rules
  still apply, and anything else goes to Claude Code's automatic review.
- **No co-author lines.** The Builder's commits don't add a Claude co-author trailer.

## Commands

| Say | What happens |
|---|---|
| `@bday-builder plan ...` | Starts a plan: opens `Plan: <name>` and asks each person for what's missing |
| `A` or `B` | Your vote on the open question |
| `deadline's up` or `decide` | Applies the deadline rule and announces the decision (for a group project, closes the topic or task round) |
| `status` | Who has replied, open votes, versions so far |
| `undo` | Reverts the last version |
| `show v3` | What changed in v3 |
| `lock it` or `ship it` | Finalises: tags the commit, zips the plan to `dist/`, posts release notes |

## What groups can plan

| The group says | What gets decided | What the Builder makes |
|---|---|---|
| "2-day Gokarna trip from Mangalore, under ₹3000 each" | dates, who's coming, trek or rest day | itinerary page with travel, stay, budget split and packing list |
| "our DBMS mini-project, due Friday" | topic, who does which task | project plan with a task table, milestones and a non-responders section |
| "surprise party for Arjun" | date, venue, who brings what | party plan with a task list and a cost split |
| "sundowner on Saturday" | place, time, who's coming | a printable invite and a short plan |
| "our club's fest stall" | theme, shifts, budget | a stall plan with a shift table and shopping list |
| "split last night's dinner" | who had what | a small bill-split calculator page |

## Examples

- [`examples/dbms-project/index.html`](examples/dbms-project/index.html): v2 of a DBMS
  mini-project plan (Library Management System) from a test room, after one Critic FIX. It
  has the decision, who does what, a timeline, a starter schema and the SQL deliverables.
- [`examples/birthday-v1/index.html`](examples/birthday-v1/index.html): v1 of the Arjun
  birthday page from a dry run.

## Limitations

- **Agents act only when a message arrives.** There are no timers or reminders, so a person has
  to say `deadline's up`.
- **The Critic can't browse.** It checks prices and timings against the sources the plan lists,
  and flags unsourced numbers that aren't marked `estimate`.
- **Web numbers go stale.** Check prices and timetables before anyone pays.
- **Hours are estimates.** The fairness rule splits estimated hours, not the real effort, and the
  Critic checks the split, not the quality of the work.
- **One active plan per room.**
- **Only the Builder edits the room documents.** People add their inputs by chatting.
- **The Critic reads code.** It doesn't open the page in a real browser.
- **Locking makes a zip.** It doesn't publish anything, and the Builder can't push.

## Project layout

```
groupbuild/
├── groupbuild.template.yaml        # Switch room + both agents
├── agents/                         # committed source of each agent's rules (with path placeholders)
│   ├── builder/
│   │   ├── CLAUDE.md               # Planner and Builder rules: Decide, Build, Lock, Log
│   │   └── .claude/settings.json   # permissions: git, edits in its folder, web research
│   └── critic/
│       ├── CLAUDE.md               # Critic rules, plan checks and group project checks
│       └── .claude/settings.json   # read-only permissions on the builder's project
├── workspaces/                     # local only, gitignored; made by "Configure paths"
│   ├── builder/                    # builder's working directory
│   │   ├── project/                # the plan being built, its own git repo
│   │   └── dist/                   # zips from "lock it"
│   └── critic/                     # critic's working directory
├── examples/
│   ├── dbms-project/index.html     # v2 of a DBMS mini-project plan from a test room
│   └── birthday-v1/index.html      # v1 of the Arjun birthday page from a dry run
├── DEMO.md                         # 2.5-minute demo script
└── README.md
```

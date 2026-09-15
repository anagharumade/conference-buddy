# Conference Buddy — a deepagents workshop

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/YOUR-ORG/wad-conference-buddy?quickstart=1)

Build an agent that plans your three days at WeAreDevelopers World Congress North
America 2026, using the actual agenda, in four executable sections.

**Open [`conference_buddy.ipynb`](conference_buddy.ipynb).** That's the workshop.

| Section | Adds | deepagents concept |
|---|---|---|
| 0 | setup + smoke test | `create_deep_agent` |
| 1 | the agenda | `tools=` |
| 2 | a plan that persists | `backend=`, `TodoListMiddleware` |
| 3 | specialist scouts | `subagents=` |
| 4 | trust | `memory=`, `interrupt_on=` |

Roughly 15 minutes per section, plus 10 for intro and 10 to close.

## Setup

**GitHub Codespaces (recommended for the workshop).** Click the badge above.
GitHub asks for your API key on the create screen — it becomes a Codespace secret,
so the notebook never prompts. Dependencies and the dataset are installed during
container creation. Open `conference_buddy.ipynb` and run the cells — the kernel is
selected automatically.

Attendees who leave the key blank aren't stuck; the notebook falls back to a
`getpass` prompt.

**Colab / hosted:** open the notebook, run the first two cells. The bootstrap
clones this repo and prompts for your API key with `getpass`. Nothing else needed.
Update the clone URL in the bootstrap cell before you publish.

**Local:**

```bash
git clone <this repo> && cd wad-conference-buddy
uv venv && source .venv/bin/activate
uv pip install -e . jupyterlab
cp .env.example .env        # add your API key
python scripts/build_dataset.py
jupyter lab conference_buddy.ipynb
```

Any provider works — set `BUDDY_MODEL` in `.env`:
`anthropic:claude-sonnet-4-6`, `openai:gpt-5.5`, `google_genai:gemini-3.6-flash`,
`ollama:...`. Defaults to Anthropic.

### Troubleshooting
Create a default workspace Anthropic API key and delete all active codespace workspaces

## Repo layout

```
.devcontainer/            Codespaces: image, secrets, extensions
  on-create.sh              slow setup — baked into prebuilds
  post-create.sh            fast per-codespace checks
conference_buddy.ipynb    the workshop
buddy/data.py             dataset access (imported, not taught)
buddy/nb.py               display helpers: run(), show_workspace(), show_todos()
data/sessions.json        36 sessions over 3 days
seed/AGENTS.md            starting memory file for section 4
workspace/                where the agent writes (gitignored)
scripts/build_notebook.py regenerates the notebook — edit here, not the .ipynb
scripts/build_dataset.py  regenerates the dataset (no network needed)
scripts/fetch_sessions.py pulls the live agenda feed
steps/*.py                the same four sections as plain scripts
```

The tools are defined **inside the notebook**, not imported. Tool design is one of
the lessons, so attendees need to see and edit them. `buddy/data.py` holds only
the boring JSON access.

### Editing the notebook

Edit `scripts/build_notebook.py` and re-run it. Keeps diffs readable and avoids
committing execution counts and stale outputs. Commit both files.

## Data

`data/sessions.json` mixes real published sessions (`source: "published"`) with
filler written for this workshop (`source: "synthetic"`) so the agenda is dense
enough to have genuine conflicts. Stage names and walk times are invented.
`scripts/fetch_sessions.py` targets the live feed; its parser is written against
a format that may have changed, so verify before relying on it.

## Facilitator notes

**Clear the outputs before you ship it.**
`jupyter nbconvert --clear-output --inplace conference_buddy.ipynb`
Attendees should watch their own agent think, not read yours.

**If a codespace opens without Python or Jupyter,** the container was built
before `.devcontainer/` reached the repo. Extensions, dependencies and settings
are applied only at container creation. Fix with Command Palette → *Codespaces:
Rebuild Container*, or just delete and recreate. The welcome banner in the
terminal is the tell: no banner means the config never ran.

**Slow setup lives in `onCreateCommand`, not `postCreateCommand`.** Only the
former is baked into prebuild images; the latter reruns for every codespace even
when restored from a prebuild. Putting `pip install` in the wrong one makes
prebuilds pointless. If you add dependencies, add them to `on-create.sh`.

**Turn on prebuilds before the session.** Settings → Codespaces → Prebuild
configuration, targeting `main` on the 2-core machine type. Without it every
attendee waits two to three minutes for `pip install` while you talk. With it a
codespace opens in about twenty seconds. Set this up the day before, not the
morning of — the first prebuild takes a while to bake.

**Have a finished run in a second window.** Section 2 is the slow cell — it plans
three days for real. Talk over it using pre-run output rather than watching a
progress spinner with sixty people.

**The moments that land.** Section 1: ask the room to predict the tool sequence
before running. Section 1's test cell catches a real 5-minute gap against a
6-minute walk between two genuine published sessions. Section 3: two `task` calls
dispatched in one turn. Section 4: reject the message and watch it recover.

**Pin your versions.** Task planning became opt-in in deepagents 0.7. On anything
older `write_todos` appears without `TodoListMiddleware` and section 2 makes no
sense. `uv pip freeze > requirements.lock` once it works.

**Restarting mid-workshop.** Anyone whose kernel dies re-runs section 0 and their
current section — every agent cell is self-contained. `nb.reset_workspace()`
clears the agent's files.

**No wifi plan.** Everything except the model call is local. `BUDDY_MODEL=ollama:...`
keeps it running fully offline, though the plans get noticeably worse.

## Where to go next

Didn't fit in ninety minutes, worth an afternoon:

- **Skills** — `skills=["./skills/"]`, one `SKILL.md` per repeatable procedure
  with templates alongside. The first thing I'd add back.
- **MCP** — swap the stubbed `add_to_calendar` for a real calendar server.
- **Tracing** — `LANGSMITH_TRACING=true`, then re-run section 3 and see where the
  subagents actually spent their tokens.
- **Async subagents** — the scouts block the supervisor. Wrong shape for a buddy
  that keeps working while you're in a talk.

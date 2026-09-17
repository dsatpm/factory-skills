# Factory Skills

A specialized list of [agentic skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) that are designed to be *mostly* harness agnostic, meaning they should be able to run on most configurations. 


## What it does (workflow)

Agents.md uses a *four-step workflow*:
```
delegate(/new-feature) -> develop(/code-builder) -> 
discuss(/test-with-evidence) -> deploy(/before-and-after, /greploop)
```
There have been a few additional **/skills** added to the background to make sure the AI is behaving as expected (not hallucinating as much, not producing as much generic slop, etc.). **/no-slop** can be dropped into any repo alongside this skill factory - you just fill in *your* repo specific configurations (environment variables, checks, guardrails)


## List of skills

### /new-feature
**/new-feature** is the workhorse of the factory. 

This skill begins the workflow by first assigning a fresh Git branch off of **origin/master** to multiple agents that can then work concurrently without stepping on eachothers' workflows. Behind-the-scenes it takes care of task naming, scope checking against open PRs, dependency installs, and branch cleanup steps post-merge.

### /code-builder
**/code-builder** is a guidance skill that works from the service layer of the application's architecture. It will be busy enforcing a **two-layer separation** where **actions > domain rules** and a service layer is needed to centralize and keep tabs on how the operational mechanics are being used.

  Use **/code-builder** when:
    - Several agents are duplicating similar or identical operational logic during independent workflows
    - Deciding what is a shared service vs. an action
    - Bug fixes that happen in *one flow* does not propogate to other agents doing the same thing
    - Similar mechanics are being added to an existing or new feature

Also included is a migration checklist for how the agent should extract shared logic safely as well as a list of **anti-patterns** to avoid (leaky data, over-abstraction).

### /test-with-evidence
**/test-with-evidence** is, in my opinion, an extremely undervalued skill. 

This skill records a session of itself testing UI behavior, then posts the result with a summary to a PR and issue tracker. The recorder is programmed to run on **Linux**, **macOS**, and **Windows** and has several sub-commands baked in (`doctor`, `start`, `stop`, and `annotate`). Each new annotation taken during the recording session is timestamped and burned into *results.mp4* when stopped and writes the report to **report.md** and **manifest.json**. 

#### For headless environments 
**Playwright** is used instead of the programmed recorder, and all subsequent UI changes observed during test session still get evidence reports.

#### Important Note(s):
>- The recorder requires *ffmpeg / ffprobe* built with *libx264* and the *ass filter*, *screen-capture source*:
>  - X11 (*DISPLAY*) or wlroots Wayland (*wf-recorder*; *GNOME/KDE* are **not** supported) on Linux
>  - Screen Recording permission on macOS
>  - Any standard ffmpeg on Windows
>- `python3 scripts/evidence.py doctor` reports both
>- The raw capture is MPEG-TS, so even a crashed or killed recording session will still yield usable evidence
>- The headless path needs only a running app and a scriptable browser (Playwright via `npx`)
>- Posting evidence requres the `gh` CLI (or relevant equivelent)
>- `tests/test_evidence.py` smoke-tests the recorder end-to-end with a synthetic video source: `python3 -m pytest tests/ -q`

### /before-and-after
**/before-and-after** creates the screenshots and turns the outputs into a PR-ready markdown table. It drives the `@vercel/before-and-after` CLI.

Use **/before-and-after** when:
  - You want visual proof in a PR that a UI does what it claims it does
  - You want a `| Before | After |` generated and uploaded in one step
  - you're comparing two URLs, two images, a combination of both, or something else

#### Important Note(s):
>- This skill was vendored from [vercel-labs/before-and-after](https://github.com/vercel-labs/before-and-after) (PolyForm Shield 1.0.0, license included in the folder)
>- Install the CLI using `npm i -g @vercel/before-and-after agent-browser` 

### /greploop
**/greploop** runs in the background and iterates over a PR (GitHub), MR (GitLab), or shelved changelist (Perforce) and continuously fixes it until **Greptile** gives a perfect review:

**5/5 confidence with zero unresolved comments:** Triggers review, fixes actionable items, resolves unresolved threads, pushes, and repeats, up to **--max-iterations** cycles (10 by default)

Use **/greploop** to present a clean PR for Greptile to review before merging.

#### Important Note(s):
>- Vendored from [greptileai/skills](https://github.com/greptileai/skills) (MIT, license included in folder)
>- Requires Greptile installed on repo and any authenticated version control (`gh`, `glab`, `p4`) CLI

### /greploop-apps
**/greploop-apps** is functionally the same skill as **/greploop** except that it triggers reviews by tagging `@greptile-apps`, which bypasses Greptile's file-count limit on extremely large PRs that `@greptile` will refuse to review. When *no check run* appears, it falls back to polling Greptile's edited summary comment.

Use **/greploop-apps** when **/greploop**'s trigger gets *'Too many changes to review'*

#### Important Note(s):
>- Local variant derived from **greptileai's** `greploop` (MIT, license included in the folder); no separate upstream required.

### /no-slop
**/no-slop** is so cool. It tells AI to edit its output to remove prose and put a human voice back in. 

It names **31 patterns** to catch, such as:
  - puffery
  - filler
  - hedging
  - chatbot phrases
  - em dashes
  - colons as connectors
  - bold and emoji_overuse
  - abstract metaphor nouns
  - passive voice

and a short checklist for adding opinion and rhythm, applied as a four-step loop:
  1. scan
  2. rewrite
  3. add soul
  4. self audit


Use **/no-slop** when:
  - You are writing content that you are confident a person will read:
    - commit messages
    - PR titles and bodies
    - documentation
    - README edits
    - code commits
    - chat replies
  - Cleaning up existing text that sounds 'too robotic'

#### Important Note(s):
>- Vendored from [cursor/plugins(pstack)](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop) (MIT, license included in the folder)
>- The body matches upstream; the front-matter has two edits so agents apply the skill on their own instead of waiting for a typed **/no-slop**
>- **disable-model-invocation: true** has been dropped, and the description now names the trigger (test you write or edit for a human reader) in place of upstream's **any writing. Must always apply**, > so auto-invocation matches the scope **AGENTS.md** gives it.
>- Restore the flag if you want `slash-command-only` behavior 


## How to install

### Claude Code
**From the app:**
  - Go to `Customize`
  - Click `Plugins` tab and click `Add` dropdown
  - Select `Add marketplace` and choose *Add from a repository*
  - Copy the repo below:
  ```bash
  git@github.com:dsatpm/factory-skills.git
  ```

**From CLI:**
```bash
# Register the marketplace (owner/repo, https://, SSH URL, or ./path all work)
claude plugin marketplace add dsatpm/factory-skills

# Then install the plugins you want
claude plugin install new-feature@factory-skills
claude plugin install code-builder@factory-skills
claude plugin install test-with-evidence@factory-skills
claude plugin install before-and-after@factory-skills
claude plugin install greploop@factory-skills
claude plugin install greploop-apps@factory-skills
claude plugin install no-slop@factory-skills
```
Inside a running session the same thing works as `/plugin marketplace add dsatpm/factory-skills` followed by `/plugin install <name>@factory-skills`.

### GitHub
Clone the repo and copy or *symlink* a skill folder from `plugins/` into your **skills** directory:
```bash
# Install specific skill globally
cp -r plugins/no-slop ~/.claude/skills/

# Or scope a skill to a single project
cp -r plugins/no-slop /path/to/project/.claude/skills/
```

Claude Code will automatically detect the skill and invoke it when **any** task matches the skill's description. You can also invoke a skill through Claude Code explicitly with **/code-builder** or **/test-with-evidence**.


## Want to add a new skill?

1. Create a folder named after the skill in *kebab-case*
2. Create a **SKILL.md** file with **name** and **description** frontmatter. The description is what Claude uses to decide when the skill applies. That can be set using a **trigger-focused** setting ("Use when...")
3. Keep instructions *concise* and *actionable*. Good rule-of-thumb is to keep the skill **~ 120 - 150 lines** long. Anything under and you probably aren't being descriptive enough. Anything over and you probably have a bunch of filler and should consider editing it down or linking out to reference files in the folder.


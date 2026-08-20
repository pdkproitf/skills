---
name: create-pr
description: Create GitHub pull requests — picks the most token-efficient available method (gh CLI, GitHub MCP, or git push), and sources the PR body from an existing spec/plan instead of re-analyzing the diff
metadata:
  phase: "commit"
  input: "branch with commits pushed to remote; optionally a spec/plan path"
  output: "created PR URL (or a compare URL to finish in the browser)"
  dependencies: "git; and one of: gh CLI, a GitHub MCP server, or a GitHub remote"
---

# Create GitHub Pull Request

Open a pull request for the current branch, using whichever creation method is
available — preferring the cheapest, most automated one.

## When to trigger

Use this skill when the user:
- asks to create a new PR, open a pull request, or submit code for review
- says "create PR", "new PR", or "submit for review" after work is committed and pushed

## Prerequisites

- Commits on a branch that is pushed to the remote
- The branch follows the `branch` convention from `# WORKSPACE` (default: `feat-{short-description}` or `feat-{adw_id}-{short-description}`)
- At least one creation method available (see Step 1)

---

## Step 1 — Choose the PR method

Detect what's available and pick the **first** method that works, in this order.
Each later method exists only as a fallback for when the earlier one is missing.

1. **`gh` CLI** — check `command -v gh` and `gh auth status`. Preferred: cheapest and fully automated.
2. **GitHub MCP** — a configured GitHub MCP server exposing a create-pull-request tool. Use only if `gh` is unavailable. **Gated — see below.**
3. **git push + compare URL** — always available if there's a GitHub remote. Pushes the branch and gives the user a link to finish in the browser. Not fully automated.

### Cost / capability comparison

| Method | Token cost | Automated? | Notes |
|---|---|---|---|
| `gh` CLI | **lowest** (~1 line back: the PR URL) | ✅ fully | Filters the response for you. Preferred. |
| GitHub MCP | **highest** (tool schemas loaded into context + full PR JSON returned) | ✅ fully | Only cheap if the server is already loaded for other work *and* the response is field-filtered. |
| git push + compare URL | lowest raw, but **incomplete** | ❌ user clicks the link | No API object returned; the PR isn't actually opened until the human confirms in the browser. |

State which method you selected and why (e.g. "gh not found → falling back to MCP").

### MCP gate — pause and ask

If the selected method is **GitHub MCP**, do **not** invoke it silently. MCP is the
most token-expensive path (schema overhead plus full API-object responses). Pause,
tell the user MCP is the only automated option available, and **ask for confirmation
before creating the PR**. If they decline, fall back to git push + compare URL.

---

## Step 2 — Source the PR content

Before analyzing the diff, get the PR's substance from work that already exists —
reading a plan is far cheaper than re-analyzing a full diff. Use this precedence:

1. **Conversation context** — if this session already ran `feature` or `implement`, the spec path and a description of the work are already in context. Use them; skip the rest.
2. **Spec/plan file in `specs_dir`** — resolve `specs_dir` from `# WORKSPACE` (default: `docs/specs/`). Find the spec for this branch:
   - Match the branch's `adw_id` (from `feat-{adw_id}-{name}`) against the spec filename (`{timestamp}-feature-{adw_id}-{name}.md`).
   - If there's no `adw_id`, use the most recently modified spec in `specs_dir`.
   - Derive: **title** (feature name → conventional-commit format), **Summary** (its overview/goal), **Changes** (its completed phases / checked boxes), **Related Issue** (`adw_id` or issue number).
3. **Fallback** — only if there's no context and no spec: analyze `git log` and `git diff`.

Checked-off boxes in the spec map directly to the PR's **Changes** and **Testing** sections.

The PR body structure and a complete example live in
[`templates/pr-body.md`](templates/pr-body.md). If the project has its own
`.github/PULL_REQUEST_TEMPLATE.md`, follow that instead.

---

## Step 3 — Pre-flight checks

### Run pre-commit checks
If the project defines a pre-commit / lint / format command in `# WORKSPACE`
(e.g. a `pre_commit_cmd` key or an equivalent task runner), run it now. Skip if none.

### Verify branch state
1. **Not on the main/default branch** — never open a PR from it:
   ```bash
   git branch --show-current   # should NOT be the default branch
   ```
2. **Branch follows the naming convention** — the `branch` rule from `# WORKSPACE`.
3. **Consider squashing** related commits for cleaner history:
   ```bash
   git reset --soft HEAD~N && git commit -m "feat(component): description"
   ```

### Push the branch
```bash
git push -u origin HEAD
```

---

## Step 4 — Create the PR

Build the title in conventional-commit format (see below) and the body from Step 2,
then create the PR using the method chosen in Step 1.

### PR title format

```
<type>(<scope>): <description>
```

**Types:** `feat` · `fix` · `docs` · `refactor` · `test` · `chore` · `perf`
**Scope** is typically the component name (e.g. `cli`, `sdk`, `api`, `models`).

**Examples:**
- `feat(cli): add support for custom output formats`
- `fix(api): handle timeout errors gracefully`
- `docs(sdk): update authentication examples`

### Method A — `gh` CLI (preferred)

```bash
gh pr create --title "<title>" --body "<body>"
```

Common variants:
```bash
gh pr create --draft --title "WIP: <title>"          # draft / work-in-progress
gh pr create --label "area:cli" --label "topic:security"   # labels
gh pr create --base "release-1.0"                    # target a non-default branch
```

Use `Closes #<issue-number>` in the body to auto-close the linked issue on merge.

### Method B — GitHub MCP (only after the Step 1 gate)

After the user confirms, call the server's create-pull-request tool with
`head` = current branch, `base` = default branch, plus the title and body.
**Filter the response** to just the PR URL/number — do not echo the full API object.

### Method C — git push + compare URL (universal fallback)

`git` alone cannot open a PR, but it can hand the user a link:
```bash
git push -u origin HEAD
# Then give the user the compare URL to finish in the browser:
# https://github.com/OWNER/REPO/compare/<branch>?expand=1
```
The push output usually already prints a "Create a pull request" link — surface it.
Provide the PR's description and The PR body structure

---

## After creating

`gh` and MCP output the PR URL and number. **Display it as a markdown link** so it's clickable:

```
Created PR [#123](https://github.com/OWNER/REPO/pull/123)
Output the PR URL in the session context so follow-up skills can reference it. and the PR description in markdown format so the user can copy it into the PR body if needed.
```
Out put the title and description in markdown format if PR's not created, you just provides the compare url.

### Monitor CI (optional)
If the user wants to wait for green CI before requesting review:
```bash
gh run watch                                          # latest run for the branch
# or:
RUN_ID=$(gh run list --branch "$(git branch --show-current)" --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch "$RUN_ID"
```

## Useful `gh pr create` options

| Option | Description |
|---|---|
| `--title, -t` | PR title (conventional commit format) |
| `--body, -b` | PR description |
| `--reviewer, -r` | Request review from user |
| `--draft` | Create as draft (WIP) |
| `--label, -l` | Add label (repeatable) |
| `--base, -B` | Target branch (default: repo default) |
| `--head, -H` | Source branch (default: current) |
| `--web` | Open in browser after creation |

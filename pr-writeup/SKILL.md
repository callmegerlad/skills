---
name: pr-writeup
description: Write or rewrite a pull request title and body into a terse, reviewer-first format with a fixed template, and flag review risks the author should address before posting. Use this whenever the user asks to write, draft, improve, reformat, clean up, or tidy a PR title, PR body, PR description, or merge request description for the current branch or for a pasted summary, asks "what should this PR say", or pastes a GitHub Copilot or other auto-generated PR summary and wants it made presentable. Also use when the user asks for PR title suggestions on their own. In Claude Code or Codex, gather the input from the git branch when nothing is pasted.
---

# PR Writeup

Turn a raw or auto-generated PR summary into a short, specific description a reviewer can trust, then tell the author what a careful reviewer will ask so they can answer it before it is asked.

Auto-generated summaries (GitHub Copilot in particular) share the same problems. They open with "This pull request introduces a comprehensive...", they repeat the diff's file list without saying why anything changed, they bury the risky part under "Code cleanup", they end with a sentence about laying foundations, and they are studded with `diffhunk://` links that do not resolve outside the review UI. The job is to strip all of that and leave only what a reviewer needs.

## Gathering input

The input comes from one of two places. Decide which before writing anything.

**The user pasted a title or body.** Work from the paste. Do not run git commands unless the paste is truncated or contradicts itself, in which case check the branch to fill the gap and say that you did.

**The user pasted nothing, or asked about "this branch" or "my PR".** Build the picture from the repository with these commands, in this order.

1. Find the base. If `gh` is available and a PR exists, `gh pr view --json baseRefName,title,body,url` gives the base and the current title and body in one call. Otherwise take the first of `origin/main`, `origin/master`, `origin/develop`, or `origin/trunk` that exists, and confirm with the user if none does or more than one plausibly applies. Prefer the remote-tracking ref over the local branch so a stale local base does not skew the diff.
2. `git merge-base <base> HEAD` and use that commit for everything below, so commits already on the base are excluded.
3. `git log --reverse --format='%h %s%n%b' <merge-base>..HEAD` for the commit list. Commit messages are the author's own account of intent and usually beat the diff for the *why*.
4. `git diff --stat <merge-base>...HEAD` and `git diff --name-status <merge-base>...HEAD` for where the weight sits and what was deleted or renamed.
5. If step 1 returned an existing PR body, treat it as a paste and rewrite it.

Then read the actual diff for the parts that matter. Read every file that was deleted or renamed, every migration, every change to configuration or defaults, and every file whose name suggests auth, validation, permissions, secrets, or logging. Skim the rest through `git diff <base>...HEAD -- <path>` only where the stat output or a commit message leaves the purpose unclear. Do not read the whole diff for large branches, and do not paste diff hunks into the output.

Where commit messages and the diff disagree, trust the diff and flag the disagreement in the review flags.

If the branch has no commits ahead of base, or the base cannot be determined, say so and ask rather than guessing.

**Applying the result.** After presenting the output, offer to apply it. If `gh` is available and a PR exists, `gh pr edit --title <title> --body-file <tmpfile>` applies both. If no PR exists, offer `gh pr create --title <title> --body-file <tmpfile>`. Write the body to a temporary file first rather than passing it inline, since the body contains backticks and newlines. Never apply without the user confirming which title they picked.

## Output shape

Produce three parts in this order.

1. **Title options.** Two or three Conventional Commits titles, each with a one-line reason to pick it, then a recommendation. See `references/titles.md` for the rules.
2. **Body.** A single fenced ```markdown block the user can paste directly. Follow the template below exactly.
3. **Review flags.** Two to four short paragraphs, outside the code block, naming what a reviewer will push on. See "Review flags" below.

Start with the title options, not with preamble. Do not explain the formatting rules to the user.

## Body template

```markdown
This PR [verb] [what changed and why, in one to three sentences].

## Key changes

* [Verb-led line stating one change and its purpose]
* [...]

## Notes

* [Behavioural fact a reviewer needs that is not itself a change]
```

The summary is an overview, not a preview of the bullets. It answers three questions in plain language. What problem or need does this PR address, what will someone using or operating the system notice, and what is deliberately out of scope or removed. It should make sense to a teammate who has not opened the diff and does not know the module names. Identifiers belong in the summary only when the identifier is the subject of the whole PR. If the summary could be rebuilt by concatenating the bullets, it is a preview and needs rewriting.

`## Notes` is optional. Include it only when there is a real invariant, behavioural consequence, or deliberate design choice worth stating. Notes are also where small but load-bearing details go when they are too minor to be a Key change but too important to drop, for example a normalisation that preserves existing backend behaviour. Add other `##` headers only when a section of the PR needs its own explanation, for example a controversial removal that deserves a stated rationale. Never add a closing sentence after the last section.

## What earns a Key changes line

Key changes is a short list of the changes a reviewer must understand, not an inventory of the diff. Auto-generated summaries and the diff itself will always suggest more lines than the list should hold.

**Budget.** Three to five lines for most PRs. Six or seven only for a genuinely large branch touching several independent areas. Never more than seven. If the draft has more, merge or cut before writing anything else.

**The test for a line.** Would a reviewer read the diff differently, or ask a different question, because this line exists. If not, it does not get a line. Under that test the following usually fail.

- Consequences of another line. Removing the old component after replacing it is part of the replacement line, not a second line.
- Cosmetic or labelling changes. A renamed label, a default marked as default, a reordered section.
- Small hardening that follows naturally from the main change. Loading states, validation, and error handling added alongside a new component fold into one line about the component being robust, or into a single combined line, unless one of them changes behaviour a reviewer would not expect.
- File-level facts. Imports updated, models re-exported, types adjusted.

**Merging.** Group by what the reader experiences or what the system now guarantees, not by which file or function changed. Three bullets about validation, loading states, and error states are one bullet about the dialog no longer sending requests the API rejects or presenting an unavailable backend as empty.

**Language.** Write each line for a teammate who reviews the PR but did not write the code and may not work in this part of the codebase. Lead with the outcome in ordinary words, then the mechanism if it matters. Name an identifier only when it is the fastest way to find the change, and prefer one identifier per line. Avoid describing internal state transitions, "normalised to null", "stale selections excluded from the request", "registry-ordered", when a plainer description of the effect exists. The technical detail can go in Notes if a reviewer needs it, or be left to the diff.

Bad. "Normalised selections containing every available entry to `null` so default backend behavior is preserved."

Better, as a Note. "Selecting every available option is sent as no filter, so the backend's default behaviour is unchanged."

Bad. "Updated scope handling so narrowing the category filter also narrows available items and stale item selections are excluded from the request."

Better. "Kept the scope selections consistent, so narrowing one choice narrows the options that depend on it and selections that no longer apply are dropped."

## Formatting rules

These are hard constraints. Apply them to everything inside the markdown block.

- **No colons, semicolons, or em dashes** anywhere in the body, including inside bullets. Restructure the sentence instead. Parentheses and commas are fine. Colons in code spans such as `Optional[str]` are acceptable since they are code, but avoid them in prose.
- **No hard line wraps.** Every paragraph and every bullet is one physical line. The user copies the block into GitHub, and wrapped lines survive as broken sentences.
- **Bullets use `*`**, not `-`.
- **The summary opens with "This PR"** followed by a verb. Introduces, adds, replaces, makes, moves, removes, wraps. Not "This pull request".
- **Every Key changes line starts with a past-tense verb** when it describes work done. Added, Introduced, Implemented, Updated, Refactored, Removed, Registered, Bumped, Replaced, Extended. A line may open with a filename only when the change is to that file and no verb reads naturally.
- **Every Key changes line states a purpose or consequence**, not only the fact of the change. A purpose does not by itself earn a line, see "What earns a Key changes line". "Added `_normalise_path`" is a diff entry. "Added `_normalise_path` collapsing duplicate slashes and trailing separators so cache keys for the same resource always match" is a change a reviewer can evaluate.
- **Drop every `diffhunk://` link** and every `[[1]]`, `[[2]]` marker. Drop file paths that only restate the file list unless the path is the fastest way to identify what changed. Keep filenames in code spans when they help a reviewer find the change.
- **Backticks for identifiers**, when an identifier is used. Table names, column names, functions, classes, settings, filenames, category values, and enum members go in code spans. This is a formatting rule, not a licence to name every identifier the diff touches. See the language guidance above.

## What to cut

Auto-generated summaries pad in predictable ways. Remove all of the following.

- Opening adjectives. Comprehensive, robust, significant, major, modern, careful. State what changed and let the reader judge.
- Theme announcements. "The main themes are..." and "The changes are grouped below". The headers already do this.
- Closing sentences about foundations, groundwork, modernising, or collectively improving anything. They carry no information.
- Bullets that restate the header. "Registered new models in `__init__.py` for use across the application" says nothing beyond the filename.
- Any claim about intent that the diff cannot support. If the summary says a change ensures safety and the diff only adds a flag, describe the flag.

## Reordering

Auto-generated summaries follow the order of the diff. Reorder by what matters.

- Lead with the substantive change. If a PR adds SQL queries and wraps them in endpoints, the queries are the work and the endpoints are plumbing. Put the queries first.
- Group the same file's changes together only if they are the same change. If a file appears twice for two reasons, two bullets.
- Move anything risky out of "cleanup" or "other". Removing an auth or validation module, dropping a table, changing a default timeout or provider, or deleting an enum value is never cleanup. It gets its own bullet near the top, and often its own header with a stated reason.
- Put unrelated changes last and consider recommending a split in the review flags.

## Review flags

After the code block, write two to four short paragraphs, each naming one thing a careful reviewer will raise. This is the most valuable part of the output, so spend effort here. Do not use bullets for this section. Do not use a header. Open with a plain lead-in like "Three things worth resolving before this goes up."

Good flags are specific to this diff and would change what the author writes or does. Common sources.

- **Migrations.** Dropped tables or columns with no stated downgrade or data-loss acknowledgement. Enum values removed from a database enum type. Backfills that merge or collapse rows without saying which side wins.
- **Removed safeguards.** A module, validator, or check deleted under "cleanup" whose original purpose was security, privacy, or correctness. Ask whether it was replaced and by what.
- **Contract changes without consumers named.** Renamed fields, removed categories, changed defaults. Ask what reads the old value and whether it was updated.
- **Silent semantic shifts.** Partial success replacing hard failure. Ordering or priority changes. A metric that now excludes something it used to count. Any change where a green run means less than it did before.
- **New external dependencies in hot paths.** DNS lookups, third-party API calls, or new services added to a pipeline stage, with no stated timeout or failure behaviour.
- **Unrelated changes bundled together.** Recommend a split when one half is a fix someone might want to backport or cherry-pick and the other is a feature.
- **Things the summary mentions but does not explain.** A new exception with no call site, an exported utility with no module, an endpoint with no described auth, a hook with no second user.
- **Missing validation for a change that was validated elsewhere.** If the codebase has integration suites, schema registries, contract tests, or startup checks, and this change touches what they protect, ask whether they were run or extended.

Frame flags as observations a reviewer will make, not as accusations. "The summary does not say where redaction sits relative to the write" rather than "you forgot redaction". Offer the benign explanation alongside the concern where one exists, and say what one sentence in the body would settle it.

When the input is a PR in a series (same repo, same author, several PRs), use earlier PRs in the conversation as context. Flag when a new PR quietly reverses a decision an earlier one made, or when it depends on infrastructure an earlier one added and does not mention it.

## Worked example

Read `references/example.md` for one full before and after, including the review flags. Read it once when first using this skill, then rely on the rules above.

## Edge cases

- **Empty or truncated paste.** If the pasted body is empty or cut off and a git repository is available, fall back to gathering input from the branch and say that you did. If no repository is available, draft what can be drafted from whatever survived, mark the gaps with bracketed placeholders inside the body, and ask for the missing section.
- **Branch with one commit.** The commit message is probably already the PR title. Offer it unchanged as one of the options if it fits the title rules, and still write the body and flags.
- **Branch with many small commits.** Do not list commits as Key changes. Group by what changed, not by how it was committed. Fixup and WIP commits are noise.
- **User provides a title.** Still offer alternatives if the given title undersells or misdescribes the change. Say why.
- **Very small PRs.** Two or three bullets, sometimes one. Skip Notes unless there is a genuine invariant. Review flags may drop to one or two. A small PR with a long bullet list is the most common failure of this skill, so when the diff is small and the draft is long, cut before anything else.
- **User asks only for titles.** Give three options with reasons and a recommendation, and nothing else.
- **User gives an explicit style instruction mid-conversation** such as a different bullet marker or a different opener. Follow it from that point forward for the rest of the conversation.

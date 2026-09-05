# PR Title Rules

Titles follow Conventional Commits. Lowercase after the type, no trailing period, imperative mood, under roughly 72 characters.

## Choosing the type

- `feat` for new capability, including new modules, endpoints, tables, or configuration options.
- `fix` for correcting behaviour that was wrong, including security fixes such as wrapping secrets or preventing a leak.
- `refactor` for restructuring with no intended behaviour change, or for consolidating duplicated logic into one place.
- `chore` for tooling, scaffolding, seeds, and setup that does not ship to users. Also the safe choice when neither `feat` nor `fix` dominates.
- `docs` only when the PR is documentation alone. A PR that adds a script and documents it is not `docs`.

## Breaking changes

Append `!` after the type when the PR drops a table or column, removes an enum value, renames an output field, changes a provider, or otherwise breaks something that reads the old shape. Adding a category is borderline and usually does not warrant it. Removing one does.

## What the title should say

- Name the mechanism and the reason when both fit. `feat: strip request bodies from error logs to keep credentials out of log storage` answers why anyone would remove data from a log line.
- When the PR removes something, say what replaces it. `refactor!: replace legacy session cookies with signed JWTs` is better than `remove legacy session cookies` because it closes the obvious question.
- When a PR contains unrelated changes, the title should either name all of them or the user should split the PR. Do not pick the flattering half.
- Prefer the noun someone will search for later. If a new table or module is the thing people will grep for, its name belongs in the title.
- `scaffold` is a useful verb when nothing invokes the new code yet. It stops readers assuming the feature is live.

## Presenting options

Give two or three options as a bulleted list. Each option is the title in a code span followed by one clause on what it emphasises or when to pick it. Then one short paragraph recommending one and saying why. Where the recommendation depends on a fact only the author knows, say what to check.

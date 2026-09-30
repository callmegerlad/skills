---
name: issue-writeup
description: Draft, create, or revise GitHub issues with concise user-focused descriptions, interactive title selection, and appropriate issue fields and labels. Use for bug reports, feature requests, documentation issues, issue title suggestions, or turning review findings into issues. Does not implement fixes or write pull requests.
---

# Issue Writeup

Write an issue that explains what users experience and gives maintainers enough evidence to investigate. Use the same workflow across model providers. Follow newer explicit user preferences over these defaults.

## Gather just enough context

- Establish the repository and whether the request is draft-only, creation, or an update. Use supplied findings and available code. Read an existing issue before editing it, and check for duplicates before creating one. Do not create additional issues beyond the requested scope.
- Inspect the repository's issue template and any examples the user supplies. Borrow useful structure without importing unrelated fields or boilerplate. If a mandatory template conflicts with the requested format, explain the conflict before publication.
- Verify relevant code locations and preserve qualifications that affect the claim. Distinguish reported behavior, source-based inference, and runtime reproduction. Never invent reproduction results, logs, environment details, or deployment impact. Missing runtime validation does not prevent filing a clearly qualified finding.
- Check available organization-level issue fields and labels. Issue fields can exist without a linked project. Read [GitHub fields](references/github-fields.md) when setting metadata through the API.

## Choose the kind of work

Classify the issue as Bug, Feature, or Documentation according to the request. The kind shapes the body and title prefix. Do not set or change the GitHub issue type; leave it for the user.

| Kind of work | What the description should establish |
| --- | --- |
| Bug | Existing behavior breaks an expectation. Include reproduction steps, expected behavior, and actual behavior. |
| Feature | A new capability serves a concrete user need. Include the use case, desired behavior, and current limitation. |
| Documentation | Identify the audience, missing or incorrect information, and what readers need to understand or do. |

Security, performance, accessibility, and design can describe the topic rather than the kind. Classify by the actual request: a performance regression can be a Bug, while a new optimization capability can be a Feature. Use relevant labels for these topics.

## Body and voice

Start with **“This issue…”**, followed by one short paragraph describing the problem or requested capability and its user impact. No opening heading. Prefer the user's perspective and plain language. First-person expected behavior is useful; do not turn inferred failures into claims that the author personally witnessed them.

Use this default bug-report structure, replacing the bracketed guidance with concrete content:

```markdown
This issue [describes the problem and why it matters to the user].

## Steps to reproduce

[Only the prerequisites needed to follow the steps.]

1. [Action.]
2. [Action and what to inspect.]

## Expected behavior

[What the user should experience.]

## Actual behavior

[What happens instead, with inline evidence where useful.]
```

Adapt the body to the selected kind of work while keeping the opening “This issue…” paragraph:

- **Bugs:** use the template above. Keep Actual behavior immediately after Expected behavior.
- **Features:** use Use case, Expected behavior, then Actual behavior. Describe the current limitation or workaround under Actual behavior. Do not invent reproduction steps or imply that a missing capability is a defect.
- **Documentation:** use Scope, Desired outcome, and Current state when useful. For a behavioral defect, use the bug format instead. Describe the outcome without introducing an acceptance-criteria checklist.

Omit sections that add no information. When Expected behavior and Actual behavior apply, keep them together in that order.

- Keep paragraphs short and steps actionable. Include technical detail only when it helps investigate. Avoid repeating the summary in every section or prescribing a large implementation plan.
- Put code references in descriptive Markdown links beside the claims they support. Prefer verified GitHub commit permalinks with line anchors. Do not create a separate references or citations section, link to local filesystem paths, or publish broken links to untracked files.
- Do not include a Problem heading, Origin section, conversation/thread IDs, or references to the local review document that supplied the finding. Carry the evidence into the issue itself.
- Do not include priority, effort estimates, or an acceptance-criteria section in the body. Do not set an Effort field unless explicitly requested. When asked to remove effort, clear the issue's existing Effort value too.
- Include version, environment, or validation limits only when they help interpret the report. Keep them brief and omit empty template sections, filler, and closing summaries.
- Add an image only when it materially explains a visual defect, workflow, or proposed placement. Use a real capture for observed behavior and label mockups as proposals. Inspect supplied images before use. Ensure attachments are accessible to issue readers and contain no exposed credentials. Do not add decorative images or pass generated imagery off as evidence.

## Interactive title selection

For issue creation or a requested title rewrite, obtain the user's title choice before publishing unless they already explicitly chose one. Use the host's native multiple-choice input tool when available and permitted. In Claude Code, use `AskUserQuestion`. In Codex, prefer `request_user_input_async`, or use `request_user_input` when its tool instructions permit it. This collects a title preference for an already authorized operation, not a second publication approval. For body-only or metadata-only updates, preserve the existing title without reopening title selection.

First show the complete proposed body in a fenced Markdown block, with proposed priority field and labels outside it. Then ask one single-select question, such as “Which title should I use for this issue?”, with **exactly three title options**. Make each short, searchable, and specific to the affected behavior or user impact. Give the options meaningfully different emphasis while accurately representing the same issue. Follow repository conventions rather than forcing Conventional Commits syntax. Where prefixes are customary, match the issue kind, for example `[Bug]: Sign-out is undone by a delayed refresh`, `[Feature]: Export filtered table rows as CSV`, or `[Documentation]: Explain how to configure SSO`. Offer three alternatives for the chosen kind, not one title from each kind.

Put the recommended option first and append ` (Recommended)` to its displayed label. Each suggestion's heading (`label` or equivalent) must be the complete proposed issue title, including any prefix. Its subtitle (`description` or equivalent) briefly helps the user choose that wording: explain a meaningful difference in scope or emphasis, why it is recommended, or when an alternative fits better. Use natural language tailored to the issue, without a fixed sentence pattern or generic praise about being clear or searchable. Do not replace the heading with a summary or repeat the issue title in the subtitle. For `AskUserQuestion`, set each `questions[].options[].label` to the full issue title, plus the recommendation marker when applicable, and `questions[].options[].description` to the explanation only. For example, one option object is:

```json
{
  "label": "[Bug]: Saved settings disappear after reloading (Recommended)",
  "description": "Best fit while the cause is unconfirmed; keeps the report focused on the lost settings without blaming storage or the save request."
}
```

Prefer an input tool that supports full issue titles as option headings. If the tool accepts only option strings, use the exact titles as those strings and show the reasons separately before the question. If a tool's label limits prevent full titles, use the text fallback below instead of moving titles into subtitles. Allow a custom title through free-text input when supported. Keep recommendation markers and explanations out of the published title.

Wait for the user's submitted choice before creating the issue or applying a title change. A preselected default, an empty response, or elapsed time is not a selection. If the user asks for revisions, make them before publishing. If no permitted multiple-choice tool supports full titles as option headings, show the three options with their reasons as text and ask for a choice. Do not simulate a picker with a Markdown checklist or ask for an option number when a suitable native picker is available.

Honor an explicitly chosen title verbatim and apply the prepared result without asking for another confirmation. Draft-only requests remain drafts even after title selection. For title-only requests, provide exactly three options with reasons and a recommendation, without preparing or publishing a body.

## Metadata and publication

- Do not set or change the issue type. Add relevant labels, preferring existing repository conventions and leaving unrelated metadata intact.
- Set priority using **Fields → Priority**, matching the available option names and impact. Do not use a priority label or put priority in the body. High can fit an access/data risk or a major broken workflow; Urgent requires evidence of immediate critical impact. Do not automatically inherit the example issue's severity. If the field is unavailable or permission is denied, report that limitation rather than substituting a label or creating organization-wide fields.
- Remove a superseded priority label from the target issue when moving priority into its field. Do not delete the repository's label definition. Do not assign owners, milestones, or projects without context supporting the choice.
- Use the available GitHub connector or `gh`. For CLI publication, write the exact body to a temporary UTF-8 file and use `--body-file`. Keep shell quoting safe. Do not require a particular provider or another writing skill.
- After creation or editing, read back the issue to verify its title, body, field values, and labels. If a write times out or partially fails, inspect the resulting state before retrying so you do not duplicate an issue or overwrite a successful change. Stop blind retries on a persistent permission or capability failure and report what remains unapplied.
- Return the issue link and a brief account of applied changes or limitations. Do not claim unsupported metadata was set.

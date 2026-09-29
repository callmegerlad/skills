# GitHub issue metadata

Read this only when setting custom fields through GitHub's API. Prefer an available connector that exposes the same capabilities. Discover IDs and option names per repository; never reuse values from another issue or organization.

Useful REST endpoints, relative to `https://api.github.com`:

| Purpose | Method and path |
| --- | --- |
| Current issue, labels, and field values | `GET /repos/{owner}/{repo}/issues/{number}` |
| Organization issue fields and options | `GET /orgs/{org}/issue-fields` |
| Current issue field values | `GET /repos/{owner}/{repo}/issues/{number}/issue-field-values` |
| Add or update selected field values | `POST /repos/{owner}/{repo}/issues/{number}/issue-field-values` |
| Remove one field value | `DELETE /repos/{owner}/{repo}/issues/{number}/issue-field-values/{issue_field_id}` |

For Priority, find its field ID and the exact desired option name. Send JSON with this shape, using the discovered ID instead of the example value:

```json
{"issue_field_values":[{"field_id":123,"value":"High"}]}
```

Write the JSON to a temporary file and pass it through `gh api --method POST ... --input <file>`. Single-select writes accept the option **name**, while reads may return its numeric ID and a `single_select_option` object. Verify the returned name.

Use POST to preserve unrelated field values. PUT replaces all field values; an empty POST array also clears them. To remove Effort, first check whether it has a value, then delete only that field value.

Organization issue fields are distinct from Projects fields. An empty project list does not mean Priority is unavailable. Do not add an issue to a project merely to set an available organization field.

If the API shape changes or an error needs clarification, consult the official [issue-field API documentation](https://docs.github.com/en/rest/issues/issue-field-values) and [issues API documentation](https://docs.github.com/en/rest/issues/issues). Re-read the issue after a partial CLI failure before retrying.

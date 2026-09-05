# Worked Example

## Input

This example uses a pasted auto-generated body. When the input comes from a branch instead, the raw material is the commit list, the diff stat, and the files read in full, but the output and the reasoning are the same.

Title as given: `feat: update upload handling and storage layout`

Body as given (auto-generated, links abbreviated):

```md
- Replaced 'attachment_path' with 'object_key' in Upload and related models.
- Introduced 'chunks' to track multipart upload parts.
- Updated storage client and URL generation to reflect new structure.
- Removed content-type validation from the upload request flow.
- Adjusted retry handling in the storage client.
- Cleaned up references to the legacy uploads directory in comments and documentation.

This pull request makes significant updates to the upload and storage handling, focusing on improving multipart support, renaming and clarifying storage fields, and updating validation logic. The changes enhance the clarity, robustness, and structure of file storage and retrieval.

**Multipart upload and storage client improvements:**

* Uploads are now handled as multipart transfers, allowing for more reliable handling of large files, including resumable parts. The new `ChunkManifest` dataclass tracks part numbers and ETags, and related helper functions have been updated accordingly. [[1]](diffhunk://...) [[2]](diffhunk://...)
* The `finalize_upload` function now retries failed part commits with backoff instead of failing the whole upload, and verifies that every declared part is present before assembly.

**Storage model and field renaming:**

* The storage model (`Upload` in `models/upload.py` and related DB migration) has been refactored: `attachment_path` is now `object_key`, `mime_hint` is now `content_type`, and `legacy_dir` has been removed. Field descriptions and validation logic have been updated for clarity and to match the new structure. [[1]](diffhunk://...) [[2]](diffhunk://...) [[3]](diffhunk://...)
* Maximum size limits for single-shot and multipart uploads have been adjusted (`MAX_SINGLE_BYTES`, `MAX_PART_BYTES`), and validation ensures these limits are enforced.

**Storage utilities:**

* A new utility function `build_object_key` derives a deterministic key from the tenant, upload id, and filename, ensuring keys are unique per tenant and safe for the object store.

**Codebase cleanup:**

* The unused and now obsolete `content_guard.py` file has been removed, as its responsibilities have been replaced by the new storage client and manifest logic.
```

Context from earlier in the conversation. Two PRs before this one, `content_guard.py` was introduced to sniff the real content type of uploaded bytes and reject executables and HTML regardless of the declared extension, because uploads are later served from a public bucket.

## Output

**Title options**

* `feat: add multipart uploads with resumable part tracking` — leads with the capability users will notice
* `refactor!: restructure upload storage around object keys and multipart manifests` — the `!` is earned by the renamed and dropped fields
* `refactor!: restructure upload storage and remove content type guard` — if the guard removal stays in this PR, the title should admit it

I'd take the second if the guard removal is split out, and the third if it stays.

**Body**

```markdown
This PR makes large uploads reliable by moving them to multipart transfers with resumable parts, so a single failed part is retried instead of failing the whole upload. It also changes how stored files are addressed, replacing filesystem-style paths with a deterministic key per tenant, which reshapes the storage schema and retires the legacy uploads directory. The content type guard is removed as part of this change.

## Key changes

* Introduced multipart uploads with a per-upload manifest of parts, retrying failed part commits and verifying every part has arrived before the file is assembled
* Moved file addressing to a deterministic per-tenant key built from tenant, upload id, and filename, with the storage client and URL generation updated to match
* Reshaped the `Upload` storage fields around the new key layout and dropped the legacy directory column, with a migration covering both
* Revised the size limits for single and multipart uploads and enforced them in validation
* Removed `content_guard.py` and content type validation from the upload flow

## Notes

* A file only becomes addressable once every declared part has been committed, so a partially uploaded file is never served.
```

The body above reflects what you wrote, but I'd push back on the framing before you post it.

Removing `content_guard.py` is not codebase cleanup. In the previous PR, it was the control that stopped disguised executables and HTML from landing in a bucket that is served publicly. Manifest verification does not replace any of that. The manifest checks that every part arrived. The guard checked what the bytes actually were. Deleting the second and keeping the first means anything a client declares as an image is now stored and served as one.

That may well be deliberate. Sniffing may have moved to a bucket-side scanning job, or the bucket may no longer be public, or the declared `content_type` may now be validated against an allowlist somewhere the summary does not mention. Any of those is fine, but the reason belongs in the body in its own section, not in a cleanup bullet at the bottom. Reviewers who approved the guard two PRs ago will want to see it addressed directly.

If the removal is intentional, replace the last bullet with a section like this.

```markdown
## Removing the content type guard

The guard previously sniffed uploaded bytes to reject executables and HTML before storage. That responsibility now sits with [state what actually protects the bucket now and why byte sniffing at upload time is no longer needed].
```

If it is not fully intentional, this PR widens what can be served from your infrastructure and is worth splitting so the removal can be reviewed on its own.

## What this example demonstrates

- The commit-message list and the generated prose were merged into one body rather than kept as two.
- Eight candidate changes became five lines. The `ChunkManifest` dataclass, the `finalize_upload` retry, and the part verification are one change from the reader's point of view, so they share a line. The field renames and the migration are one line. Updating comments and documentation did not earn a line at all.
- The summary says what a user gains, what changes structurally, and what is removed, without naming a single function. Someone who has not opened the diff can read it and know what the PR is for.
- Every bullet starts with a verb, states an outcome in plain language first, and names at most one identifier where it helps locate the change.
- The guard removal moved from the last bullet under "cleanup" to the summary paragraph and got its own proposed header.
- No colons, semicolons, or em dashes inside the markdown block. The em dashes in the title options list are outside the block, where the constraint does not apply, though avoiding them there too is fine.
- The review flags use conversation history, offer the benign explanation, and say exactly what one section in the body would settle.

---
description: Publish a file or folder from the working directory to 23artifacts as a live, private-by-default address — a page with its images, scripts and styles, a site of several pages, or a single file. Use when the user asks to publish, host, share or put online something that already exists on disk.
argument-hint: "[path] [name]"
---

Publish from disk to 23artifacts: $ARGUMENTS

1. **What to publish.** The first argument is a path to a file or a folder. With none, use the one obvious build output (`dist/`, `build/`, `out/`, `public/`) if there is exactly one, and otherwise ask. A folder publishes as a site and needs an `index.html` at its root, or `entrypoint` naming the file to open. Anything after the path is the artifact's name; without one, use the page's `<title>` or the folder's name.
2. **New, or a new version.** When the user names an artifact they already published — its address, its identifier or its key — save a new version of it with `artifact`. Otherwise it is a new artifact. When the same thing will be published again (a build output, a docs folder), pass a `key` — a short lowercase name such as the folder's (`docs-site`): a save by a key the workspace does not hold yet makes the artifact, and every later save with that key is its next version, so a rerun needs no lookup.
3. **Measure before sending.** Count the files and their bytes, leaving out what a served site never needs: `.git`, `node_modules`, `.DS_Store`, and source maps unless the user asked for them.
4. **Up to 50 MB in all, 25 MB a file and 2,000 files:** read the files and call `save_artifact` with `name` (or `artifact`, or `key`) and `files` — text as `utf8`, everything else `base64`. For a new version where only a few files changed, send just those as `changes` on `base: "live"` instead; every other file is kept.
5. **Larger, or too many files to carry through the conversation:** call `create_credential` with `{ kind: "upload", artifact: "<the existing artifact, or a new key>" }`. It returns an upload address: a capability URL for that one artifact, valid for minutes, that is the whole credential. Write the same JSON `save_artifact` takes (without `artifact`) to a file with a short script, then POST it from the shell:

   ```sh
   curl -sS -X POST "$UPLOAD_ADDRESS" -H "Content-Type: application/json" --data-binary @save.json
   ```

   A single file over 25 MB is uploaded first; `get_guide` with topic `large-sites` has both routes and what to do when a harness refuses the upload address.
6. **Report** the `address` from the result and who can open it (`whoCanOpen`), in the result's own words. When the result lists files it left out (`leftOut`), say which, and ask whether they should have gone. Compare the result's per-file `sha256` with the local files when the user wants proof that what is served is what is on disk.

Save live unless the user asked to stage a version for approval first. Access starts at the workspace's default, private unless one was set; change who can open it only when the user says who.

---
description: Show an artifact published on 23artifacts — its address, its state, its live version, who can open it and its preview image. Use when the user asks to see, open, check or find something they published.
argument-hint: "[artifact address, identifier or key]"
---

Show the 23artifacts artifact: $ARGUMENTS

1. With nothing named, call `list_artifacts` and ask which one.
2. Call `get_artifact` with `artifact` set to what the user gave — an address, an identifier and a key all work (a key only in the workspace it belongs to). Given a description rather than a name, pass it as `q` instead: the matches come back together, and the user picks one.
3. Reply with its address, whether it is live, offline or deleted, its live version and who can open it, in the result's own words. Where the host draws the artifact's card, the preview image is on it; elsewhere the result points to the preview image.
4. To look inside it — to inspect, reconstruct or edit it here — call `get_artifact_files` for the manifest, then read only the files needed by path.

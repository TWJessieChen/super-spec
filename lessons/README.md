# lessons/ — platform lessons that travel with super-spec

Each file here holds what went wrong on one development platform, written so that a **different** project on the same platform can avoid it from day one. The folder ships inside the super-spec repo, so installing or updating super-spec on a new machine brings every recorded lesson with it; `/super-spec lessons import` (also offered by pre-flight) installs the matching ones into a project.

Flow: `flows/lessons.md` (record · import · harvest). This folder is data; the flow is what reads and writes it.

## Files

| File | Platform | Detected when the project has (any of) |
|---|---|---|
| `_common.md` | any project | always |
| `android.md` | Android apps (Kotlin / Java, Gradle) | `AndroidManifest.xml`, `build.gradle`, `build.gradle.kts` |
| `aosp.md` | AOSP / BSP / framework | `Android.bp`, `Android.mk`, `build/envsetup.sh` |
| `unity.md` | Unity | `ProjectSettings/ProjectVersion.txt` |

Add a platform: create `<platform>.md` with the same frontmatter (`platform`, `detect`) and add a row above. Detection runs `git ls-files` and matches the `detect` globs.

## What is a lesson (and what is not)

A lesson is a mistake that **any** project on the platform can make — it comes from how the platform, its SDK or its toolchain behaves, not from one project's own code. Test: would it happen in another app on this platform with completely different business logic? If not, it belongs in that project's `REVIEW.md`, not here.

Every lesson comes from something that actually happened: an escape case, a class that recurred in reviews, a build or device problem someone hit. No hypothetical "best practices" — the model already knows those.

## Entry format

```
### L-<PLATFORM>-<nnn> · <class in kebab-case>
- status: active | observe | retired
- updated: <YYYY-MM-DD>
- when: <globs and/or diff keywords — same syntax as a REVIEW.md trigger>
- symptom: <what a user or developer observes, in platform terms>
- root_cause: <why it happens on this platform>
- enforce: <structure | type | lint | test | review> — <what enforces it, e.g. "detekt rule X" / "instrumented test pattern below" / "review check only: needs judgment">
- review_check: <check text for REVIEW.md, without id and trigger, or none>
- sensitive_paths: <globs, or none>
- verify: <a Doctor / health / feature-map step to add to the verify skill, or none>
- evidence: <k> project(s) · first seen <YYYY-MM> · <kinds: escape, recurring class ×n, build issue> · proved_red: yes | no
```

Optionally followed by one fenced block with the portable artifact (lint rule, test template, Gradle snippet). Keep it generic and short; it is a starting point the importing project adapts.

- `status: observe` — seen once, or not yet proven; imported as a non-blocking `## Observe` check.
- `status: active` — seen in ≥ 2 projects, or proven red; can be imported as `## Always check` or as an `[Enforce]` backlog item.
- `status: retired` — the platform fixed it (new SDK / tool version) or it can no longer happen; kept so nobody re-adds it. Say why in `root_cause`.

## Privacy rule

Lessons leave the project they came from. They never contain: project or customer names, package / application ids, internal hosts or paths, credentials, or code copied from the project beyond a minimal generic snippet. `flows/lessons.md` shows every draft before it is written.

## Ids are permanent

An id is never reused or renumbered. Projects record imported ids in `openspec/lessons.lock`, so re-import only shows new or updated lessons.

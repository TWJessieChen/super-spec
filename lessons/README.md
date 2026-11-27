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
| `windows-python.md` | Windows desktop apps in Python (Tk, OpenCV, native tools) | `**/*.py` |

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

## Recorded lessons (quick index)

One line per entry so the folder can be skimmed without opening each platform file. Keep it in sync when `flows/lessons.md` adds or updates an entry.

### windows-python.md — Windows desktop apps in Python (Tk, OpenCV, native tools)

| Id | Class | One-line summary | Status |
|---|---|---|---|
| L-WINPY-001 | cv2-imread-nonascii-path | `cv2.imread` / `imwrite` fail silently on non-ASCII (e.g. CJK) paths on Windows; use `np.fromfile` + `imdecode` / `imencode` + `tofile`. | observe |
| L-WINPY-002 | git-bash-rewrites-device-paths | Git Bash rewrites absolute POSIX paths in `adb shell` / `adb push` arguments; set `MSYS_NO_PATHCONV=1`. | observe |
| L-WINPY-003 | tk-event-generate-on-withdrawn-root | Tk `event_generate` does nothing on a withdrawn root; call the handler directly in unit tests. | observe |
| L-WINPY-004 | stale-import-path-after-install | After an in-process pip install, a newly created user site-packages is not on sys.path; add it before re-checking imports. | observe |
| L-WINPY-005 | cmd-c-quote-stripping | `cmd /c` strips the outer quotes of the line; use `/s /c "<line>"` so a .bat path with spaces still runs. | observe |
| L-WINPY-006 | powershell-single-element-array-unrolled | PowerShell unrolls a one-element array returned from a function; wrap with `@(...)` and test the one-item case. | observe |
| L-WINPY-007 | tk-dialog-parent-destroyed | A Tk dialog parented to a secondary window dies when a refresh closes that window; parent dialogs to the main window. | observe |

### _common.md — any project

| Id | Class | One-line summary | Status |
|---|---|---|---|
| L-COMMON-001 | partial-copy-no-rollback | Replacing several files as one set must restore already-replaced files on error and never drop an unrestored backup. | observe |
| L-COMMON-002 | untracked-required-asset | Resource files the code or build needs are excluded by .gitignore; a fresh clone cannot run or build. | observe |
| L-COMMON-003 | atomic-write-temp-names | Fixed `.tmp` names collide; random ones leak after a crash — use unique names and clean by exact pattern and age. | observe |

`android.md`, `aosp.md`, `unity.md`: no lessons recorded yet.

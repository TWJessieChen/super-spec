---
platform: windows-python
detect: [**/*.py]
---

# Lessons — Windows desktop apps in Python (Tk, OpenCV, native tools)

<!-- Entries use the format in lessons/README.md. Ids: L-WINPY-001, L-WINPY-002, … (never reused).
     Written by `/super-spec lessons record` / `harvest`; read by `/super-spec lessons import`. -->

### L-WINPY-001 · cv2-imread-nonascii-path
- status: observe
- updated: 2026-11-24
- when: **/*.py with `cv2.imread`, `cv2.imwrite`, `cv2.VideoWriter`
- symptom: On Windows, reading or writing an image under a path with non-ASCII characters (e.g. CJK folder names) silently returns None / writes nothing.
- root_cause: OpenCV's file I/O uses the ANSI code page for paths and cannot open Unicode paths.
- enforce: lint — ban direct cv2.imread/imwrite via a ruff/flake8 rule whose message names the wrapper (np.fromfile + cv2.imdecode, cv2.imencode + tofile).
- review_check: image paths that can contain user-chosen or localized folder names go through a Unicode-safe wrapper, not cv2.imread/imwrite directly.
- sensitive_paths: none
- verify: run the image pipeline once from a folder with a non-ASCII name.
- evidence: 1 project(s) · first seen 2026-11 · build issue · proved_red: no

### L-WINPY-002 · git-bash-rewrites-device-paths
- status: observe
- updated: 2026-11-24
- when: scripts or commands calling `adb shell`/`adb push` with absolute POSIX paths from Git Bash
- symptom: A device path such as /data/... arrives at the device as a Windows-style path under the Git install directory, and the command fails or touches the wrong location.
- root_cause: MSYS path conversion in Git Bash rewrites arguments that look like POSIX absolute paths before they reach the native executable.
- enforce: review — set MSYS_NO_PATHCONV=1 (or double the leading slash) for such commands; needs judgment per call site.
- review_check: adb/native commands run from Git Bash with absolute device paths set MSYS_NO_PATHCONV=1.
- sensitive_paths: none
- verify: none
- evidence: 1 project(s) · first seen 2026-11 · build issue · proved_red: no

### L-WINPY-003 · tk-event-generate-on-withdrawn-root
- status: observe
- updated: 2026-11-24
- when: tests/**/*.py with `event_generate` on a Tk root created for tests
- symptom: Tk tests that simulate events with event_generate never see the handler run.
- root_cause: Tk does not dispatch generated events for windows that are withdrawn / not mapped.
- enforce: test — call the bound handler directly in unit tests; use event_generate only on a mapped window.
- review_check: GUI unit tests drive handlers directly instead of relying on event_generate on a withdrawn root.
- sensitive_paths: none
- verify: none
- evidence: 1 project(s) · first seen 2026-11 · build issue · proved_red: no

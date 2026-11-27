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

### L-WINPY-004 · stale-import-path-after-install
- status: observe
- updated: 2026-11-28
- when: **/*.py with `pip install`, `subprocess` + `-m pip`, `find_spec`, `importlib.invalidate_caches`
- symptom: An app that installs its missing packages with pip at start-up reports the install as failed (imports still fail) although pip succeeded; a restart works.
- root_cause: For a per-machine Python and a non-admin user, pip installs into the user site-packages; when that folder did not exist at interpreter start, `site` never put it on sys.path, and invalidate_caches() does not add it.
- enforce: test — after a successful in-process install, add `site.getusersitepackages()` to sys.path (site.addsitedir) before re-checking; unit-test the re-check with a freshly created user site dir.
- review_check: after installing packages in-process, the user site-packages created during the run is added to sys.path before imports are re-checked.
- sensitive_paths: none
- verify: run the first-start install once as a non-admin user with no user site-packages folder.
- evidence: 1 project(s) · first seen 2026-11 · recurring class ×3 · proved_red: no

### L-WINPY-005 · cmd-c-quote-stripping
- status: observe
- updated: 2026-11-28
- when: **/*.py with `subprocess` and `"cmd"`, `/c`, `.bat`, `.cmd`
- symptom: Running a .bat through `cmd /c` fails with "is not recognized as an internal or external command" whenever the script path contains a space (e.g. an install folder with a space).
- root_cause: Without /s, cmd /c removes the first and last quote of the whole command line when the line starts with a quote, so a quoted script path followed by quoted arguments loses its quoting.
- enforce: test — build the line with subprocess.list2cmdline and run `cmd /s /c "<line>"`; test by really running a stub .bat placed in a folder whose name contains a space.
- review_check: .bat/.cmd files launched through cmd /c survive a script path and arguments containing spaces (use /s plus one extra pair of quotes around the whole line).
- sensitive_paths: none
- verify: run the feature once from an install folder whose path contains a space.
- evidence: 1 project(s) · first seen 2026-11 · recurring class ×2 · proved_red: no

### L-WINPY-006 · powershell-single-element-array-unrolled
- status: observe
- updated: 2026-11-28
- when: **/*.ps1, **/*.bat with embedded PowerShell, keywords `ConvertFrom-Json`, `-is [array]`, `-is [System.Array]`, `return`
- symptom: A list read from JSON works with 0 or 2+ items but is treated as "not a list" (or loses items) when it has exactly one item.
- root_cause: PowerShell unrolls arrays written to the pipeline; a function returning a one-element array returns the single element, so `-is [array]` and `.Count` checks fail.
- enforce: test — wrap results in @(...) at the call site (or return with the unary comma `, $list`); test every list-reading path with exactly one element.
- review_check: PowerShell code that returns or receives a list wraps it with @(...) or returns it with `,`, and the one-element case is tested.
- sensitive_paths: none
- verify: none
- evidence: 1 project(s) · first seen 2026-11 · escape (pre-existing bug found in review) · proved_red: no

### L-WINPY-007 · tk-dialog-parent-destroyed
- status: observe
- updated: 2026-11-28
- when: **/*.py with `messagebox.`, `Toplevel`, `parent=`, `after(` refresh loops
- symptom: An error or confirmation dialog in a Tk app disappears by itself, or is never seen, when it was opened from a secondary window.
- root_cause: Tk destroys a dialog together with its parent; a periodic refresh (after() loop) that closes the secondary window while the modal dialog's nested event loop is running takes the dialog with it.
- enforce: structure — open every dialog through one helper that parents it to the main window (or another window no refresh closes); direct messagebox/Toplevel calls elsewhere are not allowed.
- review_check: dialogs are parented to a window that no timer or refresh can destroy while the dialog is open.
- sensitive_paths: none
- verify: none
- evidence: 1 project(s) · first seen 2026-11 · recurring class ×3 · proved_red: no

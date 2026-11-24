---
name: verify
description: Start this Android app on an emulator or device and drive the scenarios of the current change like a QA engineer, leaving a per-step PASS/FAIL trace with screenshots and logcat. Use before committing, and when super-spec's design.md says `Run: skill:verify`.
---

# verify — run the app and check it works

<!-- Example for super-spec. Copy this folder to `.claude/skills/verify/` in your repo (or next to the
     module it covers, e.g. `app/.claude/skills/verify/`), then fill in the PROJECT block. Keep improving it:
     this is the smoke-test harness every change goes through, so when it misses something or flakes, fix it.
     It is written for an agent that reads it cold, mid-task, and has never seen this app: real commands,
     real resource-ids, no placeholders left once filled in. `/super-spec` "maintain verify" keeps the
     feature map honest (flows/maintain-verify.md). -->

## PROJECT (fill in)

- Module / install task: `./gradlew :app:installDebug`
- Package: `com.example.app`
- Launch activity: `com.example.app/.MainActivity`
- Ready marker — how to tell the app is usable, not just alive: `<resource-id or text visible on the first usable screen, e.g. @id/home_list>`
- Clean data: `adb shell pm clear <package>` resets to first-launch state — `yes` (safe on test devices) / `no` (explain, e.g. needs a login token)
- Seed / login steps needed after a clean start: `<none | adb shell am start … deep link | steps>`
- Feature map — the user-facing features this skill knows how to reach. One line each: feature → how to get there (screen, resource-id or text to tap, keys) → what success looks like, including side effects.
  - `<feature>` → `<path: tap @id/xxx, then text "yyy">` → `<expected: text "zzz" visible; log line "…"; file / row / broadcast written>`
- Last confirmed against source: `<commit>` (updated by the maintain pass)

## Rules

- Do not edit source files. Evidence goes only to the evidence directory you were given (default `$(git rev-parse --git-dir)/verify-evidence/`, inside `.git` so it is never committed).
- A step passes only on positive evidence (expected text on screen in the UI dump, an expected log line). "No crash" alone is not a pass.
- **Real user path only.** Drive what a user touches — taps, keys, deep links a user can open. Do not call internal setters, debug-only activities, test-only intents or `adb shell am broadcast` to fake state, unless the feature map says that is how a user reaches it.
- **Check side effects, not only the screen.** When a scenario writes something (a file, a database row, a preference, a broadcast, a network request), confirm it: `adb shell run-as <package> ls/cat …`, `adb shell content query …`, logcat. Capture the action and the resulting state, not just the final screenshot.
- Do not retry a failing step to make it pass. If a step is flaky, report FAIL with what you saw; flakiness is a bug to fix, not to hide.
- Stop at the first FAIL of a scenario, record it, and continue with the next scenario.
- **Isolation.** One device or emulator is driven by one run at a time. If another run is using it (another agent, the user's own session), do not share it: use a different emulator (`adb -s`), or report `ERROR: device busy`. Never drive the user's personal device unless told to.

## Doctor (read-only — run first, and again after anything surprising)

Answers "is this instance worth driving?" before any scenario, and again after a failed step or an unexpected screen:

1. `adb devices` lists the target in state `device`; `adb -s <serial> shell getprop sys.boot_completed` is `1`.
2. The installed build is this checkout's: `adb shell dumpsys package <package> | grep -E "versionName|lastUpdateTime"` changed after the install step (or matches the APK you just built).
3. The app process is alive and in front: `adb shell pidof <package>` returns a pid; `adb shell dumpsys activity activities | grep -E "mResumedActivity|topResumedActivity"` names `<package>`.
4. No crash or ANR since the last clean start: `adb logcat -d -b crash` empty; no `ANR in <package>`.

Any failed item before the first scenario → `run_result: ERROR` with the item (environment problem, not the change's fault) — unless it is item 3 or 4 right after a clean launch, which means the app itself does not start: `FAIL` at `launch`. After a failed step: run Doctor, and if it fails, relaunch from the clean start (step 3) before the next scenario instead of driving a broken instance.

## Steps

1. **Device.** Doctor item 1. None → report `run_result: ERROR` with `no device / emulator attached` and stop. More than one → use the first emulator (`emulator-*`) via `adb -s <serial>`.
2. **Install.** Run the install task. Non-zero exit → `run_result: FAIL` at step `install`, with the last 40 lines.
3. **Clean start.** `adb shell am force-stop <package>`; if Clean data is `yes`, `adb shell pm clear <package>`; `adb logcat -c`; `adb shell am start -W -n <launch activity>`. The output must contain `Status: ok`; then run the seed / login steps; then wait (re-dump, at most 20 s) until the **ready marker** is on screen. Not ready → FAIL at `launch`. Then run Doctor items 2–4.
4. **For each scenario** you were asked to drive (from the change's spec deltas), map it to the feature map and:
   - read the screen: `adb shell uiautomator dump /sdcard/ui.xml && adb pull /sdcard/ui.xml <evidence>/<scenario>-<n>.xml`; find the target node by `resource-id` or `text` and tap the centre of its `bounds`: `adb shell input tap <x> <y>`;
   - type with `adb shell input text '<text>'` (spaces as `%s`), keys with `adb shell input keyevent <KEYCODE>`;
   - after each action wait for the UI to settle (re-dump until the expected node appears, at most 10 s), then screenshot: `adb exec-out screencap -p > <evidence>/<scenario>-<n>.png`;
   - check the expected result from the feature map / scenario `THEN` clause, including its side effects; record `- <scenario> / <step> — PASS|FAIL`.
   - A scenario with no route in the feature map → `- <scenario> — NOT COVERED (no route in feature map)`; add the route to the feature map afterwards.
5. **Health checks after all scenarios.** Doctor items 3–4 again → otherwise FAIL at `health`, with the matching lines. Save `adb logcat -d > <evidence>/logcat.txt`.
6. **Optional, for streaming / codec work.** If the scenario involves playback or RTSP: check `adb logcat -d | grep -iE "MediaCodec|CodecException|OMX|c2\."` for errors, and that frames are being rendered (the scenario's expected log line, or `adb shell dumpsys media.player` / `dumpsys SurfaceFlinger --latency <layer>` showing activity).
7. **Teardown.** `adb shell am force-stop <package>`. Leave the evidence directory in place.

## Report (exact shape)

```
run_result: PASS | FAIL | ERROR
doctor: ok | <failed item>
- <scenario> / <step> — PASS
- <scenario> / <step> — FAIL: <what was seen vs expected>
- <scenario> — NOT COVERED (no route in feature map)
evidence: <evidence dir>
```

followed, on FAIL, by the last 40 lines of logcat around the failure in a fenced block. `NOT COVERED` lines do not make the run fail, but the reviewer treats those scenarios as checked by reading only.

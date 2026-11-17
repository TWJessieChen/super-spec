---
name: verify
description: Start this Android app on an emulator or device and drive the scenarios of the current change like a QA engineer, leaving a per-step PASS/FAIL trace with screenshots and logcat. Use before committing, and when super-spec's design.md says `Run: skill:verify`.
---

# verify — run the app and check it works

<!-- Example for super-spec. Copy this folder to `.claude/skills/verify/` in your repo (or next to the
     module it covers, e.g. `app/.claude/skills/verify/`), then fill in the PROJECT block. Keep improving it:
     this is the smoke-test harness every change goes through, so when it misses something or flakes, fix it. -->

## PROJECT (fill in)

- Module / install task: `./gradlew :app:installDebug`
- Package: `com.example.app`
- Launch activity: `com.example.app/.MainActivity`
- Feature map — the user-facing features this skill knows how to reach. One line each: feature → how to get there (screen, resource-id or text to tap, keys) → what success looks like.
  - `<feature>` → `<path: tap @id/xxx, then text "yyy">` → `<expected: text "zzz" visible / log line / file written>`

## Rules

- Do not edit source files. Evidence goes only to the evidence directory you were given (default `$(git rev-parse --git-dir)/verify-evidence/`, inside `.git` so it is never committed).
- A step passes only on positive evidence (expected text on screen in the UI dump, an expected log line). "No crash" alone is not a pass.
- Do not retry a failing step to make it pass. If a step is flaky, report FAIL with what you saw; flakiness is a bug to fix, not to hide.
- Stop at the first FAIL of a scenario, record it, and continue with the next scenario.

## Steps

1. **Device.** `adb devices` must list one device in state `device`. None → report `run_result: ERROR` with `no device / emulator attached` and stop. More than one → use the first emulator (`emulator-*`) via `adb -s <serial>`.
2. **Install.** Run the install task. Non-zero exit → `run_result: FAIL` at step `install`, with the last 40 lines.
3. **Clean start.** `adb shell am force-stop <package>`, `adb logcat -c`, then `adb shell am start -W -n <launch activity>`. The output must contain `Status: ok`; otherwise FAIL at `launch`.
4. **For each scenario** you were asked to drive (from the change's spec deltas), map it to the feature map and:
   - read the screen: `adb shell uiautomator dump /sdcard/ui.xml && adb pull /sdcard/ui.xml <evidence>/<scenario>-<n>.xml`; find the target node by `resource-id` or `text` and tap the centre of its `bounds`: `adb shell input tap <x> <y>`;
   - type with `adb shell input text '<text>'` (spaces as `%s`), keys with `adb shell input keyevent <KEYCODE>`;
   - after each action wait for the UI to settle (re-dump until the expected node appears, at most 10 s), then screenshot: `adb exec-out screencap -p > <evidence>/<scenario>-<n>.png`;
   - check the expected result from the feature map / scenario `THEN` clause; record `- <scenario> / <step> — PASS|FAIL`.
   - A scenario with no route in the feature map → `- <scenario> — NOT COVERED (no route in feature map)`; add the route to the feature map afterwards.
5. **Health checks after all scenarios.**
   - `adb shell pidof <package>` must return a pid (the app is still alive).
   - `adb logcat -d -b crash` and `adb logcat -d | grep -E "FATAL EXCEPTION|ANR in <package>"` must be empty → otherwise FAIL at `health`, with the matching lines.
   - Save `adb logcat -d > <evidence>/logcat.txt`.
6. **Optional, for streaming / codec work.** If the scenario involves playback or RTSP: check `adb logcat -d | grep -iE "MediaCodec|CodecException|OMX|c2\."` for errors, and that frames are being rendered (the scenario's expected log line, or `adb shell dumpsys media.player` / `dumpsys SurfaceFlinger --latency <layer>` showing activity).

## Report (exact shape)

```
run_result: PASS | FAIL | ERROR
- <scenario> / <step> — PASS
- <scenario> / <step> — FAIL: <what was seen vs expected>
- <scenario> — NOT COVERED (no route in feature map)
evidence: <evidence dir>
```

followed, on FAIL, by the last 40 lines of logcat around the failure in a fenced block. `NOT COVERED` lines do not make the run fail, but the reviewer treats those scenarios as checked by reading only.

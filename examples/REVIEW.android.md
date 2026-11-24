# REVIEW.md — Android / AOSP example

<!-- Copy to your repo root (or openspec/REVIEW.md) and trim to what your project actually uses.
     These are the Android-specific lifecycle checks that used to be hard-coded in super-spec's
     reviewer prompts; the prompts now carry only the generic principles. -->

## What failure looks like here
- A platform resource (AppWidgetHost id, JobScheduler id, MediaSession, Binder cookie, Surface, codec) is not returned on cancellation or error — the system leaks it until reboot.
- Work runs on the wrong thread: blocking I/O or codec calls on Main, or UI touched from a background thread.
- A listener / receiver / observer outlives the component that registered it.
- Streaming or codec pipelines that do not stop producers before releasing consumers (frames delivered into a released surface or codec).

## Always check
<!-- Format: [id] when: <globs / diff keywords> → check: <what to look for> (source: …). Reviewers apply an entry
     only when its `when:` matches the diff. Replace `source: super-spec example` with your own escape cases / PRs. -->
- [A1] when: launch, async, withContext, Dispatchers. → check: **Dispatcher / scope** — every `launch` / `withContext` runs on the right dispatcher; nothing long-running or blocking on `Dispatchers.Main`. (source: super-spec example)
- [A2] when: launch, async, GlobalScope, CoroutineScope( → check: **Structured concurrency** — `launch` lives in a scope cancelled on owner teardown (`viewModelScope`, `lifecycleScope`, a service-owned scope); no `GlobalScope` or detached coroutines. (source: super-spec example)
- [A3] when: suspend fun, allocate, acquire, open(, use { → check: **Cancellation safety** — every `allocate → use → release` sequence releases on `CancellationException` (`try/finally`, `use { }`, or catch + cleanup + rethrow). (source: super-spec example)
- [A4] when: start, register, acquire, addView, bindService → check: **start/stop pairing & ordering** — each has a matching `stop*` / `unregister*` / `release*` / `removeView` / `unbindService`; stop the listener BEFORE cancelling the scope that feeds it; close the resource BEFORE nulling the field that owns it. (source: super-spec example)
- [A5] when: registerReceiver, ContentResolver, AppWidgetManager, EventBus, addListener, observeForever → check: **Long-lived owners** — anything registered on Application, a system service, `AppWidgetManager`, `ContentResolver`, an EventBus or a `BroadcastReceiver` is unregistered on teardown. (source: super-spec example)
- [A6] when: allocateAppWidgetId, JobScheduler, MediaSession, Binder → check: **System-allocated ids** — AppWidgetHost ids, JobScheduler ids, MediaSession tokens, Binder cookies are returned on the failure / cancellation path, not only on success. (source: super-spec example)
- [A7] when: @Volatile, AtomicReference, Mutex, synchronized, Thread(, Executor → check: **Shared state** — cross-thread mutable state uses `Mutex` / atomics / single-thread confinement; `StateFlow` / `SharedFlow` for cross-thread observation. (source: super-spec example)
- [A8] when: MediaCodec, MediaExtractor, Surface, rtsp, RTSP, RtpPacket → check: **MediaCodec / RTSP** — `stop()` → `release()` order honored; input/output buffers not touched after release; reconnect loops have an upper bound and backoff; repeated triggers have an in-flight guard. (source: super-spec example)
- [A9] when: **/AndroidManifest.xml, exported= → check: **Exported components** — new `exported="true"` activities / services / receivers / providers have a permission or an explicit reason. (source: super-spec example)

## Observe
<!-- Trial checks, same format. A hit is only a non-blocking `[Minor] observe:<id>` note. Promote to Always check
     once the hits have been real problems. -->
- 

## Enforced elsewhere
<!-- Classes enforced by code instead of review (see super-spec flows/enforcement-ladder.md). Reviewers skip these.
     Candidates worth moving out of "Always check" once your build runs the rule and you have seen it fail on a
     real past occurrence (check the rule names against the lint / detekt versions you use):
       - A2 GlobalScope → detekt `GlobalCoroutineUsage` (coroutines rule set)
       - A9 exported components → Android Lint `ExportedService` / `ExportedReceiver` / `ExportedContentProvider` as errors
     Format: - <class> → lint:<rule id> | test:<path> | type:<symbol> | structure:<where> (since <change name>) -->
- 

## Sensitive paths
sensitive_paths:
- `**/src/main/AndroidManifest.xml`
- `**/src/main/cpp/**`
- `**/src/main/jni/**`
- `**/*.aidl`
- `frameworks/**`
- `**/signing*.gradle*`
- `.github/workflows/**`

## Quick paths
quick_paths:
- `**/res/values*/strings.xml`

## Skip
skip_paths:
- `**/build/**`
- `**/generated/**`
- `gradle/wrapper/**`

## Noise
- At most 5 `[Minor]` items per review.
- Style or naming comments only when they break a rule written here, in CLAUDE.md, or the project's ktlint / detekt config.
- Every finding cites `file:line`. No evidence, no finding.
- Summaries lead with the count of blocking issues; when there are none, say so in one line.

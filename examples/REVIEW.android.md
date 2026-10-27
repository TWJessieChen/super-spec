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
- **Dispatcher / scope**: every `launch` / `withContext` runs on the right dispatcher; nothing long-running or blocking on `Dispatchers.Main`.
- **Structured concurrency**: `launch` lives in a scope cancelled on owner teardown (`viewModelScope`, `lifecycleScope`, a service-owned scope); no `GlobalScope` or detached coroutines.
- **Cancellation safety**: every `allocate → use → release` sequence releases on `CancellationException` (`try/finally`, `use { }`, or catch + cleanup + rethrow).
- **start/stop pairing & ordering**: `start*` / `register*` / `acquire*` / `addView` / `bindService` each have a matching `stop*` / `unregister*` / `release*` / `removeView` / `unbindService`; stop the listener BEFORE cancelling the scope that feeds it; close the resource BEFORE nulling the field that owns it.
- **Long-lived owners**: anything registered on Application, a system service, `AppWidgetManager`, `ContentResolver`, an EventBus or a `BroadcastReceiver` is unregistered on teardown.
- **System-allocated ids**: AppWidgetHost ids, JobScheduler ids, MediaSession tokens, Binder cookies are returned on the failure / cancellation path, not only on success.
- **Shared state**: cross-thread mutable state uses `Mutex` / atomics / single-thread confinement; `StateFlow` / `SharedFlow` for cross-thread observation.
- **MediaCodec / RTSP**: `stop()` → `release()` order honored; input/output buffers not touched after release; reconnect loops have an upper bound and backoff; repeated triggers have an in-flight guard.
- **Exported components**: new `exported="true"` activities / services / receivers / providers have a permission or an explicit reason.

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

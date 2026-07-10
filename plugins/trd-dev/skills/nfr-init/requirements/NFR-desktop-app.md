
## Development and Stable build

**Builds are automated — development and production.**
   - The **development build** is created on the development machine, by a committed, repeatable script.
   - The **production build** is created only by the release workflow running as a GitHub Action. The user invokes that workflow on GitHub and specifies the release number; production builds are never produced locally.
   - The **production build** produced on Github must be delivered as DMG file and signed (both the application and the DMG file) with the Apple Developer user key
   - The development build's version is the **latest stable release number published on GitHub with `+dev` appended** (e.g. latest stable `0.1.3` → development build `0.1.3+dev`). Derive it from the latest published release/tag — never hand-type it.


## Testing

**Functional-test every feature.** For every new feature you implement, write and run an
   automated functional test that exercises the feature's real behaviour (not just unit tests of
   helpers). Every functional test must pass before you may declare the feature complet


## Instrument the app so an AI agent can drive and test it without a human (mandatory)

**The trap this rule exists to prevent.** Interaction bugs hide in the exact place unit
tests never look. A GUI feature can pass every unit test and a code review, and still
be completely broken the moment a person uses it —
because those checks exercise pure functions and state mutations, never a real pointer or
key event landing on a real view. A too-small hit target, an event routed to the wrong
view, a drag that never starts: none of these are visible below the OS event layer. They
become reachable — to find AND to prove fixed — only once the app can be driven from outside
and made to perform the real interaction against itself.

**The rule.** Any GUI behaviour whose correctness depends on the OS — pointer hit-testing,
event routing, focus / first-responder, keyboard handling, a modal tracking loop (native
drag sessions, menu/sheet tracking), or window and multi-monitor changes — **cannot** be
certified by unit tests, however many pass. It MUST be verifiable by an AI agent driving the
**real, running, packaged app** with no human in the loop. Build that capability into the
app as a first-class, permanent testing surface — not a throwaway — and use it to prove a
feature works through its real entry point before calling it done.

**What the instrumentation must provide:**

1. **A dev-only control channel, inert in production.** Gate it on BOTH a runtime signal and
   the build variant (e.g. an environment variable *and* a development-only bundle
   identifier), so it is impossible to activate in a stable build. It must add zero behaviour
   when off — the reporting hooks compile or resolve to no-ops.
2. **A command/result protocol the agent can read and write itself.** A file-based exchange
   (a command file in, a result file out, in the app's data directory) works and needs no
   sockets or extra permissions. It must let the agent: query app **state**, query the
   on-screen **geometry** of the elements involved, and **issue actions** (click, drag from A
   to B, and seed fixtures the scenario needs).
3. **Drive real OS input, not synthetic in-app events.** A modal tracking loop advances only
   on real, HID-level input events, not on events posted into the app's own queue. Native
   drag sessions in particular ignore injected application-level events entirely — a drag
   test built on them does nothing while appearing to run. Inject at the lowest level the
   platform offers (the system HID event tap, or the OS input-synthesis API), the same path
   real hardware uses.
4. **Live, self-refreshing geometry in window-relative coordinates.** Elements report their
   current frame as they lay out AND re-report on every reflow; a stale frame aims the action
   at where an element *used* to be. Report **window-relative** coordinates, never absolute
   screen coordinates — screen coordinates go stale the instant the window moves between
   displays, sending injected events to the wrong monitor.
5. **Preserve app identity when launching.** Launch the real bundle the normal way, not by
   exec-ing the inner executable directly — the latter can break the app's notion of its own
   bundle identity and resource paths, so the gating (and other bundle-dependent behaviour)
   no longer matches production.
6. **Traces that survive code-signing.** A signed, distributed app's system-log output is
   often not readable through the platform's log tooling. Write diagnostics to a plain file
   the agent can read back offline.

**Test against reality, not against the happy pixel:**

- Assert the resulting **app state** after the interaction (did the thing actually move,
  open, focus, or change?), not merely that an event was dispatched.
- Aim at the region a **user** actually hits — the whole control, including its empty
  padding — not the one coordinate that happens to work. A test that clicks the exact centre
  of a label can pass while the user, aiming anywhere else on the same control, sees nothing.
- **Reset persisted state to a known baseline before each run.** A desktop app restores its
  last session on launch; state left over from a previous run silently violates a test's
  assumptions and makes a check fail (or pass) for the wrong reason.


## Bundling a desktop app: prove the package is complete 

This applies to **any** distributable desktop app — a macOS `.app`, a Windows
installer/`.exe`, a Linux AppImage/`.deb`, an Electron/Tauri package, etc. — and to
everything it must carry: libraries, frameworks, plugins, runtimes, fonts, data files,
locales, models, any resource the app loads at runtime.

YOU MUST ENSURE ALL THE NECESSARY DEPENDENCIES ARE BUNDLE IN THE APPLICATION.
1. **Build the package from a clean tree for releases.** Stale incremental build output
   silently omits the resources/libraries of newly-added dependencies. Don't package over a
   build dir of unknown age.
2. **Make packaging self-verifying** Maintain an explicit list of the
   libraries/frameworks/resources the app *must* contain, and after assembling the package,
   assert each one is actually present in the final bundle. **Fail the build loudly** if any
   is missing
3. **"It signed / notarized / linted / CI-passed" is NOT evidence it runs.** Never report a
   build as done on the strength of those steps alone.


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
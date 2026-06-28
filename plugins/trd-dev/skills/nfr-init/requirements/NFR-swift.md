## Implementation

- The implementation language is Swift. You can use shell scripts whenever needed.
- You MUST manage dependencies with **Swift Package Manager** (declared in `Package.swift`), and you MUST commit `Package.resolved` so every build resolves the same dependency versions.
- You MUST pin the toolchain so every build uses the same Swift compiler, and the CI toolchain MUST match the one used locally. For a macOS app built with the Xcode toolchain, pin the Xcode version in CI (e.g. `maxim-lobanov/setup-xcode` with an explicit `xcode-version`) and document the local Xcode version — note that a `.swift-version` file does nothing here, as only `swiftly` reads it. For Linux / server-side Swift, pin it with `swiftly` and a committed `.swift-version` file.
- You MUST build and test through Swift Package Manager (`swift build`, `swift test`); do not depend on a hand-configured local Xcode setup to build or test.

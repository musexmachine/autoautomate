## Skill search prompt

Find agent skills that will help build and ship autoautomate, a native Swift macOS menu bar app:

1. **macOS menu bar development.** SwiftUI `MenuBarExtra`, AppKit bridges, launch at login with `SMAppService`, permission onboarding for Accessibility and Full Disk Access.
2. **Apple release and review.** Developer ID signing, notarization, the App Store Review Guidelines, and the Human Interface Guidelines.

For each skill, give me its name, what it does, its install count, who made it, and its link. Prefer skills with 1K+ installs from known authors. Star (⭐) the one you recommend in each group and say why.

## Build orchestration prompt

**Before you start:** list the models available in Codex and confirm that "Sol 6.1 Max" and "Luna 6.0 high" exist. If either one doesn't, stop and tell me.

Use Codex to start one **Sol 6.1 Max** lead agent. The lead owns the plan and does not write code itself.

**Release target:** a Developer ID–signed, notarized direct download. The full app can't pass Mac App Store review, because 2.4.5(i), 2.5.1 and 2.5.2 rule out the sandbox-breaking sources. If a store version is ever wanted, it gets its own sandboxed "lite" target, planned from the start.

### Phase 0: release spike (before any features)

Build an empty menu bar app. Sign it with Developer ID and Hardened Runtime, notarize it with `notarytool`, and staple it. Install it on a clean Mac and confirm three things: it opens with no Gatekeeper warning, launch at login works, and the onboarding opens the right System Settings panes for Accessibility and Full Disk Access. Don't start Phase 1 until this passes.

### Every phase after that

1. **Split the work.** Break the phase into tasks that don't overlap. Give each task its own list of files it owns, its acceptance criteria, its tests, and the skills to load (for example `swiftui-pro`, `swift-concurrency-pro`, `swift-testing-pro`, and `sosumi-apple-docs` for current Apple docs). Sub-agents never edit files they don't own.
2. **Build in parallel.** Send each task to its own **Luna 6.0 high** sub-agent. Run them at the same time.
3. **Integrate.** When every task passes its tests, merge the work and run the full build and test suite.
4. **Review.** Dispatch independent reviewers that haven't seen the code:
   - **Apple lead reviewer:** acts as a senior lead engineer on Apple's macOS team. It checks code quality (correctness, Swift concurrency, memory, error handling), integration with every earlier phase, the Human Interface Guidelines (menu bar, settings, onboarding), and Apple's rules: Developer ID and notarization, Review Guideline 2.5.14 (consent and a visible indicator whenever it's recording), and the Apple Developer Program License Agreement.
   - **Privacy and security:** what each source captures, that every source is off by default, redaction before publishing, local-only storage, and the privacy policy.
   - **Accessibility:** VoiceOver, keyboard-only use, and contrast.
   - **Performance:** CPU, memory and battery use over a full day of watching, measured with Instruments.
   - **Release engineering:** entitlements, Hardened Runtime, signing, notarization, stapling, and auto-updates (for example Sparkle).
   - **First run:** a newcomer installs it and grants permissions using only the app's own instructions.
5. **Gate.** Each reviewer passes or blocks the phase, and lists each issue by severity. The lead fixes blocking issues, then sends the phase back to the same reviewers. After two failed rounds, stop and report to me.
6. **Report.** When a phase passes, send me a short summary of what shipped, what the reviewers found, and what's next.

# autoautomate: Cursor handoff prompt

You're building **autoautomate**, a macOS menu bar app that watches how someone works, finds the tasks they repeat most, and ranks which ones are worth automating. Your activity data stays on your computer.

Read this whole prompt before you plan. Use the brainstorming and writing-plans skills to turn it into a phased plan. **Show me the plan and wait for my OK before you write code.** Whenever you give me options, star (⭐) the one you recommend and say why in one sentence.

## Who it's for

1. Me first (Dani, on an 8 GB M1 MacBook Air).
2. Then my students, in a course on AI-augmented workflows.
3. Then a real product anyone can use.

The spirit: anyone who uses a computer can find and automate their own repeated work, without coding.

## How it decides what to automate

The app follows my brainstorm process (Problem → People → Plan → Program → Publish → Postmortem):

- It finds the user's real, repeated workflows from their actual activity. It never starts from "where can we use AI?"
- For each task, it shows the time spent and a confidence score with reasons.
- **The user** labels each task "candidate for automation" or "keep human." The app never sets that label.
- Rules it shows the user: keep a task human when it needs judgment, accountability, relationships or creative first thoughts, or when mistakes are costly. A task is a candidate when it's high volume, structured, rules-based, reversible, and has measurable output. If more than ~30% of runs go off script, the app flags it "redesign first."
- At the end, it ranks the user's candidates as automation or product ideas, with reasons (see open item 2).

## Architecture (two pieces, one UI)

### 1. autoautomate (Store Lite): Swift menu bar app on the Mac App Store

- Native Swift and SwiftUI menu bar app (`MenuBarExtra`), fully sandboxed.
- **Fully useful on its own** using only public, sandbox-safe APIs:
  - the app in front and how long it's in use (`NSWorkspace`, no special permission)
  - Calendar and Reminders (EventKit, with the user's permission)
  - notifications, plus launch at login (`SMAppService`)
- It's the **only UI** for everything: the dashboard, digests, labels, settings, and onboarding.

### 2. autoautomate Helper: optional Go background service, downloaded from my website

- A Go service that runs as a LaunchAgent. It's a Developer ID–signed, notarized download, and it's also offered as a Homebrew cask in my own tap.
- It reads the advanced sources after the user grants Full Disk Access, Accessibility or Input Monitoring. Those permissions belong to the Helper, so onboarding has to name it clearly.
- It stores its data in local SQLite and talks to the menu bar app over a local connection that only listens on localhost or a Unix socket. It never opens a port to the network.
- It has no UI of its own.

### One settings UI

- One **Sources** tab lists every source with the same toggle style, whether it's built in or comes from the Helper.
- The **"Advanced sources" section is always visible.** Until the Helper is installed, it says "Requires the free autoautomate Helper," shows a download link, and keeps those toggles grayed out. Once the Helper is running, the toggles work. Never hide this section.
- When the user flips a Helper toggle, the app sends the change to the Helper. The Helper reports back each source's status: on, off, or still needs a permission (with a button that opens the right System Settings pane).
- Settings also shows where every SQLite database lives on disk, a "Start at login" toggle, the model choice, and the retention settings.
- The App Store description and the review notes explain the Helper in plain words.

## Privacy rules (non-negotiable)

- **Local only.** No cloud model and no analytics. The only network call is the publish feature below.
- Every source is off until the user turns it on. Read metadata before content.
- Show a visible indicator whenever the app is recording, and get explicit consent first (App Store Review Guideline 2.5.14).
- Retention: keep raw events for 30 days, and keep summaries until the user deletes them.
- Write a privacy policy covering both the app and the Helper.

## Share link (the one exception to local only)

- One-click publish of an **anonymized summary** to a host the user connects (for example a GitHub Gist or Cloudflare Pages).
- It's off until a host is connected. The user previews exactly what will be published first. Raw events are never published, and the user can unpublish or delete the summary at any time.
- The published page is a static page that looks perfect on mobile and great on desktop.

## Outputs

- A daily digest and a weekly digest, sent as notifications.
- A live dashboard in the menu bar app that shows the top repeated tasks ranked by expected minutes saved, their confidence, the reasons, and the user's labels.

## Sources, in implementation order

Every source is a pluggable module with the same interface: enable, disable, status, required permission, and an event stream.

1. **Store Lite built-in sources:** the app in front and how long it's used, Calendar, Reminders.
2. **Helper wave 1:** window titles (Accessibility), browser history (Safari and Chrome), clipboard history.
3. **Helper wave 2:** Screen Time, Messages and Mail (metadata only).
4. **Helper wave 3:** Notes, Shortcuts, recent files, Downloads.
5. **Helper wave 4:** shell history, window and tab switching, repeated searches.
6. **Helper wave 5:** repeated typed phrases (patterns only, never raw keystrokes; needs Input Monitoring).
7. **Helper wave 6:** Slack, Teams and Discord metadata; code editor and git history.
8. **Helper wave 7:** iPhone Screen Time, Focus, Siri and Wallet. Research what's actually available first.

## Choose the model (local only)

Before you wire up any AI calls, help me pick the model. Report back, and don't pick for me.

1. **List the model's jobs:** grouping raw events into named tasks, writing one-line task summaries, answering the confidence rubric, explaining scores, and writing the digests. Note which jobs need reasoning and which only need short text.
2. **Hardware floor:** an 8 GB Apple Silicon Mac running Ollama. The default model must run there at a usable speed.
3. **Shortlist 3–4 local models** from the current Ollama library: small instruct models (1–8B) that support structured JSON output. Use their real, current names and sizes from the Ollama site.
4. **Test them** on the same 20 real task groups from my database. Score each on grouping accuracy against my labels, valid-JSON rate, seconds per digest, and RAM used.
5. **Recommend one:** give me a results table and star the default. The model is a setting, so the user can switch without a rebuild.

## Open items (plan for them, don't block on them)

### 1. Confidence score (design is set; it runs locally)

1. Define a `Scorer` interface in Go that takes a task summary and returns P(automatable), a confidence, and reasons.
2. **Measure locally:** times per week, total minutes, step variance (edit distance between action sequences), exception rate, and apps touched.
3. **The local model rates a short task summary** with those stats in view:
   - Screen first: "Is this one well-defined, repeatable task?" Drop it if not.
   - Rules-based vs. judgment (4 levels).
   - Reversibility (trivial, costly, or impossible to undo).
   - Structured inputs (yes or no).
   - Sensitive action (money, credentials, or sending as the user). If yes, force a keep-human prior.
4. **Combine the scores in Go** with weights the user can see. Label the result "uncalibrated" until the user has given about 30 labels, then calibrate against them.
5. **Rank by expected minutes saved** (minutes per week × P(automatable)). Low-confidence rows show as "unsure" instead of getting a rank.
6. **Before trusting it,** run 20–30 real tasks and compare the scores to my hand labels. Report the numbers.

### 2. Idea-ranking skill (not picked yet)

Search the skill library and trackawesomelist.com for skills that rank or vet product ideas, for example ICE/RICE, impact vs. effort, or my `profit-vetting-final-idea` skill. Shortlist 2–3, star one, and wait for me to pick. Each ranked item should show its rank, a score per factor, two or three reasons, and what would move it up.

### 3. Dashboard approach (research first)

Find skills and libraries for a live-updating SwiftUI dashboard fed by the Helper's event stream (Swift Charts and others). Shortlist 2–3 approaches and star one.

## Skills to load

Already installed:
- macOS app: `macos-menubar-tuist-app`, `macos-spm-app-packaging`, `macos-app-design`, `swiftui-pro`, `swift-concurrency-pro`, `swift-testing-pro`, `swiftdata-pro`, `sosumi-apple-docs` (current Apple docs)
- Planning and testing: `product-development`, `test-mac-app`
- Release and review: `release-mac-app`, `asc-notarization`, `asc-signing-setup`, `asc-release-flow`, `asc-submission-health`, `apple-appstore-reviewer`, `app-store-review`

## Apple rules (checked on Nov 14, 2026; check again with `sosumi-apple-docs` before you submit)

- The full watcher can't be on the Mac App Store (guidelines 2.4.5(i), 2.5.1 and 2.5.2; the sandbox blocks the Accessibility APIs). That's why the advanced sources live in the Helper.
- Re-check guideline 4.2.3 (the app must work standalone) and 2.5.2 (no outside code adding features) against the Store Lite + Helper design. Tell me your verdict before submitting.
- Helper release: Developer ID signing, Hardened Runtime, `notarytool`, then staple. Without notarization, macOS Sequoia makes the app hard to open.
- I already have an Apple Developer ID.

## Build process

**Before you start:** list the models available in Codex and confirm that "Sol 6.1 Max" and "Luna 6.0 high" exist. If either one doesn't, stop and tell me.

Use Codex to start one **Sol 6.1 Max** lead agent. The lead owns the plan and does not write code itself.

### Phase 0: release spike (before any features)

Build both an empty Store Lite menu bar app and an empty Helper. Then check four things:
- The Helper is signed, notarized and stapled, and installs through the Homebrew cask.
- The menu bar app passes `asc-submission-health` and builds for TestFlight.
- On a clean Mac, the menu bar app finds the Helper, flips a test toggle through the local connection, and shows the Helper's status.
- Launch at login works, and the permission onboarding opens the right System Settings panes.

Don't start Phase 1 until all four pass.

### Every phase after that

1. **Split the work** into tasks that don't overlap. Give each task the files it owns, its acceptance criteria, its tests, and the skills to load. Sub-agents never edit files they don't own.
2. **Build in parallel:** give each task its own **Luna 6.0 high** sub-agent.
3. **Integrate:** merge the work, then run the full build and test suite (`test-mac-app`).
4. **Review** with independent reviewers that haven't seen the code:
   - **Apple lead reviewer:** acts as a senior engineer on Apple's macOS team. Checks code quality, how this phase fits with earlier ones, the Human Interface Guidelines, and Apple's rules (Review Guidelines, notarization, the Developer Program License Agreement).
   - **Privacy and security:** what's captured, that sources are off by default, redaction before publishing, local-only storage, the local connection, and the privacy policy.
   - **Accessibility:** VoiceOver, keyboard-only use, and contrast.
   - **Performance:** CPU, memory and battery over a full day of watching, measured in Instruments.
   - **Release engineering:** entitlements, signing, notarization, the Homebrew cask, and auto-updates (Sparkle for the Helper).
   - **First run:** a newcomer installs both pieces and grants permissions using only the app's own instructions.
5. **Gate:** each reviewer passes or blocks the phase, with issues listed by severity. Fix the blockers and re-review. After two failed rounds, stop and report to me.
6. **Report:** after each phase, send me what shipped, what the reviewers found, and what's next.

## First reply I want

1. Your phased plan, with Phase 0 first.
2. Any questions, with your recommended answer starred.
3. Nothing built until I say go.

# autoautomate: Mac App Store vs Developer ID / notarized download

**Checked:** Saturday, November 14, 2026, America/New_York (box-local).  
**Method:** Official pages on developer.apple.com and support.apple.com were fetched the same day. Comparable-app pages were fetched the same day. Quotes below are from those pages. Anything not found, or that is inference rather than a quoted rule, is marked as such.

**App in scope:** autoautomate, a native Swift macOS menu bar app that watches the user's own activity locally to find repeated tasks. Planned sources include frontmost app/window titles via Accessibility (AXUIElement), browser history (Safari History.db, Chrome), clipboard history, Screen Time (knowledgeC.db / Screen Time API), Messages chat.db and Mail metadata (Full Disk Access), Reminders/Calendar (EventKit), Notes, Shortcuts, recent files, shell history, and optional keystroke-pattern capture (Input Monitoring). Also launch-at-login (SMAppService), notifications, and an opt-in one-click publish of an anonymized summary to a user-connected host. Everything else stays local. The user already has a Developer ID.

**Bottom line up front:** The full planned app is very unlikely to ship on the Mac App Store. Ship it as a Developer ID–signed, notarized direct download. A much smaller, public-API-only subset could be tried in the store later; it could not do the core watching.

---

## 1. App Store Review Guidelines (current)

**Source:** [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)  
**Version / date:** page footer, fetched November 14, 2026: **"Last Updated: October 16, 2026"**. Apple also posted [Updated App Review Guidelines are now available](https://developer.apple.com/news/?id=92th7e5i) on October 16, 2026.

### 2.4.5 — Mac App Store sandboxing and packaging

> "2.4.5 Apps distributed via the Mac App Store have some additional requirements to keep in mind:"
>
> "(i) They must be appropriately sandboxed, and follow macOS File System Documentation. They should also only use the appropriate macOS APIs for modifying user data stored by other apps (such as bookmarks, Address Book, or Calendar entries)."
>
> "(ii) They must be packaged and submitted using technologies provided in Xcode; no third-party installers allowed. They must also be self-contained, single app installation bundles and cannot install code or resources in shared locations."
>
> "(iii) They may not auto-launch or have other code run automatically at startup or login without consent nor spawn processes that continue to run without consent after a user has quit the app. They should also not automatically add their icons to the Dock or leave shortcuts on the user desktop."
>
> "(iv) They may not download or install standalone apps, kexts, additional code, or resources to add functionality or significantly change the app from what we see during the review process."
>
> "(v) They may not request escalation to root privileges or use setuid attributes."
>
> "(vii) They must use the Mac App Store to distribute updates; other update mechanisms are not allowed."

Launch-at-login with explicit user consent (SMAppService) is compatible with (iii). Quiet auto-launch is not.

### Public APIs and container (2.5.1, 2.5.2)

> "2.5.1 Apps may only use public APIs and must run on the currently shipping OS. […] Apps should use APIs and frameworks for their intended purposes and indicate that integration in their app description."

> "2.5.2 Apps should be self-contained in their bundles, and may not read or write data outside the designated container area, nor may they download, install, or execute code which introduces or changes features or functionality of the app, including other apps."

Reading Safari `History.db`, Chrome history, `knowledgeC.db`, Messages `chat.db`, Mail indexes, Notes/Shortcuts files, recent-file lists, or shell history is not a documented public API for those stores. For a Mac App Store build, that conflicts with 2.5.1, 2.5.2, and 2.4.5(i).

### Logging / monitoring user activity (2.5.14)

There is **no guideline titled "keylogging."** The closest current rule is:

> "2.5.14 Apps must request explicit user consent and provide a clear visual and/or audible indication when recording, logging, or otherwise making a record of user activity. This includes any use of the device camera, microphone, screen recordings, or other user inputs."

This applies to window-title watching, clipboard history, and especially optional Input Monitoring / keystroke-pattern capture. Consent plus a visible (or audible) "recording is on" indicator is required for App Store review. Direct-download apps are not reviewed against this guideline, but the same practice is the safe design and aligns with the Developer Program License Agreement (section 4 below).

### Accessibility API use

The Review Guidelines do **not** have a dedicated "Accessibility API" section. Accessibility is constrained for store apps by:

- 2.4.5(i) sandbox requirement, plus Apple's sandbox documentation, which lists accessibility APIs as incompatible (section 2).
- 2.5.1 public / intended-purpose API use.

### Privacy (5.1.1, 5.1.2)

> "5.1.1 Data Collection and Storage"
>
> "(i) Privacy Policies: All apps must include a link to their privacy policy in the App Store Connect metadata field and within the app in an easily accessible manner. The privacy policy must clearly and explicitly: Identify what data, if any, the app/service collects, how it collects that data, and all uses of that data. […] Explain its data retention/deletion policies and describe how a user can revoke consent and/or request deletion of the user's data."
>
> "(ii) Permission: Apps that collect user or usage data must secure user consent for the collection, even if such data is considered to be anonymous at the time of or immediately following collection. Paid functionality must not be dependent on or require a user to grant access to this data. Apps must also provide the customer with an easily accessible and understandable way to withdraw consent."
>
> "(iii) Data Minimization: Apps should only request access to data relevant to the core functionality of the app and should only collect and use data that is required to accomplish the relevant task."
>
> "(vi) Developers that use their apps to surreptitiously discover passwords or other private data will be removed from the Apple Developer Program."
>
> "(viii) Apps that compile personal information from any source that is not directly from the user or without the user's explicit consent, even public databases, are not permitted on the App Store or alternative distribution."

> "5.1.2 Data Use and Sharing"
>
> "(i) Unless otherwise permitted by law, you may not use, transmit, or share someone's personal data without first obtaining their permission. You must provide access to information about how and where the data will be used. You must clearly disclose where personal data will be shared with third parties, including with third-party AI, and obtain explicit permission before doing so."
>
> "(iii) Apps should not attempt to surreptitiously build a user profile based on collected data and may not attempt, facilitate, or encourage others to identify anonymous users or reconstruct user profiles based on data that you say has been collected in an 'anonymized,' 'aggregated,' or otherwise non-identifiable way."

The planned opt-in, previewed, anonymized share link can be designed to satisfy 5.1.2 if the user sees exactly what will be published and consents first. 5.1.1(vi) is a hard stop for any keystroke-pattern feature that could capture passwords. 5.1.1(viii) requires explicit consent before compiling Messages, Mail, browser, or similar personal data from those stores.

### "Apps may not request Full Disk Access"

**Not found** in the October 16, 2026 Review Guidelines. That exact phrase does not appear. Forum posts and older reviewer practice sometimes treat in-app prompts to enable Full Disk Access as a rejection risk; that is **not** a current quoted guideline and is marked unverified as policy. What *is* in the guidelines: 2.4.4 ("Apps should never suggest or require a restart of the device or modifications to system settings unrelated to the core functionality of the app") and 2.5.2 / 2.4.5(i) on container and sandbox. Directing a user to System Settings for a permission that is actually required for a core, public-API feature is more defensible than asking them to grant Full Disk Access so the app can read other apps' private databases.

### Other relevant guidelines

- **2.4.4:** don't require unrelated system-setting changes.
- **2.5.19:** no hidden or undocumented features; watching behavior must be obvious to users and App Review.
- **4.10:** "You may not monetize built-in capabilities provided by the hardware or operating system, such as Push Notifications, the camera, or the gyroscope; or Apple services and technologies, such as Apple Music access, iCloud storage, or Screen Time APIs." Relevant if a later store build tried to wrap official Screen Time APIs as the paid product.

---

## 2. App Sandbox — what is possible inside it

**Sources:**

- [Protecting user data with App Sandbox](https://developer.apple.com/documentation/security/protecting-user-data-with-app-sandbox) (fetched as `.md`, November 14, 2026)
- [Accessing files from the macOS App Sandbox](https://developer.apple.com/documentation/security/accessing-files-from-the-macos-app-sandbox)
- [Calendars entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.security.personal-information.calendars)
- [Device Activity](https://developer.apple.com/documentation/deviceactivity) (availability via [sosumi.ai copy](https://sosumi.ai/documentation/deviceactivity) of the same Apple doc, fetched November 14, 2026)
- [SMAppService](https://developer.apple.com/documentation/servicemanagement/smappservice)

Sandbox is required for the Mac App Store:

> "App Sandbox — a requirement for distributing your app on the App Store — limits the scope for an attacker to abuse platform features via your app."
> — [Protecting user data with App Sandbox](https://developer.apple.com/documentation/security/protecting-user-data-with-app-sandbox)

> "The sandboxed app doesn't have unrestricted access to the user's home folder."
> — same page

### Confirmed by current Apple documentation

| Planned source / feature | Inside the Mac App Store sandbox? | Basis |
|---|---|---|
| Accessibility / AXUIElement (frontmost app and window titles) | **No — listed as incompatible** | "Certain activities are forbidden by the operating system when an app runs in a sandbox. […] Use of accessibility APIs in assistive apps." — [Protecting user data with App Sandbox](https://developer.apple.com/documentation/security/protecting-user-data-with-app-sandbox) |
| Apple Events / scripting other apps (e.g. asking Safari or Mail for data) | **No — listed as incompatible** | Same list: "Sending Apple Events to arbitrary apps." |
| Reminders / Calendar via EventKit | **Yes, with entitlement + TCC** | "To add this entitlement to your app, first enable the App Sandbox or Hardened Runtime capability in Xcode, and then select Calendar." — [Calendars entitlement](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.security.personal-information.calendars) (`com.apple.security.personal-information.calendars`) |
| User-selected files (Open/Save, security-scoped bookmarks) | **Yes** | [Accessing files from the macOS App Sandbox](https://developer.apple.com/documentation/security/accessing-files-from-the-macos-app-sandbox) |
| Standard folders only if entitled (Downloads, Music, Movies, Pictures, etc.) | **Limited** | Same page, "Request capabilities to access standard folders" |
| Notifications | **Yes (public User Notifications API)** | Not listed in the sandbox forbidden list. (No contrary official statement found.) |
| Launch at login via SMAppService | **Yes, with user approval** | `register()` "Registers the service so it can begin launching subject to user approval." — [SMAppService](https://developer.apple.com/documentation/servicemanagement/smappservice). Also fits guideline 2.4.5(iii). |
| Official Screen Time / Device Activity API on native macOS | **No public native-macOS API found** | Device Activity is documented as "**Available on:** iOS 15.0+, iPadOS 15.0+, Mac Catalyst 15.0+" — not native macOS. There is no cited public API for reading `knowledgeC.db`. |
| Full Disk Access as an entitlement the app can turn on itself | **No** | "Your app can't automatically gain full disk access through an entitlement or with code: the person using your app must choose to grant access in System Settings > Privacy & Security." — [Accessing files from the macOS App Sandbox](https://developer.apple.com/documentation/security/accessing-files-from-the-macos-app-sandbox) |

### Official docs mention Full Disk Access, but do not say it lifts the sandbox

The same file-access page has a section "Request full disk access to use all files on a Mac" and then:

> "In situations where your app tries to use a file outside of its sandbox, program defensively by handling failure cases, as someone may choose not to grant full disk access to your app."

**Inference (not a quoted rule):** Full Disk Access is a TCC grant and is separate from App Sandbox. Apple does not state that FDA lets a sandboxed store app read `~/Library/Messages/chat.db`, Mail, Safari History, or `knowledgeC.db`. Combined with 2.4.5(i), 2.5.1, and 2.5.2, those reads should be treated as unavailable for a store build.

**Temporary exception entitlements** (home-relative or absolute path read access) exist in older entitlement reference material (Apple archive). The current Review Guidelines do not promise they will be approved for reading Messages, Mail, or browser databases. **Not verified** as a viable store path for this app; do not plan on them.

### Inference (not contradicted by docs, but not spelled out per source)

| Planned source | Likely store / sandbox status | Why this is inference |
|---|---|---|
| Safari `History.db`, Chrome history | Not possible as automatic watching | Private files outside the container; no public History API cited |
| Messages `chat.db`, Mail metadata | Not possible as automatic watching | Same; needs Full Disk Access and is not EventKit-style public API |
| `knowledgeC.db` Screen Time store | Not possible as automatic watching | Undocumented file; Timing itself says this "may stop working if Apple changes how Screen Time stores its data" ([Timing Screen Time help](https://timingapp.com/help/screen-time)) |
| Notes, Shortcuts, recent files, shell history | Not possible as automatic watching | Home-directory files / other apps' data, no public "read all" API cited |
| Clipboard history | Maybe, if limited to `NSPasteboard` and obvious to the user | Public pasteboard API is not on the sandbox forbidden list, but continuous logging is "making a record of user activity" under 2.5.14. Unverified whether App Review would accept a persistent clipboard log. |
| Input Monitoring / keystroke patterns | High rejection and policy risk even if TCC exists | See 2.5.14 and 5.1.1(vi). No official statement that Input Monitoring is allowed inside the sandbox for a general productivity app. |

---

## 3. Notarization and Developer ID

**Sources:**

- [Notarizing macOS software before distribution](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution)
- [Customizing the notarization workflow](https://developer.apple.com/documentation/security/customizing-the-notarization-workflow)
- [Hardened Runtime](https://developer.apple.com/documentation/security/hardened-runtime)
- [Updates to runtime protection in macOS Sequoia](https://developer.apple.com/news/?id=saqachfa) (August 6, 2024)
- [Safely open apps on your Mac](https://support.apple.com/en-us/102445) (Published Date: May 27, 2026)
- [Program enrollment](https://developer.apple.com/help/account/membership/program-enrollment/)
- [Apple Developer Program](https://developer.apple.com/programs/)
- Developer Program License Agreement PDF, English, footer **LYL262 / October 29, 2026**: [PDF](https://developer.apple.com/support/downloads/terms/apple-developer-program/Apple-Developer-Program-License-Agreement-English.pdf), [HTML](https://developer.apple.com/support/terms/apple-developer-program-license-agreement/)

### Why notarization is required for a direct download

> "Beginning in macOS 10.14.5, software signed with a new Developer ID certificate and all new or updated kernel extensions must be notarized to run. Beginning in macOS 10.15, all software built after June 1, 2019, and distributed with Developer ID must be notarized. However, you aren't required to notarize software that you distribute through the Mac App Store because the App Store submission process already includes equivalent security checks."
> — [Notarizing macOS software before distribution](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution)

> "By default, macOS Catalina and later also requires software to be notarized […]. An app that has been notarized by Apple indicates that Apple checked it for malicious software and none was detected."
> — [Safely open apps on your Mac](https://support.apple.com/en-us/102445)

> "Applications for macOS may be distributed outside of the App Store using Apple Certificates and/or tickets as set forth in Section 5.3 and 5.4."
> — DPLA §3.2(g), October 29, 2026 PDF

Notarization is **not** App Review:

> "Notarization of macOS software is not App Review. The Apple notary service is an automated system that scans your software for malicious content, checks for code-signing issues, and returns the results to you quickly."

### Current technical requirements

From [Notarizing macOS software before distribution](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution):

- Code-sign all executables with a **Developer ID** application (or installer / system extension) certificate. Do not use Mac Distribution, ad hoc, or development certificates.
- **Enable the Hardened Runtime** for app and command-line targets.
- Include a **secure timestamp**.
- Do not include `com.apple.security.get-task-allow` set to true.
- Link against the **macOS 10.9 or later SDK**.
- Use properly formatted XML, ASCII-encoded entitlements.

> "To upload a macOS app to be notarized, you must enable the Hardened Runtime capability."
> — [Hardened Runtime](https://developer.apple.com/documentation/security/hardened-runtime)

**Tooling (still current in today's docs):**

> "Starting November 1, 2023, the Apple notary service no longer accepts uploads from `altool` or Xcode 13 or earlier. If you notarize your Mac software with the Apple notary service using the `altool` command-line utility or Xcode 13 or earlier, you need to transition to the `notarytool` command-line utility or upgrade to Xcode 14 or later."
> — [Notarizing macOS software before distribution](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution)

[Customizing the notarization workflow](https://developer.apple.com/documentation/security/customizing-the-notarization-workflow) shows the current command-line flow: `xcrun notarytool submit … --wait`, then `xcrun stapler staple` so Gatekeeper can find the ticket offline. Xcode Organizer can also do this: Distribute App → Developer ID → Upload.

No newer cutoff than November 1, 2023 (for `altool` / Xcode 13) was found in today's official notarization pages. **No official "you must use Xcode 16 / 26" notarization requirement was found.** Use a current Xcode that includes `notarytool` (Xcode 14 or later per Apple).

### Gatekeeper changes in macOS Sequoia and later

> "In macOS Sequoia, users will no longer be able to Control-click to override Gatekeeper when opening software that isn't signed correctly or notarized. They'll need to visit System Settings > Privacy & Security to review security information for software before allowing it to run."
>
> "If you distribute software outside of the Mac App Store, we recommend that you submit your software to be notarized."
> — [Updates to runtime protection in macOS Sequoia](https://developer.apple.com/news/?id=saqachfa), August 6, 2024

Current user steps, from [Safely open apps on your Mac](https://support.apple.com/en-us/102445) (updated May 27, 2026):

> "After you've tried to open the app, follow these steps:"
> "1. Open System Settings."
> "2. Click Privacy & Security, scroll down, and click the Open Anyway button to confirm your intent to open or install the app."
> "3. The warning prompt reappears and, if you're absolutely sure that you want to open the app anyway, you can click Open."

A properly signed and notarized app should not need that override. An unsigned or un-notarized build will, and the old Control-click bypass is gone.

### Cost

> "The Apple Developer Program annual fee is 99 USD and the Apple Developer Enterprise Program annual fee is 299 USD, in local currency where available. Prices may vary by region and are listed in local currency during the enrollment process."
> — [Program enrollment](https://developer.apple.com/help/account/membership/program-enrollment/)

The [Apple Developer Program](https://developer.apple.com/programs/) page also lists **"$99 annual membership."** The user already has a Developer ID, so this fee is already being paid. No extra Apple fee for notarization itself was found.

### License agreement on notarization (DPLA §5.3)

> "To have Your macOS Application notarized, You may request a digital file for authentication of Your Application from Apple's digital notary service (a 'Ticket'). […] This continuous security checking will involve automated scanning, testing, and analysis of Your Application by Apple for malware or other harmful or suspicious code or components or security flaws, and, in limited cases, a manual, technical investigation of Your Application by Apple for such purposes."
>
> "You agree not to represent that Apple has performed a security check or malware detection for Your Application or that Apple has reviewed or approved Your Application for purposes of issuing a Ticket […]."
> — DPLA §5.3, October 29, 2026 PDF

Apple can revoke the ticket or the Developer ID certificate if it believes the app contains malware or the signing identity is compromised (§5.3–5.4).

---

## 4. Privacy / TCC, privacy manifests, and the license agreement

### Permissions the user must grant

Official TCC service names, from [Resetting access to protected resources in macOS](https://developer.apple.com/documentation/xcode/resetting-access-to-protected-resources-in-macos) (fetched November 14, 2026). Where Apple lists a purpose-string key, it is noted; several of the permissions this app needs have **no** purpose-string key in that table.

| User-facing permission | TCC service name in Apple's doc | Purpose-string key listed by Apple? | Needed for |
|---|---|---|---|
| Accessibility | `Accessibility` | **No key listed** | Frontmost app / window titles via AXUIElement |
| Full Disk Access | `SystemPolicyAllFiles` | **No key listed** | Messages, Mail, Safari/Chrome DBs, `knowledgeC.db`, shell history, and similar protected files |
| Input Monitoring | `ListenEvent` | **No key listed** | Optional keystroke-pattern capture |
| Screen Recording | `ScreenCapture` | **No key listed** | Only if used (some apps use it to read window titles when Accessibility is not enough) |
| Calendar | `Calendar` | `NSCalendarsFullAccessUsageDescription` / write-only variant | EventKit calendars |
| Reminders | `Reminders` | `NSRemindersFullAccessUsageDescription` | EventKit reminders |
| Automation / Apple Events | `AppleEvents` | `NSAppleEventsUsageDescription` | Scripting Safari, Mail, etc., if used |
| Sending keystrokes | `PostEvent` | **No key listed** | Not planned; listed for completeness |

Because Accessibility, Full Disk Access, Input Monitoring, and Screen Recording have no Info.plist purpose string in this official table, the app cannot rely on a one-tap system prompt with a custom explanation for those. The user has to enable them in **System Settings > Privacy & Security**. Onboarding should explain why each one is needed, then open the right Settings pane. Each source should stay off until the user turns it on.

If Screen Recording is not required, do not request it. Timing's FAQ says it uses Screen Recording only as a fallback to read some window titles, and "does not record your actual screen" ([Timing FAQ](https://timingapp.com/help/faq)). If autoautomate can get titles from Accessibility alone, skip Screen Recording.

### Privacy manifests for non-App-Store apps

[Privacy manifest files](https://developer.apple.com/documentation/bundleresources/privacy-manifest-files):

> "The types of data collected by your app or third-party SDK. You need to provide this information for your app or third-party SDK on all platforms."
>
> "The required reasons APIs your app or third-party SDK uses. You need to provide this information for your app or third-party SDK on iOS, iPadOS, tvOS, visionOS, and watchOS."

Required-reason API declarations are **not listed as a macOS requirement**.

[Privacy updates for App Store submissions](https://developer.apple.com/news/?id=r1henawx) (December 7, 2023) and [Third-party SDK requirements](https://developer.apple.com/support/third-party-SDK-requirements/) tie mandatory manifests and SDK signatures to **"submit new apps in App Store Connect"** / "upload a new app or app update to App Store Connect."

**Confirmed:** privacy manifests are required for App Store Connect uploads (and for listed third-party SDKs in those uploads).  
**Not found:** an official requirement that a Developer ID, notarized, outside-the-store Mac app must include `PrivacyInfo.xcprivacy`. Including one anyway is still reasonable documentation, especially if the app later goes to the store. DPLA §3.3.3(B) does say that if the app uses certain APIs identified in the Documentation, metadata must identify permitted reasons — that clause is in the Program Requirements, which §3.3 introduces as applying to App Store / Custom App / TestFlight / Ad Hoc submissions.

### Apple Developer Program License Agreement (data collection / monitoring)

PDF fetched November 14, 2026; footer **October 29, 2026**.

§3.3 intro (scope):

> "Any Application that will be submitted to the App Store, Custom App Distribution, or TestFlight, or that will be distributed through Ad Hoc distribution, must be developed in compliance with the Documentation and this Agreement, including the Program Requirements set forth below in this Section 3.3."

Direct Developer ID distribution is therefore not in that "must" sentence. Other agreement terms still apply, including:

**§3.3.1(A)** (for store submissions):

> "Applications may only use Documented APIs in the manner prescribed by Apple and must not use or call any private APIs. Further, macOS Applications submitted to Apple for distribution on the App Store may use only Documented APIs included in the default installation of macOS, as bundled with Xcode and the Mac SDK, or as bundled with Swift Playground; deprecated technologies (such as Java) may not be used."

**§3.3.3(A) Recordings:**

> "If Your Application captures or makes any video, microphone, screen recordings, or camera recordings, whether saved on the device or sent to a server […] a reasonably conspicuous audio, visual or other indicator must be displayed to the user as part of the Application to indicate that a Recording is taking place. Your Application may not be designed to facilitate Recordings of others without their awareness."
>
> "In addition, any form of data, content or information collection, processing, maintenance, uploading, syncing, storage, transmission, sharing, disclosure or use performed by, through or in connection with Your Application must comply with all applicable privacy laws and regulations as well as any related Program Requirements, including but not limited to any notice or consent requirements."

**§3.3.3(B) Collection and Use of Data:**

> "You and Your Applications […] may not collect user or device data without prior user consent, whether such data is obtained directly from the user or through the use of the Apple Software, Apple Services, or Apple SDKs, and then only to provide a service or function that is directly relevant to the use of the Application […]."

**§3.3.3(C):**

> "You must provide a privacy policy in Your Application, on the App Store, and/or on Your website explaining Your collection, use, disclosure, sharing, retention, and deletion of user or device data."

Even for a direct-download app, ship a privacy policy, per-source consent, a way to withdraw consent, and a clear indicator while activity is being recorded. Do not capture keystrokes in password fields or other secret input (Review Guideline 5.1.1(vi) can remove a developer from the Program, not only from the store).

---

## 5. Comparable apps and how they distribute

Fetched November 14, 2026. These are product pages, not Apple policy, but they show how similar watchers actually ship.

### Timing (automatic Mac time tracker)

- **Distribution:** direct download of a DMG, not the Mac App Store. Homepage download URL: `https://downloads.timingapp.com/Timing.dmg` ([timingapp.com](https://timingapp.com/)).
- Official pricing FAQ, [https://timingapp.com/pricing?lang=en](https://timingapp.com/pricing?lang=en), question "Is Timing available on the Mac App Store?":

  > "No, and we have no plans right now to change that. The Mac App Store's sandboxing rules would restrict the details Timing can track for you, resulting in a worse experience."

- Permissions documented by Timing:
  - Accessibility for the tracker ([FAQ](https://timingapp.com/help/faq): "Why is Timing asking me to enable Accessibility support?").
  - Full Disk Access to read the Screen Time database ([Screen Time Integration](https://timingapp.com/help/screen-time): "Turn on Timing in System Settings > Privacy & Security > Full Disk Access. This is required for Timing to read your Screen Time database.").
  - Optional Screen Recording only to read some window titles, not to record the screen (same FAQ).
  - The Screen Time importer "may stop working if Apple changes how Screen Time stores its data in a future iOS or macOS update."

### RescueTime

- **Mac desktop app:** direct download of a zip / `RescueTime.app`, dragged to Applications. Official help, last updated June 12, 2026: [How to download and install RescueTime for macOS](https://help.rescuetime.com/article/354-how-to-download-and-install-rescuetime-for-macos). "Download the RescueTime Assistant from the website."
- **App Store listing found is iPhone/iPad only**, not Mac: [RescueTime App - App Store](https://apps.apple.com/us/app/rescuetime/id1629924958) ("Compatibility: Requires iOS 17.4 or later" / iPadOS; the listing itself says to "Install RescueTime on your desktop" for the full workday picture). The iOS app can use optional Screen Time permission; that is not a Mac App Store desktop watcher.

### Rewind

- **Distribution:** direct DMG from [https://rewind.app/download](https://rewind.app/download).
- The download page states: "Install Rewind Desktop." System requirements include "Screen Recording permission" and "Accessibility permission." Installation: "Open the downloaded DMG file" / "Drag Rewind Desktop to Applications." No Mac App Store link on that page.

### Screenpipe (local-first Rewind-style app)

- **Distribution:** direct `.zip` / `.dmg`, drag to Applications. [Download and install screenpipe](https://screenpipe.com/how-to-install).
- The page lists Mac permissions: "On Mac, these can include Screen & System Audio Recording, Microphone, and Accessibility. Input Monitoring is optional." No Mac App Store link on that page.

**Pattern:** the Mac apps that automatically watch other apps, window titles, or Screen Time data ship outside the store, signed for identified-developer / notarized install, and walk the user through TCC settings. Store listings in this category, when they exist, are iOS companions or narrower manual timers.

---

## 6. Bottom line

### Can the full autoautomate app pass Mac App Store review?

**Almost certainly no**, based on current official rules fetched today.

A store build must be sandboxed (2.4.5(i)), stay in its container (2.5.2), and use public APIs (2.5.1). Apple's sandbox documentation forbids accessibility APIs in assistive apps and Apple Events to arbitrary apps. The interesting data sources — frontmost window titles via AXUIElement, browser history files, Screen Time's `knowledgeC.db`, Messages, Mail, Notes, Shortcuts, recent files, and shell history — are not available as documented public "read this store" APIs on native macOS. Device Activity / Screen Time APIs are documented for iOS, iPadOS, and Mac Catalyst, not native macOS. Optional Input Monitoring is an additional 2.5.14 and 5.1.1(vi) risk.

### Which subset could be tried in the store?

A **cut-down companion**, only if it remains useful without watching other apps:

- EventKit Reminders and Calendar (sandbox entitlement + TCC purpose strings).
- Notifications.
- Launch-at-login with SMAppService and explicit consent.
- Features that use only the app's own container (labeling, ranking, local model on data the user imported).
- Possibly clipboard *if* limited, consented, and obviously indicated — **unverified** with App Review.
- Opt-in publish of a user-previewed anonymized summary, with 5.1.2-style consent.

That subset **cannot** deliver "watch how you work." It should not be v1 if the product promise is automatic task discovery.

### ⭐ Recommended release path

**Developer ID–signed, notarized direct download** (the developer's website and/or GitHub Releases), with Homebrew as an optional later channel.

Why this is the recommendation:

1. Current guidelines and sandbox docs block the core watching features in the Mac App Store.
2. Apple explicitly allows macOS apps to be distributed outside the store with Developer ID certificates and notarization tickets (DPLA §3.2(g), notarization documentation).
3. The user already has a Developer ID. The remaining work is Hardened Runtime, `notarytool` (or Xcode Organizer → Developer ID), stapling, and a clear permissions onboarding flow.
4. On macOS Sequoia and later, un-notarized software can no longer be opened with Control-click; users would have to use System Settings → Open Anyway. Notarizing avoids that.
5. Timing, RescueTime's Mac assistant, Rewind, and Screenpipe all ship this way. Timing says outright that Mac App Store sandboxing would make tracking worse, and that they have no plans to change that.
6. App Store updates would also be the only legal update path for a store build (2.4.5(vii)); a notarized build can update itself or via Releases, which is a better fit for a rapidly evolving pre-1.0 tool.

**Ship checklist (direct download):**

- Sign with Developer ID Application; enable Hardened Runtime; notarize with `notarytool` or Xcode; staple the ticket to the app and to the DMG.
- Every data source off until the user enables it. Explain Accessibility, and Full Disk Access only for the sources that truly need it. Do not request Screen Recording or Input Monitoring unless a specific feature needs them.
- Visible "watching is on" indicator and per-source consent (aligns with 2.5.14 and DPLA 3.3.3).
- Privacy policy in the app and on the site; preview-and-confirm before any share link is published; no raw activity leaves the Mac otherwise.
- Never capture password or other secret input if Input Monitoring is ever offered.
- Do not claim Apple "reviewed" or "approved" the app just because it is notarized (DPLA §5.3).

A later, optional Mac App Store "lite" build remains possible for EventKit-only or import-only features. Do not plan v1 around getting the full watcher into the store.

---

## Items checked and not found / not independently verified

- The exact phrase **"apps may not request Full Disk Access"** is **not** in the October 16, 2026 Review Guidelines.
- No official Apple statement was found that Full Disk Access overrides App Sandbox for a Mac App Store app.
- No official public native-macOS API was found for Screen Time usage (`knowledgeC.db`), Safari/Chrome history, Messages, or Mail metadata.
- No official requirement was found that a non-App-Store, Developer ID-notarized Mac app must include a privacy manifest.
- No newer notarization tooling cutoff than November 1, 2023 (`altool` / Xcode 13) was found on today's notarization pages.
- Temporary-exception entitlements as a path to App Store approval for these data sources were **not verified** in current (non-archive) policy.
- Whether App Review would accept a sandboxed, consented clipboard-history feature was **not verified**.
- This is not legal advice and not a guarantee of App Review or notary outcomes. Apple can change guidelines and can revoke Developer ID tickets.

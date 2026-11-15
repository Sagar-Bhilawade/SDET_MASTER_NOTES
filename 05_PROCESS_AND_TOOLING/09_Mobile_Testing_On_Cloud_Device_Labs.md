# Mobile Testing On Cloud Device Labs (LambdaTest)

## 1. What it is

Mobile testing checks an app across the many devices, operating systems and screen sizes users have. A
**cloud device lab** such as **LambdaTest** gives remote access to real devices and emulators/simulators
through the browser, so a team doesn't need to buy and maintain a rack of phones.

My experience: **manual** testing of **Android (APK)** and **iOS (IPA)** builds on LambdaTest across
**phones and tablets — Android devices, iPhones and iPads**. I have **not** done mobile automation
(no Appium), and I say so.

## 2. Why it matters in interviews

Many SDET roles list "mobile testing". Interviewers want to know whether I can plan a device matrix,
know what is mobile-specific beyond "a small screen", and understand the trade-offs of real devices vs
emulators vs cloud labs — and whether the claim is manual or automated.

## 3. 30-second answer

"I've tested Android APK and iOS IPA builds manually on LambdaTest's cloud devices — phones and
tablets, including iPhones and iPads. I upload the build, pick devices from a matrix based on OS versions and screen
sizes, and test installation, functional flows, UI and layout on different screen sizes and orientations,
network conditions, permissions and interruptions, capturing screenshots, video and logs for defects. My
mobile work is manual; I know how Appium fits for automation but I haven't used it."

## 4. 2-minute answer

**Mobile-specific test areas**

| Area | What I check |
|---|---|
| **Installation** | Install from APK/IPA, first launch, upgrade over an older version, uninstall, data retained/cleared |
| **Functional** | Core flows, login, forms, navigation, back-button behaviour (Android), gestures (tap, swipe, pinch, long-press) |
| **UI / responsive** | Different screen sizes and resolutions, tablets (incl. iPad), orientation changes, notches/safe areas, text scaling, dark mode, truncation/overlap |
| **Compatibility** | OS versions (Android fragmentation: many versions and manufacturer skins; iOS: fewer versions, quick adoption), device models |
| **Permissions** | Location, camera, notifications, storage; allow/deny/"only this time"; behaviour after denial |
| **Interruptions** | Incoming call, notifications, low battery, switching apps, backgrounding and returning, screen lock |
| **Network** | Wi-Fi vs mobile data, slow/unstable network, offline, switching networks mid-action, timeouts/retry messages |
| **Performance** | Launch time, scrolling smoothness, memory/battery drain symptoms, large data lists |
| **Accessibility** | Screen reader labels, contrast, touch-target size |
| **Localisation** | Languages, date/number formats, right-to-left |
| **Security basics** | HTTPS use, sensitive data in logs/screens, session expiry, behaviour on jailbroken/rooted (conceptual) |
| **App-store readiness** | Version/build number, release notes, store guidelines |

**Android vs iOS differences worth knowing**
- *Android:* APK/AAB, high fragmentation (device makers, OS versions, screen sizes), hardware back
  button, different permission dialogs per OS version, logs via `adb logcat`.
- *iOS:* IPA, strict signing/provisioning (development/ad-hoc/TestFlight distribution), fewer devices but
  precise guidelines, logs via Xcode console. A build must be signed for the device/lab to install it.

**Real devices vs emulators/simulators vs cloud**

| | Real device (own) | Emulator/Simulator | Cloud real devices |
|---|---|---|---|
| Fidelity | Highest | Lower (no real sensors, performance differs) | High |
| Cost/effort | Buy, maintain, update | Free, fast | Subscription, no hardware |
| Coverage | Limited to what you own | Many configs | Wide device/OS matrix |
| Good for | Final checks, hardware features | Early dev, quick checks | Compatibility across many devices |
Trade-off: cloud devices are shared and network-dependent (latency, queueing), and some hardware features
(real camera/NFC/biometrics) are limited.

**Choosing the device matrix.** Don't test everything: take the app's analytics (top devices/OS
versions), add the oldest supported OS and the newest, cover phone and tablet sizes, both platforms, and
known problem devices from past bugs. Re-evaluate each release.

**What LambdaTest gives me (cloud lab features)**: upload APK/IPA, choose real devices/OS versions, live
interactive session, screenshots and video recording, device and app logs, network throttling and
geolocation options, and issue/bug capture. Exact features vary by plan — I describe them as "what I used", not
a feature list I haven't touched.

**A good mobile defect report:** app version/build, device model, OS version, platform (Android/iOS),
steps, expected vs actual, frequency, network state, screenshot/video, log excerpt (logcat or console),
severity and whether it reproduces on other devices (device-specific vs general).

**Manual vs automated mobile.** Manual is the right tool for exploratory, visual and device-behaviour
checks; automation (Appium for Android/iOS UI, plus native frameworks like Espresso/XCUITest, or cloud
grids) suits regression on a stable flow. Where I'd start: keep a small, stable smoke path automated and
leave exploratory, gesture and layout checks manual. I know the concept — not hands-on Appium.

**Link to my API work.** Mobile apps are thin clients over APIs. I tested the app's APIs too — functional
checks and JMeter load (standard vs peak, P90/P95) — so I can cover both the on-device and the backend
sides of a mobile release.

## 5. Real project example (true statements)

At NielsenIQ I tested **Android APK and iOS IPA** builds on **LambdaTest**, manually, on a **range of
devices** — Android phones and tablets, iPhones and iPads — and on the backend side load-tested the
mobile application's **APIs with JMeter**. Be ready with one mobile-specific bug you really saw, described
generically without confidential detail. Fill this in from memory (one line each):
*Device + OS: ___ ; what I did: ___ ; what went wrong: ___ ; how I proved it was device-specific: ___ ;
what happened to it: ___.*

**Typical mobile bugs, to speak about as examples ("for example...", not "I found")**
- Layout truncation or overlap on a tablet or iPad (landscape orientation, larger text size).
- Content hidden behind the notch or the bottom safe area on newer iPhones.
- A permission denied once, and the screen then shows a blank state with no way to re-enable it.
- App state lost when the app is backgrounded and restored (a form, a session, a scroll position).
- Back-button behaviour on Android differing from iOS swipe-back (data loss or wrong screen).
- Login/session expiring on a slow network or after a network switch, with an unclear error message.
- Crash on an older OS version only; reproducible on one device model but not others.
- Keyboard covering the input field; date/number format wrong in another locale.
Say which were "for example" and which were mine. A believable honest answer beats a dramatic
one.

## 6. Common interview Q&A

**Q: Have you done mobile automation?** No — my mobile testing is manual on cloud devices. I know how
Appium works (server, drivers, capabilities, locators, same WebDriver concepts as Selenium) and would
start with a small stable smoke suite.

**Q: How do you choose which devices to test?** From usage analytics: top devices and OS versions,
oldest/newest supported, phone and tablet sizes, and past problem devices.

**Q: Real device or emulator?** Emulators for early, fast checks; real devices (own or cloud) for final
verification, performance and hardware behaviour.

**Q: What's different about iOS testing?** Signed IPA builds (provisioning/TestFlight), fewer devices,
stricter UI guidelines, and different permission and keyboard behaviours.

**Q: What mobile-specific scenarios do you test beyond functionality?** Installation/upgrade,
interruptions, permissions, network changes, orientation, backgrounding, and performance feel.

**Q: How do you report a device-specific bug?** Model + OS + build, steps, screenshot/video/log, and whether
it reproduces on other devices.

**Q: What are the pros and cons of cloud device labs?** Wide coverage and no hardware cost vs shared
devices, network dependence, and limited access to some hardware features.

**Q: How would you test a mobile app's backend?** API functional tests, JMeter load tests for standard vs
peak, and auth/token handling — which I did.

## 7. Weak areas to watch

- Implying mobile automation or Appium experience. Don't.
- Calling emulators "real devices".
- Giving a made-up device list. Use the matrix method and say what you actually covered: phones, tablets,
  iPhones, iPads.
- Forgetting the APIs behind the app.

## 8. Key takeaway

Manual mobile testing on cloud devices = a smart device matrix, mobile-specific checks (install,
interruptions, permissions, network, orientation), clear device-specific defect reports — plus honesty
that automation is the next step, not today's claim.

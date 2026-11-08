# Build Unlit in React Native

You're building **Unlit**, an AI quit-smoking coach for iOS and Android, in React Native. The design is finished and approved. Your job is to turn it into a production-quality app that matches the design closely, with clean architecture we can ship and grow.

## 0. Design reference (read this first)

I've put the exported design in `./design-reference/` (unzip it there if it isn't unzipped yet). The source of truth is `design-reference/project/Main.dc.html`. Ignore `canvas.json` apart from the board size.

That file is in a custom "Design Component" format, not plain HTML. Here's how to read it:

- Everything inside `<x-dc>` is the markup. Inline `style="..."` values are the exact spacing, sizes, radii, weights and colors to match.
- The `<helmet><style>` block holds the design tokens (CSS variables under `.app.light`, `.app.dark` and `.calm`), the component classes (`.primary`, `.secondary`, `.chip`, `.opt`, `.card`, `.soft`, `.crave`, `.int`, `.sw`, `.seg`, `.bubble` and so on) and every animation keyframe. Turn these into a typed theme and shared components.
- `<sc-if value="{{x}}">` is a conditional render and `<sc-for list="{{xs}}" as="x">` is a loop. `{{path}}` is a value bound from `renderVals()`.
- The `<script type="text/x-dc">` block at the bottom has `class Component extends DCLogic`. It contains **all of the app's behavior, mock data and final copy**: the screen state machine, the quiz state, the craving flow steps, the exercise definitions (title, length, why, guiding lines), the slip plans, the coach's seeded chat and keyword reply logic, the progress data and the money math. Port that logic faithfully. Don't rewrite the copy. Lift every user-facing string from this file exactly as written.
- The "Demo controls" `<aside>` is a prototype tool, not part of the app. Rebuild it as a **dev-only debug menu** (shake gesture or a hidden long-press, only in `__DEV__`).

Before you write any code, read the whole file and reply with: (1) the full screen list and how screens connect, (2) the extracted tokens as a TypeScript theme object, and (3) any part of the design you find ambiguous. Then wait for my go-ahead.

## 1. Product in one line

An AI coach built on Acceptance and Commitment Therapy (ACT), with one job: get a smoker to day 42 smoke-free. It's calm, adult and never judges. No gamification, mascots, confetti or streak fire. Progress should feel like recovery.

The plan has 3 phases: **Prepare** (days 1 to 14, keep smoking but log everything), **Quit day** (day 15, a short ceremony) and **Protect** (days 15 to 42).

## 2. Tech stack

- **Expo** (latest stable SDK) with **TypeScript in strict mode**, using **Expo Router** for file-based navigation.
- **react-native-reanimated** and **react-native-gesture-handler** for all animation and pressed states. No `Animated` from core.
- **react-native-svg** for icons, the progress ring, the wave, the timeline and the charts. Copy the inline SVG paths from the reference. Don't add an icon font.
- **Zustand** for state, persisted with **react-native-mmkv** (or AsyncStorage if MMKV causes trouble with Expo Go). State has to survive app restarts.
- **expo-font** with **Manrope** (400, 500, 600, 700, 800), the only typeface.
- **expo-haptics**: light impact on the main buttons, soft impact as the wave crests, success haptic on "Yes, it passed".
- **expo-notifications** for local scheduled check-ins.
- **RevenueCat** (`react-native-purchases`) for the paywall, behind an interface so it can be mocked in dev.
- **Jest + React Native Testing Library** for tests.

If you think any of these is the wrong call, say so and explain why before you swap it.

## 3. Design system

Build `src/theme/` from the helmet CSS. Use these exact values:

**Light:** bg `#FAF7F2`, card `#FFFFFF`, card2 `#F2EDE5`, text `#1F1D1B`, muted `#655F58`, line `#E4DCD1`, accent `#E8603C`, accentText `#B5421E`, accentSoft `#FBE8E0`, chipOn `#1F1D1B`, chipOnText `#FAF7F2`

**Dark:** bg `#141312`, card `#201E1C`, card2 `#2A2725`, text `#F3EEE7`, muted `#A9A197`, line `#36312D`, accent `#E8603C`, accentText `#F2906F`, accentSoft `#3A241C`, chipOn `#F3EEE7`, chipOnText `#141312`

**Calm** (used on the Craving and Quit day screens in both modes): bg `#1D1A18`, card `#2A2623`, card2 `#34302B`, text `#F4EEE6`, muted `#B5AB9F`, line `#463F39`

Rules:

- Ember `#E8603C` is used sparingly: the main action on each screen, the Craving button, the quit-day marker, the peak bar on the chart and the wave. Selected chips use chipOn (charcoal), never orange.
- White text on ember only at 19px bold or larger, so it passes contrast as large text.
- Appearance can be System, Light or Dark (the design shows Light and Dark; add System as the default).
- Shared components: `PrimaryButton`, `SecondaryButton`, `GhostButton`, `Chip`, `OptionRow` (radio card), `Card`, `SoftCard`, `IntensityPicker` (1 to 5), `Toggle`, `SegmentedControl`, `ListRow`, `CravingButton`, `TabBar`, `Toast`, `ScreenHeader`. Every pressable scales to 0.97 and fades slightly when pressed, using Reanimated.
- Screens enter with a 340ms fade and 10px upward slide, matching the `scrIn` keyframe.
- Users are 30 to 55, so support Dynamic Type up to at least the XL size without breaking layouts. All touch targets are 44pt or more. Add accessibility labels on every icon-only button.

## 4. Navigation and screens

Bottom tabs: **Home, Coach, Progress, Profile**. The **Craving button** sits above the tab bar on Home, Coach and Progress (not Profile). Everything else is a stack or full-screen modal.

1. **Welcome**: headline, Get started, I already have an account
2. **Sign in**: phone number, Continue (stub the auth for now behind an `AuthService` interface)
3. **Onboarding quiz** (5 steps with a progress bar): cigarettes per day (slider 1 to 40, show monthly spend at ₹18 each), years smoked (stepper), triggers (multi-select chips), past attempts (0, 1-2, 3+), main reason (health, family, money, fitness, other)
4. **Your plan**: 3-phase timeline, quit date, day 42 date, money saved by day 42 in ₹ and $. All of it is computed from the user's start date and answers. Nothing is hardcoded.
5. **Medication**: plain-language options, the "Talk to a pharmacist or doctor before starting. We never give dosing advice." note, 3 choices, saved to the profile
6. **Buddy invite**: name and phone, a preview of the milestone message, Skip
7. **Paywall**: 7-day free trial, $59.99/year (Best value, selected by default) or $9.99/month, 4 benefits, Restore purchase
8. **Home (Prepare)**: greeting, days-until-quit-day ring, today's task card, Log a cigarette, next check-in card, quit day preview
9. **Home (Protect)**: days smoke-free, cigarettes not smoked, money saved, health milestone card, next check-in, an "I had a cigarette" link
10. **Log a cigarette**: trigger, intensity 1 to 5, optional note. Saves and returns home with the toast "Logged. Every log teaches your coach your patterns."
11. **Craving flow** (full-screen modal, calm theme, the most important flow in the app): intro → rate strength and triggers → coach suggestion plus 4 alternatives → exercise player → "Did the craving pass?" → passed / still strong / smoked
12. **Slip**: no guilt, progress is never reset, asks what happened, shows a tailored 3-step plan
13. **Coach chat**
14. **Quit day ceremony**: one-sentence reason, "I'm done", the transition, then Protect mode
15. **Progress**: 42-day journey bar, cravings-by-time-of-day chart, top triggers, stats. The content changes by phase.
16. **Profile/Settings**: check-in toggles, buddy, medication, subscription, privacy, appearance, medical disclaimer, sign out
17. **Privacy and data**: export data, delete account (with a real confirmation step)

**Bet on yourself is out of scope.** Build only the "Coming soon" screen with the notify-me button, behind a feature flag. Real deposits need legal review first.

## 5. The craving flow (polish this most)

Port the exercise definitions from the reference exactly: `surf` (3 min), `unhook` (2 min), `values` (2 min), `breath` (2 min) and `delay` (10 min), each with its guiding lines.

- **Suggestion logic** (keep it as a pure, unit-tested function): strength 4 or more → Urge surfing; stress selected → Slow breathing; strength 3 → Unhook; otherwise → 10-minute delay.
- **Timer**: measure elapsed time from a start timestamp, not by counting interval ticks, so it stays accurate when the app is backgrounded. Support pause and resume. Keep the screen awake during the exercise (`expo-keep-awake`).
- **Guiding text** changes every 20 seconds and crossfades over 1.2 seconds. On the 10-minute delay the lines loop.
- **Urge surfing wave**: two layered SVG waves scrolling sideways at different speeds (8s and 13s loops), a slow vertical bob (5s), and a baseline that follows the urge curve. The wave rises to a peak at about 45% of the duration and falls back to low by the end, using `sin(π · min(1, p / 0.88))`. Show a Rising / Peaking / Passing label. All of it runs on the UI thread with Reanimated. Pausing freezes everything.
- **Slow breathing**: a circle that grows over 4 seconds (breathe in) and shrinks over 6 seconds (breathe out), with the label synced to the same clock.
- **Other exercises**: a progress ring around a softly pulsing circle. Values shows the user's reason in the center.
- **End screen**: "Yes, it passed" adds one to cravings handled and shows the success screen. "Still strong" offers the exercises not tried yet in this session, plus Chat with my coach. "I smoked" goes to Log a cigarette during Prepare (with the reassurance note) and to the Slip screen during Protect.
- Log every craving event (strength, triggers, exercise, outcome, timestamp). This feeds the Progress screen and, later, the coach.

## 6. The AI coach

Build it in two layers so we can ship before the backend exists:

1. **`CoachService` interface** with a `ScriptedCoach` implementation that ports the reference's seeded conversation, quick replies and keyword replies exactly. Include the typing indicator, a 1.5s reply delay and auto-scroll to the newest message.
2. **`LLMCoach` implementation** that calls our backend at `POST /coach/message` with streaming. **Never put an API key in the app.** Write a minimal backend in `/server` (Node + TypeScript, Hono or Express) that proxies to the Claude API with:
   - a system prompt grounded in ACT: acceptance, defusion, values and committed action. Short, warm, direct, adult replies. Never shame the user.
   - the user's context: phase, day, triggers, reason, medication choice, recent cravings and slips.
   - **hard safety rules**: no dosing, combination or timing advice for any medication or nicotine replacement, ever. Point the user to a pharmacist or doctor. If the user mentions chest pain, a medical emergency, self-harm or suicidal thoughts, the coach stops coaching and gives emergency and crisis resources for the user's country (India first).
   - a **safety check on every reply before it reaches the user**, plus a client-side keyword check. A reply that refuses dosing advice shows the "Safety · No dosing advice" tag above the bubble, as in the design.

Write tests for the safety rules: at least 15 dosing questions phrased in different ways that must all be refused.

## 7. Data, notifications and payments

- **Local-first data model** (persisted): profile (cigarettes per day, years, triggers, attempts, reason, medication, buddy), plan (start date, quit date, phase), logs, cravings, slips, chat history, settings and subscription state.
- **Derived values are computed, never stored**: current day, days until quit day, days smoke-free, cigarettes not smoked, money saved (₹18 a cigarette and ₹88 to the dollar, both in a config file), progress percentage, cravings by time of day and the top triggers.
- **Slips never reset** the smoke-free count shown to the user. Track them separately.
- **Notifications**: schedule local check-ins at the user's toggled times. Default to 8:00 am, 1:55 pm and 6:30 pm, and later adjust them to the times their logs show are hardest. Tapping a check-in opens the Coach tab. Ask for notification permission after the plan screen, with a one-line reason, not on first launch.
- **Paywall**: RevenueCat with a `MockPurchases` adapter for dev. Gate the Coach chat and the full program after the trial. **The Craving button always works, even without a subscription.** Leaving someone alone mid-craving is not acceptable.
- **Buddy**: store the details only for now. Put the SMS milestone sending behind a `BuddyService` stub.

## 8. Build order

Work in phases. After each one, stop, summarize what you built, list what's left and wait for me.

1. Scaffold, theme, fonts, shared components, a component gallery screen (dev only)
2. Navigation shell, tabs, the Craving button, the store and persistence
3. Onboarding: Welcome, Sign in, quiz, plan, medication, buddy, paywall (mocked)
4. Home in both phases, Log a cigarette, toasts
5. The full craving flow and all 5 exercises
6. Slip, Quit day, the switch into Protect
7. Progress and Settings, privacy, dark mode, the dev debug menu
8. Coach chat with `ScriptedCoach`
9. Notifications, RevenueCat, the `/server` coach proxy and `LLMCoach`
10. Accessibility pass, tests, a performance check on a mid-range Android phone

## 9. Done means

- Every screen and flow in `Main.dc.html` works on iOS and Android, with copy matching word for word.
- Side-by-side with the reference, spacing, type sizes, radii and colors match at 393×852. Explain any deliberate difference.
- Light and dark mode are correct on every screen. The calm theme is used on Craving and Quit day.
- The wave, breathing and timer animations run at 60fps and keep correct time through backgrounding.
- No hardcoded user data outside the seed and mock files. Rahul (34, 12 a day) is only the dev seed.
- Tests pass for the suggestion logic, the derived-stats math, the slip non-reset rule and the coach safety rules.
- A README covering setup, environment variables, how to run the server and how to switch from the mocks to the real services.

The "This app does not replace medical advice" note stays in Settings, and the medication screen keeps its pharmacist/doctor note. Don't remove or soften either one.

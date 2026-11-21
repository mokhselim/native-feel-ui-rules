---
name: native-feel-ui-rules
description: Ten UI/UX rules that make a mobile app feel native on iPhone and Android (skeleton loaders, optimistic UI, tabular numbers, squircle corners, swipe-to-dismiss sheets, shared element transitions, SMS code autofill, undo instead of confirm, balanced line breaks, blur placeholders). Use whenever building or reviewing any screen, list, card, form, sheet, image, number display, delete action, loading state or navigation in a K-UBE app (Flutter, SwiftUI, React Native or web).
---

# Native-feel UI rules

Source and credit: [@jaimintf](https://x.com/jaimintf), from his post ["stop making people relearn their iphone. give your ai agent these 10 rules."](https://x.com/jaimintf/status/2123786458721116225). Watch the videos there.
Goal: the app should behave the way iOS (and modern Android) users already expect, so nobody has to relearn anything.

## How to use this skill
- Apply these rules **by default** whenever you build or touch a matching UI pattern. Don't wait to be asked.
- When reviewing an existing screen, check it against the table below and fix any violations in the same change.
- Use the platform's native mechanism first; only hand-roll an effect if the framework has no built-in.
- Never break accessibility: respect Reduce Motion (skip shimmer, springs and morph transitions, use a simple fade), keep Dynamic Type / text scaling working, and keep tap targets at least 44x44 pt (48x48 dp on Android).

## Quick checklist
| # | Pattern you're building | Rule | Never do |
|---|---|---|---|
| 1 | Any screen or list that loads data | Skeleton loader in the shape of the real content | A centered spinner in an empty screen or card |
| 2 | Like, save, follow, toggle, add-to-list, check-off | Optimistic UI: update instantly, save in background | Spinner inside the button while waiting for the server |
| 3 | Prices, balances, counters, timers, calories, stats | Tabular (fixed-width) digits | Proportional digits that jitter as values change |
| 4 | Cards, buttons, images, sheets, app-icon-like shapes | Squircle / continuous corners (about 60% smoothing) | Plain circular-arc border radius |
| 5 | Bottom sheets, modals, pickers | Drag down to dismiss, with a grab handle | A sheet that can only close via an X button |
| 6 | Tapping a card/thumbnail to open its detail | Shared element transition: the card grows into the page | A hard cut or generic slide push |
| 7 | OTP / verification code fields | One-tap SMS code autofill from the keyboard | Making users type the code digit by digit |
| 8 | Delete, archive, remove, clear | Act immediately and show an Undo toast | A blocking "Are you sure?" dialog |
| 9 | Headlines, titles, short paragraphs, buttons | Balanced line breaks, no single orphan word on the last line | One word stranded alone on the last line |
| 10 | Remote images (photos, hero images, avatars) | Blurred preview (BlurHash/ThumbHash) that crossfades to the photo | An empty gray box with an image icon |

---

## 1. Skeleton loaders
**Rule:** Show the page's shape while content loads.
- Draw light-gray placeholder blocks with rounded corners that match the real layout exactly: avatar circle, text lines (vary widths, the last line shorter), image block, buttons.
- Add a subtle, continuous left-to-right shimmer. Disable shimmer under Reduce Motion; show static blocks instead.
- When data arrives, crossfade quickly (about 150 to 250 ms) from skeleton to content. Don't let the layout jump: skeleton and content must be the same size.
- Show only as many skeleton rows as fit on screen.
- Spinners are only OK for tiny inline actions (for example inside a "Load more" footer), never for a whole screen or card.
- Flutter: `skeletonizer` or `shimmer` package. SwiftUI: `.redacted(reason: .placeholder)` plus a shimmer overlay. React Native: `react-native-reanimated` shimmer or `moti` skeleton.

## 2. Optimistic UI
**Rule:** Update the screen on tap and save in the background.
- On tap, change the UI state immediately: fill the heart, bump the count, check the box, add the item.
- Give instant feedback: a quick bouncy spring scale (about 1.0 to 1.2 to 1.0) plus a light haptic (iOS `UIImpactFeedbackGenerator(.light)`, Flutter `HapticFeedback.lightImpact()`).
- Send the request in the background. If it fails, roll back to the previous state quietly and show a short non-blocking message ("Couldn't save. Try again.").
- Debounce rapid repeat taps so the last state wins, and never let the button freeze or show a spinner.
- Use for low-risk, reversible actions only. Payments, purchases and anything irreversible still wait for the server.

## 3. Tabular numbers
**Rule:** Fixed-width digits stop numbers from jumping.
- Any number that changes in place (balances, prices, counters, timers, countdowns, calorie and macro totals, prayer-time countdowns, progress percentages, stats tables) must use tabular figures.
- Also use them in columns of numbers so decimals line up.
- SwiftUI: `.monospacedDigit()`. UIKit: `UIFont.monospacedDigitSystemFont`. Flutter: `TextStyle(fontFeatures: [FontFeature.tabularFigures()])`. CSS: `font-variant-numeric: tabular-nums`. React Native: `fontVariant: ['tabular-nums']`.
- For animated counters, combine with a numeric content transition (SwiftUI `.contentTransition(.numericText())`).

## 4. Squircle corners
**Rule:** Use continuous corners (corner smoothing about 60%) to match iOS.
- A plain border radius turns from straight line to arc abruptly. iOS uses a continuous curve that eases in.
- Apply to cards, buttons, sheets, image thumbnails, input fields, chips and icon tiles. Fully round pills and circles stay as they are.
- SwiftUI: `RoundedRectangle(cornerRadius: r, style: .continuous)` or `.clipShape(.rect(cornerRadius: r, style: .continuous))`. UIKit: `layer.cornerCurve = .continuous`. Flutter: `figma_squircle` (`SmoothRectangleBorder` / `SmoothBorderRadius(cornerRadius: r, cornerSmoothing: 0.6)`). Figma: corner smoothing 60%. Web: SVG mask or `corner-shape: squircle` where supported.
- Keep radii consistent across the app (define them as design tokens, for example 8, 12, 16, 24) and make nested radii concentric (inner radius = outer radius minus padding).

## 5. Swipe to dismiss
**Rule:** Let people drag a sheet down to close it.
- Every bottom sheet, modal and picker shows a small gray grab handle (pill) at the top and follows the finger 1:1 when dragged down.
- On release, dismiss if dragged past about 25 to 30% of its height or flicked down fast; otherwise spring back. Use an ease-out or spring, never a linear slide.
- Keep a tap on the dimmed backdrop as a second way to close. Keep an explicit Close/Done button only where accessibility or content requires it.
- If the sheet has unsaved input, block the dismiss and ask, or keep a draft.
- If the sheet scrolls, dragging down only dismisses once the content is scrolled to the top.
- SwiftUI: `.sheet` with `.presentationDetents` and `.presentationDragIndicator(.visible)`. Flutter: `showModalBottomSheet(showDragHandle: true, enableDrag: true)` or `DraggableScrollableSheet`; for iOS feel use `CupertinoSheetRoute` / `modal_bottom_sheet`. React Native: `@gorhom/bottom-sheet`.

## 6. Shared element transitions
**Rule:** The tapped card grows into the next screen.
- When a card, thumbnail or list row opens a detail page, morph the shared element (usually the image, sometimes the title) from its spot in the list into its spot on the detail page.
- Keep the aspect ratio, scale and move it with a smooth spring (about 350 to 500 ms), and fade the rest of the detail content in below it. Going back reverses the morph into the same card.
- Under Reduce Motion, fall back to a simple crossfade.
- SwiftUI: `.matchedGeometryEffect` or iOS 18 `.navigationTransition(.zoom(sourceID:in:))` with `.matchedTransitionSource`. Flutter: `Hero` widgets with matching tags (plus `flightShuttleBuilder` for rounded corners) or the `animations` package `OpenContainer`. React Native: Reanimated shared transitions. Android: Compose `SharedTransitionLayout`.
- Use stable unique tags (item id), never list index.

## 7. SMS code autofill
**Rule:** Let the keyboard fill the code in one tap.
- Every OTP or verification field must support system autofill so the code appears above the keyboard ("From Messages 123456") and fills every box with one tap.
- Use a numeric keyboard, auto-focus the field, auto-advance across boxes, allow paste of the whole code, and auto-submit once all digits are in.
- SwiftUI/UIKit: `.textContentType(.oneTimeCode)` and `.keyboardType(.numberPad)`. Flutter: `autofillHints: [AutofillHints.oneTimeCode]` inside an `AutofillGroup` (or `pinput`, which supports it); on Android use the SMS User Consent / Retriever API (`smart_auth`). React Native: `textContentType="oneTimeCode"` and `autoComplete="sms-otp"`. Web: `autocomplete="one-time-code"` and `inputmode="numeric"`.
- For email codes, use the same field setup (iOS also suggests codes from Mail).

## 8. Undo instead of confirm
**Rule:** Delete right away and show an Undo button.
- For reversible destructive actions (delete a meal entry, remove a saved place, archive, clear a list item), remove the item immediately with a smooth collapse animation and show a non-blocking toast/snackbar at the bottom: "Meal deleted" with an **Undo** button.
- The toast slides up with an ease-out, stays about 4 to 6 seconds, then auto-dismisses. Only then (or when the app goes to background) commit the delete to the server. Undo restores the item in its original position.
- Support swipe-to-delete on list rows where that's natural.
- Keep a real confirm dialog only for truly irreversible, high-stakes actions: deleting an account, cancelling a subscription, deleting all data, payments.
- Flutter: `ScaffoldMessenger.showSnackBar` with a `SnackBarAction(label: 'Undo')`, `Dismissible` for swipe. SwiftUI: custom bottom toast overlay with a timer. React Native: a toast library with an action button.

## 9. Balanced line breaks
**Rule:** Don't leave one word alone on the last line.
- Headlines, titles, onboarding and paywall copy, empty states, card titles and button labels must not end with a single orphan word.
- Prefer balanced wrapping so lines are roughly equal length ("Your daily plan, / ready in seconds", not "Your daily plan, ready in / seconds").
- Web: `text-wrap: balance` for headings and `text-wrap: pretty` for body text. SwiftUI (iOS 17+): `.lineBreakStrategy(.pushOut)` / balanced strategy where available, or rewrite copy. Flutter: no built-in, so join the last two words with a non-breaking space (`\u00A0`) or use a small balanced-text helper. Android: `LineBreaker.STRATEGY_BALANCED` / Compose `LineBreak.Heading`.
- Test in every localized language (Arabic, Japanese, Indonesian, Malay and others), because translations change the break points. Never insert hard line breaks in translatable strings.

## 10. Blur placeholders
**Rule:** Show a soft preview until the photo loads.
- Every remote image shows a blurred preview built from that photo's own colors (BlurHash or ThumbHash) instead of a gray box with an image icon.
- Generate the hash on the server or at upload time and send it with the image URL in the API response. Fall back to the image's dominant color if no hash exists.
- When the full image loads, crossfade (about 200 to 300 ms) from blur to sharp. Reserve the final size and aspect ratio so nothing jumps.
- Cache images and use properly sized thumbnails for lists.
- Flutter: `flutter_blurhash` or `thumbhash` with `cached_network_image` (`placeholder`, `fadeInDuration`). SwiftUI: a BlurHash decoder as the placeholder in `AsyncImage` or Kingfisher/Nuke. React Native: `expo-image` (`placeholder={{ blurhash }}` or `thumbhash`, `transition={250}`). Web: Next.js `placeholder="blur"`.

---

## Review checklist before shipping a screen
- [ ] Loading shows a skeleton in the content's shape (no full-screen spinner).
- [ ] Toggle-style actions update instantly, with a spring and haptic, and roll back on failure.
- [ ] Every changing or columned number uses tabular digits.
- [ ] Rounded shapes use continuous (squircle) corners from shared radius tokens.
- [ ] Every sheet has a grab handle and drags down to dismiss.
- [ ] Card-to-detail navigation uses a shared element transition (crossfade under Reduce Motion).
- [ ] OTP fields autofill from SMS or email in one tap and auto-submit.
- [ ] Reversible deletes happen immediately with an Undo toast; confirm dialogs only for irreversible actions.
- [ ] No headline or title leaves one word alone on its last line, in every language.
- [ ] Remote images show a blurred preview and crossfade in without layout shift.
- [ ] Reduce Motion, Dynamic Type, RTL (Arabic) and minimum tap targets still work.

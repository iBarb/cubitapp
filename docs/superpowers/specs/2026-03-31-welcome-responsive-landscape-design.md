# Welcome Responsive Landscape Design Spec

Date: 2026-03-31
Topic: Welcome screen responsive fit and mobile landscape-only behavior
Status: Approved for planning

## 1. Summary

This spec defines a focused layout adjustment for the `Welcome` screen.

The current hero over-scales on laptop-sized viewports, causing the headline and lower cards to push past the visible screen height. The target behavior is:

- fit the full welcome composition within the initial viewport on laptop and desktop
- preserve the current premium horizontal composition
- allow mobile usage only in landscape orientation
- block portrait mobile with a rotate-device message instead of creating a separate portrait layout

This spec does not change app navigation, solver logic, or content structure.

## 2. Goals

- Ensure the full home screen is visible without initial scroll on common laptop viewports
- Keep the existing visual hierarchy and horizontal composition intact
- Reuse the same composition on mobile landscape
- Prevent broken or cramped portrait mobile rendering by blocking that orientation

## 3. Non-goals

- Redesign the visual language of the welcome screen
- Create a dedicated portrait mobile layout
- Change copy, navigation targets, or product flow
- Modify solver, tutorial, or playback functionality

## 4. Chosen Approach

Three approaches were considered:

1. `Landscape-only` on mobile with a compacted desktop/laptop hero
2. `Landscape-first` with a degraded portrait fallback
3. Full responsive redesign for portrait mobile

The chosen approach is `1`.

Reasoning:

- it preserves the exact composition the user wants
- it avoids building a second visual system for portrait mobile
- it solves the immediate fit issue on laptop without changing product structure

## 5. Layout Behavior

### 5.1 Desktop and laptop

The welcome screen keeps one composition:

- top branding block
- main hero title and supporting copy
- lower action area with left card, center cube, and right card

To make the full screen fit within the viewport:

- reduce hero headline scaling on intermediate viewport widths and shorter viewport heights
- tighten vertical spacing between eyebrow, headline, copy, and lower action area
- reduce shell padding on constrained heights
- reduce card padding and card minimum height where needed
- preserve the lower three-part horizontal structure on larger screens

The desired result is a complete first-screen composition within `100dvh` on laptop without initial clipping.

### 5.2 Mobile landscape

Mobile landscape reuses the same horizontal composition.

Allowed adjustments:

- proportionally smaller headline
- smaller cube
- smaller card padding and internal text sizing
- reduced gaps between layout regions

Not allowed:

- stacking cards vertically
- moving to a portrait-first hero
- introducing a separate simplified home layout

### 5.3 Mobile portrait

Phone-sized portrait view is blocked.

Instead of rendering the normal welcome screen, the app shows a dedicated orientation state with:

- a short rotate-device message
- visual indication that the app is intended for horizontal use
- blocked interaction with the normal home UI while portrait is active

This portrait block applies only to phone-like viewports, not to tablets or desktop browsers resized vertically. The implementation should use an explicit viewport-based condition in planning, and it should not depend on browser support for forcing device rotation.

## 6. Component and Styling Scope

### 6.1 `WelcomeScreen`

`WelcomeScreen` remains the source of the welcome composition.

Expected adjustments:

- keep a single structural layout for desktop, laptop, and mobile landscape
- ensure the hero content and lower action row can shrink without breaking composition
- allow the center cube to remain decorative but not force vertical overflow

### 6.2 Global styles

Primary layout tuning is expected in shared stylesheet rules.

Expected style changes include:

- `hero-title` clamp reduction
- tighter `hero-copy` sizing and spacing
- smaller `screen-shell` padding on constrained heights
- smaller `compact-card` padding and height pressure
- breakpoint and height-aware tuning for the welcome shell

### 6.3 Orientation gate

An orientation-aware state is added at app or screen level.

Responsibilities:

- detect portrait mobile conditions
- scope the block to phone-like portrait viewports only
- render the rotate-device state
- restore the normal screen automatically when the device returns to landscape

## 7. Validation Criteria

The change is considered correct when all of the following are true:

- laptop home screen is fully visible on initial load without scroll
- headline no longer dominates so aggressively that the lower cards are pushed off-screen
- desktop wide screens still preserve the premium visual feel
- mobile landscape preserves the same horizontal composition
- phone-sized portrait shows a clear rotate-device block instead of a broken compressed layout

## 8. Risks and Boundaries

### Risks

- over-reducing type could weaken the hero impact
- shrinking too many elements could make the welcome screen feel generic
- orientation detection could still be mis-scoped if the viewport threshold is chosen poorly

### Boundaries

- solve the fit issue through measured scaling, not a redesign
- keep the layout visually consistent with the current welcome experience
- treat portrait blocking as a product decision for the welcome experience, not a browser-level guarantee

## 9. Implementation Notes for Planning

The implementation plan should cover:

- viewport-height-sensitive tuning for the welcome hero
- breakpoint adjustments for laptop and mobile landscape
- orientation gating conditions, viewport threshold, and messaging
- verification on desktop, laptop, mobile landscape, and mobile portrait

# CUBITAPP Design Spec

Date: 2026-03-31
Topic: CUBITAPP: 3D Solver & Tutor
Status: Approved for planning

## 1. Product Summary

CUBITAPP is a single-page web application that teaches users how to solve a Rubik's Cube by letting them input their real cube state, generate a solution client-side, and replay the solution step by step on a 3D cube.

V1 is a focused freemium-ready MVP:

- `Welcome` screen with entry points to `Tutorial` and `Solver`
- `Tutorial` teaser screen marked as coming soon
- `Solver` screen with a functional `Beginner Method`
- Premium methods visible but locked
- No auth, payments, or backend in v1

Primary UI language is Spanish. Brand name is `CUBITAPP`, with subtitle `3D Solver & Tutor`.

## 2. Goals

### Goals

- Let the user enter a cube state accurately
- Solve the cube client-side with a beginner-oriented method
- Replay the solution with smooth 3D animation
- Preserve the dark premium visual style shown in the Stitch reference
- Keep the app freemium-ready without adding monetization complexity in v1

### Non-goals

- Multiple working solving methods in v1
- Authentication, subscriptions, or payment flows
- Camera scanning or image recognition
- Full educational tutorial content for the beginner method
- Server-side solving or persistence

## 3. Product Scope

### 3.1 Screens

#### Welcome

- Centered hero with brand and subtitle
- Two cards:
- `Aprender` -> navigates to `Tutorial`
- `Resolver` -> navigates to `Solver`

#### Tutorial

- Teaser screen only
- Primary message is `Modo Tutorial - Próximamente`
- Top-left back action `← Inicio`

#### Solver

- Main product surface
- Uses one persistent shell with internal UI states instead of separate routes
- Top-left back action `← Inicio`

### 3.2 Navigation Model

The app is a SPA without formal router dependency in v1. Screen visibility is controlled locally by app state using the requested `display: none / flex` approach.

Allowed transitions:

- `Inicio -> Tutorial`
- `Inicio -> Solver`
- `Tutorial -> Inicio`
- `Solver -> Inicio`

## 4. UX Direction

The visual direction follows the provided Stitch reference:

- dark background
- soft glassmorphism panels
- centered composition
- premium but minimal surface treatment
- clear separation between editor, 3D preview, method panel, and steps

The solver must not stay visually static. It should preserve one shell but shift emphasis by state.

### Solver state emphasis

#### Editing state

- The 2D editor is the primary element
- The 3D cube is visible as orientation/reference
- The method selector is visible
- The steps area is present but visually inactive or compressed

#### Playback state

- The steps carousel and playback controls become primary
- The 3D cube becomes the visual focus
- The 2D editor remains available but visually reduced

This approach preserves the Stitch-like layout while making the real workflow understandable.

## 5. Solver Screen Layout

The solver uses a fixed composition with state-driven emphasis rather than separate subpages.

### 5.1 Main regions

#### Top bar

- Back button `← Inicio`
- Brand label
- Small badge for active method

#### Left/primary editor region

- Full 2D cube net editor as the main input surface
- Based on the unfolded cube net pattern chosen during brainstorming
- Centers are rendered and fixed
- User paints editable stickers by selecting a palette color and clicking tiles

#### Center region

- Persistent `CubePreview3D`
- Used as orientation reference during editing
- Used as animated playback surface after solve

#### Right region

- `MethodPanel`
- `Beginner Method` enabled
- `CFOP`, `Roux`, and `ZZ` visible but locked
- Primary call to action lives here:
- `Resolver` in editing/ready state
- `Reproducir` in playback state when the generated solution has not started yet

#### Bottom region

- `StepsCarousel`
- In editing mode it is collapsed, dimmed, or inactive
- In playback mode it shows the generated move sequence, current step, and manual navigation

### 5.2 Editor design choice

The chosen editor pattern is:

- full 2D net as the main editor
- dark premium styling consistent with Stitch
- cube net is the protagonist during capture

This was chosen over face-by-face editing because it provides stronger spatial clarity for new users and matches the user's preferred visual mental model.

## 6. Functional Behavior

### 6.1 Cube input rules

- The six center stickers are predefined and non-editable
- Only the remaining 48 stickers are editable
- Each color can be used on at most 8 editable stickers
- The UI shows global color counts while editing
- `Resolver` stays disabled until all 48 editable stickers are filled
- `Reset` clears editable stickers and returns the editor to its initial state while preserving fixed centers

### 6.2 Final validation rules

Color limits reduce common mistakes but do not guarantee a physically solvable cube. Before running the solver, the app must perform a final consistency validation.

If validation fails:

- solving is blocked
- the user remains on the solver screen
- the error message is explicit and corrective, not generic

### 6.3 Solving and playback

- Solving runs entirely client-side
- V1 supports only `Beginner Method`
- Once a valid cube is solved, the app stores the generated move list in order
- The user can:
- play
- pause
- go to next step
- go to previous step
- select a step from the carousel

The current step must stay synchronized across:

- the move list
- the highlighted step card
- the 3D animation state

## 7. State Model

### 7.1 App-level screen state

- `welcome`
- `tutorial`
- `solver`

### 7.2 Solver UI state

- `editing`
- `ready`
- `solving`
- `playback`
- `error`

### 7.3 Core solver data

- `cubeState`: normalized 54-sticker state in a fixed face order
- `selectedColor`: currently active editor color
- `method`: current method id
- `solutionSteps`: ordered move list
- `playbackIndex`: current move index
- `isPlaying`: playback running or paused
- `playbackSpeed`: current playback speed
- `validationErrors`: user-facing issues that block solve

## 8. Component Boundaries

The implementation should favor small, isolated units with clear responsibilities.

### Required components/modules

- `AppShell`
- `WelcomeScreen`
- `TutorialScreen`
- `SolverScreen`
- `NetEditor`
- `ColorPalette`
- `MethodPanel`
- `CubePreview3D`
- `StepsCarousel`
- `PlaybackControls`
- `CubeStateStore`
- `SolverEngine`

### Responsibility summary

- `SolverScreen` orchestrates layout and solver state transitions
- `NetEditor` owns only cube input interactions and rendering
- `CubePreview3D` owns only Three.js rendering and animation hooks
- `MethodPanel` owns method selection display and locked-state presentation
- `StepsCarousel` and `PlaybackControls` own navigation through generated steps
- `SolverEngine` is a pure logic module that receives cube state and returns a move sequence or a failure result
- `CubeStateStore` is the source of truth for editor, solver, and playback state in the client

## 9. Technical Architecture

### Stack

- React
- Tailwind CSS
- Three.js
- CSS keyframes plus JavaScript-driven tweens for motion
- Client-side JavaScript solver implementation for Beginner Method

### Architecture decisions

- Use local app state instead of a router in v1
- Use `useReducer + context` for shared cube and solver state
- Keep Three.js isolated behind a React component boundary
- Keep solver logic outside rendering components
- Do not introduce backend infrastructure in v1

### Stitch usage

The visual implementation should follow the existing Stitch project and the approved screenshots as the reference for structure and styling. During brainstorming, direct MCP access to Stitch timed out, so the design is based on:

- the linked Stitch project id in `.stitch-project.json`
- the screenshots provided by the user
- the approved solver/editor decisions made in this spec

Implementation may retry Stitch MCP access later, but this spec does not depend on that retry.

## 10. Error Handling

### Editing errors

- incomplete cube entry
- attempting to exceed the allowed count for a color

These should be shown inline near the editor and palette.

### Validation errors

- cube coloring is complete but not physically solvable

This should block solve and present a clear corrective message.

### Solver errors

- unexpected failure while generating a solution

This should produce a clear failure state with the option to keep editing or reset.

### Locked method interactions

- locked methods remain visible
- they do not navigate away or break the current state
- they should read as premium/locked, not as broken buttons

## 11. Testing Strategy

### Unit tests

- cube state normalization
- editable color counts
- center immutability
- ready-state detection
- final validation rules
- beginner move generation helpers

### Integration tests

- edit cube -> ready -> solve -> playback
- selecting a step updates playback state
- reset returns the editor to a clean state
- navigating back to `Inicio` from `Solver` and `Tutorial`

### Manual verification

- test with a correctly entered cube
- test with incomplete input
- test with invalid but color-complete input
- test playback controls across multiple steps
- test layout usability on desktop and mobile widths

## 12. Delivery Boundaries For V1

V1 is complete when all of the following are true:

- the three requested screens exist in one SPA
- the visual language is clearly aligned with the Stitch reference
- the user can enter a full cube via the 2D net editor
- the app blocks invalid or incomplete states before solving
- the beginner method can generate a client-side solution
- the 3D cube can replay the solution step by step
- premium methods are visible but locked

V1 does not require:

- auth
- billing
- backend APIs
- camera capture
- full tutorial content
- additional solving methods

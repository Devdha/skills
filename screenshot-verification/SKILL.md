---
name: screenshot-verification
description: Use when a task requires visual verification with screenshots, especially frontend UI changes, local app previews, responsive layout checks, rendered assets, canvas or 3D output, browser-based QA, or before/after proof for review. Guides Codex to run the target, inspect it in a real browser or app surface, capture screenshots at relevant states and viewports, and report concrete evidence instead of relying on implementation-only claims.
metadata:
  short-description: Verify UI changes with screenshot evidence
---

# Screenshot Verification

Use this skill when visual correctness matters and the user asks for screenshot proof, browser verification, UI validation, rendered-state checks, responsive checks, or frontend review.

## Goal

Prove what the actual rendered UI does. A good verification includes:

- the exact target URL, app screen, route, or installed-app surface
- the viewports, devices, or screen sizes tested
- the UI states exercised
- screenshots saved as artifacts
- notable console, runtime, network, or rendering issues
- a pass/fail summary tied to the user's request

Treat screenshots as evidence, not as the goal. If the screenshot proves the wrong surface, stale state, or only a loading screen, the verification is incomplete.

## Tool Selection

Choose the visual surface based on the target:

- For local web apps such as `localhost`, `127.0.0.1`, `file://`, or obvious local previews, use the in-app Browser plugin when available.
- For authenticated remote pages, user-profile-dependent pages, or existing browser sessions, use Chrome automation when available.
- For deterministic repeated viewport screenshots, canvas pixel checks, console capture, or CI-style verification, use Playwright from the terminal.
- For native mobile or desktop apps, use the relevant app testing toolchain instead of substituting a web preview.

Do not treat a screenshot of a different surface as proof. For example, an Expo web preview is not proof of an installed Android app.

## Workflow

1. Identify the visual claim.

   State what must be proven before opening the browser or app. Examples:

   - "The pricing page renders the updated stats without overflow."
   - "The modal is usable on narrow mobile."
   - "The canvas is nonblank and framed correctly."
   - "The uploaded image appears in the generated card."

2. Run or locate the target.

   - If the app needs a dev server, start it and keep it running until verification is finished.
   - If a server is already running, reuse it when appropriate.
   - Record the final URL, route, screen, or app surface.
   - If the work happens in a git worktree and environment files are required, copy them from the source project before running.

3. Capture relevant states.

   Pick the smallest viewport and state set that can prove the claim.

   Default web viewport set:

   - desktop: `1440x900`
   - mobile: `390x844`

   Add narrower mobile such as `320x720` when text overflow, dense mobile layout, long labels, localization, or Korean copy is relevant.

   Capture before/after screenshots only when comparison is useful.

4. Inspect runtime signals.

   Check for:

   - console errors
   - failed network requests
   - missing images or assets
   - clipped, wrapped badly, or overlapping text
   - unintended horizontal scroll
   - blank canvas, video, iframe, or 3D scene
   - stuck loading, empty, or skeleton states
   - controls that shift layout on hover, focus, or content changes

5. Save artifacts.

   Save screenshots under the task's artifact or output area. Prefer, in order:

   - the user-requested output directory
   - the current Codex workspace `outputs/` directory, if present
   - a project-local artifact directory such as `artifacts/screenshots/`
   - a temporary `work/` directory only for intermediate evidence

   Prefer descriptive names:

   - `pricing-desktop.png`
   - `pricing-mobile-390.png`
   - `modal-error-state.png`
   - `canvas-nonblank-check.png`

   Do not hardcode a user-specific absolute path inside this skill. Resolve artifact paths from the current workspace, project, or explicit user instructions.

6. Report evidence.

   In the final response, include:

   - what was tested
   - which viewports, devices, or states were captured
   - screenshot artifact links or file paths
   - notable runtime findings
   - what was not verified, if anything

   When running in Codex desktop, prefer clickable artifact links for user-facing screenshots.

## Pass Criteria

Screenshot verification is complete only when:

- the target screen was opened in a real browser or real app surface
- screenshots were captured for the relevant viewport, device, or state set
- runtime errors and obvious rendering defects were checked
- the final answer references the artifacts
- any unverified scope is explicitly called out

## Failure Handling

If the browser cannot open, the app cannot start, credentials are missing, or the target cannot be reached:

- state the blocker concretely
- include the command, URL, route, or tool step that failed
- preserve partial evidence such as logs or screenshots of the failing state
- do not claim visual verification succeeded

## Review Mindset

For UI review tasks, prioritize concrete visual regressions over aesthetic preference. Look especially for:

- layout overflow
- inaccessible controls
- clipped labels
- unreadable contrast
- broken responsive behavior
- incorrect data displayed
- stale loading or empty states
- screenshots that prove the wrong surface

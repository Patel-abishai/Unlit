# Unlit

An AI quit-smoking coach built on Acceptance and Commitment Therapy (ACT). One job: get a smoker to day 42 smoke-free. Calm, adult, non-judgmental.

## What's here

- `design-reference/project/Main.dc.html` is the approved clickable prototype (iPhone 15, 393x852): every screen, flow, design token, animation and the final copy. It is the source of truth for the build.
- `design-reference/project/canvas.json` is the design canvas index.
- `docs/claude-code-prompt.md` is the build brief for the React Native (Expo) app.

## Reading the prototype

`Main.dc.html` is a Design Component file, not plain HTML:

- Markup lives inside `<x-dc>`. Inline styles are the exact values to match.
- `<helmet><style>` holds the light, dark and calm theme tokens, component classes and keyframes.
- `<sc-if>` is conditional rendering, `<sc-for>` is a loop, `{{path}}` binds values from `renderVals()`.
- The `<script type="text/x-dc">` block holds all behavior, mock data and copy.

## Next step

Build the React Native app from `docs/claude-code-prompt.md`.

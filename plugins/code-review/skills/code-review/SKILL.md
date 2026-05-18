---
name: code-review
description: Review code against ionstudioapps standards — TypeScript, React/React Native, Node.js. Use when asked to review code, check a PR, or audit a file.
---

# ionstudioapps Code Review

Review the changed or specified code against these standards. Report issues grouped by severity: **Critical**, **Warning**, **Suggestion**.

## TypeScript
- No `any` types — use proper types or `unknown`
- Prefer `interface` over `type` for object shapes
- All async functions must handle errors (try/catch or `.catch()`)
- No unused imports or variables

## React / React Native
- Components must be functional (no class components)
- `useEffect` must declare all dependencies in the dependency array
- No inline styles in React Native — use `StyleSheet.create`
- Keys in lists must be stable and unique (not array index)
- Avoid prop drilling more than 2 levels — use context or state management

## Node.js (Rocky)
- All environment variables must be accessed via `src/config.js`
- No hardcoded secrets or tokens
- Async handlers must use try/catch
- Log errors with context, not just `console.error(err)`

## General
- Functions should do one thing
- No commented-out code in PRs
- File names: `camelCase.ts` for utilities, `PascalCase.tsx` for components
- Keep files under 300 lines — split if larger

## Output format
For each issue found:
```
[CRITICAL|WARNING|SUGGESTION] filename:line — description
```
End with a one-line summary of overall quality.

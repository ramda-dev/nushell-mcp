---
status: accepted
date: 2026-11-13
decision-makers: zaynram
---

# Raw-mode input travels on stdin as text

## Context and Problem Statement

`nu_exec` with `structured: false` ran inside the same generated temp script as
structured mode, so a relative `use ./module.nu` resolved against the temp
directory instead of `cwd` (#3). Moving raw mode to plain `nu --commands`
fixes that, but raw mode still has to accept `input`. How should the input
reach the code without bringing the wrapper back?

## Decision Drivers

* Raw mode must run the code exactly as `nu -c` would in a shell.
* `input` and `bashEnv` must keep working in raw mode.
* `nu_exec` stays stateless.

## Considered Options

* Pass `input` on stdin with `nu --stdin --commands <code>`, leaving `$in` as text
* Parse `input` into `$in` by wrapping the code in `do { }`
* Keep the temp-script vehicle for calls that carry `input`

## Decision Outcome

Chosen option: "pass `input` on stdin", because it is the only option that
leaves the code unwrapped, so relative imports resolve against `cwd` with or
without input.

### Consequences

* Good, because `$in` is the input text in every top-level statement, not
  only the first pipeline.
* Good, because the input is never written into nu source, so it needs no
  quoting. Structured mode later adopted the same vehicle (#11).
* Bad, because the code parses `$in` itself (`$in | from nuon`), unlike
  structured mode.
* Bad, because the code rides the command line, which Windows caps at 32,767
  characters. `runRaw` errors before spawning past the cap and points at
  structured mode, which runs from a script file.

### Confirmation

`test/smoke.test.ts` covers relative `use` with and without input, `$in` as
text in every statement, `$in` as nothing without input, and the Windows
command-line guard.

## Pros and Cons of the Options

### Pass `input` on stdin

* Good, because the code runs unwrapped.
* Neutral, because `$in` is a string the code must parse.

### Wrap the code in `do { }`

* Good, because `$in` arrives parsed.
* Bad, because the wrap is the semantic change raw mode exists to avoid.

### Keep the temp script for input-carrying calls

* Bad, because relative `use` would break only when `input` is given.

---
status: accepted
date: 2026-11-13
decision-makers: zaynram
---

# Nushell 0.117 is the minimum supported version

## Context and Problem Statement

#5 planned a structured wrap built on nu's `run` command and had to decide how
to handle Nushell versions without it: detect the feature at runtime through
the memoized `getNuVersion()`, or set an explicit minimum version. The README
listed Nushell 0.112 as the floor.

## Decision Drivers

* One supported range keeps tests and documentation honest.
* Behaviour differs across recent releases: imports inside `do { }` are
  invisible to `scope` on 0.114.1 but listed on 0.115.1, 0.116.1 and 0.117.0.

## Considered Options

* An explicit floor at 0.117.x, the latest release that is not a pre-release
* Feature detection through `getNuVersion()` with a fallback path
* No floor

## Decision Outcome

Chosen option: "an explicit floor at 0.117.x", because it removes the need
for version-specific fallbacks such as a `do { }` path for older releases.

### Consequences

* Good, because the `scope` difference on 0.114 is out of the supported range,
  which let #5 narrow to the stdin input fix and drop the `run` wrap.
* Bad, because users on 0.112 to 0.116 are outside the supported range.
* Neutral, because the floor is documented in the README, not enforced:
  `getNuVersion()` detects the version but nothing gates on it.

### Confirmation

The test suite runs against nu 0.117.0.

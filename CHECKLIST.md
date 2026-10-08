# Pre-ship UI checklist

Run before claiming any UI work done. Every item traces to a law.

## States
- [ ] Every async surface has designed loading, empty, and error states (Law 3)
- [ ] Errors say what happened + why + next action, with an escape hatch link (Law 3)
- [ ] Empty states on filtered views explain *why* it's empty (Law 4)
- [ ] Verification surfaces distinguish authorized / missing / wrong-record / lookup-error (Law 5)

## Numbers & data
- [ ] `tabular-nums` on every countable surface; numbers right-aligned in tables (Law 6)
- [ ] Monospace for IPs / emails / IDs / hashes (Law 6)
- [ ] Tables default worst-first (Law 6)
- [ ] Every number on a dashboard traces to an attributable operation — or is labeled UNKNOWN (Law 8)

## Identity
- [ ] Zero raw hex outside the token file; `.dark` is a token column, not overrides (Law 1)
- [ ] Every status pairs color with text; dots have `sr-only` text (Law 2)
- [ ] No emoji in UI chrome; icons from the real icon set (Law 11)
- [ ] Forbid-list respected: no gradients, no cards inside cards, no glassmorphism (Law 11)

## Structure
- [ ] Squint test: exactly one loud element per screen (Law 9)
- [ ] Touch targets ≥ 24×24 px (44px mobile web); focus rings visible both themes (Law 10)
- [ ] Full keyboard path through the primary flow (Law 10)
- [ ] Contrast ≥ 4.5:1 body / 3:1 large text (Law 10)

## Honesty
- [ ] No fake urgency, confirm-shaming, or hidden limits; real constraints stated in copy (Law 12)
- [ ] Benign explanations require positive evidence (Law 13)

## Process
- [ ] Typecheck + lint + tests + build green (Rule 1)
- [ ] Real flow exercised end-to-end, not just built (Rule 1)
- [ ] Before/after screenshots attached (Rule 12 in AGENTS.md)

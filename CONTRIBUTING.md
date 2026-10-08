# CONTRIBUTING.md — the scar protocol

This playbook grows the only way engineering standards should: someone hits a real failure, writes down the receipt, and the community triages it into a law. Agents and humans contribute the same way.

## How to contribute a scar

A **scar** is a UI failure you actually hit — not a preference, not a trend take. Open an issue (or raise it on Moltbook / your platform of choice and link it) with:

```
**What broke:** (one sentence)
**Where:** (product/repo + surface, or "my own build, unpublished" — that's fine)
**Receipt:** (screenshot, diff, incident log, or the fix commit — something checkable)
**Proposed law:** (one imperative sentence, if you have one)
```

No receipts → it goes to the idea pile, not the playbook. That's the whole gate.

## Triage

1. **Duplicate?** If a law covers it, the scar becomes *evidence* on that law (you get credited there).
2. **Single-reporter rule.** New claims are labeled `single-source` in the playbook. They get promoted when either (a) an independent second source reports the same failure, or (b) the maintainer reproduces it in their own build.
3. **Conflicting claims** both stay recorded until a test or a build resolves them. Supersessions are logged in the changelog of beliefs with the reason — losing a belief publicly is how the standard stays honest.
4. **Adopted changes** require a before/after screenshot or diff in the PR.

## What we will not accept

- Style takes without failures ("I just prefer X").
- Rules that can't be verified by a checklist or a grep.
- Anything that requires shaming a named person/team/product — scars describe failures, not villains.
- Engagement-farming PRs (emoji icon additions, gradient restyles — yes, really).

## Attribution

Accepted scars and adopted laws credit the contributor by handle with a link to the source discussion — see [ATTRIBUTION.md](ATTRIBUTION.md). Say in the issue if you want a different handle or no credit.

## Critique round 2 (open)

Current open questions we specifically want scars for:

1. **Law 9 (one loud thing per screen)** — currently single-source + own-practice. Does uniform-weight failure show up in your builds too?
2. **Law 11 (forbid-by-name lists)** — do named bans outperform adjective guidance in agent code reviews?
3. **Data-dense dashboards** — when does worst-first ordering become wrong? (We've only tested DMARC/security data.)
4. **Motion** — we allow 150–200ms ease-out + skeleton pulse only. What did we miss?

# The Playbook — frontend quality laws for AI coding agents

updated: 2026-10-08 · v1.0
Reference implementation: [DmarcDuck](https://github.com/superz1402/dmarcduck) (production SaaS, Workers + Neon, shipped and critiqued in public).

Each law has five parts: **Statement** · **Why** (the failure it prevents) · **Evidence** (where it was learned) · **Implement** (the short version) · **Verify** (how to check it before shipping). Laws sourced from a single contributor are labeled — they stay labeled until independent confirmation.

---

## I. Identity & tokens

### Law 1 — Identity comes from tokens, not components

**Statement.** Define background / ink / accent / spacing / type-scale as CSS custom properties first (`:root` + `.dark`), then build components that *only* reference tokens. One deliberate accent color beats three decorative ones.

**Why.** Components that hardcode colors drift the moment a second engineer (or a second agent session) touches them. Token-first also makes dark mode a second column of data instead of a per-component hunt — bolt-on dark mode shows up broken in borders, shadows, and hover states long after the happy path looks fine.

**Evidence.** DmarcDuck's design system (warm paper background, ink text, single amber accent — deliberately not the purple-gradient "AI look") survived a 17-screenshot audit and a community critique round with zero token violations found. The "AI look" phenomenon itself is independently corroborated: multiple 2025-26 industry write-ups diagnose the same purple-gradient/glass-card convergence as training-data averaging. **Status: multi-source.**

**Implement.**
```css
:root  { --bg: #faf7f2; --ink: #1a1712; --accent: #b45309; }
.dark  { --bg: #171412; --ink: #e8e2d8; --accent: #f59e0b; }
```
Components: `background: var(--bg)` — never a raw hex. If a component needs a color the token set doesn't have, that's a token-set bug, not a component decision.

**Verify.** `grep -rE '#[0-9a-fA-F]{3,8}' src/components/` returns nothing outside the token definition file. Dark mode toggle produces no unstyled seams on any screen.

### Law 2 — Color never carries meaning alone

**Statement.** Every status pairs its color with a text label. Color-coded dots get screen-reader text.

**Why.** ~8% of men have some color-vision deficiency; statuses like pass/fail also get screenshotted into contexts (print, dark mode, foreign tools) where your palette isn't guaranteed.

**Evidence.** DmarcDuck verdict badges ship with visible text + `sr-only` verdict words on every dot. External grounding: WCAG 2.2 SC 1.4.1 (Use of Color — Level A) requires color not be the only visual means of conveying information.

**Implement.** `<Badge tone="pass">Aligned</Badge>` — tone and text travel together. Dots: `<span class="dot dot-red"><span class="sr-only">failing</span></span>`.

**Verify.** Screenshot every status surface in grayscale. If you can't tell pass from fail, neither can part of your audience.

---

## II. States — the trust triad

### Law 3 — Loading, empty, and error ship before the happy path counts as done

**Statement.** Every async surface has all three states designed, not improvised. Errors always state (a) what happened, (b) why, (c) the next action. Dead-end errors are bugs.

**Why.** The happy path is the *only* path agents naturally generate — it's the path in the prompt. Everything else is a branch the model predicts less often, which is exactly where "every screen is the happy path" comes from. A user who hits one dead-end error trusts nothing else on the page.

**Evidence.** Source: @yuigui (Moltbook critique round 1, 2026-10-07): "Every screen is the happy path. No loading state, no 'it failed'... A real screen has one loud thing and lets the rest be quiet." Triaged against DmarcDuck's own audit: analyzer error state (reason + what to try + filename chip + docs link) became the team's exemplar screen. External grounding: NN/g's error-message and empty-state guidelines converge on the same structure (error → explanation → recovery action; empty state → teach the next action). **Status: multi-source.**

**Implement.**
- Loading: skeletons matching final layout. Spinners only for sub-second waits.
- Empty: teach the next action — point at the exact button/process that fills the screen. "No reports yet" is a dead end; "Forward your first report to rua@… — it takes ~24h to arrive" is a door.
- Error: banner (not toast) for blocking errors; reason + next action + escape hatch link.

**Verify.** For each async surface, answer: what renders on timeout? on 4xx? on empty result? on slow 3G? If any answer is "whatever the framework does by default", the screen isn't done.

### Law 4 — Surfaces must agree, or explain the disagreement

**Statement.** If one surface says "1 record stored" and another says "No reports yet", both are lying. Add an explicit "stored but outside this window / filtered out" state.

**Why.** Every filtered view is a bet that the user remembers the filter exists. They don't. The disagreement between two honest-but-scoped surfaces reads as data loss — the most expensive trust failure a dashboard can have.

**Evidence.** DmarcDuck bug (2026-10-07): domain detail said "No reports stored yet" while the ingestion ledger said "1 record stored" (the row predated the 30-day window). Fix: "Reports are stored — just outside this window" with the count and latest date.

**Implement.** Any count derived from a window/filter must, at zero, say *why* it's zero: never stored / outside window / filtered out / failed to load.

**Verify.** For every empty state on a filtered surface: does it distinguish "nothing exists" from "nothing in scope"?

### Law 5 — Presence is not correctness

**Statement.** "A record exists at the name" ≠ "the right record exists." Any verification UI distinguishes authorized / missing / wrong-record / lookup-error.

**Why.** Wildcards and leftover records make naive existence checks lie. A verification checkmark that only proves DNS returns *something* teaches users to trust a check that can't catch the failure they actually care about.

**Evidence.** Live DoH test during DmarcDuck verification UX work: presence-check passed against a wildcard record that would never authorize anything. (Source: own incident, 2026-10-07.)

**Implement.** Four outcomes, four renderings. If your check can't distinguish them yet, say "unverified — check incomplete", not "verified".

**Verify.** Point the check at a wildcard record and at a typo'd record. It must fail differently.

---

## III. Data & numbers

### Law 6 — Numbers are a design surface

**Statement.** `font-variant-numeric: tabular-nums` on anything countable; numbers right-aligned in tables; worst-first ordering by default; monospace for IPs, emails, hashes, IDs; relative dates in dense tables, absolute timestamps in ledgers.

**Why.** Proportional digits make columns of counts shimmer — users can't compare rows. Left-aligned numbers of varying width do the same. These are small-cost/high-credibility moves.

**Evidence.** @yuigui's critique round 1 point 2 ("tabular numbers, right-aligned, fewer decimals") — independently matched by DmarcDuck's pre-existing `.tnum` global utility, which is why it was triaged *already handled* rather than adopted. The convergence of two independent sources is the promotion criterion this playbook uses. **Status: multi-source.**

**Implement.**
```css
.tnum { font-variant-numeric: tabular-nums; }
```

**Verify.** Open any table with >5 rows of counts. Do digits align vertically per column?

### Law 7 — A ledger beats a raw table for "did the system see my thing?"

**Statement.** For any ingestion/submission flow, ship an activity ledger: status badge + one-line summary + expandable per-item reasons. Raw tables answer "what data exists"; ledgers answer "what happened to my stuff" — the question users actually have.

**Evidence.** DmarcDuck ingestion ledger (files received, reports parsed, records stored, duplicates skipped, per-file reject reasons) was built after a garbage payload returned a silent `200 {stored: 0}` — the exact "failure that looks like success at every layer except the ledger" shape @merktop described independently in critique round 1.

**Verify.** Submit something that fails halfway. Can a user find out *how far it got* without asking anyone?

### Law 8 — Attribution beats inference

**Statement.** When the system cannot attribute an outcome to a specific operation, display UNKNOWN. Never render a number inferred from counts.

**Why.** Inferred numbers propagate: dashboards copy them, alerts fire on them, and the error is unrecoverable because the provenance is gone. An explicit UNKNOWN can clear on the next read; a wrong number never does.

**Evidence.** @settlestackresearch (critique round 1, EXP thread): "acceptance is not inclusion; delivery is not visibility" — refined into DmarcDuck's ledger v2 (Report.eventId, in-window/outside-window, explicit unknown). General form credited in the product changelog.

**Verify.** Find one number on your dashboard and ask: what operation produced this, and can I link to it? No answer = inferred. Fix it or label it.

---

## IV. Structure & hierarchy

### Law 9 — One loud thing per screen

**Statement.** Every screen has exactly one primary element (the answer to "should I worry? what do I do next?") rendered at full weight. Everything else steps down.

**Why.** Uniform element weight forces users to diff the page themselves. The AI-generated variant — every card the same elevation, every heading the same size — is uniform weight with extra steps.

**Evidence.** @yuigui critique round 1 point 1b, adopted into DmarcDuck and retro-validated: the results screen already led with a verdict banner while cards stayed quiet. **Status: single-source + own-practice convergence; promotion pending one more independent confirmation.**

**Verify.** Squint at the screen (or blur it). Does exactly one thing stand out?

### Law 10 — Touch targets and focus are infrastructure

**Statement.** Interactive targets ≥ 24×24 CSS px minimum (WCAG 2.2 SC 2.5.8, Level AA); we hold mobile web to 44px. Visible focus rings everywhere; a keyboard path through every flow; contrast ≥ 4.5:1 body / 3:1 large text (SC 1.4.3).

**Why.** These are the checks that cost nothing at design time and are expensive to retrofit. Agents skip them because they're invisible in screenshots — which is precisely why a checklist must carry them.

**Evidence.** WCAG 2.2 (2023) numbers as published by W3C; applied across DmarcDuck (focus rings, sr-only status text, keyboard flows verified in the 22-point smoke pass).

**Verify.** Tab through the primary flow: can you complete it with no mouse? Are focus rings visible on both themes?

### Law 11 — No emoji as UI icons

**Statement.** Iconography comes from a real icon set (lucide-react on web). Emoji in chrome, logos, or status indicators is an amateur tell — and an accessibility and rendering-consistency hazard (emoji look different on every platform).

**Evidence.** Adopted from @yuigui's forbid-list ("no gradients, no emoji icons, no cards inside cards") — matched by our own audit. Forbid-by-name beats adjective-based guidance ("keep it clean") because names are checkable. **Status: single-source + own-practice; promotion pending.**

**Implement.** Keep an explicit forbid-list in your UI docs: no gradients, no emoji icons, no cards inside cards, no glassmorphism unless the product is glass.

**Verify.** Grep the component tree for emoji literals. There should be none outside user content.

---

## V. Honesty

### Law 12 — Honest states over fake polish

**Statement.** Real limits in the UI ("link expires in 7 days"), strikethrough features on free plans instead of hiding them, no fake urgency, no countdown pressure, no confirm-shaming. Trust is a design feature with an ROI.

**Evidence.** DmarcDuck pricing page (strikethrough Free features + "How billing works" + "Why flat-per-account pricing") is the single most-praised screen in both audit rounds — trust through boredom, exactly the brand. (Source: own build + audit.)

**Verify.** Read every sentence of marketing-flavored copy on product surfaces. Could a cynical user quote any of it back at you?

### Law 13 — Positive evidence before mercy

**Statement.** Downgrading a negative verdict ("this looks like spoofing") to a benign one ("this is forwarding") requires receiver-stated evidence (reason codes, auth results) or a structural tell (list-shaped envelope). When unsure: state the evidence and the next check — never the verdict.

**Why.** Systems that explain scary things away without evidence train users to ignore the scary things. The UX kindness that outruns the data is a lie with good manners.

**Evidence.** DmarcDuck policy engine design, 2026-10-07. Generalization of the verification philosophy: the system speaks only where it has evidence, and labels the rest.

**Verify.** Find one benign-explanation path in your product. What evidence triggers it? If the answer is "the absence of scary signals", it's a false-mercy path.

---

## VI. The "AI-generated look" — anti-pattern catalog

Why generated UIs converge (and what to do about it). Corroborated by multiple independent industry write-ups (2025-26) and by our own build experience:

| Anti-pattern | What it looks like | The fix in this playbook |
|---|---|---|
| Training-corpus palette | purple/indigo gradients, glassy cards, glow shadows | Law 1 (tokens; one deliberate accent; a non-default base palette) |
| Happy-path-only screens | no loading/empty/error, uniform weight | Laws 3, 9 |
| Emoji iconography | 🚀✨🔒 as chrome | Law 11 |
| Adjective-driven review | "make it cleaner, more modern" | Forbid-by-name lists + one reference screen |
| Fake completeness | dead-end errors, lying counters, hidden limits | Laws 4, 5, 12 |
| Dark mode bolt-on | seams in borders/shadows/hover | Law 1 (`.dark` as second token column, never per-component overrides) |

**Deeper cause, worth naming:** an LLM predicts the most likely good design, and the most likely design is the *average* of its corpus. Averaging is exactly what identity is not. The escape isn't more creativity — it's constraints with reasons (this playbook) and one reference screen to converge on.

---

## VII. Agent working rules (how this standard gets applied)

These govern the *process*, not the pixels — they came from shipping DmarcDuck with agents writing every line:

1. **Typecheck + lint + tests + build green before claiming done; then exercise the real flow.** A passing build is not "done"; a working user journey is.
2. **Fix the test before the product.** Prove the harness is right before reporting a product bug — 5 of 7 initial smoke "failures" were harness bugs (case-sensitive headers, Secure-cookie-over-HTTP, wrong field names).
3. **One screen, one job.** If a screen needs three paragraphs of explanation in the docs, the screen is wrong.
4. **Screenshot before and after every UI change.** A UI change without a before/after didn't happen.
5. **Single-reporter caution.** A claim from one source gets investigated, not auto-adopted. Promotion to law requires either an independent second source or our own build reproducing it.

See [AGENTS.md](AGENTS.md) for the paste-ready version.

---

## Changelog of beliefs

- **2026-10-08** — v1.0. Playbook promoted from private brain notes to a public standard. 13 laws; three carry a single-source label (yuigui round-1 items awaiting independent confirmation); the "AI look" analysis upgraded to multi-source via independent industry corroboration.
- **2026-10-07** — Session-12 additions absorbed: Law 5 (presence ≠ correctness), Law 13 (positive evidence before mercy), Law 8 (attribution beats inference — from @settlestackresearch's ledger refinement).
- **2026-10-07** — Born from the DmarcDuck v0.1 audit + first Moltbook critique round (@cooperemail, @phoenixreforge, @vibejamhank, @yuigui) + smoke-suite lessons.

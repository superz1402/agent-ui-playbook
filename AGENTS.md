# AGENTS.md — UI quality rules for AI coding agents (paste into your repo)

Drop this file (or the section below) into any repository where an AI agent writes frontend code. Written for the way agents actually fail.

---

## Hard rules

1. **Typecheck + lint + tests + build green BEFORE claiming done.** Then exercise the real user flow — a passing build is not done; a working journey is.
2. **Every async surface ships loading + empty + error states before the happy path counts as done.** Errors state what happened, why, and the next action. Dead-end errors are bugs.
3. **Tokens only.** Components reference CSS custom properties (`var(--…)`) — never raw hex. Dark mode is a second token column (`:root` + `.dark`), not per-component overrides.
4. **Color never carries meaning alone.** Status = color + text label; color-coded dots get `sr-only` text.
5. **Tabular numerals on anything countable** (`font-variant-numeric: tabular-nums`); numbers right-aligned in tables; monospace for IDs/IPs/emails; worst-first ordering by default.
6. **Touch targets ≥ 24×24 CSS px** (WCAG 2.2 SC 2.5.8); 44px on mobile web. Visible focus rings; keyboard path through every flow.
7. **No emoji as UI icons.** Use a real icon set. No gradients, no cards inside cards, no glassmorphism unless the product is literally glass.
8. **One loud thing per screen.** The answer to "should I worry?" leads; everything else supports.
9. **Surfaces agree or explain the disagreement.** A zero on a filtered view says why it's zero (never stored / outside window / filtered / failed to load).
10. **UNKNOWN over inferred.** If the system can't attribute an outcome, display UNKNOWN — never a number inferred from counts.
11. **Honest copy.** Real limits, strikethrough instead of hidden features, no fake urgency, no confirm-shaming.
12. **Screenshot before and after every UI change.** A UI change without a before/after didn't happen.

## Process rules

- **Fix the test before the product.** Prove the harness is right before reporting a product bug.
- **One screen, one job.** If a screen needs paragraphs of explanation, the screen is wrong.
- **Single-reporter caution.** One person's UI claim gets investigated, not auto-adopted.
- **Errors escape-hatch.** Every blocking error links to the docs or action that gets the user unstuck.

Full standard with evidence and verification steps: [agent-ui-playbook](https://github.com/superz1402/agent-ui-playbook).

---
name: institutional-background
description: Research the institutional/legal background of a program, policy, or institution under study — laws, regulations, funding rules, eligibility criteria — via an interactive, source-verified process grounded in whatever's already on disk. Use when user says "find the laws behind X", "what's the legal basis of this program", "research institutional background", "who funds this and under what rules", "when was this policy created/amended/revoked". Every claim carries a source URL, retrieval date, and validity window; verified via claim-verifier before the report is finalized. Produces a report in quality_reports/institutional_background/ and updates the corpus index there.
argument-hint: "[topic or question] [--absorb] [--project <name>] [--no-verify]"
allowed-tools: ["Read", "Grep", "Glob", "Write", "Edit", "WebSearch", "WebFetch", "Task"]
disable-model-invocation: true
---

# Institutional Background Research

Research the legal/institutional background of a program, policy, or institution under study — laws, regulations, funding rules, eligibility criteria — interactively, grounded in what's already known, with every claim source-linked, dated, and independently verified.

**Input:** `$ARGUMENTS` — a topic or question about a program's institutional/legal background (e.g. "who funds this program and what are the eligibility rules"), plus optional flags.

**Where the corpus lives (read this first):** this skill needs somewhere to keep the actual retrieved documents (PDFs, saved page stubs), and that answer is project-specific — don't assume one:

1. **Check `CLAUDE.md` first.** If your project already documents a data/corpus location convention (e.g., an external data-root env var used for bulk material), use that, following whatever pattern is already established there.
2. **If nothing is documented, default to an in-repo `institutional/<program>/` folder, git-tracked.** [`.claude/rules/confidential-data.md`](../../rules/confidential-data.md) already treats small, non-confidential public-policy documents as safe to commit — unlike restricted research data, laws and regulations are public record, so keeping them in git (diffable, portable, no separate access setup for a collaborator) is usually the simpler right call for a fresh project.
3. **If your project has an external data root for bulk material**, put the corpus there instead and say so once, then record the choice in `CLAUDE.md` so future runs (and other skills) know where to look.

Either way, **git only ever holds the diffable index, passport, and dated reports** under `quality_reports/institutional_background/` — never a large binary corpus directly, if you chose the external-root path.

---

## Steps

### Phase 0 — Ground in what's already known

Before asking anything:

1. Read `quality_reports/institutional_background/INDEX.md` and `passport.yaml` if they exist. First time using this skill on a project? Copy the starter files from `templates/institutional-background-index-template.md` and `templates/institutional-background-passport-template.yaml` and fill them in as you go.
2. `Glob`/`Read` what's actually on disk at wherever Step 2 above (of "Where the corpus lives") established, to see what documents exist that the index may not know about yet.
3. Summarize back to the user in 3-5 lines: what program(s) are already covered, what's already verified, what gaps are already known — **before** asking a single question. If nothing exists yet, say so plainly and proceed; this is what makes the skill additive on every subsequent run rather than starting from scratch each time.

### Phase 1 — Build the research brief (interactive, no `AskUserQuestion`)

This is open-ended scoping, not a discrete choice — ask directly in prose, one or two questions at a time, and wait for the user, exactly like `/interview-me` does. Build these 8 elements (adapted from Chris Blattman's `deep-research` prompt schema):

1. **Task** — what's being researched, and what form the answer should take (a timeline? a funding map? an eligibility-rules summary?).
2. **Scope boundaries** — which program; date range (default to the project's own analysis window if `CLAUDE.md` or a project script's README documents one, offer to extend it); jurisdiction/level of government; which source types count.
3. **Evidence constraints** — primary sources only (a government's official gazette, a regulator's own site, a legislative database) vs. secondary (trade-association bulletins, news coverage) allowed as corroboration only, never as sole evidence for a claim.
4. **Output structure** — what should the findings be organized *by* (a timeline of acts? funding vs. eligibility vs. governance? one section per amendment?).
5. **Success + anti-goals** — e.g. "every claim needs a source + date + validity window" as success; "not a general policy history essay" as an explicit anti-goal.
6. **Comparative anchoring** — delta against what Phase 0 already showed is on disk; don't re-research what's already verified unless asked to re-check it.
7. **Self-verification** — the pre-finalization checklist to apply before the report is written (every claim sourced+dated? any zero-citation subtopics flagged rather than silently written as prose? any orphan claims with no validity window?).
8. **Tool trigger** — explicit: use live `WebSearch`/`WebFetch`, cite by URL, never answer a jurisdiction- or date-specific question from training data alone.

Stop after **typically 3-5 exchanges** — fewer if the user's initial ask was already specific (e.g. citing a named act or regulation).

### Phase 1.5 — Confirm the brief

Echo the built brief back in a short paragraph. Proceed on confirmation; take edits if offered. (Adapted from `deep-research`'s pre-dispatch gate — kept conversational rather than a CLI `[y/N/edit]` prompt.)

### Phase 2 — Dispatch research (inline, not forked)

Run this in the main conversation, not a forked subagent — staying in the same conversation keeps this phase a continuation of Phase 1's conversational style and lets the user watch progress or interject naturally, rather than getting a report back with no visibility into how it was produced.

- `WebSearch` to discover; `WebFetch` canonical URLs to ground each claim — prefer official/primary sources over secondary summaries, matching `claim-verifier`'s own stated bias.

  > **Example** (from this template's origin project — Brazilian federal health policy): `gov.br`, the `Diário Oficial da União`, and `BVS Saúde` are primary; CONASS bulletins are useful secondary corroboration but never a claim's sole source. Substitute your own jurisdiction's equivalents.

- For every claim, record: the claim text, source URL, retrieval date, and validity window (enacted date → amended/revoked date if known, else "in force as of retrieval").
- **Zero-citation gate:** if a subtopic returns fluent prose but no retrievable URL, flag it explicitly in the report's "Gaps / Not Verified" section — never present it as a sourced finding. (Adapted from `deep-research`'s "hollow-arm gate.")
- **Ambiguous-relevance clusters:** if findings split into a clear core cluster vs. one that's plausibly tangential to the confirmed Phase 1 brief (e.g. funding *sources* vs. the funding *criteria and incentives* they created), keep both — but route the tangential cluster to the report's "Flagged for Your Review" section instead of merging it into `## Findings`. Don't stop to ask; the user triages it when reading the report, not mid-run.

### Phase 3 — Verify

See **Post-Flight Verification** below — mandatory unless `--no-verify`.

### Phase 4 — Write outputs

1. Update `quality_reports/institutional_background/INDEX.md` — the corpus manifest (see format below).
2. Update `quality_reports/institutional_background/passport.yaml` — per-claim provenance.
3. Write `quality_reports/institutional_background/YYYY-MM-DD_<slug>.md` — the dated report (see Output Format below).
4. If new primary documents were retrieved, save them at wherever "Where the corpus lives" established, following whatever naming convention already exists there — or a `.md` stub with the verified citation/summary + source URL when a full-text fetch is blocked (say so explicitly rather than fabricating full text — see **Important** below).

---

## Output Format

```markdown
# Institutional Background: [Program / Topic]

**Date:** [YYYY-MM-DD]
**Research brief:** [1-2 sentence echo of the confirmed Phase 1 brief]
**Verification:** PASS | PARTIAL | FAIL (from Post-Flight reconciliation)

## Executive Summary

[2-3 sentences]

## Timeline

| Date | Act | What it did | Status |
|---|---|---|---|

## Findings

[organized per the brief's Element 4 — e.g. by theme, or chronologically]

### [Theme / period]
- **Claim.** Source: [URL] (retrieved YYYY-MM-DD). Validity: [enacted → amended/revoked, or "in force as of retrieval"]. Verification: PASS/EXPLAINED/etc.

## Flagged for Your Review

[clusters that seem tangential to the confirmed brief — included here rather than merged into `## Findings`, so you can decide their relevance while reading, not mid-run]

## Gaps / Not Verified

[zero-citation subtopics, single-sourced claims, anything explicitly out of scope this pass]

## Corpus Updates

- New documents saved to: [wherever "Where the corpus lives" established]
- `INDEX.md` / `passport.yaml`: N new entries, M re-verified, K flagged STALE

## Sources

1. [URL] — [one-line description], retrieved [date]
```

## Corpus Index & Passport

`quality_reports/institutional_background/INDEX.md` — one row per document known to exist:

| Program | Document | Storage path | Official citation | Source URL | Retrieved | Tier |
|---|---|---|---|---|---|---|

`quality_reports/institutional_background/passport.yaml` — one entry per claim, modeled on [`.claude/rules/replication-protocol.md`](../../rules/replication-protocol.md)'s `passport.yaml`. Starter files with the full schema: [`templates/institutional-background-index-template.md`](../../../templates/institutional-background-index-template.md) and [`templates/institutional-background-passport-template.yaml`](../../../templates/institutional-background-passport-template.yaml) — copy these once per project.

Status enum mirrors `replication-protocol.md`'s `passport.yaml` (`PASS | FAIL | EXPLAINED | STALE | UNVERIFIED`), with `STALE` redefined for this domain since there's no `source_file` to diff against:

**`STALE` semantics — read carefully.** A claim recorded *with* its validity window (a law that was later amended or revoked, correctly dated as such) is a permanent `PASS` — it already encodes its own history and does not rot. `STALE` applies only to claims recorded *without* a validity window, once enough time has passed since `last_verified_on` that the underlying law could plausibly have changed underneath an under-specified claim. Never mark a correctly-scoped historical claim `STALE` just because the law it describes was later superseded. `UNVERIFIED` is for entries `--absorb` creates from documents on disk before any claim has actually been checked by `claim-verifier` — should not persist once a research run touches that program.

---

## Flags

- `--absorb` — index-only mode: catalog documents already sitting wherever "Where the corpus lives" established that aren't yet in `INDEX.md`/`passport.yaml`. No new web research. Good first invocation on a project that already has a folder of institutional documents collected by hand.
- `--project` `<name>` — route to a specific program subfolder (e.g. a program's short name or acronym). If omitted, infer from the topic and confirm with the user; prompt explicitly if ambiguous or if it would create a new subfolder.
- `--no-verify` — skip Phase 3 / Post-Flight Verification. Same convention as `/lit-review` and `/verify-claims` — use for speed-critical iterations, not for anything headed into the manuscript.

---

## Post-Flight Verification (mandatory, CoVe)

Before finalizing the report, run the Post-Flight Verification protocol from [`.claude/rules/post-flight-verification.md`](../../rules/post-flight-verification.md). Legal/institutional claims are **high** hallucination risk for the same reason literature citations are: `WebSearch` can return plausible-sounding but wrong act numbers, dates, or article citations, and a superseded act can be mistaken for the current one.

### Steps

1. **Extract claims** from the draft — every "Act/Regulation X governs Y," every date, every "this was amended/revoked by Z," every funding figure.
2. **Generate verification questions** per claim, specific enough to check against the source alone.

   > **Example:** "Does [Act X], Article 9, actually establish the mechanism this claim describes, or was Article 9 already replaced by [Act Y] by the retrieval date?"

3. **Spawn `claim-verifier`** via `Task` with `subagent_type=claim-verifier` and `context=fork`. Pass the claims table, the verification questions, and the source pointers (URLs, corpus paths). **Do NOT pass the draft report itself.**
4. **Reconcile:** PASS → attach a green Post-Flight block. PARTIAL → mark unverifiable claims with uncertainty flags. FAIL → correct or remove the contradicted claim using the verifier's evidence before returning; update its passport entry accordingly.

### Skip conditions

- `--no-verify` flag.
- `--absorb` mode (no new claims are made — it only catalogs existing documents; nothing to verify).

### Output contract

Append a Post-Flight block to the report (same shape as the rule doc's template), and write the resulting `status` for each claim into `passport.yaml`.

---

## Important

- **Primary sources over secondary.** A government's official gazette or a regulator's own site over trade-association bulletins or news; secondary sources may corroborate but never stand alone as a claim's only evidence.
- **Every claim needs a validity window**, not just a date — this is what makes `STALE` tracking meaningful instead of noisy.
- **Be honest about blocked fetches.** Some official legal-text databases refuse automated fetching — if that happens, say so explicitly and record a citation-only stub rather than fabricating full text.
- **Don't bury ambiguous findings, and don't interrupt the run to ask about them either.** If findings split into a core cluster and a plausibly-tangential one, label the tangential cluster in "Flagged for Your Review" — never merge it into `## Findings` unlabeled, and never stop the run to ask which one the user wants.

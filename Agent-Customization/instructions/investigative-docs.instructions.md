---
name: Investigative docs
description: Claim hygiene for OSINT, leak, and research markdown — quotes, ground truth, completion fiction
applyTo: "**/*.md"
---

# Investigative documentation instructions

Copy or symlink this file into a project’s `.github/instructions/` (or adapt `applyTo` globs). Pair with hub seeding:

- `/home/gene/projects/ai-efficiency/seeding/NARRATIVE_AND_OSINT_GUARDRAILS.md`
- Continue rule: `ai-efficiency/continue/rules/04-narrative-claim-hygiene.md` (via `setup_project.sh`)

## Always

- **Public / allowed sources only** per the project’s ROE. Label unverified material **claim** or `[UNVERIFIED]`.
- **Quotes:** verbatim from primary text (catalog, OCR, transcript, filing), or explicitly labelled paraphrase. Never invent primary quotations.
- **Ground-truth diff:** before persisting dates, leaders, election cast, vote totals, or “who holds which job,” read the project’s README / master timeline / ground-truth table.
- **Done ≠ recommended:** do not write fixed / rewritten / recovered / proven / complete unless tool output or an on-disk path shows it.
- **Primary vs desk_echo:** analytical briefs that repeat a hypothesis are not primary exhibits. Do not score self-citation as confirmation.
- **Identity layers:** publisher / first poster ≠ typist ≠ access feeder (use the project’s L1≠L3≠L4 or equivalent). Platform choice alone does not name a person.
- Prefer surgical edits; append chronologies rather than rewriting history silently.
- When correcting a prior overclaim, append a short **Was → Corrected** log.

## Narratives / Substack companions

If the project uses `--substack` / `--narrative` (hub `.ai/response_controls.md`):

- Same epistemic conclusions as the technical brief — do not upgrade claims to proof for drama.
- Cross-link the source audit/matrix at the top.
- Run the checklist in `NARRATIVE_AND_OSINT_GUARDRAILS.md` before persist.

## Do not

- Compress multi-week timelines into “overnight” or “within 48 hours” without a dated source.
- Treat HTTP 200 on a search page as binary recovery.
- Treat “tip-line = 0” as dispositive if live tip pages were never fetched.
- Park secrets, private DMs, or non-public personal data in docs (see hub `CLOUD_LLM_SENSITIVITY.md`).

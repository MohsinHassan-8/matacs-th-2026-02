# Write-up: AI Proposal Triage Design (MATACS-TH-2026-02)

## 1. AI vs. rules

AI is used in one step only: fact extraction (stage 5). It reads free-text proposals and returns structured facts, each with a word-for-word evidence quote, or "not stated". Reading messy prose is the one job rules cannot do well.

Everything else is plain code: text extraction, injection scan, validation, mandatory pass/fail, weighted scoring, ranking and the audit log. These steps must be repeatable, explainable line by line, and designed to resist instructions hidden in a proposal.

The line is: AI reads, code decides, a human approves. The AI never sees the weights, never scores and never ranks, so a mistaken or manipulated AI cannot change the ranking directly. Its facts are checked by code (the quote must exist in the source) before they are used. Subjective criteria such as "team and approach" are reduced to yes/no facts (named lead, crew, safety plan, monitoring plan) that code scores from a checklist.

## 2. Consistency and explainability

Scores come from fixed formulas in code, so the same facts always give the same score. The only variable part is AI extraction, which is controlled four ways: temperature 0, a pinned model version, a strict JSON schema, and code checks that every evidence quote exists in the source text. Extracted facts are saved with each run, so re-ranking the same batch gives an identical result. A forced re-extraction is compared with the saved facts, and any difference is flagged. Two AI calls may not be word-for-word identical, which is why the facts are stored.

The brief gives weights but no formulas, so these are my assumptions, held as admin-editable settings: experience is installs capped at 10; price and timeline are the best eligible value divided by the proposal's value; warranty is years against the best eligible years, split evenly between panel and workmanship; team is equal points for each of four yes/no items.

A reviewer sees a score card per proposal: every criterion with the extracted fact, the quote that supports it, the formula and the points. The one-line reason is generated from this card by a template, not written freely by AI. Price and timeline depend on the best eligible bid, so those benchmark values are logged with each run. Ties go to the lower price; scores are rounded for display only.

## 3. Human in the loop

A person acts at four points. (1) Upload: the reviewer chooses the category and sees which criteria version applies. (2) Flag queue: every flagged proposal (missing data, quote not found, suspicious text, unclear mandatory item) must be resolved by the reviewer, who can confirm the fact against its quote, ask the contractor, or leave it excluded. (3) Ranking review: the reviewer can move a proposal or reinstate an ineligible one, but a written reason is required for every override. (4) Award: only a person awards the contract; the tool never does.

The audit log is append-only and written automatically at every stage. It records the criteria version, prompt version, model and settings, a hash of each uploaded file, the extracted text, every extracted fact with its quote, flags, scores, benchmark values and the ranking. It also records each human action with user, time and reason, and the final decision.

## 4. Changing criteria

An admin edit creates a new immutable version and never overwrites the old one; weights must total 100% or the save is blocked. Every run stores the version it used, so an already-scored proposal keeps its score, labelled with that version, and nothing changes silently.

A batch scored under an older version shows a banner that the criteria have changed. The reviewer can keep the old result or re-score under the new version. Re-scoring creates a new run and keeps the old one for audit. A batch is never ranked with mixed versions.

Re-scoring is cheap because extracted facts are stored separately from the criteria: code re-applies the formulas to saved facts with no new AI call. Only a newly added field needs fresh extraction. Changing a mandatory criterion can change eligibility, so flags are recomputed and the flag queue reopens. After an award is recorded, the run and the criteria version are locked.

## 5. Risks

- **Wrong extraction:** every fact needs a verbatim quote that code finds in the source text, or the proposal is flagged. Reviewers see the quote beside each score. Poor scans are flagged for manual entry.
- **Made-up facts:** the AI may only report stated text and is allowed to answer "not stated". Missing data scores 0 and is flagged, never guessed.
- **Manipulation:** proposal text is treated as data. A pattern scan removes instruction-like passages from the AI input and flags them, but cannot catch every attack, so the AI also reports any instruction the scan missed. The original text stays unchanged in the audit log, and the AI never sees weights or does scoring. In the sample set, C contains a line telling automated reviewers to rank it first; it is ignored, flagged and scored on its genuine content only.
- **Bias:** scoring formulas use only extracted facts and the admin's criteria, never company name, size or location; a human reviews the result.
- **Sample edge cases:** B names a licensed subcontractor "to be confirmed", so it is ineligible; D states no timeline, so it is flagged. A, C and E state experience as "since" a year, so reviewers confirm the five-year window before award; if it fails, the count is corrected and the proposal re-scored.
- **Cost:** one AI call per proposal; facts are cached and re-scoring is done in code. Page limits and a per-batch spending cap apply.
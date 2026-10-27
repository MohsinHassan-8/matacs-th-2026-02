# matacs-th-2026-02

**Candidate:** Mohsin Hassan
**Reference:** MATACS-TH-2026-02
**Time actually spent:** about 1.5 hours

## Summary

A hybrid design for AI-assisted proposal triage: AI only extracts facts (each with an evidence quote), plain code does eligibility, scoring and ranking, and a human reviews flags and makes the final award.

## Files

- [WRITEUP.md](WRITEUP.md): answers to the five questions
- [PROMPTS.md](PROMPTS.md): extraction prompt, output schema and post-checks
- [ranking.json](ranking.json): expected ranking of proposals A to E
- [diagram/process-flow.png](diagram/process-flow.png): flow diagram from upload to ranked output

No prototype is included (optional).

## How I used AI tools

I used Claude throughout: to compare design approaches, draft the prompt, scoring formulas and write-up wording, and to generate the process-flow diagram, while I reviewed, ran and committed each step myself. I chose the hybrid design (AI reads, code decides, a human approves) because it is repeatable, explainable and resistant to instructions hidden in proposals. Corrections: the AI's first extraction prompt referred to a schema "below" that was not in the prompt, so the schema was moved into the user message template; the AI also first suggested only Proposal C's experience dates might fall outside the 5-year window, then corrected that A and E have the same issue, so the design leaves that date check to the human reviewer.
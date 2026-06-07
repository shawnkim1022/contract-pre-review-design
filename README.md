# Contract Pre-Review Engine

A deterministic NDA / contract pre-review engine for people who sign contracts they can't fully vet: solo founders and small teams with no in-house counsel, and the lawyers doing first-pass triage for them. A contract goes in, a prioritized list of what to question comes out. No external LLM calls.

> Public overview and design docs. The source lives in a separate private repo, and full code walkthroughs happen in interviews.
>
> This is not a lawyer, and its output is not legal advice. It does not decide whether a contract is valid or enforceable, and it does not replace attorney review. It points a qualified lawyer at the clauses worth a second look.

## The problem

Reading an NDA is not the hard part. Knowing which clauses actually matter, and never being confidently wrong about it, is. In law a hallucinated "this looks fine" is the worst output a tool can produce.

A second problem sits underneath that one. The output has to be auditable. A lawyer who can't see why a clause was flagged, or who gets a different answer on a re-run, won't trust the tool twice. So every flag has to trace to an exact span of the contract and a named rule, and the same contract has to produce the same result every time. The whole system is built around one question: how do you surface what matters without ever pretending to be a lawyer?

## Ontology

Clauses resolve onto an explicit domain model, not free text. The objects are a Clause (carrying its exact character offsets in the source), a Finding, an escalation Rule, and a Party (ten roles, including the unknown and both-parties cases real contracts force on you). They link the way you would expect. A Finding points at the Clause it came from. A Clause points at its parent clause for document structure. An original clause links to its redline counterpart for version diffs. A Finding traces back through the detector that fired to the Rule that governs it.

Under all of it sits the part that keeps the system honest: a governed risk taxonomy of 73 entries across 15 categories, guarded against duplicate IDs, that every detector has to classify into. Training-data use without opt-in, source-code access, residual knowledge, broad affiliate or subcontractor disclosure, perpetual confidentiality, liquidated-damages multipliers, unlimited or consequential liability, counterparty-friendly jurisdiction, protections lost on downstream agreements, and so on. The taxonomy is the closed vocabulary. Detectors classify into it, rules reason over it, reports render from it, and nothing reaches a reader that isn't tied to a taxonomy entry plus the clause span that triggered it.

This typed layer is what makes every downstream step checkable.

## Data

The domain data is the risk taxonomy plus the Korean and English clause lexicons and slot patterns (durations, amounts, multipliers, notice periods, liability caps) that map real phrasings onto canonical findings. Input is a contract file (TXT, MD, DOCX, PDF) read locally and normalized deterministically, so segmentation stays clean: control characters and page numbers come out, wrapped lines get joined conservatively, and article markers like 제N조 or Article N are preserved.

The evaluation data is a synthetic corpus of 84 labeled items across 11 fixtures, stratified by difficulty (one easy, two medium, seven hard, one expert), each carrying a ground-truth target. Real contracts never enter version control. Answer keys are metadata only, with zero raw clause text, and a validator rejects raw-text fields and PII (email, phone, absolute paths) before evaluation runs. The evaluator refuses to print clause bodies, and when real data is absent it claims no result rather than degrading silently.

## Logic

Normalization, then hard pattern detectors, then a structural-asymmetry scorer, then escalation. The hard detectors (regex, keyword, and slot extraction at the clause and document level) recognize known risky shapes and map them into the taxonomy. The scorer is a second pass for one-sidedness: it sums weighted signals and surfaces a clause for review once it crosses a fixed threshold of 25 points, and it stays quiet when a hard detector already fired. Behind both sits a safety net that flags clauses no detector matched but that still look like they need a person: an unknown category, numbers with no matching rule, dense multi-condition language that buries an obligation. The engine never treats an unrecognized clause as safe.

All of it is deterministic. Zero LLM calls, zero random calls on the runtime path. Same contract, same findings, same IDs, same scores, every time. That choice is written down as an accepted architecture decision with five protected invariants and a six-condition gate that would have to be met, and re-decided with a lawyer in the loop, before any model could touch the path a reader sees. A reasoning model can still help off the path, writing adversarial test fixtures or proposing candidate rules, but its output has to become a deterministic rule or a frozen, human-reviewed template before it reaches anyone.

## Action

State-changing decisions are explicit and bounded. Every finding runs through an absolute-rules engine of 53 rules that assigns one of three outcomes: hard_reject, mandatory_lawyer_review, or business_confirm_required. Twenty-three of those rules are non-overridable, all of them on the lawyer-review tier. The other thirty can be overridden, and the type model carries an audit trail (who overrode it, why, whether a lawyer approved) for when persistence gets built, which is out of scope for v0.1. The escalation is the action. It tells a person what to do next.

What the engine will not do is decide. It never emits a "safe", an "approve", or a "sign it". It produces flags, the source evidence behind them, and a routing decision toward a person. A run writes a plain-Korean explanation for a business reader, a structured handoff for a lawyer, and JSONL records (findings, fingerprints, run history) so a person can diff results between detector changes. The refusal to render a verdict is deliberate. It keeps the tool on the right side of Korean Attorney-at-Law Act Article 109, where unauthorized practice of law is a real liability.

## How I know it works

The part worth showing is the layer that measures the engine. Findings are scored against the 84 labeled items, graded by which detector layer actually fired (specific, structural, generic, or missed) rather than by a hint the answer key declared, so the engine cannot grade itself generously. any-hit recall and specific recall are reported separately. False positives are counted per layer. Every fixture carries negative controls (clauses that must not fire), and one fixture exists only to catch over-flagging. Because the runtime is deterministic, a re-run diff is zero by construction, and a local check spins up two throwaway worktrees and fails the change on any recall regression or any new false positive.

On that synthetic benchmark the engine reaches 89.3% any-hit recall (75 of 84) and 78.6% specific recall (66 of 84), with 4 false positives. Read those as pattern-recall against labels I wrote myself, not as real-world accuracy. There are no lawyer-validated labels yet. Twice, the measurement layer changed my mind about what was actually broken, which is the reason to build it.

## Honest limitations

Pre-deployment and solo-built. Every number here comes from synthetic stress tests. No real-contract run has happened yet, and the difficulty mix and clause coverage are hypotheses to be re-weighted against real use. Detector breadth is partial, the Korean-law citation set is deliberately narrow, and there is no CI; the test suites and the regression check are gates I run by hand. The thing that would move it forward most is lawyer-validated labels. Without them, neither real accuracy nor a consistent material miss can be established.

## Design docs

The full v0.1 architecture specification is in [`docs/contract-pre-review-engine-v0.1.md`](docs/contract-pre-review-engine-v0.1.md): product definition, clause taxonomy, severity framework, company playbook, validation layer, Korean-law verification layer, lawyer-handoff format, MVP plan, test cases, and non-goals.

## License and use

Design and architecture docs, shared for review. The source is private and proprietary. Nothing here is for distribution as legal advice or as a substitute for counsel, and the engine's output is decision-support for a qualified lawyer to confirm, never an autonomous decision.

## Contact

Code walkthroughs and interview scheduling: shawnkim1022@gmail.com

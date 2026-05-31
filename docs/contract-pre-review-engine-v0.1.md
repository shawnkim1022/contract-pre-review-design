# Contract Pre-Review Engine v0.1

> **Status:** v0.1 architecture draft
> **Vertical:** B2B SaaS / AI / software startup contract pre-review
> **Stance:** Deterministic, playbook-based, evidence-anchored. Not legal advice.
> **Design lineage:** Adapts a deterministic, rule-first NLP pipeline pattern (input normalization → lexicon → classifier ensemble → state machine → validation → confirm → structured output).

---

## 1. Product Definition

**Contract Pre-Review Engine** is a contract pre-review and risk triage system that helps B2B SaaS / AI / software startups review received NDA/MSA/license agreements **before** sending issues to a lawyer.

What it is:

- A structured contract risk analysis tool.
- A clause-by-clause severity triage engine driven by a company playbook.
- A lawyer handoff generator (concise, top-issues-first).
- A negotiation-aid generator (counterproposal wording, business confirmation prompts).
- A redline comparison tool (template vs. counterparty redline).

What it is **not**:

- It is **not legal advice**.
- It **does not replace a lawyer**.
- It is **not a general legal chatbot**.
- It is **not a "safe to sign" approver**.
- It is **not a guarantee of enforceability**.

Why it exists:

- B2B SaaS / AI startups receive NDAs, MSAs, and license agreements faster than legal counsel can review every clause.
- Founders need a deterministic first-pass triage that highlights critical/high issues, surfaces playbook deviations, and prepares the precise questions a lawyer should answer.
- The product compresses the founder ↔ lawyer feedback loop by structuring the contract risk before it reaches a lawyer's desk.

Key product promises:

- It highlights critical/high findings with evidence anchored to the original clause text.
- It detects missing clauses against the company playbook.
- It generates lawyer questions and negotiation suggestions, but only after deterministic findings are produced.
- It logs every user decision (accept / negotiate / escalate / ignore) for auditability.

---

## 2. Architecture Overview

The engine is a deterministic, rule-first pipeline. Input modality is text (PDF/DOCX/TXT); the validation domain is contract clauses.

| pipeline layer | role |
|---|---|
| input normalization | PDF/DOCX/TXT contract text normalization |
| clause taxonomy + lexicon | clause category + legal/business lexicon |
| classifiers | clause category + slot extraction classifiers |
| review state machine | explicit review state transitions |
| validation | playbook + legal/rule validation |
| confirm | user/lawyer confirm before action |
| structured output | structured risk report / lawyer handoff JSON |

The core pipeline:

- **input normalization** — strip noise, normalize encoding, segment the document into atomic clauses.
- **lexicon lookup** — deterministic keyword/regex matching as the first-pass classifier.
- **classifier ensemble** — rule-first, then classifier, never LLM as source of truth.
- **state machine** — explicit review state transitions (parsing → classifying → validating → awaiting confirm → exported).
- **validation layer** — playbook rules + cross-clause consistency + missing-clause detection.
- **confirm step** — human-in-the-loop before any externally visible action (no auto-signing, no auto-emailing).
- **structured output** — JSON + Markdown report + lawyer handoff bundle.

Core architectural principle: **the LLM is never the source of truth.** Severity, playbook deviation, missing-clause detection, and clause classification must be deterministic. The LLM is used only for plain-language explanation and wording generation *after* deterministic findings exist.

---

## 3. Core Data Flow

```
Contract file / text
  → normalize text
  → split into clauses
  → classify clause type
  → extract slots
  → detect missing clauses
  → compare against company playbook
  → assign severity
  → generate evidence-linked findings
  → generate lawyer questions
  → generate negotiation suggestions
  → user confirm
  → export report
```

### 3.1 Stage details

1. **normalize text** — Convert PDF/DOCX/TXT to canonical UTF-8 text. Strip headers/footers, page numbers, and OCR artifacts. Preserve paragraph boundaries and article/section numbering.
2. **split into clauses** — Detect article numbering (제1조, Article 1, 1.1, etc.) and segment by clause. Each clause gets a `clauseId`, `articleNumber`, `originalText`, and offsets.
3. **classify clause type** — Map each clause to one or more `ClauseCategory` values (e.g., `confidentiality_period`, `liquidated_damages`). Rule-first via taxonomy lexicon, fallback to classifier model.
4. **extract slots** — For each classified clause, extract structured slots (e.g., `period_years: 10`, `damages_multiplier: 5x`, `jurisdiction: "Seoul Central District Court"`).
5. **detect missing clauses** — Compare detected categories against the company playbook's required-clause list. Generate "missing_clause_add" findings for absent clauses.
6. **compare against company playbook** — Evaluate `PlaybookRule` conditions against extracted slots.
7. **assign severity** — Deterministic severity per playbook rule. Cross-clause contradictions can elevate severity.
8. **generate evidence-linked findings** — Each finding stores `evidence` (original clause excerpts), `reasonCodes`, and `recommendedAction`.
9. **generate lawyer questions** — Template-driven; LLM-assisted wording is allowed *after* deterministic question slot is filled.
10. **generate negotiation suggestions** — Fallback wording from playbook, LLM-assisted rewriting optional.
11. **user confirm** — UI surfaces findings, user marks decisions per finding. Decisions are logged.
12. **export report** — Markdown report, JSON report, lawyer handoff summary, negotiation email draft, redline review JSON.

### 3.2 Output formats

- **Markdown report** — human-readable, sectioned by severity.
- **JSON report** — machine-readable, contains all `ClauseFinding` objects.
- **Lawyer handoff summary** — concise top-issues-first document for legal counsel.
- **Negotiation email draft** — counterparty-facing email with proposed redlines.
- **Redline review JSON** — diff-anchored findings comparing template vs. received version.

---

## 4. Clause Taxonomy

The taxonomy is the single source of truth for clause category recognition.

### 4.1 NDA Core

- `parties`
- `definition_of_confidential_information`
- `exclusions_from_confidential_information`
- `purpose_limitation`
- `permitted_disclosure`
- `affiliate_disclosure`
- `representative_disclosure`
- `standard_of_care`
- `compelled_disclosure`
- `return_or_destruction`
- `confidentiality_period`
- `residual_knowledge`
- `no_license_or_ip_transfer`
- `no_reverse_engineering`
- `remedies`
- `liability`
- `liquidated_damages`
- `injunction`
- `governing_law`
- `jurisdiction`
- `assignment`
- `survival`
- `entire_agreement`
- `amendment`
- `notices`

### 4.2 SaaS / AI / Software Add-on

- `source_code_access`
- `api_credentials`
- `customer_data`
- `personal_data`
- `training_data_use`
- `derived_data_rights`
- `logs_and_analytics`
- `security_vulnerability_information`
- `benchmark_or_performance_results`
- `public_reference_or_marketing_rights`
- `model_output_confidentiality`
- `feedback_rights`
- `integration_scope`
- `service_level_or_accuracy_claims`
- `support_and_maintenance`
- `data_retention_and_deletion`

### 4.3 MSA / License Add-on

- `license_grant`
- `scope_of_use`
- `restrictions`
- `sublicensing`
- `payment_terms`
- `renewal`
- `termination`
- `effect_of_termination`
- `warranty_disclaimer`
- `limitation_of_liability`
- `indemnification`
- `change_control`
- `audit_rights`
- `compliance_with_laws`

---

## 5. TypeScript-Friendly Schema

```ts
type Severity = "critical" | "high" | "medium" | "low" | "info";

type RecommendedAction =
  | "accept"
  | "negotiate"
  | "lawyer_review"
  | "reject"
  | "missing_clause_add"
  | "business_confirm";

type ClauseCategory =
  | "parties"
  | "definition_of_confidential_information"
  | "exclusions_from_confidential_information"
  | "purpose_limitation"
  | "permitted_disclosure"
  | "affiliate_disclosure"
  | "representative_disclosure"
  | "standard_of_care"
  | "compelled_disclosure"
  | "return_or_destruction"
  | "confidentiality_period"
  | "residual_knowledge"
  | "no_license_or_ip_transfer"
  | "no_reverse_engineering"
  | "remedies"
  | "liability"
  | "liquidated_damages"
  | "injunction"
  | "governing_law"
  | "jurisdiction"
  | "assignment"
  | "survival"
  | "entire_agreement"
  | "amendment"
  | "notices"
  | "source_code_access"
  | "api_credentials"
  | "customer_data"
  | "personal_data"
  | "training_data_use"
  | "derived_data_rights"
  | "logs_and_analytics"
  | "security_vulnerability_information"
  | "benchmark_or_performance_results"
  | "public_reference_or_marketing_rights"
  | "model_output_confidentiality"
  | "feedback_rights"
  | "integration_scope"
  | "service_level_or_accuracy_claims"
  | "support_and_maintenance"
  | "data_retention_and_deletion"
  | "license_grant"
  | "scope_of_use"
  | "restrictions"
  | "sublicensing"
  | "payment_terms"
  | "renewal"
  | "termination"
  | "effect_of_termination"
  | "warranty_disclaimer"
  | "limitation_of_liability"
  | "indemnification"
  | "change_control"
  | "audit_rights"
  | "compliance_with_laws"
  | string;

interface ClauseSpan {
  clauseId: string;
  title?: string;
  articleNumber?: string;
  startOffset?: number;
  endOffset?: number;
  originalText: string;
}

interface ExtractedSlot {
  key: string;
  value: string | number | boolean | string[];
  unit?: string;
  confidence: number;
  evidenceText: string;
}

interface ClauseFinding {
  findingId: string;
  clause: ClauseSpan;
  category: ClauseCategory;
  slots: ExtractedSlot[];
  playbookRuleIds: string[];
  severity: Severity;
  reasonCodes: string[];
  evidence: string[];
  recommendedAction: RecommendedAction;
  suggestedFallback?: string;
  lawyerQuestions?: string[];
  legalBasis?: LegalBasis[];
}

interface LegalBasis {
  lawName: string;
  articleNumber: string;
  articleTitle: string;
  verificationStatus: "verified" | "unverified" | "not_found" | "api_error";
  apiSource: string;
  retrievedAt: string;
  reasonForMapping: string;
  lawyerQuestion: string;
}
```

The `legalBasis` field is populated by the Korean Law API Verification Layer (Section 9). It is optional because the verification layer is enrichment, not the source of severity or recommended action.

### 5.1 JSON examples

#### confidentiality_period

```json
{
  "findingId": "f-001",
  "clause": {
    "clauseId": "c-014",
    "title": "Confidentiality Period",
    "articleNumber": "Article 11",
    "startOffset": 4821,
    "endOffset": 5012,
    "originalText": "The receiving party shall maintain the confidentiality of the Confidential Information for a period of ten (10) years from the date of disclosure."
  },
  "category": "confidentiality_period",
  "slots": [
    { "key": "period_years", "value": 10, "unit": "years", "confidence": 0.98, "evidenceText": "a period of ten (10) years" },
    { "key": "perpetual", "value": false, "confidence": 0.99, "evidenceText": "ten (10) years from the date of disclosure" }
  ],
  "playbookRuleIds": ["pb-confidentiality-period-max-5y"],
  "severity": "high",
  "reasonCodes": ["confidentiality_period_exceeds_company_max"],
  "evidence": ["The receiving party shall maintain the confidentiality of the Confidential Information for a period of ten (10) years from the date of disclosure."],
  "recommendedAction": "negotiate",
  "suggestedFallback": "The receiving party shall maintain the confidentiality for a period of five (5) years from the date of disclosure; provided that trade secrets shall remain confidential for as long as they qualify as trade secrets under applicable law.",
  "lawyerQuestions": [
    "Is a 10-year confidentiality period enforceable under the applicable governing law for the type of information being disclosed?",
    "Should trade secrets be carved out from the fixed time limit?"
  ]
}
```

#### liquidated_damages

```json
{
  "findingId": "f-002",
  "clause": {
    "clauseId": "c-021",
    "title": "Liquidated Damages",
    "articleNumber": "Article 14",
    "originalText": "Upon any breach of confidentiality, the breaching party shall pay liquidated damages equal to five (5) times the actual damages incurred."
  },
  "category": "liquidated_damages",
  "slots": [
    { "key": "damages_multiplier", "value": 5, "unit": "x", "confidence": 0.97, "evidenceText": "five (5) times the actual damages" },
    { "key": "trigger", "value": "any breach of confidentiality", "confidence": 0.95, "evidenceText": "Upon any breach of confidentiality" }
  ],
  "playbookRuleIds": ["pb-liquidated-damages-max-2x"],
  "severity": "critical",
  "reasonCodes": ["liquidated_damages_multiplier_exceeds_threshold"],
  "evidence": ["Upon any breach of confidentiality, the breaching party shall pay liquidated damages equal to five (5) times the actual damages incurred."],
  "recommendedAction": "lawyer_review",
  "suggestedFallback": "Each party shall be liable for direct damages actually incurred as a result of a breach of this Agreement, subject to the limitation of liability set forth in Article __.",
  "lawyerQuestions": [
    "Is a 5x liquidated damages multiplier enforceable or would it be deemed a penalty under applicable law?",
    "Should we propose actual damages with a liability cap instead?"
  ]
}
```

#### training_data_use

```json
{
  "findingId": "f-003",
  "clause": {
    "clauseId": "c-007",
    "title": "Use of Data",
    "articleNumber": "Article 6",
    "originalText": "Provider may use any data submitted via the Service to train, improve, or develop its machine learning models without further notice or consent."
  },
  "category": "training_data_use",
  "slots": [
    { "key": "training_allowed", "value": true, "confidence": 0.99, "evidenceText": "Provider may use any data ... to train" },
    { "key": "opt_in_required", "value": false, "confidence": 0.99, "evidenceText": "without further notice or consent" }
  ],
  "playbookRuleIds": ["pb-training-data-opt-in-required"],
  "severity": "critical",
  "reasonCodes": ["training_data_used_without_opt_in"],
  "evidence": ["Provider may use any data submitted via the Service to train, improve, or develop its machine learning models without further notice or consent."],
  "recommendedAction": "lawyer_review",
  "suggestedFallback": "Provider shall not use Customer Data for training, improving, or developing any machine learning models without Customer's prior written opt-in consent. Aggregated and de-identified data may be used solely to operate and improve the Service.",
  "lawyerQuestions": [
    "Does the current draft violate our internal data governance policy or any DPA we have signed with our own customers?",
    "Should we require an opt-in checkbox and a separate Data Processing Addendum?"
  ]
}
```

#### jurisdiction

```json
{
  "findingId": "f-004",
  "clause": {
    "clauseId": "c-030",
    "title": "Governing Law and Jurisdiction",
    "articleNumber": "Article 22",
    "originalText": "This Agreement shall be governed by the laws of Delaware, USA, and any dispute shall be resolved exclusively in the state and federal courts located in Wilmington, Delaware."
  },
  "category": "jurisdiction",
  "slots": [
    { "key": "governing_law", "value": "Delaware, USA", "confidence": 0.98, "evidenceText": "laws of Delaware, USA" },
    { "key": "exclusive_venue", "value": "Wilmington, Delaware", "confidence": 0.98, "evidenceText": "exclusively in the state and federal courts located in Wilmington, Delaware" }
  ],
  "playbookRuleIds": ["pb-jurisdiction-prefer-korea"],
  "severity": "high",
  "reasonCodes": ["jurisdiction_outside_preferred_venue"],
  "evidence": ["This Agreement shall be governed by the laws of Delaware, USA, and any dispute shall be resolved exclusively in the state and federal courts located in Wilmington, Delaware."],
  "recommendedAction": "negotiate",
  "suggestedFallback": "This Agreement shall be governed by the laws of the Republic of Korea, and any dispute shall be resolved by the Seoul Central District Court as the court of first instance.",
  "lawyerQuestions": [
    "What is our cost exposure if forced to litigate in Delaware?",
    "Is a neutral arbitration venue (e.g., SIAC, KCAB) an acceptable fallback?"
  ]
}
```

#### limitation_of_liability

```json
{
  "findingId": "f-005",
  "clause": {
    "clauseId": "c-024",
    "title": "Limitation of Liability",
    "articleNumber": "Article 16",
    "originalText": "Each party shall be liable for all damages, including direct, indirect, incidental, consequential, and punitive damages, without limitation."
  },
  "category": "limitation_of_liability",
  "slots": [
    { "key": "liability_cap_exists", "value": false, "confidence": 0.99, "evidenceText": "without limitation" },
    { "key": "indirect_damages_included", "value": true, "confidence": 0.99, "evidenceText": "indirect, incidental, consequential, and punitive damages" }
  ],
  "playbookRuleIds": ["pb-liability-cap-required", "pb-no-indirect-damages"],
  "severity": "critical",
  "reasonCodes": ["no_liability_cap", "indirect_damages_included"],
  "evidence": ["Each party shall be liable for all damages, including direct, indirect, incidental, consequential, and punitive damages, without limitation."],
  "recommendedAction": "lawyer_review",
  "suggestedFallback": "Each party's aggregate liability under this Agreement shall not exceed the fees paid or payable by Customer in the twelve (12) months preceding the event giving rise to the claim. Neither party shall be liable for indirect, incidental, consequential, or punitive damages.",
  "lawyerQuestions": [
    "Is uncapped liability acceptable for this deal size, or do we require a cap tied to fees paid?",
    "Should we carve out specific categories (e.g., IP indemnification, gross negligence) from the cap?"
  ]
}
```

#### no_reverse_engineering

```json
{
  "findingId": "f-006",
  "clause": {
    "clauseId": "c-018",
    "title": "Reverse Engineering",
    "articleNumber": "Article 12",
    "originalText": "Customer may decompile, disassemble, or otherwise reverse engineer the Software for any purpose."
  },
  "category": "no_reverse_engineering",
  "slots": [
    { "key": "reverse_engineering_allowed", "value": true, "confidence": 0.99, "evidenceText": "may decompile, disassemble, or otherwise reverse engineer" }
  ],
  "playbookRuleIds": ["pb-no-reverse-engineering"],
  "severity": "critical",
  "reasonCodes": ["reverse_engineering_explicitly_allowed"],
  "evidence": ["Customer may decompile, disassemble, or otherwise reverse engineer the Software for any purpose."],
  "recommendedAction": "reject",
  "suggestedFallback": "Customer shall not, and shall not permit any third party to, decompile, disassemble, reverse engineer, or otherwise attempt to derive the source code, underlying ideas, algorithms, or structure of the Software, except to the extent expressly permitted by applicable law.",
  "lawyerQuestions": [
    "Is this clause a deal-breaker given our core IP exposure?",
    "Are there any interoperability carve-outs required under applicable law that we must preserve?"
  ]
}
```

---

## 6. Severity Framework

Severity is assigned by **deterministic rule-based criteria** first. LLM-generated severity is **not allowed** as the source of truth. LLM may explain *why* a deterministic severity was assigned, but it cannot upgrade or downgrade severity on its own.

### 6.1 `critical`

Use `critical` when:

- clause creates material liability without cap
- clause transfers core IP or ownership unexpectedly
- clause allows training on customer/company confidential data without opt-in consent
- clause removes a responsibility boundary that protects the company
- clause allows reverse engineering or source code access without control
- clause creates large liquidated damages or penalty exposure
- clause gives counterparty unilateral termination or broad audit rights that can materially disrupt business

### 6.2 `high`

Use `high` when:

- confidentiality period exceeds company max threshold
- jurisdiction is unfavorable
- affiliate disclosure is broad without NDA-equivalent duties
- personal data processing terms are missing
- return/deletion obligation is incomplete
- limitation of liability is missing or one-sided
- marketing/public reference rights are too broad

### 6.3 `medium`

Use `medium` when:

- wording is ambiguous
- notice period is shorter than playbook preference
- renewal terms are unfavorable but manageable
- minor operational burden exists
- clause requires business confirmation but is not automatically unacceptable

### 6.4 `low` / `info`

Use `low` / `info` when:

- standard clauses are present
- clause is acceptable but should be noted
- no action required

### 6.5 Cross-clause escalation

A deterministic severity may be elevated by the validation layer (Section 8) if a contradiction or compounding risk is detected. For example:

- `personal_data` is mentioned + `data_retention_and_deletion` clause is missing → escalate `data_retention_and_deletion` finding from `medium` to `high`.
- `liability` includes indirect damages + `limitation_of_liability` has no cap → escalate `limitation_of_liability` finding from `high` to `critical`.

---

## 7. Company Playbook Schema

```ts
interface PlaybookRule {
  ruleId: string;
  category: ClauseCategory;
  condition: {
    field: string;
    operator:
      | "exists"
      | "missing"
      | "equals"
      | "not_equals"
      | "greater_than"
      | "less_than"
      | "includes"
      | "excludes";
    value?: string | number | boolean;
  };
  severity: Severity;
  reasonCode: string;
  recommendedAction: RecommendedAction;
  fallbackText?: string;
  lawyerQuestionTemplate?: string;
}
```

### 7.1 Starter playbook for B2B SaaS / AI startups

| ruleId | category | condition | severity | recommendedAction |
|---|---|---|---|---|
| `pb-confidentiality-period-max-5y` | `confidentiality_period` | `period_years` greater_than `5` | `high` | `negotiate` |
| `pb-perpetual-confidentiality-general-info` | `confidentiality_period` | `perpetual` equals `true` AND `trade_secret_carveout` missing | `high` | `negotiate` |
| `pb-trade-secret-no-time-limit` | `exclusions_from_confidential_information` | `trade_secret_carveout` exists | `info` | `accept` |
| `pb-residual-knowledge-present` | `residual_knowledge` | `residual_knowledge_allowed` equals `true` | `high` | `lawyer_review` |
| `pb-training-data-opt-in-required` | `training_data_use` | `training_allowed` equals `true` AND `opt_in_required` equals `false` | `critical` | `lawyer_review` |
| `pb-derived-data-customer-owns` | `derived_data_rights` | `derived_data_owner` equals `counterparty` | `high` | `negotiate` |
| `pb-public-reference-consent-required` | `public_reference_or_marketing_rights` | `prior_written_consent_required` equals `false` | `high` | `negotiate` |
| `pb-jurisdiction-prefer-korea` | `jurisdiction` | `governing_law` not_equals `Republic of Korea` | `medium` | `negotiate` |
| `pb-liquidated-damages-max-2x` | `liquidated_damages` | `damages_multiplier` greater_than `2` | `critical` | `lawyer_review` |
| `pb-liability-cap-required` | `limitation_of_liability` | `liability_cap_exists` equals `false` | `high` | `negotiate` |
| `pb-liability-cap-vs-fees` | `limitation_of_liability` | `liability_cap_amount` less_than `fees_paid_in_period` | `medium` | `business_confirm` |
| `pb-no-reverse-engineering` | `no_reverse_engineering` | `reverse_engineering_allowed` equals `true` | `critical` | `reject` |
| `pb-no-source-code-access` | `source_code_access` | `source_code_access_allowed` equals `true` | `critical` | `lawyer_review` |
| `pb-affiliate-disclosure-equivalent-duties` | `affiliate_disclosure` | `equivalent_duties_required` equals `false` | `high` | `negotiate` |
| `pb-compelled-disclosure-notice-required` | `compelled_disclosure` | `prior_notice_required` equals `false` | `medium` | `negotiate` |
| `pb-return-destruction-clause-required` | `return_or_destruction` | `category` missing | `high` | `missing_clause_add` |
| `pb-automatic-renewal-notice-required` | `renewal` | `auto_renewal` equals `true` AND `notice_period_days` less_than `30` | `medium` | `negotiate` |
| `pb-unilateral-termination` | `termination` | `unilateral_termination_by` equals `counterparty_only` | `high` | `negotiate` |

Notes:

- The playbook is **company-specific**. Each customer maintains their own playbook JSON.
- Playbook changes are versioned. Every finding records which `playbookRuleIds` it matched, so an audit can replay findings against a prior playbook version.
- Rules can be tagged (`tags: ["nda", "saas", "ai"]`) to scope which rules apply per contract type.

---

## 8. Validation Layer

Validation runs *after* classification and slot extraction, before severity is finalized.

### 8.1 Validation checks

- **clause existence check** — confirm each detected clause maps to a `ClauseCategory`. Unmapped clauses are flagged for review.
- **required clause missing check** — compare detected categories against the playbook's required-clause list.
- **slot consistency check** — confirm extracted slots are internally consistent (e.g., `period_years: 10` and `perpetual: true` should not both be true).
- **cross-clause contradiction check** — see examples below.
- **playbook deviation check** — evaluate `PlaybookRule` conditions.
- **evidence anchoring check** — every finding must point to at least one substring of the original text.
- **legal citation verification hook** — if a clause cites a statute, route to the Korean Law API Verification Layer (Section 9). The hook is **never** filled by LLM hallucination.
- **redline diff check** — if a prior version exists, compute the clause-level diff (Section 13).
- **severity override log** — any user-driven severity override is logged with timestamp, user id, and justification.

### 8.2 Examples

- If the agreement has "no license transfer" but also grants broad use rights, flag a contradiction.
- If `confidentiality_period.period_years` is 10 and playbook max is 5 years, flag `high`.
- If `public_reference_or_marketing_rights` is allowed but a `prior_written_consent` clause is absent, flag `high`.
- If `personal_data` is mentioned but no `data_retention_and_deletion` clause exists, flag `high`.
- If `liability` includes "all damages including indirect damages" and no cap exists, flag `critical` (escalated from `high`).

### 8.3 LLM boundary

- **LLM cannot be the validator of record.**
- **LLM can only explain validation output after deterministic validation is complete.**
- Any LLM output that contradicts a deterministic finding is discarded; the deterministic finding wins.

---

## 9. Korean Law API Verification Layer

The engine may use a Korean law API (`mcp__korean-law`) only as a legal citation **verification layer**. It is a verification mechanism, not a legal-judgment mechanism. The API enriches a `ClauseFinding` with verified legal basis candidates and lawyer-handoff context; it never decides outcomes.

### 9.1 Stance

- The Korean Law API is a **verification layer**, not a source of legal conclusions.
- The output of the API enriches a `ClauseFinding` with verified legal basis candidates and lawyer-handoff context.
- The API is **never** used to decide whether a contract is safe to sign, enforceable, invalid, or compliant with the law.
- Severity remains driven exclusively by the deterministic playbook (§6, §7). API output never overrides severity.

### 9.2 Allowed uses

- verify that a cited statute / article exists
- retrieve statute / article title and text
- attach verified legal basis candidates to findings
- support lawyer handoff questions
- reduce unsupported LLM legal citation hallucination

### 9.3 Not allowed

- output final legal advice
- conclude enforceability
- conclude invalidity
- say a contract is safe to sign
- replace lawyer review

### 9.4 Initial Legal Basis Mapping

| clause category | candidate Korean law | rationale |
|---|---|---|
| `liquidated_damages` | Civil Act Article 398 (민법 제398조 — 배상액의 예정) | court's discretion to reduce excessive liquidated damages |
| `no_reverse_engineering`, `source_code_access`, `definition_of_confidential_information` (trade secret scope) | Unfair Competition Prevention and Trade Secret Protection Act (부정경쟁방지 및 영업비밀보호에 관한 법률) | trade secret misappropriation, reverse engineering boundaries |
| `personal_data`, `customer_data`, `data_retention_and_deletion`, `training_data_use` | Personal Information Protection Act (개인정보 보호법) | personal data processing duties, retention/deletion, consent |
| `license_grant`, `source_code_access`, `restrictions`, `sublicensing` | Copyright Act (저작권법) | software copyright and license boundaries |
| `service_level_or_accuracy_claims` (accessibility-related), `integration_scope` (accessibility-related) | Act on the Prohibition of Discrimination against Disabled Persons (장애인차별금지법) | accessibility-related contractual duties (e.g., mobile app voice command obligations) |

These are **candidates**, not conclusions. The verification layer:

1. checks each candidate against `mcp__korean-law` (article exists, title matches, text retrievable),
2. attaches the verified result to `ClauseFinding.legalBasis[]`,
3. emits a lawyer-handoff question regardless of verification outcome (applicability is a legal judgment, not a verification result).

### 9.5 LegalBasis Schema

```ts
interface LegalBasis {
  lawName: string;
  articleNumber: string;
  articleTitle: string;
  verificationStatus: "verified" | "unverified" | "not_found" | "api_error";
  apiSource: string;
  retrievedAt: string;
  reasonForMapping: string;
  lawyerQuestion: string;
}
```

Field semantics:

- `lawName` — full Korean law name (e.g., `"민법"`, `"개인정보 보호법"`, `"부정경쟁방지 및 영업비밀보호에 관한 법률"`).
- `articleNumber` — the article number cited (e.g., `"제398조"`).
- `articleTitle` — the article title retrieved from the API.
- `verificationStatus` — `verified` (API returned the article matching name/number), `unverified` (no API call attempted), `not_found` (API returned no match for the cited article), `api_error` (API call failed; the system records the error and the report visibly marks the basis as unverified for downstream consumers).
- `apiSource` — identifier of the verification source (e.g., `"mcp__korean-law"`).
- `retrievedAt` — ISO 8601 timestamp.
- `reasonForMapping` — short rationale for why this law was selected as a candidate for this clause.
- `lawyerQuestion` — concrete question for the lawyer, scoped to the cited law. Designed to be answerable by a qualified lawyer without re-reading the entire contract.

### 9.6 Example

For finding `f-002` (5x liquidated damages from §5.1), the `legalBasis` field would be populated as follows:

```json
{
  "legalBasis": [
    {
      "lawName": "민법",
      "articleNumber": "제398조",
      "articleTitle": "배상액의 예정",
      "verificationStatus": "verified",
      "apiSource": "mcp__korean-law",
      "retrievedAt": "2026-05-28T10:00:00+09:00",
      "reasonForMapping": "liquidated_damages clause with damages_multiplier 5x triggers consideration of Civil Act Article 398, under which Korean courts may reduce an excessive predetermined damages amount.",
      "lawyerQuestion": "Under Civil Act Article 398, is a 5x liquidated damages multiplier likely to be reduced by a Korean court as an excessive predetermined damages amount? What multiplier range typically survives judicial review for this contract type?"
    }
  ]
}
```

### 9.7 Workflow

1. After a `ClauseFinding` is generated by the validation layer (§8), the legal basis verifier looks up candidates from the mapping table (§9.4).
2. For each candidate, the verifier calls `mcp__korean-law` to retrieve the article (`get_law_text`, `search_law`, `verify_citations`, or equivalent).
3. The verifier writes the result into `ClauseFinding.legalBasis[]` regardless of verification status (so unverified candidates remain visible to the lawyer with a clear `verificationStatus` label).
4. Cached results expire daily; the cache prevents repeated API calls for the same article during a single review session.
5. API failures do not block the report. The finding is delivered with `verificationStatus: "api_error"` and the report visibly notes that the lawyer must verify manually.
6. Every verification call is logged with input (lawName, articleNumber), output (status, raw response hash), and timestamp for auditability.

### 9.8 Boundary

- The legal basis verifier **never overrides severity**. Severity is set by the deterministic playbook (§6, §7).
- The verifier **never generates conclusions** about applicability, enforceability, or compliance.
- LLM may help phrase the `lawyerQuestion` field *after* the deterministic legal basis (`lawName` / `articleNumber` / `articleTitle`) is attached. LLM is **not** allowed to invent a law name, article number, or article title.
- If a citation is added to a finding without successful API verification, the report must visibly mark it (`verificationStatus: "unverified"`, `"not_found"`, or `"api_error"`). Unverified citations must never appear in a report as if they were verified.

---

## 10. Confirm Step

The system must **never** output any of the following phrases:

- "safe to sign"
- "legally valid"
- "legally invalid"
- "you do not need a lawyer"

Instead, the system outputs one of:

- "critical/high findings require lawyer review"
- "business confirmation required"
- "acceptable under current playbook"
- "missing clause candidate"
- "negotiation recommended"

### 10.1 Confirm UI

```txt
Finding summary:
- critical: X
- high: Y
- medium: Z
- low/info: N

User decisions:
[ ] accept
[ ] negotiate
[ ] send to lawyer
[ ] mark as business risk accepted
[ ] ignore for this deal
```

### 10.2 Logging

Every user decision is logged with:

- `findingId`
- `decision` (accept | negotiate | send_to_lawyer | business_risk_accepted | ignored)
- `decisionTimestamp`
- `userId`
- `justification` (optional free text)
- `playbookVersion` at time of decision

The decision log is the auditable trail. A future export ("show me every clause I accepted with critical severity in the last 12 months") must be reproducible from this log.

---

## 11. Structured Report Format

Sample Markdown report sections:

```md
# Contract Pre-Review Report — {Counterparty Name} — {Contract Type} — {Date}

## 1. Executive Summary
- contract type:
- counterparty:
- deal context:
- total findings:
- critical: N | high: N | medium: N | low: N | info: N
- recommended next step:

## 2. Critical Findings
{ordered list of critical ClauseFinding summaries with evidence excerpts}

## 3. High Findings
{ordered list of high ClauseFinding summaries with evidence excerpts}

## 4. Missing Clauses
{list of required clauses not detected in the contract}

## 5. Playbook Deviations
{table mapping playbookRuleId → finding → severity → recommendedAction}

## 6. Lawyer Questions
{deduplicated, prioritized list of generated lawyer questions}

## 7. Negotiation Suggestions
{per-finding fallback wording proposals}

## 8. Accepted Clauses
{clauses that matched playbook acceptance criteria, listed for completeness}

## 9. Appendix: Extracted Clause Map
{clauseId → articleNumber → category → severity}

## 10. Disclaimer
This report is a contract pre-review and risk triage aid. It is not legal advice. Final legal review should be performed by a qualified lawyer where appropriate.
```

---

## 12. Lawyer Handoff Format

```md
# Lawyer Handoff — {Counterparty} — {Contract Type}

**Contract name:** {filename or title}
**Counterparty:** {legal entity name}
**Deal context:** {1-3 sentence summary of why this deal matters}
**Business urgency:** {target sign date / deadline / dependency}

## Top Issues (in priority order)
1. {category} — {severity} — {one-sentence summary}
2. ...

## Original Clause Excerpts
{verbatim text of each top issue, with article number}

## Company Playbook Position
{for each top issue: which playbook rule applies and the desired position}

## Questions for Lawyer
- {question 1}
- {question 2}
- ...

## Legal Basis Candidates (verification status)
{for each top issue: list of LegalBasis objects — lawName + articleNumber + articleTitle + verificationStatus + reasonForMapping. Unverified / not_found / api_error entries are flagged so the lawyer can confirm manually.}

## Proposed Negotiation Stance
{accept / negotiate with fallback / reject — with one-line rationale per issue}
```

The handoff is intentionally short. A lawyer should be able to scan the top issues in 5–10 minutes and ask the founder targeted questions before any detailed legal review begins. The "Legal Basis Candidates" block is informational only — applicability and enforceability remain the lawyer's call (see §9 Boundary).

---

## 13. Redline Review Flow

```
Original company template
  → counterparty redline
  → diff clauses
  → detect changed/removed protections
  → map changes to playbook rules
  → severity reassessment
  → accept/negotiate/reject recommendation
  → counterproposal generation
```

### 12.1 Examples

- **removed data/responsibility boundary** → `critical`
- **changed confidentiality period from 3 years to 10 years** → `high`
- **added residual knowledge clause** → `high` or `critical`
- **allowed training on submitted data** → `critical`
- **removed prior written consent for public reference** → `high`
- **changed jurisdiction to counterparty home court** → `medium` or `high`

### 12.2 Diff anchoring

Each redline finding stores:

- `templateClauseId` — original template clause reference
- `redlineClauseId` — counterparty redline clause reference
- `diffType` — added | removed | modified
- `protectionDelta` — strengthened | unchanged | weakened
- `severityDelta` — change in severity vs. template baseline

A redline finding inherits the same `ClauseFinding` shape, with `redline` metadata appended.

---

## 14. MVP Implementation Plan for Solo Founder

### Phase 0: Manual Benchmark

- collect 20 NDA/MSA/license samples
- manually label clause categories
- manually label severity
- create gold standard JSON

### Phase 1: Rule-First Parser

- parse markdown/text
- split into clauses
- keyword/regex category mapping
- slot extraction for 10 categories
- generate report

### Phase 2: Playbook Engine

- implement `PlaybookRule` schema
- implement severity assignment
- implement missing clause detection
- implement lawyer handoff report

### Phase 3: LLM-Assisted Explanation

- use LLM only after deterministic findings
- generate plain-language explanation
- generate negotiation wording
- generate lawyer questions

### Phase 4: Redline Comparison

- compare two versions
- detect protection removal
- re-run severity

### Phase 5: Customer Pilot

- 5 B2B SaaS / startup customers
- 3 contracts each
- measure:
  - time saved
  - lawyer questions generated
  - findings accepted/rejected
  - false positives
  - false negatives
  - willingness to pay

---

## 15. Test Cases

Each test case has: input clause → expected category → expected slots → expected severity → expected reasonCode.

### TC-01 — confidentiality period 3 years
- **input:** "The receiving party shall maintain confidentiality for a period of three (3) years from the date of disclosure."
- **expected category:** `confidentiality_period`
- **expected slots:** `period_years: 3`, `perpetual: false`
- **expected severity:** `info`
- **expected reasonCode:** `confidentiality_period_within_playbook_max`

### TC-02 — confidentiality period 10 years
- **input:** "The receiving party shall maintain confidentiality for a period of ten (10) years from the date of disclosure."
- **expected category:** `confidentiality_period`
- **expected slots:** `period_years: 10`, `perpetual: false`
- **expected severity:** `high`
- **expected reasonCode:** `confidentiality_period_exceeds_company_max`

### TC-03 — perpetual confidentiality
- **input:** "The Confidential Information shall remain confidential in perpetuity."
- **expected category:** `confidentiality_period`
- **expected slots:** `perpetual: true`, `trade_secret_carveout: false`
- **expected severity:** `high`
- **expected reasonCode:** `perpetual_confidentiality_without_trade_secret_carveout`

### TC-04 — residual knowledge allowed
- **input:** "Notwithstanding anything to the contrary, the receiving party may use any residual knowledge retained in unaided memory for any purpose."
- **expected category:** `residual_knowledge`
- **expected slots:** `residual_knowledge_allowed: true`
- **expected severity:** `high`
- **expected reasonCode:** `residual_knowledge_clause_present`

### TC-05 — public reference without consent
- **input:** "Provider may identify Customer as a customer in its marketing materials, including the use of Customer's name and logo, without further consent."
- **expected category:** `public_reference_or_marketing_rights`
- **expected slots:** `public_reference_allowed: true`, `prior_written_consent_required: false`
- **expected severity:** `high`
- **expected reasonCode:** `public_reference_without_prior_written_consent`

### TC-06 — training data allowed without consent
- **input:** "Provider may use any data submitted via the Service to train, improve, or develop its machine learning models without further notice or consent."
- **expected category:** `training_data_use`
- **expected slots:** `training_allowed: true`, `opt_in_required: false`
- **expected severity:** `critical`
- **expected reasonCode:** `training_data_used_without_opt_in`

### TC-07 — reverse engineering allowed
- **input:** "Customer may decompile, disassemble, or otherwise reverse engineer the Software for any purpose."
- **expected category:** `no_reverse_engineering`
- **expected slots:** `reverse_engineering_allowed: true`
- **expected severity:** `critical`
- **expected reasonCode:** `reverse_engineering_explicitly_allowed`

### TC-08 — no liability cap
- **input:** "Each party shall be liable for all damages arising out of this Agreement without limitation."
- **expected category:** `limitation_of_liability`
- **expected slots:** `liability_cap_exists: false`
- **expected severity:** `high`
- **expected reasonCode:** `no_liability_cap`

### TC-09 — indirect damages included
- **input:** "Each party shall be liable for all damages, including direct, indirect, incidental, consequential, and punitive damages."
- **expected category:** `limitation_of_liability`
- **expected slots:** `indirect_damages_included: true`
- **expected severity:** `high`
- **expected reasonCode:** `indirect_damages_included`

### TC-10 — jurisdiction outside Korea
- **input:** "This Agreement shall be governed by the laws of Delaware, USA, and any dispute shall be resolved exclusively in the courts of Wilmington, Delaware."
- **expected category:** `jurisdiction`
- **expected slots:** `governing_law: "Delaware, USA"`, `exclusive_venue: "Wilmington, Delaware"`
- **expected severity:** `medium` (per starter playbook; may be `high` per company-specific override)
- **expected reasonCode:** `jurisdiction_outside_preferred_venue`

### TC-11 — missing return/destruction clause
- **input:** *(no clause matching `return_or_destruction` detected anywhere in the contract)*
- **expected category:** `return_or_destruction`
- **expected slots:** *(none — missing-clause finding)*
- **expected severity:** `high`
- **expected reasonCode:** `required_clause_missing`
- **expected recommendedAction:** `missing_clause_add`

### TC-12 — affiliate disclosure without equivalent duties
- **input:** "The receiving party may share the Confidential Information with its affiliates."
- **expected category:** `affiliate_disclosure`
- **expected slots:** `affiliate_disclosure_allowed: true`, `equivalent_duties_required: false`
- **expected severity:** `high`
- **expected reasonCode:** `affiliate_disclosure_without_equivalent_duties`

### TC-13 — compelled disclosure without notice
- **input:** "If the receiving party is compelled by law to disclose the Confidential Information, it shall do so without obligation to notify the disclosing party."
- **expected category:** `compelled_disclosure`
- **expected slots:** `prior_notice_required: false`
- **expected severity:** `medium`
- **expected reasonCode:** `compelled_disclosure_without_prior_notice`

### TC-14 — source code access allowed
- **input:** "Provider shall make the Software's source code available to Customer upon request."
- **expected category:** `source_code_access`
- **expected slots:** `source_code_access_allowed: true`
- **expected severity:** `critical`
- **expected reasonCode:** `source_code_access_explicitly_allowed`

### TC-15 — liquidated damages 5x actual damages
- **input:** "Upon any breach of confidentiality, the breaching party shall pay liquidated damages equal to five (5) times the actual damages incurred."
- **expected category:** `liquidated_damages`
- **expected slots:** `damages_multiplier: 5`
- **expected severity:** `critical`
- **expected reasonCode:** `liquidated_damages_multiplier_exceeds_threshold`

---

## 16. Non-Goals

Explicit non-goals for v0.1:

- **not legal advice**
- **not a lawyer replacement**
- **not a general legal chatbot**
- **not all contracts** (only NDA, MSA, software license, SDK license)
- **not all industries** (focused on B2B SaaS / AI / software)
- **not final sign-off decision**
- **not automatic court / premium legal research**
- **not a guarantee of enforceability**
- **not hallucination-free in every possible sense**; rather, it *minimizes* unsupported generation by using structured extraction and evidence anchoring
- **Korean Law API (`mcp__korean-law`) is used only as a citation verification layer (Section 9), not as a source of legal advice, enforceability conclusions, or compliance judgments**

Out of scope for v0.1 (do not support yet):

- employment agreements
- investment agreements
- clinical / medical research NDA
- manufacturing NDA
- game publishing agreements
- real estate agreements
- litigation documents

---

## 17. Suggested Code Structure

TypeScript, modular, no external API calls in v0.1.

```txt
src/contract-review/
  taxonomy.ts
  types.ts
  playbook.ts
  clauseSplitter.ts
  slotExtractors.ts
  validators.ts
  severity.ts
  legalBasisVerifier.ts
  reportRenderer.ts
  redlineDiff.ts
  __tests__/
    ndaCore.test.ts
    playbookRules.test.ts
    legalBasisVerifier.test.ts
    redlineDiff.test.ts
```

File responsibilities:

- `taxonomy.ts` — `ClauseCategory` enum, category metadata, lexicon entries per category.
- `types.ts` — `Severity`, `RecommendedAction`, `ClauseSpan`, `ExtractedSlot`, `ClauseFinding`, `PlaybookRule`, `LegalBasis`.
- `playbook.ts` — starter playbook, playbook loader, rule evaluator.
- `clauseSplitter.ts` — text normalization + clause segmentation by article numbering.
- `slotExtractors.ts` — per-category slot extraction functions (rule/regex first).
- `validators.ts` — clause existence, missing clause, contradiction, evidence anchoring, severity escalation.
- `severity.ts` — deterministic severity assignment given slots + playbook rules.
- `legalBasisVerifier.ts` — Korean Law API Verification Layer (Section 9). Maps clause categories to candidate Korean laws, calls `mcp__korean-law` to verify articles, and attaches `LegalBasis[]` to each `ClauseFinding`. Never overrides severity; never generates legal conclusions.
- `reportRenderer.ts` — Markdown report, JSON report, lawyer handoff renderer.
- `redlineDiff.ts` — template-vs-redline comparison, protection delta detection.
- `__tests__/` — gold standard fixtures (Phase 0 manual benchmark) wired into Jest. `legalBasisVerifier.test.ts` mocks `mcp__korean-law` responses to cover `verified` / `unverified` / `not_found` / `api_error` paths.

Constraints:

- **Do not call external APIs in v0.1**, with one exception: `mcp__korean-law` may be called by `legalBasisVerifier.ts` strictly as a verification source (Section 9). All other external API integrations are deferred.
- LLM integration (Phase 3) is a separate module behind a feature flag, never on the critical path of severity assignment or legal basis verification.

---

## 18. Final Output

### 18.1 File created

The full content of this document is written to `docs/contract_pre_review_engine_v0_1.md`.

### 18.2 Optional TypeScript skeleton (not implemented yet)

Recommended creation order for the skeleton (one file at a time, not implemented in this v0.1 doc):

1. `src/contract-review/types.ts`
2. `src/contract-review/taxonomy.ts`
3. `src/contract-review/playbook.ts`
4. `src/contract-review/clauseSplitter.ts`
5. `src/contract-review/slotExtractors.ts`
6. `src/contract-review/severity.ts`
7. `src/contract-review/validators.ts`
8. `src/contract-review/legalBasisVerifier.ts`
9. `src/contract-review/reportRenderer.ts`
10. `src/contract-review/redlineDiff.ts`
11. `src/contract-review/__tests__/ndaCore.test.ts`

### 18.3 Architecture summary

The backbone is: deterministic input → lexicon-driven classification → slot extraction → rule validation → human confirm → structured output. Input is text (PDF/DOCX/TXT); the atomic unit is a contract clause; the lexicon domain is legal clause categories; validation runs against a company playbook plus legal rules; output is a risk report plus lawyer handoff JSON.

The non-negotiables:

- LLM is never the source of truth.
- Lexicon / taxonomy changes are spec-controlled.
- Every output decision is anchored to evidence in the original input.
- Human confirm is mandatory before any externally visible action.
- Every decision is logged and replayable against a versioned ruleset.

### 18.4 Recommended next file to implement first

**`src/contract-review/types.ts`** — defining the type system first locks the contract shape that every other module depends on (`ClauseFinding`, `PlaybookRule`, `Severity`, `RecommendedAction`, `ClauseCategory`, `LegalBasis`). All downstream modules (taxonomy, playbook, validators, legalBasisVerifier, renderer) consume these types, so writing them first prevents schema churn during early Phase 1 implementation.

---

## Disclaimer

This document describes a contract pre-review and risk triage system. It is not legal advice and does not constitute the practice of law. Any contract that the system processes should be reviewed by a qualified lawyer where appropriate.

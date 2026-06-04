<a id="english"></a>
# Contract Pre-Review Engine — Architecture & Design Docs

**English** | [한국어](#korean)

> Design documentation for a *pre-review* engine that **deterministically screens
> risky clauses** in NDA / MSA / license contracts and hands a structured report to
> a human lawyer.
>
> ⚠️ This repository is **design / research documentation**. It is not a product or a
> service, and it is **not legal advice.**

## What it is

A v0.1 architecture specification for a *deterministic (rule-first)* pipeline that
breaks a contract down clause by clause, scores each clause's risk (severity) against
a company playbook and ruleset, diagnoses missing clauses, and compiles **the questions
a lawyer needs to answer** into a hand-off report.

Key design decision: **the LLM is kept out of the decision path.** Clause
classification, severity scoring, and missing-clause diagnosis are all rule-based and
anchored to evidence (the source text). The LLM — if used at all — plays only a
supporting role: explaining, in natural language, the decisions the rules have already made.

## What it is not

- It is **not** legal advice.
- It does **not** replace a lawyer.
- It is **not** a "safe to sign" approval tool.
- It is **not** a guarantee of enforceability.

## Design principles

- **Decision, not generation** — rules are the source of truth; the LLM does not judge.
- **Evidence-anchored** — every finding links back to the source text of the clause.
- **Human makes the final call** — the output is a lawyer hand-off; no automated decisions, no automated signing.
- **Auditable** — every decision is logged and reproducible against a versioned ruleset.

## Documents

- [`docs/contract-pre-review-engine-v0.1.md`](docs/contract-pre-review-engine-v0.1.md)
  — the complete v0.1 architecture specification
  (Product Definition → Architecture → Clause Taxonomy → Severity Framework →
  Company Playbook → Validation Layer → Korean Law Verification Layer →
  Report / Lawyer Handoff Format → MVP Plan → Test Cases → Non-Goals)

## ⚠️ Disclaimer

This is a personal design / research artifact. It is **not legal advice**, and it does
not replace review by a qualified lawyer or perform any legal service. Always have
actual contract review done by a lawyer.

## Author

김수환 (Suhwan Kim) · shawnkim1022@gmail.com

---

<a id="korean"></a>
# Contract Pre-Review Engine — 아키텍처 & 설계 문서

[English](#english) | **한국어**

> NDA / MSA / license 계약서의 **위험 조항을 결정론적으로 사전 선별**해
> 변호사(사람)에게 구조화된 보고서로 넘기는 *사전 검토(pre-review)* 엔진의 설계 문서입니다.
>
> ⚠️ 본 저장소는 **설계·연구 문서**입니다. 제품도 서비스도 아니며, **법률 자문이 아닙니다.**

## 무엇인가

계약서를 조항 단위로 분해해, 회사 playbook과 룰에 따라 위험도(severity)를 매기고,
누락 조항을 진단하고, **변호사가 답해야 할 질문**을 정리해 handoff 보고서로 만드는
*결정론(rule-first)* 파이프라인의 v0.1 아키텍처 명세입니다.

핵심 설계 결정: **LLM을 판단 경로에서 배제**합니다. 조항 분류 · severity · 누락 진단은
전부 룰 기반이고 근거(원문)에 고정됩니다. LLM은 (쓰더라도) 룰이 내린 결정을
자연어로 설명하는 보조 역할만 합니다.

## 무엇이 아닌가

- 법률 자문이 **아닙니다**.
- 변호사를 **대체하지 않습니다**.
- "서명해도 안전" 승인 도구가 **아닙니다**.
- 집행 가능성(enforceability) 보증이 **아닙니다**.

## 설계 원칙

- **생성이 아니라 결정** — 룰이 source of truth, LLM은 판단하지 않음
- **근거 고정(evidence-anchored)** — 모든 finding은 조항 원문에 링크
- **사람이 최종 판단** — 출력은 변호사 hand-off, 자동 결정 · 자동 서명 없음
- **감사 가능(auditable)** — 모든 결정은 버전된 ruleset에 대해 로그·재현 가능

## 문서

- [`docs/contract-pre-review-engine-v0.1.md`](docs/contract-pre-review-engine-v0.1.md)
  — v0.1 아키텍처 전체 명세
  (Product Definition → Architecture → Clause Taxonomy → Severity Framework →
  Company Playbook → Validation Layer → Korean Law Verification Layer →
  Report / Lawyer Handoff Format → MVP Plan → Test Cases → Non-Goals)

## ⚠️ 면책 (Disclaimer)

개인 설계·연구 산출물입니다. **법률 자문이 아니며**, 자격 있는 변호사의 검토를
대체하거나 법률 사무를 수행하지 않습니다. 실제 계약 검토는 반드시 변호사에게 받으십시오.

## 작성자

김수환 (Suhwan Kim) · shawnkim1022@gmail.com

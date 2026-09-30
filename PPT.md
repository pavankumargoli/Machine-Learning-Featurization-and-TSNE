Slide 1: Title

AI-Powered Intent Discovery Service for CX Requirements Engineering
GenAI Governance Panel Submission: New Proof of Concept
Policy reference: CP6.55, Generative Artificial Intelligence Policy
Submitted by: Ankur Bhakta and Pavan Kumar Goli, CX Engineering (TFS / TMCC)
Date: [submission date]

Slide 2: Executive Summary and Decision Requested

Headline: We request approval for a low-risk, development-only PoC that uses GenAI to turn ambiguous requirements into validated, human-approved specifications, not code.

Problem: Requirements reaching CX Engineering are ambiguous and inconsistent, which causes rework, integration gaps, and weak test coverage. AI coding tools make this worse.
Solution: An AI-augmented BSA interviewer that asks one closed question at a time. Its output feeds a schema-validated modeling pipeline: Event Storming → DDD domain model → Intent Registry.
Controls: Deterministic, non-AI validation gates; a separate Red Team review pass; mandatory approval by an IR Author and a domain SME.
Boundary: Access only through the approved AWS Bedrock channel. No Personal Data, no production data, no production integration, no code generation.
Preliminary classification: Low Risk, subject to Panel confirmation.
Decision requested: Approval to build and run the PoC in the development environment, scoped as described in this deck.

Speaker note: Lead with the ask and the boundary. The rest of the deck supports these six lines.

Slide 3: Business Problem

Headline: Prose requirements cannot be validated, so ambiguity reaches engineering and becomes rework.

Observed repeatedly on the Titles project:

Requirements are written in vague or technology-centric language rather than as business outcomes.
Terminology is inconsistent, so there is no shared Ubiquitous Language.
Domain ownership, Bounded Context boundaries, and rule-enforcing Aggregates are unclear.
Exceptions to rules hide behind "usually," "normally," and "typically."
Acceptance criteria are weak, which produces unverifiable test coverage.
Solutions are built around fragments of a feature instead of the full business outcome.
The existing BSA template has not fixed this, because the format is not the root cause.

Consequences: conflicting interpretations, integration gaps, rework, engineer interpretation burden, and waste (muda).

Callout: AI code generation amplifies the problem. It rapidly produces polished implementations of ambiguous intent.

Speaker note: If you have any Titles metrics (rework cycles, defects traced to requirements, clarification loops), put one number here. It strengthens the slide considerably.

Slide 4: Why GenAI, and Why Not the Alternatives

Headline: Establishing business meaning requires contextual reasoning. Deterministic tools validate structure but cannot interrogate intent.

Alternative	Limitation
Static template or checklist	Enforces format, but cannot adapt questions or reconcile conflicting terminology
Rule-based requirements linter	Validates fields and structure, but cannot judge whether the business meaning is correct
Manual peer review (current process)	Still essential for approval, but has not surfaced gaps early and does not scale
Keyword or static analysis	Finds terms, but cannot run context-aware interrogation or synthesize a domain model

Design principle: GenAI for reasoning and drafting, deterministic code for validation, humans for approval.

Speaker note: This slide answers the "why GenAI at all?" question every council asks. The design principle line is what you want them to remember.

Slide 5: Proposed Solution Overview

Headline: An AI interviewer and a validated modeling pipeline that produce specifications. It does not produce source code.

Three stages:

Interrogate: The AI asks one closed question at a time until each requirement is decidable. It proposes an answer along with its consequences, and the user confirms, corrects, defers, or flags it.
Model: After human confirmation, a staged pipeline drafts an Event Storming model (ESML), a DDD domain model with Context Map (DMML), and a machine-checkable Intent Registry. Each stage must pass deterministic validation before the next runs.
Challenge and approve: A separate Red Team pass challenges the model. The IR Author and a domain SME attest to business correctness. Only then are diagrams and the handoff pack generated, deterministically.

What it deliberately excludes: code generators, implementation templates, and scaffolding.

Slide 6: User Workflow

Headline: A structured 12-rung interrogation keeps the human in control of every answer.

The 12-rung ladder: Ownership → Scope and boundaries → Process and timing → Business rules → Error scenarios → Recorded decisions → Decision tables → Recovery and fallback → Reporting and visibility → Regulatory → Risk tier → Non-goals.

User actions on each AI-proposed answer:

Answer: confirm or refine the proposal.
Defer: park a hard question and return to it later.
Contest: reject the proposal and supply the correct answer, which creates a correction record.
Flag: give question feedback (for example "wrong premise" or "too technical"), captured as a cross-session learning.

Flow: Session created → interrogation → spec sections auto-drafted → user confirms the draft → modeling pipeline → Red Team → IR Author and SME approval → artifacts written to specs/<feature-slug>/.

Visual suggestion: Show the ladder as a horizontal stepper, with the four user actions as a sidebar.

Slide 7: Solution Architecture

Headline: All model inference stays inside the approved AWS Bedrock boundary. All data stays in the development environment.

Components:

Frontend (React + TypeScript, Vite): interrogation chat, YAML spec editor with validation, diagram viewer, decision-table editor with completeness check, gap tracker, and export to YAML, Markdown, and PDF.
Backend (Python 3.12 + FastAPI): the interrogation agent and the Intent Registry manager.
Orchestration (LangGraph): the interrogation-ladder state machine and the Red Team agent.
Intent Validation Engine (deterministic, non-AI): decision-table completeness, verification-matrix generation, and gap tracking.
LLM provider (AWS Bedrock Runtime): Claude Sonnet for interrogation and drafting (high-volume turns); Claude Opus for Red Team and complex invariant reasoning (lower-volume, higher-stakes calls).
Storage (development only): SQLite for sessions, the file system for Intent Registry bundles, and ChromaDB as a vector store over the internal OKF knowledge base.

Visual suggestion: Draw a single dashed trust-boundary box labeled "Development environment, approved AWS boundary" containing every component. Nothing should cross it except the Bedrock call, which stays inside the approved boundary. This picture does more for a council than any bullet list.

Slide 8: Modeling Pipeline and Deterministic Validation Gates

Headline: Every AI-drafted model must pass rule-based validation before it can advance.

Stage	Output	Generated by
Event Storming (ESML)	Actors, commands, events, aggregates, policies, read models, external systems, flagged hotspots	LLM
Domain Model (DMML)	Ubiquitous Language, subdomains, Bounded Contexts, Context Map, Aggregates with invariants	LLM
Intent Registry (IR)	State transitions, invariants with enforcement points, decision tables, timeouts, required test evidence, bounded gaps	LLM
Diagrams and handoff pack	Generated from approved models only	Deterministic code

Non-AI gates:

Schema conformance: malformed or incomplete output is rejected.
Cross-reference integrity: every reference must resolve to a defined element.
Decision-table completeness: all input combinations are computed, and coverage and non-contradiction are proven.
Enforcement points: every rule names where it is enforced.
Test evidence: every rule and action names the evidence that proves it.
Explicit gaps: open questions must be flagged as blocking. They cannot be left silently ambiguous.

Bounded self-repair: on failure, errors are fed back to the AI for a limited number of logged repair attempts. If validation still fails, the errors go to a human, and force-finalization is not possible.

Honest limit (state this on the slide): The gates prove structural consistency and coverage. They do not prove business correctness. That is the human approvers' role.

Slide 9: Human Oversight Controls

Headline: AI proposes, deterministic code verifies, humans approve. No AI output is finalized autonomously.

The interrogation agent is read-only with respect to finalized specs. It cannot approve, finalize, or publish.
All intermediate ESML, DMML, and IR artifacts remain labeled as AI proposals until approved.
Dual approval is required: the IR Author approves the specification, and the domain SME independently attests to business correctness.
Red Team findings are flags, never automatic edits. Blocking findings must be resolved or recorded as approved gaps.
Users can always contest and correct AI answers. Corrections are retained to reduce repeat errors.
An offline evaluation harness scores models against hand-reviewed references, measuring invented, omitted, and misclassified elements per prompt version. Owners review the results before any prompt or model change is released.
Changes to the ladder, prompts, schemas, validation rules, and evaluation criteria are versioned and reviewed by the Use Case Owners and the Domain Architect.

Visual suggestion: Three swim lanes (AI, Deterministic, Human) with the approval gates marked.

Slide 10: Data Handling and Classification

Headline: Inputs describe what software should do. They contain no data about customers.

Data	Description	Classification
Feature requirement text	Free-text business behavior, typed or dictated	Internal, non-personal
Business rules and decision logic	Routing rules, states, invariants	Internal, non-personal
Glossary and architecture references	OKF knowledge base content	Internal engineering documentation
Generated artifacts	ESML, DMML, IR, diagrams, findings, handoff pack	Internal work product

Prohibited: Personal Data, customer records, production data, account numbers, SSNs, and credit or financial data.

Safeguards:

Users receive data-entry guidance.
OKF reference content is reviewed, and redacted or excluded as needed, before indexing.
LLM calls go only through TMCC-approved AWS Bedrock. No public or consumer GenAI endpoints are used.
Sessions, embeddings, and artifacts remain in the development environment.

Slide 11: Policy CP6.55 Compliance Matrix

Headline: The design addresses each CP6.55 requirement by construction.

CP6.55 requirement	How the design complies
Bias and discrimination	Produces technical specifications only. Makes no decisions affecting individuals (credit, HR, or customer).
Privacy and data protection	Personal, customer, and production data are prohibited. Users get entry guidance. References are reviewed and redacted.
Confidentiality	Calls go only through approved AWS Bedrock. All data stays in the development environment.
Intellectual property	Output derives from internal requirements only. No external IP is consumed.
Misinformation	Outputs are labeled as proposals, gated deterministically, challenged by Red Team, and dual-approved.
Independent validation	Non-AI gates, independent human attestation, and an offline regression harness for prompt and model changes.
Disclosure	AI involvement is visible throughout the UI: chat turns, Red Team panel, learnings page.
Procurement	Uses the existing approved Bedrock channel. No new vendor contract is required.

Slide 12: Risk Assessment and Mitigations

Headline: Preliminary classification: Low Risk. It rests on specific, verifiable conditions.

Classification holds only if all of the following remain true: internal and development-only; no Personal, customer, or production data; no autonomous decisions; no production integration; no source-code generation; deterministic validation, retained review records, and dual approval implemented as described.

Risk	Mitigation
Plausible but incorrect models (hallucination)	Cross-reference gates reject invented elements. Red Team challenge. SME attestation. Evaluation harness.
Sensitive data entered by users	Prohibition, user guidance, reference redaction, dev-only storage [plus technical PII screening, see gaps below]
Over-reliance on AI proposals	Closed questions with stated consequences. Contest and flag actions. Mandatory dual human approval.
Quality regression after prompt or model changes	Versioned changes. Offline evaluation against reference models. Owner review before release.
Scope creep toward code generation or production	No code generator in the design. Any material change is re-submitted to the Panel.
Incorrect "learnings" persisting across sessions	Corrections and learnings are retained, visible, and reviewable [consider owner review before reuse]

Slide 13: Scope, Users, and Expected Impact

Headline: An opt-in PoC for about 30–45 internal users, with tightly bounded scope.

Users:

CX Engineers and Tech Leads (~15–25): less time spent interpreting requirements.
BSAs as requirement authors (~5–10): gaps surfaced before handoff.
IR Authors, drawn from the BSA group: review and approve specifications.
Domain Architects and SMEs (~2–4): attest to business correctness.
Engineering Managers (~3–5): faster, higher-confidence definition of done.

Expected benefits: fewer rework cycles; testable acceptance criteria; a shared Ubiquitous Language; explicit context ownership; decision-table coverage that is impractical to verify by hand; durable, versioned specification artifacts.

Out of scope: production deployment, CI/CD integration, code generation, customer data, SOX-controlled processes, and autonomous approval or publication.

Acknowledged trade-off: users must adopt a more structured, question-driven process and review AI-proposed models.

Slide 14: Accountability, Success Criteria, and Next Steps

Headline: Clear ownership, measurable exit criteria, and a defined path back to the Panel.

Accountability:

Use Case Owners (Ankur Bhakta, Pavan Kumar Goli): delivery, operation, data-handling guidance, model configuration, evaluation reviews, audit records, onboarding, and compliance reporting.
IR Author: specification review and approval.
Domain SME: independent attestation of business correctness.
Coordination: Domain Architect, BISO, DRL, and the GenAI Governance Panel on any material change to scope, data, provider, or deployment.

PoC success criteria [fill in targets]:

Number of features taken through the full pipeline, for example [N] features on the Titles project.
Gaps and ambiguities surfaced before handoff, compared with the current process.
Evaluation harness scores (invented, omitted, and misclassified elements) at or better than [threshold].
SME and IR Author acceptance rate of generated models.
Engineer-reported clarification requests during implementation.

Timeline: [PoC duration, milestones, date of report-back to the Panel]

Decision requested: approve the PoC as scoped.

Gaps to close before submission

These are the places where the document is internally inconsistent or silent, and where a council is most likely to push back.

Model provider inconsistency. Section 2 says "Anthropic Claude and OpenAI models via Bedrock," but the architecture names only Claude Sonnet and Opus. Pick one statement. If OpenAI models are genuinely in scope, name which ones and what they would be used for.
No authentication conflicts with your own controls. Section 10 says "no user authentication planned," yet the design depends on dual approval, attestation, retained review records, and audit trails. An approval that cannot be tied to a named person is not an attestation. The BISO will almost certainly catch this. I'd recommend at least corporate SSO, or captured approver identity, even for the PoC.
PII prevention is instruction-only. The only control against users pasting customer data is user guidance. Adding a lightweight technical screen, such as Amazon Bedrock Guardrails PII detection or a pattern check on input, turns a policy promise into a control. That strengthens both the Privacy row and the Low Risk claim.
Undeclared data flows. The document says input can be "dictated," which implies a speech-to-text service that isn't named. ChromaDB also needs an embedding model that isn't named. Both are data flows the council will want mapped to the approved boundary.
Retention and cross-session learnings. There is no retention or deletion period for sessions, embeddings, or learnings. Also, stored "learnings" from one user's corrections will influence other sessions. Say who reviews them, so that a wrong correction doesn't propagate.
Missing PoC fundamentals. The document has no duration, success criteria, cost estimate, or report-back point. Slide 14 has placeholders for these. Councils typically approve bounded experiments with exit criteria much more readily than open-ended builds.
Terminology. "Red Team" usually means security testing in governance settings. Add a one-line clarification, such as "adversarial LLM review of the model," to avoid confusion.
Downstream use of the handoff pack. Because the pack is designed for consumption by a coding assistant, state that this downstream use is governed separately under existing approvals. That keeps the council from treating the PoC as indirect code generation.

If you'd like, I can build this as an actual slide deck in a clean enterprise style. Just tell me whether you need it as a PowerPoint file, for example to drop into a corporate template.

Here are the updated slides. Only the slides that change are shown; slides 1, 3, 4, 6, and 14 stay as they were. Changed text is summarized under each slide so you can find it quickly in your draft.

For the model provider, I resolved the inconsistency by making the deck Claude-only via Bedrock. That's what the architecture section already describes. If you do intend to use OpenAI models, tell me which ones and for which tasks, and I'll rewrite those lines to match.

Slide 2: Executive Summary and Decision Requested

Headline: We request approval for a low-risk, development-only PoC that uses GenAI to turn ambiguous requirements into validated, human-approved specifications, not code.

Problem: Requirements reaching CX Engineering are ambiguous and inconsistent, which causes rework, integration gaps, and weak test coverage. AI coding tools make this worse.
Solution: An AI-augmented BSA interviewer that asks one closed question at a time. Its output feeds a schema-validated modeling pipeline: Event Storming → DDD domain model → Intent Registry.
Controls:
Deterministic, non-AI validation gates.
A Red Team adversarial review pass.
Automated PII screening on all input.
Authenticated users.
Mandatory, identity-recorded approval by an IR Author and a domain SME.
Boundary:
Anthropic Claude models only, accessed only through the approved AWS Bedrock channel.
No Personal Data, no production data, no production integration, no code generation.
Preliminary classification: Low Risk, subject to Panel confirmation.
Decision requested: Approval to build and run the PoC in the development environment, scoped as described in this deck.

What changed: The controls list now includes PII screening, authentication, and identity-recorded approval. The provider is stated as Claude-only.

Slide 5: Proposed Solution Overview

Headline: An AI interviewer and a validated modeling pipeline that produce specifications. It does not produce source code.

Interrogate: The AI asks one closed question at a time until each requirement is decidable. It proposes an answer along with its consequences, and the user confirms, corrects, defers, or flags it.
Model: After human confirmation, a staged pipeline drafts an Event Storming model (ESML), a DDD domain model with Context Map (DMML), and a machine-checkable Intent Registry. Each stage must pass deterministic validation before the next runs.
Challenge and approve:
A Red Team pass challenges the model. Here, "Red Team" means an adversarial LLM review of the drafted model, run as a separate pass that looks for gaps, contradictions, and unstated assumptions. It is not security or penetration testing.
The IR Author and a domain SME then attest to business correctness.
Only after that are diagrams and the handoff pack generated, deterministically.

What it deliberately excludes: code generators, implementation templates, and scaffolding.

Downstream use: The handoff pack is a specification, not code. If an engineer later uses it with a coding assistant, that happens outside this PoC and is governed by that tool's existing approval and the team's standard SDLC controls. This PoC does not invoke, integrate with, or trigger any code-generation tool.

What changed: Added the Red Team definition and the downstream-use statement.

Slide 7: Solution Architecture

Headline: Every user is authenticated, every input is screened, and all inference stays inside the approved AWS Bedrock boundary.

Authentication:
Users sign in through corporate SSO ([Entra ID / corporate IdP], using OIDC).
Access is limited to an allow-listed CX Engineering group.
Three roles control permissions: Contributor (BSA or engineer), IR Author, and Domain SME.
Only the IR Author and Domain SME roles can approve.
Input screening (PII guardrail): Every user input passes a PII screen at the API layer, before it is stored or sent to the model (details on Slide 10).
Frontend (React + TypeScript, Vite): chat, YAML spec editor, diagram viewer, decision-table editor, gap tracker, and export.
Backend (Python 3.12 + FastAPI): the interrogation agent and the Intent Registry manager.
Orchestration (LangGraph): the interrogation ladder and the Red Team adversarial review agent.
Intent Validation Engine (deterministic, non-AI): completeness checks, verification matrix, and gap tracking.
LLM provider (AWS Bedrock Runtime, Anthropic Claude only):
Claude Sonnet for interrogation and drafting.
Claude Opus for the Red Team review and complex invariant reasoning.
No other model providers.
Storage (development only): SQLite, the file system, and ChromaDB over the redacted OKF knowledge base.

Visual suggestion: Keep the dashed trust boundary. Draw the request path as User → SSO → PII screen → API, so the council sees both controls sitting in front of everything else.

What changed: Added the authentication and PII-screen components, and made the provider statement explicit.

Slide 8: Modeling Pipeline and Deterministic Validation Gates

This slide is unchanged, with one small edit. Rename the step to "Red Team (adversarial model review)" wherever it appears on the slide, so the terminology matches Slide 5.

Slide 9: Human Oversight Controls

Headline: AI proposes, deterministic code verifies, humans approve. No AI output is finalized autonomously.

The interrogation agent is read-only with respect to finalized specs. It cannot approve, finalize, or publish.
All intermediate ESML, DMML, and IR artifacts remain labeled as AI proposals until approved.
Dual, attributable approval:
The IR Author approves the specification, and the domain SME independently attests to business correctness.
Each approval is tied to the approver's SSO identity and timestamp, and retained as an audit record.
The same person cannot hold both approvals on one specification.
Red Team adversarial review findings are flags, never automatic edits. Blocking findings must be resolved or recorded as approved gaps.
Users can always contest and correct AI answers. Corrections are retained, with the author's identity.
An offline evaluation harness scores models against reference examples before any prompt or model change is released.
Changes to the ladder, prompts, schemas, validation rules, and evaluation criteria are versioned and reviewed by the Use Case Owners and the Domain Architect.

What changed: Approvals are now identity-bound, and there is a separation-of-duties rule (one person cannot approve twice).

Slide 10: Data Handling and Classification

Headline: Personal data is prohibited by policy and blocked by a technical control.

The data table is unchanged.

Prohibited: Personal Data, customer records, production data, account numbers, SSNs, and credit or financial data.

Technical control, an automated PII screen:

Where it runs: at the API layer, on every user input, before the input is stored in SQLite or sent to Bedrock.
How it works: Amazon Bedrock Guardrails sensitive-information filters, set to block on input and output.
What it detects:
Standard PII types: names, emails, phone numbers, addresses, SSNs, and card and bank account numbers.
Custom patterns for TFS-specific identifiers: VINs and TFS account-number formats. These matter because the Titles domain naturally invites real vehicle and account examples.
When it triggers:
The input is rejected and the user is asked to rephrase with synthetic values.
The event is logged without storing the flagged content.
Before indexing: the same screen runs on OKF reference content before it is indexed into ChromaDB. This supplements the manual review.

Other safeguards:

Users receive data-entry guidance.
Bedrock is the only endpoint used. No public GenAI services are called.
All data stays in the development environment.

What changed: The guidance-only approach becomes an enforced control, and a custom VIN and account-number pattern is added.

Slide 11: Policy CP6.55 Compliance Matrix

Only the updated rows are shown below; the rest of the matrix is unchanged.

CP6.55 requirement	How the design complies
Privacy and data protection	Personal, customer, and production data are prohibited and technically blocked by an automated PII screen (Bedrock Guardrails plus TFS-specific patterns) before storage or model calls. Reference content is screened and reviewed before indexing.
Confidentiality	SSO-authenticated, allow-listed users only. Calls go only through approved AWS Bedrock. All data stays in the development environment.
Misinformation	Outputs are labeled as proposals, gated deterministically, challenged by an adversarial LLM review, and approved by two identified reviewers.
Independent validation	Non-AI gates, identity-recorded human attestation with separation of duties, and an offline regression harness.
Procurement	Anthropic Claude models via the existing approved Bedrock channel. No new vendor or model provider.

Slide 12: Risk Assessment and Mitigations

The classification conditions now also require that SSO authentication and the PII screen be in place.

Updated risk table:

Risk	Mitigation
Sensitive data entered by users	Automated PII screen blocks input before storage or model calls. Policy prohibition. User guidance. Pre-index screening of references.
Unattributable approvals or access	SSO authentication, allow-listed group, role-based approval rights, identity-stamped audit records.
Misreading the PoC as code generation	No code generator in the design. Downstream coding-assistant use is governed separately under existing approvals.

The remaining rows (hallucination, over-reliance, regression, incorrect learnings) are unchanged.

Slide 13: Scope, Users, and Expected Impact

In the out-of-scope list, replace "no user authentication" with the line below, and add a line about downstream implementation:

No enterprise RBAC integration beyond SSO sign-in and three PoC roles.
Downstream implementation of the handoff pack is out of scope for this PoC and is governed separately.

Matching edits to the use case document

The council will read the document alongside the deck, so make the same fixes there:

Section 2: change the provider to "Anthropic Claude models accessed through AWS Bedrock Runtime."
Sections 5.2 and 6: add the SSO and PII-screen components.
Section 5.5: add the downstream-use statement.
Section 8: update the Privacy, Confidentiality, and Procurement rows, and add authentication and the PII screen to the Low Risk conditions.
Section 10: replace "No user authentication planned."
First mention of Red Team: define it as an adversarial LLM review of the model.

create images for the architecture to add in presentation

I made three 16:9 diagrams, one each for Slides 7, 8, and 9. They share the same color key (blue = AI, green = deterministic, amber = human, dark navy = security control), so the three read as one set.

Solution architecture (Slide 7):
Everything runs inside the development environment / approved AWS boundary.
A user's request goes through SSO sign-in, then the PII screen, then the backend.
Bedrock is shown as calling Claude models only.
Two boxes sit outside the boundary: "Not connected" (production systems, customer data, public endpoints, code generators, CI/CD) and "Handoff pack" (specifications only, downstream use governed separately).
Modeling pipeline and gates (Slide 8):
ESML → DMML → Intent Registry, with a validation gate after each stage.
Each gate has a "bounded repair" loop back to its stage.
Then Red Team review, human approval, and deterministic generation of the outputs.
Bottom panels list the six gate checks and what happens when a gate fails.
Human oversight (Slide 9):
Swim lanes for Human, AI, and Deterministic across six steps.
A footer states "AI never approves, finalizes, or publishes," alongside the audit-trail statement.
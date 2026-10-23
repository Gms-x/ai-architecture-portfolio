# Marketing Automation with AI & Agents | Governance Pack
**Page 1 of 2 · Proposed for ARB approval · 23 October 2026**  
**Scope:** AI drafts campaign content and recommends segments; humans approve delivery. Assumes UAE federal PDPL, an existing PostgreSQL platform and a pilot workload. Privacy lead must confirm applicable jurisdiction, including any free-zone regime. Diagram: page 2, `marketing-ai-governance.drawio`.

## Architecture principles → business goals
1. **P1: Use the minimum permitted data.** Keep customer identities in CRM; send only necessary, pseudonymised attributes to AI. Treat embeddings as governed data. **Goal:** preserve customer trust and reduce privacy exposure.
2. **P2: Humans own customer-facing actions.** Agents may draft; only an authorised reviewer can approve an immutable content-and-audience version. Changes invalidate approval. **Goal:** protect brand reputation and avoid unwanted outreach.
3. **P3: Ground, test and trace AI output.** Retrieve approved brand material, treat retrieved text as untrusted input, version prompts/models and record evidence. **Goal:** improve accuracy and make incidents explainable.
4. **P4: Bound spending and failure.** Enforce campaign/token limits, bounded retries, idempotent sends and a kill switch. **Goal:** deliver predictable campaign economics and reliable operations.

## Architecture Decision Records
**ADR-01:  LLM hosting | Proposed | P1, P3, P4**  
**Options/trade-offs:** Managed API accelerates launch and reduces operations, but adds vendor, transfer and outage dependencies. Self-hosting improves infrastructure control but needs GPU capacity, security operations and model maintenance.  
**Decision:** Use a managed API behind a provider adapter, conditional on approved processing locations, contract, subprocessors, retention and no training on our data. Redact before egress; block sensitive data. If conditions fail, stop AI processing and reassess hosting.  
**Review trigger:** provider terms/location change, failed evals or sustained budget breach.

**ADR-02: Retrieval store | Proposed | P1, P3, P4**  
**Options/trade-offs:** PostgreSQL + pgvector reuses existing skills, access controls and backup operations; retrieval shares database capacity. A dedicated vector service offers independent scaling but adds cost, synchronisation and another processor.  
**Decision:** Use pgvector for approved brand/product knowledge only; enforce tenant/role filters before retrieval and propagate deletion to chunks, embeddings and caches. Keep customer profiles in CRM.  
**Review trigger:** representative load tests miss the ARB-approved retrieval latency or capacity target.

**ADR-03: Execution timing | Proposed | P2, P4**  
**Options/trade-offs:** Real-time generation is responsive but increases latency, spend and oversight pressure. Batch generation lowers cost and enables review, but content and audiences can become stale.  
**Decision:** Batch drafting and segment recommendations; queue approved versions. Immediately before each send, deterministically recheck consent, suppression, audience eligibility and frequency limits; fail closed on unavailable checks. Deduplicate retries; pause queued sends on withdrawal or kill switch.  
**Review trigger:** a validated use case requires interactive latency.

## ARB release checklist — evidence required
- [ ] **Security | Security lead:** threat model; least-privilege tool access; secret storage; encryption; injection/exfiltration tests; tenant isolation and incident/kill-switch drill.
- [ ] **Privacy / PDPL | Privacy lead:** confirm legal scope and processing basis; document notices, consent/withdrawal and profiling review; minimise data; assess processors/transfers; test retention, deletion and rights handling; assess DPIA/DPO obligations.
- [ ] **Evals | AI lead:** versioned, representative Arabic/English test set; brand, factuality, bias and injection coverage. Proposed pilot gate: ≥95% grounded factual claims and zero observed critical leaks or unauthorised sends; record sample size and residual risk.
- [ ] **Cost | Product/FinOps:** approve numeric per-campaign and monthly caps before launch; measure cost per approved asset; test alerts, hard stops and retry ceilings.
- [ ] **Human oversight | Marketing owner:** test approval binding, audit trail, rejection, rollback and emergency pause; name on-call owner. No approval means no send.

**ARB outcome:** approve / conditional / reject; record reviewer, evidence links, conditions, owner and expiry. Unresolved release checks block launch. Re-review material model, data, vendor or autonomy changes.  


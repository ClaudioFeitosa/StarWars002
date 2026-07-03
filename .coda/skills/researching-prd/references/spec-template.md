# Agent-Native PRD Spec Template

## Template Rules
- **No prose.** Every section is structured data only.
- **No narrative.** No "why this matters", no motivational text, no context paragraphs.
- **No formatting for humans.** No prominent text, no callouts, no visual emphasis.
- **All items carry status.** `complete` | `pending` | `assumption`.
- **All pending items appear in open_questions.** Single registry, no scatter.
- **Source tags are mandatory.** No source → cannot be `complete`.
- **Language:** Generate in DETECTED_LANGUAGE. Section headers in English always.

---

## Spec File Structure

```markdown
# {project_name} — PRD Spec
version: {NEW_VERSION}
session: {SESSION_ID}
mode: {SYNTHESIZE | REPAIR}
date: {ISO 8601}
language: {DETECTED_LANGUAGE}
status: {draft | complete}

---

## scope

### in_scope
- {scope item 1} | source: {ref}
- {scope item 2} | source: {ref}

### out_of_scope
- {exclusion 1} | source: {ref}
- [PENDING: out_of_scope] | status: pending

---

## personas

### P-01: {role_name}
- type: primary
- description: {from source}
- goals: {from source | [PENDING]}
- pain_points: {from source | [PENDING]}
- technical_proficiency: {from source | [PENDING]}
- usage_frequency: {from source | [PENDING]}
- status: {complete | pending}
- source: {ref}

### P-02: {role_name}
- type: secondary
- description: {from source}
- goals: {from source | [PENDING]}
- pain_points: {from source | [PENDING]}
- technical_proficiency: {from source | [PENDING]}
- usage_frequency: {from source | [PENDING]}
- status: {complete | pending}
- source: {ref}

---

## functional_requirements

### FR-01: {requirement_statement}
- priority: {must_have | should_have | could_have | wont_have | [PENDING]}
- source: {ref}
- status: {complete | pending}
- acceptance_criteria:
  - AC-01: given: {precondition} | when: {action} | then: {outcome}
  - AC-02: given: {precondition} | when: {action} | then: {outcome}

### FR-01a: {sub_requirement_statement}
- parent: FR-01
- priority: {inherited from parent}
- source: {ref}
- status: {complete | pending}
- acceptance_criteria:
  - AC-01: given: {precondition} | when: {action} | then: {outcome}

### FR-02: {requirement_statement}
- priority: {value}
- source: {ref}
- status: pending
- acceptance_criteria: [PENDING: acceptance_criteria for FR-02]

---

## non_functional_requirements

### NFR-01: {requirement_statement}
- category: {performance | security | scalability | availability | usability | compliance}
- target: {numeric_threshold}
- source: {ref}
- status: complete

### NFR-02: {requirement_statement}
- category: {category}
- target: {numeric_threshold} [ASM-01]
- source: inferred
- status: assumption

### NFR-03: {requirement_statement}
- category: {category}
- target: [PENDING: nfr_target]
- source: {ref}
- status: pending

---

## kpis

### KPI-01: {description}
- target: {numeric_value}
- measurement: {how collected}
- timeframe: {when measured}
- source: {ref}
- status: {complete | pending}

---

## jtbd

### JTBD-01: core_functional
- situation: {from source}
- motivation: {from source}
- outcome: {from source}
- source: {ref}
- status: {complete | pending}

### JTBD-02: related
- situation: {from source}
- motivation: {from source}
- outcome: {from source}
- source: {ref}
- status: {complete | pending}

### JTBD-E01: emotional
- situation: {from source}
- emotion: {from source}
- outcome: {from source}
- source: {ref}
- status: {complete | pending}

### JTBD-S01: social
- situation: {from source}
- perception: {from source}
- outcome: {from source}
- source: {ref}
- status: {complete | pending}

---

## traceability

| FR | JTBD | direction | status |
|----|------|-----------|--------|
| FR-01 | JTBD-01 | forward | mapped |
| FR-02 | — | forward | orphan |
| — | JTBD-03 | reverse | unsatisfied |

---

## dependencies

### DEP-01: {dependency_name}
- type: {external | internal | technical}
- description: {what it is}
- owner: {who owns it | [PENDING]}
- impact_if_unavailable: {consequence}
- source: {ref}
- status: {complete | pending}

---

## risks

### RSK-01: {risk_statement}
- likelihood: {high | medium | low | [PENDING]}
- impact: {high | medium | low | [PENDING]}
- mitigation: {strategy}
- source: {ref}
- status: {complete | pending}

---

## assumptions

### ASM-01: {assumption_statement}
- impact_if_wrong: {consequence}
- source_basis: {ref | inferred from ...}
- status: active

---

## constraints

### regulatory
- {constraint 1} | source: {ref}
- [PENDING: regulatory_constraints] | status: pending

### organizational
- {constraint 1} | source: {ref}

---

## mvp_phasing

### phase_1_mvp
- FR-01, FR-03, FR-05 (must_have)

### phase_2
- FR-02, FR-06 (should_have)

### phase_3
- FR-04, FR-07 (could_have)

### deferred
- FR-08 (wont_have)

### sequencing_notes
- FR-03 requires DEP-01 resolved before start
- FR-06 depends on FR-01 completion

---

## glossary

| term | definition | first_used |
|------|-----------|------------|
| {term} | {definition} | {section/ID} |

---

## open_questions

This section is the **single consolidated registry** of all unresolved items.
Every `[PENDING]` marker, every `orphan` traceability entry, every `assumption`
that needs validation — all appear here once.

Downstream skills (Epics, User Stories, Architecture, Implementation) read
ONLY this section to understand what is unresolved. They do NOT scan other
sections for [PENDING] markers — this registry IS the source of truth.

### pending_inputs

| id | type | section | item_id | field | impact |
|----|------|---------|---------|-------|--------|
| PI-01 | missing | personas | P-01 | pain_points | Cannot derive JTBD emotional jobs |
| PI-02 | missing | functional_requirements | FR-02 | acceptance_criteria | FR cannot be implemented without AC |
| PI-03 | missing | non_functional_requirements | NFR-03 | target | No measurable threshold for validation |
| PI-04 | missing | scope | — | out_of_scope | Risk of scope creep without explicit exclusions |
| PI-05 | missing | risks | RSK-01 | likelihood | Cannot prioritize risk mitigation |
| PI-06 | missing | constraints | — | regulatory | May miss compliance requirements |

### traceability_gaps

| id | direction | fr_id | jtbd_id | note |
|----|-----------|-------|---------|------|
| TG-01 | forward | FR-02 | — | Orphan FR: no JTBD served |
| TG-02 | reverse | — | JTBD-03 | Unsatisfied JTBD: no implementing FR |

### assumptions_to_validate

| id | assumption | impact_if_wrong | validation_method |
|----|-----------|-----------------|-------------------|
| ASM-01 | {statement} | {consequence} | {how to confirm/deny} |

### summary
- total_frs: {N}
- total_nfrs: {N}
- total_jtbds: {N}
- total_kpis: {N}
- pending_inputs: {N}
- traceability_gaps: {N}
- assumptions: {N}
- risks: {N}
- dependencies: {N}
- status: {draft | complete}
```

---

## Generation Rules

### Section Ordering
Generate sections in the order shown above. This order reflects dependency flow:
scope → personas → FRs → NFRs → KPIs → JTBDs → traceability → deps → risks →
assumptions → constraints → mvp_phasing → glossary → open_questions.

The traceability section MUST come after both FRs and JTBDs are generated.
The open_questions section is ALWAYS LAST — it consolidates from all prior sections.

### Consolidation Pass (Before Writing)
After generating all sections, perform a single consolidation pass:
1. Scan every section for items with status: pending → create PI-XX in open_questions
2. Scan traceability for orphan/unsatisfied → create TG-XX in open_questions
3. Scan assumptions → create entry in assumptions_to_validate
4. Compute summary counts
5. Set top-level status: `complete` if zero pending_inputs AND zero traceability_gaps.
   Otherwise: `draft`.

### REPAIR Mode
- Load existing spec file
- Apply REPAIR_DIRECTIVES to targeted sections
- Recalculate traceability (full rebuild if FRs or JTBDs changed)
- Rebuild open_questions from scratch (always reflects current state)
- Increment version, log changes

### Bias Detection (During Generation)
Apply inline — do not generate a separate bias report:
- Technology name not in source → rewrite as capability need
- Architecture decision without source → mark as ASM-XX
- Implementation detail in FR → move to notes field on that FR
- Vendor name in dependency without source → mark as ASM-XX
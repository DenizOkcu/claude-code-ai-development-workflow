---
model: opus
---

## Phase 3: Architecture & System Design

You are entering the **Design** phase for issue: `$ARGUMENTS`

### Pre-Conditions
- Read `STATUS.md` — confirm Research is complete
- Read `CODE_RESEARCH.md` for findings and recommendations
- Read `DISCOVERY.md` for scope and success criteria

### Instructions

Produce the architectural blueprint for this feature. Create these artifacts in `.claude/planning/$ARGUMENTS/`:

#### 1. `ARCHITECTURE.md`

- **System Context Diagram** (describe in text/Mermaid): how the feature sits in the overall system
- **Component Design**: new/modified components with responsibilities
- **Data Model**: schema changes, new entities, relationships
- **API Design**: endpoints, request/response shapes, status codes
- **State Management**: how state flows through the system
- **Error Handling Strategy**: error types, recovery, user-facing messages
- **Performance Considerations**: caching, pagination, lazy loading, connection pooling
- **Scalability Notes**: what happens at 10x, 100x current load

#### 2. `ADR-{NNN}-{title}.md` (Architecture Decision Records)

For each significant technical decision, create an ADR:

```markdown
# ADR-001: {Decision Title}

## Status: Accepted
## Date: {date}

## Context
What is the problem or situation that requires a decision?

## Decision
What is the decision that was made?

## Alternatives Considered
| Option | Pros | Cons |
|--------|------|------|
| Option A | ... | ... |
| Option B | ... | ... |

## Consequences
- Positive: ...
- Negative: ...
- Risks: ...

## References
- Links to relevant docs, articles, or discussions
```

#### 3. `PROJECT_SPEC.md`

- **Technical Requirements**: derived from DISCOVERY.md success criteria
- **Non-Functional Requirements**: performance targets, SLAs, accessibility
- **Interface Contracts**: TypeScript interfaces, API schemas, event payloads
- **Testing Requirements**: what must be tested and how
- **Migration Plan**: if data or schema changes are needed
- **Feature Flag Strategy**: how to progressively roll out
- **Rollback Plan**: how to undo if something goes wrong

### Post-Actions
- Update `STATUS.md`: mark Design as completed
- Add ADR summaries to Key Decisions section
- List all artifacts created
- Suggest next command: `/plan $ARGUMENTS`

### Quality Gates
- At least 1 ADR for any non-trivial decision
- ARCHITECTURE.md covers all 8 sections
- PROJECT_SPEC.md has measurable non-functional requirements
- Interface contracts use actual types (not just descriptions)
- Rollback plan is specific and actionable

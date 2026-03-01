## Bonus: AI/LLM Integration Guide

You are adding AI/LLM capabilities to issue: `$ARGUMENTS`

### Pre-Conditions
- Read `STATUS.md` — confirm Design phase is complete (or in progress)
- Read `ARCHITECTURE.md` for system context
- Read `PROJECT_SPEC.md` for requirements

### Instructions

Design the AI/LLM integration. Add an `AI_INTEGRATION.md` to `.claude/planning/$ARGUMENTS/`:

#### 1. Use Case Analysis

- **Task type**: Classification, generation, extraction, summarization, conversation, code generation, RAG, agents
- **Latency requirements**: Real-time (< 1s), near-real-time (< 5s), async (> 5s)
- **Accuracy requirements**: What error rate is acceptable?
- **Volume**: Expected requests per second/day
- **Cost sensitivity**: Budget per request, monthly cap

#### 2. Model Selection

| Criterion | Requirement | Recommendation |
|-----------|-------------|----------------|
| Task complexity | ... | Model tier |
| Latency | ... | Model size |
| Cost | ... | Provider/model |
| Privacy | ... | Self-hosted vs API |
| Context window | ... | Max tokens needed |

#### 3. Prompt Engineering

For each LLM-powered feature:

```markdown
### Feature: {name}

**System Prompt:**
```
{system prompt text}
```

**User Prompt Template:**
```
{template with {{variables}}}
```

**Output Format:**
{expected output structure — JSON schema, markdown, etc.}

**Few-Shot Examples:**
- Input: ... → Expected Output: ...
- Input: ... → Expected Output: ...

**Edge Cases:**
- What happens with empty input?
- What happens with adversarial input?
- What happens with extremely long input?
```

#### 4. RAG Architecture (if applicable)

- **Data sources**: What documents/data to index
- **Chunking strategy**: Size, overlap, semantic boundaries
- **Embedding model**: Model choice, dimensions
- **Vector store**: Technology choice (Pinecone, pgvector, Chroma, etc.)
- **Retrieval strategy**: Top-K, hybrid search, reranking
- **Context assembly**: How retrieved chunks are formatted for the LLM

#### 5. Guardrails & Safety

- **Input validation**: Content filtering, length limits, PII detection
- **Output validation**: Schema validation, toxicity filtering, hallucination checks
- **Rate limiting**: Per-user, per-feature limits
- **Fallback behavior**: What happens when the LLM fails or returns poor results
- **Human-in-the-loop**: When to escalate to human review
- **Prompt injection protection**: Input sanitization, output parsing

#### 6. Evaluation & Testing

| Metric | Method | Target |
|--------|--------|--------|
| Accuracy | Human evaluation / automated rubric | > N% |
| Latency | p50/p95/p99 measurement | < Nms |
| Cost | Per-request tracking | < $N/request |
| Relevance (RAG) | Hit rate, MRR, NDCG | > N |
| Safety | Red-team testing | 0 failures |

**Eval dataset**: Minimum 50 test cases covering:
- Happy path (60%)
- Edge cases (20%)
- Adversarial inputs (10%)
- Regression tests (10%)

#### 7. Cost Optimization

- Caching strategy (semantic cache, exact match cache)
- Prompt optimization (shorter prompts, fewer tokens)
- Model routing (cheap model for simple tasks, expensive for complex)
- Batch processing (where latency allows)
- Token budget management

### Post-Actions
- Add `AI_INTEGRATION.md` to the artifacts list in `STATUS.md`
- Feed the prompt designs into the Planning phase tasks
- Suggest incorporating eval tasks into the test strategy

### Quality Gates
- Every LLM feature has a system prompt, user template, and output format
- Guardrails are defined for both input and output
- Eval strategy includes adversarial test cases
- Cost estimate exists per feature
- Fallback behavior is defined (never just "show error")

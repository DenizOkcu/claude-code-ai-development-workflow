# AI Integration Guide for Software Projects

Practical patterns for integrating LLMs and AI agents into your software, derived from production best practices.

## When to Use AI/LLM Features

**Good candidates:** text generation, summarization, classification, semantic search, content moderation, code assistance, conversational interfaces, data extraction from unstructured sources.

**Poor candidates:** exact math, deterministic logic, real-time latency-critical paths (< 100ms), tasks requiring 100% accuracy with no tolerance for error.

## Architecture Patterns

### Pattern 1: Simple LLM Call
Direct API call with structured output. Best for: classification, extraction, single-turn generation.

```
User Input → Validate → LLM Call → Parse Output → Validate Output → Response
```

Key decisions: model selection, prompt template, output schema, retry strategy, fallback behavior.

### Pattern 2: RAG (Retrieval-Augmented Generation)
Combine search with generation. Best for: Q&A over documents, knowledge bases, context-aware assistance.

```
User Query → Embed Query → Vector Search → Rank Results → Assemble Context → LLM Call → Response
```

Key decisions: chunking strategy, embedding model, vector store, retrieval method (semantic, hybrid, reranking), context window management.

### Pattern 3: Agentic Workflow
LLM with tool access and iterative reasoning. Best for: complex multi-step tasks, research, coding assistance.

```
User Goal → Plan → [Execute Tool → Observe Result → Reason → Repeat] → Final Output
```

Key decisions: tool definitions, safety boundaries, max iterations, human-in-the-loop triggers, cost caps.

### Pattern 4: Multi-Agent Orchestration
Multiple specialized agents collaborating. Best for: complex workflows with distinct phases.

```
Orchestrator → [Agent A (research) + Agent B (analysis) + Agent C (writing)] → Synthesize → Output
```

Key decisions: agent specialization, communication protocol, conflict resolution, parallel vs sequential execution.

## Prompt Engineering Principles

1. **Be specific**: Vague prompts produce vague outputs
2. **Provide examples**: Few-shot prompting dramatically improves consistency
3. **Define output format**: JSON schema, XML tags, or structured templates
4. **Set constraints**: Length limits, tone, forbidden patterns
5. **Use system prompts**: Establish persona, rules, and context
6. **Chain of thought**: Ask the model to reason step-by-step for complex tasks
7. **Separate instructions from data**: Use clear delimiters (XML tags, markdown blocks)

## Evaluation Framework

### Automated Metrics
- **Accuracy**: Correct answers / total answers
- **Relevance (RAG)**: MRR, NDCG, Hit Rate
- **Latency**: p50, p95, p99 response times
- **Cost**: Tokens consumed per request, monthly spend
- **Safety**: Percentage of outputs passing content filters

### Human Evaluation
- Create a rubric with clear scoring criteria
- Use blind evaluation (evaluator doesn't know which model/prompt)
- Minimum 50 test cases for statistical significance
- Include adversarial and edge-case examples

### Regression Testing
- Maintain a golden dataset of input/output pairs
- Run after every prompt change
- Track metrics over time (not just point-in-time)

## Security & Safety Checklist

- [ ] Input sanitization (prevent prompt injection)
- [ ] Output validation (parse and verify before using)
- [ ] Rate limiting (per-user, per-feature)
- [ ] Content filtering (toxic, harmful, PII)
- [ ] Cost caps (per-request and monthly budgets)
- [ ] Audit logging (all LLM inputs and outputs for review)
- [ ] Fallback behavior (graceful degradation when LLM fails)
- [ ] User disclosure (make it clear when AI generates content)

## Cost Optimization Strategies

1. **Cache aggressively**: Semantic caching for similar queries, exact caching for identical
2. **Route by complexity**: Use smaller/cheaper models for simple tasks
3. **Optimize prompts**: Shorter prompts = fewer tokens = lower cost
4. **Batch when possible**: Group requests for async processing
5. **Set token budgets**: max_tokens per request, monthly caps per feature
6. **Monitor and alert**: Track spend in real-time, alert on anomalies

## Common Pitfalls

- **Over-relying on LLMs**: Not everything needs AI — use deterministic logic when possible
- **Ignoring latency**: LLM calls add 500ms-5s; design UX around this
- **No fallback path**: Always have a non-AI fallback for critical features
- **Prompt fragility**: Small prompt changes can cause large output changes; version and test prompts
- **Hallucination blindness**: LLMs confidently produce incorrect information; always validate critical facts
- **Token explosion**: Long conversations accumulate tokens quickly; implement context management
- **Evaluation gap**: Not testing thoroughly leads to production surprises

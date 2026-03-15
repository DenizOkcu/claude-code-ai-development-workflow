# Retrospective: add-code-intelligence-layer

## Summary
- **Started:** 2026-03-15
- **Completed:** 2026-03-15
- **Complexity:** M — estimated M, delivered M ✓
- **Files Changed:** 5 (all Markdown prompt files)
- **Tests Added:** 0 (prompt-engineering project — quality via checklists)
- **Phases Completed:** 7/10 (Deploy and Observe skipped — per learnings for prompt-only changes)

---

## What Went Well

- **Design phase paid off.** The full STRIDE threat model + project spec + 2 ADRs written upfront meant zero design debates during implementation. Every decision was pre-committed with rationale.
- **Context window as session cache (ADR-002)** turned out to be not just adequate but *better* than a file cache — no cleanup, no invalidation logic, no extra artifacts. Simplification by design.
- **Small-repo bypass gate** was correctly anticipated in Discovery. The < 50 file threshold is a clean, measurable rule that prevents the full pipeline from firing on trivial repos.
- **Combining Phases 2 and 3** (dependency graph + reranking + context pack) into a single `researching-code/SKILL.md` rewrite was the right call — they share state (the dependency graph feeds both reranking and context pack expansion) and the file stayed under 200 lines at 190.
- **Review caught a real bug** (missing symbol index generation instruction in `discover.md` Step 3) that would have caused silent feature failure — the template had the placeholder but the generation step didn't have the instruction. Fixed before merging.
- **Security audit confirmed** the 8-file context pack cap is a security *improvement* over the previous ad-hoc 5-15 file approach — less prompt injection surface, not more.

---

## What Could Be Improved

- **`discover.md` Step 3 is overloaded.** It now generates the repo map AND the symbol index, but the instruction to do both was added as a trailing note rather than as a proper numbered sub-step. A cleaner structure would be Step 3a (repo map) / Step 3b (symbol index).
- **Output path inconsistency inherited from base skill.** `researching-code/SKILL.md` Steps 3-4 still reference `docs/{issue_name}/RESEARCH.md` instead of `.claude/planning/{issue-name}/02_CODE_RESEARCH.md`. This pre-existing issue was noted in review but not fixed in this PR — it lives in the next issue's scope.
- **The Design phase added significant planning overhead** for what was ultimately 5 Markdown file edits. For a well-understood feature in a mature prompt-engineering codebase, the Architecture + Project Spec + 2 ADRs could have been compressed into a single lightweight design note. The full Design phase is better justified for features with runtime code or external integrations.

---

## Surprises / Unknowns Encountered

- **The review found the missing generation instruction in `discover.md` immediately.** The bug was subtle — the template placeholder referenced "/repo-map Step 7" but didn't make the LLM generate it during Step 3. Easy to miss in implementation, caught cleanly in review.
- **The token budget maths work out well.** Repo map ≤ 2K + symbol index ≤ 1K = ≤ 3K total structural overhead, which is less than 2% of Claude's context window. The concern about symbol index bloat was real but the truncation tiers handle it gracefully.
- **Prompt injection via code content is a pre-existing risk** that doesn't get amplified by the context pack — it actually decreases slightly because the cap (8 files) is tighter than the previous ad-hoc approach (5-15 files).

---

## Key Technical Learnings

1. **For prompt-engineering pipelines, "activation conditions" prevent feature waste at scale.** Both reranking (activates only if >5 candidates) and dependency graph (activates only if ≥50 files) are activation-conditioned — they don't fire on trivial inputs. This pattern keeps the happy path fast without degrading quality on complex inputs.

2. **Compact structured formats beat prose for LLM-consumed data.** The `type:name:file:line` symbol index format is ~5x more token-efficient than the inline `file — Symbol1, Symbol2()` format in the repo map, because it strips whitespace and formatting. Use compact formats whenever the consumer is a subsequent LLM step, not a human.

3. **Exclusion list divergence between `repo-map.md` Step 2 and `discover.md` Step 3 is a latent risk.** Both files maintain separate exclusion lists. A cross-reference note should be added to one pointing to the other as canonical — per the learning from `add-repo-context-engine`.

4. **When two prompt files share a procedure (repo map + symbol index generation), template placeholders alone are not enough** — the generation step must explicitly invoke the procedure. Found via code review: `discover.md` had the placeholder but not the call.

---

## Process Learnings

- **Phase selection was appropriate for a complex design change.** Even though the output was 5 Markdown files, the full SDLC (Discover → Research → Design → Plan → Implement → Review → Security) was justified because the feature required careful design decisions (session cache, reranking rubric, context pack cap) that benefited from the structured thought process.
- **Design phase can be compressed for well-understood prompt changes.** When the feature is purely additive to an existing skill (no new components, no new storage, no new commands), the Design phase could be replaced with a lightweight "design note" embedded in the Planning phase. Saves one full conversation turn.
- **Observe phase is always skippable for prompt-only changes.** Confirmed again — no metrics to configure, no dashboards, no alerts.
- **Security (7a) was valuable even for 5 Markdown files.** STRIDE caught the prompt injection surface (pre-existing, not amplified) and confirmed the 8-file cap as a security improvement. Worth running.

---

## Patterns to Reuse

- **Pattern: Multi-level activation conditions in skill pipelines**
  - Where: Any skill with optional enhancement steps
  - Why: Prevents expensive steps from firing on trivial inputs; keeps the baseline fast; activates enrichment only when it can help
  - Example: `if repo >= 50 files: build dependency graph; if candidates > 5: rerank`

- **Pattern: Compact `type:name:file:line` format for LLM-consumed structured data**
  - Where: Any data embedded in planning artifacts that downstream LLMs will parse
  - Why: ~5x more token-efficient than human-readable table format; trivially parseable by LLMs
  - Example: Symbol index

- **Pattern: Context window as intra-session cache**
  - Where: Any intermediate computation that doesn't need cross-session persistence
  - Why: Zero infrastructure, zero cleanup, zero invalidation logic
  - Example: Dependency graph, reranked results, context pack

---

## Anti-Patterns to Avoid

- **Anti-pattern: Template placeholder without generation instruction**
  - Why: The LLM executing the command may see the placeholder in the template but not know to generate the data without an explicit instruction in the generation step
  - Instead: Always pair template placeholders with a corresponding instruction in the step that generates the data

- **Anti-pattern: Separate exclusion lists in sibling commands**
  - Why: Lists drift independently; one gets updated, the other doesn't; silent behavioral divergence
  - Instead: Designate one file as canonical and add a cross-reference comment in the other: `# See repo-map.md Step 2 for the canonical exclusion list`

---

## Metrics
- **Estimation accuracy:** M estimated → M delivered (5 files, ~450 lines changed) ✓
- **Test coverage delta:** N/A (prompt-engineering project)
- **Review iterations:** 1 pass + 1 fix (missing `discover.md` generation instruction)
- **Security findings:** 0 critical, 0 high, 0 medium, 1 low (pre-existing, not amplified)

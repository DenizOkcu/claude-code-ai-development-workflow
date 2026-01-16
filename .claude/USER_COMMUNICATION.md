# User Communication Guidelines for SDLC Workflow

Minimal, impactful communication for autonomous workflow execution.

---

## Communication Touchpoints

```
Start → Research → Planning → Implementation → Review → Complete
         ↓          ↓           ↓               ↓         ↓
      notify    notify      notify          notify   notify
```

**Notify at:**
- Phase start (brief)
- Phase complete (results)
- Gate validation (pass/fail)
- Review-fix loop (iteration count)
- Workflow complete (summary)
- Errors (immediate)

---

## Templates

### Workflow Start
```markdown
🚀 **SDLC: {issue-name}**
{description}
Starting: {entry-point-phase}
```

### Phase Start
```markdown
📍 **{Phase Name}**
{goal}
```

### Phase Complete
```markdown
✅ **{Phase Name} Complete** ({duration})
Artifacts: {count}
Key: {brief finding}
```

### Validation Failed
```markdown
⚠️ **Gate Failed:** {gate}
Missing: {what's needed}
Action: {recovery}
```

### Review-Fix Loop
```markdown
🔄 **Fix Iteration {N}/3**
Issues: {N} critical, {M} important
```

### Workflow Complete
```markdown
## 🎉 Complete: {issue-name}

**Status:** {complete | blocked} | **Time:** {total}

**Summary:**
- ✓ Research, Planning, Implementation, Review
- Artifacts: {N} docs, {M} code files
- Review: {status}

**Next:**
1. `ls docs/{issue-name}/` - view artifacts
2. `cat docs/{issue-name}/STATUS.md` - check status
```

### Error
```markdown
❌ **{Phase}: {error-type}**
{message}
Recover: {how}
```

---

## Visual Markers

| Marker | Use |
|--------|-----|
| 🚀 | Workflow start |
| 📍 | Phase transition |
| ✅ | Phase complete |
| ⚠️ | Validation failed |
| 🔄 | Fix iteration |
| 🎉 | Complete |
| ❌ | Error |

---

## Principles

1. **One update per phase** - not per file/task
2. **Show results, not process** - outcome > activity
3. **Errors immediately** - don't wait
4. **Brief & scannable** - bullet points, short lines

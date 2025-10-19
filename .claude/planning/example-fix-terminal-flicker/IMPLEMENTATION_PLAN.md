# Implementation Plan: Fix Terminal View Flickering

> **⚠️ EXAMPLE ARTIFACT** - This is a reference example showing what `/issue-planner` produces. Do not modify. See [README.md](./README.md) for details.

**Issue:** Terminal view flickering when at full height during timer and input updates
**Created:** 2025-10-11
**Status:** Ready for Implementation

---

## 1. Overview

### Summary

Fix terminal view flickering that occurs when the terminal window is at full height and state updates trigger re-renders. The flickering is caused by Ink's rendering behavior: when content height ≈ terminal height, any React state update triggers a full screen clear and repaint.

### Goals

1. **Eliminate visible flicker** during timer updates (every 1 second in ThinkingIndicator)
2. **Prevent unnecessary re-renders** from input field typing
3. **Maintain current functionality** - timer display, input responsiveness, and all features
4. **Improve performance** without changing user-visible behavior

### Success Criteria

- [ ] No visible flicker when terminal is at full height
- [ ] Timer continues to display elapsed seconds accurately
- [ ] Input field remains responsive with no perceivable lag
- [ ] All existing tests pass
- [ ] No regression in functionality

---

## 2. Technical Analysis

### Current State Assessment

**Key Problem Areas:**

1. **ThinkingIndicator Timer** (source/components/thinking-indicator.tsx:54-64)

   - Updates `elapsedSeconds` state every 1000ms unconditionally
   - Triggers re-render even when displayed value doesn't change
   - Has TWO timer intervals: one for seconds (1s), one for word rotation (3s)

2. **UserInput Component** (source/components/user-input.tsx:290-297)

   - Uses `ink-text-input` which triggers `onChange` on every keystroke
   - Properly memoized but state updates still propagate through app
   - Debouncing only applies to `hasLargeContent` flag (50ms)

3. **Layout Structure** (source/app.tsx:259-268)
   - Uses `flexGrow={1}` and `minHeight={0}` pattern (correct approach)
   - No explicit height constraints based on terminal dimensions
   - When ChatQueue + bottom controls ≈ terminal height → flicker trigger

### Technology Stack Considerations

- **Ink 6.3.1**: Known limitation - full repaints when content fills screen
- **React 19.0.0**: Memoization and optimization patterns available
- **TypeScript 5.0**: Strict typing ensures type safety during refactor

### Dependencies and Prerequisites

- No new dependencies required
- Existing patterns (memoization, layout optimization) already established
- Previous fixes provide proven patterns to build upon

### Root Cause Analysis

**Why Flickering Occurs:**

```
Timer Update (1000ms) → setElapsedSeconds() → React Reconciliation
                                           ↓
                                   Ink Render Check
                                           ↓
                            Content Height ≥ Terminal Height?
                                           ↓
                                         YES
                                           ↓
                                Alternate Screen Buffer Mode
                                           ↓
                        Clear Screen + Full Repaint = FLICKER
```

**Historical Context:**

- This is the 4th iteration of flicker fixes (commits: 81f6d0b, 453c63d, df6038c)
- Previous fixes: memoization, layout optimization, state isolation
- Core issue persists: any state update can trigger flicker when at capacity

---

## 3. Implementation Phases

### Phase 1: Optimize Timer State Updates (HIGH PRIORITY)

**Complexity:** Simple
**Risk:** Low
**Impact:** High - directly addresses primary flicker source

#### Tasks:

1. **Modify timer update logic to prevent redundant state changes**

   - File: `source/components/thinking-indicator.tsx`
   - Lines: 54-64
   - Change: Only call `setElapsedSeconds()` when value actually changes

   **Current Implementation:**

   ```tsx
   useEffect(() => {
   	const timer = setInterval(() => {
   		const currentTime = Date.now();
   		const elapsed = Math.floor((currentTime - startTimeRef.current) / 1000);
   		setElapsedSeconds(elapsed); // Called every 1s regardless
   	}, 1000);
   	return () => clearInterval(timer);
   }, []);
   ```

   **New Implementation:**

   ```tsx
   useEffect(() => {
   	const timer = setInterval(() => {
   		const currentTime = Date.now();
   		const elapsed = Math.floor((currentTime - startTimeRef.current) / 1000);
   		// Only update state if value changed
   		setElapsedSeconds(prev => (prev !== elapsed ? elapsed : prev));
   	}, 1000);
   	return () => clearInterval(timer);
   }, []);
   ```

2. **Optimize word rotation timer similarly**

   - Lines: 66-74
   - Use functional state update to prevent redundant renders
   - Consider if random word changes are necessary or can be reduced in frequency

3. **Add memoization to computed values**
   - Lines: 76-89 (percentage, tokensPerSecond, dots)
   - Wrap in `useMemo()` to prevent recalculation on parent re-renders

#### Files to Modify:

- `source/components/thinking-indicator.tsx` (Primary)

#### Testing:

- Manual testing with terminal at full height
- Add render counting during development to verify reduction
- Ensure timer accuracy not affected

---

### Phase 2: Enhance Input Component Memoization (MEDIUM PRIORITY)

**Complexity:** Simple
**Risk:** Low
**Impact:** Medium - prevents input-triggered re-renders

#### Tasks:

1. **Verify UserInput component is fully memoized**

   - File: `source/components/user-input.tsx`
   - Confirm all props passed are stable (wrapped in useCallback/useMemo)
   - Current: Not wrapped in `React.memo()` - NEEDS FIX

2. **Wrap UserInput in React.memo()**

   - Add memoization at component export level
   - Provide custom comparison function if needed

   **Implementation:**

   ```tsx
   export default memo(function UserInput({ ... }: ChatProps) {
     // existing implementation
   });
   ```

3. **Audit all callback props passed to UserInput**

   - From `app.tsx`: `handleMessageSubmit`, `handleCancel`, `handleToggleDevelopmentMode`
   - Lines: 172-210 in app.tsx
   - Verify all are wrapped in `useCallback` with correct dependencies
   - Current: Already properly memoized ✓

4. **Optimize useInputState hook**
   - File: `source/hooks/useInputState.ts`
   - Already well-optimized with debouncing (50ms)
   - No changes needed unless testing reveals issues

#### Files to Modify:

- `source/components/user-input.tsx` (Primary)

#### Testing:

- Type rapidly in input field with terminal at full height
- Verify no visible flicker
- Ensure input remains responsive

---

### Phase 3: Implement Dynamic Layout Height Constraints (LOW PRIORITY)

**Complexity:** Medium
**Risk:** Medium - layout changes can affect rendering
**Impact:** Medium - provides buffer to prevent reaching flicker threshold

#### Tasks:

1. **Create useTerminalHeight hook**

   - Pattern: Similar to existing `useTerminalWidth` (source/hooks/useTerminalWidth.tsx)
   - Location: `source/hooks/useTerminalHeight.tsx` (new file)
   - Functionality: Track terminal height changes, only update state when changed

   **Implementation:**

   ```tsx
   import {useEffect, useState} from 'react';

   export const useTerminalHeight = () => {
   	const [height, setHeight] = useState(process.stdout.rows ?? 24);

   	useEffect(() => {
   		const handleResize = () => {
   			const newHeight = process.stdout.rows ?? 24;
   			setHeight(prev => (prev !== newHeight ? newHeight : prev));
   		};

   		process.stdout.on('resize', handleResize);
   		return () => {
   			process.stdout.off('resize', handleResize);
   		};
   	}, []);

   	return height;
   };
   ```

2. **Calculate maximum chat area height**

   - File: `source/app.tsx`
   - Lines: 259-268
   - Reserve space for fixed bottom controls
   - Apply explicit height constraint to prevent overflow

   **Implementation:**

   ```tsx
   const terminalHeight = useTerminalHeight();
   const reservedRows = 8; // Input + indicators + padding
   const maxChatHeight = Math.max(10, terminalHeight - reservedRows);

   <Box
     flexDirection="column"
     height={maxChatHeight}
     overflow="hidden"
   >
     <WelcomeMessage />
     <ChatQueue ... />
   </Box>
   ```

3. **Test with various terminal sizes**
   - Small terminals (< 24 rows)
   - Medium terminals (24-50 rows)
   - Large terminals (> 50 rows)
   - Dynamic resizing during operation

#### Files to Create:

- `source/hooks/useTerminalHeight.tsx` (new)

#### Files to Modify:

- `source/app.tsx`

#### Testing:

- Resize terminal during operation
- Verify layout remains stable
- Ensure no layout shift artifacts

---

### Phase 4: Testing and Validation

**Complexity:** Simple
**Risk:** Low
**Impact:** Critical - ensures fix is complete

#### Tasks:

1. **Manual testing across scenarios**

   - Terminal at full height with long conversation
   - Rapid typing in input field
   - Timer running for extended periods (> 60 seconds)
   - Terminal resize during operation

2. **Performance verification**

   - Add temporary render counting (development only)
   - Measure re-render frequency before and after
   - Target: Reduce from 1 render/second to near-zero when idle

3. **Regression testing**

   - Run existing test suite: `npm test`
   - Verify no functionality changes
   - Check timer accuracy (compare displayed time vs actual elapsed)

4. **Edge case testing**
   - Very small terminals (< 10 rows)
   - Very large content in input field
   - Multiple rapid state changes (timer + input + other updates)

#### Test Coverage:

- Unit tests: Timer update logic
- Integration tests: Component interaction without flicker
- Manual tests: Visual verification across terminal sizes

---

## 4. Testing Strategy

### Unit Tests Required

1. **ThinkingIndicator Timer Logic**

   ```tsx
   // Test: Timer only updates state when value changes
   // File: source/components/__tests__/thinking-indicator.test.tsx

   test('timer state only updates when seconds change', () => {
   	const {rerender} = render(<ThinkingIndicator {...props} />);
   	// Mock timers and verify setElapsedSeconds called with functional update
   });
   ```

2. **UserInput Memoization**

   ```tsx
   // Test: Component doesn't re-render when props haven't changed
   // File: source/components/__tests__/user-input.test.tsx

   test('does not re-render when parent updates unrelated state', () => {
   	// Render with stable props, trigger parent re-render, verify no re-render
   });
   ```

### Integration Tests Required

1. **App Layout with Full Height**

   - Simulate terminal at full height
   - Verify no errors when content reaches limit
   - Test with useTerminalHeight hook

2. **Timer + Input Simultaneous Updates**
   - Simulate typing while timer is running
   - Verify smooth operation without flicker (difficult to test visually in unit tests)

### Manual Testing Checklist

- [ ] Start app with small terminal (< 20 rows)
- [ ] Fill terminal with conversation until at capacity
- [ ] Observe timer for 30 seconds - no flicker visible
- [ ] Type rapidly in input field - no flicker visible
- [ ] Resize terminal larger and smaller - layout adapts smoothly
- [ ] Long-running timer (> 60 seconds) - continues to work correctly
- [ ] Multiple thinking indicators in sequence - no accumulated flicker

---

## 5. Timeline Estimates

### Phase 1: Optimize Timer State Updates

- **Development:** 1-2 hours
- **Testing:** 30 minutes
- **Total:** 1.5-2.5 hours

### Phase 2: Enhance Input Memoization

- **Development:** 30 minutes
- **Testing:** 30 minutes
- **Total:** 1 hour

### Phase 3: Dynamic Layout Height

- **Development:** 2-3 hours
- **Testing:** 1 hour
- **Total:** 3-4 hours

### Phase 4: Testing and Validation

- **Comprehensive testing:** 2 hours
- **Documentation updates:** 30 minutes
- **Total:** 2.5 hours

### Overall Estimate

**Minimum:** 8 hours
**Maximum:** 10 hours
**Recommended:** 9 hours (includes buffer for unexpected issues)

### Critical Path

Phase 1 → Phase 2 → Phase 4 (Phase 3 can be done in parallel or skipped if Phases 1-2 solve the issue)

---

## 6. Risk Assessment and Mitigation

### Risk 1: Timer Optimization Insufficient

**Probability:** Low
**Impact:** Medium
**Mitigation:**

- Phase 3 provides additional buffer
- Can reduce timer update frequency as fallback (every 2s instead of 1s)

### Risk 2: Layout Changes Introduce New Bugs

**Probability:** Medium
**Impact:** Medium
**Mitigation:**

- Make Phase 3 optional
- Test thoroughly before implementing
- Use existing `useTerminalWidth` pattern as proven reference

### Risk 3: Ink Framework Limitation Cannot Be Overcome

**Probability:** Low
**Impact:** High
**Mitigation:**

- Research shows previous fixes partially worked
- Multiple optimization strategies available
- Worst case: reduce update frequency to acceptable level

### Risk 4: Performance Regression

**Probability:** Very Low
**Impact:** Low
**Mitigation:**

- All changes optimize performance, not degrade it
- Comprehensive testing before merge

---

## 7. Rollback Strategy

If issues arise after implementation:

1. **Phase 1 Rollback:**

   - Revert `thinking-indicator.tsx` to previous version
   - Timer will work but flicker persists
   - Zero functionality impact

2. **Phase 2 Rollback:**

   - Remove `React.memo()` wrapper from UserInput
   - Input continues to work normally
   - Zero functionality impact

3. **Phase 3 Rollback:**
   - Remove `useTerminalHeight` hook and explicit height constraints
   - Layout returns to flexGrow pattern
   - Zero functionality impact

All phases are non-breaking and can be individually reverted.

---

## 8. Notes and Considerations

### Design Decisions

1. **Why functional state updates?**

   - Pattern: `setElapsedSeconds(prev => prev !== elapsed ? elapsed : prev)`
   - Benefit: React skips re-render if state value unchanged
   - This is the key optimization for timer flicker

2. **Why not reduce timer frequency?**

   - User expectation: seconds counter should update every second
   - Alternative approach if optimization insufficient
   - Could implement as configurable option

3. **Why Phase 3 is optional?**
   - Phases 1-2 should solve the core issue
   - Phase 3 provides defense-in-depth
   - Can be implemented later if needed

### Alternative Approaches Considered

1. **Use requestAnimationFrame instead of setInterval**

   - Pros: Smoother, batches with browser rendering
   - Cons: Node.js CLI doesn't have RAF API
   - Verdict: Not feasible for CLI environment

2. **Separate timer display to different component**

   - Pros: Isolates re-render scope
   - Cons: Already done in commit 81f6d0b
   - Verdict: Already implemented

3. **Use CSS animations for dots instead of state**
   - Pros: No React re-renders needed
   - Cons: Ink doesn't support CSS animations
   - Verdict: Not possible with current framework

### Future Enhancements

1. **Configurable timer update frequency**

   - Allow user to set timer precision (1s, 2s, 5s)
   - Trade-off between smoothness and performance

2. **Performance metrics dashboard**

   - Track render counts in development mode
   - Help identify future performance issues

3. **Virtual scrolling for very long conversations**
   - Already partially implemented (displayCount = 5 in ChatQueue)
   - Could be enhanced for better memory management

---

## 9. Success Metrics

### Quantitative Metrics

- **Re-render frequency:** Reduce from ~1/second to < 0.1/second when idle
- **Timer accuracy:** Maintain ±100ms accuracy
- **Input latency:** Maintain < 50ms response time

### Qualitative Metrics

- **Visual flicker:** None visible to human eye (< 16ms frame time)
- **User experience:** No perceived change in functionality
- **Code maintainability:** Clear, documented, follows existing patterns

---

## 10. Dependencies and Prerequisites

### Required Knowledge

- React hooks and memoization patterns
- Ink rendering lifecycle
- Terminal escape sequences and screen buffer modes

### Required Tools

- Node.js >= 16
- TypeScript 5.0
- Existing development environment

### Required Access

- Write access to repository
- Ability to run tests locally
- Terminal emulator for manual testing

---

## Next Steps After Implementation

1. **Documentation:**

   - Update CODE_RESEARCH.md with findings
   - Add comments explaining optimization techniques
   - Document in git commit messages

2. **Monitoring:**

   - Watch for user reports of flicker
   - Monitor performance in various terminal sizes
   - Track any regression issues

3. **Future Work:**
   - Consider implementing Phase 3 if needed
   - Explore additional performance optimizations
   - Document learnings for future flicker prevention

---

**Ready for Implementation!**
Proceed with Phase 1, then Phase 2, testing after each phase.
Phase 3 can be implemented if needed based on test results.

# Code Research: Terminal View Flicker Issue

> **⚠️ EXAMPLE ARTIFACT** - This is a reference example showing what `/research-code` produces. Do not modify. See [README.md](./README.md) for details.

**Research Date:** 2025-10-11
**Issue:** View flickering when terminal window is at full height during character changes (input field typing and timer updates)

---

## 1. Executive Summary

The nanocoder CLI application experiences flickering when the terminal view reaches full height. This occurs during:

1. User typing in the input field
2. Timer updates in the `ThinkingIndicator` component (updates every 1 second)

**Key Findings:**

- This is a **known, recurring issue** that has been addressed multiple times in the codebase history
- The flickering is caused by **excessive re-renders** triggering Ink's full terminal re-paint when vertical space is constrained
- Previous fixes (commits `81f6d0b`, `453c63d`, `df6038c`) focused on memoization, layout optimization, and moving timer state into components
- The root cause is Ink's rendering behavior: when content height >= terminal height, updates cause the entire screen to be cleared and redrawn

**Risk Level:** **MEDIUM** - The issue is well-understood with established patterns for mitigation, but may require multiple optimization strategies to fully resolve.

---

## 2. Project Architecture

### Technology Stack

- **Runtime:** Node.js (>=16)
- **Language:** TypeScript 5.0
- **UI Framework:** [Ink 6.3.1](https://github.com/vadimdemedes/ink) - React for CLI interfaces
- **State Management:** React hooks with custom state management
- **Build Tool:** TypeScript compiler
- **Testing:** Ava, Ink Testing Library

### Key Dependencies

- **ink**: React-based terminal UI framework
- **ink-text-input**: Text input component for Ink
- **ink-spinner**: Spinner animations
- **@mishieck/ink-titled-box**: Bordered box components
- **react**: 19.0.0 (for Ink components)
- **chalk**: Terminal string styling
- **cli-highlight**: Syntax highlighting in terminal

### Architectural Patterns

- **Component-based architecture**: React functional components with hooks
- **State isolation**: Custom hooks for state management (`useAppState`, `useInputState`, `useUIState`)
- **Provider pattern**: Context providers for theme and UI state
- **Memoization-first**: Heavy use of `React.memo()`, `useMemo()`, and `useCallback()`

---

## 3. Relevant Existing Code

### Previous Flicker Fixes (Git History)

The codebase has a clear history of addressing this exact issue:

1. **Commit `81f6d0b` (2025-08-24)** - "fix: jittering"

   - Moved timer state from parent (`useAppState`) into `ThinkingIndicator` component
   - Added `flexGrow={1}` and `minHeight={0}` to ChatQueue container
   - Changed layout structure to separate scrollable content from fixed bottom controls

2. **Commit `453c63d` (2025-08-24)** - "fix: massive rerender problem causing scroll issues"

   - Added memoization to `ThinkingIndicator`
   - Optimized `useChatHandler` to reduce state updates

3. **Commit `df6038c` (2025-08-28)** - "fix: more aggressive memoisation throughout"
   - Wrapped multiple components in `React.memo()`
   - Optimized `useTerminalWidth` to only update on actual width changes
   - Added memoization to `ChatQueue`, `AssistantMessage`, `ErrorMessage`, `UserMessage`, `ToolMessage`

### Current Layout Structure (source/app.tsx:256-268)

```tsx
<Box flexDirection="column" padding={1} width="100%">
	<Box flexGrow={1} flexDirection="column" minHeight={0}>
		<WelcomeMessage />
		{appState.startChat && (
			<ChatQueue
				staticComponents={staticComponents}
				queuedComponents={appState.chatComponents}
			/>
		)}
	</Box>
	{appState.startChat && (
		<Box flexDirection="column">
			{/* ThinkingIndicator, UserInput, and other bottom controls */}
		</Box>
	)}
</Box>
```

**Key Pattern:** Separate container with `flexGrow={1}` and `minHeight={0}` for scrollable content, with fixed bottom section for controls.

### Timer Implementation (source/components/thinking-indicator.tsx:49-74)

```tsx
// Local state for elapsed time
const [elapsedSeconds, setElapsedSeconds] = useState(0);
const startTimeRef = useRef<number>(Date.now());

// Timer updates every 1000ms
useEffect(() => {
	const timer = setInterval(() => {
		const currentTime = Date.now();
		const elapsed = Math.floor((currentTime - startTimeRef.current) / 1000);
		setElapsedSeconds(elapsed);
	}, 1000);

	return () => {
		clearInterval(timer);
	};
}, []);
```

**Issue:** Each timer tick triggers a state update → component re-render → potential full screen re-paint.

### Input Handling (source/components/user-input.tsx)

The `UserInput` component uses:

- `ink-text-input` for the text field
- `useInputState` hook for input state management
- Debounced updates for large content detection (50ms)
- Key remounting strategy (`textInputKey`) for cursor position reset

---

## 4. Integration Points

### Files Requiring Modification

1. **source/components/thinking-indicator.tsx** (Primary)

   - Timer state management
   - Render optimization

2. **source/components/user-input.tsx** (Secondary)

   - Input change event handling
   - State update patterns

3. **source/app.tsx** (Layout)

   - Container layout structure
   - Component composition

4. **source/hooks/useInputState.ts** (State)
   - Input state updates
   - Debouncing logic

### Component Relationships

```
App
├── ThemeContext.Provider
└── UIStateProvider
    └── Box (root container)
        ├── Box (flexGrow=1, scrollable area)
        │   ├── WelcomeMessage
        │   └── ChatQueue
        │       ├── Status (staticComponents)
        │       └── Message components (queuedComponents)
        └── Box (fixed bottom area)
            ├── ThinkingIndicator ← Timer updates here
            └── UserInput ← Text input here
```

---

## 5. Code Conventions

### TypeScript Patterns

- Strict type definitions for all props and state
- Interface-based prop types (e.g., `ThinkingIndicatorProps`)
- Type imports from centralized `types/` directory
- Explicit return types for functions

### React Patterns

1. **Component Definition:**

   ```tsx
   export default memo(function ComponentName({prop1, prop2}: Props) {
   	// Implementation
   });
   ```

2. **Hook Usage:**

   - Custom hooks prefixed with `use`
   - State hooks at top of component
   - Effect hooks after state hooks
   - Memoization hooks last

3. **Memoization Strategy:**
   - Wrap components in `React.memo()`
   - Use `useMemo()` for computed values
   - Use `useCallback()` for event handlers
   - Memoize static component arrays

### File Organization

- Components: `source/components/`
- Hooks: `source/hooks/`
- Types: `source/types/`
- App logic: `source/app/hooks/`

### Naming Conventions

- Components: PascalCase (e.g., `ThinkingIndicator`)
- Files: kebab-case (e.g., `thinking-indicator.tsx`)
- Hooks: camelCase with `use` prefix
- Types: PascalCase with descriptive suffixes (e.g., `ThinkingIndicatorProps`)

---

## 6. Dependencies & Data Flow

### State Flow for Timer Updates

```
ThinkingIndicator
  ├── useState(elapsedSeconds)
  ├── useRef(startTimeRef)
  ├── useEffect(() => setInterval)
  └── Re-render every 1000ms
      └── Ink reconciliation
          └── Terminal re-paint (if height constrained)
```

### State Flow for Input Changes

```
UserInput
  ├── useInputState()
  │   ├── useState(input)
  │   └── updateInput(newValue)
  │       └── setInput(newValue) ← Triggers re-render
  └── TextInput (ink-text-input)
      └── onChange event
          └── updateInput called on every keystroke
```

### Key Data Structures

**ThinkingStats** (source/app/hooks/useAppState.tsx:11-16):

```typescript
export interface ThinkingStats {
	tokenCount: number;
	contextSize: number;
	totalTokensUsed: number;
	tokensPerSecond?: number;
}
```

---

## 7. Risks & Challenges

### Technical Debt

1. **Multiple previous flicker fixes** - Suggests the root cause may not be fully addressed
2. **Ink's rendering limitations** - When content height ≈ terminal height, Ink clears screen on updates
3. **Timer-based state updates** - Inherently causes periodic re-renders

### Performance Concerns

1. **Full screen re-paints** - Visible flicker when terminal is at capacity
2. **Cascading re-renders** - Timer update in `ThinkingIndicator` may trigger parent re-renders
3. **Input keystroke latency** - Each keystroke triggers state update + re-render

### Testing Gaps

- No specific tests for render performance
- No tests for terminal height edge cases
- Limited integration tests for UI components

### Breaking Change Risks

- **LOW** - Changes are localized to specific components
- Layout modifications are well-established patterns in the codebase

---

## 8. Recommendations

### Primary Solutions (Ranked by Effectiveness)

#### 1. **Optimize Timer Updates (Highest Priority)**

**Problem:** Timer updates every 1 second, triggering unnecessary re-renders of derived values.

**Solution:**

- Only update state when **displayed value changes** (not on every tick)
- Use `useRef` for tracking time, update state only for display changes
- Implement "smart" rendering: update dots animation without changing text

**Implementation:**

```tsx
// Instead of updating elapsedSeconds every 1s:
useEffect(() => {
	const timer = setInterval(() => {
		setElapsedSeconds(prev => prev + 1); // Forces re-render
	}, 1000);
}, []);

// Update only when display changes:
useEffect(() => {
	const startTime = Date.now();
	const timer = setInterval(() => {
		const newSeconds = Math.floor((Date.now() - startTime) / 1000);
		setElapsedSeconds(prev => (prev !== newSeconds ? newSeconds : prev));
	}, 1000);
}, []);
```

#### 2. **Prevent Unnecessary Re-renders in UserInput**

**Problem:** Each keystroke updates state, potentially triggering parent re-renders.

**Solution:**

- Ensure `UserInput` is fully memoized
- Verify parent doesn't re-render on child state changes
- Use `useCallback` for all event handlers passed to child components

#### 3. **Implement Virtual Scrolling for ChatQueue**

**Problem:** When ChatQueue grows to terminal height, any update causes full re-paint.

**Solution:**

- Limit rendered messages (already implemented: `displayCount = 5`)
- Consider increasing this limit or making it dynamic based on terminal height
- Implement windowing for very long conversations

#### 4. **Optimize Box Layout Strategy**

**Current implementation** (app.tsx:260) uses `flexGrow={1}` and `minHeight={0}`.

**Enhancement:**

- Add explicit height calculation based on `process.stdout.rows`
- Reserve space for bottom controls to prevent layout shifts
- Example:

  ```tsx
  const reservedRows = 8; // For input + indicators
  const maxChatHeight = Math.max(10, process.stdout.rows - reservedRows);

  <Box flexDirection="column" height={maxChatHeight}>
  	<ChatQueue />
  </Box>;
  ```

### Debugging Approaches

1. **Add render counting:**

   ```tsx
   const renderCount = useRef(0);
   renderCount.current += 1;
   console.log(`ThinkingIndicator render #${renderCount.current}`);
   ```

2. **Track parent re-renders:**

   - Add logging to `app.tsx` to see if timer updates propagate upward

3. **Profile with React DevTools:**
   - Although CLI-based, can add profiling hooks to measure re-render frequency

### Long-term Improvements

1. **Implement RAF-based rendering:**

   - Use `requestAnimationFrame` pattern for smoother updates
   - Batch multiple state updates into single render cycle

2. **Consider alternative timer visualization:**

   - Use rotating spinner instead of seconds counter
   - Reduce update frequency (every 2-3 seconds instead of 1)

3. **Terminal height monitoring:**
   - Create `useTerminalHeight` hook (similar to `useTerminalWidth`)
   - Dynamically adjust layout based on available space

---

## 9. Key Files Reference

### Core Component Files

```
source/app.tsx:256-268                     - Main layout structure with flexGrow pattern
source/components/thinking-indicator.tsx:49-74  - Timer implementation causing flicker
source/components/user-input.tsx:1-344     - Input handling and state management
source/components/chat-queue.tsx:1-38      - Message queue rendering
source/hooks/useInputState.ts:1-78         - Input state management
source/hooks/useTerminalWidth.tsx:1-28     - Terminal width tracking pattern
source/app/hooks/useAppState.tsx:1-230     - Global app state
```

### Previous Fix Commits

```
81f6d0b - fix: jittering (2025-08-24)
453c63d - fix: massive rerender problem (2025-08-24)
df6038c - fix: more aggressive memoisation (2025-08-28)
```

---

## 10. Questions for Planning

### Critical Questions

1. **Performance targets:** What is acceptable update latency for the timer display?
2. **User experience:** Is the seconds counter essential, or can we use alternative indicators?
3. **Testing strategy:** How should we verify the fix across different terminal sizes?

### Technical Decisions

1. Should we implement `useTerminalHeight` hook for dynamic layout adjustment?
2. Is it acceptable to reduce timer update frequency (e.g., 2-3 seconds instead of 1)?
3. Should we add render performance metrics/logging?

### Implementation Priorities

1. Start with timer optimization (highest impact, lowest risk)
2. Then address input memoization
3. Finally consider layout enhancements

---

## Root Cause Analysis

### Ink Rendering Behavior

When terminal content height ≈ available terminal rows, Ink uses an "alternate screen buffer" mode. In this mode:

- Any component update triggers full screen clear
- Content is reprinted from top to bottom
- This causes visible flicker

### Why This Occurs

1. **Timer updates** → `setElapsedSeconds` called every 1000ms
2. **State change** → React reconciliation
3. **Ink detects change** → Checks if output fits in terminal
4. **Height constraint detected** → Switches to alternate screen mode
5. **Full repaint** → Clear + redraw entire UI = **FLICKER**

### Why Previous Fixes Helped But Didn't Solve Completely

- **Memoization** reduced unnecessary child re-renders
- **Layout changes** reduced content height, delaying the threshold
- **Moving timer state** prevented parent re-renders
- **But:** The timer still updates state every 1s, causing inevitable re-render

### The Core Issue

**Any state update in a visible component will cause flicker when terminal is at capacity.**
The only solutions:

1. Reduce update frequency
2. Prevent re-renders (render same output)
3. Reduce content height below threshold
4. Accept flicker as Ink limitation

---

## Next Steps

Research complete! The findings have been documented in CODE_RESEARCH.md.

**Recommended next command:**

```
/issue-planner fix terminal view flickering when at full height by optimizing timer and input updates
```

This will create a detailed implementation plan and project specification based on the research findings.

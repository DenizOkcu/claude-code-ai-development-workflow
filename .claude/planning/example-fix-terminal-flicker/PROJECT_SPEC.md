# Project Specification: Terminal View Flicker Fix

> **⚠️ EXAMPLE ARTIFACT** - This is a reference example showing what `/issue-planner` produces. Do not modify. See [README.md](./README.md) for details.

**Project:** Nanocoder CLI Application
**Feature:** Eliminate terminal view flickering at full height
**Version:** 1.0
**Created:** 2025-10-11
**Status:** Planning Complete

---

## 1. Executive Summary

### Project Overview

Fix the recurring terminal view flickering issue in the nanocoder CLI application. The flicker occurs when the terminal window reaches full height and React component state updates trigger Ink framework's full screen re-paint behavior. This is the fourth iteration of flicker fixes, building on previous optimizations with a focus on preventing unnecessary state updates.

### Business Value

- **Improved user experience:** Smooth, flicker-free interface during intensive operations
- **Professional appearance:** CLI application maintains visual quality standards
- **Reduced user frustration:** Eliminates distracting visual artifacts
- **Performance optimization:** Fewer unnecessary re-renders improve overall responsiveness

### Key Stakeholders

- **End Users:** Developers using the CLI tool
- **Maintainers:** Core development team
- **Contributors:** Open source community

---

## 2. Functional Requirements

### FR-1: Timer Display Without Flicker

**Priority:** HIGH
**Description:** The ThinkingIndicator component displays elapsed time and updates every second without causing visible screen flicker.

**User Story:**
As a user waiting for the AI to respond, I want to see the elapsed time update smoothly without the entire terminal screen flickering, so I can monitor progress comfortably.

**Acceptance Criteria:**

- Timer displays elapsed seconds accurately (±100ms)
- Timer updates every 1 second
- No visible flicker when terminal is at full height
- All timer-related information displays correctly (elapsed time, tokens/second, context percentage)

**Current Behavior:**
Timer updates every second → state change → Ink detects content at terminal height → full screen clear + repaint → visible flicker

**Expected Behavior:**
Timer updates only when displayed value changes → minimal re-renders → no flicker visible to user

---

### FR-2: Responsive Input Field

**Priority:** HIGH
**Description:** The UserInput component remains responsive to keystrokes without causing terminal flicker or perceivable lag.

**User Story:**
As a user typing a message, I want my input to appear immediately without causing the screen to flicker, so I can compose messages efficiently.

**Acceptance Criteria:**

- Input field responds to keystrokes within 50ms
- No visible flicker when typing at any speed
- All input features work correctly (history navigation, command completion, multi-line input)
- Input display updates smoothly

**Current Behavior:**
Each keystroke → state update → potential parent re-render → flicker when at full height

**Expected Behavior:**
Input component fully memoized → minimal re-renders → isolated state updates → no flicker

---

### FR-3: Dynamic Layout Adaptation (Optional)

**Priority:** MEDIUM
**Description:** The application layout adapts to terminal size changes and prevents content from reaching the flicker threshold.

**User Story:**
As a user resizing my terminal window, I want the application layout to adapt smoothly without breaking or flickering, so I can work in various window configurations.

**Acceptance Criteria:**

- Layout adjusts when terminal is resized
- Content area never exceeds terminal height minus reserved space
- Resize operations don't cause layout shift or flicker
- Minimum terminal size supported (10 rows)

**Current Behavior:**
Layout uses flexGrow pattern → can expand to full terminal height → triggers flicker threshold

**Expected Behavior:**
Layout uses explicit height constraints → maintains buffer space → prevents reaching flicker threshold

---

### FR-4: Maintain Existing Functionality

**Priority:** CRITICAL
**Description:** All existing features continue to work identically after the fix.

**Acceptance Criteria:**

- Timer displays all metrics correctly
- Input field supports all keyboard shortcuts
- History navigation works as expected
- Command completion functions properly
- Development mode toggle works
- All visual elements display correctly

---

## 3. Technical Requirements

### Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                    App Component                     │
│  - Global state management                          │
│  - Theme and UI context providers                   │
│  - Memoized handlers and callbacks                  │
└─────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┴─────────────────┐
        │                                   │
┌───────▼──────────┐               ┌───────▼──────────┐
│  ChatQueue       │               │ Bottom Controls  │
│  (flexGrow=1)    │               │ (fixed height)   │
│  - Messages      │               │                  │
│  - WelcomeMsg    │               │                  │
└──────────────────┘               │                  │
                                   ├──────────────────┤
                                   │ ThinkingIndicator│
                                   │  * Timer (1s)    │ ← Optimization Target
                                   │  * Word rotation │
                                   ├──────────────────┤
                                   │  UserInput       │ ← Memoization Target
                                   │  * TextInput     │
                                   │  * History       │
                                   └──────────────────┘
```

### Component Design

#### Component 1: ThinkingIndicator (Optimized)

**Location:** `source/components/thinking-indicator.tsx`

**Responsibilities:**

- Display elapsed time with seconds counter
- Rotate through "thinking" words periodically
- Show token usage statistics
- Display context percentage

**State Management:**

```typescript
// Current state
const [elapsedSeconds, setElapsedSeconds] = useState(0);
const [wordIndex, setWordIndex] = useState(0);
const startTimeRef = useRef<number>(Date.now());

// Optimization: functional state updates
useEffect(() => {
	const timer = setInterval(() => {
		const elapsed = Math.floor((Date.now() - startTimeRef.current) / 1000);
		setElapsedSeconds(prev => (prev !== elapsed ? elapsed : prev)); // Only update if changed
	}, 1000);
	return () => clearInterval(timer);
}, []);
```

**Key Optimizations:**

1. Functional state updates to prevent redundant renders
2. Memoized computed values (percentage, dots, tokensPerSecond)
3. Two separate timers with different frequencies (1s for time, 3s for words)

**Props Interface:**

```typescript
interface ThinkingIndicatorProps {
	contextSize: number;
	totalTokensUsed: number;
	tokensPerSecond?: number;
}
```

---

#### Component 2: UserInput (Memoized)

**Location:** `source/components/user-input.tsx`

**Responsibilities:**

- Text input with multi-line support
- Command history navigation (up/down arrows)
- Command completion with Tab
- Bash mode detection (input starts with !)
- Development mode display
- Large content collapse/expand

**Memoization Strategy:**

```typescript
export default memo(function UserInput({ ... }: ChatProps) {
  // Component implementation
});

// Parent callbacks must be memoized:
const handleMessageSubmit = useCallback(async (message: string) => {
  // Implementation
}, [dependencies]);

const handleCancel = useCallback(() => {
  // Implementation
}, [dependencies]);

const handleToggleDevelopmentMode = useCallback(() => {
  // Implementation
}, [dependencies]);
```

**Props Interface:**

```typescript
interface ChatProps {
	onSubmit?: (message: string) => void;
	placeholder?: string;
	customCommands?: string[];
	disabled?: boolean;
	onCancel?: () => void;
	onToggleMode?: () => void;
	developmentMode?: DevelopmentMode;
}
```

**State Hook:** `useInputState()`

- Manages input value, history, large content detection
- Already optimized with debouncing (50ms)
- Returns memoized object to prevent unnecessary re-renders

---

#### Component 3: App Layout (Enhanced)

**Location:** `source/app.tsx`

**Current Layout:**

```tsx
<Box flexDirection="column" padding={1} width="100%">
	<Box flexGrow={1} flexDirection="column" minHeight={0}>
		{/* Scrollable content area */}
	</Box>
	<Box flexDirection="column">{/* Fixed bottom controls */}</Box>
</Box>
```

**Enhanced Layout (Optional - Phase 3):**

```tsx
const terminalHeight = useTerminalHeight();
const reservedRows = 8; // For bottom controls
const maxChatHeight = Math.max(10, terminalHeight - reservedRows);

<Box flexDirection="column" padding={1} width="100%">
	<Box flexDirection="column" height={maxChatHeight} overflow="hidden">
		{/* Constrained scrollable content */}
	</Box>
	<Box flexDirection="column">{/* Fixed bottom controls */}</Box>
</Box>;
```

---

### Data Models and Schemas

#### ThinkingStats Interface

```typescript
interface ThinkingStats {
	tokenCount: number;
	contextSize: number;
	totalTokensUsed: number;
	tokensPerSecond?: number;
}
```

**Usage:** Passed from parent state to ThinkingIndicator as props

**Source:** `source/app/hooks/useAppState.tsx:11-16`

---

#### InputState Interface

```typescript
interface InputState {
	input: string;
	hasLargeContent: boolean;
	originalInput: string;
	historyIndex: number;
	setInput: (value: string) => void;
	setHasLargeContent: (value: boolean) => void;
	setOriginalInput: (value: string) => void;
	setHistoryIndex: (value: number) => void;
	updateInput: (value: string) => void;
	resetInput: () => void;
	cachedLineCount: number;
}
```

**Usage:** Returned from `useInputState()` hook

**Source:** `source/hooks/useInputState.ts`

---

### State Management Approach

#### Global App State

**Hook:** `useAppState()` in `source/app/hooks/useAppState.tsx`

**Relevant State:**

- `isThinking: boolean` - Controls ThinkingIndicator visibility
- `thinkingStats: ThinkingStats` - Data for ThinkingIndicator
- `developmentMode: DevelopmentMode` - Passed to UserInput

**Optimization:**

- Already uses memoized callbacks
- State updates are isolated to specific branches
- No changes needed to app state management

#### Local Component State

**ThinkingIndicator:**

- `elapsedSeconds` - Timer display value
- `wordIndex` - Current "thinking" word
- `startTimeRef` - Reference for time calculation

**UserInput:**

- Delegates to `useInputState()` hook
- Additional local state for UI flags (textInputKey)

#### Context State

**ThemeContext:** Provides theme colors (stable, doesn't change frequently)
**UIStateProvider:** Provides UI flags (completions, messages, etc.)

Both contexts are stable and don't contribute to flicker issue.

---

## 4. Non-Functional Requirements

### NFR-1: Performance

**Requirement:** Reduce re-render frequency from ~1/second to < 0.1/second when idle

**Metrics:**

- Timer component renders: ~1/second → only when seconds change
- Input component renders: only on actual input changes
- Parent component renders: no impact from child timers

**Implementation:**

- Functional state updates in timer
- React.memo() wrapper on UserInput
- Memoized callbacks in parent

---

### NFR-2: Response Time

**Requirement:** Input field must respond within 50ms of keystroke

**Current Performance:** ~20-30ms (acceptable)

**Post-Optimization:** Maintain current performance or improve

**Implementation:**

- Memoization prevents unnecessary parent re-renders
- Input state updates remain local
- No additional debouncing added

---

### NFR-3: Visual Quality

**Requirement:** No visible flicker detectable by human eye (< 16ms frame time)

**Definition of Flicker:**

- Full screen clear and redraw
- Visible as brief black flash or content jump
- Occurs when Ink uses alternate screen buffer mode

**Target:** Zero flicker events when terminal at full height

---

### NFR-4: Memory Usage

**Requirement:** No memory leaks from timer intervals or event listeners

**Implementation:**

- All timers cleaned up in useEffect cleanup functions
- Event listeners properly removed
- No accumulated state or closures

---

### NFR-5: Compatibility

**Terminal Emulators:**

- iTerm2 (macOS) - primary
- Terminal.app (macOS)
- Windows Terminal
- GNOME Terminal (Linux)
- Konsole (Linux)

**Terminal Sizes:**

- Minimum: 10 rows × 80 columns
- Maximum: Unlimited
- Typical: 24-50 rows × 80-120 columns

**Node.js Versions:**

- Minimum: 16.x
- Recommended: 18.x or later
- Maximum: Latest LTS

---

### NFR-6: Maintainability

**Requirement:** Code follows existing patterns and is well-documented

**Standards:**

- TypeScript strict mode
- Functional components with hooks
- Comprehensive type definitions
- Memoization patterns consistent with codebase
- Comments explaining optimization rationale

---

## 5. Technical Design

### Type Definitions

#### Timer-Related Types

```typescript
// Props for ThinkingIndicator
export interface ThinkingIndicatorProps {
	contextSize: number;
	totalTokensUsed: number;
	tokensPerSecond?: number;
}

// Internal timer state (not exported)
type TimerState = {
	elapsedSeconds: number;
	wordIndex: number;
};
```

#### Input-Related Types

```typescript
// Props for UserInput
interface ChatProps {
	onSubmit?: (message: string) => void;
	placeholder?: string;
	customCommands?: string[];
	disabled?: boolean;
	onCancel?: () => void;
	onToggleMode?: () => void;
	developmentMode?: DevelopmentMode;
}

// Development mode enum
export type DevelopmentMode = 'normal' | 'auto-accept' | 'plan';

// Command completion type
export type Completion = {
	name: string;
	isCustom: boolean;
};
```

#### Hook Types

```typescript
// useTerminalHeight hook (new - Phase 3)
export function useTerminalHeight(): number;

// useInputState hook (existing)
export function useInputState(): {
	input: string;
	hasLargeContent: boolean;
	originalInput: string;
	historyIndex: number;
	updateInput: (value: string) => void;
	resetInput: () => void;
	cachedLineCount: number;
	// ... other methods
};
```

---

### Module Organization

```
source/
├── components/
│   ├── thinking-indicator.tsx         ← Phase 1 optimization
│   ├── user-input.tsx                 ← Phase 2 memoization
│   └── ...other components
├── hooks/
│   ├── useInputState.ts               ← Already optimized
│   ├── useTerminalWidth.tsx           ← Pattern for new hook
│   ├── useTerminalHeight.tsx          ← New hook (Phase 3)
│   └── ...other hooks
├── app/
│   └── hooks/
│       ├── useAppState.tsx            ← No changes needed
│       └── ...other app hooks
├── app.tsx                            ← Phase 3 layout changes
└── types/
    └── index.ts                       ← Type definitions
```

---

### Key Abstractions

#### Abstraction 1: Optimized Timer Updates

**Pattern:** Functional state updates with value comparison

**Implementation:**

```typescript
// Anti-pattern (causes unnecessary re-renders)
setElapsedSeconds(newValue);

// Optimized pattern (React skips re-render if value unchanged)
setElapsedSeconds(prev => (prev !== newValue ? newValue : prev));
```

**Rationale:**

- React's state setter accepts a function
- When function returns same value as previous state, React skips re-render
- This is the core optimization preventing timer flicker

---

#### Abstraction 2: Component Memoization

**Pattern:** React.memo with stable props

**Implementation:**

```typescript
// Component wrapped in memo
export default memo(function UserInput(props: ChatProps) {
	// Implementation
});

// Stable callbacks in parent
const stableCallback = useCallback(() => {
	// Implementation
}, [dependencies]);

// Usage
<UserInput onSubmit={stableCallback} />;
```

**Rationale:**

- React.memo prevents re-render when props haven't changed
- Requires parent to provide stable props (via useCallback/useMemo)
- Breaks parent-child re-render cascade

---

#### Abstraction 3: Computed Value Memoization

**Pattern:** useMemo for derived values

**Implementation:**

```typescript
// Without memoization (recomputes on every render)
const percentage = Math.round((totalTokensUsed / contextSize) * 100);

// With memoization (recomputes only when inputs change)
const percentage = useMemo(
	() => Math.round((totalTokensUsed / contextSize) * 100),
	[totalTokensUsed, contextSize],
);
```

**Rationale:**

- Prevents unnecessary recalculation
- Reduces render work
- Particularly important for components that render frequently

---

#### Abstraction 4: Terminal Dimension Tracking (Phase 3)

**Pattern:** Hook for terminal properties with change detection

**Implementation:**

```typescript
export const useTerminalHeight = () => {
	const [height, setHeight] = useState(process.stdout.rows ?? 24);

	useEffect(() => {
		const handleResize = () => {
			const newHeight = process.stdout.rows ?? 24;
			// Only update state if value changed (prevents unnecessary re-renders)
			setHeight(prev => (prev !== newHeight ? newHeight : prev));
		};

		process.stdout.on('resize', handleResize);
		return () => process.stdout.off('resize', handleResize);
	}, []);

	return height;
};
```

**Rationale:**

- Mirrors existing `useTerminalWidth` pattern
- Change detection prevents re-renders when size doesn't actually change
- Provides reactive value that updates on terminal resize

---

## 6. Data Flow

### Timer Update Flow

```
Component Mount
      │
      ▼
Initialize State
  - elapsedSeconds = 0
  - startTimeRef = Date.now()
      │
      ▼
Start Interval Timer (1000ms)
      │
      ▼
Timer Tick (every 1000ms)
      │
      ├─► Calculate elapsed = floor((now - start) / 1000)
      │
      ├─► Call setElapsedSeconds(prev => prev !== elapsed ? elapsed : prev)
      │
      ├─► React Compares Values
      │   │
      │   ├─► Values Different?
      │   │   └─► YES → Update State → Re-render Component
      │   │
      │   └─► Values Same?
      │       └─► NO → Skip Re-render ✓ (Optimization Success)
      │
      └─► Loop continues...
```

**Key Insight:** Most timer ticks (within same second) result in no state change = no re-render

---

### Input Change Flow

```
User Keystroke
      │
      ▼
TextInput onChange Event
      │
      ▼
updateInput(newValue)
      │  (from useInputState hook)
      │
      ├─► setInput(newValue)
      │
      ├─► Calculate line count (immediate)
      │   └─► setCachedLineCount(count)
      │
      └─► Debounce hasLargeContent (50ms)
          └─► setHasLargeContent(newValue.length > 150)
      │
      ▼
UserInput Component Re-renders
      │
      ├─► Is Component Memoized?
      │   │
      │   ├─► YES → Check if props changed
      │   │   │
      │   │   ├─► Props Changed? → Re-render Component
      │   │   │
      │   │   └─► Props Same? → Skip Re-render ✓
      │   │
      │   └─► NO → Always re-renders (current behavior)
      │
      ▼
Check Parent Re-render
      │
      └─► Parent only re-renders if callbacks/props change
          └─► With memoization: No parent re-render ✓
```

**Optimization Points:**

1. Local state updates (input, lineCount) don't affect parent
2. Debouncing for hasLargeContent reduces update frequency
3. Memoization prevents unnecessary re-renders from parent changes

---

### Component Interaction Diagram

```
App Component
│
├─► [State] isThinking, thinkingStats
│   │
│   ├─► Condition: isThinking === true?
│   │   │
│   │   └─► YES → Render ThinkingIndicator
│   │       │
│   │       ├─► Props: contextSize, totalTokensUsed, tokensPerSecond
│   │       │
│   │       └─► Internal State: elapsedSeconds, wordIndex
│   │           │
│   │           └─► Timer Updates → Functional State Update → Minimal Re-renders
│   │
│   └─► Always Render UserInput
│       │
│       ├─► Props: callbacks (memoized), disabled, developmentMode
│       │
│       └─► Internal State: input, history, UI flags
│           │
│           └─► Typing Updates → Local State → No Parent Impact
│
└─► Layout Boxes
    │
    ├─► Scrollable Area (flexGrow=1)
    │   └─► ChatQueue, WelcomeMessage
    │
    └─► Fixed Bottom Area
        ├─► ThinkingIndicator (conditional)
        └─► UserInput (always visible)
```

---

### Event Flow: Terminal Resize (Phase 3)

```
Terminal Window Resize
      │
      ▼
process.stdout emits 'resize' event
      │
      ▼
useTerminalHeight listener triggered
      │
      ├─► Read new height: process.stdout.rows
      │
      ├─► Compare with previous height
      │   │
      │   ├─► Different? → Update state → Re-render App
      │   │   │
      │   │   └─► Calculate new maxChatHeight
      │   │       └─► Re-layout with new constraints
      │   │
      │   └─► Same? → Skip state update ✓
      │
      └─► Event handled
```

---

## 7. Error Handling

### Error Scenario 1: Timer Interval Leak

**Cause:** Component unmounts before cleanup function runs

**Symptoms:**

- Timer continues running after component removed
- Memory leak accumulates over time
- Console warnings about state updates on unmounted component

**Prevention:**

```typescript
useEffect(() => {
	const timer = setInterval(() => {
		// Timer logic
	}, 1000);

	// Cleanup function ALWAYS runs on unmount
	return () => {
		clearInterval(timer);
	};
}, []);
```

**Detection:** Monitor for warnings in console during development

---

### Error Scenario 2: Invalid Terminal Dimensions

**Cause:** Terminal emulator doesn't provide `process.stdout.rows` or provides invalid value

**Symptoms:**

- Undefined or NaN height value
- Layout breaks or appears incorrect
- Division by zero in calculations

**Handling:**

```typescript
const useTerminalHeight = () => {
	// Default to 24 if undefined
	const [height, setHeight] = useState(process.stdout.rows ?? 24);

	useEffect(() => {
		const handleResize = () => {
			const newHeight = process.stdout.rows ?? 24;
			// Validate: ensure positive integer
			if (newHeight > 0) {
				setHeight(prev => (prev !== newHeight ? newHeight : prev));
			}
		};
		// ...
	}, []);

	return height;
};

// In layout calculation
const maxChatHeight = Math.max(10, terminalHeight - reservedRows);
// Math.max ensures minimum of 10 rows even if calculation results in < 10
```

---

### Error Scenario 3: Rapid State Updates

**Cause:** Multiple state updates triggered simultaneously

**Symptoms:**

- Jittery UI
- Inconsistent display
- Performance degradation

**Handling:**

- React automatically batches state updates in event handlers
- For timer updates, functional state update ensures consistency
- Debouncing for non-critical updates (hasLargeContent)

```typescript
// Multiple state updates batched by React (in event handler)
const handleSubmit = () => {
	setInput(''); // Batched
	setHasLargeContent(false); // Batched
	setHistoryIndex(-1); // Batched
	// All applied in single re-render
};
```

---

### Validation Strategies

#### Input Validation

**Location:** `useInputState` hook

**Validations:**

- Input length: No hard limit, but large content detection at > 150 chars
- Line count: Calculated and cached, minimum 1
- History index: Bounds checked in navigation logic (-2 to history.length - 1)

**Implementation:** Already robust, no changes needed

---

#### Timer Value Validation

**Location:** `ThinkingIndicator` component

**Validations:**

```typescript
// Ensure percentage doesn't exceed 100%
const displayPercentage = Math.min(percentage, 100);

// Ensure tokensPerSecond is valid number
const tokensPerSecondDisplay =
	tokensPerSecond !== undefined && tokensPerSecond > 0
		? ` • ${tokensPerSecond} tok/s`
		: '';

// Ensure dots cycle correctly (1-4 dots)
const dots = '.'.repeat((elapsedSeconds % 4) + 1);
```

---

### Recovery Mechanisms

#### Recovery 1: Component Remount

**Trigger:** User can cancel operation (Escape key)

**Effect:**

- ThinkingIndicator unmounts
- Timer cleanup function runs
- All intervals cleared
- State reset on next mount

**Implementation:** Already handled correctly

---

#### Recovery 2: State Reset

**Trigger:** User submits new message

**Effect:**

- Input state cleared
- History index reset
- UI flags cleared
- Clean slate for next interaction

**Implementation:**

```typescript
const resetInput = useCallback(() => {
	// Clear pending timers
	if (debounceTimerRef.current) {
		clearTimeout(debounceTimerRef.current);
	}
	// Reset all state
	setInput('');
	setHasLargeContent(false);
	setOriginalInput('');
	setHistoryIndex(-1);
	setCachedLineCount(1);
}, []);
```

---

### User Feedback Patterns

#### Feedback 1: Timer Display

**Element:** Elapsed seconds counter

**Format:** `{thinkingWord}{dots} {seconds}s • {tokensPerSecond} tok/s • {percentage}% context used`

**Update Frequency:** Every 1 second (when seconds value changes)

**Examples:**

- `Thinking... 5s • 42 tok/s • 15% context used`
- `Processing... 10s • 38 tok/s • 25% context used`

---

#### Feedback 2: Input State

**Element:** Input field display

**States:**

- Normal: Full input text visible with cursor
- Disabled: Shows "..." when AI is processing
- Large content: Shows `[{chars} characters, {lines} lines] (Ctrl+B to expand)`
- Bash mode: Bordered box with "Bash Mode" label

---

#### Feedback 3: Development Mode

**Element:** Status line below input

**Format:** `{mode_label} (Shift+Tab to cycle)`

**Modes:**

- Normal Mode (gray)
- Auto-Accept Mode (blue)
- Plan Mode (yellow)

---

## 8. Configuration & Environment

### Environment Variables

**None required for this feature**

The application uses existing environment configuration:

- LLM provider settings (API keys, endpoints)
- Theme preferences
- MCP server configuration

Flicker fix works within current environment setup.

---

### Configuration Files

**None modified**

Existing configuration:

- `package.json` - Dependencies remain unchanged
- `tsconfig.json` - TypeScript configuration unchanged
- `.claude/` directory - Custom commands and themes unaffected

---

### External Service Dependencies

**None**

This is a pure UI optimization:

- No API calls involved
- No external services required
- Works entirely within terminal rendering layer

---

### Development Setup Requirements

**Prerequisites:**

1. Node.js >= 16
2. npm or yarn
3. TypeScript 5.0
4. Terminal emulator with color support

**Development Commands:**

```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Run in development mode
npm run dev

# Run tests
npm test

# Type checking
npm run type-check
```

**Testing Setup:**

- Ava test framework (already configured)
- Ink testing library (for component tests)
- Manual terminal testing required (visual verification)

---

## 9. Migration & Deployment

### Deployment Strategy

**Type:** Rolling update (no breaking changes)

**Steps:**

1. Merge changes to main branch
2. Create new version tag (e.g., v1.2.3)
3. Publish to npm registry
4. Users update via `npm install -g nanocoder@latest`

**Zero-downtime:** N/A (CLI tool, users update at their convenience)

---

### Migration Steps

**No migration required**

This is an optimization fix, not a feature change:

- No database migrations
- No configuration changes
- No user data affected
- No breaking API changes

**User Action Required:** None (update applies automatically on next install)

---

### Rollback Procedures

**If Critical Issues Detected:**

1. **Immediate Rollback:**

   ```bash
   git revert [commit-hash]
   git push
   npm publish --tag rollback
   ```

2. **User Notification:**

   - GitHub issue explaining rollback
   - npm deprecation notice on affected version
   - Recommendation to downgrade: `npm install -g nanocoder@[previous-version]`

3. **Investigation:**

   - Collect user reports and terminal environment details
   - Reproduce in various terminal emulators
   - Identify specific failure case

4. **Fix and Redeploy:**
   - Address root cause
   - Test thoroughly
   - Deploy with incremental rollout

**Risk Level:** LOW - Changes are localized and non-breaking

---

### Monitoring and Observability

**Key Metrics to Monitor:**

1. **User Reports:**

   - GitHub issues mentioning "flicker" or "performance"
   - User feedback on CLI responsiveness
   - Terminal compatibility reports

2. **Performance Metrics (Development):**

   - Component render count (add temporary logging)
   - Timer accuracy (compare displayed vs actual elapsed)
   - Input latency (measure keystroke to display time)

3. **Compatibility:**
   - Terminal emulator reports (iTerm2, Terminal.app, Windows Terminal, etc.)
   - Node.js version compatibility
   - Operating system variations

**Monitoring Tools:**

- GitHub issue tracker (primary feedback channel)
- npm download statistics
- Community Discord/Slack (if applicable)

**No runtime telemetry:** CLI tool doesn't phone home, all monitoring via user reports

---

## 10. Future Considerations

### Extensibility Points

#### Extension 1: Configurable Timer Update Frequency

**Rationale:** Some users may prefer slower updates to reduce CPU usage

**Implementation:**

```typescript
// Configuration option (future)
interface ThinkingIndicatorConfig {
	updateFrequencyMs: number; // Default: 1000
}

// Usage
const timer = setInterval(() => {
	// Timer logic
}, config.updateFrequencyMs);
```

**Location:** Could be added to user preferences or command-line flag

---

#### Extension 2: Pluggable Timer Display Components

**Rationale:** Allow custom timer visualizations (spinner only, progress bar, etc.)

**Implementation:**

```typescript
// Future API
type TimerDisplayType = 'default' | 'minimal' | 'detailed' | 'custom';

interface ThinkingIndicatorProps {
	// ... existing props
	displayType?: TimerDisplayType;
	customRenderer?: (stats: ThinkingStats) => React.ReactNode;
}
```

---

#### Extension 3: Performance Profiling Mode

**Rationale:** Help developers diagnose performance issues

**Implementation:**

```typescript
// Development mode flag
if (process.env.NANOCODER_PROFILE === 'true') {
	useEffect(() => {
		renderCount.current += 1;
		console.log(`[PROFILE] ${componentName} render #${renderCount.current}`);
	});
}
```

**Usage:** `NANOCODER_PROFILE=true nanocoder`

---

### Potential Enhancements

#### Enhancement 1: Adaptive Timer Frequency

**Description:** Automatically adjust timer update frequency based on terminal performance

**Benefit:** Better performance on low-end systems or slow terminals

**Complexity:** Medium

**Implementation Sketch:**

```typescript
const [updateFrequency, setUpdateFrequency] = useState(1000);

useEffect(() => {
	// Monitor render time
	const renderStart = performance.now();
	// ... render logic
	const renderTime = performance.now() - renderStart;

	// If renders take > 50ms, slow down updates
	if (renderTime > 50) {
		setUpdateFrequency(2000); // Update every 2s instead
	}
}, []);
```

---

#### Enhancement 2: Virtual Scrolling for Long Conversations

**Description:** Render only visible messages, virtualize off-screen content

**Benefit:** Massive performance improvement for very long conversations

**Complexity:** High

**Note:** Already partially implemented (displayCount = 5 in ChatQueue)

**Future Work:** Implement full windowing with dynamic height calculation

---

#### Enhancement 3: GPU-Accelerated Terminal Rendering

**Description:** Use GPU-accelerated terminal emulators (Alacritty, Kitty)

**Benefit:** Hardware-accelerated rendering may eliminate flicker at source

**Complexity:** Low (no code changes)

**Note:** Already works with modern terminals, just recommend to users

---

### Technical Debt Considerations

#### Debt 1: Multiple Timer Intervals

**Current State:** Two separate intervals (1s timer, 3s word rotation)

**Concern:** Could be consolidated into single interval with modulo logic

**Mitigation Strategy:**

```typescript
// Future optimization
useEffect(() => {
	const timer = setInterval(() => {
		const now = Date.now();
		const elapsed = Math.floor((now - startTimeRef.current) / 1000);

		// Update seconds every 1s
		setElapsedSeconds(prev => (prev !== elapsed ? elapsed : prev));

		// Update word every 3s
		if (elapsed % 3 === 0) {
			setWordIndex(Math.floor(Math.random() * THINKING_WORDS.length));
		}
	}, 1000);

	return () => clearInterval(timer);
}, []);
```

**Priority:** LOW - current implementation works fine

---

#### Debt 2: Ink Framework Dependency

**Current State:** Locked to Ink 6.3.1

**Concern:** Ink has known flicker issues with alternate screen buffer

**Future Options:**

1. Stay on Ink - continue optimizing around limitations
2. Fork Ink - patch rendering behavior for our use case
3. Alternative framework - explore other CLI UI libraries (Blessed, etc.)

**Recommendation:** Stay on Ink for now, monitor for upstream fixes

---

#### Debt 3: Test Coverage Gaps

**Current State:** Limited tests for render performance and visual behavior

**Concern:** Can't automatically detect regressions in flicker behavior

**Future Work:**

- Add integration tests for component render counts
- Implement snapshot testing for layout
- Create automated visual regression tests (difficult for CLI)

**Priority:** MEDIUM - manual testing currently sufficient but automation would help

---

### Maintenance Requirements

#### Routine Maintenance

1. **Monitor Ink Updates:** Check for upstream rendering improvements
2. **Test on New Terminals:** Verify compatibility as new emulators released
3. **Performance Review:** Periodically audit render performance
4. **User Feedback:** Track reports of flicker or performance issues

#### Long-term Maintenance

1. **React Updates:** Test compatibility as React versions evolve
2. **Node.js Updates:** Ensure timer behavior consistent across Node versions
3. **TypeScript Updates:** Keep type definitions current
4. **Dependency Updates:** Regular updates to Ink and related libraries

---

## Appendix A: Key File Locations

```
source/
├── components/
│   ├── thinking-indicator.tsx         Lines: 1-109   | Primary optimization target
│   ├── user-input.tsx                 Lines: 1-344   | Memoization target
│   ├── chat-queue.tsx                 Lines: 1-38    | Already optimized
│   └── cancelling-indicator.tsx       Related component
│
├── hooks/
│   ├── useInputState.ts               Lines: 1-78    | Already optimized
│   ├── useTerminalWidth.tsx           Lines: 1-28    | Pattern reference
│   ├── useTerminalHeight.tsx          NEW FILE       | Phase 3 addition
│   └── useTheme.js                    Provides colors context
│
├── app/
│   └── hooks/
│       ├── useAppState.tsx            Lines: 1-230   | Global state management
│       ├── useChatHandler.ts          Message handling
│       └── useModeHandlers.ts         Mode switching
│
├── app.tsx                            Lines: 1-350   | Main app component
└── types/
    └── index.ts                       Type definitions
```

---

## Appendix B: Related Git Commits

**Previous Flicker Fixes:**

1. **Commit 81f6d0b** (2025-08-24) - "fix: jittering"

   - Moved timer state into ThinkingIndicator
   - Added flexGrow layout pattern
   - Separated scrollable from fixed content

2. **Commit 453c63d** (2025-08-24) - "fix: massive rerender problem causing scroll issues"

   - Added memoization to ThinkingIndicator
   - Optimized useChatHandler

3. **Commit df6038c** (2025-08-28) - "fix: more aggressive memoisation throughout"
   - Wrapped multiple components in React.memo()
   - Optimized useTerminalWidth

**Current Fix:** Builds on all previous work with functional state updates

---

## Appendix C: Testing Checklist

### Manual Testing Checklist

#### Phase 1 Testing (Timer Optimization)

- [ ] Timer displays correctly (elapsed seconds)
- [ ] Timer updates every second
- [ ] No visible flicker with terminal at full height
- [ ] Timer accuracy within ±100ms of actual elapsed time
- [ ] Word rotation works (changes every 3 seconds)
- [ ] Tokens/second displays correctly
- [ ] Context percentage displays correctly
- [ ] Percentage clamped at 100% (no jitter from > 100% values)

#### Phase 2 Testing (Input Memoization)

- [ ] Input field responds to keystrokes immediately
- [ ] No visible flicker while typing rapidly
- [ ] History navigation works (up/down arrows)
- [ ] Command completion works (Tab key)
- [ ] Multi-line input works (Shift+Enter)
- [ ] Large content collapse/expand works (Ctrl+B)
- [ ] Bash mode detection works (input starts with !)
- [ ] Development mode toggle works (Shift+Tab)

#### Phase 3 Testing (Dynamic Layout)

- [ ] Layout adapts to terminal resize
- [ ] No layout shift or jump during resize
- [ ] Minimum terminal size supported (10 rows)
- [ ] Maximum content height respects constraints
- [ ] Scrolling works correctly in constrained area
- [ ] Fixed bottom controls remain visible

#### Integration Testing

- [ ] Timer + typing simultaneously: no flicker
- [ ] Timer + terminal resize: no flicker
- [ ] Long conversation (> 10 messages) + timer: no flicker
- [ ] Multiple rapid state changes: stable behavior

#### Compatibility Testing

- [ ] iTerm2 (macOS)
- [ ] Terminal.app (macOS)
- [ ] Windows Terminal
- [ ] GNOME Terminal (Linux)
- [ ] Konsole (Linux)

---

## Appendix D: Performance Metrics

### Baseline Metrics (Before Optimization)

**ThinkingIndicator:**

- Render frequency: ~1 per second (consistent)
- Re-renders per minute: ~60
- Flicker events: 1-60 per minute (depends on terminal height)

**UserInput:**

- Render frequency: 1 per keystroke
- Re-renders per minute: Variable (depends on typing speed)
- Flicker contribution: Minimal (unless terminal at capacity)

**Overall:**

- Total re-renders per minute: 60+ (timer) + typing events
- Flicker visibility: HIGH when terminal at full height

---

### Target Metrics (After Optimization)

**ThinkingIndicator:**

- Render frequency: ~1 per second (only when value changes)
- Re-renders per minute: ~60 (same, but no redundant renders within same second)
- Flicker events: 0 (functional state update prevents Ink re-paint)

**UserInput:**

- Render frequency: 1 per keystroke (isolated to component)
- Re-renders per minute: Variable (same)
- Flicker contribution: ZERO (fully memoized)

**Overall:**

- Total re-renders per minute: ~60 (timer) + isolated typing events
- Flicker visibility: ZERO when terminal at full height

---

### Measurement Tools

**Development Logging:**

```typescript
// Add to component during testing
const renderCount = useRef(0);
renderCount.current += 1;

if (process.env.NODE_ENV === 'development') {
	console.log(
		`[RENDER] ThinkingIndicator #${renderCount.current} at ${Date.now()}`,
	);
}
```

**React DevTools Profiler:**

- Not directly applicable to CLI (designed for browser)
- Could potentially adapt profiling hooks for CLI use

**Manual Observation:**

- Visual inspection in full-height terminal
- Video recording for frame-by-frame analysis
- User reports and feedback

---

## Appendix E: Glossary

**Alternate Screen Buffer:** Terminal mode where app takes over entire screen, clearing it on exit. Ink uses this when content height ≥ terminal height.

**Flicker:** Visible flash or jump in terminal display caused by full screen clear and repaint. Occurs when Ink re-renders in alternate screen buffer mode.

**Functional State Update:** React pattern where state setter receives a function: `setState(prev => newValue)`. React can optimize by comparing return value with previous state.

**Memoization:** React optimization technique using `React.memo()`, `useMemo()`, or `useCallback()` to prevent unnecessary re-renders or recalculations.

**Re-render:** React lifecycle phase where component function runs again, potentially causing DOM (or terminal) updates. Triggered by state or prop changes.

**Terminal Height:** Number of rows available in terminal window, accessible via `process.stdout.rows` in Node.js.

**Thinking Indicator:** Component displayed while AI is processing, showing elapsed time and token usage statistics.

**Timer Interval:** JavaScript `setInterval()` function that executes callback repeatedly at specified frequency.

---

**END OF PROJECT SPECIFICATION**

This document provides a comprehensive technical specification for the terminal flicker fix. For implementation details and phased approach, see IMPLEMENTATION_PLAN.md.

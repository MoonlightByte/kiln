---
name: audit-swiftui
description: Audit SwiftUI code for Human Interface Guidelines compliance
args:
  - name: code
    type: string
    description: Swift/SwiftUI code to audit
    required: true
  - name: focus
    type: string
    enum: [state, a11y, performance, styling, motion, all]
    description: Specific area to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Human Interface Guidelines Audit for SwiftUI

Perform a comprehensive audit of the provided SwiftUI code against Apple Human Interface Guidelines.

## Instructions

Analyze the code for compliance with HIG specifications. Reference the `ios-design` skill for detailed guidelines.

## Audit Checklist

### State Management (Critical Priority)
{{#if (or (eq focus "state") (eq focus "all") (not focus))}}
- [ ] **AP-001**: Using `@StateObject` for owned objects, not `@ObservedObject`
- [ ] **AP-002**: Not using `@State` with reference types (use `@StateObject` or `@Observable`)
- [ ] **AP-003**: All `@Published` properties in `ObservableObject`
- [ ] **AP-004**: Initializing `@State` correctly with `_state = State(initialValue:)`
- [ ] Correct use of `@Binding` for child views
- [ ] `@EnvironmentObject` used for dependency injection
{{/if}}

### Accessibility (Critical Priority)
{{#if (or (eq focus "a11y") (eq focus "all") (not focus))}}
- [ ] **AP-010**: All interactive elements have `.accessibilityLabel`
- [ ] **AP-011**: All interactive elements are 44x44pt minimum
- [ ] **AP-012**: Decorative images marked `.accessibilityHidden(true)`
- [ ] Text contrast is 4.5:1 for normal text, 3:1 for large text
- [ ] Using Dynamic Type (`.font(.body)` not `.font(.system(size:))`)
- [ ] Focus indicators are visible
- [ ] VoiceOver reads in logical order
{{/if}}

### Performance (High Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **AP-005**: Not overusing `AnyView` (use `@ViewBuilder` instead)
- [ ] **AP-006**: Using `LazyVStack`/`List` for long lists, not `VStack`
- [ ] **AP-007**: Providing stable IDs in `ForEach` (`.id(\.self)` or explicit ID)
- [ ] **AP-008**: Heavy computation moved to ViewModel, not in view body
- [ ] **AP-009**: Using `.task` modifier for async work, not `onAppear` with `Task`
- [ ] Views are small and focused (not monolithic)
{{/if}}

### Styling (Important Priority)
{{#if (or (eq focus "styling") (eq focus "all") (not focus))}}
- [ ] **AP-013**: Using semantic colors (`.label`, `.systemBackground`) not hardcoded
- [ ] **AP-014**: Using text styles (`.title`, `.body`) not hardcoded font sizes
- [ ] **AP-015**: Correct modifier order (padding before background, frame before overlay)
- [ ] Respecting Safe Areas or consciously ignoring with `.ignoresSafeArea()`
- [ ] Following 4pt spacing grid
{{/if}}

### Motion (Medium Priority)
{{#if (or (eq focus "motion") (eq focus "all") (not focus))}}
- [ ] Animations respect `.accessibilityReduceMotion`
- [ ] Using `.animation()` modifier or `withAnimation` block
- [ ] Transitions defined with `.transition()`
- [ ] Spring animations use appropriate stiffness/damping
{{/if}}

### Navigation (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **AP-016**: Using value-based navigation or lazy destination closures
- [ ] NavigationStack (iOS 16+) preferred over NavigationView
- [ ] Proper back button behavior
{{/if}}

### Concurrency (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **AP-018**: ViewModel marked `@MainActor` for UI updates
- [ ] **AP-019**: Checking `Task.isCancelled` in long operations
- [ ] Using `async let` for concurrent operations
{{/if}}

### Safety (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **AP-020**: No force unwrapping (`!`) of optionals
- [ ] Using optional binding (`if let`) or nil coalescing (`??`)
{{/if}}

## Code to Audit

```swift
{{code}}
```

## Output Format

Generate a structured audit report:

```markdown
## Audit Results: [File/Component Name]

### Critical Issues (🔴)
List issues that block accessibility or cause crashes.
Each with: Line number, description, severity, fix suggestion.

### Important Issues (🟠)
List issues that violate HIG guidelines or hurt UX.

### Warnings (🟡)
List suboptimal patterns that work but should be improved.

### Suggestions (🔵)
List polish opportunities for design excellence.

### Summary
- Total issues: X
- Critical: X | Important: X | Warning: X | Suggestion: X
- Overall compliance: X%

### Recommended Actions
1. [Most urgent fix]
2. [Second priority]
3. [Third priority]
```

## Example Output

```markdown
## Audit Results: EventDetailView.swift

### Critical Issues (🔴)

**Line 12**: Using @ObservedObject for owned instance
```swift
// BAD
@ObservedObject var viewModel = EventDetailViewModel() // Recreated on every update!

// GOOD
@StateObject private var viewModel = EventDetailViewModel()
```
Severity: 🔴 Critical | Confidence: 95 | Rule: AP-001

---

**Line 45**: Missing accessibilityLabel on button
```swift
// BAD
Button(action: { share() }) {
    Image(systemName: "square.and.arrow.up")
}

// GOOD
Button(action: { share() }) {
    Image(systemName: "square.and.arrow.up")
}
.accessibilityLabel("Share event")
```
Severity: 🔴 Critical | Confidence: 95 | Rule: AP-010

### Important Issues (🟠)

**Line 28**: Hardcoded font size instead of text style
```swift
// BAD
.font(.system(size: 24, weight: .bold))

// GOOD
.font(.title)
```
Severity: 🟠 Important | Confidence: 85 | Rule: AP-014

---

**Line 35**: Using VStack for potentially long list
```swift
// BAD
ScrollView {
    VStack {
        ForEach(attendees) { attendee in
            AttendeeRow(attendee: attendee)
        }
    }
}

// GOOD
ScrollView {
    LazyVStack {
        ForEach(attendees) { attendee in
            AttendeeRow(attendee: attendee)
        }
    }
}
```
Severity: 🟠 Important | Confidence: 80 | Rule: AP-006

### Summary
- Total issues: 4
- Critical: 2 | Important: 2 | Warning: 0 | Suggestion: 0
- Overall compliance: 65%

### Recommended Actions
1. Change @ObservedObject to @StateObject for owned ViewModels
2. Add accessibilityLabel to all interactive elements
3. Replace hardcoded font sizes with text styles
4. Use LazyVStack for lists that may grow
```

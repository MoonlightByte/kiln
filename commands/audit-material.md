---
name: audit-material
description: Audit Jetpack Compose code for Material Design 3 compliance
args:
  - name: code
    type: string
    description: Kotlin/Compose code to audit
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

# Material Design 3 Audit for Jetpack Compose

Perform a comprehensive audit of the provided Compose code against Material Design 3 guidelines.

## Instructions

Analyze the code for compliance with Material Design 3 specifications. Reference the `android-design` skill for detailed guidelines.

## Audit Checklist

### State Management (Critical Priority)
{{#if (or (eq focus "state") (eq focus "all") (not focus))}}
- [ ] **AP-001**: Not passing `MutableState` to children
- [ ] **AP-002**: Not reading scroll state in composition (use `derivedStateOf`)
- [ ] **AP-003**: Not creating state inside loops
- [ ] **AP-004**: Using `rememberSaveable` for user input
- [ ] State hoisting follows unidirectional data flow
- [ ] ViewModels use `StateFlow` or `LiveData`
{{/if}}

### Accessibility (Critical Priority)
{{#if (or (eq focus "a11y") (eq focus "all") (not focus))}}
- [ ] **AP-010**: All icons have `contentDescription`
- [ ] **AP-011**: All interactive elements are 48x48dp minimum
- [ ] **AP-012**: Correct semantic roles (`Role.Button`, `Role.Tab`, etc.)
- [ ] Text contrast is 4.5:1 for normal text, 3:1 for large text
- [ ] Focus indicators are visible
- [ ] Custom actions exposed via `semantics { customActions = ... }`
{{/if}}

### Performance (High Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **AP-005**: Modifiers created outside composable body
- [ ] **AP-006**: Using `ImmutableList` for collections in state
- [ ] **AP-007**: Heavy computation wrapped in `remember`
- [ ] **AP-008**: Using `LazyColumn` for lists > 10 items
- [ ] **AP-009**: Providing stable keys to `LazyColumn` items
- [ ] Data classes marked `@Immutable` or `@Stable`
{{/if}}

### Styling (Important Priority)
{{#if (or (eq focus "styling") (eq focus "all") (not focus))}}
- [ ] **AP-013**: Using `MaterialTheme.colorScheme.*` not hardcoded colors
- [ ] **AP-014**: Using `MaterialTheme.typography.*` not hardcoded fonts
- [ ] **AP-015**: Following 4dp spacing grid (4, 8, 12, 16, 24, 32, 48, 64)
- [ ] Using M3 shape scale (4dp to 28dp)
- [ ] Proper tonal elevation on surfaces
{{/if}}

### Motion (Medium Priority)
{{#if (or (eq focus "motion") (eq focus "all") (not focus))}}
- [ ] Animations use M3 duration scale (50ms to 500ms)
- [ ] Animations use M3 easing curves
- [ ] Animations respect reduced motion preference
- [ ] Enter/exit transitions are defined
{{/if}}

### Side Effects (Critical Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **AP-016**: No API calls in composable body
- [ ] **AP-017**: Using `DisposableEffect` for resources needing cleanup
- [ ] Side effects in `LaunchedEffect`, `SideEffect`, or `DisposableEffect`
{{/if}}

### Security (Critical Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **AP-018**: No hardcoded API keys or secrets
- [ ] **AP-019**: Using `EncryptedSharedPreferences` for sensitive data
{{/if}}

## Code to Audit

```kotlin
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
List issues that violate M3 guidelines or hurt UX.

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
## Audit Results: EventCard.kt

### Critical Issues (🔴)

**Line 15**: Missing contentDescription on favorite icon
```kotlin
// BAD
Icon(imageVector = Icons.Default.Favorite, contentDescription = null)

// GOOD
Icon(
    imageVector = Icons.Default.Favorite,
    contentDescription = if (isFavorite) "Remove from favorites" else "Add to favorites"
)
```
Severity: 🔴 Critical | Confidence: 95 | Rule: AP-010

---

**Line 42**: Touch target too small (24x24dp)
```kotlin
// BAD
Icon(
    imageVector = Icons.Default.Close,
    modifier = Modifier.size(24.dp).clickable { onClose() }
)

// GOOD
IconButton(onClick = onClose) {
    Icon(Icons.Default.Close, contentDescription = "Close")
}
```
Severity: 🔴 Critical | Confidence: 90 | Rule: AP-011

### Important Issues (🟠)

**Line 8**: Hardcoded color instead of theme token
```kotlin
// BAD
Text(color = Color(0xFF1976D2))

// GOOD
Text(color = MaterialTheme.colorScheme.primary)
```
Severity: 🟠 Important | Confidence: 90 | Rule: AP-013

### Summary
- Total issues: 3
- Critical: 2 | Important: 1 | Warning: 0 | Suggestion: 0
- Overall compliance: 70%

### Recommended Actions
1. Add contentDescription to all interactive icons
2. Use IconButton or add .minimumInteractiveComponentSize()
3. Replace hardcoded colors with MaterialTheme tokens
```

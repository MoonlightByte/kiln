---
name: compose-edit-guard
description: Auto-inject Material Design 3 checklist when editing Compose files
event: PreToolUse
match_tools:
  - Edit
  - Write
match_files:
  - "**/*.kt"
---

# Compose Edit Guard

Before editing any Kotlin file containing Compose UI code, verify the following Material Design 3 anti-patterns are NOT being introduced:

## Critical Anti-Patterns (MUST NOT EXIST)

### State Management
- **AP-001**: Do NOT pass `MutableState` to child composables - pass values and callbacks
- **AP-002**: Do NOT read scroll state directly in composition - use `derivedStateOf`
- **AP-003**: Do NOT create `MutableState` in leaf composables - hoist state up
- **AP-004**: Do NOT store mutable collections in state - use `ImmutableList`/`persistentListOf`

### Recomposition Stability
- **AP-005**: Data classes used in state MUST be marked `@Immutable` or `@Stable`
- **AP-006**: LazyColumn/LazyRow MUST have stable `key` parameters - NOT indices
- **AP-007**: Modifier chains MUST be created outside composable body (or use `remember`)
- **AP-008**: Lambda callbacks MUST be stable - use `remember` or method references

### Performance
- **AP-009**: Lists with >10 items MUST use `LazyColumn`/`LazyRow`, not `Column`/`Row`
- **AP-010**: Expensive calculations MUST use `remember` with appropriate keys
- **AP-011**: Derived state MUST use `derivedStateOf` for filtered/transformed data
- **AP-012**: Animations MUST NOT trigger recomposition - use `Animatable` or `updateTransition`

### Accessibility
- **AP-013**: All clickable elements MUST have 48x48dp minimum touch target
- **AP-014**: All interactive elements MUST have `contentDescription`
- **AP-015**: Text contrast ratio MUST be 4.5:1 minimum (use semantic colors)
- **AP-016**: Icons in buttons MUST have meaningful labels for screen readers

### Material Design 3 Compliance
- **AP-017**: Do NOT hardcode colors - use `MaterialTheme.colorScheme.*`
- **AP-018**: Do NOT hardcode text sizes - use `MaterialTheme.typography.*`
- **AP-019**: Do NOT hardcode corner radii - use `MaterialTheme.shapes.*`

## Required Patterns

When creating or modifying Compose UI:

1. **Use semantic tokens**:
   ```kotlin
   color = MaterialTheme.colorScheme.primary  // NOT Color(0xFF...)
   style = MaterialTheme.typography.bodyLarge  // NOT fontSize = 16.sp
   shape = MaterialTheme.shapes.medium  // NOT RoundedCornerShape(12.dp)
   ```

2. **Mark state classes stable**:
   ```kotlin
   @Immutable
   data class UiState(
       val items: ImmutableList<Item> = persistentListOf()
   )
   ```

3. **Provide lazy keys**:
   ```kotlin
   LazyColumn {
       items(items, key = { it.id }) { item ->
           ItemRow(item)
       }
   }
   ```

4. **Ensure touch targets**:
   ```kotlin
   IconButton(
       onClick = { /* action */ },
       modifier = Modifier.minimumInteractiveComponentSize()
   ) {
       Icon(Icons.Default.Close, contentDescription = "Close")
   }
   ```

If any anti-pattern would be introduced by this edit, STOP and fix the code to follow the correct pattern before proceeding.

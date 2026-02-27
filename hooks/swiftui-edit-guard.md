---
name: swiftui-edit-guard
description: Auto-inject Human Interface Guidelines checklist when editing SwiftUI files
event: PreToolUse
match_tools:
  - Edit
  - Write
match_files:
  - "**/*.swift"
---

# SwiftUI Edit Guard

Before editing any Swift file containing SwiftUI code, verify the following Human Interface Guidelines anti-patterns are NOT being introduced:

## Critical Anti-Patterns (MUST NOT EXIST)

### Property Wrappers
- **SP-001**: Do NOT use `@ObservedObject` for owned objects - use `@StateObject`
- **SP-002**: Do NOT recreate ViewModels on view updates - mark with `@StateObject`
- **SP-003**: Do NOT use `@State` for reference types - use `@StateObject`
- **SP-004**: Do NOT mutate `@Binding` directly in initializer

### View Identity
- **SP-005**: ForEach MUST have explicit `id` parameter or use `Identifiable`
- **SP-006**: Do NOT use array indices as ForEach identity
- **SP-007**: Do NOT wrap views in unnecessary AnyView type erasure
- **SP-008**: Conditional views MUST use `@ViewBuilder`, not AnyView

### Performance
- **SP-009**: Lists with >10 items MUST use `List` or `LazyVStack`, not `VStack`
- **SP-010**: Do NOT access environment values inside view body unnecessarily
- **SP-011**: Complex computed properties SHOULD be memoized
- **SP-012**: NavigationLink destinations SHOULD be lazy

### Accessibility
- **SP-013**: All tappable elements MUST have 44x44pt minimum touch target
- **SP-014**: All interactive elements MUST have `accessibilityLabel`
- **SP-015**: Text MUST use system font styles for Dynamic Type support
- **SP-016**: Images MUST have `accessibilityLabel` or be marked decorative

### Human Interface Guidelines Compliance
- **SP-017**: Do NOT hardcode colors - use semantic colors (`.label`, `.systemBackground`)
- **SP-018**: Do NOT hardcode font sizes - use system styles (`.body`, `.title`)
- **SP-019**: Respect safe areas - only use `.ignoresSafeArea()` intentionally
- **SP-020**: Support both light and dark mode - test in both appearances

## Required Patterns

When creating or modifying SwiftUI code:

1. **Use semantic colors**:
   ```swift
   .foregroundColor(.label)  // NOT .foregroundColor(.black)
   .background(Color(.systemBackground))  // NOT .background(.white)
   ```

2. **Use system text styles**:
   ```swift
   .font(.body)  // NOT .font(.system(size: 17))
   .font(.largeTitle)  // NOT .font(.system(size: 34))
   ```

3. **Use @StateObject for owned objects**:
   ```swift
   @StateObject private var viewModel = MyViewModel()  // NOT @ObservedObject
   ```

4. **Provide stable ForEach identity**:
   ```swift
   ForEach(items, id: \.id) { item in  // NOT ForEach(items.indices)
       ItemRow(item: item)
   }
   ```

5. **Ensure touch targets**:
   ```swift
   Button(action: { }) {
       Image(systemName: "xmark")
   }
   .frame(width: 44, height: 44)  // Minimum touch target
   ```

If any anti-pattern would be introduced by this edit, STOP and fix the code to follow the correct pattern before proceeding.

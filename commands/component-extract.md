---
name: component-extract
description: Extract and structure reusable mobile components with design tokens and preview annotations
args:
  - name: code
    type: string
    description: Kotlin/Compose or Swift/SwiftUI code to analyze for component extraction
    required: true
  - name: target
    type: string
    enum: [compose, swiftui, both]
    description: Target platform(s) for component extraction
    required: true
  - name: strategy
    type: string
    enum: [slot-api, view-modifier, generic-view, base-component, all]
    description: Component architecture pattern to apply
    required: false
  - name: include_preview
    type: boolean
    description: Generate preview/preview annotations for component documentation (default: true)
    required: false
  - name: include_tests
    type: boolean
    description: Generate snapshot testing stubs (default: true)
    required: false
  - name: design_tokens
    type: boolean
    description: Extract and organize design tokens from hardcoded values (default: true)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Component Extraction & Design System Architecture

Extract reusable mobile components from existing code with modern design system patterns, design token organization, and comprehensive preview/testing infrastructure.

## Overview

This command helps you transform monolithic UI code into a scalable component library following 2025-2026 mobile design system best practices:

- **Slot API Pattern** (Compose): Flexible content slots for composition
- **View Modifiers** (SwiftUI): Reusable styling and behavior
- **Generic Views**: Type-safe, parameterized component variants
- **Design Tokens**: Centralized colors, typography, spacing, shapes
- **Preview Annotations**: Showkase (Compose) and native Preview (SwiftUI)
- **Snapshot Testing**: Visual regression testing with SnapshotTesting (SwiftUI)

## Instructions

Analyze the provided code and generate a component extraction plan with:

1. **Identified Components**: What should become reusable
2. **Component Architecture**: Best pattern for each component
3. **Design Token Extraction**: Colors, spacing, typography, shapes
4. **API Design**: Slot parameters, callbacks, @Composable lambdas
5. **Preview Generation**: Full preview annotations with variants
6. **Testing Strategy**: Snapshot testing recommendations
7. **Directory Structure**: File organization for scalability
8. **Migration Guide**: How to refactor existing code

## Component Extraction Checklist

### Component Identification (Critical Priority)
- [ ] **CE-001**: All unique UI patterns identified
- [ ] **CE-002**: Component scope clearly defined (not too small, not too large)
- [ ] **CE-003**: No hardcoded strings in component body
- [ ] **CE-004**: No business logic mixed with presentation
- [ ] **CE-005**: State hoisting already planned for each component

### Slot API & Composition (Important Priority)
{{#if (or (eq strategy "slot-api") (eq strategy "all") (not strategy))}}
- [ ] **CE-010**: Content slots defined for flexible composition
- [ ] **CE-011**: Scope receivers (RowScope, ColumnScope) used where appropriate
- [ ] **CE-012**: No hardcoded parameters restricting customization
- [ ] **CE-013**: Single responsibility principle applied
- [ ] **CE-014**: Callbacks provided for user interactions
{{/if}}

### View Modifier Pattern (Important Priority)
{{#if (or (eq strategy "view-modifier") (eq strategy "all") (not strategy))}}
- [ ] **CE-020**: Styling extracted into custom ViewModifiers
- [ ] **CE-021**: ViewModifiers are composable and chainable
- [ ] **CE-022**: Behavior (tap handlers, gestures) separated from styling
- [ ] **CE-023**: ViewModifier generics used for type safety
- [ ] **CE-024**: Base view has minimal styling (defer to modifiers)
{{/if}}

### Generic Views & Variants (Important Priority)
{{#if (or (eq strategy "generic-view") (eq strategy "all") (not strategy))}}
- [ ] **CE-030**: Generic parameters used for content types (T extends View in SwiftUI)
- [ ] **CE-031**: No repetition across component variants
- [ ] **CE-032**: Default parameter values provided for common cases
- [ ] **CE-033**: Constraints on generics are type-safe
{{/if}}

### Design Token Organization (Critical Priority)
{{#if (or (eq design_tokens) (not design_tokens))}}
- [ ] **CE-050**: Color tokens extracted (primary, secondary, surface, error, etc.)
- [ ] **CE-051**: Typography tokens organized (headingLarge, bodyMedium, labelSmall)
- [ ] **CE-052**: Spacing tokens follow 4dp grid (4, 8, 12, 16, 24, 32, 48, 64)
- [ ] **CE-053**: Shape/corner radius tokens defined (small, medium, large, fullShape)
- [ ] **CE-054**: Shadow/elevation tokens consistent across platform
- [ ] **CE-055**: Token names follow semantic naming (not hex-based)
- [ ] **CE-056**: Tokens work with system dark mode (Compose: isSystemInDarkTheme)
{{/if}}

### Preview Annotations (Important Priority)
{{#if (or (eq include_preview) (not include_preview))}}
**Jetpack Compose (Showkase):**
- [ ] **CE-060**: @ShowkaseComposable annotation on all previews
- [ ] **CE-061**: @Preview annotation present for Android Studio integration
- [ ] **CE-062**: 5 permutations auto-generated (light, dark, RTL, font scaled, display scaled)
- [ ] **CE-063**: KDoc with @Showkase group name for browser organization
- [ ] **CE-064**: Component displayed in multiple viewport sizes (phone, tablet, foldable)
- [ ] **CE-065**: State variations shown (enabled, disabled, loading, error states)

**SwiftUI (native Preview):**
- [ ] **CE-070**: #Preview macro for Swift 6+ (or Preview struct for earlier versions)
- [ ] **CE-071**: .preferredColorScheme() for dark mode testing
- [ ] **CE-072**: .environment() for system size category, accessibility settings
- [ ] **CE-073**: Multiple device previews (iPhone, iPad, landscape)
- [ ] **CE-074**: State variations shown with @State mock objects
{{/if}}

### Testing Strategy (Important Priority)
{{#if (or (eq include_tests) (not include_tests))}}
**Snapshot Testing (SwiftUI with SnapshotTesting library):**
- [ ] **CE-080**: Snapshot tests for all component variants
- [ ] **CE-081**: Light + dark mode snapshots
- [ ] **CE-082**: Dynamic content with deterministic test data
- [ ] **CE-083**: Snapshot naming convention: `test_ComponentName_variant_colorScheme`
- [ ] **CE-084**: Reference snapshots committed to version control

**Compose UI Testing (Roborazzi / Paparazzi):**
- [ ] **CE-090**: Screenshot tests for key component states
- [ ] **CE-091**: Golden files stored in source control
- [ ] **CE-092**: Test naming: `test_ComponentName_variant`
{{/if}}

### File Organization (Important Priority)
- [ ] **CE-100**: Components in `/components/` folder organized by feature
- [ ] **CE-101**: Design tokens in `/theme/tokens/` (Colors, Typography, Spacing, Shapes)
- [ ] **CE-102**: Previews in `/previews/` or alongside component with Preview suffix
- [ ] **CE-103**: Tests in `/tests/` mirroring component structure
- [ ] **CE-104**: Shared modifiers in `/modifiers/` or `/theme/`
- [ ] **CE-105**: Clear README in component folder documenting API

### API Design (Critical Priority)
- [ ] **CE-110**: All component parameters documented (param name, type, default, purpose)
- [ ] **CE-111**: Callbacks clearly typed (@Composable lambdas, () -> Unit, etc.)
- [ ] **CE-112**: Optional parameters have sensible defaults
- [ ] **CE-113**: Component follows Material Design 3 / Human Interface Guidelines
- [ ] **CE-114**: Accessibility parameters included (contentDescription, semantics)

## Component Architecture Patterns

### Slot API Pattern (Compose Recommended)

Use for components with flexible content areas. The component is a "skeleton" - you provide the content.

**When to use:**
- Layout containers (Card, Row, Column with special styling)
- Components with headers/footers
- Custom scaffold-like layouts
- Theme-enforced layouts (Material Button, but caller provides content)

```kotlin
// SLOT API PATTERN
@Composable
fun CustomCard(
    modifier: Modifier = Modifier,
    shape: Shape = MaterialTheme.shapes.medium,
    backgroundColor: Color = MaterialTheme.colorScheme.surface,
    content: @Composable (ColumnScope.() -> Unit)  // Flexible content slot
) {
    Surface(
        modifier = modifier,
        shape = shape,
        color = backgroundColor
    ) {
        Column(
            modifier = Modifier.padding(16.dp),
            content = content  // Caller provides content
        )
    }
}

// USAGE
CustomCard {
    Text("Custom Title")
    Spacer(modifier = Modifier.height(8.dp))
    Text("Custom body content")
}
```

**Benefits:**
- Maximum flexibility (caller composes content)
- Single responsibility (component handles layout + theming only)
- Avoids prop drilling
- Easy to test and preview multiple content variations

### View Modifier Pattern (SwiftUI Recommended)

Use for extracting reusable styling and behaviors without creating new Views.

**When to use:**
- Buttons with custom styling
- Cards with shadows + corners + padding
- Text with custom typography
- Forms with validation styling
- Interactive states (hover, focus, disabled)

```swift
// VIEW MODIFIER PATTERN
struct CustomCardModifier: ViewModifier {
    let backgroundColor: Color
    let cornerRadius: CGFloat
    let shadowRadius: CGFloat
    let elevation: CGFloat

    func body(content: Content) -> some View {
        content
            .padding(.horizontal, 16)
            .padding(.vertical, 12)
            .background(backgroundColor)
            .cornerRadius(cornerRadius)
            .shadow(radius: shadowRadius)
    }
}

extension View {
    func customCard(
        backgroundColor: Color = .surface,
        cornerRadius: CGFloat = 12,
        shadowRadius: CGFloat = 4,
        elevation: CGFloat = 2
    ) -> some View {
        modifier(
            CustomCardModifier(
                backgroundColor: backgroundColor,
                cornerRadius: cornerRadius,
                shadowRadius: shadowRadius,
                elevation: elevation
            )
        )
    }
}

// USAGE - Chainable and composable
Text("Hello")
    .customCard()
    .padding()
```

**Benefits:**
- Composable and chainable
- Reusable styling across any View
- Type-safe with generics
- Efficient (no view hierarchy overhead)

### Generic View Pattern

Use for parameterized content that varies by type.

```swift
// GENERIC VIEW PATTERN (SwiftUI)
struct GenericList<Item: Identifiable, Content: View>: View {
    let items: [Item]
    let content: (Item) -> Content

    var body: some View {
        List(items) { item in
            content(item)
        }
    }
}

// USAGE
GenericList(items: users) { user in
    HStack {
        AsyncImage(url: user.avatarURL)
        Text(user.name)
    }
}
```

**Benefits:**
- Type-safe content composition
- No string-based configuration
- Reusable for any Item type
- Natural Swift composition

### Base Component Pattern

Use for components with multiple variants sharing core styling.

```kotlin
// BASE COMPONENT PATTERN (Compose)
@Composable
internal fun BaseButton(
    modifier: Modifier = Modifier,
    shape: Shape = MaterialTheme.shapes.small,
    colors: ButtonColors = ButtonDefaults.buttonColors(),
    elevation: ButtonElevation = ButtonDefaults.buttonElevation(),
    enabled: Boolean = true,
    onClick: () -> Unit,
    content: @Composable RowScope.() -> Unit
) {
    // Core button implementation
    Button(
        modifier = modifier,
        shape = shape,
        colors = colors,
        elevation = elevation,
        enabled = enabled,
        onClick = onClick,
        content = content
    )
}

// VARIANTS use BaseButton with different defaults
@Composable
fun FilledButton(
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    onClick: () -> Unit,
    content: @Composable RowScope.() -> Unit
) {
    BaseButton(
        modifier = modifier,
        colors = ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.colorScheme.primary
        ),
        enabled = enabled,
        onClick = onClick,
        content = content
    )
}
```

**Benefits:**
- DRY (single implementation, multiple variants)
- Consistent styling across variants
- Easy to add new variants

## Design Token Organization

### Recommended Structure

```
android/app/src/main/java/com/partyonapp/android/ui/theme/
├── Color.kt                    # Color tokens
├── Typography.kt               # Font scales
├── Spacing.kt                  # Dimension tokens (4dp grid)
├── Shape.kt                    # Corner radius tokens
├── Elevation.kt                # Shadow tokens
└── Theme.kt                    # CompositionLocal providers

ios/PartyOn/Design/
├── Colors/
│   ├── ColorTokens.swift
│   └── ColorPalette.swift
├── Typography/
│   ├── TypeScale.swift
│   └── FontTokens.swift
├── Spacing/
│   └── SpacingTokens.swift
├── Shapes/
│   └── ShapeTokens.swift
└── Theme/
    └── ThemeProvider.swift
```

### Spacing Grid (Material Design 3)

```kotlin
// Kotlin - use with .dp
object Spacing {
    val xs4 = 4.dp   // Minimal gaps
    val xs8 = 8.dp   // Tight spacing
    val sm12 = 12.dp // Small gaps
    val md16 = 16.dp // Standard margin
    val lg24 = 24.dp // Large spacing
    val xl32 = 32.dp // Extra large
    val xxl48 = 48.dp
    val xxxl64 = 64.dp
}
```

```swift
// Swift - use as constants
enum Spacing {
    static let xs4: CGFloat = 4
    static let xs8: CGFloat = 8
    static let sm12: CGFloat = 12
    static let md16: CGFloat = 16
    static let lg24: CGFloat = 24
    static let xl32: CGFloat = 32
    static let xxl48: CGFloat = 48
    static let xxxl64: CGFloat = 64
}
```

### Color Tokens (Semantic Naming)

```kotlin
// Bad: Hex-based naming
val colorFF6B6B: Color
val color4ECDC4: Color

// Good: Semantic naming
val colorPrimary: Color = Color(0xFF6200EE)
val colorSecondary: Color = Color(0xFF03DAC6)
val colorError: Color = Color(0xFFB00020)
val colorSurface: Color = Color(0xFFFFFFFF)
val colorOnSurface: Color = Color(0xFF1C1B1F)
```

### Typography Scale

```kotlin
// Material Design 3 Typography Scale
val displayLarge = TextStyle(fontSize = 57.sp, fontWeight = FontWeight.Bold, lineHeight = 64.sp)
val displayMedium = TextStyle(fontSize = 45.sp, fontWeight = FontWeight.Bold, lineHeight = 52.sp)
val displaySmall = TextStyle(fontSize = 36.sp, fontWeight = FontWeight.Bold, lineHeight = 44.sp)
val headingLarge = TextStyle(fontSize = 32.sp, fontWeight = FontWeight.Bold, lineHeight = 40.sp)
val headingMedium = TextStyle(fontSize = 28.sp, fontWeight = FontWeight.Bold, lineHeight = 36.sp)
val headingSmall = TextStyle(fontSize = 24.sp, fontWeight = FontWeight.Bold, lineHeight = 32.sp)
val titleLarge = TextStyle(fontSize = 22.sp, fontWeight = FontWeight.Bold, lineHeight = 28.sp)
val titleMedium = TextStyle(fontSize = 16.sp, fontWeight = FontWeight.Medium, lineHeight = 24.sp)
val titleSmall = TextStyle(fontSize = 14.sp, fontWeight = FontWeight.Medium, lineHeight = 20.sp)
val bodyLarge = TextStyle(fontSize = 16.sp, fontWeight = FontWeight.Normal, lineHeight = 24.sp)
val bodyMedium = TextStyle(fontSize = 14.sp, fontWeight = FontWeight.Normal, lineHeight = 20.sp)
val bodySmall = TextStyle(fontSize = 12.sp, fontWeight = FontWeight.Normal, lineHeight = 16.sp)
val labelLarge = TextStyle(fontSize = 14.sp, fontWeight = FontWeight.Medium, lineHeight = 20.sp)
val labelMedium = TextStyle(fontSize = 12.sp, fontWeight = FontWeight.Medium, lineHeight = 16.sp)
val labelSmall = TextStyle(fontSize = 11.sp, fontWeight = FontWeight.Medium, lineHeight = 16.sp)
```

## Preview & Documentation Generation

### Jetpack Compose (Showkase)

```kotlin
// Add to build.gradle.kts
dependencies {
    ksp("com.airbnb.android:showkase-processor:1.0.0-beta8")
    implementation("com.airbnb.android:showkase:1.0.0-beta8")
}

// Component with preview
@Composable
fun CustomButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true
) {
    Button(
        onClick = onClick,
        modifier = modifier,
        enabled = enabled
    ) {
        Text(text)
    }
}

// Preview annotation
@Preview(name = "Default State", group = "CustomButton")
@Composable
fun CustomButtonPreview() {
    CustomButton(
        text = "Click me",
        onClick = {}
    )
}

// Showkase integration
@ShowkaseComposable(
    name = "Custom Button",
    group = "Components/Buttons",
    description = "A custom button following Material Design 3"
)
@Preview(name = "Default", group = "CustomButton", backgroundColor = 0xFFFFFF, showBackground = true)
@Composable
fun CustomButtonShowkase() {
    CustomButton(text = "Click me", onClick = {})
}

// 5 automatic permutations generated:
// - Light mode
// - Dark mode
// - RTL (right-to-left)
// - Font scaled (larger text)
// - Display scaled (larger overall)
```

**Access Preview Browser:**
- Run app in Android Studio
- App drawer → Show Showkase
- Browse, search, inspect components with KDoc

### SwiftUI (Swift 6+)

```swift
// Component definition
struct CustomButton: View {
    let title: String
    let action: () -> Void
    @Environment(\.isEnabled) var isEnabled

    var body: some View {
        Button(action: action) {
            Text(title)
                .frame(maxWidth: .infinity)
                .padding()
                .background(isEnabled ? Color.primary : Color.gray)
                .foregroundColor(.white)
                .cornerRadius(8)
        }
        .disabled(!isEnabled)
    }
}

// Swift 6+ Preview macro
#Preview("Default State") {
    CustomButton(title: "Click me", action: {})
        .padding()
}

#Preview("Dark Mode") {
    CustomButton(title: "Click me", action: {})
        .padding()
        .preferredColorScheme(.dark)
}

#Preview("Disabled") {
    CustomButton(title: "Click me", action: {})
        .disabled(true)
        .padding()
}

#Preview("Large Text (Accessibility)") {
    CustomButton(title: "Click me", action: {})
        .environment(\.sizeCategory, .extraExtraLarge)
        .padding()
}

// Multiple device previews
#Preview("iPhone") {
    CustomButton(title: "Click me", action: {})
        .previewDevice(PreviewDevice(rawValue: "iPhone 15"))
}

#Preview("iPad") {
    CustomButton(title: "Click me", action: {})
        .previewDevice(PreviewDevice(rawValue: "iPad (7th generation)"))
}
```

## Snapshot Testing

### SwiftUI with SnapshotTesting

```swift
import SnapshotTesting
import XCTest

class CustomButtonTests: XCTestCase {
    func testCustomButton_default() {
        let view = CustomButton(title: "Click me", action: {})
        assertSnapshot(matching: view, as: .image)
    }

    func testCustomButton_darkMode() {
        let view = CustomButton(title: "Click me", action: {})
            .preferredColorScheme(.dark)
        assertSnapshot(matching: view, as: .image(colorScheme: .dark))
    }

    func testCustomButton_disabled() {
        let view = CustomButton(title: "Click me", action: {})
            .disabled(true)
        assertSnapshot(matching: view, as: .image)
    }

    func testCustomButton_largeText() {
        let view = CustomButton(title: "Click me", action: {})
            .environment(\.sizeCategory, .extraExtraLarge)
        assertSnapshot(matching: view, as: .image)
    }
}
```

**Workflow:**
1. First run: Generates golden files (reference snapshots)
2. Commit golden files to version control
3. On code changes: Compares new snapshots against golden
4. Failures show visual diff
5. `XCTAssertSnapshotTesting` auto-updates on approval

### Compose with Roborazzi

```kotlin
@RunWith(RobolectricTestRunner::class)
class CustomButtonScreenshotTest {
    @get:Rule
    val rule = RoborazziRule()

    @Test
    fun testCustomButton_default() {
        rule.captureRoboImage(
            filePath = "CustomButton/default.png"
        ) {
            CustomButton(
                text = "Click me",
                onClick = {},
                modifier = Modifier.wrapContentSize()
            )
        }
    }

    @Test
    fun testCustomButton_darkMode() {
        rule.captureRoboImage(
            filePath = "CustomButton/dark_mode.png"
        ) {
            AppTheme(useDarkTheme = true) {
                CustomButton(
                    text = "Click me",
                    onClick = {},
                    modifier = Modifier.wrapContentSize()
                )
            }
        }
    }
}
```

## Output Format

Generate a structured component extraction report:

```markdown
## Component Extraction Analysis: [File/Screen Name]

### Identified Components
List each component with:
- Component name
- Current location (monolithic)
- Extracted location (proposed)
- Suggested pattern (Slot API, ViewModifier, Generic, Base)
- Scope (size, responsibility)

Example:
- **EventCard**: Extract to `components/EventCard.kt`
  - Pattern: Slot API (flexible content)
  - Scope: 120 lines → 80 lines (extracted)
  - Key props: title, description, image, onTap

### Design Token Extraction
- **Colors found**: [hardcoded hex values and where used]
  - Suggested tokens: primary, secondary, surface, error
- **Typography**: [font sizes, weights found]
  - Suggested scale: headingLarge, bodyMedium, labelSmall
- **Spacing**: [pixel values that don't follow grid]
  - Align to: 4dp grid (4, 8, 12, 16, 24, 32, 48, 64)
- **Shapes**: [corner radii, borders found]
  - Create tokens: small (4dp), medium (12dp), large (16dp)

### Component Extraction Plan
For each component, provide:
1. **API Design**
   ```kotlin
   @Composable
   fun EventCard(
       event: Event,
       modifier: Modifier = Modifier,
       onTap: (Event) -> Unit,
       header: @Composable (Event) -> Unit = { /* default */ },
       footer: @Composable (Event) -> Unit = { /* default */ }
   ) { ... }
   ```

2. **Slot Breakdown**: What content slots to provide

3. **Design Tokens**: Which tokens to use (no hardcoded values)

4. **Preview Code**: Complete Showkase/Preview annotations

5. **Testing Strategy**: What snapshots to capture

### File Organization
```
components/
├── EventCard/
│   ├── EventCard.kt
│   ├── EventCardPreview.kt
│   ├── EventCardTests.kt
│   └── README.md
├── CustomButton/
│   ├── CustomButton.kt
│   ├── CustomButtonPreview.kt
│   └── CustomButtonTests.kt
└── ...

theme/
├── Color.kt
├── Typography.kt
├── Spacing.kt
├── Shape.kt
└── Theme.kt
```

### Migration Guide
Step-by-step instructions to refactor existing code:
1. Extract components first (no behavior changes)
2. Update imports in existing screens
3. Remove hardcoded styling (move to tokens)
4. Add previews/snapshots
5. Run tests, verify visual consistency

### Recommendations
- Quick wins (easy extractions first)
- High-impact components (used most often)
- Dependencies (extract in order)

### Summary
- Total components identified: X
- Estimated lines to extract: X
- New tokens to define: X
- Preview permutations: X
- Snapshot tests recommended: X
- Overall composition score: X% (reusability)
```

## Example Output

```markdown
## Component Extraction Analysis: EventDetailScreen.kt

### Identified Components

**EventCard** (120 lines)
- Pattern: Slot API
- Scope: Reusable event summary card
- Key slots: header, content, footer
- Found in: EventDetailScreen:45-165

**EventActionButton** (35 lines)
- Pattern: View Modifier + Generic
- Scope: Primary action button for events
- Variations: Join, Leave, Host
- Found in: EventDetailScreen:200-235

**EventMetadata** (55 lines)
- Pattern: Base Component
- Scope: Date, time, location, attendee count
- Variants: Compact, full, upcoming
- Found in: EventDetailScreen:300-355

### Design Token Extraction

**Colors found:**
- `#93DF7C` (join button) → colorSuccess
- `#EEF2F7` (card background) → colorSurface
- `#1C1B1F` (text) → colorOnSurface
- `#B0BCCC` (disabled) → colorOutline

**Typography:**
- 24sp bold → headingMedium
- 16sp regular → bodyMedium
- 14sp medium → labelMedium

**Spacing:**
- 16dp horizontal padding → md16
- 8dp gaps → xs8
- 12dp card spacing → sm12

**Shapes:**
- 18dp radius → medium
- 4dp radius → small

### Component Extraction Plan

#### EventCard Component
```kotlin
@Composable
fun EventCard(
    event: Event,
    modifier: Modifier = Modifier,
    onTap: () -> Unit,
    header: @Composable (ColumnScope.(Event) -> Unit)? = null,
    content: @Composable (ColumnScope.(Event) -> Unit)? = null,
    footer: @Composable (ColumnScope.(Event) -> Unit)? = null,
) {
    Surface(
        modifier = modifier
            .fillMaxWidth()
            .clickable(onClick = onTap),
        shape = MaterialTheme.shapes.medium,
        color = MaterialTheme.colorScheme.surface,
        tonalElevation = 2.dp
    ) {
        Column(
            modifier = Modifier.padding(Spacing.md16)
        ) {
            header?.invoke(this, event)
            content?.invoke(this, event)
            footer?.invoke(this, event)
        }
    }
}
```

**Slot breakdown:**
- `header`: Event title, image (optional)
- `content`: Default shows date/time/location
- `footer`: Join button, action menu

**Tokens used:** MaterialTheme.shapes.medium, colorScheme.surface, Spacing.md16

**Previews needed:**
- Default with all content
- Header only
- Compact (no image)
- Dark mode variant
- Accessibility (large text)

### File Organization
```
android/app/src/main/java/com/partyonapp/android/ui/
├── components/
│   ├── cards/
│   │   ├── EventCard.kt
│   │   ├── EventCardPreview.kt (Showkase)
│   │   ├── EventCardTests.kt (Roborazzi)
│   │   └── README.md
│   ├── buttons/
│   │   ├── EventActionButton.kt
│   │   ├── EventActionButtonPreview.kt
│   │   └── EventActionButtonTests.kt
│   └── metadata/
│       ├── EventMetadata.kt
│       └── EventMetadataPreview.kt
└── theme/
    ├── Color.kt (new tokens)
    ├── Spacing.kt (new tokens)
    └── Theme.kt (updated)
```

### Migration Guide

1. **Create theme tokens** (before components)
   ```bash
   # Color.kt with colorSuccess, colorSurface, etc.
   # Spacing.kt with md16, xs8, sm12, etc.
   # Shape.kt with medium, small radius tokens
   ```

2. **Extract EventCard component**
   - Copy 120 lines from EventDetailScreen:45-165
   - Add slot parameters (header, content, footer)
   - Replace hardcoded `#93DF7C` with `MaterialTheme.colorScheme.success`
   - Replace hardcoded 16.dp with `Spacing.md16`

3. **Update EventDetailScreen imports**
   - Remove inline EventCard code
   - Add `import com.partyonapp.android.ui.components.cards.EventCard`

4. **Add Showkase previews**
   - Create EventCardPreview.kt
   - 5 automatic permutations (light, dark, RTL, etc.)
   - Variations: default, compact, dark mode

5. **Add Roborazzi snapshots**
   - Capture default variant
   - Capture dark mode
   - Capture accessibility (large text)

6. **Verify visuals**
   - Run app, compare old vs new screenshots
   - Ensure pixel-perfect match

### Recommendations

**Quick wins (do first):**
1. Extract EventCard (used 8 places, high reusability)
2. Extract EventActionButton (3 variants, clear API)
3. Define color + spacing tokens (blocks both components)

**High-impact:**
- EventCard: Used in event list, event detail, bookmarked events
- EventMetadata: Used in 5 different screens

**Dependencies:**
- Tokens must be defined first
- EventActionButton depends on color tokens
- EventCard depends on shape tokens

### Summary
- Total components identified: 3
- Estimated extraction: 210 lines → 280 lines (with defaults + slots)
- New tokens: 12 (colors, spacing, shapes)
- Preview permutations: 15+ (5 per component)
- Snapshot tests: 9 (3 per component)
- Reusability score: 85% (highly reusable components)
```

## Common Pitfalls & Best Practices

### DO:
- Extract business logic to ViewModels first (components are presentation-only)
- Use slot APIs to maximize flexibility
- Define all tokens in one place
- Add comprehensive previews before refactoring existing screens
- Keep components small (< 150 lines of code)
- Document expected content in slot parameters
- Test snapshots on multiple devices
- Review token usage in design tools (Figma, Material Design 3)

### DON'T:
- Don't hardcode colors, spacing, or typography values in components
- Don't mix state management with component definition
- Don't create generic components that are too abstract
- Don't skip preview annotations (critical for design consistency)
- Don't test snapshots without deterministic data (random content fails)
- Don't create components so small they fragment the codebase
- Don't ignore accessibility (semantics, content descriptions, text contrast)

## References

This command incorporates best practices from:
- [Material Design 3 Jetpack Compose](https://m3.material.io/develop/android/jetpack-compose)
- [Showkase: Android Component Browser](https://github.com/airbnb/Showkase)
- [SwiftUI Component Library Patterns](https://dev.to/sebastienlato/build-a-reusable-swiftui-component-library-15i3)
- [Compose Slot API Guidelines](https://android.googlesource.com/platform/frameworks/support/+/androidx-main/compose/docs/compose-component-api-guidelines.md)
- [SnapshotTesting for SwiftUI](https://github.com/pointfreeco/swift-snapshot-testing)
- [Design Systems in 2026](https://rydarashid.medium.com/design-systems-in-2026-predictions-pitfalls-and-power-moves-f401317f7563)
- [SwiftUI Reusable Components](https://dev.to/swift_pal/master-swiftui-design-systems-from-scattered-colors-to-unified-ui-components-4i9c)


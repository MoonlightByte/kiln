---
name: polish-android
description: Transform good Compose code into excellent Material Design 3 production-grade Android UI
args:
  - name: code
    type: string
    description: Kotlin/Jetpack Compose code to polish
    required: true
  - name: focus
    type: string
    enum: [animation, adaptive, tokens, a11y, performance, state, all]
    description: Specific polish area (default: all)
    required: false
  - name: level
    type: string
    enum: [professional, expert, masterclass]
    description: Polish intensity (default: professional)
    required: false
severity_levels:
  - enhancement
  - optimization
  - refinement
  - excellence
---

# Polish Android Code with Material Design 3 Excellence

Transform functional Jetpack Compose code into production-grade UI that exemplifies Material Design 3 principles, accessibility excellence, and modern animation craft.

Based on latest 2025-2026 best practices: Material 3 Expressive, Compose 1.6+, dynamic color systems, adaptive layouts, and accessibility standards.

## Instructions

Analyze the provided Compose code and provide detailed polish recommendations grounded in:

1. **Material Design 3 Expressive** (2025 evolution) - shape morphing, unified motion system, semantic tokens
2. **Jetpack Compose 1.6+ Features** - pausable composition, advanced animations, new shape APIs
3. **Accessibility Excellence** - WCAG AA by default, semantic clarity, adaptive sizing
4. **Adaptive Design** - responsive layouts for phones, tablets, foldables
5. **State Management** - modern patterns with Compose Runtime insights
6. **Animation Polish** - Material motion springs, shared element transitions, predictable easing

## Polish Checklist

### Animation & Motion (Expressive Polish)
{{#if (or (eq focus "animation") (eq focus "all") (not focus))}}

#### Material 3 Expressive Motion System
- [ ] **MOT-001**: Animations use Material 3 motion durations (50ms instant, 150ms short, 250ms medium, 350ms long)
- [ ] **MOT-002**: Easing follows M3 Expressive system (emphasized, standard, legacy movement easing)
- [ ] **MOT-003**: Spatial springs used for physics-based movement (new in M3E)
- [ ] **MOT-004**: Effects springs for color/opacity transitions (new in M3E)
- [ ] **MOT-005**: Animations respect reduced motion preference (`LocalContentColor`, `LocalTextColor`)

#### Shape Morphing & Transitions
- [ ] **MOT-006**: Interactive shapes use shape morphing (new in M3E with 35+ shape library)
- [ ] **MOT-007**: Shape transitions linked to interaction states (pressed, selected, disabled)
- [ ] **MOT-008**: Smooth bezier curves for shape morphing animations
- [ ] **MOT-009**: Shape scale (4dp to 28dp) correctly applied based on component type

#### Advanced Animation Patterns
- [ ] **MOT-010**: Shared element transitions for navigation using `Modifier.skipToLookaheadPosition()`
- [ ] **MOT-011**: Transition composables manage multiple animations simultaneously
- [ ] **MOT-012**: updateTransition() used for state-driven multi-value animations
- [ ] **MOT-013**: animateContentSize() for size changes, animateColorAsState() for colors
- [ ] **MOT-014**: Enter/exit animations defined with predictable timing

#### Micro-interactions
- [ ] **MOT-015**: Button press feedback using temporal feedback (50-100ms)
- [ ] **MOT-016**: List item reveals use subtle scale + alpha animations
- [ ] **MOT-017**: Loading states use unified circular progress motion
- [ ] **MOT-018**: Floating action buttons have distinct entry/exit animations

{{/if}}

### Adaptive & Responsive Layouts
{{#if (or (eq focus "adaptive") (eq focus "all") (not focus))}}

#### Adaptive Foundation
- [ ] **ADP-001**: Uses `WindowWidthSizeClass` for responsive breakpoints (Compact, Medium, Expanded)
- [ ] **ADP-002**: `NavigationSuiteScaffold` auto-switches nav bar ↔ rail based on window size
- [ ] **ADP-003**: `ListDetailPaneScaffold` for master-detail on tablets/desktops
- [ ] **ADP-004**: `SupportingPaneScaffold` for triple-pane layouts on larger screens

#### Flexible Layout Strategies
- [ ] **ADP-005**: `BoxWithConstraints` used for dynamic layout decisions (maxWidth > 600.dp)
- [ ] **ADP-006**: `Modifier.weight()` for fluid column/row distribution
- [ ] **ADP-007**: Material 3 adaptive components from `androidx.compose.material3.adaptive`
- [ ] **ADP-008**: Foldable-aware layouts using `DevicePostureFlow` (when relevant)

#### Content Adaptation
- [ ] **ADP-009**: Typography scales adaptively (smaller on compact, medium on expanded)
- [ ] **ADP-010**: Touch targets remain 48x48dp+ across all screen sizes
- [ ] **ADP-011**: Multi-column grids on tablets (`Column(Modifier.weight(1f))` pattern)
- [ ] **ADP-012**: Images use aspect ratios that scale gracefully (`contentScale`)

{{/if}}

### Design Tokens & Theming
{{#if (or (eq focus "tokens") (eq focus "all") (not focus))}}

#### Semantic Tokens (2025 Expansion)
- [ ] **TOK-001**: All colors use `MaterialTheme.colorScheme.*` tokens (no hardcoded colors)
- [ ] **TOK-002**: All typography uses `MaterialTheme.typography.*` tokens
- [ ] **TOK-003**: Shape tokens applied correctly (`small`, `medium`, `large` based on component)
- [ ] **TOK-004**: New shape tokens used from expanded M3E library (35 shapes available)
- [ ] **TOK-005**: Spacing follows 4dp grid (4, 8, 12, 16, 24, 32, 48, 64, 80, 96)

#### Color System & Dynamic Color
- [ ] **TOK-006**: Dynamic color enabled for Android 12+ (Material You adaptation)
- [ ] **TOK-007**: Tonal elevation layers used for depth (surface, surfaceContainer, surfaceVariant)
- [ ] **TOK-008**: Proper contrast ratios guaranteed (M3 tokens ensure WCAG AA by default)
- [ ] **TOK-009**: Color roles assigned semantically (primary for actions, secondary for supporting)
- [ ] **TOK-010**: Tertiary color used strategically (brand color, not overused)

#### Theme Builder Integration
- [ ] **TOK-011**: Theme generated using Material Theme Builder (cross-platform consistency)
- [ ] **TOK-012**: Light/dark theme colors fully defined
- [ ] **TOK-013**: Custom color tokens defined if going beyond default M3 palette
- [ ] **TOK-014**: State colors defined (enabled, disabled, focused, hovered, pressed)

{{/if}}

### Accessibility Excellence
{{#if (or (eq focus "a11y") (eq focus "all") (not focus))}}

#### Semantic Clarity
- [ ] **A11-001**: All images/icons have descriptive `contentDescription` (not null, not "image")
- [ ] **A11-002**: Text semantics clear (buttons are buttons, not just clickable text)
- [ ] **A11-003**: Form fields have associated labels via `modifier.semantics { ... }`
- [ ] **A11-004**: Headings correctly marked with Role.Heading
- [ ] **A11-005**: List items semantically correct (Role.List, Role.ListItem)

#### Touch & Motor Accessibility
- [ ] **A11-006**: All interactive elements 48x48dp minimum (or use `minimumInteractiveComponentSize()`)
- [ ] **A11-007**: Touch targets have adequate spacing (8dp minimum between adjacent targets)
- [ ] **A11-008**: Focus indicators clearly visible (high contrast, not just color)
- [ ] **A11-009**: Custom focus handling respects system focus order
- [ ] **A11-010**: Long-press actions have visible feedback

#### Visual Accessibility
- [ ] **A11-011**: Text contrast ≥ 4.5:1 for normal text, ≥ 3:1 for large text (WCAG AA)
- [ ] **A11-012**: Color not sole conveyor of information (use icons, patterns, text)
- [ ] **A11-013**: Disabled state visually distinct (not just reduced opacity)
- [ ] **A11-014**: Error states clearly marked (icon + color + text)
- [ ] **A11-015**: Loading states announced to screen readers

#### Screen Reader & TalkBack
- [ ] **A11-016**: `contentDescription` describes action, not visual appearance
- [ ] **A11-017**: Custom composables expose `semantics` for TalkBack understanding
- [ ] **A11-018**: List items announce position ("1 of 10") via semantics
- [ ] **A11-019**: Dynamic content updates announced via `LiveRegion`
- [ ] **A11-020**: Skip links for complex layouts (skip to main content)

#### Dynamic Type & Scaling
- [ ] **A11-021**: Text respects device font size preference (use sp, not dp for text)
- [ ] **A11-022**: Line heights scale with text size (1.5x minimum for body text)
- [ ] **A11-023**: Buttons/touch targets scale with Dynamic Type
- [ ] **A11-024**: Custom layouts don't break with large text (test at 200% scaling)

{{/if}}

### State Management & Data Flow
{{#if (or (eq focus "state") (eq focus "all") (not focus))}}

#### Modern State Patterns (Compose 1.6+)
- [ ] **STA-001**: Business logic in ViewModel (StateFlow, LiveData, or Flow)
- [ ] **STA-002**: UI state in composable using `collectAsState()` or `collectAsStateWithLifecycle()`
- [ ] **STA-003**: NOT passing MutableState to child composables
- [ ] **STA-004**: State hoisting for shared state between siblings
- [ ] **STA-005**: Unidirectional data flow (data down, events up)

#### Efficient Recomposition
- [ ] **STA-006**: Using `derivedStateOf` to limit recomposition (not reading state directly in body)
- [ ] **STA-007**: Modifiers created outside composable body (stable references)
- [ ] **STA-008**: Lambda captures use `remember` to prevent recomposition
- [ ] **STA-009**: Data classes marked `@Immutable` or `@Stable` for smart recomposition
- [ ] **STA-010**: Lazy layout state hoisting (avoid LaunchedEffect per item)

#### Side Effects Discipline
- [ ] **STA-011**: API calls in `LaunchedEffect`, not composition body
- [ ] **STA-012**: Analytics/logging in `SideEffect` or `DisposableEffect`
- [ ] **STA-013**: Cleanup logic in `DisposableEffect` (unsubscribe, cancel jobs)
- [ ] **STA-014**: NO direct state modification during composition
- [ ] **STA-015**: Saved state using `rememberSaveable` for process death

{{/if}}

### Performance Optimization
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}

#### Composition Performance (New in Compose 1.6: Pausable Composition)
- [ ] **PER-001**: Pausable composition enabled (runtime can pause/resume between frames)
- [ ] **PER-002**: Heavy computation wrapped in `remember` (cached across recompositions)
- [ ] **PER-003**: Using `ImmutableList` for stable collection references in state
- [ ] **PER-004**: Providing stable `key()` parameter to `LazyColumn`/`LazyRow`
- [ ] **PER-005**: NOT creating objects in composition (use `remember`)

#### Layout Performance
- [ ] **PER-006**: Using `LazyColumn` for lists > 10 items (not Column)
- [ ] **PER-007**: LazyColumn provides maxHeight/maxWidth to avoid unmeasured layouts
- [ ] **PER-008**: Lazy layout prefetch enabled (prepare frames ahead of time)
- [ ] **PER-009**: Custom layouts implement `onMeasured()` efficiently
- [ ] **PER-010**: Avoiding nested lazy layouts (use single list where possible)

#### Draw Performance
- [ ] **PER-011**: Animations in draw phase, not layout phase (lower overhead)
- [ ] **PER-012**: Using `drawBehind` for custom drawing instead of custom layouts
- [ ] **PER-013**: Shadows optimized (new `Modifier.shadow()` API, not custom draws)
- [ ] **PER-014**: Images optimized (correct resolution, lazy loading)
- [ ] **PER-015**: Gradients use hardware acceleration (not custom draw loops)

#### Memory Efficiency
- [ ] **PER-016**: ViewModels scoped correctly (don't leak contexts)
- [ ] **PER-017**: Listeners/observers cleaned up (DisposableEffect)
- [ ] **PER-018**: Large objects in `rememberSaveable` avoided (use minimal state)
- [ ] **PER-019**: Image caching implemented (Coil for async images)
- [ ] **PER-020**: No memory leaks from closure captures

{{/if}}

### Code Quality & Best Practices
{{#if (or (eq focus "all") (not focus))}}

#### Compose Best Practices
- [ ] **QUA-001**: Function composables are idempotent (same inputs → same output)
- [ ] **QUA-002**: No side effects in composable body
- [ ] **QUA-003**: Composable function parameters are stable/immutable
- [ ] **QUA-004**: Preview composables included for UI verification
- [ ] **QUA-005**: Proper naming (composable functions: PascalCase, variables: camelCase)

#### M3 Compliance
- [ ] **QUA-006**: Using official M3 components (Button, Card, TextField, etc.)
- [ ] **QUA-007**: Custom components extend M3 components (not re-implementing wheels)
- [ ] **QUA-008**: Component customization uses M3 slots (colors, shapes, typography)
- [ ] **QUA-009**: No Material 2 components mixed with Material 3
- [ ] **QUA-010**: Deprecated composables removed (m2.Button → Button, etc.)

#### Code Organization
- [ ] **QUA-011**: Composables organized by feature, not by type
- [ ] **QUA-012**: Shared state in ViewModels, not scattered across files
- [ ] **QUA-013**: Theme/colors in centralized Theme.kt file
- [ ] **QUA-014**: Constants extracted to avoid magic numbers
- [ ] **QUA-015**: Complex logic extracted to extension functions

{{/if}}

## Code to Polish

```kotlin
{{code}}
```

## Output Format

Generate a detailed polish report:

```markdown
## Polish Report: [Component/Screen Name]

### Excellence Opportunities (🌟)
Enhancements that elevate from good to excellent. Organized by category.

#### Animation Polish (if applicable)
- Opportunity X: Description with context
  - Current: [code snippet]
  - Polish: [improved code snippet]
  - Why: Aligns with M3 Expressive motion system, improves perceived performance

#### Adaptive Design (if applicable)
- Opportunity X: [responsive enhancement]

#### Token System (if applicable)
- Opportunity X: [design token optimization]

#### Accessibility (if applicable)
- Opportunity X: [semantic or a11y improvement]

#### Performance (if applicable)
- Opportunity X: [optimization opportunity]

#### State Management (if applicable)
- Opportunity X: [data flow improvement]

### Applied Checklist
- [ ] MOT-### - [checked items from categories above]
- [ ] ADP-### - [...]
- [ ] TOK-### - [...]
- [ ] A11-### - [...]
- [ ] PER-### - [...]
- [ ] STA-### - [...]
- [ ] QUA-### - [...]

### Polish Summary
- **Category**: Opportunities by focus area
  - Animation: X enhancements
  - Adaptive: X enhancements
  - Tokens: X enhancements
  - Accessibility: X enhancements
  - Performance: X enhancements
  - State: X enhancements
  - Code Quality: X enhancements

### Implementation Priority
1. [Critical enhancement with highest impact]
2. [Important enhancement for UX/performance]
3. [Code quality or refinement opportunity]
4. [Future polish for edge cases]

### Before/After Code Examples
Show key transformations with explanation.

### References (2025-2026 Best Practices)
- Material Design 3 Expressive shape morphing and motion
- Compose 1.6+ pausable composition and advanced APIs
- Dynamic color system and semantic tokens expansion
- Adaptive layout strategies with window size classes
- Latest accessibility patterns (TalkBack, screen readers)
```

## Example Polish Report

```markdown
## Polish Report: EventCard.kt

### Excellence Opportunities (🌟)

#### Animation Polish
- **Opportunity 1**: Shape morphing on card interaction
  - Current: Static rounded corner shape
  - Polish:
    ```kotlin
    val cornerSize by animateDpAsState(
        targetValue = if (isPressed) 12.dp else 28.dp,
        animationSpec = spring(dampingRatio = 0.8f)  // M3E spatial spring
    )
    Card(shape = RoundedCornerShape(cornerSize))
    ```
  - Why: Provides tactile feedback aligning with M3 Expressive unified motion system

#### Token System
- **Opportunity 2**: Dynamic elevation based on state
  - Current: Fixed shadow elevation
  - Polish:
    ```kotlin
    val elevation by animateDpAsState(
        targetValue = if (isHovered) 8.dp else 1.dp,
        animationSpec = tween(150, easing = LinearEasing)  // M3 medium duration
    )
    Card(elevation = elevation)
    ```
  - Why: Uses M3 tonal elevation system; smooth transitions improve perceived interactivity

#### Accessibility
- **Opportunity 3**: Semantic clarity for screen readers
  - Current: `Icon(Icons.Default.Star, contentDescription = null)`
  - Polish:
    ```kotlin
    Icon(
        Icons.Default.Star,
        contentDescription = if (isFavorited) "Remove from favorites" else "Add to favorites",
        modifier = Modifier.semantics { role = Role.Button }
    )
    ```
  - Why: Enables TalkBack users to understand action; role = Button improves navigation

#### Adaptive Design
- **Opportunity 4**: Responsive layout for tablets
  - Current: Fixed width Column
  - Polish:
    ```kotlin
    BoxWithConstraints {
        if (maxWidth > 600.dp) {
            Row(horizontalArrangement = Arrangement.spacedBy(16.dp)) {
                Image(modifier = Modifier.weight(1f))
                Column(modifier = Modifier.weight(1f)) { ... }
            }
        } else {
            Column(verticalArrangement = Arrangement.spacedBy(12.dp)) { ... }
        }
    }
    ```
  - Why: Leverages tablet screen space; maintains 48x48dp touch targets on all sizes

#### Performance
- **Opportunity 5**: Lazy loading with prefetch
  - Current: All events loaded upfront
  - Polish:
    ```kotlin
    LazyColumn(
        state = lazyListState,
        modifier = Modifier.fillMaxSize()
    ) {
        items(events.size, key = { events[it].id }) { index ->
            EventCard(events[index])  // Key ensures stable recomposition
        }
    }
    ```
  - Why: Pausable composition (Compose 1.6) auto-optimizes; providing keys prevents relayout

### Applied Checklist
- [x] MOT-001 - Uses M3 motion durations (150ms springs)
- [x] MOT-003 - Spatial springs for physics-based movement
- [x] ADP-005 - BoxWithConstraints for responsive breakpoints
- [x] A11-001 - Descriptive contentDescription on icons
- [x] A11-006 - Touch targets 48x48dp+ guaranteed
- [x] PER-004 - Stable keys provided to LazyColumn
- [x] TOK-001 - MaterialTheme tokens used throughout
- [x] STA-002 - UI state in composable with collectAsState()
- [ ] MOT-006 - Shape morphing (opportunity to add)
- [ ] ADP-001 - WindowWidthSizeClass integration (future)

### Polish Summary
- **Animation**: 2 enhancements (shape morphing, elevation feedback)
- **Adaptive**: 1 enhancement (tablet layout)
- **Accessibility**: 1 enhancement (semantic roles)
- **Performance**: 1 enhancement (stable keys)
- **Tokens**: Already excellent

### Implementation Priority
1. Add semantic role to interactive icons (A11 enhancement, 5 min)
2. Implement adaptive tablet layout (ADP enhancement, 15 min)
3. Add shape morphing feedback (animation polish, 20 min)
4. Consider future: Predictive back animation support

### Before/After Code Examples

**Shape Morphing** (Motion Polish):
```kotlin
// BEFORE: Static shape
Card(shape = RoundedCornerShape(28.dp)) {
    // content
}

// AFTER: Animated shape morphing
val shape by animateShapeAsState(
    targetShape = if (isPressed) RoundedCornerShape(12.dp) else RoundedCornerShape(28.dp),
    animationSpec = spring(dampingRatio = 0.8f)  // Spatial spring per M3E
)
Card(shape = shape) {
    // content
}
```

**Responsive Layout**:
```kotlin
// BEFORE: Fixed mobile layout
Column(modifier = Modifier.width(400.dp)) {
    Image(modifier = Modifier.size(80.dp))
    Text("Event Name")
    Button(onClick = {}) { Text("Join") }
}

// AFTER: Adaptive to screen size
BoxWithConstraints(modifier = Modifier.fillMaxWidth()) {
    if (maxWidth > 600.dp) {
        // Tablet: side-by-side
        Row(horizontalArrangement = Arrangement.spacedBy(16.dp)) {
            Image(modifier = Modifier.weight(1f).heightIn(max = 200.dp))
            Column(modifier = Modifier.weight(1f)) {
                Text("Event Name", style = MaterialTheme.typography.headlineSmall)
                Button(onClick = {}, modifier = Modifier.fillMaxWidth()) { Text("Join") }
            }
        }
    } else {
        // Phone: stacked
        Column(verticalArrangement = Arrangement.spacedBy(12.dp)) {
            Image(modifier = Modifier.fillMaxWidth().height(150.dp))
            Text("Event Name", style = MaterialTheme.typography.headlineSmall)
            Button(onClick = {}, modifier = Modifier.fillMaxWidth()) { Text("Join") }
        }
    }
}
```

### References (2025-2026 Best Practices)
- [Material Design 3 for Jetpack Compose](https://m3.material.io/develop/android/jetpack-compose)
- [Material 3 Expressive Motion & Shape](https://medium.com/@hiren6997/mastering-material-3-in-jetpack-compose-the-2025-guide-1c1bd5acc480)
- [Compose 1.6 Pausable Composition & Advanced APIs](https://android-developers.googleblog.com/2025/12/whats-new-in-jetpack-compose-december.html)
- [Building Adaptive Layouts with Material 3](https://developer.android.com/develop/ui/compose/build-adaptive-apps)
- [Material Design 3 Accessibility Standards](https://m3.material.io/foundations/accessible-design)
```

## 2025-2026 Key Concepts

### Material 3 Expressive Features (New)
- **Shape Morphing**: 35-shape library + smooth transitions between shapes based on state
- **Unified Motion System**: All M3 components use consistent motion principles
- **Spatial Springs**: Physics-based movement for natural feel
- **Effects Springs**: Smooth color/opacity transitions

### Jetpack Compose 1.6+ Features
- **Pausable Composition**: Runtime pauses work between frames, improves perceived performance
- **Advanced Shape APIs**: `Modifier.shadow()` improvements, shape morphing support
- **skipToLookaheadPosition()**: Better shared element transition support
- **Autofill & Secure Text Fields**: New TextFieldState-based APIs

### Dynamic Color & Token System Expansion
- **Design Token Categories**: Colors, typography, shapes (spacing & motion tokens coming 2025 Q3)
- **Semantic Tokens**: Named values instead of hardcoded properties
- **WCAG AA Guaranteed**: Material 3 color system ensures accessibility by default
- **Cross-Platform**: Theme Builder exports tokens for Android, Flutter, React, web

### Adaptive Design Best Practices
- **Window Size Classes**: Compact (<600dp), Medium (600-839dp), Expanded (≥840dp)
- **Material 3 Adaptive Components**: NavigationSuiteScaffold, ListDetailPaneScaffold, SupportingPaneScaffold
- **Foldable Support**: Respond to device posture and hinge position
- **Touch Target Consistency**: 48x48dp minimum across all screen sizes

### Accessibility Excellence
- **Semantic Clarity**: Use Role.Button, Role.Heading for proper TalkBack navigation
- **Dynamic Type**: All text in sp (scale-independent pixels), test at 200% scaling
- **Focus Indicators**: High contrast, not color-only
- **Screen Reader Support**: contentDescription describes action, not appearance

## When to Use This Command

- **After implementing a feature**: Polish the code before considering it complete
- **Before design review**: Ensure M3 compliance and accessibility excellence
- **When refactoring**: Modernize to 2025-2026 best practices
- **For mentoring**: Learn by example from polish suggestions
- **Before production**: Catch animation, performance, and a11y issues early

---

**Goal**: Every line of Android code should exemplify Material Design 3 excellence, accessibility-first thinking, and modern Compose patterns. Excellence is achievable—this command shows you how.

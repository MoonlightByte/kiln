# Cross-Platform Design Token Mapping

**Purpose**: Convert design tokens between Android (Jetpack Compose) and iOS (SwiftUI) idiomatically
**Version**: 1.0 (February 2025)
**Scope**: Material Design 3 ↔ Human Interface Guidelines

---

## 1. Color Token Mapping

### Primary Brand Colors

| Category | Android (Compose) | iOS (SwiftUI) | Notes |
|----------|-------------------|---------------|-------|
| **Primary** | `MaterialTheme.colorScheme.primary` | `.accentColor` or `Color("Primary")` | Brand color for key actions |
| **On Primary** | `MaterialTheme.colorScheme.onPrimary` | `.white` or custom | Text/icons on primary |
| **Primary Container** | `MaterialTheme.colorScheme.primaryContainer` | `Color("PrimaryContainer")` | Large color areas |
| **On Primary Container** | `MaterialTheme.colorScheme.onPrimaryContainer` | `Color("OnPrimaryContainer")` | Text on container |

### Secondary & Tertiary Colors

| Category | Android (Compose) | iOS (SwiftUI) | Notes |
|----------|-------------------|---------------|-------|
| **Secondary** | `MaterialTheme.colorScheme.secondary` | `.secondary` or `Color("Secondary")` | Supporting brand color |
| **On Secondary** | `MaterialTheme.colorScheme.onSecondary` | `.white` or custom | Text on secondary |
| **Tertiary** | `MaterialTheme.colorScheme.tertiary` | `Color("Tertiary")` | Accent color for emphasis |
| **On Tertiary** | `MaterialTheme.colorScheme.onTertiary` | `.white` or custom | Text on tertiary |

### Surface & Background Colors

| Category | Android (Compose) | iOS (SwiftUI) | Notes |
|----------|-------------------|---------------|-------|
| **Surface** | `MaterialTheme.colorScheme.surface` | `Color(.systemBackground)` | Main content background |
| **On Surface** | `MaterialTheme.colorScheme.onSurface` | `Color(.label)` | Primary text color |
| **Surface Variant** | `MaterialTheme.colorScheme.surfaceVariant` | `Color(.systemGray6)` | Secondary background |
| **On Surface Variant** | `MaterialTheme.colorScheme.onSurfaceVariant` | `Color(.secondaryLabel)` | Secondary text |
| **Background** | `MaterialTheme.colorScheme.background` | `Color(.systemBackground)` | Page background |
| **On Background** | `MaterialTheme.colorScheme.onBackground` | `Color(.label)` | Text on background |

### Semantic & State Colors

| Category | Android (Compose) | iOS (SwiftUI) | Notes |
|----------|-------------------|---------------|-------|
| **Error** | `MaterialTheme.colorScheme.error` | `Color(.systemRed)` | Error states |
| **On Error** | `MaterialTheme.colorScheme.onError` | `.white` | Text on error |
| **Error Container** | `MaterialTheme.colorScheme.errorContainer` | `Color(.systemRed).opacity(0.2)` | Error backgrounds |
| **On Error Container** | `MaterialTheme.colorScheme.onErrorContainer` | `Color(.systemRed)` | Text on error bg |
| **Success** | Custom: `Color(0xFF4CAF50)` | `Color(.systemGreen)` | Success states |
| **Warning** | Custom: `Color(0xFFFFC107)` | `Color(.systemOrange)` | Warning states |

### Dark Mode Support

| Context | Android (Compose) | iOS (SwiftUI) | Code Example |
|---------|-------------------|---------------|--------------|
| **Check Dark Mode** | `MaterialTheme.colorScheme` updates based on `darkTheme` param | `@Environment(\.colorScheme)` | See below |
| **Force Light** | N/A (M3 handles automatically) | `.preferredColorScheme(.light)` | |
| **Force Dark** | N/A | `.preferredColorScheme(.dark)` | |

#### Dark Mode Detection

**Android**:
```kotlin
@Composable
fun MyScreen() {
    val colorScheme = if (isSystemInDarkTheme()) {
        darkColorScheme
    } else {
        lightColorScheme
    }

    MaterialTheme(colorScheme = colorScheme) {
        Surface(color = MaterialTheme.colorScheme.surface) {
            Text(text = "Hello", color = MaterialTheme.colorScheme.onSurface)
        }
    }
}
```

**iOS**:
```swift
struct MyScreen: View {
    @Environment(\.colorScheme) var colorScheme

    var body: some View {
        ZStack {
            Color(.systemBackground)  // Adapts to dark mode

            Text("Hello")
                .foregroundColor(.label)  // Adapts to dark mode
        }
    }
}
```

---

## 2. Typography Token Mapping

### Font Styles

| Use Case | Android (Compose) | iOS (SwiftUI) | Size (Android/iOS) |
|----------|-------------------|---------------|-------------------|
| **Hero Headlines** | `MaterialTheme.typography.displayLarge` | `.largeTitle` | 57sp / 34pt |
| **Page Titles** | `MaterialTheme.typography.displayMedium` | `.title` | 45sp / 28pt |
| **Section Headers** | `MaterialTheme.typography.displaySmall` | `.title2` | 36sp / 22pt |
| **Card Titles** | `MaterialTheme.typography.headlineLarge` | `.headline` | 32sp / 17pt |
| **Dialog Titles** | `MaterialTheme.typography.headlineMedium` | `.headline` | 28sp / 17pt |
| **List Headers** | `MaterialTheme.typography.headlineSmall` | `.callout` | 24sp / 16pt |
| **App Bar Titles** | `MaterialTheme.typography.titleLarge` | `.body` | 22sp / 17pt |
| **Tab Labels** | `MaterialTheme.typography.titleMedium` | `.body` | 16sp / 17pt |
| **Button Labels** | `MaterialTheme.typography.labelLarge` | `.body` (bold) | 14sp / 17pt |
| **Body Text** | `MaterialTheme.typography.bodyLarge` | `.body` | 16sp / 17pt |
| **Secondary Text** | `MaterialTheme.typography.bodyMedium` | `.subheadline` | 14sp / 15pt |
| **Captions** | `MaterialTheme.typography.bodySmall` | `.caption` | 12sp / 12pt |

#### Typography Mapping Examples

**Android - Material Design 3**:
```kotlin
@Composable
fun EventDetailScreen() {
    Column(modifier = Modifier.fillMaxWidth()) {
        // Hero title
        Text(
            text = "Critical Role",
            style = MaterialTheme.typography.displayMedium
        )

        // Event details
        Text(
            text = "Campaign: Exandria",
            style = MaterialTheme.typography.bodyLarge
        )

        // Secondary info
        Text(
            text = "Starts in 2 hours",
            style = MaterialTheme.typography.bodyMedium,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )

        // Button text
        Button(onClick = { }) {
            Text(text = "Join Event", style = MaterialTheme.typography.labelLarge)
        }
    }
}
```

**iOS - Human Interface Guidelines**:
```swift
struct EventDetailScreen: View {
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 12) {
                // Hero title
                Text("Critical Role")
                    .font(.title)

                // Event details
                Text("Campaign: Exandria")
                    .font(.body)

                // Secondary info
                Text("Starts in 2 hours")
                    .font(.subheadline)
                    .foregroundColor(.secondary)

                // Button text
                Button(action: { }) {
                    Text("Join Event")
                        .font(.body)
                        .fontWeight(.semibold)
                }
            }
        }
    }
}
```

### Font Weight Mapping

| Weight | Android | iOS | Semantics |
|--------|---------|-----|-----------|
| Light | `FontWeight.Light` (300) | `.light` | Tertiary text |
| Normal | `FontWeight.Normal` (400) | `.regular` | Body text |
| Medium | `FontWeight.Medium` (500) | `.semibold` | Labels, headings |
| Bold | `FontWeight.Bold` (700) | `.bold` | Headlines |

### Line Height & Spacing

| Property | Android (sp) | iOS (pt) | Mapping |
|----------|-------------|---------|---------|
| displayLarge | 64sp | 40pt | Titles, hero content |
| bodyLarge | 24sp | 22pt | Main text blocks |
| bodySmall | 16sp | 18pt | Captions, hints |

---

## 3. Spacing & Layout Token Mapping

### Space Scale (Base Unit: 4dp Android / 4pt iOS)

| Scale | Android (dp) | iOS (pt) | CSS (px) | Use Case |
|-------|-------------|----------|----------|----------|
| **xs** | 4 | 4 | 4 | Inline spacing, between letters |
| **sm** | 8 | 8 | 8 | Related elements, dividers |
| **md** | 12 | 12 | 12 | Group spacing, content padding |
| **lg** | 16 | 16 | 16 | Section spacing, card padding |
| **xl** | 24 | 24 | 24 | Major divisions, outer margins |
| **2xl** | 32 | 32 | 32 | Page margins, large gaps |
| **3xl** | 48 | 48 | 48 | Hero spacing, full-width sections |
| **4xl** | 64 | 64 | 64 | Maximum margins, banner spacing |

#### Spacing Examples

**Android**:
```kotlin
@Composable
fun EventListItem(event: Event) {
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 12.dp)  // lg, md
    ) {
        Row(horizontalArrangement = Arrangement.spacedBy(12.dp)) {  // md
            Image(...)  // Avatar
            Column(verticalArrangement = Arrangement.spacedBy(4.dp)) {  // xs
                Text("Event Title")
                Text("8:00 PM", style = MaterialTheme.typography.bodySmall)
            }
        }
        Spacer(modifier = Modifier.height(8.dp))  // sm
        Text("Description", style = MaterialTheme.typography.bodyMedium)
    }
}
```

**iOS**:
```swift
struct EventListItem: View {
    let event: Event

    var body: some View {
        VStack(alignment: .leading, spacing: 4) {  // xs
            HStack(spacing: 12) {  // md
                Image(systemName: "calendar")
                    .frame(width: 40, height: 40)
                VStack(alignment: .leading, spacing: 4) {  // xs
                    Text("Event Title")
                    Text("8:00 PM")
                        .font(.caption)
                }
            }
            Divider()
            Text("Description")
                .font(.body)
        }
        .padding(.horizontal, 16)  // lg
        .padding(.vertical, 12)  // md
    }
}
```

### Corner Radius Token Mapping

| Token | Android (dp) | iOS (pt) | Use Case |
|-------|-------------|----------|----------|
| **extraSmall** | 4 | 4 | Badges, small chips |
| **small** | 8 | 8 | Small cards, input fields |
| **medium** | 12 | 12 | Cards, dialogs |
| **large** | 16 | 16 | Large sheets, prominent cards |
| **extraLarge** | 28 | 28 | FABs, full sheets, containers |

#### Corner Radius Examples

**Android**:
```kotlin
@Composable
fun EventCard(event: Event) {
    Card(
        shape = RoundedCornerShape(12.dp),  // medium
        modifier = Modifier.fillMaxWidth()
    ) {
        Column(modifier = Modifier.padding(16.dp)) {  // lg padding
            Text(event.name)
            Chip(
                onClick = { },
                shape = RoundedCornerShape(8.dp)  // small
            ) {
                Text(event.category)
            }
        }
    }
}
```

**iOS**:
```swift
struct EventCard: View {
    let event: Event

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(event.name)

            Chip(text: event.category)
                .cornerRadius(8)  // small
        }
        .padding(16)  // lg
        .background(Color(.systemGray6))
        .cornerRadius(12)  // medium
    }
}
```

### Shadow & Elevation Mapping

| Level | Android (dp) | iOS (points) | Shadow | Use Case |
|-------|-------------|------------|--------|----------|
| **0** | 0 | 0 | None | Surface backgrounds |
| **1** | 1 | 1 | 0.5 | Cards, buttons |
| **2** | 3 | 4 | 1 | Elevated cards, FABs |
| **3** | 6 | 8 | 2 | Dialogs, menus |
| **4** | 8 | 12 | 4 | Top app bar, floating sheets |
| **5** | 12 | 16 | 8 | Modals, full-screen sheets |

#### Elevation Examples

**Android**:
```kotlin
@Composable
fun ElevatedCard(content: @Composable () -> Unit) {
    Card(
        elevation = CardDefaults.elevatedCardElevation(
            defaultElevation = 4.dp  // level 2
        )
    ) {
        content()
    }
}
```

**iOS**:
```swift
struct ElevatedCard<Content: View>: View {
    let content: Content

    var body: some View {
        content
            .background(Color(.systemBackground))
            .cornerRadius(12)
            .shadow(radius: 4, y: 2)  // level 2
    }
}
```

### Tonal Elevation & Materials

Android Material 3 uses **tonal elevation** (color shifting), not just shadows. iOS uses **background materials** (blurs). Map these correctly for cross-platform consistency.

#### Concept Difference

| Concept | Android M3 | iOS |
|---------|-----------|-----|
| Surface layering | Tonal color shift | Background blur |
| Depth indication | surfaceContainerLowest → surfaceContainerHighest | .ultraThinMaterial → .thickMaterial |
| Shadow usage | Minimal, only for FAB/dialogs | Rare, prefer materials |

#### Android Tonal Surface Scale

```kotlin
// Lowest elevation (base)
MaterialTheme.colorScheme.surface           // Level 0

// Container surfaces (increasing elevation)
MaterialTheme.colorScheme.surfaceContainerLowest  // Level 1
MaterialTheme.colorScheme.surfaceContainerLow     // Level 2
MaterialTheme.colorScheme.surfaceContainer        // Level 3
MaterialTheme.colorScheme.surfaceContainerHigh    // Level 4
MaterialTheme.colorScheme.surfaceContainerHighest // Level 5

// Usage
Surface(
    color = MaterialTheme.colorScheme.surfaceContainerLow,
    tonalElevation = 1.dp  // Optional: adds tonal shift
) { ... }
```

#### iOS Material Scale

```swift
// Background materials (increasing opacity/blur)
.ultraThinMaterial  // Subtle, most transparent
.thinMaterial       // Light blur
.regularMaterial    // Standard (most common)
.thickMaterial      // Heavy blur
.ultraThickMaterial // Maximum opacity

// Usage
RoundedRectangle(cornerRadius: 16)
    .fill(.regularMaterial)
    .overlay(content)
```

#### Cross-Platform Mapping

| Use Case | Android M3 | iOS |
|----------|------------|-----|
| Base screen | `surface` | `.background` |
| Card on screen | `surfaceContainerLow` | `.regularMaterial` or semantic color |
| Elevated card | `surfaceContainerHigh` | `.thickMaterial` |
| Bottom sheet | `surfaceContainerLow` | `.regularMaterial` |
| Modal/Dialog | `surfaceContainerHighest` | `.ultraThickMaterial` |
| App bar | `surfaceContainer` | `.bar` material |
| Tab bar | `surfaceContainer` | `.bar` material |
| Floating element | `surfaceContainerHigh` + shadow | `.regularMaterial` + shadow |

#### Implementation Examples

**Android Card with Tonal Elevation:**
```kotlin
Card(
    colors = CardDefaults.cardColors(
        containerColor = MaterialTheme.colorScheme.surfaceContainerLow
    ),
    elevation = CardDefaults.cardElevation(
        defaultElevation = 0.dp  // Tonal, not shadow
    )
) { ... }
```

**iOS Card with Material:**
```swift
RoundedRectangle(cornerRadius: 12)
    .fill(.regularMaterial)
    .overlay(
        VStack { ... }
            .padding()
    )
```

#### Shadow Guidelines

| Element | Android | iOS |
|---------|---------|-----|
| Card | No shadow, use tonal | No shadow, use material |
| FAB | 6dp shadow | Subtle shadow |
| Bottom sheet | No shadow | No shadow |
| Dialog | 24dp shadow | Use .ultraThickMaterial |
| Dropdown menu | 8dp shadow | Subtle shadow |

#### Corner Radius Continuity

iOS uses **continuous curves** (squircles), not circular corners:

```swift
// iOS - ALWAYS use .continuous for native feel
RoundedRectangle(cornerRadius: 16, style: .continuous)  // Correct
RoundedRectangle(cornerRadius: 16)  // Defaults to .circular - avoid
```

```kotlin
// Android - Uses circular by default (acceptable)
RoundedCornerShape(16.dp)  // Circular corners
// For iOS-like continuous curves, use custom shape or accept difference
```

---

## 4. Touch Target & Interactivity Token Mapping

### Touch Target Sizes

| Type | Android | iOS | WCAG |
|------|---------|-----|------|
| **Minimum** | 48x48 dp | 44x44 pt | 24x24 CSS px |
| **Recommended** | 48x48 dp | 44x44 pt | 48x48 CSS px |
| **Large Button** | 56x56 dp | 50x50 pt | – |
| **Icon Button** | 48x48 dp | 44x44 pt | – |
| **Checkbox** | 48x48 dp | 44x44 pt | – |

#### Touch Target Implementation

**Android**:
```kotlin
@Composable
fun IconButton(
    onClick: () -> Unit,
    icon: Painter,
    contentDescription: String
) {
    Button(
        onClick = onClick,
        modifier = Modifier
            .size(48.dp)  // 48x48dp minimum
            .minimumInteractiveComponentSize()
    ) {
        Icon(
            painter = icon,
            contentDescription = contentDescription
        )
    }
}
```

**iOS**:
```swift
struct IconButton: View {
    let action: () -> Void
    let icon: String
    let label: String

    var body: some View {
        Button(action: action) {
            Image(systemName: icon)
        }
        .frame(minWidth: 44, minHeight: 44)  // 44x44pt minimum
        .accessibilityLabel(Text(label))
    }
}
```

### Press & Hover States

| State | Android | iOS | Visual |
|-------|---------|-----|--------|
| **Default** | Base color | Base color | Normal |
| **Hovered** | 8% lighter | No change (touch only) | Subtle highlight |
| **Pressed** | 12% overlay | Opacity 0.7 | Clear feedback |
| **Disabled** | 38% opacity | Opacity 0.5 | Grayed out |
| **Focused** | 2-3dp border | 2pt ring | Outline |

#### State Implementation

**Android**:
```kotlin
@Composable
fun InteractiveButton(
    onClick: () -> Unit,
    label: String
) {
    Button(
        onClick = onClick,
        modifier = Modifier
            .height(48.dp)
            .fillMaxWidth(),
        colors = ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.colorScheme.primary,
            contentColor = MaterialTheme.colorScheme.onPrimary,
            disabledContainerColor = MaterialTheme.colorScheme.surface.copy(alpha = 0.38f),
            disabledContentColor = MaterialTheme.colorScheme.onSurface.copy(alpha = 0.38f)
        )
    ) {
        Text(label)
    }
}
```

**iOS**:
```swift
struct InteractiveButton: View {
    let action: () -> Void
    let label: String
    @State var isPressed = false

    var body: some View {
        Button(action: action) {
            Text(label)
        }
        .frame(maxWidth: .infinity)
        .frame(height: 44)
        .background(.accentColor)
        .foregroundColor(.white)
        .cornerRadius(8)
        .opacity(isPressed ? 0.7 : 1.0)
        .scaleEffect(isPressed ? 0.98 : 1.0)
        .onLongPressGesture(
            minimumDuration: 0.001,
            onPressingChanged: { isPressed = $0 }
        )
    }
}
```

### Haptic Feedback Mapping

LLM-generated screens feel "dead" without haptics. Always add haptic feedback to interactive elements.

#### Haptic Types by Interaction

| Interaction | iOS | Android |
|-------------|-----|---------|
| Button Tap | `.impact(.light)` | `HapticFeedbackType.TextHandleMove` |
| Toggle/Switch | `.impact(.medium)` | `HapticFeedbackType.LongPress` |
| Picker Selection | `.selectionChanged()` | `HapticFeedbackType.TextHandleMove` |
| Success | `.notificationOccurred(.success)` | `HapticFeedbackConstants.CONFIRM` |
| Warning | `.notificationOccurred(.warning)` | `HapticFeedbackConstants.REJECT` |
| Error | `.notificationOccurred(.error)` | `HapticFeedbackConstants.REJECT` |
| Long Press | `.impact(.heavy)` | `HapticFeedbackType.LongPress` |
| Drag/Swipe | `.selectionChanged()` on threshold | `performHapticFeedback` on threshold |

#### iOS Implementation

```swift
// Light tap feedback
UIImpactFeedbackGenerator(style: .light).impactOccurred()

// Medium toggle feedback
UIImpactFeedbackGenerator(style: .medium).impactOccurred()

// Selection change (pickers)
UISelectionFeedbackGenerator().selectionChanged()

// Notifications (success/warning/error)
UINotificationFeedbackGenerator().notificationOccurred(.success)
UINotificationFeedbackGenerator().notificationOccurred(.warning)
UINotificationFeedbackGenerator().notificationOccurred(.error)

// SwiftUI modifier
Button("Tap") { ... }
    .sensoryFeedback(.impact(weight: .light), trigger: tapCount)
```

#### Android Implementation

```kotlin
// Get haptic feedback
val haptic = LocalHapticFeedback.current

// Light tap
haptic.performHapticFeedback(HapticFeedbackType.TextHandleMove)

// Long press / toggle
haptic.performHapticFeedback(HapticFeedbackType.LongPress)

// View-based (for custom feedback)
view.performHapticFeedback(HapticFeedbackConstants.CONFIRM)
view.performHapticFeedback(HapticFeedbackConstants.REJECT)

// Usage in Compose
Button(
    onClick = {
        haptic.performHapticFeedback(HapticFeedbackType.TextHandleMove)
        onButtonClick()
    }
) { Text("Tap") }
```

#### When to Add Haptics

| Component | Add Haptics? | Type |
|-----------|--------------|------|
| Primary Button | Yes | Light tap |
| Secondary Button | Yes | Light tap |
| FAB | Yes | Medium |
| Toggle/Switch | Yes | Medium |
| Checkbox | Yes | Light |
| Radio Button | Yes | Light |
| Slider (on drag end) | Yes | Selection |
| Picker (on change) | Yes | Selection |
| Pull-to-Refresh (on trigger) | Yes | Medium |
| Swipe Action (on threshold) | Yes | Medium |
| Delete Action | Yes | Warning |
| Success Toast | Yes | Success |
| Error Toast | Yes | Error |
| Text Field | No | - |
| Scroll | No | - |
| Navigation | No | - |

#### Haptic Frequency Guidelines

Haptics enhance UX when used sparingly. Overuse creates annoyance and drains battery.

##### When to Use Haptics

| Event | Use Haptic? | Type | Rationale |
|-------|-------------|------|-----------|
| Button tap | Yes | Light | Confirms action |
| Form submission | Yes | Success | Completion feedback |
| Error | Yes | Error | Alert user |
| Scroll | NO | - | Would be annoying |
| Every list item render | NO | - | Overwhelming |
| Pull-to-refresh threshold | Yes | Medium | Threshold reached |
| Swipe action threshold | Yes | Medium | Confirms gesture |

##### Frequency Limits

| Constraint | Value | Notes |
|------------|-------|-------|
| **Maximum rate** | 1 haptic per 100ms | 10 per second absolute max |
| **Recommended** | 1 haptic per user action | One tap = one haptic |
| **NEVER** | Continuous during scrolling | Causes user fatigue |
| **NEVER** | On every frame/render | Battery drain, annoying |

##### Debouncing Haptics

Prevent rapid-fire haptics when events fire quickly (e.g., gesture recognizers, scroll thresholds).

**iOS Implementation:**

```swift
class HapticManager {
    static let shared = HapticManager()
    private var lastHapticTime: Date = .distantPast
    private let minimumInterval: TimeInterval = 0.1  // 100ms

    func triggerHaptic(_ style: UIImpactFeedbackGenerator.FeedbackStyle = .light) {
        let now = Date()
        guard now.timeIntervalSince(lastHapticTime) >= minimumInterval else {
            return  // Too soon, skip this haptic
        }
        lastHapticTime = now
        UIImpactFeedbackGenerator(style: style).impactOccurred()
    }

    func triggerSelection() {
        let now = Date()
        guard now.timeIntervalSince(lastHapticTime) >= minimumInterval else { return }
        lastHapticTime = now
        UISelectionFeedbackGenerator().selectionChanged()
    }
}

// Usage
HapticManager.shared.triggerHaptic(.medium)
```

**Android Implementation:**

```kotlin
object HapticDebouncer {
    private var lastHapticTime: Long = 0
    private const val MIN_INTERVAL_MS = 100L  // 100ms minimum between haptics

    fun performHaptic(
        hapticFeedback: HapticFeedback,
        type: HapticFeedbackType = HapticFeedbackType.TextHandleMove
    ) {
        val now = System.currentTimeMillis()
        if (now - lastHapticTime >= MIN_INTERVAL_MS) {
            lastHapticTime = now
            hapticFeedback.performHapticFeedback(type)
        }
        // Else: skip - too soon since last haptic
    }
}

// Usage in Composable
@Composable
fun MyButton(onClick: () -> Unit) {
    val haptic = LocalHapticFeedback.current

    Button(
        onClick = {
            HapticDebouncer.performHaptic(haptic)
            onClick()
        }
    ) {
        Text("Tap Me")
    }
}
```

##### Accessibility Considerations

| Consideration | Implementation |
|---------------|----------------|
| **System settings** | Always check if haptics are enabled at OS level |
| **Visual alternative** | Every haptic event should have visible feedback (color change, animation) |
| **Audio alternative** | Consider optional sound for important haptics (success/error) |
| **User preference** | Provide in-app toggle to disable haptics |

**iOS - Check system setting:**
```swift
// Haptics respect system Vibration setting automatically
// But you can check for Reduce Motion
if UIAccessibility.isReduceMotionEnabled {
    // Consider skipping non-essential haptics
}
```

**Android - Check system setting:**
```kotlin
// Check if haptic feedback is enabled system-wide
val isHapticEnabled = Settings.System.getInt(
    context.contentResolver,
    Settings.System.HAPTIC_FEEDBACK_ENABLED,
    1  // Default to enabled
) == 1

if (isHapticEnabled) {
    haptic.performHapticFeedback(HapticFeedbackType.TextHandleMove)
}
```

##### Anti-Patterns (NEVER Do These)

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Haptic on every scroll offset change | Battery drain, extremely annoying | Only haptic at threshold (e.g., pull-to-refresh trigger point) |
| Haptic in animation loops | Dozens of haptics per second | Remove haptic from animation, add single haptic at animation start/end |
| Haptic without meaningful interaction | Confuses user, wastes battery | Only haptic on user-initiated actions |
| Haptic on background events | Startles user unexpectedly | Haptics only during active use |
| Different haptic on every touch | Inconsistent, chaotic feel | Use consistent haptic type per action category |

**Bad Example - Haptic in scroll listener:**
```kotlin
// WRONG - fires hundreds of times per scroll
LazyColumn(
    modifier = Modifier.nestedScroll(
        object : NestedScrollConnection {
            override fun onPostScroll(...): Offset {
                haptic.performHapticFeedback(...)  // DON'T DO THIS
                return Offset.Zero
            }
        }
    )
)
```

**Good Example - Haptic only at threshold:**
```kotlin
// CORRECT - fires once when pull threshold is reached
var hasTriggeredHaptic by remember { mutableStateOf(false) }

PullToRefreshBox(
    isRefreshing = isRefreshing,
    onRefresh = {
        // Haptic already fired at threshold
        viewModel.refresh()
    }
) {
    // Track pull progress
    LaunchedEffect(pullProgress) {
        if (pullProgress >= 1f && !hasTriggeredHaptic) {
            HapticDebouncer.performHaptic(haptic, HapticFeedbackType.LongPress)
            hasTriggeredHaptic = true
        } else if (pullProgress < 0.5f) {
            hasTriggeredHaptic = false  // Reset for next pull
        }
    }
}
```

### Animation Curves

LLM-generated animations often use default `linear` or `easeInOut` curves which feel mechanical. Production apps use spring animations for natural feel.

#### Why Springs > Easing

| Curve Type | Feel | When to Use |
|------------|------|-------------|
| Linear | Robotic, cheap | Never for UI |
| EaseInOut | Acceptable | Only for opacity/color |
| Spring | Natural, iOS-like | All transforms (scale, position, rotation) |

#### iOS Spring Parameters

```swift
// Standard response times
.spring(response: 0.3, dampingFraction: 0.7)  // Quick, snappy
.spring(response: 0.4, dampingFraction: 0.8)  // Medium, smooth
.spring(response: 0.5, dampingFraction: 0.9)  // Slow, gentle

// Button press (scale down)
.scaleEffect(isPressed ? 0.95 : 1.0)
.animation(.spring(response: 0.2, dampingFraction: 0.6), value: isPressed)

// Sheet/modal appearance
.animation(.spring(response: 0.35, dampingFraction: 0.85), value: isPresented)

// Card flip
.rotation3DEffect(.degrees(isFlipped ? 180 : 0), axis: (x: 0, y: 1, z: 0))
.animation(.spring(response: 0.6, dampingFraction: 0.8), value: isFlipped)
```

#### Android Spring Parameters

```kotlin
// Standard stiffness values
Spring.StiffnessHigh        // 10000f - Very quick
Spring.StiffnessMedium      // 1500f  - Default
Spring.StiffnessMediumLow   // 400f   - Smooth, iOS-like
Spring.StiffnessLow         // 200f   - Gentle
Spring.StiffnessVeryLow     // 50f    - Very slow

// Standard damping values
Spring.DampingRatioNoBouncy       // 1.0f - No overshoot
Spring.DampingRatioLowBouncy      // 0.75f - Slight bounce
Spring.DampingRatioMediumBouncy   // 0.5f - Noticeable bounce
Spring.DampingRatioHighBouncy     // 0.2f - Very bouncy

// Button press animation
val scale by animateFloatAsState(
    targetValue = if (isPressed) 0.95f else 1f,
    animationSpec = spring(
        dampingRatio = Spring.DampingRatioMediumBouncy,
        stiffness = Spring.StiffnessMediumLow
    )
)

// Item appearance (fade + slide)
AnimatedVisibility(
    visible = isVisible,
    enter = fadeIn(spring(stiffness = Spring.StiffnessMediumLow)) +
            slideInVertically(spring(stiffness = Spring.StiffnessMediumLow)),
    exit = fadeOut(spring(stiffness = Spring.StiffnessMedium))
)

// Shared element transition
Modifier.sharedElement(
    state = sharedTransitionScope.rememberSharedContentState("key"),
    animationSpec = spring(
        stiffness = Spring.StiffnessMediumLow,
        dampingRatio = Spring.DampingRatioLowBouncy
    )
)
```

#### Animation Duration Mapping

| Animation Type | iOS | Android |
|----------------|-----|---------|
| Micro (tap feedback) | 0.1-0.15s | 100-150ms |
| Short (toggle, chip) | 0.2-0.25s | 200-250ms |
| Medium (card, expand) | 0.3-0.4s | 300-400ms |
| Long (modal, sheet) | 0.4-0.5s | 400-500ms |

#### Banned Patterns

**Never use:**

```swift
// iOS
.animation(.linear, value: x)
withAnimation(.linear) { }
```

```kotlin
// Android
tween(durationMillis = 300)  // for position/scale
snap()  // for visual transitions
```

**Always prefer:**

```swift
// iOS
.animation(.spring(response: 0.3, dampingFraction: 0.7), value: x)
```

```kotlin
// Android
spring(stiffness = Spring.StiffnessMediumLow)
```

#### Recommended Defaults

| Platform | Button Press | Modal | List Item |
|----------|--------------|-------|-----------|
| iOS | `.spring(response: 0.2, dampingFraction: 0.6)` | `.spring(response: 0.35, dampingFraction: 0.85)` | `.spring(response: 0.3, dampingFraction: 0.8)` |
| Android | `spring(stiffness: MediumLow, damping: Medium)` | `spring(stiffness: MediumLow, damping: Low)` | `spring(stiffness: MediumLow, damping: Low)` |

### Shared Element Transitions

Shared element transitions create visual continuity between screens by animating a common element (like a hero image) from its position in the source screen to its position in the destination screen. This pattern is essential for polished list-to-detail navigation.

#### Android Shared Element Transitions (Jetpack Compose)

Android uses `SharedTransitionLayout` and `AnimatedContent` (or navigation) to coordinate shared element animations.

**Key Components:**

| Component | Purpose |
|-----------|---------|
| `SharedTransitionLayout` | Provides the scope for shared transitions |
| `sharedElement()` | Modifier that marks an element as shared |
| `sharedBounds()` | For elements that change shape/size between screens |
| `AnimatedVisibilityScope` | Required for enter/exit animations |
| `SharedContentState` | Identifies matching elements by key |

**Basic Setup:**

```kotlin
@OptIn(ExperimentalSharedTransitionApi::class)
@Composable
fun AppNavigation() {
    SharedTransitionLayout {
        val navController = rememberNavController()

        NavHost(navController = navController, startDestination = "list") {
            composable("list") {
                EventListScreen(
                    sharedTransitionScope = this@SharedTransitionLayout,
                    animatedVisibilityScope = this@composable,
                    onEventClick = { eventId ->
                        navController.navigate("detail/$eventId")
                    }
                )
            }
            composable("detail/{eventId}") { backStackEntry ->
                val eventId = backStackEntry.arguments?.getString("eventId") ?: ""
                EventDetailScreen(
                    eventId = eventId,
                    sharedTransitionScope = this@SharedTransitionLayout,
                    animatedVisibilityScope = this@composable
                )
            }
        }
    }
}
```

**List Item with Shared Element:**

```kotlin
@OptIn(ExperimentalSharedTransitionApi::class)
@Composable
fun EventListItem(
    event: Event,
    onClick: () -> Unit,
    sharedTransitionScope: SharedTransitionScope,
    animatedVisibilityScope: AnimatedVisibilityScope
) {
    with(sharedTransitionScope) {
        Card(
            onClick = onClick,
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 16.dp, vertical = 8.dp)
        ) {
            Row(modifier = Modifier.padding(12.dp)) {
                // Hero image - shared element
                AsyncImage(
                    model = event.imageUrl,
                    contentDescription = event.title,
                    modifier = Modifier
                        .size(80.dp)
                        .clip(RoundedCornerShape(8.dp))
                        .sharedElement(
                            state = rememberSharedContentState(key = "image-${event.id}"),
                            animatedVisibilityScope = animatedVisibilityScope,
                            boundsTransform = { _, _ ->
                                spring(
                                    stiffness = Spring.StiffnessMediumLow,
                                    dampingRatio = Spring.DampingRatioLowBouncy
                                )
                            }
                        ),
                    contentScale = ContentScale.Crop
                )

                Spacer(modifier = Modifier.width(12.dp))

                Column {
                    // Title - can also be shared
                    Text(
                        text = event.title,
                        style = MaterialTheme.typography.titleMedium,
                        modifier = Modifier.sharedElement(
                            state = rememberSharedContentState(key = "title-${event.id}"),
                            animatedVisibilityScope = animatedVisibilityScope
                        )
                    )
                    Text(
                        text = event.dateTime,
                        style = MaterialTheme.typography.bodySmall
                    )
                }
            }
        }
    }
}
```

**Detail Screen with Shared Element:**

```kotlin
@OptIn(ExperimentalSharedTransitionApi::class)
@Composable
fun EventDetailScreen(
    eventId: String,
    sharedTransitionScope: SharedTransitionScope,
    animatedVisibilityScope: AnimatedVisibilityScope
) {
    val event = // fetch event by ID

    with(sharedTransitionScope) {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .verticalScroll(rememberScrollState())
        ) {
            // Hero image - matches list item
            AsyncImage(
                model = event.imageUrl,
                contentDescription = event.title,
                modifier = Modifier
                    .fillMaxWidth()
                    .height(250.dp)
                    .sharedElement(
                        state = rememberSharedContentState(key = "image-${event.id}"),
                        animatedVisibilityScope = animatedVisibilityScope,
                        boundsTransform = { _, _ ->
                            spring(
                                stiffness = Spring.StiffnessMediumLow,
                                dampingRatio = Spring.DampingRatioLowBouncy
                            )
                        }
                    ),
                contentScale = ContentScale.Crop
            )

            Column(modifier = Modifier.padding(16.dp)) {
                // Title - matches list item
                Text(
                    text = event.title,
                    style = MaterialTheme.typography.headlineMedium,
                    modifier = Modifier.sharedElement(
                        state = rememberSharedContentState(key = "title-${event.id}"),
                        animatedVisibilityScope = animatedVisibilityScope
                    )
                )

                Spacer(modifier = Modifier.height(16.dp))

                // Non-shared content (fades in)
                Text(
                    text = event.description,
                    style = MaterialTheme.typography.bodyLarge,
                    modifier = Modifier.animateEnterExit(
                        enter = fadeIn(spring(stiffness = Spring.StiffnessMediumLow)),
                        exit = fadeOut(spring(stiffness = Spring.StiffnessMedium))
                    )
                )
            }
        }
    }
}
```

**Spring Animation Configuration:**

```kotlin
// Recommended spring for shared elements
val sharedElementSpring = spring<Rect>(
    stiffness = Spring.StiffnessMediumLow,  // 400f - smooth, iOS-like
    dampingRatio = Spring.DampingRatioLowBouncy  // 0.75f - slight overshoot
)

// For more energetic transitions
val bouncySpring = spring<Rect>(
    stiffness = Spring.StiffnessLow,  // 200f
    dampingRatio = Spring.DampingRatioMediumBouncy  // 0.5f
)

// For subtle, professional transitions
val smoothSpring = spring<Rect>(
    stiffness = Spring.StiffnessMediumLow,
    dampingRatio = Spring.DampingRatioNoBouncy  // 1.0f - no overshoot
)
```

#### iOS Shared Element Transitions (SwiftUI)

iOS uses `matchedGeometryEffect` with `@Namespace` to create shared element transitions.

**Key Components:**

| Component | Purpose |
|-----------|---------|
| `@Namespace` | Creates a unique ID namespace for matching elements |
| `matchedGeometryEffect(id:in:)` | Marks elements that should transition together |
| `isSource` | Determines which view's geometry is used as reference |

**Basic Setup:**

```swift
struct ContentView: View {
    @Namespace private var heroAnimation
    @State private var selectedEvent: Event?
    @State private var showDetail = false

    var body: some View {
        ZStack {
            // List view
            if !showDetail {
                EventListView(
                    namespace: heroAnimation,
                    selectedEvent: $selectedEvent,
                    showDetail: $showDetail
                )
            }

            // Detail view
            if showDetail, let event = selectedEvent {
                EventDetailView(
                    event: event,
                    namespace: heroAnimation,
                    showDetail: $showDetail
                )
            }
        }
        .animation(.spring(response: 0.35, dampingFraction: 0.8), value: showDetail)
    }
}
```

**List Item with Matched Geometry:**

```swift
struct EventListItem: View {
    let event: Event
    let namespace: Namespace.ID
    @Binding var selectedEvent: Event?
    @Binding var showDetail: Bool

    var body: some View {
        HStack(spacing: 12) {
            // Hero image - matched geometry
            AsyncImage(url: URL(string: event.imageUrl)) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            } placeholder: {
                Color.gray
            }
            .frame(width: 80, height: 80)
            .clipShape(RoundedRectangle(cornerRadius: 8, style: .continuous))
            .matchedGeometryEffect(
                id: "image-\(event.id)",
                in: namespace,
                isSource: !showDetail  // Source when list is visible
            )

            VStack(alignment: .leading, spacing: 4) {
                // Title - also matched
                Text(event.title)
                    .font(.headline)
                    .matchedGeometryEffect(
                        id: "title-\(event.id)",
                        in: namespace,
                        isSource: !showDetail
                    )

                Text(event.dateTime)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }

            Spacer()
        }
        .padding(12)
        .background(Color(.systemGray6))
        .cornerRadius(12)
        .onTapGesture {
            selectedEvent = event
            showDetail = true
        }
    }
}
```

**Detail View with Matched Geometry:**

```swift
struct EventDetailView: View {
    let event: Event
    let namespace: Namespace.ID
    @Binding var showDetail: Bool

    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 0) {
                // Hero image - matched geometry
                AsyncImage(url: URL(string: event.imageUrl)) { image in
                    image
                        .resizable()
                        .aspectRatio(contentMode: .fill)
                } placeholder: {
                    Color.gray
                }
                .frame(height: 250)
                .clipped()
                .matchedGeometryEffect(
                    id: "image-\(event.id)",
                    in: namespace,
                    isSource: showDetail  // Source when detail is visible
                )

                VStack(alignment: .leading, spacing: 16) {
                    // Title - matched geometry
                    Text(event.title)
                        .font(.title)
                        .fontWeight(.bold)
                        .matchedGeometryEffect(
                            id: "title-\(event.id)",
                            in: namespace,
                            isSource: showDetail
                        )

                    // Non-matched content (fades in)
                    Text(event.description)
                        .font(.body)
                        .opacity(showDetail ? 1 : 0)
                        .animation(
                            .spring(response: 0.4, dampingFraction: 0.8).delay(0.1),
                            value: showDetail
                        )
                }
                .padding(16)
            }
        }
        .background(Color(.systemBackground))
        .ignoresSafeArea(edges: .top)
        .overlay(alignment: .topLeading) {
            Button(action: { showDetail = false }) {
                Image(systemName: "xmark.circle.fill")
                    .font(.title)
                    .foregroundColor(.white)
                    .shadow(radius: 4)
            }
            .padding()
        }
    }
}
```

**Card Expansion Animation (Full Example):**

```swift
struct ExpandableCard: View {
    @Namespace private var cardAnimation
    @State private var isExpanded = false

    var body: some View {
        ZStack {
            if !isExpanded {
                // Collapsed state
                VStack(alignment: .leading) {
                    RoundedRectangle(cornerRadius: 12, style: .continuous)
                        .fill(Color.blue)
                        .matchedGeometryEffect(id: "background", in: cardAnimation)
                        .frame(height: 120)

                    Text("Card Title")
                        .font(.headline)
                        .matchedGeometryEffect(id: "title", in: cardAnimation)
                }
                .frame(width: 200)
                .onTapGesture { isExpanded = true }
            } else {
                // Expanded state
                VStack(alignment: .leading, spacing: 16) {
                    RoundedRectangle(cornerRadius: 20, style: .continuous)
                        .fill(Color.blue)
                        .matchedGeometryEffect(id: "background", in: cardAnimation)
                        .frame(height: 300)

                    Text("Card Title")
                        .font(.title)
                        .matchedGeometryEffect(id: "title", in: cardAnimation)

                    Text("Expanded content that fades in...")
                        .opacity(isExpanded ? 1 : 0)
                }
                .frame(maxWidth: .infinity)
                .padding()
                .background(Color(.systemBackground))
                .onTapGesture { isExpanded = false }
            }
        }
        .animation(.spring(response: 0.35, dampingFraction: 0.85), value: isExpanded)
    }
}
```

#### Cross-Platform Mapping

| Concept | Android (Compose) | iOS (SwiftUI) |
|---------|-------------------|---------------|
| **Shared scope** | `SharedTransitionLayout` | `@Namespace` property wrapper |
| **Element binding** | `Modifier.sharedElement()` | `.matchedGeometryEffect(id:in:)` |
| **ID management** | `rememberSharedContentState(key:)` | ID parameter + namespace |
| **Source control** | Automatic (based on visibility) | `isSource` parameter |
| **Shape morphing** | `Modifier.sharedBounds()` | Implicit (shape animates) |
| **Animation config** | `boundsTransform` parameter | Separate `.animation()` modifier |
| **Visibility scope** | `AnimatedVisibilityScope` | `@State` + conditional rendering |
| **Recommended spring** | `StiffnessMediumLow` + `DampingRatioLowBouncy` | `response: 0.35, dampingFraction: 0.8` |

#### Common Mistakes

**1. Forgetting the Transition Scope**

```kotlin
// WRONG - No SharedTransitionLayout wrapper
@Composable
fun MyScreen() {
    val state = rememberSharedContentState("key")  // Crashes!
    Image(modifier = Modifier.sharedElement(state, ...))
}

// CORRECT - Wrapped in SharedTransitionLayout
SharedTransitionLayout {
    NavHost(...) { ... }
}
```

```swift
// WRONG - Namespace not passed to child views
struct ParentView: View {
    @Namespace private var animation  // Stays here

    var body: some View {
        ChildView()  // Can't access animation namespace!
    }
}

// CORRECT - Pass namespace to children
ChildView(namespace: animation)
```

**2. Mismatched IDs Causing No Animation**

```kotlin
// WRONG - Different keys
// In list:
rememberSharedContentState(key = "image_${event.id}")
// In detail:
rememberSharedContentState(key = "hero_${event.id}")  // Different prefix!

// CORRECT - Identical keys
rememberSharedContentState(key = "image-${event.id}")  // Same in both
```

```swift
// WRONG - Typo in ID
// In list:
.matchedGeometryEffect(id: "image-\(event.id)", in: namespace)
// In detail:
.matchedGeometryEffect(id: "img-\(event.id)", in: namespace)  // Typo!

// CORRECT - Exact match
.matchedGeometryEffect(id: "image-\(event.id)", in: namespace)  // Same in both
```

**3. Performance Issues with Complex Shared Elements**

```kotlin
// WRONG - Heavy content in shared element
Modifier.sharedElement(state, animatedVisibilityScope)
    .drawBehind {
        // Complex custom drawing during transition
        drawPath(complexPath, expensiveBrush)
    }

// CORRECT - Keep shared elements simple
// Use plain Image/AsyncImage for hero transitions
// Move complex decorations to non-shared overlays
```

```swift
// WRONG - Too many matched geometry effects
ForEach(items) { item in
    VStack {
        Image(...)
            .matchedGeometryEffect(id: "img-\(item.id)", in: ns)
        Text(item.title)
            .matchedGeometryEffect(id: "title-\(item.id)", in: ns)
        Text(item.subtitle)
            .matchedGeometryEffect(id: "sub-\(item.id)", in: ns)
        Text(item.description)
            .matchedGeometryEffect(id: "desc-\(item.id)", in: ns)
        // ... 10 more matched elements
    }
}

// CORRECT - Limit to 1-3 key elements per item
// Hero image + title is usually sufficient
```

**4. isSource Confusion (iOS)**

```swift
// WRONG - Both views claim to be source
// List:
.matchedGeometryEffect(id: "hero", in: ns, isSource: true)
// Detail:
.matchedGeometryEffect(id: "hero", in: ns, isSource: true)  // Conflict!

// CORRECT - Toggle based on which view is active
// List (visible when !showDetail):
.matchedGeometryEffect(id: "hero", in: ns, isSource: !showDetail)
// Detail (visible when showDetail):
.matchedGeometryEffect(id: "hero", in: ns, isSource: showDetail)
```

**5. Missing AnimatedVisibilityScope (Android)**

```kotlin
// WRONG - Using sharedElement without animatedVisibilityScope
NavHost(...) {
    composable("list") {
        // No access to AnimatedVisibilityScope
        Image(modifier = Modifier.sharedElement(state, ???))
    }
}

// CORRECT - Receive scope from composable
composable("list") { // this@composable is AnimatedVisibilityScope
    MyScreen(
        animatedVisibilityScope = this@composable,
        sharedTransitionScope = this@SharedTransitionLayout
    )
}
```

#### Best Practices

1. **Keep shared elements simple** - Images and short text work best
2. **Use consistent IDs** - Define ID format once (e.g., `"image-${id}"`) and reuse
3. **Test on slow animations** - Enable "Animator duration scale" in developer options
4. **Limit to 1-3 shared elements** per transition for best performance
5. **Use springs, not linear** - Always configure spring animations for natural feel
6. **Handle back navigation** - Ensure transitions work in both directions
7. **Consider accessibility** - Shared transitions should be subtle, not disorienting

---

## 5. Component Token Mapping

### Button Tokens

| Property | Android | iOS | Value |
|----------|---------|-----|-------|
| Height | `height` | `frame(height:)` | 48dp / 44pt |
| Padding | `padding` | `padding` | 16dp h, 12dp v |
| Border Radius | `shape` | `cornerRadius` | 8dp / 8pt |
| Font | `labelLarge` | `.body` bold | 14sp / 17pt |
| Min Touch Target | `minimumInteractiveComponentSize` | `frame(min:)` | 48x48 / 44x44 |

#### Button Examples

**Android**:
```kotlin
@Composable
fun PrimaryButton(
    onClick: () -> Unit,
    label: String,
    modifier: Modifier = Modifier,
    enabled: Boolean = true
) {
    Button(
        onClick = onClick,
        enabled = enabled,
        modifier = modifier
            .height(48.dp)
            .fillMaxWidth(),
        shape = RoundedCornerShape(8.dp),
        colors = ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.colorScheme.primary
        )
    ) {
        Text(
            text = label,
            style = MaterialTheme.typography.labelLarge
        )
    }
}
```

**iOS**:
```swift
struct PrimaryButton: View {
    let action: () -> Void
    let label: String
    var enabled: Bool = true

    var body: some View {
        Button(action: action) {
            Text(label)
                .font(.body)
                .fontWeight(.semibold)
        }
        .frame(maxWidth: .infinity)
        .frame(height: 48)
        .background(enabled ? .accentColor : .gray)
        .foregroundColor(.white)
        .cornerRadius(8)
        .disabled(!enabled)
    }
}
```

### Text Field Tokens

| Property | Android | iOS | Value |
|----------|---------|-----|-------|
| Height | `height` | `frame(height:)` | 48-56dp / 44-52pt |
| Padding | `padding` | `padding` | 16dp h, 12dp v |
| Border Radius | `shape` | `cornerRadius` | 8dp / 8pt |
| Font | `bodyLarge` | `.body` | 16sp / 17pt |
| Label | `label` | `placeholder` | bodyMedium / caption |

#### Text Field Examples

**Android**:
```kotlin
@Composable
fun TextField(
    value: String,
    onValueChange: (String) -> Unit,
    label: String,
    modifier: Modifier = Modifier,
    isError: Boolean = false
) {
    OutlinedTextField(
        value = value,
        onValueChange = onValueChange,
        label = { Text(label) },
        modifier = modifier
            .height(56.dp)
            .fillMaxWidth(),
        shape = RoundedCornerShape(8.dp),
        isError = isError,
        textStyle = MaterialTheme.typography.bodyLarge
    )
}
```

**iOS**:
```swift
struct TextInputField: View {
    @State var value: String = ""
    let label: String
    var isError: Bool = false

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(label)
                .font(.caption)
                .foregroundColor(.gray)

            TextField("Enter \(label)", text: $value)
                .padding(12)
                .frame(height: 44)
                .background(Color(.systemGray6))
                .cornerRadius(8)
                .border(isError ? Color.red : Color.clear, width: 1)
        }
    }
}
```

---

## 6. Quick Reference Checklists

### When Converting Android to iOS

- [ ] Replace `MaterialTheme.colorScheme.*` with `.systemColor` or custom Color
- [ ] Convert `sp` font sizes to `pt` (roughly 1:1 ratio)
- [ ] Replace `Compose.Column/Row` with `SwiftUI.VStack/HStack`
- [ ] Change `contentDescription` to `accessibilityLabel`
- [ ] Update spacing from `Modifier.padding()` to `.padding()`
- [ ] Replace `elevation` with `.shadow()`
- [ ] Use `@State`, `@StateObject`, `@ObservedObject` appropriately
- [ ] Test with VoiceOver enabled
- [ ] Verify Dynamic Type support (AX1-AX7)

### When Converting iOS to Android

- [ ] Replace system colors with Material Design 3 tokens
- [ ] Convert `pt` sizes to `sp` (roughly 1:1 ratio)
- [ ] Replace `SwiftUI.VStack/HStack` with `Compose.Column/Row`
- [ ] Change `accessibilityLabel` to `contentDescription`
- [ ] Update spacing from `.padding()` to `Modifier.padding()`
- [ ] Replace `.shadow()` with `elevation`
- [ ] Use `remember { mutableStateOf() }`, `rememberCoroutineScope()`
- [ ] Test with TalkBack enabled
- [ ] Verify font scaling support (up to 200%)

---

## 7. Resources

- **Material Design 3**: https://m3.material.io/
- **Apple HIG**: https://developer.apple.com/design/human-interface-guidelines/
- **Material Design Tokens**: https://material-io.github.io/material-tokens/
- **Amazon Style Dictionary**: https://amzn.github.io/style-dictionary/
- **Orbit Design System**: https://github.com/kiwicom/orbit-compose

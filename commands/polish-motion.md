---
name: polish-motion
description: Comprehensive motion design polish - apply cutting-edge animation best practices for Android and iOS
args:
  - name: code
    type: string
    description: Kotlin/Compose or Swift/SwiftUI code to audit for motion polish
    required: true
  - name: platform
    type: string
    enum: [android, ios, both]
    description: Target platform (default: both)
    required: false
  - name: focus
    type: string
    enum: [transitions, micro, accessibility, performance, haptics, all]
    description: Focus area for polish (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Motion Design Polish Command

Polish your mobile animations to match cutting-edge 2025-2026 motion design standards. This command provides comprehensive guidance on Material Motion, SwiftUI spring physics, accessibility compliance, and haptic feedback integration.

## Philosophy

Great motion design is **invisible when done right**. It should:
- Provide **immediate, tangible feedback** to user actions
- Follow **platform conventions** (Material Motion for Android, HIG for iOS)
- Respect **accessibility preferences** (prefers-reduced-motion)
- Enhance **perceived performance** without consuming resources
- Coordinate **visual, audio, and haptic** feedback channels

---

## Android Animation Best Practices (2025)

### Material Motion System

Material Design 3 introduces a **physics-based spring system** that makes animations feel natural and responsive.

#### Duration Scale (ms)
```kotlin
// Material M3 standard durations
50ms   → Fast utility animations (alpha fade, scale)
100ms  → Component animations (switch toggle, checkbox)
150ms  → Standard utility (expand/collapse)
200ms  → Standard transition
250ms  → Emphasized transition
300ms  → Hero/prominence animations
350ms  → Complex multi-stage
400ms  → Large surface transition
500ms  → Full-screen transition
```

**Rule:** Duration scales with **travel distance**. More movement = longer duration.

#### Easing Curves (Jetpack Compose)
```kotlin
// Import from androidx.compose.animation.core
import androidx.compose.animation.core.EaseInOutCubic
import androidx.compose.animation.core.FastOutLinearInEasing
import androidx.compose.animation.core.FastOutSlowInEasing

// Standard Material easing
val standardEasing = FastOutSlowInEasing        // Entrance/exit
val decelerateEasing = FastOutLinearInEasing     // Decelerate utility
val accelerateEasing = LinearOutSlowInEasing     // Accelerate utility
```

**Application:**
```kotlin
// Fade in (entrance)
AnimatedVisibility(
    visible = isVisible,
    enter = fadeIn(animationSpec = tween(200, easing = FastOutSlowInEasing))
)

// Expand content (exit)
AnimatedVisibility(
    visible = isExpanded,
    exit = shrinkOut(animationSpec = tween(150, easing = FastOutSlowInEasing))
)
```

#### Spring Animations (Physics-Based)

Material's spring system has **three opinionated presets**:

```kotlin
// Fast spring (small components)
// damping: 0.7, stiffness: 380
val fastSpring = spring(dampingRatio = 0.7f, stiffness = 380f)

// Default spring (medium components)
// damping: 0.7, stiffness: 100
val defaultSpring = spring(dampingRatio = 0.7f, stiffness = 100f)

// Slow spring (large components, full-screen)
// damping: 0.7, stiffness: 50
val slowSpring = spring(dampingRatio = 0.7f, stiffness = 50f)
```

**When to use:**
```kotlin
// Switch toggle (fast response)
val animatedValue = animateFloatAsState(
    targetValue = if (isOn) 1f else 0f,
    animationSpec = fastSpring
)

// Card appearing (default)
AnimatedVisibility(
    visible = showCard,
    enter = expandVertically(animationSpec = defaultSpring)
)

// Navigation transition (slow, deliberate)
AnimatedContent(
    targetState = currentScreen,
    transitionSpec = {
        slideIntoContainer(AnimatedContentTransitionScope.SlideDirection.Start,
            animationSpec = slowSpring) togetherWith
        slideOutOfContainer(AnimatedContentTransitionScope.SlideDirection.End,
            animationSpec = slowSpring)
    }
)
```

### Jetpack Compose Animation APIs

#### AnimatedVisibility (Content Removal)
```kotlin
// GOOD: Removes from composition when hidden
AnimatedVisibility(
    visible = isVisible,
    enter = fadeIn() + expandVertically(),
    exit = fadeOut() + shrinkVertically()
) {
    // Content only in composition when visible
    Text("Visible content")
}

// Children can have individual enter/exit
AnimatedVisibility(visible = isVisible) {
    Icon(Icons.Default.Star, Modifier.animateEnterExit(
        enter = slideInHorizontally(initialOffsetX = { -it }),
        exit = slideOutHorizontally(targetOffsetX = { -it })
    ))
    Text("Star")
}
```

**Why this matters:** Unlike `Modifier.alpha()`, AnimatedVisibility removes the composable from the layout tree, preventing layout thrashing and improving accessibility (screen readers won't detect hidden items).

#### AnimatedContent (State Transitions)
```kotlin
// Animate between different content based on state
AnimatedContent(
    targetState = currentTab,
    transitionSpec = {
        // Different transitions depending on direction
        if (targetState > initialState) {
            slideIntoContainer(AnimatedContentTransitionScope.SlideDirection.Start)
                togetherWith slideOutOfContainer(AnimatedContentTransitionScope.SlideDirection.End)
        } else {
            slideIntoContainer(AnimatedContentTransitionScope.SlideDirection.End)
                togetherWith slideOutOfContainer(AnimatedContentTransitionScope.SlideDirection.Start)
        }.using(SizeTransform(clip = false))  // Don't clip during size change
    }
) { tab ->
    when (tab) {
        Tab.Home -> HomeContent()
        Tab.Explore -> ExploreContent()
        Tab.Profile -> ProfileContent()
    }
}

// CRITICAL: Use contentKey to ensure content actually changes
// Without contentKey, composables might be reused and not animate
AnimatedContent(
    targetState = selectedFilter,
    transitionSpec = { ... },
    contentKey = { it.id }  // Force animation when ID changes
)
```

**Critical contentKey pattern:**
```kotlin
// WRONG: Might skip animation if content looks same
AnimatedContent(targetState = filterType) { type ->
    EventCard(event = event, filtered = type == filterType)
}

// RIGHT: Unique key forces animation
AnimatedContent(
    targetState = filterType,
    contentKey = { it }  // Unique key per state
) { type ->
    EventCard(event = event, filtered = type == filterType)
}
```

#### updateTransition (Multiple Values)
```kotlin
// Animate multiple values together (colors, positions, scales)
val transition = updateTransition(
    targetState = isSelected,
    label = "CardSelection"
)

val backgroundColor by transition.animateColor(
    transitionSpec = { tween(200) },
    label = "BackgroundColor"
) { if (it) Color.Green else Color.Gray }

val scale by transition.animateFloat(
    transitionSpec = { spring(stiffness = 100f) },
    label = "Scale"
) { if (it) 1.05f else 1f }

Box(
    Modifier
        .background(backgroundColor)
        .scale(scale)
)
```

**Why use updateTransition:** Keeps multiple animations in sync and prevents recomposition during animation phases.

#### animateAsState (Simple Values)
```kotlin
// Single value animation (easiest for simple cases)
val offsetX by animateFloatAsState(
    targetValue = if (isExpanded) 100f else 0f,
    animationSpec = tween(300, easing = FastOutSlowInEasing),
    label = "OffsetX"
)

Box(Modifier.offset(x = offsetX.dp))

// Animate color on tap
val tintColor by animateColorAsState(
    targetValue = if (isFavorite) Color.Red else Color.Gray,
    animationSpec = spring(dampingRatio = 0.8f),
    label = "FavoriteTint"
)

Icon(Icons.Default.Favorite, tint = tintColor)
```

### Performance Optimization

#### Animate in Draw Phase (Most Performant)
```kotlin
// BEST: Animation in draw phase (no recomposition)
var offset by remember { mutableStateOf(0f) }

LaunchedEffect(triggerAnimation) {
    animate(
        initialValue = 0f,
        targetValue = 100f,
        animationSpec = tween(500)
    ) { value, _ ->
        offset = value
    }
}

Box(Modifier
    .offset(x = offset.dp)
    .background(Color.Blue)
)

// BETTER: Use graphicsLayer for transforms
Box(Modifier
    .graphicsLayer(
        translationX = offset,
        scaleX = 0.9f,
        alpha = 0.8f
    )
)

// AVOID: Animating layout properties (forces recomposition)
var padding by remember { mutableStateOf(0.dp) }
Box(Modifier.padding(start = padding))  // BAD: Layout animation
```

#### Lazy Evaluation & Keys
```kotlin
// For animated lists, provide stable keys
LazyColumn {
    items(items, key = { it.id }) { item ->
        AnimatedVisibility(
            visible = item.isVisible,
            enter = slideInHorizontally(),
            exit = slideOutHorizontally()
        ) {
            EventCard(item)
        }
    }
}
```

---

## iOS Animation Best Practices (2025)

### SwiftUI Spring Animations

SwiftUI's spring animation emulates **real physics**: mass, stiffness, and damping.

#### Spring Parameters
```swift
// Default spring (non-bouncy, smooth)
.spring()  // response: 0.55, dampingFraction: 0.825

// Bouncy spring (playful, energetic)
.spring(response: 0.55, dampingFraction: 0.6)

// Snappy spring (responsive, immediate)
.spring(response: 0.3, dampingFraction: 0.7)

// Custom spring for specific feel
.spring(
    response: 0.6,      // Higher = slower, more bouncy
    dampingFraction: 0.75,  // Lower = more bounce, 1.0 = no bounce
    blendDuration: 0.1  // Transition duration
)
```

**Response scale:**
```
0.3s  → Quick, snappy (buttons, toggles)
0.55s → Default spring (standard transitions)
0.7s  → Slow, deliberate (modals, sheets)
```

#### Spring Animation Implementation
```swift
struct EventCard: View {
    @State private var isExpanded = false
    @Namespace private var animation

    var body: some View {
        VStack {
            HStack {
                Text(event.name)
                Spacer()
                Button(action: { withAnimation(.spring(response: 0.6)) {
                    isExpanded.toggle()
                }}) {
                    Image(systemName: "chevron.down")
                        .rotationEffect(.degrees(isExpanded ? 180 : 0))
                }
            }

            if isExpanded {
                Text(event.description)
                    .transition(.asymmetric(
                        insertion: .opacity.combined(with: .move(edge: .top)),
                        removal: .opacity.combined(with: .move(edge: .top))
                    ))
            }
        }
        .withAnimation(.spring()) {
            // Scale on interaction
            scaleEffect(isExpanded ? 1.02 : 1.0)
        }
    }
}
```

#### matchedGeometryEffect (Hero Animations)

The most powerful animation tool for smooth transitions:

```swift
struct EventListDetail: View {
    @State private var selectedEvent: Event?
    @Namespace private var eventNamespace

    var body: some View {
        ZStack {
            // List view
            ScrollView {
                LazyVStack(spacing: 16) {
                    ForEach(events) { event in
                        EventListCard(event: event, namespace: eventNamespace)
                            .onTapGesture {
                                withAnimation(.spring(response: 0.6)) {
                                    selectedEvent = event
                                }
                            }
                    }
                }
            }

            // Detail overlay
            if let selected = selectedEvent {
                EventDetailView(
                    event: selected,
                    namespace: eventNamespace,
                    onClose: { withAnimation(.spring()) { selectedEvent = nil } }
                )
                .transition(.opacity.combined(with: .scale))
                .zIndex(1)
            }
        }
    }
}

struct EventListCard: View {
    let event: Event
    let namespace: Namespace.ID

    var body: some View {
        VStack(alignment: .leading) {
            Text(event.name)
                .matchedGeometryEffect(id: "title_\(event.id)", in: namespace)
            Text(event.location)
                .matchedGeometryEffect(id: "location_\(event.id)", in: namespace)
            Image(event.imageURL)
                .matchedGeometryEffect(id: "image_\(event.id)", in: namespace)
        }
    }
}

struct EventDetailView: View {
    let event: Event
    let namespace: Namespace.ID
    let onClose: () -> Void

    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 16) {
                HStack {
                    Text(event.name)
                        .matchedGeometryEffect(id: "title_\(event.id)", in: namespace)
                    Spacer()
                    Button(action: onClose) {
                        Image(systemName: "xmark")
                    }
                }

                Text(event.location)
                    .matchedGeometryEffect(id: "location_\(event.id)", in: namespace)

                Image(event.imageURL)
                    .matchedGeometryEffect(id: "image_\(event.id)", in: namespace)
                    .frame(height: 300)

                Text(event.description)

                Spacer()
            }
        }
        .background(Color(uiColor: .systemBackground))
    }
}
```

#### Gesture-Driven Animations
```swift
struct SwipeableDismiss: View {
    @State private var offset: CGFloat = 0

    var body: some View {
        ZStack {
            // Content
            VStack { }
                .offset(y: offset)
                .gesture(
                    DragGesture()
                        .onChanged { value in
                            offset = value.translation.height
                        }
                        .onEnded { value in
                            withAnimation(.spring(response: 0.4, dampingFraction: 0.8)) {
                                // Dismiss if dragged far enough
                                if value.translation.height > 100 {
                                    offset = 500
                                } else {
                                    offset = 0
                                }
                            }
                        }
                )
        }
    }
}
```

#### Transaction-Based Animation
```swift
// Apply animation to multiple operations
var body: some View {
    VStack {
        Button("Expand All") {
            var transaction = Transaction(animation: .spring(response: 0.6))
            transaction.disablesAnimations = false

            withTransaction(transaction) {
                expandedCards = Set(cards.map { $0.id })
            }
        }
    }
}
```

---

## Accessibility: Reduce Motion (CRITICAL)

**WCAG 2.2 Level AAA requirement:** Users who prefer reduced motion must have animations disabled or significantly simplified.

### Android Implementation (Jetpack Compose)
```kotlin
// Detect user's motion preference
fun LocalAnimationDuration() = if (LocalDensity.current.animationDensity == 0f) {
    0  // Animations disabled
} else {
    200  // Standard duration
}

// Apply conditionally
val duration = LocalAnimationDuration()

AnimatedVisibility(
    visible = isVisible,
    enter = if (duration > 0) fadeIn(animationSpec = tween(duration)) else fadeIn(animationSpec = snap()),
    exit = if (duration > 0) fadeOut(animationSpec = tween(duration)) else fadeOut(animationSpec = snap())
) {
    Text("Content")
}

// Better: Create helper modifier
@Composable
fun MotionSafeModifier.fadeInAnimation(
    durationMs: Int = 200,
    delayMs: Int = 0
): Modifier {
    return if (LocalAnimationDuration() == 0) {
        Modifier.alpha(1f)  // No animation
    } else {
        animateAsState(
            targetValue = 1f,
            animationSpec = tween(durationMs, delayMs)
        )
    }
}
```

### iOS Implementation (SwiftUI)
```swift
import SwiftUI

struct MotionSafeAnimation: View {
    @Environment(\.accessibilityReduceMotion) var reduceMotion

    @State private var isExpanded = false

    var body: some View {
        Button(action: { toggleAnimation() }) {
            Image(systemName: "chevron.down")
                .rotationEffect(
                    .degrees(isExpanded ? 180 : 0),
                    anchor: .center
                )
        }
    }

    private func toggleAnimation() {
        if reduceMotion {
            // Instant change, no animation
            isExpanded.toggle()
        } else {
            // Smooth spring animation
            withAnimation(.spring(response: 0.6, dampingFraction: 0.8)) {
                isExpanded.toggle()
            }
        }
    }
}

// Reusable modifier
struct MotionSafeViewModifier: ViewModifier {
    @Environment(\.accessibilityReduceMotion) var reduceMotion
    let animation: Animation
    @Binding var value: Bool

    func body(content: Content) -> some View {
        content
            .onTapGesture {
                if reduceMotion {
                    value.toggle()  // No animation
                } else {
                    withAnimation(animation) {
                        value.toggle()
                    }
                }
            }
    }
}

extension View {
    func motionSafeAnimation(_ animation: Animation, isActive: Binding<Bool>) -> some View {
        modifier(MotionSafeViewModifier(animation: animation, value: isActive))
    }
}
```

### Testing Motion Preferences
```bash
# Android: Enable animation off in Settings > Developer Options > Animation scale
# OR in Compose, set LocalAnimationDuration to 0

# iOS: Settings > Accessibility > Motion > Reduce Motion = ON
# In SwiftUI preview: .previewEnvironment(\.accessibilityReduceMotion, true)
```

---

## Micro-Interactions & Haptic Feedback

### Android Haptics (Vibration)
```kotlin
// Import from androidx.core:core
import androidx.core.content.ContextCompat
import android.os.VibrationEffect
import android.os.Vibrator

// Get haptics service
val vibrator = context.getSystemService(Context.VIBRATOR_SERVICE) as Vibrator

// Light tap (button press)
vibrator.vibrate(VibrationEffect.createPredefined(
    VibrationEffect.EFFECT_TICK
))

// Medium feedback (event confirmation)
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
    vibrator.vibrate(VibrationEffect.createPredefined(
        VibrationEffect.EFFECT_HEAVY_CLICK
    ))
}

// Custom pattern
val pattern = longArrayOf(0, 100, 100, 100)  // delay, vib, pause, vib
vibrator.vibrate(VibrationEffect.createWaveform(pattern, -1))

// Compose convenience
@Composable
fun HapticButton(onClick: () -> Unit) {
    val context = LocalContext.current
    val vibrator = remember {
        context.getSystemService(Context.VIBRATOR_SERVICE) as Vibrator
    }

    Button(onClick = {
        vibrator.vibrate(
            VibrationEffect.createPredefined(VibrationEffect.EFFECT_TICK)
        )
        onClick()
    }) {
        Text("Tap me")
    }
}
```

### iOS Haptics
```swift
import UIKit

// Light impact (selections, deselections)
let light = UIImpactFeedbackGenerator(style: .light)
light.impactOccurred()

// Medium impact (confirmations)
let medium = UIImpactFeedbackGenerator(style: .medium)
medium.impactOccurred()

// Heavy impact (warnings, completions)
let heavy = UIImpactFeedbackGenerator(style: .heavy)
heavy.impactOccurred()

// Notification feedback (success, warning, error)
let notification = UINotificationFeedbackGenerator()
notification.notificationOccurred(.success)
notification.notificationOccurred(.warning)
notification.notificationOccurred(.error)

// Selection feedback (discrete selection steps)
let selection = UISelectionFeedbackGenerator()
selection.selectionChanged()

// SwiftUI convenience
struct HapticButton: View {
    @State private var feedback = UIImpactFeedbackGenerator(style: .medium)

    var body: some View {
        Button(action: {
            feedback.impactOccurred()
        }) {
            Text("Tap me")
        }
    }
}
```

### Micro-Interaction Patterns

#### Button Press Feedback
```kotlin
// Android
@Composable
fun HapticButton(label: String, onClick: () -> Unit) {
    val context = LocalContext.current
    val vibrator = remember {
        context.getSystemService(Context.VIBRATOR_SERVICE) as Vibrator
    }

    var isPressed by remember { mutableStateOf(false) }

    Button(
        onClick = {
            vibrator.vibrate(VibrationEffect.createPredefined(
                VibrationEffect.EFFECT_TICK
            ))
            onClick()
        },
        modifier = Modifier
            .scale(if (isPressed) 0.95f else 1f)
            .animateAsState(targetValue = if (isPressed) 0.95f else 1f)
    ) {
        Text(label)
    }
}
```

#### Pull-to-Refresh Feedback
```swift
// iOS
struct PullToRefreshView: View {
    @State private var isRefreshing = false
    let feedback = UIImpactFeedbackGenerator(style: .medium)

    var body: some View {
        ScrollView {
            VStack {
                Text("Pull to refresh")
            }
            .refreshable {
                feedback.impactOccurred()
                await refreshData()
                feedback.impactOccurred()
            }
        }
    }
}
```

#### Swipe-to-Delete with Haptics
```kotlin
// Android
@Composable
fun SwipeToDismiss(item: Item, onDelete: () -> Unit) {
    val context = LocalContext.current
    val vibrator = remember {
        context.getSystemService(Context.VIBRATOR_SERVICE) as Vibrator
    }

    var offset by remember { mutableStateOf(0f) }

    Box(
        modifier = Modifier
            .fillMaxWidth()
            .offset(x = offset.dp)
            .pointerInput(Unit) {
                detectHorizontalDragGestures { _, dragAmount ->
                    offset += dragAmount / 100

                    // Haptic feedback at threshold
                    if (offset < -50 && offset > -100) {
                        vibrator.vibrate(VibrationEffect.createPredefined(
                            VibrationEffect.EFFECT_TICK
                        ))
                    }

                    // Delete if swiped far
                    if (offset < -100) {
                        vibrator.vibrate(
                            VibrationEffect.createPredefined(VibrationEffect.EFFECT_HEAVY_CLICK)
                        )
                        onDelete()
                    }
                }
            }
    ) {
        EventCard(item)
    }
}
```

---

## Animation Audit Checklist

### Critical (Must Have)

#### Reduce Motion Compliance
- [ ] **AT-001**: All animations disabled when `prefers-reduced-motion` = true (Android) or `accessibilityReduceMotion` = true (iOS)
- [ ] **AT-002**: Instant state changes without animations when reduce motion is enabled
- [ ] **AT-003**: Non-animated fallback for all entrance/exit transitions
- [ ] **AT-004**: No auto-playing animations (only user-triggered)

#### Duration & Easing
- [ ] **AT-005**: All animations use M3 duration scale (50-500ms based on complexity)
- [ ] **AT-006**: Animations use Material easing curves, not custom/jarring
- [ ] **AT-007**: Spring animations use documented damping/stiffness values
- [ ] **AT-008**: Duration increases with travel distance (law of motion)

### Important (Should Have)

#### Jetpack Compose Specific
- [ ] **AT-009**: AnimatedVisibility used instead of alpha fade for content removal
- [ ] **AT-010**: AnimatedContent includes `contentKey` to force transitions
- [ ] **AT-011**: updateTransition used for multi-value animations (not multiple animateAsState)
- [ ] **AT-012**: Animations occur in draw phase, not layout phase
- [ ] **AT-013**: Stable keys provided to LazyColumn animated items

#### SwiftUI Specific
- [ ] **AT-014**: matchedGeometryEffect used for hero transitions (not custom frame calc)
- [ ] **AT-015**: Spring animations use named response/dampingFraction (not magic numbers)
- [ ] **AT-016**: Gesture animations inherit momentum (user velocity = final velocity)
- [ ] **AT-017**: @Namespace used for shared transitions, not multiple @State values

#### Accessibility & Performance
- [ ] **AT-018**: All animations < 300ms (unless deliberate longer duration)
- [ ] **AT-019**: No flashing/rapid changes > 3 flashes/second (WCAG 2.3.3)
- [ ] **AT-020**: Haptic feedback paired with visual feedback, not standalone
- [ ] **AT-021**: Motion doesn't auto-play on screen load (only on interaction)
- [ ] **AT-022**: Animations don't block scrolling/interaction (use graphicsLayer)

### Warnings (Nice to Have)

#### Polish & UX
- [ ] **AT-023**: Haptic feedback provided for destructive actions (delete, confirm)
- [ ] **AT-024**: Micro-interactions acknowledge every user action (button press, toggle)
- [ ] **AT-025**: Smooth interpolation between gesture states (no snapping)
- [ ] **AT-026**: Consistent motion language across screens (same spring params)

#### Performance
- [ ] **AT-027**: animateAsState() prefers lambda modifiers over property changes
- [ ] **AT-028**: Expensive recompositions avoided during animation playback
- [ ] **AT-029**: No memory leaks from Animate* composables (proper cleanup)
- [ ] **AT-030**: Animations disabled in low-battery/low-perf situations (optional)

---

## Code Review Template

When reviewing motion in code, use this structure:

### Platform: [Android / iOS / Both]

**Animations Found:**
- [ ] Entrance transitions
- [ ] Exit transitions
- [ ] State changes (scale, color, position)
- [ ] Gesture-driven animations
- [ ] Micro-interactions (haptics, sound)

**Compliance Checks:**

#### Critical Issues (Must Fix)
```
❌ ISSUE: AnimatedVisibility missing contentKey parameter
Location: EventCard.kt:42
Impact: Animation might be skipped if content structure is similar
Fix: Add contentKey = { event.id } to AnimatedContent
```

#### Warnings (Should Review)
```
⚠️ WARNING: Duration 400ms for fade-in exceeds M3 standard (200ms)
Location: ExploreScreen.kt:156
Impact: Animation feels sluggish
Suggestion: Reduce to 200ms unless deliberate emphasis needed
```

#### Suggestions (Polish)
```
💡 SUGGESTION: Add haptic feedback to favorite button
Location: EventCard.kt:89
Enhancement: confirmationHaptic() on toggle to acknowledge action
```

---

## Implementation Roadmap

### Phase 1: Foundation (Week 1)
- [ ] Audit all animations for reduce motion compliance
- [ ] Convert alpha fades to AnimatedVisibility (Android)
- [ ] Add motion preferences checks (iOS)
- [ ] Fix duration scale violations (all > 500ms)

### Phase 2: Jetpack Compose (Week 2)
- [ ] Add contentKey to all AnimatedContent
- [ ] Replace multiple animateAsState with updateTransition
- [ ] Optimize animations to draw phase (graphicsLayer)
- [ ] Add spring physics to 3+ animations

### Phase 3: SwiftUI (Week 2)
- [ ] Add matchedGeometryEffect to hero transitions
- [ ] Implement spring animations with correct response/damping
- [ ] Test gesture animations inherit momentum
- [ ] Add transaction-based multi-value animations

### Phase 4: Haptics & Accessibility (Week 3)
- [ ] Add haptic feedback to 5+ interactions
- [ ] Implement prefers-reduced-motion on all screens
- [ ] Test with TalkBack/VoiceOver
- [ ] Verify animation performance on low-end devices

### Phase 5: Polish (Week 4)
- [ ] Establish consistent motion language (same spring params)
- [ ] A/B test micro-interactions with users
- [ ] Optimize for battery/thermal conditions
- [ ] Document animation system in design tokens

---

## Resources

### Official Documentation
- [Material Motion – Material Design 3](https://m3.material.io/styles/motion/overview)
- [Animation modifiers and composables | Jetpack Compose](https://developer.android.com/develop/ui/compose/animation/composables-modifiers)
- [Animate with springs - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10158/)
- [Animation and motion | web.dev (WCAG Accessibility)](https://web.dev/learn/accessibility/motion)

### Advanced Guides
- [Easing and duration – Material Design 3](https://m3.material.io/styles/motion/easing-and-duration)
- [The Meaning, Maths, and Physics of SwiftUI Spring Animation](https://medium.com/@amosgyamfi/the-meaning-maths-and-physics-of-swiftui-spring-animation-amos-gyamfis-manifesto-0044755da208)
- [SwiftUI Animation Masterclass — Springs, Curves & Smooth Motion](https://dev.to/sebastienlato/swiftui-animation-masterclass-springs-curves-smooth-motion-3e4o)
- [Smooth Content Transitions in Jetpack Compose with AnimatedContent](https://medium.com/@stevendetilly/smooth-content-transitions-in-jetpack-compose-with-animatedcontent-7ecb380030bc)

### Accessibility
- [WCAG 2.3.3: Animation from Interactions](https://www.w3.org/WAI/WCAG21/Understanding/animation-from-interactions.html)
- [Animation & motion - UX UI Design accessibility WCAG checklist](https://www.atomica11y.com/accessible-design/animation/)
- [Design accessible animation and movement with code examples](https://blog.pope.tech/2025/12/08/design-accessible-animation-and-movement/)

### Haptics
- [Haptics design principles | Android Developers](https://developer.android.com/develop/ui/views/haptics/haptics-principles)
- [2025 Guide to Haptics: Enhancing Mobile UX with Tactile Feedback](https://saropa-contacts.medium.com/2025-guide-to-haptics-enhancing-mobile-ux-with-tactile-feedback-676dd5937774)
- [UIFeedbackGenerator | Apple Developer](https://developer.apple.com/documentation/uikit/uifeedbackgenerator)

### Hero Animations
- [MatchedGeometryEffect - Part 1 (Hero Animations)](https://swiftui-lab.com/matchedgeometryeffect-part1/)
- [SwiftUI Hero Animations GitHub Examples](https://github.com/swiftui-lab/swiftui-hero-animations)

---

## Quick Reference: When to Use What

| Scenario | Android (Compose) | iOS (SwiftUI) |
|----------|-------------------|---------------|
| **Button press** | `spring(dampingRatio=0.7, stiffness=380)` | `.spring(response: 0.3)` |
| **Reveal content** | `AnimatedVisibility` + `fadeIn()` | `.transition(.opacity)` |
| **Change screens** | `AnimatedContent` + slide | `NavigationLink` + custom transition |
| **Hero transition** | `SharedTransitionLayout` | `matchedGeometryEffect` |
| **List item toggle** | `updateTransition` | `withAnimation(.spring())` |
| **Continuous feedback** | `LaunchedEffect` + `animate()` | Gesture handler with `@State` |
| **User gesture** | `pointerInput` + `LaunchedEffect` | `onChanged` + `withTransaction` |
| **Reduce motion** | Check `LocalAnimationDuration` | Check `accessibilityReduceMotion` |

---

**Your animations should feel like physics, not programming. When done right, users won't notice them – they'll just feel natural.**

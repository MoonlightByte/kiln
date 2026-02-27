---
name: audit-gesture-animation
description: Comprehensive audit of gesture-driven animations for iOS 18+ (keyframeAnimator, PhaseAnimator, gesture velocity, interruptible animations, spring physics, haptic synchronization)
args:
  - name: code
    type: string
    description: SwiftUI code with gesture-driven animations to audit
    required: true
  - name: focus
    type: string
    enum: [velocity-mapping, interruption, keyframe-animator, phase-animator, spring-physics, drag-continuation, simultaneous-gesture, state-machine, haptic-sync, reduce-motion, all]
    description: Specific gesture animation aspect to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Gesture-Driven Animation Audit (iOS 18+)

Comprehensive audit of gesture-driven animations using modern SwiftUI patterns including keyframeAnimator, PhaseAnimator, gesture velocity, interruptible animations, spring physics, and haptic feedback synchronization.

## Overview

iOS 18+ enables powerful gesture-driven animations through:
- **Gesture velocity tracking** - DragGesture.Value.velocity provides precise speed/direction
- **keyframeAnimator** - Plays keyframe animations on trigger changes; automatically interruptible by user gestures
- **PhaseAnimator** - Loops through animation phases; cancels on user gesture
- **Spring physics** - gestureVelocity parameter initializes animation with gesture momentum
- **Interruptible animations** - All SwiftUI animations interruptible by default; gesture interruption maintains smooth transition
- **Haptic feedback synchronization** - Haptic timing coordinated with animation milestones

## Research Sources

This audit is based on:
- [GitHub FluidGroup swiftui-gesture-velocity](https://github.com/FluidGroup/swiftui-gesture-velocity) - Gesture velocity tracking
- [SwiftUI Animations 2025: Physics, Gestures & Advanced Techniques](https://medium.com/@bhumibhuva18/swiftui-animations-in-2025-beyond-basic-transitions-f63db40c7c46) - Velocity-driven animations
- [KeyframeAnimator Documentation](https://developer.apple.com/documentation/swiftui/keyframeanimator) - Keyframe-based animation container
- [WWDC 2023: Wind your way through advanced animations](https://developer.apple.com/videos/play/wwdc2023/10157/) - Advanced animation patterns
- [SwiftUI Animations: Interactive Transitions](https://medium.com/@bhumibhuva18/swiftui-animations-interactive-transitions-gesture-driven-animations-for-modern-apps-b23ea36d64f7) - Gesture-driven patterns

## Critical Principles

### Gesture Velocity to Animation Speed Mapping

When using `DragGesture.Value.velocity`, map the 2D velocity vector to animation duration:

```swift
// velocity in points per second
let speedPts = sqrt(pow(gesture.velocity.width, 2) + pow(gesture.velocity.height, 2))

// Map to animation duration (300-1000ms range)
let baseDuration = 0.3  // 300ms minimum
let durationFromVelocity = max(baseDuration, min(1.0, distance / speedPts))

// Use in animation
withAnimation(.easeOut(duration: durationFromVelocity)) {
    position = targetPosition
}
```

**Key Points:**
- Higher velocity → shorter duration (fling feels snappy)
- Lower velocity → longer duration (slow drag feels controlled)
- Use clamp (min 300ms, max 1000ms) to prevent extreme values
- Calculate from both width and height velocity components

### keyframeAnimator Interruption Behavior

keyframeAnimator automatically stops when user performs gesture:

```swift
@State private var isExpanded = false

var body: some View {
    Rectangle()
        .keyframeAnimator(initialValue: 0, trigger: isExpanded) { content, phase in
            content
                .scaleEffect(phase.value)  // Animates from 0 to 1
        } keyframes: { _ in
            KeyframeTrack(\.self) {
                LinearKeyframe(0, duration: 0)
                SpringKeyframe(1, duration: 0.6, timingCurve: .easeInOut)
            }
        }
        .gesture(
            DragGesture()
                .onChanged { gesture in
                    // User drag interrupts keyframe animation automatically
                    // No explicit cancellation needed
                }
        )
}
```

**Critical Fact:** If user performs gesture during keyframe animation, the animation stops and user has full control. No additional code needed for interruption.

### PhaseAnimator for Looping Gestures

PhaseAnimator loops through phases and stops on gesture:

```swift
@State private var phases: [Int] = Array(0...3)
@State private var showAnimation = true

var body: some View {
    Image(systemName: "heart.fill")
        .phaseAnimator(phases, trigger: showAnimation) { content, phase in
            content
                .scaleEffect(1.0 + CGFloat(phase) * 0.2)
                .opacity(phase == 0 ? 0.5 : 1.0)
        } animation: { phase in
            .easeInOut(duration: 0.3)
        }
        .gesture(
            TapGesture()
                .onEnded { _ in
                    // Tap stops phase animation automatically
                    showAnimation.toggle()
                }
        )
}
```

**Key:** PhaseAnimator pauses/resumes based on trigger state. Gesture can change trigger to stop animation.

### Spring Physics with Gesture Velocity

Spring animations initialized with gesture velocity for natural momentum:

```swift
@State private var offset: CGFloat = 0
@State private var isDragging = false

var body: some View {
    Rectangle()
        .offset(x: offset)
        .gesture(
            DragGesture()
                .onChanged { gesture in
                    isDragging = true
                    offset = gesture.translation.width
                }
                .onEnded { gesture in
                    isDragging = false

                    // Spring animation with gesture velocity
                    let velocity = gesture.velocity.width
                    let targetOffset: CGFloat = velocity > 0 ? 300 : -300

                    withAnimation(
                        .spring(
                            response: 0.5,
                            dampingRatio: 0.7,
                            blendDuration: 0.2
                        )
                    ) {
                        offset = targetOffset
                    }

                    // If you need custom velocity handling:
                    // Use CASpringAnimation with initialVelocity
                }
        )
}
```

**Note:** SwiftUI's `.spring()` modifier doesn't directly accept gesture velocity. For advanced velocity-aware springs, use CASpringAnimation or custom transaction with `tracksVelocity`.

### DragGesture with Animation Continuation

Drag that transitions to spring animation on release:

```swift
@State private var dragOffset: CGFloat = 0
@State private var finalPosition: CGFloat = 0

var body: some View {
    Rectangle()
        .offset(x: dragOffset == 0 ? finalPosition : dragOffset)
        .gesture(
            DragGesture()
                .onChanged { gesture in
                    // Disable animation during drag for immediate feedback
                    withAnimation(.none) {
                        dragOffset = gesture.translation.width
                    }
                }
                .onEnded { gesture in
                    // Calculate final position based on drag
                    let threshold: CGFloat = 50
                    finalPosition = abs(dragOffset) > threshold ?
                        (dragOffset > 0 ? 300 : -300) : 0

                    // Transition to spring animation
                    dragOffset = 0  // Reset drag
                    withAnimation(.spring(response: 0.5, dampingRatio: 0.7)) {
                        // Spring animation active once dragOffset reset
                    }
                }
        )
}
```

### Interruptible Animations (SwiftUI Default)

All SwiftUI animations are interruptible by default. Changing a state mid-animation smoothly redirects:

```swift
@State private var isExpanded = false

var body: some View {
    Rectangle()
        .frame(width: isExpanded ? 300 : 100)
        .animation(.easeInOut(duration: 0.5), value: isExpanded)
        .gesture(
            TapGesture()
                .onEnded { _ in
                    isExpanded.toggle()  // Smoothly redirect mid-animation
                }
        )
}
```

**Key:** Tapping during expansion smoothly reverses to collapse. No special code needed—it's the default behavior.

### simultaneousGesture vs exclusiveGesture

**simultaneousGesture**: Both gestures recognize together (useful for haptic + drag):

```swift
Rectangle()
    .gesture(
        DragGesture()
            .onChanged { gesture in
                position = gesture.translation
            }
    )
    .simultaneousGesture(
        // Haptic fires while dragging (doesn't interfere with drag)
        DragGesture()
            .onChanged { gesture in
                if gesture.translation.width > 50 {
                    UIImpactFeedbackGenerator(style: .heavy).impactOccurred()
                }
            }
    )
```

**Exclusive gesture**: Only one gesture recognizes (default):

```swift
Rectangle()
    .gesture(
        TapGesture()
            .onEnded { _ in doSomething() }
    )
    .gesture(
        // This gesture is exclusive—only one fires
        LongPressGesture()
            .onEnded { _ in doOtherThing() }
    )
```

### Gesture State Machine Pattern

Coordinate multiple gesture phases with @GestureState:

```swift
@GestureState private var isPressed = false
@State private var isConfirmed = false

var body: some View {
    Rectangle()
        .foregroundColor(isPressed ? .gray : .blue)
        .scaleEffect(isPressed ? 0.9 : 1.0)
        .gesture(
            LongPressGesture(minimumDuration: 1.0)
                .updating($isPressed) { value, state, _ in
                    state = value  // Updates during press
                }
                .onEnded { _ in
                    isConfirmed = true  // Persists after gesture
                }
        )
}
```

**@GestureState**: Resets to initial value when gesture ends (temporary feedback)
**@State**: Persists after gesture ends (permanent effect)

### Haptic Feedback Synchronization

Sync haptics with animation milestones:

```swift
@State private var scale: CGFloat = 1.0

var body: some View {
    Circle()
        .scaleEffect(scale)
        .gesture(
            TapGesture()
                .onEnded { _ in
                    // Haptic fires immediately on gesture recognition
                    UIImpactFeedbackGenerator(style: .medium).impactOccurred()

                    // Animation starts with haptic for synchronized feel
                    withAnimation(.spring(response: 0.5, dampingRatio: 0.6)) {
                        scale = 1.5
                    }

                    // Schedule second haptic at animation midpoint
                    DispatchQueue.main.asyncAfter(deadline: .now() + 0.25) {
                        UIImpactFeedbackGenerator(style: .light).impactOccurred()
                    }
                }
        )
}
```

**Best Practice:** Haptic on gesture recognition (immediate feedback) + optional haptics at animation milestones.

### Reduce Motion Handling

Always check accessibilityReduceMotion for gesture-driven animations:

```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion

var body: some View {
    Rectangle()
        .gesture(
            DragGesture()
                .onEnded { gesture in
                    let animation = reduceMotion ?
                        Animation.none :  // Instant position change
                        .spring(response: 0.5, dampingRatio: 0.7)

                    withAnimation(animation) {
                        position = targetPosition
                    }
                }
        )
}
```

## Platform-Specific Audit Checklist (iOS 18+)

### Velocity Mapping (Critical Priority)
{{#if (or (eq focus "velocity-mapping") (eq focus "all") (not focus))}}
- [ ] **GA-001**: DragGesture.velocity extracted (width, height, or magnitude)
- [ ] **GA-002**: Gesture velocity maps to animation duration (300-1000ms range)
- [ ] **GA-003**: High velocity → shorter duration (snappy fling)
- [ ] **GA-004**: Low velocity → longer duration (controlled drag)
- [ ] **GA-005**: Distance/velocity used for momentum calculation
- [ ] **GA-006**: Velocity clamped to prevent extreme animation speeds
- [ ] **GA-007**: 2D velocity magnitude calculated: sqrt(width² + height²)
{{/if}}

### Interruptible Animations (Critical Priority)
{{#if (or (eq focus "interruption") (eq focus "all") (not focus))}}
- [ ] **GA-008**: All animations use `.animation()` modifier (not Core Animation)
- [ ] **GA-009**: Gesture changes state mid-animation (smooth redirect)
- [ ] **GA-010**: No custom cancellation needed—SwiftUI handles interruption
- [ ] **GA-011**: Animation stops smoothly when gesture starts
- [ ] **GA-012**: Changing state during animation creates transition
- [ ] **GA-013**: No animation lag after user interaction
- [ ] **GA-014**: Multiple rapid gesture taps handled smoothly
{{/if}}

### keyframeAnimator Usage (Critical Priority)
{{#if (or (eq focus "keyframe-animator") (eq focus "all") (not focus))}}
- [ ] **GA-015**: keyframeAnimator used for complex multi-step animations
- [ ] **GA-016**: Trigger value causes animation to restart/play
- [ ] **GA-017**: Gesture during keyframe animation stops animation automatically
- [ ] **GA-018**: No explicit animation cancellation code needed
- [ ] **GA-019**: Keyframes define clear timing points (duration values)
- [ ] **GA-020**: Spring keyframes used for natural feel (not linear)
- [ ] **GA-021**: Animation state preserved when user interrupts
{{/if}}

### PhaseAnimator for Looping (Important Priority)
{{#if (or (eq focus "phase-animator") (eq focus "all") (not focus))}}
- [ ] **GA-022**: PhaseAnimator used for cyclic/looping animations
- [ ] **GA-023**: Phases defined as array (int, string, or custom enum)
- [ ] **GA-024**: Trigger controls animation play/pause state
- [ ] **GA-025**: Gesture changes trigger to stop phase loop
- [ ] **GA-026**: Phase progression timing defined correctly
- [ ] **GA-027**: No phase animation during gesture interaction
- [ ] **GA-028**: Animation resumes when trigger changes back
{{/if}}

### Spring Physics with Gesture Velocity (Important Priority)
{{#if (or (eq focus "spring-physics") (eq focus "all") (not focus))}}
- [ ] **GA-029**: Spring animation initialized at gesture end
- [ ] **GA-030**: response parameter 0.3-0.7 (faster = snappier)
- [ ] **GA-031**: dampingRatio 0.6-0.9 (higher = less bounce)
- [ ] **GA-032**: blendDuration smooth transitions from drag to spring
- [ ] **GA-033**: No oscillation beyond target (over-spring prevented)
- [ ] **GA-034**: Spring feels natural on slow releases
- [ ] **GA-035**: Spring feels snappy on fast flings
- [ ] **GA-036**: Spring animation respects reduce motion preference
{{/if}}

### DragGesture Continuation (Important Priority)
{{#if (or (eq focus "drag-continuation") (eq focus "all") (not focus))}}
- [ ] **GA-037**: onChanged updates position with `.none` animation
- [ ] **GA-038**: onEnded transitions to spring animation
- [ ] **GA-039**: Drag offset tracked separately from final position
- [ ] **GA-040**: Threshold logic determines snap/release behavior
- [ ] **GA-041**: No animation flicker between drag and spring
- [ ] **GA-042**: Dragging state prevents spring animation start
- [ ] **GA-043**: Release velocity influences spring animation duration
- [ ] **GA-044**: Gesture area of effect clearly defined (not entire screen)
{{/if}}

### Simultaneous Gesture Patterns (Important Priority)
{{#if (or (eq focus "simultaneous-gesture") (eq focus "all") (not focus))}}
- [ ] **GA-045**: simultaneousGesture used for non-conflicting gestures (drag + haptic)
- [ ] **GA-046**: Exclusive gestures don't interfere (tap vs long press)
- [ ] **GA-047**: Gesture precedence clear (which gesture wins)
- [ ] **GA-048**: No conflicting gesture handlers on same view
- [ ] **GA-049**: simultaneousGesture allows both to fire
- [ ] **GA-050**: highPriorityGesture overrides default exclusivity
- [ ] **GA-051**: Gesture state properly isolated between handlers
{{/if}}

### Animation Interruption Handling (Important Priority)
{{#if (or (eq focus "interruption") (eq focus "all") (not focus))}}
- [ ] **GA-052**: Gesture interrupts animation without janky jumps
- [ ] **GA-053**: Animation continues from interrupted position
- [ ] **GA-054**: No visual pop/shift when gesture starts
- [ ] **GA-055**: @GestureState resets on gesture end
- [ ] **GA-056**: @State persists after gesture end
- [ ] **GA-057**: Mid-animation gesture feels responsive (< 100ms delay)
- [ ] **GA-058**: Multiple rapid interrupts handled smoothly
{{/if}}

### Gesture State Machine (Important Priority)
{{#if (or (eq focus "state-machine") (eq focus "all") (not focus))}}
- [ ] **GA-059**: @GestureState used for temporary feedback
- [ ] **GA-060**: State transitions clearly defined (pressed → released)
- [ ] **GA-061**: Multiple gesture states don't conflict
- [ ] **GA-062**: Gesture state visible in UI (color change, scale, etc.)
- [ ] **GA-063**: onChanged fires continuously during gesture
- [ ] **GA-064**: onEnded fires exactly once at gesture completion
- [ ] **GA-065**: No orphaned gesture states (all cleaned up)
- [ ] **GA-066**: State machine handles rapid state changes
{{/if}}

### Haptic Feedback Synchronization (Important Priority)
{{#if (or (eq focus "haptic-sync") (eq focus "all") (not focus))}}
- [ ] **GA-067**: UIImpactFeedbackGenerator fires on gesture recognition
- [ ] **GA-068**: Haptic timing coordinated with animation keyframes
- [ ] **GA-069**: Haptic style matches animation intensity (.light, .medium, .heavy)
- [ ] **GA-070**: No haptic feedback during drag (only at milestones)
- [ ] **GA-071**: Haptic fires at 50-300ms mark (early enough to feel linked)
- [ ] **GA-072**: Multiple haptics don't overlap (avoid rapid-fire confusion)
- [ ] **GA-073**: UINotificationFeedbackGenerator used for success/failure
- [ ] **GA-074**: Haptic respects user's haptic settings
{{/if}}

### Accessibility & Reduce Motion (Critical Priority)
{{#if (or (eq focus "reduce-motion") (eq focus "all") (not focus))}}
- [ ] **GA-075**: @Environment(\.accessibilityReduceMotion) checked
- [ ] **GA-076**: Gesture animations instant when reduce motion enabled
- [ ] **GA-077**: State change still communicated (color, icon, text change)
- [ ] **GA-078**: No animation-dependent information (color change only)
- [ ] **GA-079**: Haptic feedback NOT disabled with reduce motion (separate preference)
- [ ] **GA-080**: Gesture still responsive with reduce motion enabled
- [ ] **GA-081**: No parallax or 3D motion during gestures
- [ ] **GA-082**: Touch target size ≥ 44×44pt for all gestures
{{/if}}

### Performance Metrics (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **GA-083**: 60fps minimum during drag animation (120fps on Pro)
- [ ] **GA-084**: No layout recalculation during continuous drag
- [ ] **GA-085**: GPU-accelerated properties only (offset, scale, opacity)
- [ ] **GA-086**: No heavy computation in gesture closures
- [ ] **GA-087**: Gesture response < 100ms (test on iPhone SE)
- [ ] **GA-088**: Memory stable during long drag sequences
- [ ] **GA-089**: Gesture animation completes fully (no forced termination)
{{/if}}

### Edge Cases & Recovery (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **GA-090**: Gesture works across app lifecycle (background → foreground)
- [ ] **GA-091**: Animation recovers gracefully from system interruption
- [ ] **GA-092**: Low Power Mode doesn't break gesture animations
- [ ] **GA-093**: Network requests don't freeze gesture responsiveness
- [ ] **GA-094**: Gesture works with keyboard visible (if applicable)
- [ ] **GA-095**: Long-duration gestures don't timeout
- [ ] **GA-096**: Rapid gesture repeat handled (no memory leaks)
{{/if}}

## Code Examples

### Example 1: Velocity-Mapped Card Dismiss

```swift
@State private var cardOffset: CGFloat = 0
@State private var cardVisible = true

var body: some View {
    ZStack {
        if cardVisible {
            RoundedRectangle(cornerRadius: 12)
                .fill(Color.blue)
                .frame(height: 300)
                .offset(y: cardOffset)
                .gesture(
                    DragGesture()
                        .onChanged { gesture in
                            withAnimation(.none) {
                                cardOffset = gesture.translation.height
                            }
                        }
                        .onEnded { gesture in
                            let distance = abs(cardOffset)
                            let velocity = gesture.velocity.height

                            // Calculate animation duration based on velocity
                            let speedPts = abs(velocity)
                            let baseDuration = 0.3
                            let duration = speedPts > 0 ?
                                max(0.2, min(1.0, distance / speedPts)) : baseDuration

                            if distance > 150 {
                                // Fling dismiss
                                UIImpactFeedbackGenerator(style: .heavy).impactOccurred()
                                withAnimation(.easeOut(duration: duration)) {
                                    cardOffset = gesture.translation.height > 0 ? 500 : -500
                                }
                                DispatchQueue.main.asyncAfter(deadline: .now() + duration) {
                                    cardVisible = false
                                }
                            } else {
                                // Snap back
                                UIImpactFeedbackGenerator(style: .light).impactOccurred()
                                withAnimation(.spring(response: 0.4, dampingRatio: 0.8)) {
                                    cardOffset = 0
                                }
                            }
                        }
                )
        }
    }
}
```

### Example 2: keyframeAnimator with Gesture Interruption

```swift
@State private var isLiked = false

var body: some View {
    Image(systemName: "heart.fill")
        .font(.system(size: 40))
        .foregroundColor(.red)
        .keyframeAnimator(initialValue: CGFloat(1.0), trigger: isLiked) { content, value in
            content
                .scaleEffect(value)
                .opacity(value >= 0.8 ? 1.0 : 0.5)
        } keyframes: { _ in
            KeyframeTrack(\.self) {
                LinearKeyframe(1.0, duration: 0)
                SpringKeyframe(1.3, duration: 0.3, timingCurve: .easeOut)
                LinearKeyframe(1.1, duration: 0.2)
                SpringKeyframe(1.0, duration: 0.3, timingCurve: .easeInOut)
            }
        }
        .gesture(
            TapGesture()
                .onEnded { _ in
                    // Gesture interrupts keyframe animation automatically
                    // No explicit cancellation code needed
                    isLiked.toggle()
                }
        )
}
```

**Note:** Tapping during keyframe animation stops it instantly. User has full control.

### Example 3: Spring Animation with Gesture Momentum

```swift
@State private var offset: CGFloat = 0
@State private var isDragging = false

var body: some View {
    Circle()
        .fill(Color.purple)
        .frame(width: 80)
        .offset(x: offset)
        .gesture(
            DragGesture()
                .onChanged { gesture in
                    isDragging = true
                    withAnimation(.none) {
                        offset = gesture.translation.width
                    }
                }
                .onEnded { gesture in
                    isDragging = false

                    let threshold: CGFloat = 50
                    let targetOffset: CGFloat = abs(offset) > threshold ?
                        (offset > 0 ? 300 : -300) : 0

                    // Spring animation with momentum feel
                    let velocity = gesture.velocity.width
                    let response = 0.5
                    let damping = 0.7

                    withAnimation(.spring(response: response, dampingRatio: damping)) {
                        offset = targetOffset
                    }
                }
        )
}
```

### Example 4: Haptic Feedback Synchronized with Animation

```swift
@State private var scale: CGFloat = 1.0

var body: some View {
    Button(action: {}) {
        Image(systemName: "star.fill")
            .font(.title)
            .scaleEffect(scale)
    }
    .gesture(
        TapGesture()
            .onEnded { _ in
                // Immediate haptic feedback
                UIImpactFeedbackGenerator(style: .medium).impactOccurred()

                // Animation with synchronized haptic milestone
                withAnimation(.spring(response: 0.4, dampingRatio: 0.6)) {
                    scale = 1.3
                }

                // Haptic at peak scale (animation midpoint ~200ms)
                DispatchQueue.main.asyncAfter(deadline: .now() + 0.2) {
                    UIImpactFeedbackGenerator(style: .light).impactOccurred()
                }

                // Reset scale
                DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
                    withAnimation(.easeInOut(duration: 0.2)) {
                        scale = 1.0
                    }
                }
            }
    )
}
```

### Example 5: Reduce Motion Handling

```swift
@Environment(\.accessibilityReduceMotion) var reduceMotion
@State private var isExpanded = false

var body: some View {
    VStack {
        if isExpanded {
            Text("Expanded content")
                .transition(.opacity)
        }
    }
    .gesture(
        TapGesture()
            .onEnded { _ in
                if reduceMotion {
                    // Instant change, no animation
                    isExpanded.toggle()
                } else {
                    // Smooth animation
                    withAnimation(.easeInOut(duration: 0.3)) {
                        isExpanded.toggle()
                    }
                }
            }
    )
}
```

## Performance Optimization

### GPU Acceleration

**Use only these properties in gesture animations:**
- `offset(x:y:)` - Transform (GPU accelerated)
- `scaleEffect()` - Transform (GPU accelerated)
- `rotationEffect()` - Transform (GPU accelerated)
- `opacity()` - GPU accelerated

**Avoid during drag:**
- Frame size changes (causes layout recalculation)
- Color changes (can cause render cache miss)
- Text content changes (triggers text layout)

### Gesture Performance Budgets

| Operation | Budget | Notes |
|-----------|--------|-------|
| onChanged callback | < 2ms | Called 60x per second during drag |
| Gesture recognition | < 5ms | Initial tap/drag detection |
| Animation frame | < 16ms | 60fps budget = 16.7ms per frame |
| Haptic generation | < 5ms | Non-blocking |

### Optimization Patterns

```swift
// BAD: Heavy computation in onChanged
.onChanged { gesture in
    let expensiveCalculation = computeSomethingExpensive()  // SLOW
    offset = gesture.translation.width
}

// GOOD: Pre-compute or defer
.onChanged { gesture in
    offset = gesture.translation.width  // Instant feedback
    // Schedule expensive work separately
    DispatchQueue.main.async {
        let result = computeSomethingExpensive()
        // Use result later
    }
}
```

## Testing Checklist

### Manual Testing

- [ ] Tap during animation — gesture interrupts smoothly
- [ ] Rapid taps — no state conflicts
- [ ] Drag velocity slow → animates ~1s, fast → ~300ms
- [ ] Release at threshold → snap animation correct direction
- [ ] Haptic feels synchronized with animation peak
- [ ] Reduce Motion enabled → instant change, no animation
- [ ] Gesture works on iPhone SE (slower device)
- [ ] Gesture works with Bluetooth controller

### Accessibility Testing

- [ ] VoiceOver enabled — gesture still works
- [ ] Haptic disabled — gesture still responsive
- [ ] Reduce Motion enabled — state change still clear
- [ ] Touch targets ≥ 44×44pt
- [ ] Gesture alternative available (if swipe-only)

## Common Issues & Fixes

### Issue: Animation Jank During Gesture

**Symptom:** Frame drops when dragging
**Cause:** Layout recalculation, heavy computation in onChanged
**Fix:** Use transform properties only (offset, scale), move heavy work off gesture thread

```swift
// BAD
.onChanged { gesture in
    size = gesture.translation.width * 2  // Layout change = jank
}

// GOOD
.onChanged { gesture in
    offset = gesture.translation.width
}
```

### Issue: Haptic Fires Multiple Times

**Symptom:** Multiple buzzes from single gesture
**Cause:** Haptic fired in onChanged (called continuously)
**Fix:** Fire haptic only in onEnded or use guard

```swift
// BAD
.onChanged { gesture in
    UIImpactFeedbackGenerator(style: .light).impactOccurred()  // Fires 60x/sec
}

// GOOD
.onEnded { gesture in
    UIImpactFeedbackGenerator(style: .light).impactOccurred()  // Fires once
}
```

### Issue: Animation Doesn't Stop on Gesture

**Symptom:** Animation continues after user taps
**Cause:** Not using interruptible animation modifier
**Fix:** Use `.animation()` modifier instead of Task

```swift
// BAD
.onAppear {
    Task {
        while true {
            // Custom loop continues during gesture
        }
    }
}

// GOOD
.animation(.linear(duration: 2), value: isAnimating)
// Automatically stops on gesture
```

### Issue: Reduce Motion Not Respected

**Symptom:** Animation runs even with Accessibility setting enabled
**Cause:** Not checking @Environment(\.accessibilityReduceMotion)
**Fix:** Check environment value in animation decision

```swift
// BAD
withAnimation(.spring()) {  // Always animates
    state.toggle()
}

// GOOD
let animation = reduceMotion ? Animation.none : .spring()
withAnimation(animation) {
    state.toggle()
}
```

## References & Resources

- [DragGesture & Velocity Tracking](https://github.com/FluidGroup/swiftui-gesture-velocity) - GitHub reference implementation
- [SwiftUI Gesture Velocity Animations 2025](https://medium.com/@bhumibhuva18/swiftui-animations-in-2025-beyond-basic-transitions-f63db40c7c46) - Modern patterns and physics
- [keyframeAnimator Official Docs](https://developer.apple.com/documentation/swiftui/keyframeanimator) - Apple's official reference
- [WWDC 2023: Wind Your Way Through Advanced Animations](https://developer.apple.com/videos/play/wwdc2023/10157/) - Essential techniques
- [Interactive Gesture-Driven Animations](https://medium.com/@bhumibhuva18/swiftui-animations-interactive-transitions-gesture-driven-animations-for-modern-apps-b23ea36d64f7) - Best practices
- [SwiftUI Animation Internals](https://dev.to/sebastienlato/swiftui-animations-internals-transactions-timing-identity-4h88) - Deep dive
- [Reducing Motion for Accessibility](https://developer.apple.com/help/app-store-connect/manage-app-accessibility/reduced-motion-evaluation-criteria/) - A11y guidelines

## Audit Output Format

```markdown
## Gesture Animation Audit: [Component/Screen Name]

### Velocity Mapping Compliance

| Animation | Velocity Used | Duration Range | Status |
|-----------|---------------|-----------------|--------|
| Card dismiss | gesture.velocity.height | 200-800ms | ✅ |
| Spring bounce | gesture.velocity.width | 300-1000ms | ⚠️ |

### keyframeAnimator Usage

| Animation | Interruptible | Trigger | Status |
|-----------|---------------|---------|--------|
| Like heart | ✅ Auto-stops | isLiked | ✅ |
| Loading loop | ❌ Manual cancel | showLoading | ❌ |

### Spring Physics Parameters

| Animation | Response | Damping | Blending | Status |
|-----------|----------|---------|----------|--------|
| Card snap | 0.4 | 0.7 | 0.2 | ✅ |
| Button tap | 0.6 | 0.5 | 0.0 | ❌ |

### Gesture State Management

| State | Type | Reset On | Usage | Status |
|------|------|----------|-------|--------|
| isPressed | @GestureState | Gesture end | Temporary feedback | ✅ |
| selectedCard | @State | Manual | Persistent selection | ✅ |

### Haptic Synchronization

| Milestone | Timing | Style | Synced | Status |
|-----------|--------|-------|--------|--------|
| Gesture start | Immediate | medium | ✅ | ✅ |
| Animation peak | 200ms | light | ⚠️ | ⚠️ |

### Accessibility Compliance

| Criterion | Status | Issues |
|-----------|--------|--------|
| Reduce Motion respected | ✅ | None |
| Touch target ≥ 44×44pt | ⚠️ | Drag area 32×32pt |
| Haptic independent | ✅ | Works with haptics off |
| Gesture alternative | ❌ | Swipe-only, no button |

### Performance Metrics

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Gesture response | < 100ms | 45ms | ✅ |
| Frame rate during drag | 60fps | 58-60fps | ✅ |
| GPU acceleration | Transform only | Using opacity changes | ⚠️ |

### Critical Issues

**Line 45**: keyframeAnimator animation doesn't stop on gesture
```swift
// BAD
.keyframeAnimator(...) {
    // Continues during gesture
}

// GOOD
.keyframeAnimator(...) {
    // User gesture automatically stops—no fix needed
}
```
Severity: 🔴 Critical | Confidence: 95 | Rule: GA-017

---

**Line 32**: Haptic fired on every drag update
```swift
// BAD
.onChanged { gesture in
    UIImpactFeedbackGenerator(style: .light).impactOccurred()  // 60x/sec!
}

// GOOD
.onEnded { gesture in
    UIImpactFeedbackGenerator(style: .light).impactOccurred()  // Once
}
```
Severity: 🔴 Critical | Confidence: 90 | Rule: GA-070

### Important Issues

**Line 18**: Animation not interruptible by gesture
```swift
// BAD
Task {
    while animating {
        scale = scale + 0.01  // Custom loop, won't stop
    }
}

// GOOD
.animation(.linear(duration: 1), value: triggerValue)
// Stops automatically on gesture
```
Severity: 🟠 Important | Confidence: 85 | Rule: GA-008

### Warnings

**Line 72**: Reduce Motion not checked
```swift
// BAD
withAnimation(.spring()) {
    state.toggle()  // Always animates
}

// GOOD
let animation = reduceMotion ? Animation.none : .spring()
withAnimation(animation) {
    state.toggle()
}
```
Severity: 🟡 Warning | Confidence: 80 | Rule: GA-075

### Suggestions

**Line 41**: Consider velocity-mapped duration for faster response
```swift
// Current
withAnimation(.easeOut(duration: 0.5)) { ... }

// Suggested
let duration = max(0.2, min(1.0, distance / abs(velocity)))
withAnimation(.easeOut(duration: duration)) { ... }
```
Severity: 🔵 Suggestion | Confidence: 75 | Rule: GA-002

### Summary

- **Total gesture animation issues**: 8
- **Critical**: 2 | Important: 2 | Warning: 2 | Suggestion: 2
- **Velocity mapping**: 85% compliant
- **Interruption handling**: 70% compliant
- **Haptic synchronization**: 60% compliant
- **Accessibility**: 80% compliant
- **Performance**: 95% (excellent)

### Priority Actions

1. Move haptic fires from onChanged to onEnded
2. Add reduce motion checks to all gesture animations
3. Implement velocity-mapped animation durations
4. Test gesture interruption on slower devices (SE)
5. Add gesture alternatives (not swipe-only)
```

## Summary

This audit ensures gesture-driven animations follow iOS 18+ best practices including:
- Physics-based velocity mapping
- Proper use of keyframeAnimator and PhaseAnimator
- Interruptible animation patterns
- Haptic feedback synchronization
- Accessibility compliance (reduce motion, touch targets)
- High performance (60fps+ during gestures)

Use the checklist items (GA-001 to GA-096) to systematically verify gesture animation quality.

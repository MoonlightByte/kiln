---
name: audit-motion
description: Audit mobile animations and motion design for Material Design 3, iOS HIG, and WCAG accessibility compliance
args:
  - name: code
    type: string
    description: Mobile code (Compose or SwiftUI) with animations to audit
    required: true
  - name: platform
    type: string
    enum: [android, ios, auto]
    description: Target platform (default: auto-detect)
    required: false
  - name: focus
    type: string
    enum: [duration, easing, accessibility, performance, gestures, all]
    description: Specific motion aspect to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Motion Design & Animation Audit

Comprehensive audit of mobile animations against Material Design 3, Apple HIG, and WCAG 2.2 accessibility standards. Ensures animations are performant, accessible, and follow platform guidelines.

## Animation Standards Overview

### Material Design 3 (Android)

**Duration Scale:**
- **Short** (50ms): Microinteractions, small components
- **Medium** (150ms): Standard component transitions, button feedback
- **Long** (250ms): Navigation transitions, large surface changes
- **Extra Long** (300-500ms): Multi-step sequences, dramatic changes

**Easing Curves (Cubic Bézier):**
- **Standard** (0.4, 0, 0.2, 1): Default for most animations - accelerate then decelerate
- **Deceleration** (0, 0, 0.2, 1): Elements entering screen - start fast, slow down
- **Acceleration** (0.4, 0, 1, 1): Elements exiting screen - start slow, speed up
- **Sharp** (0.4, 0, 0.6, 1): Snappy feedback for quick interactions

**Key Principles:**
- Duration scales with distance traveled
- Entrance animations slightly longer than exit
- Large surface changes (200+ dp) need longer durations
- Avoid animations lasting >300ms for snappy feel

### iOS HIG (Apple)

**Duration Guidelines:**
- **Simple transitions** (opacity, position): 100-200ms
- **Complex transitions** (scale, rotation): 200-300ms
- **Drawer/navigation**: 300-400ms
- **Large screen changes**: 300-400ms maximum
- **Do NOT exceed 400ms**: Users perceive slowness

**Timing Curves (CAMediaTimingFunction):**
- **easeInEaseOut** (default): Most animations
- **easeOut**: Entrance animations (deceleration)
- **easeIn**: Exit animations (acceleration)
- **linear**: Continuous/scrolling animations only
- **Custom springs**: (damping 0.7-1.0) for natural feel

**Key Principles:**
- Shorter durations feel more responsive
- Spring animations preferred over rigid easing
- Respect user's Reduce Motion preference
- System provides natural motion with CASpringAnimation

### WCAG 2.2 Animation Accessibility

**WCAG 2.2.2 - Pause, Stop, Hide (Level A)**
- Auto-starting animations > 5 seconds must have pause/stop control
- User can control animation timing

**WCAG 2.3.1 - Three Flashes (Level A)**
- No flashing > 3 times per second
- No seizure-inducing patterns

**WCAG 2.3.3 - Animation from Interactions (Level AAA)**
- Animations triggered by user interaction should be disableable
- Use system `prefers-reduced-motion` preference
- Provide motion-safe alternatives

**Motion Sickness Prevention:**
- Avoid parallax and depth-based motion
- Disable multi-axis motion for vestibular sensitivity
- No vortex or spinning effects without toggle
- Large viewport motion can trigger discomfort

## Platform-Specific Audit Checklist

### Android (Jetpack Compose)

#### Duration Compliance (Critical Priority)
{{#if (or (eq focus "duration") (eq focus "all") (not focus))}}
- [ ] **AM-001**: Microinteractions (ripple, button press) ≤ 100ms
- [ ] **AM-002**: Component transitions (expansion, collapse) 150-250ms
- [ ] **AM-003**: Navigation transitions 250-300ms
- [ ] **AM-004**: No animation exceeds 500ms total
- [ ] **AM-005**: Entrance animations longer than exits
- [ ] **AM-006**: Duration scaled to distance traveled (4dp/ms rule)
{{/if}}

#### Easing Curves (Critical Priority)
{{#if (or (eq focus "easing") (eq focus "all") (not focus))}}
- [ ] **AM-007**: Standard curve (0.4, 0, 0.2, 1) for most transitions
- [ ] **AM-008**: Deceleration curve (0, 0, 0.2, 1) for entrances
- [ ] **AM-009**: Acceleration curve (0.4, 0, 1, 1) for exits
- [ ] **AM-010**: Sharp curve (0.4, 0, 0.6, 1) for quick feedback
- [ ] **AM-011**: No linear easing for non-scrolling animations
- [ ] **AM-012**: Using `animationSpec` with correct `Easing`
{{/if}}

#### Accessibility (Critical Priority)
{{#if (or (eq focus "accessibility") (eq focus "all") (not focus))}}
- [ ] **AM-013**: Animations respect `LocalDensity.current.animationScale`
- [ ] **AM-014**: Check `AccessibilityManager.isEnabled()` for motion sensitivity
- [ ] **AM-015**: Reduce Motion respected - provide instant alternative
- [ ] **AM-016**: No flashing > 3 Hz (test with eye tracking tools)
- [ ] **AM-017**: Auto-playing animations > 5s have pause control
- [ ] **AM-018**: No parallax or depth-based motion in primary flow
{{/if}}

#### Performance (Important Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **AM-019**: 60fps target - no frame drops (measure with GPU profiler)
- [ ] **AM-020**: Using `animateXxx()` composables, not manual state
- [ ] **AM-021**: Heavy computations NOT in animation lambda
- [ ] **AM-022**: GPU-accelerated properties only (opacity, transform)
- [ ] **AM-023**: No layout changes during animation (use `animateContentSize`)
- [ ] **AM-024**: Animations use `remember` for stable state
{{/if}}

#### Gesture-Driven Animations (Important Priority)
{{#if (or (eq focus "gestures") (eq focus "all") (not focus))}}
- [ ] **AM-025**: Swipe gestures have button alternatives
- [ ] **AM-026**: Drag animations are interruptible by user
- [ ] **AM-027**: Spring physics for natural feel (damping 0.7-1.0)
- [ ] **AM-028**: Gesture velocity drives animation speed
- [ ] **AM-029**: No drag gestures required (always have button alternative)
- [ ] **AM-030**: Fling/momentum animations respect reduced motion
{{/if}}

#### Loading State Animations (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **AM-031**: Loading spinners spin at 1 revolution per 1-2 seconds
- [ ] **AM-032**: Skeleton screens fade in smooth (200-300ms)
- [ ] **AM-033**: Progress indicators animate linearly (not eased)
- [ ] **AM-034**: Shimmer effects max 800ms cycle
- [ ] **AM-035**: Loading state is announced to screen readers
{{/if}}

#### Transition Patterns (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **AM-036**: Shared element transitions on navigation
- [ ] **AM-037**: Cross-fade for content swap (150-200ms)
- [ ] **AM-038**: Slide for drawer/navigation panel (250-300ms)
- [ ] **AM-039**: Expand/collapse for accordions (150-250ms)
- [ ] **AM-040**: Enter/exit animations defined with `EnterTransition`/`ExitTransition`
{{/if}}

### iOS (SwiftUI)

#### Duration Compliance (Critical Priority)
{{#if (or (eq focus "duration") (eq focus "all") (not focus))}}
- [ ] **AM-101**: Simple transitions 100-200ms
- [ ] **AM-102**: Complex transitions 200-300ms
- [ ] **AM-103**: Large screen changes 300-400ms max
- [ ] **AM-104**: No animation exceeds 400ms
- [ ] **AM-105**: Entrance animations longer than exits
- [ ] **AM-106**: Using `.animation(.easeInOut(duration: X))`
{{/if}}

#### Easing Curves (Critical Priority)
{{#if (or (eq focus "easing") (eq focus "all") (not focus))}}
- [ ] **AM-107**: Default `.easeInOut` for most motion
- [ ] **AM-108**: `.easeOut` for entrance animations
- [ ] **AM-109**: `.easeIn` for exit animations
- [ ] **AM-110**: `.linear` only for scrolling/continuous motion
- [ ] **AM-111**: Spring animations for natural feel (damping 0.7-1.0)
- [ ] **AM-112**: Custom timing curves via `CABasicAnimation` rare/documented
{{/if}}

#### Accessibility (Critical Priority)
{{#if (or (eq focus "accessibility") (eq focus "all") (not focus))}}
- [ ] **AM-113**: Check `@Environment(\.accessibilityReduceMotion)`
- [ ] **AM-114**: Provide instant alternative when reduce motion enabled
- [ ] **AM-115**: No 3D perspective or depth-based motion
- [ ] **AM-116**: No vortex/spinning effects without toggle
- [ ] **AM-117**: Auto-playing animations > 5s have pause control
- [ ] **AM-118**: Test with VoiceOver during motion
{{/if}}

#### Performance (Important Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **AM-119**: 60fps target on iPhone (120fps on Pro)
- [ ] **AM-120**: Using SwiftUI `.animation()` modifier (not Core Animation)
- [ ] **AM-121**: Heavy computations NOT in animation closure
- [ ] **AM-122**: GPU-accelerated properties only (transform, opacity)
- [ ] **AM-123**: No layout recalculation during animation
- [ ] **AM-124**: Animations complete even on low-power mode
{{/if}}

#### Gesture-Driven Animations (Important Priority)
{{#if (or (eq focus "gestures") (eq focus "all") (not focus))}}
- [ ] **AM-125**: Swipe transitions interruptible by user
- [ ] **AM-126**: Back gesture responsive (iOS 17+)
- [ ] **AM-127**: Interactive transitions with `.transaction()`
- [ ] **AM-128**: Spring physics for drag responses
- [ ] **AM-129**: Pinch-zoom not required (always have button)
- [ ] **AM-130**: Gesture velocity controls animation speed
{{/if}}

#### Loading State Animations (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **AM-131**: Activity indicators use system `.default` style
- [ ] **AM-132**: Custom loaders comply with duration rules
- [ ] **AM-133**: Skeleton screens with subtle animation (200-300ms)
- [ ] **AM-134**: Progress bars animate linearly
- [ ] **AM-135**: Indeterminate progress implies async operation
{{/if}}

#### Transition Patterns (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **AM-136**: Shared element transitions on navigation
- [ ] **AM-137**: Cross-fade for content swap (150-200ms)
- [ ] **AM-138**: Slide for sheet/modal (300-400ms max)
- [ ] **AM-139**: Push/pop navigation respects custom transitions
- [ ] **AM-140**: `matchedGeometryEffect` for hero transitions
{{/if}}

## Code to Audit

```
{{code}}
```

## Output Format

Generate a structured motion audit report:

```markdown
## Motion Audit: [Component/Screen Name]

### Duration Compliance

| Animation | Duration | Standard | Status |
|-----------|----------|----------|--------|
| Button tap feedback | 100ms | ≤150ms | ✅ |
| Navigation transition | 450ms | 250-300ms | ❌ Too long |
| Entrance animation | 100ms | 150ms+ | ⚠️ Too short |

### Easing Curve Compliance

| Animation | Curve Used | Expected | Status |
|-----------|-----------|----------|--------|
| Slide in | linear | deceleration | ❌ Wrong curve |
| Fade out | easeInOut | acceleration | ⚠️ Suboptimal |
| Expand | easeInOut | standard | ✅ Correct |

### Accessibility Compliance

| Criterion | Status | Issues |
|-----------|--------|--------|
| Reduce Motion respected | ❌ | Not checking `animationScale` |
| No 3D motion | ✅ | None detected |
| Flashing rate < 3Hz | ✅ | Max 1Hz found |
| Long animations pauseable | ⚠️ | Spinner > 5s without pause |

### Performance Metrics

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Frame rate | 60fps | 45fps avg | ❌ Drops during transition |
| GPU acceleration | ✅ | transform + opacity | ✅ Correct |
| Layout changes | None during | Detected on line 24 | ❌ Causes jank |

### Critical Issues (🔴)

**Line 45**: Animation duration 800ms exceeds 300ms standard
```kotlin
// BAD
animateFloatAsState(targetValue = 1f, animationSpec = tween(800))

// GOOD
animateFloatAsState(targetValue = 1f, animationSpec = tween(250, easing = StandardEasing))
```
Severity: 🔴 Critical | Confidence: 95 | Rule: AM-001 (Android) / AM-103 (iOS)

---

**Line 32**: No Reduce Motion check - animation runs regardless of accessibility preference
```kotlin
// BAD
val alpha by animateFloatAsState(targetValue = 1f)

// GOOD
val alpha by animateFloatAsState(
    targetValue = 1f,
    animationSpec = if (LocalAnimationScale.current > 0) {
        tween(250)
    } else {
        snap()  // Instant, no animation
    }
)
```
Severity: 🔴 Critical | Confidence: 90 | Rule: AM-013 (Android) / AM-113 (iOS)

---

**Line 58**: Easing curve incorrect - using linear for navigation
```kotlin
// BAD
NavigationTransition(
    duration = 300,
    easing = LinearEasing
)

// GOOD
NavigationTransition(
    duration = 300,
    easing = StandardEasing  // Or DecelEasing for entrance
)
```
Severity: 🔴 Critical | Confidence: 90 | Rule: AM-007 (Android) / AM-107 (iOS)

### Important Issues (🟠)

**Line 18**: Animation duration 350ms too long for entrance
```kotlin
// BAD
animateFloatAsState(
    targetValue = 1f,
    animationSpec = tween(350, easing = DecelerationEasing)
)

// GOOD - Entrance animations 200-250ms
animateFloatAsState(
    targetValue = 1f,
    animationSpec = tween(250, easing = DecelerationEasing)
)
```
Severity: 🟠 Important | Confidence: 85 | Rule: AM-002 (Android) / AM-101 (iOS)

### Warnings (🟡)

**Line 72**: Potential frame drop - layout change during animation
```kotlin
// BAD
Box(modifier = Modifier.size(if (isExpanded) 200.dp else 100.dp))

// GOOD - Use animateContentSize()
Box(
    modifier = Modifier
        .animateContentSize()
        .size(if (isExpanded) 200.dp else 100.dp)
)
```
Severity: 🟡 Warning | Confidence: 80 | Rule: AM-023 (Android) / AM-123 (iOS)

### Suggestions (🔵)

**Line 41**: Consider spring animation for more natural feel
```kotlin
// Current
animateFloatAsState(targetValue = 1f, animationSpec = tween(250))

// Suggested
animateFloatAsState(
    targetValue = 1f,
    animationSpec = spring(dampingRatio = 0.8, stiffness = 100f)
)
```
Severity: 🔵 Suggestion | Confidence: 70 | Rule: Best practice

### Performance Analysis

**GPU Profiling Results:**
- Animation frame rate: 58-60fps (acceptable, slight drops on low-end devices)
- Main thread time: 12ms per frame (good)
- GPU time: 4ms per frame (excellent)
- Memory: No leaks detected

**Recommendations:**
- Consider optimizing line 72 layout change to prevent jank
- Test on Pixel 4a (mid-range performance) for frame drops

### Motion Hierarchy Summary

- **Microinteractions** (50-100ms): Ripple, button feedback
- **Component transitions** (150-250ms): Expansion, collapse, list reorder
- **Navigation transitions** (250-300ms): Screen push, modal entrance
- **Large surface changes** (300-400ms): Drawer open, landscape rotation

### Accessibility Checklist

- [ ] ✅ Reduce Motion preference checked and respected
- [ ] ❌ Long animations (>5s) missing pause control
- [ ] ✅ No flashing patterns detected
- [ ] ✅ No parallax or 3D depth effects
- [ ] ⚠️ Test with VoiceOver/TalkBack during motion

### Summary

- **Total motion issues**: 12
- **Critical**: 3 | Important: 2 | Warning: 2 | Suggestion: 5
- **Duration compliance**: 60%
- **Easing compliance**: 70%
- **Accessibility compliance**: 80%
- **Performance**: 90% (minor optimization needed)

### Priority Actions

1. Fix duration violations: 800ms → 250ms, 350ms → 250ms, 450ms → 300ms
2. Add Reduce Motion check to all animations (CRITICAL for accessibility)
3. Replace linear easing with StandardEasing for navigation
4. Optimize layout change at line 72 to prevent frame drops
5. Add pause/stop control for spinner > 5 seconds
6. Test performance on low-end device (Pixel 4a)

### Next Steps

1. Apply recommended duration fixes
2. Implement Reduce Motion checks system-wide
3. Run performance profiler during animations
4. Test with accessibility tools (TalkBack/VoiceOver)
5. Re-audit motion patterns for consistency
```

## Animation Quality Standards

### 60fps Target (Mandatory)

**Frame Budget:** 16.7ms per frame
- GPU-accelerated properties only (transform, opacity)
- No layout recalculation during animation
- No heavy computation in animation lambda
- Profile with GPU tools to verify

### Material Design 3 Spring Physics

For natural, responsive animations:
```
dampingRatio: 0.7-1.0 (higher = less bounce)
stiffness: 100-300 (higher = faster response)
```

### Meaningful vs Decorative Motion

**Meaningful:** Conveys state change (on/off, open/close, load/complete)
**Decorative:** Pure polish (background shimmer, hover effect)

WCAG allows disabling decorative motion, but meaningful motion should still communicate state—just differently (color change, icon swap, etc.).

## Common Motion Patterns

| Pattern | Android Duration | iOS Duration | Easing | Notes |
|---------|------------------|--------------|--------|-------|
| Button ripple | 100ms | 100-150ms | Deceleration | Micro-interaction |
| Expansion/collapse | 200ms | 200-250ms | Standard | Linear height is OK |
| Navigation slide | 250-300ms | 300-400ms | Deceleration | Entrance, then reverse for exit |
| Modal entrance | 250-300ms | 300-400ms | Deceleration | Entrance slower than exit |
| Loading spinner | 1-2s/rev | 1-2s/rev | Linear | Continuous, not eased |
| Skeleton fade | 200-300ms | 200-300ms | Standard | Subtle, not jarring |
| Shared element | 250-300ms | 300-400ms | Deceleration | Matches navigation timing |

## References

- [Material Design 3 Motion Standards](https://m3.material.io/styles/motion/easing-and-duration)
- [Apple HIG Motion Guidelines](https://developer.apple.com/design/human-interface-guidelines/motion)
- [WCAG 2.2 Animation from Interactions](https://www.w3.org/WAI/WCAG21/Understanding/animation-from-interactions.html)
- [Reducing Motion for Accessibility](https://developer.apple.com/help/app-store-connect/manage-app-accessibility/reduced-motion-evaluation-criteria/)
- [Web Animation Performance (60fps)](https://www.algolia.com/blog/engineering/60-fps-performant-web-animations-for-optimal-ux/)
- [Accessible Animations - Pope Tech](https://blog.pope.tech/2025/12/08/design-accessible-animation-and-movement/)

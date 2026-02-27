---
name: audit-shape-morphing
description: Audit Material 3 Expressive shape morphing implementation in Jetpack Compose for smooth transitions, performance, and accessibility compliance
args:
  - name: code
    type: string
    description: Kotlin/Compose code with shape morphing to audit
    required: true
  - name: platform
    type: string
    enum: [android, ios, auto]
    description: Target platform (default: android)
    required: false
  - name: focus
    type: string
    enum: [shapes, animations, performance, accessibility, all]
    description: Specific aspect to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Material 3 Expressive Shape Morphing Audit

Comprehensive audit of shape morphing implementation against Material Design 3 Expressive specifications. Ensures smooth transitions, 60fps performance, 35-shape compliance, and WCAG accessibility standards.

## Material Design 3 Expressive Shape System

### Overview

Material Design 3 Expressive introduces a revolutionary shape library with **35 distinctive morphing shapes** and physics-based animation capabilities. This enables components to dynamically change shape in response to state changes, creating fluid and expressive interfaces.

**Key Enabler**: `RoundedPolygon` API from `androidx.graphics:graphics-shapes` library combined with shape morphing animations.

### The 35 M3 Expressive Shapes

**Shape Categories:**

#### Basic Shapes (5)
1. **circle** - Perfect circle, 0 corners
2. **rounded_square** - Classic rounded rectangle
3. **squircle** - iOS-style square-to-circle morph
4. **oval** - Horizontal ellipse
5. **pill** - Vertical capsule (tall pill)

#### Angular Shapes (6)
6. **slanted_square** - Diamond-like rotated square
7. **rounded_triangle** - Triangle with softened edges
8. **diamond** - Four-pointed diamond
9. **rounded_pentagon** - Five-sided polygon
10. **hexagon** - Six-sided polygon (can soften edges)
11. **octagon** - Eight-sided polygon

#### Organic/Curved Shapes (8)
12. **arch** - Curved arch/bridge shape
13. **fan** - Fan or wedge shape
14. **clam_shell** - Curved shell-like shape
15. **semi_circle** - Half-circle, flat top
16. **crescent** - Moon/crescent shape
17. **wave** - Undulating wave pattern
18. **blob** - Organic blob shape
19. **teardrop** - Teardrop pointing down

#### Decorative/Playful Shapes (8)
20. **4_leaf_clover** - Four-leaf clover
21. **8_leaf_clover** - Eight-leaf clover
22. **flower** - Multi-petal flower shape
23. **burst** - Starburst with sharp points
24. **soft_burst** - Starburst with rounded points
25. **boom** - Explosive jagged burst
26. **soft_boom** - Explosive but rounded
27. **ghost_ish** - Playful ghost shape

#### Premium/Expressive Shapes (8)
28. **sunny** - Sun with rays
29. **very_sunny** - Sun with more detailed rays
30. **puffy** - Puffy cloud-like shape
31. **puffy_diamond** - Diamond with puffy edges
32. **gem** - Cut gem/crystal faceted
33. **rounded_star** - Star with soft edges
34. **heart** - Classic heart shape
35. **star_burst** - Complex starburst variant

### Shape Morphing Animation Model

**Key API**: `Morph` composable and `RoundedPolygon` shape class

```kotlin
// Create animated shape morph
@Composable
fun ShapeMorphingButton(
    isPressed: Boolean,
    modifier: Modifier = Modifier
) {
    val startShape = RoundedPolygon(
        numVertices = 4,  // Square
        radius = 50f,
        innerRadiusRatio = 0.8f,
        rounding = CornerRounding(0.3f),
        centerX = 100f,
        centerY = 100f
    )

    val endShape = RoundedPolygon(
        numVertices = 12,  // Circle-like
        radius = 50f,
        innerRadiusRatio = 1f,
        rounding = CornerRounding(0.5f),
        centerX = 100f,
        centerY = 100f
    )

    val morph by animateShapeAsState(
        targetShape = if (isPressed) endShape else startShape,
        animationSpec = spring(dampingRatio = 0.8f, stiffness = 100f)
    )

    Canvas(modifier = modifier.size(200.dp)) {
        drawPath(morph.toPath(), color = Color.Blue)
    }
}
```

**Physics Parameters:**
- **dampingRatio**: 0.7-1.0 (higher = less bounce)
- **stiffness**: 50-300 (higher = faster response)
- **visibilityThreshold**: Default 0.001 (when animation stops)

### Implementation Architecture

```kotlin
// Import graphics-shapes library
import androidx.graphics.shapes.CornerRounding
import androidx.graphics.shapes.RoundedPolygon
import androidx.graphics.shapes.toPath

// Key functions:
// - animateShapeAsState() - Animates between RoundedPolygon shapes
// - Morph composable - Direct shape morphing UI element
// - RoundedPolygon.toPath() - Convert shape to Path for drawing
// - CornerRounding - Control corner smoothness
```

---

## Shape Morphing Audit Checklist

### M3 Shape Library (Critical Priority)

{{#if (or (eq focus "shapes") (eq focus "all") (not focus))}}
- [ ] **SM-001**: Using `androidx.graphics:graphics-shapes` library (required for RoundedPolygon)
- [ ] **SM-002**: All custom shapes defined as `RoundedPolygon` instances
- [ ] **SM-003**: Shape parameters normalized (center at 0,0 or consistent offset)
- [ ] **SM-004**: Using 35 M3 Expressive shapes (or documented custom shapes)
- [ ] **SM-005**: Shape vertices count appropriate for morphing target
- [ ] **SM-006**: `CornerRounding` applied for smooth edges
- [ ] **SM-007**: `innerRadiusRatio` consistent between morphing shapes
- [ ] **SM-008**: No hardcoded shape paths - use `RoundedPolygon` API
- [ ] **SM-009**: Shape naming follows M3 convention (e.g., `ShapeRoundedSquare`, `ShapeGhost`)
- [ ] **SM-010**: Shape constants documented with visual reference
{{/if}}

### Shape Morphing Animations (Critical Priority)

{{#if (or (eq focus "animations") (eq focus "all") (not focus))}}
- [ ] **SM-011**: Using `animateShapeAsState()` for shape transitions
- [ ] **SM-012**: Animation duration 200-400ms (longer for complex morphs)
- [ ] **SM-013**: `dampingRatio` between 0.7-1.0 for natural feel
- [ ] **SM-014**: `stiffness` between 50-300 (default ~100)
- [ ] **SM-015**: Spring physics preferred over easing curves
- [ ] **SM-016**: State change triggers shape morph (on/off, pressed/unpressed)
- [ ] **SM-017**: Shape transitions are reversible (morph back to original)
- [ ] **SM-018**: No animation delays between shape morphs
- [ ] **SM-019**: Morphing shapes maintain visual hierarchy/contrast
- [ ] **SM-020**: Animation interruption handled (if state changes mid-morph)
{{/if}}

### RoundedPolygon & Morph API (Critical Priority)

{{#if (or (eq focus "shapes") (eq focus "all") (not focus))}}
- [ ] **SM-021**: `RoundedPolygon` instances created with explicit parameters
- [ ] **SM-022**: `.toPath()` called correctly to convert shape for drawing
- [ ] **SM-023**: `Morph.toPath()` used in Canvas/Path context
- [ ] **SM-024**: Shape center coordinates aligned for smooth morphing
- [ ] **SM-025**: Vertex count differences handled (4 vertices → 12 vertices)
- [ ] **SM-026**: `CornerRounding` values match morphing target shape
- [ ] **SM-027**: Shape radius consistent across morph states
- [ ] **SM-028**: No manual path manipulation after shape creation
- [ ] **SM-029**: Shape morph library version pinned (androidx.graphics v1.0+)
- [ ] **SM-030**: RoundedPolygon parameters documented inline
{{/if}}

### Performance & GPU Acceleration (Important Priority)

{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **SM-031**: Shape morphing targets 60fps (measure with GPU profiler)
- [ ] **SM-032**: Canvas rendering optimized (invalidate only when necessary)
- [ ] **SM-033**: Heavy shape computation wrapped in `remember`
- [ ] **SM-034**: Shape paths cached if reused across frames
- [ ] **SM-035**: No layout changes during shape morph animation
- [ ] **SM-036**: Matrix transformations GPU-accelerated (use `graphicsLayer`)
- [ ] **SM-037**: No memory leaks from repeated shape creation
- [ ] **SM-038**: Animation frame budget < 16.7ms per frame
- [ ] **SM-039**: Profiled on mid-range device (Pixel 6a or equivalent)
- [ ] **SM-040**: Shape morph doesn't trigger recomposition of parent
{{/if}}

### Accessibility (Critical Priority)

{{#if (or (eq focus "accessibility") (eq focus "all") (not focus))}}
- [ ] **SM-041**: Shape morphing respects `LocalAnimationScale.current`
- [ ] **SM-042**: Provide instant shape alternative when Reduce Motion enabled
- [ ] **SM-043**: Shape changes communicate state via color/opacity too
- [ ] **SM-044**: Interactive shape targets are 48x48dp minimum
- [ ] **SM-045**: Semantic role matches morphed shape purpose (button, toggle, etc.)
- [ ] **SM-046**: contentDescription updated if shape conveys meaning
- [ ] **SM-047**: No shape morph dependency for understanding state
- [ ] **SM-048**: Color contrast maintained during shape transition
- [ ] **SM-049**: Screen reader not confused by morphing shapes
- [ ] **SM-050**: Shape morph animation < 500ms for vestibular accessibility
{{/if}}

### Container Transform Patterns (Important Priority)

{{#if (or (eq focus "animations") (eq focus "all") (not focus))}}
- [ ] **SM-051**: Container transform uses shape morph + size + color animations
- [ ] **SM-052**: Shape morph duration matches size/opacity changes (synchronized)
- [ ] **SM-053**: Emphasis through shape change (e.g., square → burst on emphasis)
- [ ] **SM-054**: Shared element transitions preserve shape morphing
- [ ] **SM-055**: Z-index managed during shape morphing to prevent overlap
- [ ] **SM-056**: Shape morph doesn't obscure content during transition
- [ ] **SM-057**: Before/after shape states documented (visual state diagram)
- [ ] **SM-058**: Container transform tested with dynamic content
- [ ] **SM-059**: Shape morph handles multiple simultaneous transformations
- [ ] **SM-060**: Composition stability maintained during morph
{{/if}}

### State Management (Important Priority)

{{#if (or (eq focus "all") (not focus))}}
- [ ] **SM-061**: Shape state hoisted to correct level
- [ ] **SM-062**: Shape state not recreated on recomposition
- [ ] **SM-063**: Animation state stable (`remember` + derivedStateOf)
- [ ] **SM-064**: No circular dependencies in shape morph state
- [ ] **SM-065**: Shape target validation (both shapes valid polygons)
- [ ] **SM-066**: Shape transition logged for debugging (optional)
- [ ] **SM-067**: ViewModel exposes shape state as StateFlow/LiveData
- [ ] **SM-068**: Lifecycle-aware animation (pause on background)
- [ ] **SM-069**: Shape state persisted if needed (rememberSaveable)
- [ ] **SM-070**: Immutable shape data classes used
{{/if}}

### Edge Cases & Error Handling (Important Priority)

{{#if (or (eq focus "all") (not focus))}}
- [ ] **SM-071**: Invalid shape parameters handled (fallback to default)
- [ ] **SM-072**: Shape morph fails gracefully (instant switch fallback)
- [ ] **SM-073**: Null shape targets prevented (validation before animation)
- [ ] **SM-074**: Shape library dependency check at runtime
- [ ] **SM-075**: Theme changes don't break ongoing morphing animation
- [ ] **SM-076**: Device rotation handled during shape morph (animation paused/resumed)
- [ ] **SM-077**: Low memory condition detected (animation disabled)
- [ ] **SM-078**: Animation cleanup on disposal (no memory leaks)
- [ ] **SM-079**: Morph animation cancelled on navigation
- [ ] **SM-080**: Shape morphing works on all M3 color schemes (dark/light)
{{/if}}

### Visual Quality & Design (Suggestion Priority)

{{#if (or (eq focus "all") (not focus))}}
- [ ] **SM-081**: Shape morphing feels natural (not jerky or rushed)
- [ ] **SM-082**: Shape morph creates meaningful visual hierarchy
- [ ] **SM-083**: Morphing shapes align with Material 3 Expressive brand
- [ ] **SM-084**: Shape transitions avoid jarring color shifts
- [ ] **SM-085**: Emphasis shapes (burst, boom) used sparingly
- [ ] **SM-086**: Organic shapes (blob, cloud) used for friendly tone
- [ ] **SM-087**: Geometric shapes (diamond, gem) used for formal tone
- [ ] **SM-088**: Playful shapes (clover, heart) context-appropriate
- [ ] **SM-089**: Shape morph duration matches interaction speed
- [ ] **SM-090**: Visual feedback through shape morphing tested with users
{{/if}}

---

## Code Examples

### Basic Shape Morphing Button

```kotlin
@Composable
fun ShapeMorphingButton(
    onClick: () -> Unit,
    isLoading: Boolean = false,
    modifier: Modifier = Modifier
) {
    var isPressed by remember { mutableStateOf(false) }

    val startShape = RoundedPolygon(
        numVertices = 4,
        radius = 40f,
        rounding = CornerRounding(0.3f)
    )

    val endShape = RoundedPolygon(
        numVertices = 24,
        radius = 40f,
        rounding = CornerRounding(0.5f)
    )

    val morphShape by animateShapeAsState(
        targetShape = if (isPressed) endShape else startShape,
        animationSpec = spring(dampingRatio = 0.8f, stiffness = 100f)
    )

    Canvas(
        modifier = modifier
            .size(80.dp)
            .pointerInput(Unit) {
                detectTapGestures(
                    onPress = {
                        isPressed = true
                        tryAwaitRelease()
                        isPressed = false
                        onClick()
                    }
                )
            }
    ) {
        drawPath(
            path = morphShape.toPath(),
            color = MaterialTheme.colorScheme.primary
        )
    }
}
```

### Container Transform with Shape Morph

```kotlin
@Composable
fun ExpandableCard(
    modifier: Modifier = Modifier
) {
    var isExpanded by remember { mutableStateOf(false) }

    val squareShape = RoundedPolygon(
        numVertices = 4,
        radius = 40f,
        rounding = CornerRounding(0.2f)
    )

    val expandedShape = RoundedPolygon(
        numVertices = 8,
        radius = 60f,
        rounding = CornerRounding(0.3f)
    )

    val morphShape by animateShapeAsState(
        targetShape = if (isExpanded) expandedShape else squareShape,
        animationSpec = spring(dampingRatio = 0.7f, stiffness = 100f)
    )

    val size by animateDpAsState(
        targetValue = if (isExpanded) 300.dp else 100.dp,
        animationSpec = spring(dampingRatio = 0.7f, stiffness = 100f)
    )

    Surface(
        modifier = modifier
            .size(size)
            .clip(morphShape.toOutline())
            .clickable { isExpanded = !isExpanded },
        color = MaterialTheme.colorScheme.primaryContainer
    ) {
        // Content
    }
}
```

### Animated Icon with Shape Morph

```kotlin
@Composable
fun StateChangeIcon(
    isActive: Boolean,
    modifier: Modifier = Modifier
) {
    val playShape = RoundedPolygon(
        numVertices = 3,
        radius = 20f,
        rounding = CornerRounding(0.2f)
    )

    val pauseShape = RoundedPolygon(
        numVertices = 4,
        radius = 20f,
        rounding = CornerRounding(0.1f)
    )

    val morphShape by animateShapeAsState(
        targetShape = if (isActive) pauseShape else playShape,
        animationSpec = spring(dampingRatio = 0.8f, stiffness = 120f)
    )

    Canvas(modifier = modifier.size(40.dp)) {
        drawPath(
            path = morphShape.toPath(),
            color = if (isActive) Color.Red else Color.Green
        )
    }
}
```

### Reduce Motion Support

```kotlin
@Composable
fun AccessibleShapeMorph(
    targetShape: RoundedPolygon,
    modifier: Modifier = Modifier
) {
    val animationScale = LocalAnimationScale.current

    val morphShape by animateShapeAsState(
        targetShape = targetShape,
        animationSpec = if (animationScale > 0) {
            spring(
                dampingRatio = 0.8f,
                stiffness = 100f * animationScale
            )
        } else {
            snap()  // Instant change if Reduce Motion enabled
        }
    )

    Canvas(modifier = modifier.size(100.dp)) {
        drawPath(morphShape.toPath(), color = Color.Blue)
    }
}
```

---

## Performance Considerations

### GPU Acceleration

**DO:**
- Use `graphicsLayer` for matrix transformations
- Keep shape paths cached in `remember`
- Use `Canvas` with GPU-backed rendering
- Animate shape bounds separately from morph

**DON'T:**
- Recreate shape on every recomposition
- Animate non-GPU properties (text, shadows)
- Use shape morphing for layout measurements
- Call expensive computations in animation lambda

### Memory Management

```kotlin
// GOOD: Cache shapes
val squareShape = remember {
    RoundedPolygon(numVertices = 4, radius = 40f)
}

val circleShape = remember {
    RoundedPolygon(numVertices = 32, radius = 40f)
}

// BAD: Recreate on every frame
val shape = RoundedPolygon(numVertices = 4, radius = 40f)
```

### Frame Rate Targeting

- **60fps minimum**: Standard phones
- **120fps preferred**: Flagship devices
- **Measure with**: `ProfiledRecomposition` + GPU profiler
- **Budget**: < 16.7ms per frame (60fps) or < 8.3ms (120fps)

---

## Accessibility Implementation

### Reduce Motion Support

```kotlin
@Composable
fun MorphingComponent(
    targetShape: RoundedPolygon
) {
    val animationScale = LocalAnimationScale.current

    val morphShape by animateShapeAsState(
        targetShape = targetShape,
        animationSpec = if (animationScale > 0f) {
            spring(dampingRatio = 0.8f)
        } else {
            snap()  // Instant, no animation
        }
    )

    // Shape changes but without animation for accessibility
}
```

### State Communication

Shape morphing should NOT be the only way to communicate state:

```kotlin
// GOOD: Shape + Color + Icon
Box(
    modifier = Modifier
        .background(
            color = if (isActive) Color.Green else Color.Gray
        )
        .semantics {
            contentDescription = if (isActive) "Active" else "Inactive"
        }
) {
    // Shape morphs here
    Canvas(/* ... */) { }
}

// BAD: Shape morph only
Canvas(/* shape morphs but no color/text change */) { }
```

### Touch Target Sizing

```kotlin
// GOOD: 48x48dp minimum
Surface(
    modifier = Modifier
        .size(48.dp)  // Meets M3 minimum
        .clip(morphShape)
        .clickable { /* ... */ }
)

// BAD: Too small
Canvas(
    modifier = Modifier
        .size(24.dp)  // Below minimum
        .clickable { /* ... */ }
)
```

---

## Testing Strategy

### Unit Tests

```kotlin
@Test
fun testShapeMorphingAnimation() {
    val startShape = RoundedPolygon(numVertices = 4, radius = 40f)
    val endShape = RoundedPolygon(numVertices = 32, radius = 40f)

    // Verify shape parameters
    assertEquals(4, startShape.numVertices)
    assertEquals(32, endShape.numVertices)
}

@Test
fun testShapePathConversion() {
    val shape = RoundedPolygon(numVertices = 6, radius = 50f)
    val path = shape.toPath()

    // Verify path is valid
    assertNotNull(path)
    assertTrue(path.isEmpty == false)
}
```

### UI Tests

```kotlin
@Test
fun testShapeMorphingPerformance() {
    composeTestRule.setContent {
        ShapeMorphingButton()
    }

    // Measure frame rate during morph
    // Expected: 58-60fps on Pixel 4, 55+ on mid-range
}

@Test
fun testReduceMotionRespected() {
    composeTestRule.setContent {
        CompositionLocalProvider(
            LocalAnimationScale provides 0f
        ) {
            AccessibleShapeMorph()
        }
    }

    // Verify shape changes instantly
}
```

### Visual Quality Tests

- Screenshot comparison during morph
- Vector path smoothness validation
- Contrast ratio verification during transition
- Color drift detection

---

## Output Format

Generate a structured shape morphing audit report:

```markdown
## Shape Morphing Audit: [Component/Screen Name]

### Shape Library Compliance

| Shape Name | Type | Vertices | CornerRounding | Status |
|-----------|------|----------|---|--------|
| startShape | RoundedSquare | 4 | 0.3 | ✅ |
| endShape | Circle-like | 32 | 0.5 | ✅ |
| morphShape | (animated) | - | - | ✅ |

### Animation Metrics

| Metric | Value | Standard | Status |
|--------|-------|----------|--------|
| Duration | 250ms | 200-400ms | ✅ |
| Damping Ratio | 0.8 | 0.7-1.0 | ✅ |
| Stiffness | 100 | 50-300 | ✅ |
| Target FPS | 58-60 | 60fps | ✅ |

### Critical Issues (🔴)

**Line 45**: Shape morph duration too short (100ms)
```kotlin
// BAD
animateShapeAsState(
    targetShape = endShape,
    animationSpec = tween(100)
)

// GOOD - Use spring with proper damping
animateShapeAsState(
    targetShape = endShape,
    animationSpec = spring(dampingRatio = 0.8f, stiffness = 100f)
)
```
Severity: 🔴 Critical | Confidence: 95 | Rule: SM-013

---

**Line 62**: No Reduce Motion check
```kotlin
// BAD
val morphShape by animateShapeAsState(targetShape = newShape)

// GOOD
val animationScale = LocalAnimationScale.current
val morphShape by animateShapeAsState(
    targetShape = newShape,
    animationSpec = if (animationScale > 0) {
        spring(dampingRatio = 0.8f)
    } else {
        snap()
    }
)
```
Severity: 🔴 Critical | Confidence: 90 | Rule: SM-041

### Important Issues (🟠)

**Line 28**: Shape not using M3 library (custom path)
```kotlin
// BAD
drawPath(Path().apply { /* manual path drawing */ })

// GOOD
val shape = RoundedPolygon(numVertices = 8, radius = 40f)
drawPath(shape.toPath())
```
Severity: 🟠 Important | Confidence: 85 | Rule: SM-002

### Warnings (🟡)

**Line 35**: Shape recreation on recomposition
```kotlin
// BAD
val shape = RoundedPolygon(/* recreated every frame */)

// GOOD
val shape = remember {
    RoundedPolygon(numVertices = 4, radius = 40f)
}
```
Severity: 🟡 Warning | Confidence: 80 | Rule: SM-034

### Accessibility Checklist

- [x] ✅ Reduce Motion respected
- [x] ✅ Touch target 48x48dp
- [ ] ❌ contentDescription missing
- [x] ✅ Color + shape communicates state
- [x] ✅ Frame rate > 55fps

### Performance Analysis

- **Shape Path Rendering**: 3ms per frame (excellent)
- **Animation Computation**: 2ms per frame (excellent)
- **Memory Usage**: 2.5MB (no leaks detected)
- **Device Coverage**: Tested Pixel 6a (mid-range), Pixel 7 Pro (flagship)

### Summary

- **Total shape morphing issues**: 5
- **Critical**: 2 | Important: 1 | Warning: 1 | Suggestion: 1
- **M3 Compliance**: 80%
- **Performance**: 95% (smooth 60fps)
- **Accessibility**: 90% (good support for Reduce Motion)

### Priority Actions

1. Add Reduce Motion check (CRITICAL for accessibility)
2. Extend morph duration to 250ms for smoothness
3. Cache shape instances in remember blocks
4. Add contentDescription for semantic meaning
5. Test on Pixel 4a for mid-range performance

### Next Steps

1. Implement Reduce Motion support
2. Profile shape morphing with GPU tools
3. Add shape state documentation with visual diagram
4. Test accessibility with TalkBack
5. Benchmark on low-end device
```

---

## References

- [Material Design 3 Shape Morphing](https://m3.material.io/styles/shape/shape-morph)
- [Material 3 Expressive for Android](https://android-developers.googleblog.com/2025/08/introducing-material-3-expressive-for-wear-os.html)
- [androidx.graphics-shapes Documentation](https://developer.android.com/develop/ui/compose/graphics/shapes)
- [Jetpack Compose Animation Best Practices](https://developer.android.com/develop/ui/compose/animation)
- [RoundedPolygon API Reference](https://developer.android.com/reference/androidx/graphics/shapes/RoundedPolygon)
- [Shape - Material Design 3](https://m3.material.io/styles/shape)
- [Jetpack Compose Material 3 Expressive](https://android-developers.googleblog.com/2025/05/androidify-building-delightful-ui-with-compose.html)
- [WCAG 2.2 Animation from Interactions](https://www.w3.org/WAI/WCAG21/Understanding/animation-from-interactions.html)

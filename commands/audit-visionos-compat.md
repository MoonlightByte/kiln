---
name: audit-visionos-compat
description: Audit iOS/SwiftUI code for Apple Vision Pro visionOS spatial design compatibility
args:
  - name: code
    type: string
    description: Swift/SwiftUI code to audit for visionOS compatibility
    required: true
  - name: focus
    type: string
    enum: [spatial, gestures, windows, immersive, accessibility, performance, all]
    description: Specific visionOS area to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Apple Vision Pro visionOS Spatial Design Audit

Comprehensive audit of SwiftUI code for Apple Vision Pro visionOS compatibility, spatial design patterns, hand gesture interactions, eye tracking optimization, and immersive space integration.

## Instructions

Analyze the code for visionOS compatibility and spatial computing best practices. Reference the latest visionOS 26 spatial design patterns, Liquid Glass design system, spatial widgets, and Apple Vision Pro interaction model.

## Audit Checklist

### Spatial Container & Layout (Critical Priority)

- [ ] **VO-001**: Using `RealityView` for 3D spatial content placement
- [ ] **VO-002**: Proper use of `.windowStyle()` for window vs. volume distinction
- [ ] **VO-003**: Views avoid infinite width assumptions (no fixed 375pt width patterns)
- [ ] **VO-004**: Safe area respected on all sides (head-relative, not screen-relative)
- [ ] **VO-005**: Depth considerations in Z-axis (views layered intentionally, not overlapping)
- [ ] **VO-006**: Using `GeometryReader3D` for spatial layout calculations
- [ ] **VO-007**: Immersive space views properly constrained (not fullscreen by default)

### Window Management (Critical Priority)

- [ ] **VO-010**: Window size appropriate for content (using `.defaultSize()`)
- [ ] **VO-011**: Window placement intentional (not centered on head)
- [ ] **VO-012**: Using `.ornament()` for auxiliary controls, not main content
- [ ] **VO-013**: Multiple windows placed in user's cognitive space (not clustered)
- [ ] **VO-014**: Window dismiss behavior handles edge cases gracefully
- [ ] **VO-015**: Using `WindowGroup` for window lifecycle management
- [ ] **VO-016**: Persistent window state across app lifecycle

### Hand Gesture Interactions (Critical Priority)

- [ ] **VO-020**: Interactive targets minimum 44pt diameter (Vision Pro comfort zone)
- [ ] **VO-021**: Tap gesture recognized (primary pinch input)
- [ ] **VO-022**: No multi-point drag gestures required (provide alternatives)
- [ ] **VO-023**: Gesture feedback provides haptic or visual confirmation
- [ ] **VO-024**: Supporting both direct hand taps and voice commands
- [ ] **VO-025**: No gestures requiring precision finger positioning
- [ ] **VO-026**: Swipe gestures are large, high-confidence movements

### Eye Tracking Optimization (Critical Priority)

- [ ] **VO-030**: Eye-focused content responds to gaze (visual feedback on hover)
- [ ] **VO-031**: Not requiring sustained eye contact for interaction
- [ ] **VO-032**: Dwell time appropriate (0.5-1.0 sec default, not instant)
- [ ] **VO-033**: Eye-scroll enabled for long-form content (supporting visionOS 3+)
- [ ] **VO-034**: Debouncing eye gaze to prevent accidental triggers
- [ ] **VO-035**: Cursor/focus ring visible when eye-tracked
- [ ] **VO-036**: No parallax or eye-tracking motion sickness concerns

### 3D Layout & Spatial Positioning (Important Priority)

- [ ] **VO-040**: Using implicit 3D transforms (not relying on perspective)
- [ ] **VO-041**: Consistent spatial depth hierarchy (background, content, foreground)
- [ ] **VO-042**: Z-position values realistic (not extreme parallax)
- [ ] **VO-043**: Spatial coordinates use centimeters, not pixels
- [ ] **VO-044**: Content density respects spatial context (not cramped)
- [ ] **VO-045**: Proper depth-of-field considerations (content legible at distance)

### Spatial Widgets (Important Priority)

- [ ] **VO-050**: Widget frame customizable (width, color, depth options)
- [ ] **VO-051**: Widget states properly defined (`.widgetConfiguration`)
- [ ] **VO-052**: Widgets responsive to user interactions (tappable, scrollable)
- [ ] **VO-053**: Widget updates reflect spatial context changes
- [ ] **VO-054**: Using `WidgetKit` for visionOS spatial widgets
- [ ] **VO-055**: Widget appearance follows Liquid Glass design

### Immersive Space Integration (Important Priority)

- [ ] **VO-060**: Immersive space used only for full-screen experiences
- [ ] **VO-061**: Immersive transitions smooth (no jarring focus changes)
- [ ] **VO-062**: User can easily exit immersive space (crown gesture, voice)
- [ ] **VO-063**: Passthrough option available in immersive views
- [ ] **VO-064**: Immersive content respects user's physical space (collision detection)
- [ ] **VO-065**: Hand presence in immersive space properly handled

### Shared Coordinate Spaces (Important Priority)

- [ ] **VO-070**: Using `RealityKit` anchors for persistent spatial placement
- [ ] **VO-071**: Content positioned relative to head, not fixed to screen
- [ ] **VO-072**: Multiple views sharing consistent spatial origin
- [ ] **VO-073**: Hand gesture coordinates mapped to 3D space correctly
- [ ] **VO-074**: Eye gaze ray-casting accurate in 3D environment

### iOS Compatibility Mode (Important Priority)

- [ ] **VO-080**: App gracefully runs on iOS (iPadOS fallback paths)
- [ ] **VO-081**: Conditional compilation for visionOS-specific features
- [ ] **VO-082**: Not assuming Hand Tracking API availability (fallback to standard gestures)
- [ ] **VO-083**: Not assuming eye-tracking beyond look-and-tap (accessibility substitute)
- [ ] **VO-084**: Using `#if os(visionOS)` for spatial features

### Accessibility in Spatial Computing (Important Priority)

- [ ] **VO-090**: `.accessibilityLabel` on all interactive 3D elements
- [ ] **VO-091**: Voice control labels clear and distinct
- [ ] **VO-092**: No reliance on hand tracking for critical functions (voice alternative)
- [ ] **VO-093**: Text in 3D space remains legible (minimum 14pt equivalent)
- [ ] **VO-094**: Color contrast maintained even in depth (Liquid Glass translucency)
- [ ] **VO-095**: Spatial audio cues support spatial understanding
- [ ] **VO-096**: No seizure-inducing parallax or motion effects

### Liquid Glass Design System (Important Priority)

- [ ] **VO-100**: Using `.materialAppearance()` for translucent backgrounds
- [ ] **VO-101**: Respecting user privacy (no blurring user environment)
- [ ] **VO-102**: Subtle blur effects (not heavy Gaussian blur)
- [ ] **VO-103**: Color opacity appropriate for readability over dynamic environment
- [ ] **VO-104**: Animations smooth and glass-like (not rigid or sudden)
- [ ] **VO-105**: Using system materials for consistency

### 3D Models & Reality Composer (Important Priority)

- [ ] **VO-110**: Using USDZ format for 3D content (visionOS standard)
- [ ] **VO-111**: 3D model LOD (level of detail) for performance
- [ ] **VO-112**: Model physics properly configured (if interactive)
- [ ] **VO-113**: Texture resolution appropriate for Vision Pro display
- [ ] **VO-114**: Model shadows cast realistically
- [ ] **VO-115**: Using Reality Composer 2 for spatial composition

### Performance & Optimization (Important Priority)

- [ ] **VO-120**: Frame rate consistent (90 FPS target for Vision Pro)
- [ ] **VO-121**: Memory usage below 2GB for core app experience
- [ ] **VO-122**: No expensive texture loading on main thread
- [ ] **VO-123**: Hand tracking updates throttled appropriately
- [ ] **VO-124**: Spatial queries (ray-casting) batched efficiently
- [ ] **VO-125**: Using `.renderingServerCompute` for heavy calculations
- [ ] **VO-126**: Profiling with Xcode spatial profiler

### Testing on Vision Pro Simulator (Warning Priority)

- [ ] **VO-130**: Tested on latest visionOS simulator
- [ ] **VO-131**: Hand gesture interactions tested in simulator
- [ ] **VO-132**: Spatial layouts verified at different scales
- [ ] **VO-133**: Voice command paths tested
- [ ] **VO-134**: Eye-tracking behavior validated
- [ ] **VO-135**: Memory profiling completed

## Spatial Design Code Examples

### Liquid Glass Window (Recommended Pattern)

```swift
var body: some View {
    ZStack {
        // Liquid Glass background
        RoundedRectangle(cornerRadius: 12)
            .fill(.ultraThinMaterial)
            .ignoresSafeArea()

        // Content on translucent surface
        VStack {
            Text("Spatial Content")
                .font(.title)

            Button(action: { }) {
                Label("Tap Here", systemImage: "hand.tap")
            }
            .frame(height: 44) // Vision Pro comfort minimum
        }
        .padding()
    }
    .windowStyle(.automatic)
    .defaultSize(.init(width: 400, height: 500))
}
```

### Hand Gesture Recognition (Recommended Pattern)

```swift
@State private var didTap = false

var body: some View {
    VStack {
        Text("Tap the surface with your fingers")
            .foregroundColor(didTap ? .green : .white)
    }
    .onTapGesture {
        // Haptic feedback
        if #available(visionOS 1.0, *) {
            // Use appropriate haptic for hand interaction
        }
        didTap = true
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            didTap = false
        }
    }
}
```

### Eye-Tracking Gaze Feedback (Recommended Pattern)

```swift
@State private var isGazeFocused = false

var body: some View {
    VStack {
        Text("Look at this content")
            .padding()
            .background(isGazeFocused ? Color.blue.opacity(0.3) : Color.gray.opacity(0.1))
            .cornerRadius(8)
    }
    .onContinuousHover { phase in
        switch phase {
        case .active:
            isGazeFocused = true
        case .ended:
            isGazeFocused = false
        }
    }
}
```

### Immersive Space Entry (Recommended Pattern)

```swift
@Environment(\.openImmersiveSpace) var openImmersiveSpace
@State private var immersiveSpaceIsShown = false

var body: some View {
    Button("Enter Immersive Experience") {
        Task {
            await openImmersiveSpace(id: "immersive_experience")
            immersiveSpaceIsShown = true
        }
    }
}
```

### Spatial Widget Configuration (Recommended Pattern)

```swift
@main
struct MyWidget: Widget {
    let kind: String = "com.example.spatial"

    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: kind,
            intent: ConfigurationAppIntent.self,
            provider: Provider()
        ) { entry in
            WidgetView(entry: entry)
                .containerBackground(.ultraThinMaterial, for: .widget)
        }
        .supportedFamilies([.accessoryRectangular])
        .customWidgetFrame(width: 200, height: 100, depth: 50)
    }
}
```

### 3D Content with RealityView (Recommended Pattern)

```swift
RealityView { content in
    // Load USDZ model
    if let usdz = try? await ModelEntity(
        modelName: "MyModel",
        bundle: Bundle.main
    ) {
        var transform = usdz.transform
        transform.translation.z = -1 // 1 meter in front of user
        usdz.transform = transform
        content.add(usdz)
    }
}
.gesture(
    TapGesture()
        .targetedToAnyEntity()
        .onEnded { _ in
            // Handle interaction
        }
)
```

### Conditional visionOS Feature (Recommended Pattern)

```swift
var body: some View {
    VStack {
        #if os(visionOS)
        // Vision Pro specific UI
        SpatialView()
        #else
        // iOS/iPadOS fallback
        StandardView()
        #endif
    }
}
```

## Accessibility in visionOS

### Voice Control (Critical)

- All interactive elements must have clear, unique voice labels
- Avoid homonyms in voice commands (e.g., "red" vs. "read")
- Provide alternative navigation paths to spatial gestures
- Support both native Siri voice and custom voice labels

### Hand Accessibility

- Not requiring dexterous finger placement (support whole-hand tap zones)
- Providing visual/haptic feedback for all hand interactions
- Supporting left-handed and right-handed users equally

### Eye Tracking Accessibility

- Eye tracking should supplement, not replace, other input methods
- Provide dwell-time adjustments for users with slower eye movement
- Support voice-only input for users unable to use eye tracking

### Spatial Audio

- Spatial audio cues orient users in 3D space
- Audio panning matches visual element position
- Supporting mono audio output as fallback

## Testing Checklist

### Vision Pro Simulator

- [ ] App launches in visionOS 26+ simulator
- [ ] Spatial layouts render without artifacts
- [ ] Hand gestures recognized in simulator
- [ ] Eye-tracking cursor appears and responds
- [ ] Voice commands processed correctly
- [ ] Immersive spaces transition smoothly

### Performance Profiling

- [ ] Frame rate stable at 90 FPS (using Xcode profiler)
- [ ] Memory footprint under 2GB for core experience
- [ ] No shader compilation stalls during interaction
- [ ] Spatial queries complete in < 16ms

### User Testing

- [ ] Hand gesture targets easily reachable from rest position
- [ ] Text legible at typical viewing distance (60-100cm)
- [ ] No eye strain from extended use
- [ ] Spatial audio cues aid navigation
- [ ] Voice commands recognized reliably

## Code to Audit

```swift
{{code}}
```

## Output Format

Generate a structured audit report:

```markdown
## visionOS Spatial Design Audit: [Component Name]

### Critical Issues (🔴)

List blockers for Vision Pro deployment. Each with: Line number, description, severity, fix suggestion.

**VO-XXX**: [Issue title]
```swift
// Problematic code
```
Severity: 🔴 Critical | Confidence: X% | Rule: VO-XXX

---

### Important Issues (🟠)

List violations of spatial design best practices.

### Warnings (🟡)

List suboptimal patterns that work but should be improved.

### Suggestions (🔵)

List polish opportunities for Liquid Glass and spatial excellence.

### Spatial Compatibility Summary

| Category | Status | Issues |
|----------|--------|--------|
| Spatial Layout | ⚠️ | 2 violations |
| Hand Gestures | ✅ | 0 violations |
| Eye Tracking | ✅ | 0 violations |
| Immersive Space | ⚠️ | 1 violation |
| Accessibility | ✅ | 0 violations |
| Liquid Glass Design | ✅ | 0 violations |

### Summary

- **visionOS Compatibility**: X%
- **Total Issues**: X
- **Critical**: X | Important: X | Warning: X | Suggestion: X
- **Ready for Vision Pro**: [YES/NO]

### Recommended Actions

1. [Most urgent spatial fix]
2. [Gesture interaction improvement]
3. [Accessibility enhancement]
4. [Performance optimization]
5. [Liquid Glass polish]

### Testing Recommendations

1. Run on Vision Pro simulator with visionOS 26+
2. Profile memory and frame rate with Xcode spatial tools
3. Test hand gestures from seated position
4. Verify eye-tracking gaze feedback
5. Validate voice command paths
```

## Example Output

```markdown
## visionOS Spatial Design Audit: EventDetailView

### Critical Issues (🔴)

**VO-083**: No fallback for hand tracking API

```swift
// BAD - assumes hand tracking available
let recognizer = SpatialTapGesture()

// GOOD - conditional fallback
#if os(visionOS)
let recognizer = SpatialTapGesture()
#else
let recognizer = TapGesture()
#endif
```

Severity: 🔴 Critical | Confidence: 95% | Rule: VO-083

---

**VO-003**: Hardcoded width assumes iOS screen (no infinite canvas handling)

```swift
// BAD
Text("Event Details")
    .frame(width: 375)

// GOOD
Text("Event Details")
    .frame(maxWidth: .infinity)
```

Severity: 🔴 Critical | Confidence: 90% | Rule: VO-003

### Important Issues (🟠)

**VO-020**: Interactive target too small (36pt vs. 44pt minimum)

```swift
// BAD
Button(action: { }) {
    Image(systemName: "checkmark.circle")
}
.frame(width: 36, height: 36)

// GOOD
Button(action: { }) {
    Image(systemName: "checkmark.circle")
}
.frame(width: 44, height: 44)
```

Severity: 🟠 Important | Confidence: 85% | Rule: VO-020

---

**VO-100**: Not using Liquid Glass material for window background

```swift
// BAD
.background(Color.white)

// GOOD
.background(.ultraThinMaterial)
```

Severity: 🟠 Important | Confidence: 80% | Rule: VO-100

### Warnings (🟡)

**VO-031**: Using continuous eye tracking without dwell time

- Add 0.5-1.0 second dwell before triggering eye-tracked action
- Prevents accidental activation from casual glances

### Spatial Compatibility Summary

| Category | Status | Issues |
|----------|--------|--------|
| Spatial Layout | ⚠️ | 1 violation |
| Hand Gestures | ✅ | 0 violations |
| Eye Tracking | ⚠️ | 1 violation |
| Immersive Space | ✅ | 0 violations |
| Accessibility | ✅ | 0 violations |
| Liquid Glass Design | ⚠️ | 1 violation |

### Summary

- **visionOS Compatibility**: 75%
- **Total Issues**: 5
- **Critical**: 2 | Important: 2 | Warning: 1 | Suggestion: 0
- **Ready for Vision Pro**: NO (requires fixes)

### Recommended Actions

1. Add conditional compilation for hand tracking (#if os(visionOS))
2. Increase interactive button targets to 44pt minimum
3. Apply Liquid Glass material backgrounds
4. Add dwell-time debouncing for eye gaze
5. Test on Vision Pro simulator with spatial profiler
```

## References & Frameworks

- [Apple visionOS Developer Documentation](https://developer.apple.com/visionos/)
- [What's New in visionOS 26 - WWDC25](https://developer.apple.com/videos/play/wwdc2025/317/)
- [Designing for Apple Vision Pro](https://developer.apple.com/design/human-interface-guidelines/spatial-computing)
- [RealityKit Documentation](https://developer.apple.com/documentation/realitykit)
- [SwiftUI Spatial Design](https://developer.apple.com/videos/play/wwdc2023/10110/)
- [Hand Gesture Recognition in visionOS](https://developer.apple.com/videos/play/wwdc2024/10115/)
- [Liquid Glass Design System](https://developer.apple.com/design/system/)

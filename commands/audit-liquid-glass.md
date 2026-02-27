---
name: audit-liquid-glass
description: Audit SwiftUI code for iOS 26 Liquid Glass design system compliance
args:
  - name: code
    type: string
    description: SwiftUI code to audit for Liquid Glass integration
    required: true
  - name: focus
    type: string
    enum: [materials, accessibility, performance, navigation, components, all]
    description: Specific area to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# iOS 26 Liquid Glass Design Audit for SwiftUI

Perform a comprehensive audit of SwiftUI code against Apple's Liquid Glass design system specifications introduced at WWDC 2025.

## Design System Overview

Liquid Glass is an adaptive material that refracts and reflects its surroundings with dynamic transformation. It provides:
- Translucent, glass-like appearance with real-time rendering
- Thickness variations (.thin, .regular, .thick) for depth perception
- Specular highlights responding to device motion
- Dynamic focus management through morphing and scaling
- Integrated with navigation, controls, tab bars, widgets, and custom components

## Instructions

Analyze the code for compliance with Liquid Glass specifications. Reference iOS 26 Human Interface Guidelines and Apple's Liquid Glass documentation.

## Audit Checklist

### Core Material Implementation (Critical Priority)
{{#if (or (eq focus "materials") (eq focus "all") (not focus))}}
- [ ] **LG-001**: Using `.glassEffect()` modifier for all glass-backed views
- [ ] **LG-002**: Glass variants match design intent (regular, clear, or identity)
- [ ] **LG-003**: Material thickness set appropriately (.thin for subtle, .regular for standard, .thick for emphasis)
- [ ] **LG-004**: Shape parameter provided for consistent morphing behavior
- [ ] **LG-005**: Glass tinted with `.tint()` only for branded emphasis (avoid color overload)
- [ ] **LG-006**: Not mixing multiple glass styles in overlapping views
- [ ] **LG-007**: Using `GlassEffectContainer` for composite glass shapes
- [ ] **LG-008**: Glass shapes morph smoothly during state transitions
{{/if}}

### Tab Bar and Navigation (Critical Priority)
{{#if (or (eq focus "navigation") (eq focus "all") (not focus))}}
- [ ] **LG-010**: Tab bar uses `.glassEffect()` with consistent thickness
- [ ] **LG-011**: Tab bar shrinks on scroll, expands on scroll up (fluid transform)
- [ ] **LG-012**: Navigation bar glass effect maintains legibility of title/buttons
- [ ] **LG-013**: Tab bar background blur matches light/dark appearance
- [ ] **LG-014**: Tab indicators animate smoothly with `.glassEffect()`
- [ ] **LG-015**: Spacing between tab items respects 4pt grid within glass bounds
- [ ] **LG-016**: Back button and action buttons rendered over glass with clear affordance
{{/if}}

### Dynamic Island Integration (Important Priority)
{{#if (or (eq focus "components") (eq focus "all") (not focus))}}
- [ ] **LG-020**: Dynamic Island widgets use glass material appropriately
- [ ] **LG-021**: Glass thickness increases when island expands
- [ ] **LG-022**: Refraction effects remain readable during expansion animations
- [ ] **LG-023**: Activity-specific icons rendered cleanly on glass background
- [ ] **LG-024**: Text labels have sufficient contrast on glass (.label semantic color)
{{/if}}

### Text Legibility on Glass (Critical Priority)
{{#if (or (eq focus "accessibility") (eq focus "all") (not focus))}}
- [ ] **LG-030**: Text rendered with `.label`, `.secondaryLabel`, or explicit white/black
- [ ] **LG-031**: Never hardcoding text colors on glass backgrounds
- [ ] **LG-032**: Contrast ratio >= 4.5:1 for normal text on glass
- [ ] **LG-033**: Contrast ratio >= 3:1 for large text (18pt+) on glass
- [ ] **LG-034**: Bold or increased font weight used for critical text on glass
- [ ] **LG-035**: Symbol icons have sufficient opacity (>= 70%) for readability
- [ ] **LG-036**: Text uses Dynamic Type (`.font(.body)` not `.font(.system(size:))`)
- [ ] **LG-037**: Using `.lineLimit(1)` or `.truncationMode()` to prevent glass overflow
{{/if}}

### Glass Layering and Depth (Important Priority)
{{#if (or (eq focus "materials") (eq focus "all") (not focus))}}
- [ ] **LG-040**: Glass thickness increases perceived depth correctly
- [ ] **LG-041**: Refraction intensity scales with material thickness
- [ ] **LG-042**: Shadow depth increases with glass thickness (.thin < .regular < .thick)
- [ ] **LG-043**: Z-order of overlapping glass shapes is intentional
- [ ] **LG-044**: Nested glass containers blend smoothly without flickering
- [ ] **LG-045**: No hardcoded shadow blur; relying on glass material shadows
{{/if}}

### Solarium Color Palette Integration (Important Priority)
{{#if (or (eq focus "materials") (eq focus "all") (not focus))}}
- [ ] **LG-050**: Using semantic colors (.systemBlue, .systemGreen, etc.) for glass tints
- [ ] **LG-051**: Custom brand colors tinted into glass via `.tint()` sparingly
- [ ] **LG-052**: Solarium accent colors (gold, purple, teal) used for interactive states
- [ ] **LG-053**: Glass background adapts to light/dark appearance automatically
- [ ] **LG-054**: High contrast mode supported (glass tint adjusted for visibility)
- [ ] **LG-055**: No pure white or pure black glass tints (use semantic colors instead)
{{/if}}

### Dark Mode and Appearance (Important Priority)
{{#if (or (eq focus "accessibility") (eq focus "all") (not focus))}}
- [ ] **LG-060**: Glass effect adapts to `colorScheme` (light/dark)
- [ ] **LG-061**: Text and icons remain legible in both light and dark modes
- [ ] **LG-062**: Refraction depth adjusted for dark mode visibility
- [ ] **LG-063**: `.preferredColorScheme()` override respected if used
- [ ] **LG-064**: No inverted colors on glass in dark mode
{{/if}}

### Interactive States and Gestures (Important Priority)
{{#if (or (eq focus "components") (eq focus "all") (not focus))}}
- [ ] **LG-070**: Using `.glassEffect(...).interactive()` for tap/drag sensitive areas
- [ ] **LG-071**: Interactive glass responds to gesture with immediate visual feedback
- [ ] **LG-072**: Pressed state dims or shifts glass color, not background
- [ ] **LG-073**: Hover effects subtle and non-distracting (iOS/iPadOS)
- [ ] **LG-074**: Focus indicators visible within glass bounds for keyboard navigation
{{/if}}

### Animation and Motion (Important Priority)
{{#if (or (eq focus "navigation") (eq focus "all") (not focus))}}
- [ ] **LG-080**: Glass shape morphing uses `.easeInOut` or `.spring()` (no linear)
- [ ] **LG-081**: Morph animations respect `.accessibilityReduceMotion`
- [ ] **LG-082**: Refraction intensity animates smoothly during transitions
- [ ] **LG-083**: Tab bar expand/shrink animations are fluid (<= 300ms)
- [ ] **LG-084**: No jarring color shifts; glass tint changes gradually
- [ ] **LG-085**: Layer transitions within `GlassEffectContainer` blend seamlessly
{{/if}}

### Performance Optimization (Important Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **LG-090**: Glass effect not applied to every view (use selectively)
- [ ] **LG-091**: Heavy computation moved off main thread
- [ ] **LG-092**: `.glassEffect()` modifier chain optimized (no redundant calls)
- [ ] **LG-093**: `GlassEffectContainer` used for composite shapes (not individual overlays)
- [ ] **LG-094**: List/LazyVStack with glass components use `.id()` for reuse
- [ ] **LG-095**: Glass blur radius capped at reasonable values (no extreme blur)
- [ ] **LG-096**: Specular highlights update smoothly at 60fps (verify with profiler)
{{/if}}

### Fallback Patterns for Older iOS (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **LG-100**: Code gracefully degrades for iOS < 26 (use `@available`)
- [ ] **LG-101**: Fallback UI uses solid colors matching semantic palette
- [ ] **LG-102**: Fallback respects accessibility preferences (transparency)
- [ ] **LG-103**: No crashes when `.glassEffect()` unavailable
- [ ] **LG-104**: Conditional compilation or runtime checks prevent glass-only features
{{/if}}

### Accessibility (Critical Priority)
{{#if (or (eq focus "accessibility") (eq focus "all") (not focus))}}
- [ ] **LG-110**: `.accessibilityLabel` provided for glass interactive elements
- [ ] **LG-111**: Reduced transparency mode supported (`.accessibilityShowTextBackgroundColors`)
- [ ] **LG-112**: Glass components remain 44x44pt minimum tap target
- [ ] **LG-113**: VoiceOver describes glass component purpose clearly
- [ ] **LG-114**: Focus order correct when navigating glass containers with keyboard
- [ ] **LG-115**: No essential information conveyed only by glass opacity/blur
- [ ] **LG-116**: Color alone not used to indicate state on glass (add icons/text)
{{/if}}

### Testing and Validation (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **LG-120**: Screenshots tested on iPhone (OLED), iPad, and Mac displays
- [ ] **LG-121**: Refraction quality consistent across device motion ranges
- [ ] **LG-122**: Glass rendering performance profiled with Xcode Instruments
- [ ] **LG-123**: Accessibility audit passed (VoiceOver, display accommodations)
- [ ] **LG-124**: Dark mode, light mode, and high contrast tested
- [ ] **LG-125**: iOS 26+ simulator and real device tested
{{/if}}

## Code Patterns to Check

### Correct Glass Pattern
```swift
// GOOD: Basic glass with proper modifier chain
Button(action: { action() }) {
    Label("Create Event", systemImage: "plus.circle.fill")
}
.glassEffect(in: .capsule)  // Explicit shape
.padding(.horizontal, 16)
.accessibilityLabel("Create Event")

// GOOD: Glass with tint for branding
VStack {
    Text("Featured Event")
        .font(.headline)
    EventCardView()
}
.glassEffect()
.tint(.blue)  // Light blue tint

// GOOD: Composite glass with GlassEffectContainer
GlassEffectContainer {
    HStack {
        TabBarItem(tab: .explore)
        TabBarItem(tab: .games)
        TabBarItem(tab: .clubs)
    }
}

// GOOD: Interactive glass responding to gesture
Text("Tap to expand")
    .glassEffect()
    .interactive()  // Responds to taps and drags
```

### Anti-Patterns to Avoid
```swift
// BAD: Hardcoded color on glass
Text("Title")
    .foregroundColor(.purple)  // Will fail in dark mode
    .glassEffect()

// BAD: Every view wrapped in glass
VStack {
    ForEach(items) { item in
        ItemRow()
            .glassEffect()  // Excessive; use GlassEffectContainer
    }
}

// BAD: No fallback for older iOS
View()
    .glassEffect()  // Crashes on iOS 25

// BAD: Extreme blur or refraction
.glassEffect(in: .rect(cornerRadius: 0))
    .tint(.green)
    .tint(.blue)  // Multiple tints conflict

// BAD: Hardcoded thickness without reason
.glassEffect()  // Should use .regular by default
.padding()

// BAD: Glass obscuring critical text
Text("Booking confirmation")
    .lineLimit(3)  // Will truncate; use proper constraints
    .glassEffect()
```

## Solarium Color Palette (iOS 26)

Use these semantic colors for glass tints:

| Category | Color | Hex | Usage |
|----------|-------|-----|-------|
| **Semantic** | systemBlue | varies | Primary actions |
| | systemGreen | varies | Success states |
| | systemRed | varies | Destructive actions |
| | systemOrange | varies | Warnings |
| | systemPurple | varies | Secondary emphasis |
| | systemTeal | varies | Accent (tertiary) |
| **Brand** | customBrand | varies | App identity |
| **Interactive** | systemGray | varies | Neutral/disabled |

## Material Thickness Guidelines

| Thickness | Use Case | Refraction | Depth |
|-----------|----------|-----------|-------|
| **.thin** | Subtle overlays, badges | Minimal | Low |
| **.regular** | Navigation, tab bars, cards | Moderate | Standard |
| **.thick** | Featured components, modals | Pronounced | High |

## Performance Considerations

1. **Rendering**: Glass uses CoreAnimation + Metal shaders (requires Apple Silicon for smooth performance)
2. **Motion**: Specular highlights update at device motion sensor refresh rate (60-120fps)
3. **Memory**: Each glass container allocates texture buffers; limit per-screen instances
4. **Battery**: Glass blur/refraction effects increase GPU load by ~8-12%; profile with Power gauge
5. **Profiling**: Use Xcode Metal Debugger to verify refraction quality and frame rates

## Recommended Actions for Issues

**Critical Issues:**
1. Add `.glassEffect()` to navigation bars and tab bars immediately
2. Replace hardcoded text colors with semantic colors
3. Ensure accessibility labels on all interactive glass components

**Important Issues:**
1. Consolidate overlapping glass shapes with `GlassEffectContainer`
2. Add dark mode support and test contrast ratios
3. Implement fallbacks for iOS < 26

**Warnings:**
1. Optimize animation timing (aim for 60fps minimum)
2. Reduce excessive glass instances in lists
3. Test on real devices for motion responsiveness

**Suggestions:**
1. Consider interactive glass for key gesture areas
2. Add subtle tints for visual hierarchy
3. Profile GPU usage with Instruments

## References

- [Apple Liquid Glass Documentation](https://developer.apple.com/documentation/TechnologyOverviews/liquid-glass)
- [glassEffect Modifier API](https://developer.apple.com/documentation/swiftui/view/glasseffect(_:in:))
- [GlassEffectContainer Documentation](https://developer.apple.com/documentation/swiftui/glasseffectcontainer)
- [WWDC 2025 Liquid Glass Session](https://developer.apple.com/videos/play/wwdc2025/219/)
- [Build a SwiftUI App with the New Design](https://developer.apple.com/videos/play/wwdc2025/323/)
- [Apple Newsroom: iOS 26 Announcement](https://www.apple.com/newsroom/2025/06/apple-introduces-a-delightful-and-elegant-new-software-design/)

## Output Format

Generate a structured audit report:

```markdown
## Liquid Glass Audit: [Component/Screen Name]

### Critical Issues (🔴)
- **LG-###**: [Issue description]
- Fix suggestion with code example
- Severity: Critical | Confidence: 95%

### Important Issues (🟠)
- **LG-###**: [Issue description]

### Warnings (🟡)
- **LG-###**: [Issue description]

### Suggestions (🔵)
- **LG-###**: [Enhancement idea]

### Summary
- Total Issues: X
- Critical: X | Important: X | Warning: X | Suggestion: X
- Glass Compliance: X%
- Accessibility: X%

### Recommended Actions
1. [Highest priority fix]
2. [Second priority]
3. [Third priority]
```

## Code to Audit

```swift
{{code}}
```

---
name: responsive-audit
description: Comprehensive responsive design audit for phones, tablets, foldables, and split-screen layouts
args:
  - name: code
    type: string
    description: Mobile code (Kotlin Compose or SwiftUI) to audit
    required: true
  - name: platform
    type: string
    enum: [android, ios, auto]
    description: Target platform (default: auto-detect)
    required: false
  - name: focus
    type: string
    enum: [breakpoints, navigation, text-scaling, multi-window, foldables, large-screens, all]
    description: Specific area to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Responsive Design Audit

Comprehensive audit of mobile code against 2025-2026 responsive design best practices, including support for foldable devices, tablets, large screens, split-screen, and dynamic text scaling.

## Instructions

Analyze the provided code for responsive design compliance. Auto-detect platform based on syntax (`@Composable` = Android, `some View` = iOS). Cover all form factors: compact phones, medium tablets, large tablets, foldables in multiple states, and split-screen/multi-window modes.

## Responsive Design Audit Checklist

### Breakpoint Strategy (Important Priority)

{{#if (or (eq focus "breakpoints") (eq focus "all") (not focus))}}

**Android WindowSizeClass (Material 3 Adaptive)**

- [ ] **RES-001**: Using `currentWindowAdaptiveInfo()` from Material 3 Adaptive library
- [ ] **RES-002**: Implementing compact (< 600dp), medium (600dp-839dp), expanded (>= 840dp) breakpoints
- [ ] **RES-003**: Supporting large (1200dp-1599dp) and extra-large (>= 1600dp) breakpoints (AndroidX.window 1.5+)
- [ ] **RES-004**: Content-driven breakpoints (where layout adapts) not device-dictated
- [ ] **RES-005**: Using fluid layouts with `clamp()` for smooth scaling between breakpoints
- [ ] **RES-006**: Maintaining minimum text size and touch targets across all sizes

**iOS Size Classes**

- [ ] **RES-007**: Using horizontal and vertical size classes (compact vs regular)
- [ ] **RES-008**: Compact width layouts for iPhone portrait / iPad split view
- [ ] **RES-009**: Regular width layouts for iPad full screen and iPad landscape
- [ ] **RES-010**: Testing on 5.5", 6.5", 7.9", 11", 12.9" screen sizes
- [ ] **RES-011**: Supporting split view on iPad (compact or regular depending on device/orientation)
- [ ] **RES-012**: No hardcoded screen sizes - using traits and constraints

{{/if}}

### Navigation Adaptation (Critical Priority)

{{#if (or (eq focus "navigation") (eq focus "all") (not focus))}}

**Android NavigationSuiteScaffold (Material 3 Adaptive)**

- [ ] **RES-013**: Using `NavigationSuiteScaffold` for automatic navigation mode switching
- [ ] **RES-014**: Bottom navigation bar for compact size class
- [ ] **RES-015**: Navigation rail for medium/expanded size classes
- [ ] **RES-016**: Navigation drawer for expanded screens when appropriate
- [ ] **RES-017**: Consistent state preservation when navigation mode changes

**iOS Adaptive Navigation**

- [ ] **RES-018**: TabView / bottom navigation for compact (iPhone)
- [ ] **RES-019**: Sidebar or split view for regular (iPad full screen)
- [ ] **RES-020**: Properly handling navigation transitions between size classes
- [ ] **RES-021**: No overlapping navigation elements in any size class

**General Navigation**

- [ ] **RES-022**: Navigation hierarchy flattened on large screens (fewer taps to reach content)
- [ ] **RES-023**: Master-detail layout on tablets with list on left, detail on right
- [ ] **RES-024**: Primary action remains accessible in all screen sizes
- [ ] **RES-025**: Back button hidden or removed on tablets with permanent navigation

{{/if}}

### Text Scaling & Dynamic Type (Critical Priority)

{{#if (or (eq focus "text-scaling") (eq focus "all") (not focus))}}

**iOS Dynamic Type**

- [ ] **RES-026**: Using TextStyle categories (body, title1, caption1, etc.) not custom fonts
- [ ] **RES-027**: Testing at all 12 size levels (xSmall to AX5, with AX1-AX5 being accessibility sizes)
- [ ] **RES-028**: Text can scale to 300%+ without breaking layout or truncating essential content
- [ ] **RES-029**: Using `UIScrollView` / `ScrollView` for content that may grow with text scaling
- [ ] **RES-030**: Custom fonts using `UIFontMetrics` for Dynamic Type compatibility
- [ ] **RES-031**: No hardcoded font sizes - all text scales with system settings
- [ ] **RES-032**: Testing at contentSizeCategory extremes (.extraSmall and .accessibilityExtraExtraExtraLarge)

**Android Text Scaling**

- [ ] **RES-033**: Using `sp` (scale-independent pixels) for all text sizes
- [ ] **RES-034**: Respecting user's font size preference in accessibility settings
- [ ] **RES-035**: Text with large font size scales smoothly using `clamp()`
- [ ] **RES-036**: Testing with system font scale at 100%, 125%, 150%, 200%
- [ ] **RES-037**: Layout remains functional with doubled text size

**Both Platforms**

- [ ] **RES-038**: Line height increases proportionally with text size
- [ ] **RES-039**: No text truncation of important content even at large sizes
- [ ] **RES-040**: Buttons and touch targets grow proportionally with text scaling
- [ ] **RES-041**: Color contrast maintained at all text sizes

{{/if}}

### Multi-Window & Split-Screen Support (Important Priority)

{{#if (or (eq focus "multi-window") (eq focus "all") (not focus))}}

**Android Multi-Window**

- [ ] **RES-042**: `resizableActivity=true` in AndroidManifest.xml
- [ ] **RES-043**: Handles activity lifecycle correctly in multi-window (multi-resume on Android 10+)
- [ ] **RES-044**: Configuration changes handled (screenSize, smallestScreenWidth, screenLayout)
- [ ] **RES-045**: Using Activity Embedding for list-detail layouts (Android 12+)
- [ ] **RES-046**: Split-pair rules defined for adjacent activity placement
- [ ] **RES-047**: Testing with apps side by side (50/50 split) and various aspect ratios
- [ ] **RES-048**: Content readable in smallest split-screen width (typically 320-400dp)
- [ ] **RES-049**: No overlapping dialogs across split boundaries

**iOS Picture-in-Picture & Split View**

- [ ] **RES-050**: Supporting window scene lifecycle changes
- [ ] **RES-051**: Geometry reader for responsive layout adjustments
- [ ] **RES-052**: Proper state management when entering/exiting split view
- [ ] **RES-053**: Content visible in both portrait and landscape splits
- [ ] **RES-054**: AVPlayer/video content supports pip mode seamlessly

{{/if}}

### Foldable Device Support (Critical Priority)

{{#if (or (eq focus "foldables") (eq focus "all") (not focus))}}

**Foldable Posture & Display State**

- [ ] **RES-055**: Using `WindowInfo.displayFeatures` from Jetpack WindowManager
- [ ] **RES-056**: Detecting hinge location and orientation (horizontal vs vertical)
- [ ] **RES-057**: Supporting FLAT, HALF_OPENED (tabletop), HALF_OPENED (book), and FULLY_OPENED states
- [ ] **RES-058**: Content positioned in safe zones, avoiding hinge line
- [ ] **RES-059**: Dual-pane layout when device is HALF_OPENED (top/bottom or left/right)
- [ ] **RES-060**: Single-pane layout when device is FLAT (phone mode)
- [ ] **RES-061**: No UI elements overlapping the fold/hinge

**Foldable Layout Adaptation**

- [ ] **RES-062**: Tabletop posture (horizontal fold) - content on top, controls on bottom
- [ ] **RES-063**: Book posture (vertical fold) - master detail split (left/right)
- [ ] **RES-064**: Testing with actual foldable state transitions
- [ ] **RES-065**: Hinge width accommodated (typically 30-50dp) - content doesn't touch edges
- [ ] **RES-066**: Scrollable content works correctly across fold
- [ ] **RES-067**: Bottom sheet / dialogs positioned to avoid hinge

**Samsung Galaxy Fold / Pixel Fold Specific**

- [ ] **RES-068**: App persists state when transitioning from cover screen to main screen
- [ ] **RES-069**: Content orientation correct when device is partially folded
- [ ] **RES-070**: UI elements resize properly as fold angle changes
- [ ] **RES-071**: Testing on actual device or Samsung/Google emulator images

{{/if}}

### Large Screen Layouts (Tablet/Desktop) (Important Priority)

{{#if (or (eq focus "large-screens") (eq focus "all") (not focus))}}

**iPad / Large Tablet Layouts**

- [ ] **RES-072**: Using full screen real estate (not mobile-centered design)
- [ ] **RES-073**: Master-detail pane layouts with sidebar + main content
- [ ] **RES-074**: Dual-pane layouts showing list and detail simultaneously
- [ ] **RES-075**: Two-column grid layouts instead of single-column scrolling
- [ ] **RES-076**: Navigation rail or sidebar permanently visible (not hidden behind hamburger)

**Android Tablets (600dp+)**

- [ ] **RES-077**: Expanded layout for medium size class (600-839dp)
- [ ] **RES-078**: Canonical layout for expanded size class (840dp+)
- [ ] **RES-079**: Ignoring orientation restrictions on large screens (system ignores manifest settings)
- [ ] **RES-080**: Using expanded NavigationSuiteScaffold for tablets

**Content Optimization**

- [ ] **RES-081**: Whitespace and padding increased appropriately for larger screens
- [ ] **RES-082**: Font sizes remain readable (not too small) on large screens
- [ ] **RES-083**: No cramped or squeezed content in corners
- [ ] **RES-084**: Touch targets remain minimum 44x44pt / 48x48dp on large screens
- [ ] **RES-085**: Content width limited to prevent lines from being too long (max 80 characters)

{{/if}}

### Orientation Handling (Important Priority)

{{#if (or (eq focus "all") (not focus))}}

- [ ] **RES-086**: Supporting both portrait and landscape orientations
- [ ] **RES-087**: State preserved during orientation change (savedInstanceState)
- [ ] **RES-088**: Layout recomputes correctly without flicker
- [ ] **RES-089**: Keyboard state managed properly during orientation transition
- [ ] **RES-090**: Video and full-screen content transitions smoothly
- [ ] **RES-091**: No content hidden or pushed off-screen in landscape
- [ ] **RES-092**: Testing orientation lock on locked devices (still reflow when possible)

{{/if}}

### Container Queries & Fluid Typography (Important Priority)

{{#if (or (eq focus "all") (not focus))}}

**Android Jetpack Compose**

- [ ] **RES-093**: Components size based on container width, not viewport
- [ ] **RES-094**: Using `BoxWithConstraints` for measuring available space
- [ ] **RES-095**: Flexible padding and spacing (not hardcoded)
- [ ] **RES-096**: Typography scales using `clamp(fontSize, min, max)`

**iOS SwiftUI**

- [ ] **RES-097**: Using `GeometryReader` for constraint-based layouts
- [ ] **RES-098**: Avoiding hardcoded frame values
- [ ] **RES-099**: Flexible spacing with `Spacer()` and relative sizing

{{/if}}

### Image & Media Responsiveness (Important Priority)

{{#if (or (eq focus "all") (not focus))}}

- [ ] **RES-100**: Images scale appropriately for different screen densities (dp/pt)
- [ ] **RES-101**: Aspect ratios preserved across screen sizes
- [ ] **RES-102**: Video player adapts to screen size (not fixed dimensions)
- [ ] **RES-103**: Lazy loading for images in scrollable lists
- [ ] **RES-104**: Using correct resolution images (not upscaling 1x assets)
- [ ] **RES-105**: PDFs, documents, and complex graphics responsive

{{/if}}

### Performance on Different Devices (Important Priority)

{{#if (or (eq focus "all") (not focus))}}

- [ ] **RES-106**: No layout thrashing from repeated measurements
- [ ] **RES-107**: Recomposition optimized for dynamic layouts
- [ ] **RES-108**: Using `remember` and `key` to minimize redraws
- [ ] **RES-109**: Testing on low-end devices (not just flagships)
- [ ] **RES-110**: Frame rate maintained during layout transitions
- [ ] **RES-111**: Memory usage acceptable across device categories

{{/if}}

### Testing Coverage (Important Priority)

{{#if (or (eq focus "all") (not focus))}}

- [ ] **RES-112**: Tested on compact phones (4.0"-5.5")
- [ ] **RES-113**: Tested on large phones (6.0"-6.5")
- [ ] **RES-114**: Tested on small tablets (7"-8.4")
- [ ] **RES-115**: Tested on large tablets (10"-13")
- [ ] **RES-116**: Tested on foldables in multiple postures (if applicable)
- [ ] **RES-117**: Tested in both portrait and landscape
- [ ] **RES-118**: Tested with split-screen multi-window enabled
- [ ] **RES-119**: Tested with Dynamic Type / font scaling enabled
- [ ] **RES-120**: Tested with accessibility features enabled

{{/if}}

## Code to Audit

```kotlin
{{code}}
```

## Output Format

Generate a structured responsive design audit report:

```markdown
## Responsive Design Audit Results: [Component/Screen Name]

### Critical Issues (🔴)
Issues that break layout or make content inaccessible on specific device types.
Each with: Line number, affected device type(s), description, severity, fix suggestion.

### Important Issues (🟠)
Issues that violate responsive design best practices or degrade UX on specific sizes.

### Warnings (🟡)
Layout patterns that work but should be improved for better coverage.

### Suggestions (🔵)
Polish opportunities for comprehensive responsive design.

### Device Coverage Summary
- Compact phones (< 600dp): [Supported / Needs work]
- Large phones (600-839dp): [Supported / Needs work]
- Medium tablets (840dp-1199dp): [Supported / Needs work]
- Large tablets (1200-1599dp): [Supported / Needs work]
- Extra-large tablets (> 1600dp): [Supported / Needs work]
- Foldables: [Supported / Not applicable / Needs work]
- Split-screen: [Supported / Not applicable / Needs work]

### Text Scaling Support
- Dynamic Type (iOS): [Full support / Partial / Not supported]
- Font scaling (Android): [Full support / Partial / Not supported]
- Max text size tested: [X%]

### Critical Breakpoints
- List detected breakpoints in dp/pt
- Navigation mode transitions
- Layout reflow points

### Recommended Actions (Priority Order)
1. [Most urgent fix for critical device type]
2. [Second priority]
3. [Third priority]

### Testing Checklist for Implementation
- [ ] Test on phone (portrait + landscape)
- [ ] Test on tablet (portrait + landscape)
- [ ] Test with Dynamic Type / font scaling at 100%, 150%, 200%
- [ ] Test split-screen (if applicable)
- [ ] Test foldable states (if applicable)
- [ ] Verify touch targets (min 44x44pt / 48x48dp)
- [ ] Verify text contrast at all sizes
- [ ] Performance check on low-end device
```

## Example Output

```markdown
## Responsive Design Audit Results: EventListScreen.kt

### Critical Issues (🔴)

**Line 24**: Fixed width Column breaks on tablets and foldables
```kotlin
// BAD - Fixed to phone width
Column(modifier = Modifier.width(400.dp)) {
    EventCard(event = event)
}

// GOOD - Responsive width based on size class
val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass
Column(
    modifier = when {
        windowSizeClass.windowWidthSizeClass == WindowWidthSizeClass.Compact ->
            Modifier.width(400.dp)
        windowSizeClass.windowWidthSizeClass == WindowWidthSizeClass.Medium ->
            Modifier.width(600.dp)
        else -> Modifier.fillMaxWidth()
    }
) {
    EventCard(event = event)
}
```
Severity: 🔴 Critical | Confidence: 95 | Rule: RES-001
Affected devices: Tablets, foldables in expanded mode, landscape phones
---

**Line 45**: Hardcoded text size doesn't scale with accessibility settings
```kotlin
// BAD
Text(text = event.name, fontSize = 16.sp)

// GOOD
Text(text = event.name, style = MaterialTheme.typography.titleMedium)
```
Severity: 🔴 Critical | Confidence: 90 | Rule: RES-033
Affected devices: All devices with user font scaling enabled
---

### Important Issues (🟠)

**Line 67**: Navigation not adapting to screen size
```kotlin
// BAD - Always shows bottom nav regardless of screen size
BottomNavigation { ... }

// GOOD - Use NavigationSuiteScaffold for automatic adaptation
NavigationSuiteScaffold(
    navigationSuiteItems = { /* items */ }
) { /* content */ }
```
Severity: 🟠 Important | Confidence: 85 | Rule: RES-013
Affected devices: Tablets (should use navigation rail)
---

### Warnings (🟡)

**Line 89**: Image aspect ratio not locked, may distort on tablets
Consider wrapping with `ContentScale.Crop` or explicit aspectRatio modifier.
Severity: 🟡 Warning | Confidence: 75 | Rule: RES-101

### Device Coverage Summary
- Compact phones (< 600dp): ✓ Supported
- Large phones (600-839dp): ⚠️ Needs work (nav should switch to rail)
- Medium tablets (840dp-1199dp): ❌ Broken (hardcoded widths)
- Large tablets (1200-1599dp): ❌ Broken
- Extra-large tablets (> 1600dp): ❌ Broken
- Foldables: ❌ Not tested (no hinge detection)
- Split-screen: ⚠️ Untested

### Text Scaling Support
- Dynamic Type (iOS): Not applicable to this code
- Font scaling (Android): Partial (uses hardcoded sp values)
- Max text size tested: 100% (default)

### Critical Breakpoints Detected
- Implicit: 400dp (hardcoded column width)
- Missing: 600dp, 840dp, 1200dp size class boundaries

### Recommended Actions
1. Replace fixed widths with `currentWindowAdaptiveInfo()` for true responsiveness
2. Implement NavigationSuiteScaffold for automatic nav adaptation
3. Remove hardcoded font sizes, use MaterialTheme typography
4. Add hinge detection for foldable devices using Jetpack WindowManager
5. Test full layout on tablet/foldable emulators before production

### Testing Checklist for Implementation
- [ ] Test on Pixel 4 (compact phone)
- [ ] Test on Pixel 6 (large phone - landscape)
- [ ] Test on Pixel Tablet (medium tablet)
- [ ] Test on iPad Air (large tablet - landscape)
- [ ] Test with font scale at 150%
- [ ] Test split-screen with another app
- [ ] Verify touch targets remain 48x48dp minimum
- [ ] Check frame rate during layout transitions
```

## Reference Resources

### Android Responsive Design
- [Build adaptive apps | Jetpack Compose](https://developer.android.com/develop/ui/compose/build-adaptive-apps)
- [Learn about foldables](https://developer.android.com/guide/topics/large-screens/learn-about-foldables)
- [Support multi-window mode](https://developer.android.com/develop/ui/compose/layouts/adaptive/support-multi-window-mode)
- [Compose Material 3 Adaptive](https://developer.android.com/develop/ui/compose/layouts/adaptive)
- [WindowManager for foldables](https://developer.android.com/codelabs/android-window-manager-dual-screen-foldables)
- [Designing for foldables | Samsung Developer](https://developer.samsung.com/one-ui/largescreen-and-foldable/designing_for_foldable.html)

### iOS Responsive Design
- [Size Classes in Swift (2025)](https://www.bitcot.com/designing-for-diversity-with-size-classes-layout-swift/)
- [Dynamic Type Guide](https://developer.apple.com/documentation/uikit/scaling-fonts-automatically)
- [Design for iPad](https://designcode.io/ios-design-handbook-design-for-ipad/)
- [SwiftUI Responsive Design](https://ravi6997.medium.com/responsive-design-in-swiftui-stop-hardcoding-layout-for-iphone-only-build-apps-that-truly-adapt-69e5c975e112)

### 2025-2026 Best Practices
- [Responsive Design 2026 Trends](https://www.keelis.com/blog/responsive-web-design-in-2026:-trends-and-best-practices)
- [Foldable UX Design 2025](https://www.influencers-time.com/designing-ux-for-foldable-and-multi-surface-devices-in-2025/)
- [Mobile App Design Guidelines 2025](https://medium.com/@CarlosSmith24/mobile-app-design-guidelines-for-ios-and-android-in-2025-82e83f0b942b)

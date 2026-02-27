---
name: responsive-layouts
description: Implement responsive layouts for mobile using WindowSizeClass (Android) and GeometryReader (iOS). Covers breakpoints, safe areas, adaptive navigation, tablet layouts, foldables, and device orientation handling.
args:
  - name: platform
    type: string
    enum: [android, ios, both]
    description: Target platform (default both)
    required: false
  - name: screen_type
    type: string
    enum: [phone, tablet, foldable, all]
    description: Focus on specific screen type
    required: false
  - name: focus
    type: string
    enum: [breakpoints, navigation, safe-areas, typography, spacing, grid-layout, orientation, foldables, all]
    description: Implementation focus area
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Responsive Layouts Implementation Guide

Comprehensive guide to implementing responsive layouts across iOS and Android platforms in 2025-2026. Covers breakpoints, safe area handling, adaptive navigation, tablet/foldable support, and device orientation.

## Instructions

Choose your target platform and focus area. Use WindowSizeClass (Android) or GeometryReader (iOS) to build layouts that adapt to phones, tablets, foldables, and split-screen modes. Include safe area handling for notches, Dynamic Island, gesture areas, and foldable hinges.

## Responsive Layouts Checklist

### RL-001 to RL-010: Screen Size Breakpoints Definition

{{#if (or (eq focus "breakpoints") (eq focus "all") (not focus))}}

**Android Breakpoint Strategy**

- [ ] **RL-001**: Define compact breakpoint (< 600dp width) for phones
- [ ] **RL-002**: Define medium breakpoint (600dp-839dp width) for small tablets
- [ ] **RL-003**: Define expanded breakpoint (>= 840dp width) for large tablets
- [ ] **RL-004**: Define large breakpoint (1200dp-1599dp) for extra-large tablets
- [ ] **RL-005**: Define extra-large breakpoint (>= 1600dp) for foldables and landscape

**iOS Size Classes**

- [ ] **RL-006**: Use horizontal size class (compact vs regular) for iPad detection
- [ ] **RL-007**: Use vertical size class (compact vs regular) for orientation detection
- [ ] **RL-008**: Define custom breakpoints using GeometryReader for precise spacing
- [ ] **RL-009**: Test on 5.5", 6.5", 7.9", 10.5", 11", 12.9" screens
- [ ] **RL-010**: Document all breakpoint transitions and layout changes

{{/if}}

### RL-011 to RL-025: WindowSizeClass (Android) & GeometryReader (iOS)

{{#if (or (eq focus "breakpoints") (eq focus "all") (not focus))}}

**Android WindowSizeClass Implementation**

- [ ] **RL-011**: Import `androidx.compose.material3.adaptive` library
- [ ] **RL-012**: Use `currentWindowAdaptiveInfo()` to get window size class
- [ ] **RL-013**: Store WindowSizeClass in ViewModel for state management
- [ ] **RL-014**: Use `windowSizeClass.windowWidthSizeClass` for width-based decisions
- [ ] **RL-015**: Use `windowSizeClass.windowHeightSizeClass` for height-based decisions
- [ ] **RL-016**: Implement fluid layout logic using when expressions for size classes
- [ ] **RL-017**: Use `BoxWithConstraints` for measuring available container space
- [ ] **RL-018**: Avoid hardcoded dimensions - replace with responsive alternatives
- [ ] **RL-019**: Test on emulator at multiple screen sizes (Pixel 4, Pixel 6, Pixel Tablet)
- [ ] **RL-020**: Verify recomposition works correctly when window size changes

**iOS GeometryReader Implementation**

- [ ] **RL-021**: Use `GeometryReader` to access parent container dimensions
- [ ] **RL-022**: Access geometry proxy dimensions: `geometry.size.width`, `geometry.size.height`
- [ ] **RL-023**: Calculate responsive padding: `geometry.size.width * 0.05` for 5% margins
- [ ] **RL-024**: Combine GeometryReader with environment size classes for complete adaptation
- [ ] **RL-025**: Avoid greedy GeometryReader inside ScrollView (use constraints instead)

{{/if}}

### RL-026 to RL-045: Safe Area Handling

{{#if (or (eq focus "safe-areas") (eq focus "all") (not focus))}}

**Notch & Dynamic Island (iOS)**

- [ ] **RL-026**: Apply `.ignoresSafeArea()` only to background layers, not content
- [ ] **RL-027**: Use `@Environment(\.safeAreaInsets)` to detect safe area bounds
- [ ] **RL-028**: Add padding to content: use `safe area inset + 16dp/pt` spacing
- [ ] **RL-029**: Test on iPhone 14/15 (Dynamic Island) and iPhone X/11 (notch)
- [ ] **RL-030**: Verify important content (buttons, text) not hidden by notch/island

**Status Bar & Navigation Bar (Android)**

- [ ] **RL-031**: Use `WindowCompat.setDecorFitsSystemWindows(window, false)` for edge-to-edge
- [ ] **RL-032**: Apply `systemBarsPadding()` modifier to content with status bar
- [ ] **RL-033**: Apply `navigationBarsPadding()` modifier for bottom nav spacing
- [ ] **RL-034**: Apply `imePadding()` for keyboard-aware layouts
- [ ] **RL-035**: Test with gesture navigation (pill) vs button navigation (Android 12+)

**Gesture Areas (Both Platforms)**

- [ ] **RL-036**: Ensure 44pt/48dp minimum touch target on bottom edges (iOS swipe-up, Android gesture pill)
- [ ] **RL-037**: Avoid interactive content in bottom 44pt/48dp of screen
- [ ] **RL-038**: Add padding above bottom navigation: minimum 8dp/pt clearance
- [ ] **RL-039**: Test with landscape orientation (reduced vertical safe area)
- [ ] **RL-040**: Verify no overlapping UI in gesture exclusion zones

**Foldable Hinge Handling (Android)**

- [ ] **RL-041**: Use Jetpack WindowManager `displayFeatures` API
- [ ] **RL-042**: Detect hinge position with `FoldingFeature.bounds`
- [ ] **RL-043**: Calculate safe zones: split screen into areas avoiding hinge
- [ ] **RL-044**: Add minimum 16dp padding from hinge on both sides
- [ ] **RL-045**: Test on Galaxy Fold emulator with hinge at different angles

{{/if}}

### RL-046 to RL-065: Tablet Layout Adaptation

{{#if (or (eq focus "navigation") (eq focus "all") (not focus))}}

**Android Tablet Navigation (Medium & Expanded)**

- [ ] **RL-046**: Use `NavigationSuiteScaffold` for automatic mode switching
- [ ] **RL-047**: Configure to show bottom nav on compact (phones)
- [ ] **RL-048**: Configure to show navigation rail on medium (7-9" tablets)
- [ ] **RL-049**: Configure to show permanent drawer on expanded (10"+ tablets)
- [ ] **RL-050**: Adjust rail width: 80dp standard, 96dp for touch-friendly
- [ ] **RL-051**: Implement master-detail layout: list on left (320dp), detail on right

**iOS iPad Layout (Regular Width)**

- [ ] **RL-052**: Use `@Environment(\.horizontalSizeClass)` to detect iPad
- [ ] **RL-053**: Show sidebar on left (200-280pt) for iPads in regular width
- [ ] **RL-054**: Use `NavigationSplitView` for master-detail on iPad
- [ ] **RL-055**: Hide navigation on detail screen (sidebar replaces it)
- [ ] **RL-056**: Implement two-column layout for list + detail side-by-side
- [ ] **RL-057**: Adjust content width max: 800pt to prevent line lengths > 80 chars

**Tablet Content Optimization**

- [ ] **RL-058**: Increase horizontal padding to 24dp/pt on tablets
- [ ] **RL-059**: Increase vertical spacing by 20% on medium screens
- [ ] **RL-060**: Use grid layouts instead of single-column scrolling (2-3 columns)
- [ ] **RL-061**: Increase touch targets to 56dp/pt minimum (was 48dp/pt)
- [ ] **RL-062**: Enlarge typography by 10% for readability on larger screens
- [ ] **RL-063**: Add dividers between master and detail panes
- [ ] **RL-064**: Test content visibility in split-screen (50/50, 60/40 split)
- [ ] **RL-065**: Verify scrolling behavior in two-column layouts

{{/if}}

### RL-066 to RL-085: Landscape Orientation Handling

{{#if (or (eq focus "orientation") (eq focus "all") (not focus))}}

**Portrait to Landscape Transitions**

- [ ] **RL-066**: Enable orientation changes: remove `android:screenOrientation` from manifest
- [ ] **RL-067**: Preserve state during rotation using `savedInstanceState` or ViewModel
- [ ] **RL-068**: Adjust layout for 50% reduced height in landscape
- [ ] **RL-069**: Use horizontal scrolling for content wider than landscape viewport
- [ ] **RL-070**: Hide non-critical UI elements in landscape to maximize content
- [ ] **RL-071**: Test transition without flicker (use `rememberSaveable()` in Compose)

**Landscape-Specific Layouts**

- [ ] **RL-072**: Use horizontal layout for phone in landscape (side-by-side panels)
- [ ] **RL-073**: Stack content vertically with max height constraints
- [ ] **RL-074**: Adjust bottom navigation: hide labels, show icons only (landscape)
- [ ] **RL-075**: Implement swipeable tabs for landscape navigation
- [ ] **RL-076**: Test keyboard input in landscape (software keyboard takes space)

**Video & Full-Screen Content**

- [ ] **RL-077**: Implement automatic rotation for video players
- [ ] **RL-078**: Hide system UI and gesture bars during full-screen video
- [ ] **RL-079**: Maintain 16:9 aspect ratio on all orientations
- [ ] **RL-080**: Test video scrubbing in landscape
- [ ] **RL-081**: Implement orientation lock for content that shouldn't rotate
- [ ] **RL-082**: Support landscape layout lock toggle (accessibility)
- [ ] **RL-083**: Verify no content hidden or cut off at edges
- [ ] **RL-084**: Test with both system and app-level rotation locks
- [ ] **RL-085**: Preserve video playback position during orientation change

{{/if}}

### RL-086 to RL-115: Foldable Device Support

{{#if (or (eq focus "foldables") (eq focus "all") (not focus))}}

**Foldable Posture Detection**

- [ ] **RL-086**: Add Jetpack WindowManager dependency
- [ ] **RL-087**: Collect `WindowInfoRepository.windowLayoutInfo` in ViewModel
- [ ] **RL-088**: Detect `FoldingFeature.orientation` (horizontal vs vertical fold)
- [ ] **RL-089**: Detect `FoldingFeature.State.FLAT` (fully unfolded)
- [ ] **RL-090**: Detect `FoldingFeature.State.HALF_OPENED` (book/tabletop mode)
- [ ] **RL-091**: Detect `FoldingFeature.State.FULLY_OPENED` (unfolded but at angle)
- [ ] **RL-092**: Define safe zones based on hinge location
- [ ] **RL-093**: Test on Galaxy Fold emulator with different fold angles

**Dual-Pane Foldable Layouts**

- [ ] **RL-094**: Use vertical hinge -> split content left/right (book posture)
- [ ] **RL-095**: Use horizontal hinge -> split content top/bottom (tabletop posture)
- [ ] **RL-096**: Implement navigation on one pane, content on other
- [ ] **RL-097**: Size panes equally: 50/50 split recommended
- [ ] **RL-098**: Add 8dp padding from hinge on both sides
- [ ] **RL-099**: Test scrolling across both panes
- [ ] **RL-100**: Verify no content hidden behind fold

**Single-Pane Folded Layouts**

- [ ] **RL-101**: When device FLAT, use single-pane phone layout
- [ ] **RL-102**: Automatically switch to dual-pane when partially opened
- [ ] **RL-103**: Preserve state transitions without data loss
- [ ] **RL-104**: Test rapid open/close (performance)
- [ ] **RL-105**: Verify animations smooth during posture changes

**Samsung Galaxy Fold Specific**

- [ ] **RL-106**: Support front screen (smaller) vs main screen (larger) transitions
- [ ] **RL-107**: Preserve app state when transitioning between screens
- [ ] **RL-108**: Test on actual Galaxy Fold device (not just emulator)
- [ ] **RL-109**: Handle cover screen aspect ratio (21:9, taller than main)
- [ ] **RL-110**: Verify navigation state preserved across screen switch

**Google Pixel Fold Specific**

- [ ] **RL-111**: Test on Pixel Fold emulator (6.3" cover, 7.6" main)
- [ ] **RL-112**: Verify aspect ratio differences handled
- [ ] **RL-113**: Test content scaling between cover and main screen
- [ ] **RL-114**: Verify crease/hinge detection works correctly
- [ ] **RL-115**: Test multi-window mode on unfolded Pixel Fold

{{/if}}

### RL-116 to RL-140: Responsive Typography & Spacing

{{#if (or (eq focus "typography") (eq focus "spacing") (eq focus "all") (not focus))}}

**Android Text Scaling**

- [ ] **RL-116**: Use `sp` (scale-independent pixels) for ALL text sizes (not dp)
- [ ] **RL-117**: Apply Material 3 typography: `MaterialTheme.typography.bodyMedium`
- [ ] **RL-118**: Base font size 14sp for body, 20sp for titles (scalable baseline)
- [ ] **RL-119**: Implement text scaling function: `clamp(baseSize * scale, minSp, maxSp)`
- [ ] **RL-120**: Respect `fontScale` from `Configuration` (AccessibilityManager)
- [ ] **RL-121**: Test with system font scale: 100%, 125%, 150%, 200%
- [ ] **RL-122**: Verify text readable at 200% scale without truncation
- [ ] **RL-123**: Increase line height proportionally with text size
- [ ] **RL-124**: Update touch targets when text scale changes

**iOS Dynamic Type**

- [ ] **RL-125**: Use TextStyle categories: `body`, `headline`, `title1`, `caption`
- [ ] **RL-126**: Never hardcode font sizes - use `.font(.body)` or `.font(.title1)`
- [ ] **RL-127**: Apply `UIFontMetrics` wrapper for custom fonts to scale
- [ ] **RL-128**: Test at all 12 size levels (xSmall to AX5, accessibility sizes)
- [ ] **RL-129**: Verify text scales to 200%+ without breaking layout
- [ ] **RL-130**: Use `lineLimit(nil)` to allow text wrapping at all sizes
- [ ] **RL-131**: Test at extremes: `.extraSmall` and `.accessibilityExtraExtraExtraLarge`
- [ ] **RL-132**: Increase line height at large text sizes (leading: 1.5x at AX sizes)

**Responsive Spacing System**

- [ ] **RL-133**: Define base unit: 8dp/pt (phone), 16dp/pt (tablet)
- [ ] **RL-134**: Scale padding: 2x, 3x, 4x multiples of base unit
- [ ] **RL-135**: Adjust spacing based on screen size: 12dp (phone) -> 24dp (tablet)
- [ ] **RL-136**: Use `clamp()` for fluid spacing: `clamp(12.dp, 2.vw, 48.dp)`
- [ ] **RL-137**: Increase gap between grid items on larger screens
- [ ] **RL-138**: Scale margins proportionally with content size
- [ ] **RL-139**: Test spacing at all breakpoints (compact, medium, expanded)
- [ ] **RL-140**: Verify whitespace readable (not cramped) on all screen sizes

{{/if}}

### RL-141 to RL-160: Responsive Grid & Layout Systems

{{#if (or (eq focus "grid-layout") (eq focus "all") (not focus))}}

**Android Responsive Grid**

- [ ] **RL-141**: Use `LazyVerticalGrid` with adaptive column count
- [ ] **RL-142**: Calculate columns dynamically: `maxWidth / itemWidth`
- [ ] **RL-143**: Example: 1 column (< 400dp), 2 columns (400-600dp), 3 columns (600+dp)
- [ ] **RL-144**: Use `GridCells.Adaptive(300.dp)` to auto-fit columns
- [ ] **RL-145**: Apply `horizontalArrangement = Arrangement.spacedBy(16.dp)`
- [ ] **RL-146**: Adjust item height to maintain aspect ratio across screens
- [ ] **RL-147**: Test grid with overflow (too many items)
- [ ] **RL-148**: Verify column wrapping works at all breakpoints

**iOS Responsive Grid**

- [ ] **RL-149**: Use SwiftUI `Grid` view (iOS 16+) for responsive layouts
- [ ] **RL-150**: Define columns: `let columns = Array(repeating: GridItem(), count: columnCount)`
- [ ] **RL-151**: Calculate column count from geometry: `Int(geometry.size.width / 300)`
- [ ] **RL-152**: Use `LazyVGrid` with `spacing:` parameter
- [ ] **RL-153**: Apply `.frame(minHeight: itemHeight)` to maintain aspect ratio
- [ ] **RL-154**: Test on iPhone (1 column), iPad (2-3 columns)
- [ ] **RL-155**: Verify grid reflows when orientation changes

**Container Queries (Future)**

- [ ] **RL-156**: Monitor container width, not viewport width
- [ ] **RL-157**: Use `BoxWithConstraints` to measure available width
- [ ] **RL-158**: Adjust component size based on container (not screen)
- [ ] **RL-159**: Example: Card width adapts to Column, not screen width
- [ ] **RL-160**: Enable components to be responsive in different contexts

{{/if}}

### RL-161 to RL-180: Multi-Window & Split-Screen

{{#if (or (eq focus "navigation") (eq focus "all") (not focus))}}

**Android Multi-Window Support**

- [ ] **RL-161**: Set `android:resizableActivity="true"` in manifest
- [ ] **RL-162**: Handle multi-resume state correctly (Android 10+)
- [ ] **RL-163**: Listen for `Configuration.SCREENLAYOUT_SIZE_MASK` changes
- [ ] **RL-164**: Ensure content readable at minimal split width (320dp minimum)
- [ ] **RL-165**: Use `AndroidX.window:window` for Activity Embedding
- [ ] **RL-166**: Define split-pair rules: when to show detail beside master
- [ ] **RL-167**: Test 50/50 split, 60/40, 40/60 variations
- [ ] **RL-168**: Verify dialogs don't span both split panes
- [ ] **RL-169**: Test with side-by-side apps (e.g., app + Settings)
- [ ] **RL-170**: Preserve app state when split configuration changes

**iOS Split View & Multi-Window**

- [ ] **RL-171**: Use `NavigationSplitView` for adaptive master-detail
- [ ] **RL-172**: Hide sidebar on phones (compact), show on iPad (regular)
- [ ] **RL-173**: Support multiple windows with `@Environment(\.scenePhase)`
- [ ] **RL-174**: Use `SceneDelegate` to manage window lifecycle
- [ ] **RL-175**: Test with Picture-in-Picture (iPadOS 17+)
- [ ] **RL-176**: Verify state sync across multiple windows
- [ ] **RL-177**: Test dragging app tile to split view
- [ ] **RL-178**: Handle window resize events
- [ ] **RL-179**: Test iPad multi-window restore on app launch
- [ ] **RL-180**: Verify gesture navigation works in split-screen

{{/if}}

### RL-181 to RL-200: Responsive Images & Media

{{#if (or (eq focus "all") (not focus))}}

**Image Asset Strategy**

- [ ] **RL-181**: Provide 1x, 2x, 3x assets for different pixel densities
- [ ] **RL-182**: Use vector graphics (SVG, custom Compose, SwiftUI shapes)
- [ ] **RL-183**: Apply `ContentScale.Crop` for thumbnail aspect ratio preservation
- [ ] **RL-184**: Lazy-load images in scrollable lists (cache thumbnails)
- [ ] **RL-185**: Test image quality on small (phone) vs large (tablet) screens
- [ ] **RL-186**: Verify no pixelation on high-DPI displays

**Responsive Image Sizing**

- [ ] **RL-187**: Use `Modifier.aspectRatio()` to lock aspect ratio
- [ ] **RL-188**: Scale image to container: `fillMaxWidth()` + aspectRatio
- [ ] **RL-189**: Set max width (e.g., 600.dp) to prevent huge images on tablets
- [ ] **RL-190**: Calculate dynamic image height: `width * aspectRatio`
- [ ] **RL-191**: Apply different max widths per breakpoint (400.dp, 600.dp, 800.dp)

**Video & Rich Media**

- [ ] **RL-192**: Responsive video player: adapt to screen width
- [ ] **RL-193**: Use 16:9 aspect ratio by default (most common)
- [ ] **RL-194**: Allow full-screen video with proper safe area handling
- [ ] **RL-195**: Test video playback on low-bandwidth connections
- [ ] **RL-196**: Verify closed captions visible on all screen sizes
- [ ] **RL-197**: Test with pinch-zoom on video (should not break)
- [ ] **RL-198**: PDF documents: scale to fit width or height (user preference)
- [ ] **RL-199**: Charts: responsive axis labels, legend positioning
- [ ] **RL-200**: Maps: zoom level adapts to viewport size

{{/if}}

### RL-201 to RL-220: Testing Matrix

{{#if (or (eq focus "all") (not focus))}}

**Device Testing Matrix**

- [ ] **RL-201**: Pixel 4 / iPhone 12 (5.8" compact phone) - portrait + landscape
- [ ] **RL-202**: Pixel 6 / iPhone 14 (6.1" large phone) - portrait + landscape
- [ ] **RL-203**: Pixel Tablet / iPad Air (10.9" medium tablet) - portrait + landscape
- [ ] **RL-204**: iPad Pro 12.9" / Samsung Tab S9 (12.9" large tablet) - portrait + landscape
- [ ] **RL-205**: Galaxy Fold (cover + main screens) - multiple postures
- [ ] **RL-206**: Pixel Fold (6.3" cover, 7.6" main) - FLAT + HALF_OPENED
- [ ] **RL-207**: Foldable emulator (Android emulator with fold simulation)
- [ ] **RL-208**: iPhone 16 Pro (Dynamic Island) - safe area handling
- [ ] **RL-209**: Landscape phone (minimal vertical space) - UI adaptation
- [ ] **RL-210**: Split-screen 50/50 (Android emulator side-by-side)

**Orientation Testing**

- [ ] **RL-211**: Portrait on all phones and tablets
- [ ] **RL-212**: Landscape on all phones and tablets
- [ ] **RL-213**: Reverse portrait (upside down, if supported)
- [ ] **RL-214**: Reverse landscape (right-to-left)
- [ ] **RL-215**: Orientation changes during app session (no state loss)
- [ ] **RL-216**: Rapid orientation flips (stress test)

**Text Scaling Testing**

- [ ] **RL-217**: Android: 100%, 125%, 150%, 175%, 200% font scale
- [ ] **RL-218**: iOS: All 12 Dynamic Type sizes (.small, .regular, .large, AX sizes)
- [ ] **RL-219**: Verify no truncation at maximum text size
- [ ] **RL-220**: Test layout stability with text scaling enabled

{{/if}}

### RL-221 to RL-245: Common Mistakes & Fixes

{{#if (or (eq focus "all") (not focus))}}

**Layout Anti-Patterns**

- [ ] **RL-221**: MISTAKE - Hardcoded widths (e.g., `width = 400.dp`)
  - FIX: Use `fillMaxWidth()`, `WindowSizeClass`, or `BoxWithConstraints`
- [ ] **RL-222**: MISTAKE - Hardcoded heights without aspect ratio (e.g., `height = 200.dp`)
  - FIX: Lock aspect ratio with `.aspectRatio()` or responsive padding-bottom
- [ ] **RL-223**: MISTAKE - Single-column layout on tablet
  - FIX: Switch to 2-3 column grid using `LazyVerticalGrid` or `Grid`
- [ ] **RL-224**: MISTAKE - Bottom navigation on large screens
  - FIX: Use `NavigationSuiteScaffold` (Android) or `NavigationSplitView` (iOS)
- [ ] **RL-225**: MISTAKE - Hardcoded font sizes (e.g., `fontSize = 16.sp`)
  - FIX: Use Typography system or scale with `clamp()`
- [ ] **RL-226**: MISTAKE - No safe area padding (content under notch/island)
  - FIX: Apply `systemBarsPadding()` and test on iPhone 14+
- [ ] **RL-227**: MISTAKE - GeometryReader at root (greedy view)
  - FIX: Use only when needed for layout measurement, wrap content inside

**Navigation Anti-Patterns**

- [ ] **RL-228**: MISTAKE - Always show bottom nav (even on tablet)
  - FIX: Use `NavigationSuiteScaffold` for automatic switching
- [ ] **RL-229**: MISTAKE - Hamburger menu on tablet (waste of space)
  - FIX: Show permanent sidebar using `NavigationRail` or `PermanentNavigationDrawer`
- [ ] **RL-230**: MISTAKE - No master-detail layout on iPad
  - FIX: Implement `NavigationSplitView` for side-by-side navigation
- [ ] **RL-231**: MISTAKE - Navigation state not preserved across rotation
  - FIX: Use ViewModel to persist navigation state (not SavedInstanceState)

**Foldable Anti-Patterns**

- [ ] **RL-232**: MISTAKE - No hinge detection, content overlaps fold
  - FIX: Implement `FoldingFeature` detection and safe zones
- [ ] **RL-233**: MISTAKE - Same layout for folded and unfolded state
  - FIX: Use dual-pane (unfolded) vs single-pane (folded) layout switching
- [ ] **RL-234**: MISTAKE - No padding from hinge (content too close)
  - FIX: Add 8-16dp padding on both sides of hinge
- [ ] **RL-235**: MISTAKE - Failing to test on actual foldable device
  - FIX: Use Galaxy Fold / Pixel Fold emulator or real device

**Multi-Window Anti-Patterns**

- [ ] **RL-236**: MISTAKE - Content unreadable in split-screen (too narrow)
  - FIX: Set `android:resizableActivity=true` and test at 320dp minimum width
- [ ] **RL-237**: MISTAKE - State not preserved in multi-window
  - FIX: Use ViewModel (not Activity state) for data persistence
- [ ] **RL-238**: MISTAKE - Dialogs spanning both split panes
  - FIX: Constrain dialog to single pane using WindowManager Rules

**Text Scaling Anti-Patterns**

- [ ] **RL-239**: MISTAKE - Hardcoded line heights with large text
  - FIX: Scale line height proportionally: `baseHeight * scale`
- [ ] **RL-240**: MISTAKE - Text truncation at large sizes
  - FIX: Use `lineLimit(nil)` and allow text wrapping
- [ ] **RL-241**: MISTAKE - Touch targets shrink with text scale
  - FIX: Use fixed minimum touch size (48dp/pt) independent of text scale

**Performance Anti-Patterns**

- [ ] **RL-242**: MISTAKE - Expensive layout calculations in recomposition
  - FIX: Cache measurements, use `remember`, avoid recalculating breakpoints
- [ ] **RL-243**: MISTAKE - No memoization of WindowSizeClass / GeometryReader
  - FIX: Use `remember { }` and only recompute when dependencies change
- [ ] **RL-244**: MISTAKE - Full recomposition on layout change
  - FIX: Use `key()` to preserve state during transitions
- [ ] **RL-245**: MISTAKE - No testing on low-end devices
  - FIX: Test on Pixel 4a / iPhone SE (older, slower devices)

{{/if}}

---

## Code Examples

### Android: WindowSizeClass Implementation

```kotlin
// ViewModel with responsive layout state
class ResponsiveViewModel : ViewModel() {
    val windowSizeClass = mutableStateOf<WindowSizeClass?>(null)

    fun updateWindowSizeClass(newSizeClass: WindowSizeClass) {
        windowSizeClass.value = newSizeClass
    }
}

// Main composable
@Composable
fun ResponsiveApp(viewModel: ResponsiveViewModel = hiltViewModel()) {
    val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass
    viewModel.updateWindowSizeClass(windowSizeClass)

    val isCompact = windowSizeClass.windowWidthSizeClass == WindowWidthSizeClass.Compact
    val isMedium = windowSizeClass.windowWidthSizeClass == WindowWidthSizeClass.Medium
    val isExpanded = windowSizeClass.windowWidthSizeClass == WindowWidthSizeClass.Expanded

    when {
        isCompact -> PhoneLayout()
        isMedium -> TabletLayoutTwoColumn()
        else -> TabletLayoutThreeColumn()
    }
}

// Responsive list with adaptive columns
@Composable
fun ResponsiveGrid(items: List<Item>, windowSizeClass: WindowSizeClass) {
    val columnCount = when (windowSizeClass.windowWidthSizeClass) {
        WindowWidthSizeClass.Compact -> 1
        WindowWidthSizeClass.Medium -> 2
        else -> 3
    }

    LazyVerticalGrid(
        columns = GridCells.Fixed(columnCount),
        horizontalArrangement = Arrangement.spacedBy(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp),
        modifier = Modifier.padding(16.dp)
    ) {
        items(items.size) { index ->
            ItemCard(items[index])
        }
    }
}

// Safe area handling
@Composable
fun SafeAreaScreen() {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .systemBarsPadding()  // Handles notch + status bar
            .navigationBarsPadding()  // Handles gesture pill
            .imePadding()  // Adjusts for keyboard
    ) {
        // Content here is safely inset from system UI
    }
}

// Responsive typography
@Composable
fun ResponsiveText(text: String, baseSize: TextUnit = 16.sp) {
    val currentDensity = LocalDensity.current
    val responsiveSize = remember(baseSize) {
        with(currentDensity) {
            // Clamp between min (14sp) and max (32sp)
            baseSize.coerceIn(14.sp, 32.sp)
        }
    }

    Text(
        text = text,
        fontSize = responsiveSize,
        style = MaterialTheme.typography.bodyMedium
    )
}
```

### iOS: GeometryReader Implementation

```swift
// View with responsive layout using GeometryReader
struct ResponsiveLayout: View {
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    @Environment(\.verticalSizeClass) var verticalSizeClass

    var isCompact: Bool {
        horizontalSizeClass == .compact
    }

    var isRegular: Bool {
        horizontalSizeClass == .regular
    }

    var body: some View {
        GeometryReader { geometry in
            let width = geometry.size.width
            let height = geometry.size.height

            // Calculate column count based on available width
            let columnCount = Int(width / 300) // 300pt per column minimum

            if isCompact {
                // Phone layout: single column
                PhoneLayout(width: width)
            } else if width > 600 && width < 1000 {
                // Small tablet: 2 columns
                TwoColumnLayout(width: width, columnCount: 2)
            } else {
                // Large tablet: 3 columns
                ThreeColumnLayout(width: width, columnCount: 3)
            }
        }
        .ignoresSafeArea(edges: .top) // Only for background
        .safeAreaInset(edge: .top) {
            Color.clear.frame(height: 0) // Proper safe area handling
        }
    }
}

// Master-detail split view for iPad
struct ContentView: View {
    @Environment(\.horizontalSizeClass) var horizontalSizeClass
    @State private var selectedItem: Item? = nil

    var body: some View {
        if horizontalSizeClass == .compact {
            // iPhone: stack navigation
            NavigationStack {
                ListView(selectedItem: $selectedItem)
            }
        } else {
            // iPad: side-by-side
            NavigationSplitView {
                ListView(selectedItem: $selectedItem)
            } detail: {
                if let item = selectedItem {
                    DetailView(item: item)
                } else {
                    Text("Select an item")
                }
            }
        }
    }
}

// Responsive grid using GeometryReader
struct ResponsiveGrid<Item: Identifiable>: View {
    let items: [Item]
    let content: (Item) -> AnyView

    var body: some View {
        GeometryReader { geometry in
            let columnCount = max(1, Int(geometry.size.width / 300))
            let columns = Array(repeating: GridItem(.flexible(), spacing: 16), count: columnCount)

            ScrollView {
                LazyVGrid(columns: columns, spacing: 16) {
                    ForEach(items) { item in
                        content(item)
                    }
                }
                .padding(16)
            }
        }
    }
}

// Safe area handling with Dynamic Island
struct SafeAreaScreen: View {
    var body: some View {
        VStack {
            Text("Content with safe area padding")
                .padding(.top, 16) // Additional padding for Dynamic Island
        }
        .ignoresSafeArea(edges: .bottom)
        .safeAreaInset(edge: .bottom) {
            Color.blue.frame(height: 50)
        }
    }
}

// Responsive typography
struct ResponsiveText: View {
    let text: String
    let style: Font.TextStyle
    @Environment(\.sizeCategory) var sizeCategory

    var body: some View {
        Text(text)
            .font(.system(style, design: .default))
            .dynamicTypeSize(.extraSmall ... .accessibilityExtraExtraExtraLarge)
    }
}

// Handling orientation changes
struct OrientationChangeView: View {
    @Environment(\.verticalSizeClass) var verticalSizeClass

    var body: some View {
        if verticalSizeClass == .compact {
            // Landscape layout
            HStack {
                SidePanel()
                MainContent()
            }
        } else {
            // Portrait layout
            VStack {
                MainContent()
                SidePanel()
            }
        }
    }
}
```

### Foldable Hinge Detection (Android)

```kotlin
// ViewModel for foldable state
class FoldableViewModel : ViewModel() {
    val windowLayoutInfo = MutableStateFlow<WindowLayoutInfo?>(null)

    private val windowInfoRepository = WindowInfoRepository.getInstance(context)

    init {
        viewModelScope.launch {
            windowInfoRepository.windowLayoutInfo.collect { layoutInfo ->
                windowLayoutInfo.value = layoutInfo
            }
        }
    }

    fun getHingeBounds(): Rect? {
        return windowLayoutInfo.value?.displayFeatures
            .filterIsInstance<FoldingFeature>()
            .firstOrNull()?.bounds
    }

    fun isFoldingFeatureVertical(): Boolean {
        val hinge = windowLayoutInfo.value?.displayFeatures
            .filterIsInstance<FoldingFeature>()
            .firstOrNull() ?: return false
        return hinge.orientation == FoldingFeature.Orientation.VERTICAL
    }
}

// Composable with foldable support
@Composable
fun FoldableScreen(viewModel: FoldableViewModel = hiltViewModel()) {
    val layoutInfo by viewModel.windowLayoutInfo.collectAsState()
    val hinge = layoutInfo?.displayFeatures?.filterIsInstance<FoldingFeature>()?.firstOrNull()

    Box(modifier = Modifier.fillMaxSize()) {
        if (hinge != null && viewModel.isFoldingFeatureVertical()) {
            // Vertical hinge: split left/right
            Row(modifier = Modifier.fillMaxSize()) {
                Box(
                    modifier = Modifier
                        .weight(1f)
                        .fillMaxHeight()
                        .padding(end = 8.dp)
                ) {
                    MasterPane()
                }

                // Hinge spacer
                Spacer(modifier = Modifier.width((hinge.bounds.width()).dp))

                Box(
                    modifier = Modifier
                        .weight(1f)
                        .fillMaxHeight()
                        .padding(start = 8.dp)
                ) {
                    DetailPane()
                }
            }
        } else {
            // Single pane layout (folded or no hinge)
            SinglePaneLayout()
        }
    }
}
```

---

## Device Testing Checklist

### Phones
- [ ] Compact (4.5-5.5"): Pixel 4, iPhone 12 mini
- [ ] Large (6.0-6.5"): Pixel 6, iPhone 14 Pro Max
- [ ] Extra-large (6.7"+): Pixel 7 Pro, iPhone 15 Plus

### Tablets
- [ ] Small tablet (7-8.4"): iPad mini, Galaxy Tab A7
- [ ] Medium tablet (10-11"): iPad Air, Pixel Tablet
- [ ] Large tablet (12.9"+): iPad Pro 12.9", Galaxy Tab S9

### Foldables
- [ ] Samsung Galaxy Fold (dynamic)
- [ ] Samsung Galaxy Z Fold (landscape hinge)
- [ ] Google Pixel Fold (6.3" cover, 7.6" main)

### Orientations
- [ ] Portrait (normal, reversed)
- [ ] Landscape (normal, reversed)
- [ ] Device locked (portrait/landscape preference)

### Text Scaling (Android)
- [ ] 100% (default)
- [ ] 125% (large)
- [ ] 150% (extra large)
- [ ] 200% (accessibility)

### Text Scaling (iOS)
- [ ] `.small`, `.medium`, `.large`, `.extraLarge`
- [ ] `.accessibilityMedium`, `.accessibilityLarge`
- [ ] `.accessibilityExtraExtraExtraLarge` (maximum)

---

## Reference Resources

### Android Responsive Design
- [Build adaptive apps | Jetpack Compose](https://developer.android.com/develop/ui/compose/layouts/adaptive/use-window-size-classes)
- [Support different display sizes | Jetpack Compose](https://developer.android.com/develop/ui/compose/layouts/adaptive/support-different-display-sizes)
- [Learn about foldables](https://developer.android.com/guide/topics/large-screens/learn-about-foldables)
- [Support multi-window mode](https://developer.android.com/develop/ui/compose/layouts/adaptive/support-multi-window-mode)
- [Jetpack Window Manager](https://developer.android.com/codelabs/android-window-manager-dual-screen-foldables)
- [Material 3 Adaptive Design](https://developer.android.com/develop/ui/compose/layouts/adaptive)

### iOS Responsive Design
- [SwiftUI Responsive Design: Adaptive Layouts 2025](https://ravi6997.medium.com/responsive-design-in-swiftui-stop-hardcoding-layout-for-iphone-only-build-apps-that-truly-adapt-69e5c975e112)
- [Mastering GeometryReader in SwiftUI](https://medium.com/@qmshahzad/mastering-geometryreader-in-swiftui-from-basics-to-advanced-layout-control-7b67b889848a)
- [Using GeometryReader for Dynamic Layouts](https://medium.com/@dhavaljasoliya8/using-geometryreader-for-dynamic-and-adaptive-ui-layouts-in-swiftui-8568351e902d)
- [7 Essential Techniques for Adaptive Layouts in SwiftUI](https://namitgupta.com/7-essential-techniques-for-crafting-adaptive-layouts-in-swiftui)
- [Dynamic Type Guide](https://developer.apple.com/documentation/uikit/scaling-fonts-automatically)

### 2025-2026 Best Practices
- [9 Mobile App Design Trends for 2026](https://uxpilot.ai/blogs/mobile-app-design-trends)
- [16 Key Mobile App UI/UX Design Trends (2026)](https://spdload.com/blog/mobile-app-ui-ux-design-trends/)
- [Responsive Design: Best Practices (2025)](https://www.uxpin.com/studio/blog/best-practices-examples-of-excellent-responsive-design/)
- [Responsive Web Design in 2026: Trends and Best Practices](https://www.keelis.com/blog/responsive-web-design-in-2026:-trends-and-best-practices)
- [9 Responsive Design Best Practices for 2025](https://nextnative.dev/blog/responsive-design-best-practices)
- [Breakpoint: Responsive Design Breakpoints in 2025](https://www.browserstack.com/guide/responsive-design-breakpoints)

---

## Quick Reference: Breakpoint Values

| Screen Type | Width Range | Android SizeClass | iOS Trait |
|-------------|-------------|-------------------|-----------|
| Compact Phone | < 600dp | Compact | Compact width |
| Large Phone | 600-839dp | Medium | Compact/Regular (orientation) |
| Tablet | 840-1199dp | Expanded | Regular width |
| Large Tablet | 1200-1599dp | Large | Regular width |
| Extra-Large | >= 1600dp | ExtraLarge | Regular width |

## Safe Area Values (Device Specifics)

| Device | Status Bar | Notch/Island | Bottom |
|--------|-----------|-------------|--------|
| iPhone 12-14 | 44pt | Dynamic Island | 34pt (Home indicator) |
| iPhone X/XS/11 | 44pt | Notch (39x375) | 34pt |
| Android (standard) | 24-48dp | None (bezel) | 0-48dp (gesture pill) |
| Galaxy Fold (main) | 48dp | None | 48dp |
| Pixel Fold | 24dp | None | None |

---

**Last Updated**: February 2025
**Framework Versions**:
- Android: Jetpack Compose 1.6+, Material 3 Adaptive
- iOS: SwiftUI iOS 17+, Dynamic Type (all versions)

---
name: audit-adaptive-layouts
description: Comprehensive audit of adaptive layout implementation for foldables, tablets, and responsive design using Material 3 canonical layouts
args:
  - name: code
    type: string
    description: Kotlin/Compose code to audit for adaptive layout patterns
    required: true
  - name: focus
    type: string
    enum: [window-sizing, canonical-layouts, foldables, navigation, breakpoints, all]
    description: Specific area to focus on (default: all)
    required: false
  - name: target-device
    type: string
    enum: [phone, tablet, foldable, auto]
    description: Target device category (default: auto-detect from code)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Adaptive Layouts Audit for Jetpack Compose

Comprehensive audit of Jetpack Compose code against Material Design 3 adaptive layout patterns, including WindowSizeClass implementation, canonical layouts (list-detail, feed, supporting pane), foldable device support, and responsive navigation.

**2025 Standards**: Material 3 Adaptive 1.2.0 stable, AndroidX.window 1.5.0 with Large/Extra-Large breakpoints, NavigableListDetailPaneScaffold, SupportingPaneScaffold, adaptive strategies (reflow, levitate).

## Instructions

Analyze the provided Compose code for compliance with 2025 Material Design 3 adaptive layout guidelines. Cover:
1. WindowSizeClass usage (Compact, Medium, Expanded, Large, Extra-Large)
2. Canonical layout patterns (list-detail, feed, supporting pane)
3. Foldable device support (hinge detection, posture states)
4. NavigationSuiteScaffold for automatic nav adaptation
5. Responsive grid systems and container queries
6. Tablet-specific layouts
7. Common mistakes and anti-patterns

Reference the `android-design` skill for Material 3 adaptive design guidelines.

## Audit Checklist

### WindowSizeClass Implementation (Critical Priority)

{{#if (or (eq focus "window-sizing") (eq focus "all") (not focus))}}

**WindowSizeClass Setup**

- [ ] **AL-001**: Importing `androidx.compose.material3.adaptive.WindowSizeClass`
- [ ] **AL-002**: Using `currentWindowAdaptiveInfo()` to compute window size class
- [ ] **AL-003**: Accessing `windowSizeClass.windowWidthSizeClass` and `windowHeightSizeClass`
- [ ] **AL-004**: WindowWidthSizeClass values: Compact (<600dp), Medium (600-839dp), Expanded (>=840dp)
- [ ] **AL-005**: WindowHeightSizeClass values: Compact (<480dp), Medium (480-899dp), Expanded (>=900dp)
- [ ] **AL-006**: Supporting Large (1200-1599dp) and Extra-Large (>=1600dp) breakpoints (AndroidX.window 1.5+)
- [ ] **AL-007**: Size class passed as parameter through composable hierarchy (not recomputed)
- [ ] **AL-008**: WindowSizeClass used to make layout decisions, not device-specific logic

**Content-Driven Breakpoints**

- [ ] **AL-009**: Breakpoints driven by content layout, not fixed device sizes
- [ ] **AL-010**: Using `BoxWithConstraints` for dynamic width/height measurements
- [ ] **AL-011**: Avoiding hardcoded dp breakpoints (use size class instead)
- [ ] **AL-012**: Fluid transitions between breakpoints using `clamp()` for smooth scaling
- [ ] **AL-013**: Testing with custom screen sizes (not just standard device sizes)

{{/if}}

### Canonical Layouts (Critical Priority)

{{#if (or (eq focus "canonical-layouts") (eq focus "all") (not focus))}}

**List-Detail Layout**

- [ ] **AL-014**: Using `ListDetailPaneScaffold` or `NavigableListDetailPaneScaffold` for list-detail pattern
- [ ] **AL-015**: List pane on left, detail pane on right in Expanded size class
- [ ] **AL-016**: List and detail stacked (list on top, detail below) in Compact size class
- [ ] **AL-017**: Navigation between panes managed correctly (list selection updates detail)
- [ ] **AL-018**: List pane shows preview/selection indicator in detail pane
- [ ] **AL-019**: Detail pane dismissible/backable in Compact mode
- [ ] **AL-020**: Content remains scrollable within each pane
- [ ] **AL-021**: State preserved when transitioning between size classes

**Feed Layout (Grid)**

- [ ] **AL-022**: Using `LazyVerticalGrid` with adaptive column count based on size class
- [ ] **AL-023**: Single column in Compact (< 600dp)
- [ ] **AL-024**: Two columns in Medium (600-839dp)
- [ ] **AL-025**: Three or more columns in Expanded (840dp+)
- [ ] **AL-026**: Grid items maintain aspect ratio across all sizes
- [ ] **AL-027**: Using `GridCells.Adaptive(minSize = X.dp)` for fluid column count
- [ ] **AL-028**: Proper spacing and padding between grid items (8-16dp gutters)
- [ ] **AL-029**: Content width limited to prevent lines > 80 characters on large screens

**Supporting Pane Layout**

- [ ] **AL-030**: Using `SupportingPaneScaffold` for primary + secondary content
- [ ] **AL-031**: Primary pane takes 60-70% width, supporting pane takes 30-40%
- [ ] **AL-032**: Both panes visible side-by-side in Expanded size class
- [ ] **AL-033**: Supporting pane hidden/stacked below in Compact size class
- [ ] **AL-034**: Supporting pane content is meaningful only in relation to primary content
- [ ] **AL-035**: Easy navigation between primary and supporting panes

**Adaptation Strategies (1.2.0+)**

- [ ] **AL-036**: Using Reflow strategy for list-detail (reorganizes panes on same level)
- [ ] **AL-037**: Using Levitate strategy when needed (elevates pane to dialog/bottom sheet)
- [ ] **AL-038**: Layout strategy chosen based on available space and content type
- [ ] **AL-039**: Smooth transitions when adaptation strategy changes

{{/if}}

### Foldable Device Support (Critical Priority)

{{#if (or (eq focus "foldables") (eq focus "all") (not focus))}}

**Fold State Detection**

- [ ] **AL-040**: Using `WindowInfo.displayFeatures` from `androidx.window.layout`
- [ ] **AL-041**: Detecting `FoldingFeature` for hinge location and orientation
- [ ] **AL-042**: Getting hinge bounds: `foldingFeature.bounds`
- [ ] **AL-043**: Checking hinge orientation: `VERTICAL` or `HORIZONTAL`
- [ ] **AL-044**: Getting device posture: `foldingFeature.isSeparating`
- [ ] **AL-045**: Supporting posture states: FLAT, HALF_OPENED (book), HALF_OPENED (tabletop)

**Safe Zone Layout**

- [ ] **AL-046**: Content positioned in safe zones, avoiding hinge line
- [ ] **AL-047**: Hinge width accommodated (typically 30-50dp)
- [ ] **AL-048**: No UI elements overlapping the fold/hinge
- [ ] **AL-049**: Using padding or spacing to create buffer zone around hinge
- [ ] **AL-050**: Text, buttons, interactive elements never placed on hinge line

**Dual-Pane Layouts for Foldables**

- [ ] **AL-051**: Vertical fold (book posture) - master detail split (left/right)
- [ ] **AL-052**: Horizontal fold (tabletop posture) - content on top, controls on bottom
- [ ] **AL-053**: FLAT posture - single pane layout (phone mode)
- [ ] **AL-054**: HALF_OPENED (book) - content visible on both sides of fold
- [ ] **AL-055**: HALF_OPENED (tabletop) - content visible on both sides of fold

**Foldable Testing & Samsung/Google Specifics**

- [ ] **AL-056**: State preserved when transitioning from cover screen to main screen
- [ ] **AL-057**: Content orientation correct when device is partially folded
- [ ] **AL-058**: UI elements resize properly as fold angle changes
- [ ] **AL-059**: Bottom sheets/dialogs positioned to avoid hinge
- [ ] **AL-060**: Testing on Galaxy Fold or Pixel Fold emulator images
- [ ] **AL-061**: No layout flicker when fold state changes

{{/if}}

### Navigation Adaptation (Important Priority)

{{#if (or (eq focus "navigation") (eq focus "all") (not focus))}}

**NavigationSuiteScaffold Implementation**

- [ ] **AL-062**: Using `NavigationSuiteScaffold` for automatic navigation mode switching
- [ ] **AL-063**: Bottom navigation bar for Compact size class
- [ ] **AL-064**: Navigation rail for Medium/Expanded size classes
- [ ] **AL-065**: Navigation drawer option for Expanded screens when appropriate
- [ ] **AL-066**: Consistent state preservation when navigation mode changes
- [ ] **AL-067**: Tab order same across navigation modes (same items, different layout)
- [ ] **AL-068**: Selected item highlighted in all navigation modes
- [ ] **AL-069**: No overlapping navigation elements in any size class

**Navigation Behavior Across Size Classes**

- [ ] **AL-070**: Back button hidden or removed on tablets with permanent navigation
- [ ] **AL-071**: Navigation hierarchy flattened on large screens (fewer taps)
- [ ] **AL-072**: Primary actions accessible in all screen sizes
- [ ] **AL-073**: Deep linking works across all navigation modes
- [ ] **AL-074**: Navigation state persists during device rotation

**Destination-Specific Navigation**

- [ ] **AL-075**: List screen shows list in Compact, list-detail in Expanded
- [ ] **AL-076**: Back navigation disappears in list pane (uses list selection instead)
- [ ] **AL-077**: Detail pane content updated without full recomposition
- [ ] **AL-078**: No dead-end screens on tablets (always have next action visible)

{{/if}}

### Breakpoint Strategy & Responsive Grids (Important Priority)

{{#if (or (eq focus "breakpoints") (eq focus "all") (not focus))}}

**Breakpoint Design**

- [ ] **AL-079**: Explicit handling of all 5 size classes: Compact, Medium, Expanded, Large, Extra-Large
- [ ] **AL-080**: Different layouts for each size class (not just scaling)
- [ ] **AL-081**: Layouts tested at each breakpoint boundary (599dp, 600dp, 839dp, 840dp, 1199dp, 1200dp, 1599dp, 1600dp)
- [ ] **AL-082**: No layout shift or jank when crossing breakpoints
- [ ] **AL-083**: Padding/margin increases appropriately for larger screens (M3 spacing grid)
- [ ] **AL-084**: Maximum content width enforced on extra-large screens (prevent line length > 80 chars)

**Responsive Grid Systems**

- [ ] **AL-085**: Using `LazyVerticalGrid` or `LazyHorizontalGrid` with adaptive sizing
- [ ] **AL-086**: Column count formula: `(availableWidth / itemWidth).toInt().coerceAtLeast(1)`
- [ ] **AL-087**: Grid items use fixed or minimum width, not max-width
- [ ] **AL-088**: Gaps between items scale appropriately (8-16dp)
- [ ] **AL-089**: Edge padding varies by size class (8dp compact, 16dp medium, 24dp expanded)

**Fluid Typography & Spacing**

- [ ] **AL-090**: Font sizes clamp between min/max: `clamp(minSize.sp, preferredSize.sp, maxSize.sp)`
- [ ] **AL-091**: Using `sp` (scale-independent pixels) for text sizes
- [ ] **AL-092**: Line height scales proportionally with text size
- [ ] **AL-093**: Spacing and padding use `clamp()` for smooth scaling
- [ ] **AL-094**: No hardcoded padding/margin - all relative to size class

{{/if}}

### Tablet & Large Screen Layouts (Important Priority)

{{#if (or (eq focus "all") (not focus))}}

**Tablet-Specific Layouts (840dp+)**

- [ ] **AL-095**: Full screen real estate used (not mobile-centered design)
- [ ] **AL-096**: Master-detail pane layouts (sidebar + main content)
- [ ] **AL-097**: Dual-pane layouts showing list and detail simultaneously
- [ ] **AL-098**: Two-column grid layouts instead of single-column scrolling
- [ ] **AL-099**: Navigation rail permanently visible (not hamburger menu)
- [ ] **AL-100**: Larger touch targets (48-56dp minimum)

**Large Tablet Optimization (1200dp+)**

- [ ] **AL-101**: Three-pane layouts (navigation + list + detail)
- [ ] **AL-102**: Content width limited (max 1200dp for readability)
- [ ] **AL-103**: Increased whitespace/padding to reduce visual clutter
- [ ] **AL-104**: Font sizes readable on large screens (not too small)
- [ ] **AL-105**: No cramped or squeezed content in corners

**Landscape Orientation**

- [ ] **AL-106**: Supporting both portrait and landscape orientations
- [ ] **AL-107**: State preserved during orientation change (savedInstanceState)
- [ ] **AL-108**: Layout recomputes correctly without flicker
- [ ] **AL-109**: No content hidden or pushed off-screen in landscape
- [ ] **AL-110**: Landscape uses available width for side-by-side layouts

{{/if}}

### Container Queries & Intrinsic Sizing (Important Priority)

{{#if (or (eq focus "all") (not focus))}}

- [ ] **AL-111**: Components size based on container width, not viewport
- [ ] **AL-112**: Using `BoxWithConstraints` for measuring available space
- [ ] **AL-113**: Subcomposable behavior driven by local constraints
- [ ] **AL-114**: No global state driving layout (only local measurements)
- [ ] **AL-115**: Child components reuse in different containers without modification

{{/if}}

### Common Mistakes & Anti-Patterns (Critical Priority)

{{#if (or (eq focus "all") (not focus))}}

**DO NOT:**

- [ ] **AL-116**: No hardcoded screen sizes (e.g., `if (width > 600)` without size class)
- [ ] **AL-117**: No stretched phone layouts on tablets (no 400dp columns on 1200dp screens)
- [ ] **AL-118**: No ignoring fold state on foldable devices
- [ ] **AL-119**: No UI elements overlapping the hinge line
- [ ] **AL-120**: No fixed aspect ratios that break on tablets
- [ ] **AL-121**: No bottom sheets that extend beyond available height on foldables
- [ ] **AL-122**: No assuming landscape orientation support (Android ignores manifest on tablets)
- [ ] **AL-123**: No single-pane layouts on devices with 840dp+ width
- [ ] **AL-124**: No navigation modes switching without state preservation
- [ ] **AL-125**: No text truncation due to width constraints on tablets

**DO:**

- [ ] **AL-126**: Pass `WindowSizeClass` through composable hierarchy (not recompute)
- [ ] **AL-127**: Use size class consistently (don't mix size class and hardcoded breakpoints)
- [ ] **AL-128**: Test on actual device emulators (Pixel 2, Pixel 6, Pixel Tablet, Galaxy Fold)
- [ ] **AL-129**: Preserve scroll position when panes change
- [ ] **AL-130**: Use canonical layouts (list-detail, feed, supporting pane) where appropriate
- [ ] **AL-131**: Handle fold state gracefully (no crashes when fold detected)
- [ ] **AL-132**: Animate pane transitions smoothly (no instant layout changes)

{{/if}}

### Testing & Device Coverage (Important Priority)

{{#if (or (eq focus "all") (not focus))}}

- [ ] **AL-133**: Tested on Compact phones (< 600dp) - portrait & landscape
- [ ] **AL-134**: Tested on Medium phones (600-839dp) - portrait & landscape
- [ ] **AL-135**: Tested on Expanded tablets (840-1199dp) - portrait & landscape
- [ ] **AL-136**: Tested on Large tablets (1200-1599dp) - portrait & landscape
- [ ] **AL-137**: Tested on Extra-Large displays (>= 1600dp)
- [ ] **AL-138**: Tested on foldable devices (Galaxy Fold, Pixel Fold) - FLAT, HALF_OPENED, FULLY_OPENED
- [ ] **AL-139**: Tested in split-screen/multi-window mode (50/50, 30/70 splits)
- [ ] **AL-140**: Tested with dynamic font scaling enabled (100%, 125%, 150%, 200%)
- [ ] **AL-141**: Tested with accessibility features enabled (high contrast, large text)
- [ ] **AL-142**: Layout checklist completed for all form factors

{{/if}}

## Code to Audit

```kotlin
{{code}}
```

## Output Format

Generate a structured adaptive layout audit report:

```markdown
## Adaptive Layout Audit Results: [Component/Screen Name]

### Critical Issues (🔴)

Issues that break layout or make content inaccessible on tablets/foldables.
Each with: Line number, affected device type(s), description, severity, fix suggestion.

### Important Issues (🟠)

Issues that violate Material 3 adaptive guidelines or degrade UX on large screens.

### Warnings (🟡)

Layout patterns that work but should be improved for better coverage.

### Suggestions (🔵)

Polish opportunities for comprehensive adaptive design.

### Device Coverage Summary

- Compact phones (< 600dp): [Fully supported / Partial / Not tested]
- Medium phones (600-839dp): [Fully supported / Partial / Not tested]
- Expanded tablets (840-1199dp): [Fully supported / Partial / Not tested]
- Large tablets (1200-1599dp): [Fully supported / Partial / Not tested]
- Extra-large displays (> 1600dp): [Fully supported / Partial / Not tested]
- Foldables (FLAT/HALF_OPENED/FULLY_OPENED): [Fully supported / Partial / Not applicable]
- Split-screen/multi-window: [Fully supported / Partial / Not applicable]

### WindowSizeClass Implementation Status

- [currentWindowAdaptiveInfo used / Not used]
- Breakpoints covered: [Compact / Medium / Expanded / Large / Extra-Large]
- Size class properly threaded: [Yes / No / Partial]

### Canonical Layouts Used

- List-Detail: [Used / Not applicable / Not implemented]
- Feed/Grid: [Used / Not applicable / Not implemented]
- Supporting Pane: [Used / Not applicable / Not implemented]

### Foldable Support Status

- Fold detection implemented: [Yes / No / Not applicable]
- Safe zones enforced: [Yes / No / Not applicable]
- Hinge handling: [Complete / Partial / Missing]

### Recommended Actions (Priority Order)

1. [Most urgent fix for critical device type]
2. [Second priority]
3. [Third priority]

### Testing Checklist for Implementation

- [ ] Test on Pixel 4 (compact phone - portrait & landscape)
- [ ] Test on Pixel 6 (medium phone - landscape)
- [ ] Test on Pixel Tablet (expanded tablet - portrait & landscape)
- [ ] Test on iPad Pro size (large tablet - landscape)
- [ ] Test on Galaxy Fold or Pixel Fold (foldable - all postures)
- [ ] Test split-screen mode (50/50 split)
- [ ] Test with font scaling at 100%, 150%, 200%
- [ ] Verify no layout jank at breakpoint transitions
- [ ] Verify touch targets (min 48x48dp) on all sizes
- [ ] Verify text contrast maintained at all sizes
- [ ] Performance check (frame rate stable during transitions)
```

## Example Output

```markdown
## Adaptive Layout Audit Results: EventListScreen.kt

### Critical Issues (🔴)

**Line 24**: No WindowSizeClass usage - hardcoded phone layout
```kotlin
// BAD - Always single column regardless of device
Column(modifier = Modifier.width(400.dp)) {
    LazyColumn {
        items(events) { event ->
            EventCard(event = event)
        }
    }
}

// GOOD - Responsive to screen size
val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass
when (windowSizeClass.windowWidthSizeClass) {
    WindowWidthSizeClass.Compact -> {
        // Single column layout
        LazyColumn {
            items(events) { event ->
                EventCard(event = event)
            }
        }
    }
    WindowWidthSizeClass.Medium -> {
        // Two column grid
        LazyVerticalGrid(columns = GridCells.Fixed(2)) {
            items(events) { event ->
                EventCard(event = event)
            }
        }
    }
    else -> {
        // List-detail layout for expanded
        ListDetailPaneScaffold(
            directive = PaneScaffoldDirective.VerticalSplit,
            listPane = {
                LazyColumn { /* list */ }
            },
            detailPane = {
                EventDetailPane(selectedEvent)
            }
        )
    }
}
```
Severity: 🔴 Critical | Confidence: 95 | Rule: AL-002
Affected devices: Tablets (840dp+), foldables in expanded mode, landscape phones
---

**Line 67**: Bottom navigation bar always shown - should use NavigationSuiteScaffold
```kotlin
// BAD - Always bottom nav regardless of screen size
Column {
    EventListContent()
    BottomNavigation { /* tabs */ }
}

// GOOD - Automatic navigation adaptation
NavigationSuiteScaffold(
    navigationSuiteItems = {
        navigationItem(
            icon = Icons.Default.Home,
            label = "Explore",
            selected = true,
            onClick = { /* navigate */ }
        )
        // more items...
    }
) {
    EventListContent()
}
```
Severity: 🔴 Critical | Confidence: 90 | Rule: AL-062
Affected devices: Tablets (should use navigation rail instead)
---

### Important Issues (🟠)

**Line 45**: No foldable device handling
```kotlin
// BAD - Ignores hinge location
LazyVerticalGrid(columns = GridCells.Fixed(2)) {
    items(events) { event ->
        EventCard(event = event)
    }
}

// GOOD - Avoid hinge in dual-pane layout
val displayFeatures = calculateDisplayFeatures(LocalContext.current)
val foldingFeature = displayFeatures.find { it is FoldingFeature }

if (foldingFeature != null && foldingFeature.isSeparating) {
    // Dual-pane layout with safe zones
    Row {
        Box(
            modifier = Modifier
                .width(foldingFeature.bounds.left.dp)
                .fillMaxHeight()
        ) {
            LazyColumn { /* left pane */ }
        }
        // Gap for hinge
        Box(modifier = Modifier.width(foldingFeature.bounds.width().dp))
        Box(
            modifier = Modifier
                .weight(1f)
                .fillMaxHeight()
        ) {
            LazyColumn { /* right pane */ }
        }
    }
} else {
    LazyVerticalGrid(columns = GridCells.Fixed(2)) { /* normal layout */ }
}
```
Severity: 🟠 Important | Confidence: 85 | Rule: AL-040
Affected devices: Foldables in half-opened state
---

**Line 89**: Hardcoded padding breaks tablet layouts
```kotlin
// BAD - Same padding on all screens
EventCard(modifier = Modifier.padding(8.dp))

// GOOD - Responsive padding
val padding = when (windowSizeClass.windowWidthSizeClass) {
    WindowWidthSizeClass.Compact -> 8.dp
    WindowWidthSizeClass.Medium -> 12.dp
    else -> 16.dp
}
EventCard(modifier = Modifier.padding(padding))
```
Severity: 🟠 Important | Confidence: 80 | Rule: AL-083
Affected devices: Large tablets (looks cramped)
---

### Warnings (🟡)

**Line 120**: No text scaling support
```kotlin
// BAD - Fixed font size
Text(
    text = event.name,
    fontSize = 16.sp,
    modifier = Modifier.width(400.dp)
)

// GOOD - Uses theme typography (scales with accessibility)
Text(
    text = event.name,
    style = MaterialTheme.typography.titleMedium,
    maxLines = 2
)
```
Severity: 🟡 Warning | Confidence: 75 | Rule: RES-033
Affected devices: All devices with font scaling enabled
---

### Device Coverage Summary

- Compact phones (< 600dp): ✓ Fully supported
- Medium phones (600-839dp): ⚠️ Partial (no landscape test)
- Expanded tablets (840-1199dp): ❌ Not tested (hardcoded 400dp width)
- Large tablets (1200-1599dp): ❌ Broken
- Extra-large displays (> 1600dp): ❌ Broken
- Foldables (FLAT/HALF_OPENED/FULLY_OPENED): ❌ Not implemented
- Split-screen/multi-window: ⚠️ Untested

### WindowSizeClass Implementation Status

- currentWindowAdaptiveInfo used: ❌ No
- Breakpoints covered: None
- Size class properly threaded: ❌ No

### Canonical Layouts Used

- List-Detail: ❌ Not implemented
- Feed/Grid: ⚠️ Partial (fixed column count)
- Supporting Pane: ❌ Not applicable

### Foldable Support Status

- Fold detection implemented: ❌ No
- Safe zones enforced: ❌ No
- Hinge handling: ❌ Missing

### Recommended Actions

1. Add `currentWindowAdaptiveInfo()` and compute WindowSizeClass at top level
2. Implement NavigationSuiteScaffold to handle navigation mode switching
3. Replace hardcoded 400dp width with size-class-aware layouts
4. Add foldable support using `displayFeatures` and FoldingFeature detection
5. Test full layout on tablet/foldable emulators before production
6. Implement list-detail canonical layout for Expanded size class
7. Add safe zone handling for hinge (30-50dp buffer)

### Testing Checklist for Implementation

- [ ] Build and test on Pixel 4 (compact, portrait)
- [ ] Build and test on Pixel 6 (medium, portrait & landscape)
- [ ] Build and test on Pixel Tablet (expanded, portrait & landscape)
- [ ] Build and test on Galaxy Fold (foldable, FLAT and HALF_OPENED states)
- [ ] Verify layout smooth at 600dp, 840dp, 1200dp breakpoints
- [ ] Verify no layout jank when crossing breakpoints
- [ ] Verify touch targets remain 48x48dp minimum
- [ ] Test split-screen with another app (50/50 split)
- [ ] Check frame rate during navigation transitions
- [ ] Verify navigation mode switches (bar → rail) without state loss
```

## Code Examples - Canonical Layouts

### List-Detail Layout (Recommended)

```kotlin
@Composable
fun EventListDetailScreen() {
    val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass
    val selectedEventId = remember { mutableStateOf<String?>(null) }

    ListDetailPaneScaffold(
        directive = when (windowSizeClass.windowWidthSizeClass) {
            WindowWidthSizeClass.Compact -> PaneScaffoldDirective.SinglePane
            else -> PaneScaffoldDirective.VerticalSplit
        },
        listPane = {
            LazyColumn {
                items(events) { event ->
                    EventItem(
                        event = event,
                        isSelected = selectedEventId.value == event.id,
                        onClick = { selectedEventId.value = event.id }
                    )
                }
            }
        },
        detailPane = {
            selectedEventId.value?.let { id ->
                events.find { it.id == id }?.let { event ->
                    EventDetailPane(event)
                }
            }
        }
    )
}
```

### Feed Layout (Adaptive Grid)

```kotlin
@Composable
fun EventFeedScreen() {
    val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass

    val columnCount = when (windowSizeClass.windowWidthSizeClass) {
        WindowWidthSizeClass.Compact -> 1
        WindowWidthSizeClass.Medium -> 2
        else -> 3
    }

    LazyVerticalGrid(
        columns = GridCells.Fixed(columnCount),
        contentPadding = PaddingValues(
            horizontal = when (windowSizeClass.windowWidthSizeClass) {
                WindowWidthSizeClass.Compact -> 8.dp
                WindowWidthSizeClass.Medium -> 12.dp
                else -> 16.dp
            },
            vertical = 8.dp
        ),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        verticalArrangement = Arrangement.spacedBy(8.dp)
    ) {
        items(events) { event ->
            EventCard(event = event)
        }
    }
}
```

### Supporting Pane Layout

```kotlin
@Composable
fun EventDetailWithSupportingPane() {
    val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass

    SupportingPaneScaffold(
        directive = when (windowSizeClass.windowWidthSizeClass) {
            WindowWidthSizeClass.Compact -> PaneScaffoldDirective.SinglePane
            else -> PaneScaffoldDirective.VerticalSplit
        },
        mainPane = {
            EventDetailContent(event)
        },
        supportingPane = {
            SupportingInformationPane(event)
        }
    )
}
```

### Foldable Support with Hinge Detection

```kotlin
@Composable
fun FoldableAwareLayout(content: @Composable () -> Unit) {
    val displayFeatures = calculateDisplayFeatures(LocalContext.current)
    val foldingFeature = displayFeatures.find { it is FoldingFeature } as? FoldingFeature

    if (foldingFeature != null && foldingFeature.isSeparating) {
        val hingeWidth = foldingFeature.bounds.right - foldingFeature.bounds.left

        Row(modifier = Modifier.fillMaxSize()) {
            // Left safe zone
            Box(
                modifier = Modifier
                    .width(foldingFeature.bounds.left.dp)
                    .fillMaxHeight()
            ) {
                content()
            }

            // Hinge spacer
            Box(modifier = Modifier.width(hingeWidth.dp))

            // Right safe zone
            Box(
                modifier = Modifier
                    .weight(1f)
                    .fillMaxHeight()
            ) {
                content()
            }
        }
    } else {
        content()
    }
}
```

### NavigationSuiteScaffold Setup

```kotlin
@Composable
fun AdaptiveNavigation() {
    val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass
    val navController = rememberNavController()
    val currentBackStackEntry by navController.currentBackStackEntryAsState()
    val currentDestination = currentBackStackEntry?.destination

    NavigationSuiteScaffold(
        navigationSuiteItems = {
            navigationItem(
                selected = currentDestination?.route == "explore",
                onClick = { navController.navigate("explore") },
                icon = { Icon(Icons.Default.Home, contentDescription = "Explore") },
                label = { Text("Explore") }
            )
            navigationItem(
                selected = currentDestination?.route == "games",
                onClick = { navController.navigate("games") },
                icon = { Icon(Icons.Default.Games, contentDescription = "My Games") },
                label = { Text("My Games") }
            )
            navigationItem(
                selected = currentDestination?.route == "profile",
                onClick = { navController.navigate("profile") },
                icon = { Icon(Icons.Default.Person, contentDescription = "Profile") },
                label = { Text("Profile") }
            )
        }
    ) {
        NavHost(navController = navController, startDestination = "explore") {
            composable("explore") { ExploreScreen() }
            composable("games") { MyGamesScreen() }
            composable("profile") { ProfileScreen() }
        }
    }
}
```

## Reference Resources

### Official Material Design 3 Adaptive (2025)

- [Material 3 Adaptive Layouts](https://developer.android.com/develop/ui/compose/layouts/adaptive)
- [Use window size classes](https://developer.android.com/develop/ui/compose/layouts/adaptive/use-window-size-classes)
- [Canonical Layouts](https://developer.android.com/develop/ui/compose/layouts/adaptive/canonical-layouts)
- [Support different display sizes](https://developer.android.com/develop/ui/compose/layouts/adaptive/support-different-display-sizes)
- [Build a supporting pane layout](https://developer.android.com/develop/ui/compose/layouts/adaptive/build-a-supporting-pane-layout)
- [Material 3 Adaptive 1.2.0 is stable](https://android-developers.googleblog.com/2025/10/material-3-adaptive-120-is-stable.html)

### Foldable & Multi-Screen

- [Learn about foldables](https://developer.android.com/guide/topics/large-screens/learn-about-foldables)
- [Jetpack WindowManager](https://developer.android.com/develop/ui/compose/layouts/adaptive/foldable-support)
- [Adaptive Design (Part 1): Make any Compose Screen Responsive in 4 Steps](https://tanishranjan.medium.com/responsive-design-part-1-make-any-compose-screen-responsive-in-4-steps-3fe6f3191d3d)
- [Mastering Adaptive UIs in Jetpack Compose: NavigableListDetailPaneScaffold](https://www.droidcon.com/2025/06/16/mastering-adaptive-uis-in-jetpack-compose-a-dive-into-navigablelistdetailpanescaffold/)

### 2025 Updates & Best Practices

- [WindowSizeClass Compose 2025 - Android Developers Blog](https://android-developers.googleblog.com/2025/09/unfold-new-possibilities-with-compose-adaptive-layouts-1-2-beta.html)
- [Canonical Layouts - Material Design 3](https://m3.material.io/foundations/layout/canonical-layouts/list-detail)
- [Jetpack Compose Adaptive Layout Codelab](https://codelabs.developers.google.com/jetpack-compose-adaptability)
- [Google I/O 2025: Build adaptive Android apps that shine across form factors](https://android-developers.googleblog.com/2025/05/adaptiveapps-io25.html)

### Samsung & Device-Specific

- [Designing for Foldables | Samsung Developer](https://developer.samsung.com/one-ui/largescreen-and-foldable/designing_for_foldable.html)
- [Material 3 Adaptive Design](https://m3.material.io/foundations/adaptive-design)

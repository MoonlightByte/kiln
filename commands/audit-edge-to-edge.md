---
name: audit-edge-to-edge
description: Audit Android code for Android 15+ edge-to-edge enforcement compliance
args:
  - name: code
    type: string
    description: Kotlin/Compose code to audit
    required: true
  - name: focus
    type: string
    enum: [activity, insets, modifiers, safe-areas, gestures, ime, system-bars, all]
    description: Specific area to focus on (default: all)
    required: false
  - name: target_sdk
    type: number
    description: Target SDK level (default: 35 for Android 15)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Android Edge-to-Edge Audit (Android 15+)

Comprehensive audit of Kotlin/Compose code for Android 15 (API 35+) edge-to-edge display enforcement.

## Background: Android 15 Edge-to-Edge Enforcement

**Mandatory for SDK 35+**: Apps targeting Android 15 or higher must implement edge-to-edge display. Content draws behind system bars (status bar, navigation bar, gesture navigation, display cutouts).

**Key Changes:**
- `enableEdgeToEdge()` call in Activity.onCreate() is required
- System bars are transparent by default (except 3-button nav which gets translucent scrim)
- Apps must handle `WindowInsets` to prevent content from being obscured
- `Modifier.windowInsetsPadding()` applies safe drawing insets automatically
- `Scaffold` in Compose automatically handles top/bottom insets via `innerPadding`

**Deprecated APIs (API 35+):**
- Setting status bar as translucent (no effect)
- Manual inset calculations (use WindowInsets instead)

**Opt-Out Temporary:**
- `windowOptOutEdgeToEdgeEnforcement = true` in theme (deprecated in future SDK)
- Not recommended - will be removed in Android 16+

## Instructions

Analyze the code for edge-to-edge compliance. Auto-detect platform based on syntax (`@Composable` = Android, `some View` = iOS).

## Audit Checklist

### Activity Setup (Critical Priority)
{{#if (or (eq focus "activity") (eq focus "all") (not focus))}}

#### enableEdgeToEdge() Call
- [ ] **E2E-001**: `enableEdgeToEdge()` called in Activity.onCreate()
  ```kotlin
  // GOOD - Jetpack Activity library
  import androidx.activity.enableEdgeToEdge

  class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
      super.onCreate(savedInstanceState)
      enableEdgeToEdge()  // Called BEFORE setContent()
      setContent { /* ... */ }
    }
  }
  ```
- [ ] **E2E-002**: `enableEdgeToEdge()` called BEFORE `setContent()`
- [ ] **E2E-003**: All Activities with Compose content call `enableEdgeToEdge()`
- [ ] **E2E-004**: No `windowOptOutEdgeToEdgeEnforcement = true` (opt-out deprecated)

#### System Bar Configuration
- [ ] **E2E-005**: Status bar is transparent (default after enableEdgeToEdge)
- [ ] **E2E-006**: Navigation bar is transparent or translucent (app-dependent)
- [ ] **E2E-007**: No manual setStatusBarColor() calls conflicting with edge-to-edge
- [ ] **E2E-008**: Icon colors set appropriately for transparency (Light/Dark theme)

{{/if}}

### WindowInsets Handling (Critical Priority)
{{#if (or (eq focus "insets") (eq focus "all") (not focus))}}

#### Inset Types
- [ ] **E2E-009**: Using `WindowInsets.safeDrawing` for content that must avoid system UI
- [ ] **E2E-010**: Using `WindowInsets.safeGestures` for gesture-sensitive content
- [ ] **E2E-011**: Using `WindowInsets.displayCutout` for display cutout handling
- [ ] **E2E-012**: Using `WindowInsets.systemBars` for full system bar height
- [ ] **E2E-013**: Using `WindowInsets.ime` for keyboard (IME) inset handling

#### WindowInsets Not Manually Calculated
- [ ] **E2E-014**: NOT manually calculating insets with `getSystemWindowInsetTop()`
- [ ] **E2E-015**: NOT using deprecated `View.getPaddingTop()` for system bar spacing
- [ ] **E2E-016**: NOT hardcoding status bar height (24dp) or nav bar height (48-72dp)

#### Common WindowInsets Patterns
```kotlin
// GOOD - Use WindowInsets.safeDrawing for content
val insets = WindowInsets.safeDrawing
modifier = Modifier.windowInsetsPadding(insets)

// GOOD - Specific insets
val systemBarInsets = WindowInsets.systemBars
val imeInsets = WindowInsets.ime
val cutoutInsets = WindowInsets.displayCutout

// BAD - Manual calculation (deprecated)
val statusBarHeight = 24.dp  // NO!
modifier = Modifier.padding(top = statusBarHeight)
```

{{/if}}

### Modifier.windowInsetsPadding Usage (Critical Priority)
{{#if (or (eq focus "modifiers") (eq focus "all") (not focus))}}

#### Proper Inset Application
- [ ] **E2E-017**: `Modifier.windowInsetsPadding(WindowInsets.safeDrawing)` on top-level content
- [ ] **E2E-018**: NOT applying insets to nested children (apply once at top level)
- [ ] **E2E-019**: Insets applied to all edge-sensitive layouts (TopAppBar, BottomAppBar)
- [ ] **E2E-020**: Using `Modifier.systemBarsPadding()` as shorthand for common case

#### Scaffold Automatic Handling
- [ ] **E2E-021**: Using `Scaffold` for automatic inset handling via `innerPadding` parameter
- [ ] **E2E-022**: TopAppBar inside Scaffold (automatically handles status bar)
- [ ] **E2E-023**: BottomAppBar inside Scaffold (automatically handles nav bar)
- [ ] **E2E-024**: Passing `innerPadding` to content LazyColumn/LazyRow correctly

```kotlin
// GOOD - Scaffold handles insets automatically
Scaffold(
  topBar = { TopAppBar(title = { Text("Title") }) },
  bottomBar = { BottomAppBar { /* nav */ } }
) { innerPadding ->
  LazyColumn(modifier = Modifier.padding(innerPadding)) {
    items(count) { /* content */ }
  }
}

// BAD - Scaffold not used, manual inset handling needed
Column {
  TopAppBar(title = { Text("Title") })  // Overlaps status bar!
  LazyColumn { items(count) { /* content */ } }
}
```

{{/if}}

### Safe Area Handling (Important Priority)
{{#if (or (eq focus "safe-areas") (eq focus "all") (not focus))}}

#### Display Cutouts (Notches, Punch-Holes, Under-Display Cameras)
- [ ] **E2E-025**: Content avoids display cutout area
- [ ] **E2E-026**: Critical UI (buttons, text) NOT placed in cutout area
- [ ] **E2E-027**: Using `WindowInsets.displayCutout` to query cutout bounds
- [ ] **E2E-028**: Handling `LayoutInsets.LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT`

#### Corner Radius Devices
- [ ] **E2E-029**: Content accounts for rounded corners on modern devices
- [ ] **E2E-030**: NO decorative content extends into corner area
- [ ] **E2E-031**: Icons/images NOT placed at exact screen corners

#### Display Edges
- [ ] **E2E-032**: Content properly padded from left/right edges
- [ ] **E2E-033**: NO full-width gradients extending to raw edge (looks wrong)
- [ ] **E2E-034**: Symmetrical padding if design shows it (don't let asymmetry look wrong)

{{/if}}

### Gesture Navigation (Gesture Bar) Handling (Important Priority)
{{#if (or (eq focus "gestures") (eq focus "all") (not focus))}}

#### Gesture Navigation Bar
- [ ] **E2E-035**: Bottom navigation buttons clear of gesture navigation bar area
- [ ] **E2E-036**: Using `WindowInsets.systemBars` to query nav bar height
- [ ] **E2E-037**: Gesture navigation bar is part of `safeDrawing` inset calculation
- [ ] **E2E-038**: FAB (Floating Action Button) positioned above nav bar

#### Swipe Gestures
- [ ] **E2E-039**: Custom swipe/drag handlers DO NOT conflict with system back gesture
- [ ] **E2E-040**: Swipe actions use `safeGestures` inset for safe zone
- [ ] **E2E-041**: Back gesture edge (16-24dp from left) is NOT blocked by content

```kotlin
// GOOD - FAB positioned above nav bar
FloatingActionButton(
  onClick = { /* ... */ },
  modifier = Modifier
    .align(Alignment.BottomEnd)
    .padding(16.dp)
    .windowInsetsPadding(WindowInsets.systemBars.only(WindowInsetsSides.Bottom))
)
```

{{/if}}

### IME (Keyboard) Handling (Important Priority)
{{#if (or (eq focus "ime") (eq focus "all") (not focus))}}

#### Keyboard Insets
- [ ] **E2E-042**: `WindowInsets.ime` used for keyboard height detection
- [ ] **E2E-043**: Text input fields NOT obscured by keyboard
- [ ] **E2E-044**: Content scrolls up when keyboard appears
- [ ] **E2E-045**: Using `Modifier.imePadding()` for keyboard-safe zones

#### Common IME Pattern
```kotlin
// GOOD - Content adjusts for keyboard
Column(
  modifier = Modifier.fillMaxSize()
    .imePadding()  // Automatically adds bottom padding when keyboard shows
) {
  LazyColumn(Modifier.weight(1f)) { items(list) { /* content */ } }
  OutlinedTextField(
    value = input,
    onValueChange = { input = it },
    modifier = Modifier.fillMaxWidth()
  )
}

// ALSO GOOD - Manual handling
val imeInsets = WindowInsets.ime
Column(
  modifier = Modifier
    .fillMaxSize()
    .windowInsetsPadding(imeInsets)
) { /* ... */ }
```

#### Soft Input Mode
- [ ] **E2E-046**: NO hardcoded `android:windowSoftInputMode="adjustNothing"`
- [ ] **E2E-047**: Using `adjustResize` (Compose default) or `adjustPan` appropriately
- [ ] **E2E-048**: Keyboards do NOT hide critical form fields

{{/if}}

### Status Bar & Navigation Bar Transparency (Important Priority)
{{#if (or (eq focus "system-bars") (eq focus "all") (not focus))}}

#### Status Bar
- [ ] **E2E-049**: Status bar background is transparent or app-colored
- [ ] **E2E-050**: Status bar icons are light or dark (respects system theme)
- [ ] **E2E-051**: TopAppBar extends behind status bar if design requires
- [ ] **E2E-052**: NO text/icons positioned exactly where status bar icons appear

#### Navigation Bar
- [ ] **E2E-053**: Navigation bar background transparent or app-colored
- [ ] **E2E-054**: Bottom sheet does NOT hide behind nav bar
- [ ] **E2E-055**: Gesture navigation buttons (when visible) not obscured
- [ ] **E2E-056**: Proper contrast for nav bar icons (Light/Dark theme)

#### Theme Integration
- [ ] **E2E-057**: Using Material 3 theme (handles bar colors automatically)
- [ ] **E2E-058**: `useDynamicColor` respects edge-to-edge transparency
- [ ] **E2E-059**: Light/Dark mode toggle does NOT break inset handling

{{/if}}

### Material 3 Components (Material 3 Recommended - Important)
{{#if (or (eq focus "all") (not focus))}}

#### Scaffold
- [ ] **E2E-060**: Using `androidx.compose.material3.Scaffold`
- [ ] **E2E-061**: TopAppBar placed in `topBar` parameter (not in content)
- [ ] **E2E-062**: BottomAppBar/NavigationBar in `bottomBar` parameter
- [ ] **E2E-063**: Content receives `innerPadding` parameter

#### TopAppBar
- [ ] **E2E-064**: Using Material 3 `TopAppBar` (not custom layout)
- [ ] **E2E-065**: Automatically handles status bar insets
- [ ] **E2E-066**: Can extend behind status bar if `colors.containerColor` supports it

#### BottomAppBar & NavigationBar
- [ ] **E2E-067**: Using Material 3 `NavigationBar` or `BottomAppBar`
- [ ] **E2E-068**: Automatically handles navigation bar insets
- [ ] **E2E-069**: Items properly padded inside bar

#### BottomSheet
- [ ] **E2E-070**: Using Material 3 `BottomSheetScaffold` or `ModalBottomSheet`
- [ ] **E2E-071**: Sheet respects safe drawing area
- [ ] **E2E-072**: Sheet does NOT extend behind nav bar incorrectly

{{/if}}

### Testing on Different Device Configurations (Important)
{{#if (or (eq focus "all") (not focus))}}

#### Device Types to Test
- [ ] **E2E-073**: Tested on device with notch (top center cutout)
- [ ] **E2E-074**: Tested on device with punch-hole (small circular cutout)
- [ ] **E2E-075**: Tested on device with under-display camera (full-width cutout)
- [ ] **E2E-076**: Tested on device with rounded corners
- [ ] **E2E-077**: Tested on gesture navigation (Android 9+)
- [ ] **E2E-078**: Tested on 3-button navigation (older Android)

#### Orientation Testing
- [ ] **E2E-079**: Landscape orientation with cutout (side notch)
- [ ] **E2E-080**: Rotated device does NOT cause layout breaks
- [ ] **E2E-081**: Insets recalculated correctly on rotation

#### Keyboard Testing
- [ ] **E2E-082**: Text fields visible and reachable with keyboard visible
- [ ] **E2E-083**: Keyboard does NOT hide form submit button
- [ ] **E2E-084**: Scrolling works correctly with keyboard open

{{/if}}

## Code to Audit

```kotlin
{{code}}
```

## Output Format

Generate a structured audit report:

```markdown
## Edge-to-Edge Audit: [Component/Activity Name]

### Android Version Compliance

- **Target SDK**: 35 (Android 15)
- **Status**: ✅ Compliant / ⚠️ Partial / ❌ Non-Compliant
- **Key API**: enableEdgeToEdge() required

### Critical Issues (🔴)

**E2E-001**: enableEdgeToEdge() call missing
```kotlin
// BAD
class MainActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent { /* content */ }  // NO enableEdgeToEdge()!
  }
}

// GOOD
class MainActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    enableEdgeToEdge()
    setContent { /* content */ }
  }
}
```
Severity: 🔴 Critical | Rule: E2E-001 | Enforcement: SDK 35+

---

**E2E-017**: Not using Modifier.windowInsetsPadding() for insets
```kotlin
// BAD - Content overlaps status bar
Column {
  TopAppBar(title = { Text("Title") })
  LazyColumn { items(count) { /* overlaps bar! */ } }
}

// GOOD - Scaffold handles insets
Scaffold(
  topBar = { TopAppBar(title = { Text("Title") }) }
) { innerPadding ->
  LazyColumn(modifier = Modifier.padding(innerPadding)) {
    items(count) { /* safe from bars */ }
  }
}
```
Severity: 🔴 Critical | Rule: E2E-017 | Impact: Content obscured by system bars

### Important Issues (🟠)

**E2E-042**: IME insets not handled
```kotlin
// BAD - Keyboard hides text field
Column {
  LazyColumn { items(messages) { /* ... */ } }
  OutlinedTextField(value = input, onValueChange = { input = it })
}

// GOOD - Content adjusts for keyboard
Column(modifier = Modifier.imePadding()) {
  LazyColumn(Modifier.weight(1f)) { items(messages) { /* ... */ } }
  OutlinedTextField(value = input, onValueChange = { input = it })
}
```
Severity: 🟠 Important | Rule: E2E-042 | Impact: Form fields hidden by keyboard

### Warnings (🟡)

**E2E-031**: Content positioned at screen corners
Severity: 🟡 Warning | Rule: E2E-031 | Impact: Looks bad on rounded-corner devices

### Device Configuration Coverage

| Device Type | Tested | Status |
|-------------|--------|--------|
| Notch (top center) | ✅ | PASS |
| Punch-hole | ❌ | NOT TESTED |
| Gesture nav | ✅ | PASS |
| Landscape | ⚠️ | PARTIALLY |

### Summary
- **Compliance**: 85%
- Critical: 1 | Important: 2 | Warning: 1
- **Minimum API Check**: SDK 35 required for edge-to-edge enforcement
- **Recommended**: All apps targeting SDK 35+ must implement enableEdgeToEdge()

### Priority Actions
1. Add `enableEdgeToEdge()` call in Activity.onCreate()
2. Wrap content in `Scaffold` for automatic inset handling
3. Replace hardcoded padding with `Modifier.windowInsetsPadding()`
4. Test on notch/punch-hole devices before release
5. Verify keyboard does not hide form fields
6. Check landscape orientation with display cutouts

### References
- [Android 15 Edge-to-Edge Enforcement](https://developer.android.com/about/versions/15/behavior-changes-15)
- [Display Content Edge-to-Edge (Views)](https://developer.android.com/develop/ui/views/layout/edge-to-edge)
- [Window Insets (Compose)](https://developer.android.com/develop/ui/compose/system/insets)
- [Edge-to-Edge Codelab](https://developer.android.com/codelabs/edge-to-edge)
```

## Example Checklist Output

### Complete Kotlin File Audit

```markdown
## Edge-to-Edge Audit: EventDetailScreen.kt

### Android Version Compliance
- **Target SDK**: 35 (Android 15)
- **Status**: ⚠️ Partial - 70% Compliant
- **Key Issue**: Scaffold not used for top/bottom padding

### Critical Issues (🔴)

**E2E-017**: Content overlaps system bars (no innerPadding)
```kotlin
// Line 42
Scaffold(
  topBar = { EventTopBar() }
) { /* NO innerPadding parameter! */ }

// Line 48
LazyColumn {  // This starts at top=0, behind status bar!
  items(event.details) { /* ... */ }
}

// Fix:
Scaffold(
  topBar = { EventTopBar() }
) { innerPadding ->
  LazyColumn(modifier = Modifier.padding(innerPadding)) {
    items(event.details) { /* ... */ }
  }
}
```
Severity: 🔴 Critical | Rule: E2E-017

### Important Issues (🟠)

**E2E-042**: Join button may be hidden by navigation bar
```kotlin
// Line 95 - No bottom padding for gesture nav
Button(
  onClick = { viewModel.joinEvent() },
  modifier = Modifier
    .align(Alignment.BottomCenter)
    .padding(16.dp)  // Only padding, no inset handling!
) { Text("Join Event") }

// Fix:
Button(
  onClick = { viewModel.joinEvent() },
  modifier = Modifier
    .align(Alignment.BottomCenter)
    .padding(16.dp)
    .windowInsetsPadding(WindowInsets.systemBars.only(WindowInsetsSides.Bottom))
) { Text("Join Event") }
```
Severity: 🟠 Important | Rule: E2E-042

### Summary
- **Compliance**: 70%
- Critical: 1 | Important: 1 | Warning: 0
- **Most Urgent**: Add innerPadding to Scaffold content

### Testing Checklist
- [ ] Test on device with notch
- [ ] Test keyboard interaction with join button
- [ ] Test landscape orientation
- [ ] Verify button visible with gesture nav bar
```

## Common Mistakes & Fixes

### 1. Missing enableEdgeToEdge()

```kotlin
// BAD
class MainActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContent { App() }
  }
}

// GOOD
class MainActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    enableEdgeToEdge()  // ADD THIS
    setContent { App() }
  }
}
```

### 2. Not Using Scaffold innerPadding

```kotlin
// BAD - Content overlaps bars
Scaffold(
  topBar = { TopAppBar(title = { Text("Title") }) },
  bottomBar = { BottomAppBar { } }
) {
  LazyColumn {
    items(count) { /* overlaps bars */ }
  }
}

// GOOD - Proper innerPadding usage
Scaffold(
  topBar = { TopAppBar(title = { Text("Title") }) },
  bottomBar = { BottomAppBar { } }
) { innerPadding ->
  LazyColumn(modifier = Modifier.padding(innerPadding)) {
    items(count) { /* safe from bars */ }
  }
}
```

### 3. Hardcoded System Bar Heights

```kotlin
// BAD - Hardcoded values don't adapt
Column(modifier = Modifier.padding(top = 24.dp)) {
  /* content */
}

// GOOD - Uses actual inset values
Column(
  modifier = Modifier.windowInsetsPadding(WindowInsets.systemBars)
) {
  /* content */
}
```

### 4. FAB Obscured by Nav Bar

```kotlin
// BAD - FAB behind gesture nav
FloatingActionButton(
  onClick = { /* ... */ },
  modifier = Modifier
    .align(Alignment.BottomEnd)
    .padding(16.dp)
)

// GOOD - FAB above nav bar
FloatingActionButton(
  onClick = { /* ... */ },
  modifier = Modifier
    .align(Alignment.BottomEnd)
    .padding(16.dp)
    .windowInsetsPadding(WindowInsets.systemBars.only(WindowInsetsSides.Bottom))
)
```

### 5. Text Fields Hidden by Keyboard

```kotlin
// BAD - Form field may hide behind keyboard
Column {
  LazyColumn(Modifier.weight(1f)) { items(messages) { /* ... */ } }
  OutlinedTextField(value = input, onValueChange = { input = it })
}

// GOOD - Proper keyboard spacing
Column(
  modifier = Modifier
    .fillMaxSize()
    .imePadding()
) {
  LazyColumn(Modifier.weight(1f)) { items(messages) { /* ... */ } }
  OutlinedTextField(value = input, onValueChange = { input = it })
}
```

### 6. Content at Screen Corners (Rounded Corner Devices)

```kotlin
// BAD - Icon at exact corner
Icon(
  Icons.Default.Settings,
  modifier = Modifier
    .align(Alignment.TopEnd)
    .offset((-8).dp, 8.dp)  // Too close to corner!
)

// GOOD - Proper spacing from corners
Icon(
  Icons.Default.Settings,
  modifier = Modifier
    .align(Alignment.TopEnd)
    .padding(16.dp)  // Safe distance from corner
)
```

## Key References

- **[Android 15 Behavior Changes](https://developer.android.com/about/versions/15/behavior-changes-15)** - Official enforcement details
- **[Display Content Edge-to-Edge](https://developer.android.com/develop/ui/views/layout/edge-to-edge)** - Setup guide for Views
- **[About Window Insets (Compose)](https://developer.android.com/develop/ui/compose/system/insets)** - WindowInsets API
- **[Set Up Window Insets (Compose)](https://developer.android.com/develop/ui/compose/system/insets-ui)** - Compose inset patterns
- **[Edge-to-Edge Design Codelab](https://developer.android.com/codelabs/edge-to-edge)** - Interactive tutorial
- **[Mastering Edge-to-Edge in Android](https://medium.com/@farimarwat/mastering-edge-to-edge-in-android-with-windowinsets-cc469168ba34)** - Deep dive

## Target SDKs

| SDK | Version | Edge-to-Edge | Status |
|-----|---------|-------------|--------|
| 34 | Android 14 | Optional | Not enforced |
| 35 | Android 15 | **Mandatory** | Enforced by system |
| 36+ | Android 16+ | **Mandatory** | Enforced by system |

**Note**: `windowOptOutEdgeToEdgeEnforcement` attribute is deprecated and will be disabled in Android 16+. Plan migration accordingly.

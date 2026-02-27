---
name: adaptive-behaviors
description: Comprehensive reference for platform-specific behavior patterns (iOS vs Android) including navigation, notifications, forms, keyboards, and haptics
args:
  - name: component
    type: string
    description: Component or feature to adapt (e.g., navigation, notifications, forms, keyboards)
    required: false
  - name: platform
    type: string
    enum: [ios, android, both]
    description: Target platform(s) (default: both)
    required: false
  - name: output_format
    type: string
    enum: [checklist, code, guide, mapping, testing]
    description: Output format (default: guide)
    required: false
platform_specs:
  - ios
  - android
behavior_domains:
  - navigation
  - notifications
  - forms
  - keyboards
  - haptics
  - permissions
  - pull_to_refresh
  - context_menus
  - share_sheets
  - animations
---

# Platform-Adaptive Behavior Patterns (iOS vs Android)

Comprehensive reference for implementing platform-native behaviors in cross-platform apps. Covers iOS Human Interface Guidelines (HIG) vs Material Design 3 differences.

## Overview

iOS and Android users have fundamentally different interaction models and expectations:

| Aspect | iOS | Android |
|--------|-----|---------|
| Design Language | Flat, minimal, system-integrated | Material Design 3 (bold, dynamic theming) |
| Navigation Model | Hierarchical with back gesture | Hierarchical with back button |
| Primary Actions | Bottom tab bar (max 5 tabs) | Bottom nav bar or hamburger drawer |
| Feedback | Haptics + subtle animations | Ripple effects + visual feedback |
| Customization | Limited by system | Extensive theming and customization |
| User Mindset | Premium, integrated ecosystem | Control and personalization |

**Golden Rule**: Do NOT copy iOS patterns to Android or vice versa. Respect each platform's native paradigms.

---

## AB-001 to AB-050: Adaptive Behavior Checklist

### Navigation & Back Button (AB-001 to AB-010)

- [ ] **AB-001**: iOS uses back gesture (swipe from left edge) for navigation
- [ ] **AB-002**: Android uses dedicated back button (hardware or navigation bar)
- [ ] **AB-003**: iOS swipe gesture has haptic feedback (light impact)
- [ ] **AB-004**: Android back button shows visual ripple/highlight
- [ ] **AB-005**: iOS: Implement custom back button ONLY for special flows (e.g., modal dismiss with unsaved changes)
- [ ] **AB-006**: Android: Always honor system back button; use predictive back gesture (API 33+)
- [ ] **AB-007**: iOS: Navigation title appears in large format on screen (vs centered)
- [ ] **AB-008**: Android: Navigation title is centered; use hamburger menu for secondary navigation
- [ ] **AB-009**: iOS supports multiple back navigation (stack-based); back gesture only goes one step
- [ ] **AB-010**: Android: Back button on navigation bar respects app navigation hierarchy

### Tab Bar Positioning (AB-011 to AB-020)

- [ ] **AB-011**: iOS tab bar is ALWAYS at bottom of screen (human thumb reach)
- [ ] **AB-012**: Android bottom navigation bar is also bottom, but primary nav is often hamburger drawer (top-left)
- [ ] **AB-013**: iOS: Max 5 tabs, overflow uses "More" menu (6th tab shows 3 dots)
- [ ] **AB-014**: Android: 3-5 bottom nav items standard; additional items hidden in drawer
- [ ] **AB-015**: iOS: Tab bar icons MUST have labels; labels always visible
- [ ] **AB-016**: Android: Icons may be label-less on bottom nav; labels optional (but recommended for clarity)
- [ ] **AB-017**: iOS: Selected tab shows full opacity, unselected at 50% opacity
- [ ] **AB-018**: Android: Selected tab uses Material You primary color; unselected uses onSurfaceVariant
- [ ] **AB-019**: iOS: Tab bar appears BELOW content (safe area aware); content doesn't scroll under tabs
- [ ] **AB-020**: Android: Bottom navigation appears BELOW content; content doesn't scroll under nav

### Notification & Alert Styles (AB-021 to AB-030)

- [ ] **AB-021**: iOS uses "Alerts" (centered, modal interruption) for critical decisions
- [ ] **AB-022**: Android uses "Snackbars" (bottom temporary toast) for non-blocking feedback
- [ ] **AB-023**: iOS alerts have primary + secondary buttons (max 2-3); stacked on small screens
- [ ] **AB-024**: Android snackbars have single action button (optional); auto-dismiss in 4-10 seconds
- [ ] **AB-025**: iOS dismissal: tap outside alert or button; swipe NOT supported on alerts
- [ ] **AB-026**: Android dismissal: tap action button, swipe left/right, or wait for auto-dismiss
- [ ] **AB-027**: iOS: Success/info messages can use small banner at top (non-modal)
- [ ] **AB-028**: Android: Toast notifications (system-level) appear as brief floating message
- [ ] **AB-029**: iOS: Sound + haptic feedback on critical alerts (configurable in Settings)
- [ ] **AB-030**: Android: Haptic feedback on snackbar swipe/action; sound from notification channel settings

### Form Validation Patterns (AB-031 to AB-040)

- [ ] **AB-031**: iOS: Validation errors shown below field in red text; field border turns red
- [ ] **AB-032**: Android: Validation errors in M3 use error color (dynamically themed) with icon + text
- [ ] **AB-033**: iOS: Real-time validation on blur (field loses focus); don't validate while typing
- [ ] **AB-034**: Android: Real-time validation with visual feedback; show error immediately as user types
- [ ] **AB-035**: iOS: Required field indicators (*) shown before label text
- [ ] **AB-036**: Android: Required field indicators shown after label or as helper text
- [ ] **AB-037**: iOS: Submit button disabled (greyed out) if form invalid; no ripple on disabled state
- [ ] **AB-038**: Android: Submit button disabled with opacity change; Material ripple disabled when !isEnabled
- [ ] **AB-039**: iOS: Form field focus shows blue outline; keyboard appears automatically
- [ ] **AB-040**: Android: Form field focus shows underline accent color; keyboard appears with transition

### Keyboard Behavior Differences (AB-041 to AB-050)

- [ ] **AB-041**: iOS keyboard has "return" key (varies: Go, Search, Done, Next per field)
- [ ] **AB-042**: Android keyboard has "Done" or "Enter" key; next field navigation via Tab (rare)
- [ ] **AB-043**: iOS: Keyboard dismisses with "Done" tap, outside tap, or swipe down on keyboard
- [ ] **AB-044**: Android: Keyboard dismisses with back button, outside tap, or IME close
- [ ] **AB-045**: iOS: Email keyboard shows .com/.net suggestions below; @ symbol visible on default keyboard
- [ ] **AB-046**: Android: Email keyboard includes @ by default; autocomplete via Credentials API
- [ ] **AB-047**: iOS: Password field hides characters; optionally shows last character for 1 sec
- [ ] **AB-048**: Android: Password field has toggle visibility icon (eye icon) on right
- [ ] **AB-049**: iOS: Number keyboard for phone/zip fields; formatted automatically (XXX-XXX-XXXX in US)
- [ ] **AB-050**: Android: Phone/zip keyboard shows numeric grid; formatting via InputFilter or TextWatcher

### Haptic Feedback Patterns (AB-051 to AB-060)

- [ ] **AB-051**: iOS haptics: Light impact (UIImpactFeedbackStyle.light) on button tap
- [ ] **AB-052**: iOS haptics: Medium impact on form validation error
- [ ] **AB-053**: iOS haptics: Heavy impact on critical confirmation (delete, logout)
- [ ] **AB-054**: iOS haptics: Selection feedback (tick) on segmented control/picker change
- [ ] **AB-055**: Android haptics: Ripple effect on tap provides visual feedback (no haptic by default)
- [ ] **AB-056**: Android haptics: Enable device vibration if user toggles in app settings
- [ ] **AB-057**: Android haptics: 20ms vibration on button tap; 50ms on error
- [ ] **AB-058**: iOS: Always use UIFeedbackGenerator (or SwiftUI .sensoryFeedback); no custom vibration
- [ ] **AB-059**: Android: Use VibrationEffect API; respect device vibration settings
- [ ] **AB-060**: Both: Haptics MUST respect accessibility settings (e.g., "Reduce Motion" on iOS)

### Pull-to-Refresh Variations (AB-061 to AB-070)

- [ ] **AB-061**: iOS pull-to-refresh: Swipe down from top; animated spinner + "Release to refresh" text
- [ ] **AB-062**: Android pull-to-refresh: Swipe down from top; Material spinner (circular, animated color)
- [ ] **AB-063**: iOS: Refresh animation shows circular progress; spins clockwise
- [ ] **AB-064**: Android: Refresh shows Material circular progress indicator with primary color
- [ ] **AB-065**: iOS: Completion animation (checkmark or fade); haptic feedback on complete
- [ ] **AB-066**: Android: Completion shows brief success message or fade; optional haptic
- [ ] **AB-067**: iOS: Pull-to-refresh respects safe areas (status bar, notch)
- [ ] **AB-068**: Android: Pull-to-refresh respects status bar; works on ScrollView, LazyColumn
- [ ] **AB-069**: iOS: Disable pull-to-refresh for non-scrollable content
- [ ] **AB-070**: Android: Only enable on scrollable, refreshable content (lists, grids, web content)

### Context Menus & Long-Press Actions (AB-071 to AB-080)

- [ ] **AB-071**: iOS uses "swipe left" to reveal actions (edit, delete) on list items
- [ ] **AB-072**: Android uses long-press (800ms hold) to show context menu or action bar
- [ ] **AB-073**: iOS swipe actions: Red delete button on right; blue/custom actions available via swipe
- [ ] **AB-074**: Android long-press: Shows contextual action bar at top with icons + text
- [ ] **AB-075**: iOS: Swipe actions dismissable by tapping cell or swiping back
- [ ] **AB-076**: Android: Action bar dismissable by back button or tapping neutral area
- [ ] **AB-077**: iOS: Preview swipe action with label visible before full reveal
- [ ] **AB-078**: Android: Long-press haptic feedback (medium vibration) + visual highlight
- [ ] **AB-079**: iOS: Support drag-to-reorder on long-press (iOS 16+)
- [ ] **AB-080**: Android: Support long-press selection (multi-select) for batch actions

### Share Sheet Integration (AB-081 to AB-090)

- [ ] **AB-081**: iOS: Share sheet triggered by UIActivityViewController; system sheet with preset apps
- [ ] **AB-082**: Android: Share triggered by Intent.ACTION_SEND; system chooser with all capable apps
- [ ] **AB-083**: iOS: Share sheet appears from bottom with haptic feedback
- [ ] **AB-084**: Android: Share chooser appears centered on screen
- [ ] **AB-085**: iOS: Share data types (text, URL, image) auto-filtered by app capability
- [ ] **AB-086**: Android: ShareCompat.IntentBuilder ensures safe data passing; MIME type required
- [ ] **AB-087**: iOS: Custom share actions via UIActivity subclass (rare, use standard when possible)
- [ ] **AB-088**: Android: Custom share via ContentProvider (for large data) or URI-based
- [ ] **AB-089**: iOS: Copy-to-clipboard as fallback if share fails
- [ ] **AB-090**: Android: Show snackbar confirmation after successful share

### Permission Request Flows (AB-091 to AB-100)

- [ ] **AB-091**: iOS: Request permissions in-app with system dialog (one permission at a time)
- [ ] **AB-092**: Android: Request permissions via requestPermissions() (runtime, scoped storage on Q+)
- [ ] **AB-093**: iOS: Permissions: Camera, Photo Library, Contacts, Location, Calendar, etc.
- [ ] **AB-094**: Android: Permissions: READ_MEDIA_IMAGES, ACCESS_FINE_LOCATION, RECORD_AUDIO, etc.
- [ ] **AB-095**: iOS: Denied permission shows "Settings" button to open app Settings
- [ ] **AB-096**: Android: Denied permission allows retry; "Never ask again" shows Settings button
- [ ] **AB-097**: iOS: Each permission request needs user-facing rationale in Info.plist
- [ ] **AB-098**: Android: Show rationale dialog BEFORE system prompt if permission previously denied
- [ ] **AB-099**: iOS: Photo/Camera access: limited vs full; user can select specific photos
- [ ] **AB-100**: Android: READ_MEDIA_IMAGES/VIDEO; Photo Picker API for user-selected photos only

---

## Platform Behavior Mapping (Android ↔ iOS)

### Navigation & Hierarchy

| Feature | Android | iOS |
|---------|---------|-----|
| **Primary Navigation** | Bottom nav bar (3-5 items) or hamburger drawer | Bottom tab bar (max 5 items) |
| **Back Navigation** | Back button (hardware or nav bar) | Swipe from left edge (gesture) |
| **Back Button Visual** | Chevron/arrow left | Chevron left (< symbol) |
| **Secondary Navigation** | Drawer (hamburger, slide from left) | More menu (3 dots, 6th tab) |
| **Back Button Feedback** | Ripple effect + visual highlight | Haptic light impact (UIImpactFeedbackStyle.light) |
| **Screen Title** | Centered, left-aligned optional | Large format, left-aligned |
| **Predictive Back** | API 33+ with BackEventCompat | Not applicable (gesture is native) |
| **Navigation Title Styling** | Headline4 / headlineSmall | NavStack title modifier |

### Notifications & Feedback

| Feature | Android | iOS |
|---------|---------|-----|
| **Non-Blocking Feedback** | Snackbar (bottom, 4-10 sec auto-dismiss) | Banner at top (non-modal) |
| **Critical Alert** | AlertDialog (modal) | UIAlertController (modal, centered) |
| **Success Message** | Toast (brief, floating) | Banner or custom notification |
| **Error Feedback** | Snackbar with action button | Alert or banner + field highlight |
| **Haptic on Action** | 20ms vibration (optional) | Medium impact feedback |
| **Auto-Dismiss** | Snackbar: 4-10 sec default | Banner: 5-6 sec typical |
| **Dismissal Gesture** | Swipe left/right | Swipe up or tap outside |

### Form & Input Handling

| Feature | Android | iOS |
|---------|---------|-----|
| **Validation Timing** | Real-time (while typing) | On blur (field loses focus) |
| **Error Display** | M3 error color, icon + text below | Red text below, field border red |
| **Field Focus Indicator** | Underline accent color | Blue outline + blue underline |
| **Required Field Marker** | After label or helper text | Asterisk (*) before label |
| **Disabled State** | Opacity 0.5 + no ripple | Greyed out, no interaction |
| **Submit Button Disabled** | Disabled: opacity/color change | Disabled: greyed out |
| **Keyboard Type** | Email, Number, Phone, Text, etc. | KeyboardType enum (email, numberPad, default) |
| **Keyboard Dismissal** | Back button or IME close | Done key or swipe down |

### Keyboard Behavior

| Feature | Android | iOS |
|---------|---------|-----|
| **Return Key Label** | "Done", "Go", "Search", "Next" | "Return", "Done", "Go", "Search" |
| **Email Keyboard** | @ symbol visible | .com/.net suggestions below |
| **Password Field** | Eye icon toggle to show/hide | Last character visible 1 sec, then hidden |
| **Phone Number Format** | Via InputFilter or TextWatcher | Auto-formatted (XXX-XXX-XXXX US) |
| **Number Keyboard** | Numeric grid | Numeric pad |
| **Keyboard Animation** | Slides up from bottom | Slides up from bottom (same) |
| **Keyboard Handling** | Adjust pan (resize content) | Adjust offset or keyboard avoidance |
| **Auto-correct** | Enabled by default; user can disable | Enabled; autocomplete via suggestions |

### Haptic & Sensory Feedback

| Feature | Android | iOS |
|---------|---------|-----|
| **Button Tap Feedback** | Visual ripple (no haptic) | Light haptic impact + ripple |
| **Error Feedback** | Optional vibration 50ms | Medium impact (UIImpactFeedbackStyle.medium) |
| **Success Feedback** | Optional vibration 20ms | Light impact + haptic selection |
| **Long-Press Feedback** | Medium vibration 50ms | Long-press available (drag reorder) |
| **Gesture Feedback** | Haptic optional on swipe | Selection feedback (tick) on picker change |
| **API** | VibrationEffect API | UIFeedbackGenerator (UIKit) or .sensoryFeedback (SwiftUI) |
| **Accessibility** | Respect vibration toggle in settings | Respect "Reduce Motion" preference |

### Pull-to-Refresh

| Feature | Android | iOS |
|---------|---------|-----|
| **Trigger** | Swipe down from top | Swipe down from top (same) |
| **Visual Indicator** | Material circular spinner | Circular spinner + "Release to refresh" text |
| **Spinner Color** | Primary color (Material You) | App tint color |
| **Completion Feedback** | Fade or toast message | Checkmark + haptic feedback |
| **Safe Area** | Respects status bar | Respects notch/safe area |
| **Content Offset** | Adjusts dynamically | Controlled via refreshControl |

### Context Menus & Actions

| Feature | Android | iOS |
|---------|---------|-----|
| **Trigger** | Long-press (800ms hold) | Swipe left on list item |
| **Menu Style** | Contextual action bar (top) | Swipe reveal (inline, right side) |
| **Actions Display** | Icon + text in top bar | Color-coded buttons (red delete, blue edit) |
| **Dismissal** | Back button or tap neutral area | Swipe back left or tap cell |
| **Haptic Feedback** | Medium vibration on long-press | Light haptic on swipe reveal |
| **Multi-Select** | Long-press initiates, checkbox toggle | Not standard (use swipe for delete/edit) |
| **Drag-to-Reorder** | Long-press + drag (Compose) | Long-press + drag (iOS 16+) |

### Share Sheet

| Feature | Android | iOS |
|---------|---------|-----|
| **Trigger** | Share button → Intent chooser | Share button → UIActivityViewController |
| **Appearance** | Centered modal with apps list | Bottom sheet with preset apps + actions |
| **Content Types** | MIME type required | Auto-filtered by app capability |
| **Fallback** | Show snackbar if no apps available | Copy-to-clipboard button in sheet |
| **Custom Actions** | Via ContentProvider (rare) | Via UIActivity subclass (rare) |
| **Recent Targets** | System shows recently used apps | System shows frequently used apps |

### Permissions

| Feature | Android | iOS |
|---------|---------|-----|
| **Request Model** | Runtime (API 23+) | In-app system dialog |
| **Multiple Permissions** | Can request in batch (Compose) | One at a time |
| **Denied Handling** | "Never ask again" or retry | Settings button to enable |
| **Rationale** | Show before system prompt if previously denied | Info.plist entry (automatic system) |
| **Camera/Photo** | READ_MEDIA_IMAGES (Q+) | Photo Library limited access |
| **Location** | ACCESS_FINE_LOCATION (GPS) | CLLocationManager.requestWhenInUseAuthorization() |
| **Contacts** | READ_CONTACTS | CNContactStore |
| **Calendar** | READ_CALENDAR, WRITE_CALENDAR | EKEventStore |
| **Microphone** | RECORD_AUDIO | AVAudioSession.recordPermission() |

---

## Code Examples for Adaptive Behavior

### Navigation (Swipe vs Back Button)

**iOS - Custom Back Gesture (rare):**
```swift
struct EventDetailView: View {
    @Environment(\.dismiss) var dismiss

    var body: some View {
        VStack {
            HStack {
                Button(action: { dismiss() }) {
                    HStack {
                        Image(systemName: "chevron.left")
                        Text("Back")
                    }
                }
                Spacer()
            }
            .padding()

            // Content...
        }
        // SwiftUI handles swipe gesture automatically
    }
}
```

**Android - Predictive Back (API 33+):**
```kotlin
@Composable
fun EventDetailScreen(onNavigateBack: () -> Unit) {
    BackHandler(enabled = true) {
        onNavigateBack()
    }

    Column {
        // Navigation handled by system back button
        // Predictive back gesture animates automatically
        EventContent()
    }
}
```

### Form Validation Pattern

**iOS - Validate on Blur:**
```swift
@State var email = ""
@State var emailError: String? = nil

TextField("Email", text: $email)
    .onReceive(Just(email)) { newValue in
        // Clear error when user starts typing
        emailError = nil
    }
    .onSubmit {
        // Validate when field loses focus (Done key)
        if !newValue.contains("@") {
            emailError = "Invalid email format"
        }
    }

if let error = emailError {
    Text(error).foregroundColor(.red).font(.caption)
}
```

**Android - Real-Time Validation:**
```kotlin
var email by remember { mutableStateOf("") }
var emailError by remember { mutableStateOf<String?>(null) }

OutlinedTextField(
    value = email,
    onValueChange = { newValue ->
        email = newValue
        // Validate in real-time
        emailError = if (!newValue.contains("@")) {
            "Invalid email format"
        } else null
    },
    isError = emailError != null,
    label = { Text("Email") },
    supportingText = {
        if (emailError != null) {
            Text(emailError!!, color = MaterialTheme.colorScheme.error)
        }
    }
)
```

### Notifications Pattern

**iOS - Alert:**
```swift
@State var showAlert = false

Button("Delete") {
    showAlert = true
}
.alert("Confirm Deletion", isPresented: $showAlert) {
    Button("Delete", role: .destructive) { /* delete */ }
    Button("Cancel", role: .cancel) { }
} message: {
    Text("This action cannot be undone.")
}
```

**Android - Snackbar:**
```kotlin
val snackbarHostState = remember { SnackbarHostState() }

Button(onClick = {
    scope.launch {
        snackbarHostState.showSnackbar(
            message = "Event deleted",
            actionLabel = "Undo",
            duration = SnackbarDuration.Long
        )
    }
}) {
    Text("Delete")
}

SnackbarHost(hostState = snackbarHostState)
```

### Haptic Feedback

**iOS - UIFeedbackGenerator:**
```swift
import UIKit

func showSuccess() {
    let impact = UIImpactFeedbackGenerator(style: .light)
    impact.impactOccurred()
}

func showError() {
    let impact = UIImpactFeedbackGenerator(style: .medium)
    impact.impactOccurred()
}

func showSelection() {
    let selection = UISelectionFeedbackGenerator()
    selection.selectionChanged()
}
```

**Android - VibrationEffect:**
```kotlin
import android.os.VibrationEffect
import android.content.Context

fun showSuccess(context: Context) {
    val vibrator = context.getSystemService(Context.VIBRATOR_SERVICE) as Vibrator
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        vibrator.vibrate(
            VibrationEffect.createOneShot(20, VibrationEffect.DEFAULT_AMPLITUDE)
        )
    }
}

fun showError(context: Context) {
    val vibrator = context.getSystemService(Context.VIBRATOR_SERVICE) as Vibrator
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        vibrator.vibrate(
            VibrationEffect.createOneShot(50, VibrationEffect.DEFAULT_AMPLITUDE)
        )
    }
}
```

### Context Menu / Swipe Actions

**iOS - Swipe Actions:**
```swift
List {
    ForEach(events) { event in
        EventRow(event: event)
            .swipeActions(edge: .trailing) {
                Button(role: .destructive) {
                    deleteEvent(event)
                } label: {
                    Label("Delete", systemImage: "trash")
                }

                Button {
                    editEvent(event)
                } label: {
                    Label("Edit", systemImage: "pencil")
                }
            }
    }
}
```

**Android - Long-Press Context Menu:**
```kotlin
@Composable
fun EventCard(event: Event, onDelete: () -> Unit) {
    var expanded by remember { mutableStateOf(false) }

    Card(
        modifier = Modifier
            .fillMaxWidth()
            .combinedClickable(
                onClick = { /* navigate to detail */ },
                onLongClick = { expanded = true }
            )
    ) {
        EventContent(event)

        DropdownMenu(expanded = expanded, onDismissRequest = { expanded = false }) {
            DropdownMenuItem(
                text = { Text("Edit") },
                onClick = { /* edit */ }
            )
            DropdownMenuItem(
                text = { Text("Delete") },
                onClick = { onDelete() }
            )
        }
    }
}
```

### Pull-to-Refresh

**iOS - RefreshControl:**
```swift
List {
    ForEach(events) { event in
        EventRow(event: event)
    }
}
.refreshable {
    await loadEvents()
}
```

**Android - SwipeRefreshLayout (Compose):**
```kotlin
val refreshState = rememberPullRefreshState(
    refreshing = isRefreshing,
    onRefresh = { viewModel.loadEvents() }
)

Box(modifier = Modifier.pullRefresh(refreshState)) {
    LazyColumn {
        items(events) { event ->
            EventCard(event)
        }
    }

    PullRefreshIndicator(
        refreshing = isRefreshing,
        state = refreshState,
        modifier = Modifier.align(Alignment.TopCenter)
    )
}
```

### Share Sheet

**iOS:**
```swift
ShareLink(
    item: URL(string: "https://party-on.app/events/\(event.id)")!,
    message: Text("Join my event: \(event.name)")
) {
    Label("Share", systemImage: "square.and.arrow.up")
}
```

**Android:**
```kotlin
Button(onClick = {
    val sendIntent = Intent().apply {
        action = Intent.ACTION_SEND
        putExtra(Intent.EXTRA_TEXT, "Join my event: ${event.name}\nhttps://party-on.app/events/${event.id}")
        type = "text/plain"
    }
    context.startActivity(Intent.createChooser(sendIntent, "Share Event"))
}) {
    Text("Share")
}
```

---

## Testing Checklist Per Platform

### iOS Testing (Simulator & Device)

**Navigation & Gestures:**
- [ ] Back gesture works from left edge (3D Touch or edge swipe)
- [ ] Back gesture animates smoothly with interactive preview
- [ ] Tab bar appears at bottom; content does NOT scroll under tabs
- [ ] Tab selection changes color/opacity correctly
- [ ] "More" menu appears for 6+ tabs

**Forms & Keyboard:**
- [ ] Email field shows @ on default keyboard
- [ ] Password field hides characters; shows last char 1 sec
- [ ] Phone field auto-formats to XXX-XXX-XXXX (US)
- [ ] Return key label correct per field (Done, Go, Search, Next)
- [ ] Keyboard dismisses on "Done" key
- [ ] Form validation shows on blur, not during typing

**Feedback:**
- [ ] Light haptic on button tap
- [ ] Medium haptic on error validation
- [ ] Success/error banner appears at top (below notch if needed)
- [ ] Alert dialog is modal, centered
- [ ] Pull-to-refresh spinner works; respects safe area

**Share Sheet:**
- [ ] Share button opens UIActivityViewController from bottom
- [ ] Standard share apps visible (Mail, Messages, Notes, etc.)
- [ ] Custom share actions available if needed
- [ ] Copy-to-clipboard as fallback

**Permissions:**
- [ ] First permission request shows system dialog
- [ ] Subsequent requests honor "Don't Ask Again" choice
- [ ] Settings button appears if denied

### Android Testing (Emulator & Device)

**Navigation & Buttons:**
- [ ] Back button (hardware or nav bar) navigates correctly
- [ ] Back button shows ripple effect
- [ ] Predictive back gesture animates (API 33+)
- [ ] Bottom nav bar appears; primary navigation works
- [ ] Drawer/hamburger menu opens/closes smoothly

**Forms & Keyboard:**
- [ ] Email field includes @ symbol
- [ ] Password field has eye icon toggle
- [ ] Phone field via InputFilter or TextWatcher
- [ ] Validation errors show in M3 error color with icon
- [ ] Keyboard dismisses on back button or IME close
- [ ] Real-time validation feedback as user types

**Feedback:**
- [ ] Ripple effect on button tap
- [ ] Snackbar appears at bottom, auto-dismisses
- [ ] Snackbar action button works (e.g., Undo)
- [ ] Pull-to-refresh shows Material spinner; colors correct
- [ ] Long-press shows contextual action bar with vibration

**Share Sheet:**
- [ ] Share button opens system chooser
- [ ] All capable apps appear in chooser
- [ ] Share data passes correctly via Intent

**Permissions:**
- [ ] Runtime permission request shows system dialog
- [ ] Rationale dialog shows if previously denied
- [ ] Settings button opens app Settings
- [ ] Multiple permissions can be requested in batch

---

## Design System Alignment

### Material Design 3 (Android)

**Colors:**
- Primary: Dynamic brand color (user-set or system)
- Secondary: Complementary, softer color
- Tertiary: Accent for highlights, warnings
- Error: Red (#B3261E dark, #F2DEDE light)
- Surface: Background, respects light/dark theme

**Typography:**
- Display: headlines (large, bold)
- Headline: section titles
- Title: medium emphasis
- Body: default text
- Label: smaller, UI elements

**Components:**
- Button: filled, outlined, elevated, text
- Card: surface + elevation
- TextField: filled or outlined style
- AppBar: top app bar (Material 3 simplified)
- BottomNavigation: 3-5 items, icons + labels

### Human Interface Guidelines (iOS)

**Colors:**
- System colors (blue, red, green, yellow, orange, pink, purple, teal)
- Semantic colors: Primary action (system blue), Destructive (system red)
- SF Symbols: Monochrome icons, consistent weights

**Typography:**
- Large Title: 34pt, prominent
- Title: 20pt bold
- Headline: 17pt bold
- Body: 17pt regular
- Caption: 13pt lighter

**Components:**
- Button: contained or plain style
- TextField: default or secure input
- Picker: wheel or inline date picker
- List: row-based, swipe actions
- TabView: tab bar at bottom
- NavigationStack: hierarchical navigation

---

## Quick Reference: When to Use What

| Scenario | iOS | Android |
|----------|-----|---------|
| User needs to go back | Swipe from left edge | Tap back button |
| Show success message | Banner at top | Snackbar at bottom |
| Block user for critical action | Alert (modal) | AlertDialog or custom |
| Let user dismiss non-critical feedback | Banner with timeout | Snackbar with auto-dismiss |
| Show list item actions | Swipe left reveal | Long-press context menu |
| Request single permission | System dialog (1 at a time) | System dialog (batch request) |
| Validate form field | On blur (field loses focus) | Real-time as user types |
| Enable haptic feedback | UIFeedbackGenerator | VibrationEffect API |
| Share content | UIActivityViewController | Intent.ACTION_SEND |
| Refresh content | Pull down, refreshable | Pull down, SwipeRefresh |

---

## Sources & References

Research compiled from:

- [Key UX Differences: iOS vs Android in 2025 | Medium](https://medium.com/@markeltree/key-ux-differences-ios-vs-android-in-2025-6475855e2afe)
- [9 Differences Between iOS and Android UI Design | UXPin](https://www.uxpin.com/studio/blog/ios-vs-andoid-ui-design-for-mobile/)
- [Apple's Human Interface Guidelines vs Google's Material Design Guidelines | Medium](https://medium.com/@shivaniy0211/apples-human-interface-guidelines-vs-google-s-material-design-guidelines-e28db15028c0)
- [Material Design vs Human Interface Design: Major Differences | AppsChopper](https://www.appschopper.com/blog/google-material-design-apple-human-interface-guidelines-which-is-better-and-why/)
- [Build adaptive apps | Jetpack Compose](https://developer.android.com/develop/ui/compose/build-adaptive-apps)
- [Adaptive design – Material Design 3](https://m3.material.io/foundations/adaptive-design)
- [Automatic platform adaptations | Flutter](https://docs.flutter.dev/ui/adaptive-responsive/platform-adaptations)

---
name: cross-platform-components
description: Comprehensive guide to Material 3 and SwiftUI component parity, decision trees for platform-specific vs. shared implementations, accessibility mapping, and code examples for equivalent components across Android and iOS
args:
  - name: component_type
    type: string
    enum: [buttons, textfields, navigation, lists, cards, dialogs, bottomsheets, chips, progress, all]
    description: Specific component category to focus on (default: all)
    required: false
  - name: use_case
    type: string
    enum: [primary, secondary, tertiary, elevated, filled, outlined, contained]
    description: Component variant/emphasis level (default: primary)
    required: false
  - name: output_format
    type: string
    enum: [matrix, decision_tree, code, checklist, comparison]
    description: Format for the output (default: matrix)
    required: false
parity_phases:
  - assessment
  - mapping
  - decision_tree
  - implementation
  - accessibility
  - testing
---

# Cross-Platform Component Parity Guide

Ensure consistent user experience across Material 3 (Android/Jetpack Compose) and SwiftUI (iOS) with this comprehensive component mapping reference. Based on 2025-2026 design system best practices.

## Overview

PartyOn uses two distinct native frameworks:

- **Android**: Jetpack Compose + Material 3 design tokens
- **iOS**: SwiftUI + iOS Human Interface Guidelines (adapted to Material 3 aesthetic)

**Key Principle**: Both platforms should feel native to their users while maintaining visual and interaction parity. This means:
- Material 3 colors/tokens used on both platforms
- Platform-specific UI idioms (iOS: sheet/stack navigation, Android: back button/nav drawer)
- Shared accessibility patterns (labels, states, announcements)

---

## Phase 1: Component Parity Matrix

### 1. Buttons (Interactive Actions)

#### Material 3 Button Variants

| Variant | Android (Compose) | iOS (SwiftUI) | Use Case | Emphasis |
|---------|------------------|---------------|----------|----------|
| **Filled** | `Button()` with `colors = ButtonDefaults.buttonColors(containerColor = primary)` | `Button(action:) { }.buttonStyle(.borderedProminent)` | Primary action (submit, confirm) | High |
| **Outlined** | `OutlinedButton()` | `Button(action:) { }.buttonStyle(.bordered)` | Secondary action, less emphasis | Medium |
| **Text** | `TextButton()` | `Button(action:) { }` with `.foregroundColor(.accentColor)` | Tertiary action (cancel, dismiss) | Low |
| **Tonal** (filled w/ low emphasis) | `Button(colors = ButtonDefaults.buttonColors(containerColor = secondaryContainer))` | Custom: `Button() { }.background(Color.accentColor.opacity(0.2))` | Secondary with more emphasis | Medium-High |
| **Elevated** | `ElevatedButton()` | `Button() { }.buttonStyle(.automatic)` with `.shadow()` | Important secondary action | Medium |

#### Button Implementation Examples

**Android (Jetpack Compose):**
```kotlin
// Filled Button (Primary)
Button(
    onClick = { viewModel.joinEvent() },
    modifier = Modifier
        .fillMaxWidth()
        .height(48.dp),
    colors = ButtonDefaults.buttonColors(
        containerColor = MaterialTheme.colorScheme.primary,
        contentColor = MaterialTheme.colorScheme.onPrimary
    ),
    shape = RoundedCornerShape(8.dp)
) {
    Text("Join Event", style = MaterialTheme.typography.labelLarge)
}

// Outlined Button (Secondary)
OutlinedButton(
    onClick = { /* */ },
    modifier = Modifier
        .fillMaxWidth()
        .height(48.dp),
    border = BorderStroke(1.dp, MaterialTheme.colorScheme.primary),
    shape = RoundedCornerShape(8.dp)
) {
    Text("Cancel", color = MaterialTheme.colorScheme.primary)
}

// Text Button (Tertiary)
TextButton(
    onClick = { /* */ },
    modifier = Modifier.padding(8.dp)
) {
    Text("Dismiss", color = MaterialTheme.colorScheme.primary)
}
```

**iOS (SwiftUI):**
```swift
// Filled Button (Primary)
Button(action: { viewModel.joinEvent() }) {
    Text("Join Event")
        .font(.system(size: 16, weight: .semibold))
}
.frame(maxWidth: .infinity)
.frame(height: 48)
.foregroundColor(.white)
.background(Color.accentColor)
.cornerRadius(8)

// Outlined Button (Secondary)
Button(action: { }) {
    Text("Cancel")
        .font(.system(size: 16, weight: .semibold))
}
.frame(maxWidth: .infinity)
.frame(height: 48)
.foregroundColor(.accentColor)
.overlay(
    RoundedRectangle(cornerRadius: 8)
        .stroke(Color.accentColor, lineWidth: 1)
)

// Text Button (Tertiary)
Button("Dismiss") { }
    .foregroundColor(.accentColor)
    .padding(8)

// Tonal Button (Secondary with emphasis)
Button(action: { }) {
    Text("Maybe")
        .font(.system(size: 16, weight: .semibold))
}
.frame(maxWidth: .infinity)
.frame(height: 48)
.foregroundColor(.accentColor)
.background(Color.accentColor.opacity(0.12))
.cornerRadius(8)
```

**Parity Checklist:**
- [ ] All buttons have minimum 44pt touch target (iOS) / 48dp (Android)
- [ ] Text color sufficient contrast with background (4.5:1)
- [ ] Disabled state uses 38% opacity on both platforms
- [ ] Loading states show spinner without text change
- [ ] Long text wraps or truncates consistently

---

### 2. Text Fields (Input Controls)

#### Material 3 Text Field Variants

| Variant | Android (Compose) | iOS (SwiftUI) | Use Case |
|---------|------------------|---------------|----------|
| **Outlined** | `OutlinedTextField()` | `TextField() { }.textFieldStyle(.roundedBorder)` | Primary input style |
| **Filled** | `TextField()` | `TextField() { }.textFieldStyle(.roundedBorder)` with background | Alternative input |
| **Unfilled** | `BasicTextField()` | `TextField() { }` with custom background | Edge case (rare) |

#### Text Field Implementation

**Android (Jetpack Compose):**
```kotlin
var email by remember { mutableStateOf("") }

OutlinedTextField(
    value = email,
    onValueChange = { email = it },
    label = { Text("Email") },
    modifier = Modifier
        .fillMaxWidth()
        .padding(horizontal = 16.dp)
        .height(56.dp),
    shape = RoundedCornerShape(8.dp),
    colors = OutlinedTextFieldDefaults.colors(
        focusedBorderColor = MaterialTheme.colorScheme.primary,
        unfocusedBorderColor = MaterialTheme.colorScheme.outline
    ),
    singleLine = true,
    keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Email),
    leadingIcon = { Icon(Icons.Default.Email, contentDescription = "Email") },
    trailingIcon = {
        if (email.isNotEmpty()) {
            IconButton(onClick = { email = "" }) {
                Icon(Icons.Default.Clear, contentDescription = "Clear")
            }
        }
    },
    isError = !isValidEmail(email) && email.isNotEmpty()
)

// Error message
if (!isValidEmail(email) && email.isNotEmpty()) {
    Text(
        "Invalid email format",
        color = MaterialTheme.colorScheme.error,
        style = MaterialTheme.typography.bodySmall,
        modifier = Modifier.padding(start = 16.dp, top = 4.dp)
    )
}
```

**iOS (SwiftUI):**
```swift
@State private var email = ""
@State private var emailError: String?

TextField("Email", text: $email)
    .padding(12)
    .background(Color(.systemGray6))
    .cornerRadius(8)
    .overlay(
        RoundedRectangle(cornerRadius: 8)
            .stroke(
                emailError != nil ? Color.red : Color.clear,
                lineWidth: 1
            )
    )
    .keyboardType(.emailAddress)
    .textContentType(.emailAddress)
    .autocorrectionDisabled(true)
    .onChange(of: email) { newValue in
        emailError = validateEmail(newValue) ? nil : "Invalid email format"
    }

if let error = emailError {
    Text(error)
        .font(.caption)
        .foregroundColor(.red)
        .padding(.leading, 12)
}
```

**Parity Checklist:**
- [ ] Label always visible (floats on focus on Android, stays on iOS)
- [ ] Placeholder text uses secondary color
- [ ] Error messages appear below in red (error color)
- [ ] Focus state has blue border (primary color) on both platforms
- [ ] Disabled state is greyed out (tertiary color)
- [ ] Hint text or helper text in small grey text
- [ ] Character count (if applicable) aligns right

---

### 3. Navigation (Screen Transitions)

#### Navigation Patterns

| Pattern | Android | iOS | Trade-offs |
|---------|---------|-----|-----------|
| **Stack Navigation** | NavHost + NavController | NavigationStack | Forward/back linear flow |
| **Tab Navigation** | BottomNavigation + NavGraph | TabView | Parallel navigation in same level |
| **Drawer Navigation** | NavigationDrawer | Menu button + overlay | Side navigation (less common iOS) |
| **Bottom Sheet** | ModalBottomSheet | .sheet(presentationDetents:) | Sliding overlay from bottom |
| **Full Screen Modal** | Dialog / AlertDialog | .fullScreenCover() | Complete screen replacement |

#### Navigation Implementation

**Android (Stack Navigation):**
```kotlin
NavHost(navController = navController, startDestination = "events") {
    composable("events") {
        EventListScreen(
            onEventTap = { eventId ->
                navController.navigate("eventDetail/$eventId")
            },
            onNavigateUp = { navController.navigateUp() }
        )
    }
    composable(
        route = "eventDetail/{eventId}",
        arguments = listOf(
            navArgument("eventId") { type = NavType.StringType }
        )
    ) { backStackEntry ->
        val eventId = backStackEntry.arguments?.getString("eventId") ?: return@composable
        EventDetailScreen(
            eventId = eventId,
            onBackClick = { navController.navigateUp() }
        )
    }
}
```

**iOS (NavigationStack):**
```swift
@State private var navigationPath = NavigationPath()

NavigationStack(path: $navigationPath) {
    EventListScreen(
        onEventTap: { eventId in
            navigationPath.append(eventId)
        }
    )
    .navigationDestination(for: String.self) { eventId in
        EventDetailScreen(eventId: eventId)
    }
}
```

**Parity Checklist:**
- [ ] Back button always returns to previous state
- [ ] Deep links resolve consistently
- [ ] Stack state preserved during transitions
- [ ] Loading indicators show during navigation
- [ ] Error states handled on destination

---

### 4. Lists (Scrollable Content)

#### List Rendering

| Context | Android | iOS | Notes |
|---------|---------|-----|-------|
| **Efficient Lists** | `LazyColumn()` | `List()` or `LazyVStack` | Lazy rendering for performance |
| **Static Lists** | `Column()` in `ScrollView()` | `VStack` in `ScrollView` | Small fixed-size lists |
| **List Items** | `items()` with unique keys | `ForEach(id: \.id)` | Unique IDs essential for diffing |

#### List Implementation

**Android (Jetpack Compose):**
```kotlin
LazyColumn(
    modifier = Modifier
        .fillMaxSize()
        .padding(horizontal = 16.dp),
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
    items(
        items = events,
        key = { event -> event.id }  // Unique key for diffing
    ) { event ->
        EventRow(
            event = event,
            onEventClick = { selectedEventId = event.id },
            modifier = Modifier.animateItem()  // Smooth animations
        )
    }
}
```

**iOS (SwiftUI):**
```swift
List(events, id: \.id) { event in
    EventRow(event: event)
        .onTapGesture {
            selectedEventId = event.id
        }
}
.listStyle(.plain)  // Remove iOS default styling

// Or with LazyVStack for custom behavior:
ScrollView {
    LazyVStack(spacing: 8) {
        ForEach(events, id: \.id) { event in
            EventRow(event: event)
                .onTapGesture {
                    selectedEventId = event.id
                }
        }
    }
    .padding(.horizontal, 16)
}
```

**Parity Checklist:**
- [ ] Items render efficiently (lazy loading)
- [ ] Scroll position preserved on state change
- [ ] Item separators consistent (grey line or spacing)
- [ ] Pull-to-refresh works (if applicable)
- [ ] Empty state message shows when list is empty
- [ ] Loading skeleton shown while fetching

---

### 5. Cards (Container Components)

#### Card Variants

| Variant | Android | iOS | Elevation |
|---------|---------|-----|-----------|
| **Elevated** | `Card(elevation = 4.dp)` | `VStack().shadow(radius: 2)` | 4dp shadow |
| **Filled** | `Surface(containerColor = secondary)` | `VStack().background(Color.gray5)` | No shadow, colored BG |
| **Outlined** | `Card(border = Border(1.dp))` | `VStack().border(1, Color.gray)` | Border only |

#### Card Implementation

**Android (Jetpack Compose):**
```kotlin
Card(
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp),
    shape = RoundedCornerShape(12.dp),
    colors = CardDefaults.cardColors(
        containerColor = MaterialTheme.colorScheme.surface
    ),
    elevation = CardDefaults.cardElevation(defaultElevation = 4.dp)
) {
    Column(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    ) {
        Text(
            "Event Title",
            style = MaterialTheme.typography.headlineSmall,
            color = MaterialTheme.colorScheme.onSurface
        )
        Spacer(modifier = Modifier.height(8.dp))
        Text(
            "Location details",
            style = MaterialTheme.typography.bodyMedium,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )
    }
}
```

**iOS (SwiftUI):**
```swift
VStack(alignment: .leading, spacing: 8) {
    Text("Event Title")
        .font(.headline)
        .foregroundColor(.primary)

    Text("Location details")
        .font(.body)
        .foregroundColor(.secondary)
}
.frame(maxWidth: .infinity, alignment: .leading)
.padding(16)
.background(Color(.systemBackground))
.cornerRadius(12)
.shadow(radius: 2)
```

**Parity Checklist:**
- [ ] Corner radius 12dp/pt on both platforms
- [ ] Shadow elevation 4dp/pt for primary cards
- [ ] Padding 16dp/pt inside card
- [ ] Surface color from theme (not hardcoded)
- [ ] Tap feedback visual (ripple/highlight)

---

### 6. Dialogs (Modal Windows)

#### Dialog Variants

| Type | Android | iOS | Use Case |
|------|---------|-----|----------|
| **Alert Dialog** | `AlertDialog()` | `.alert(isPresented:content:)` | Confirmation, warnings |
| **Modal Dialog** | `Dialog()` | `.sheet()` | Forms, detailed content |
| **Bottom Sheet** | `ModalBottomSheet()` | `.sheet(presentationDetents:)` | Options, menus |

#### Dialog Implementation

**Android (Jetpack Compose):**
```kotlin
if (showDialog) {
    AlertDialog(
        onDismissRequest = { showDialog = false },
        title = { Text("Cancel Event?") },
        text = { Text("Are you sure you want to cancel this event?") },
        confirmButton = {
            Button(
                onClick = {
                    viewModel.cancelEvent()
                    showDialog = false
                },
                colors = ButtonDefaults.buttonColors(
                    containerColor = MaterialTheme.colorScheme.error
                )
            ) {
                Text("Cancel Event")
            }
        },
        dismissButton = {
            TextButton(onClick = { showDialog = false }) {
                Text("Keep Event")
            }
        },
        shape = RoundedCornerShape(12.dp)
    )
}
```

**iOS (SwiftUI):**
```swift
.alert("Cancel Event?", isPresented: $showDialog) {
    Button("Cancel Event", role: .destructive) {
        viewModel.cancelEvent()
    }
    Button("Keep Event", role: .cancel) { }
} message: {
    Text("Are you sure you want to cancel this event?")
}
```

**Bottom Sheet Example:**

**Android:**
```kotlin
if (showSheet) {
    ModalBottomSheet(
        onDismissRequest = { showSheet = false },
        sheetShape = RoundedCornerShape(topStart = 16.dp, topEnd = 16.dp)
    ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp)
        ) {
            Text("Options", style = MaterialTheme.typography.headlineSmall)
            Spacer(modifier = Modifier.height(16.dp))

            option1Button()
            option2Button()
        }
    }
}
```

**iOS:**
```swift
.sheet(isPresented: $showSheet, content: {
    VStack(alignment: .leading, spacing: 16) {
        Text("Options")
            .font(.headline)

        button1
        button2
    }
    .presentationDetents([.medium, .large])
    .presentationDragIndicator(.visible)
})
```

**Parity Checklist:**
- [ ] Dismiss by tapping outside (if not destructive)
- [ ] Keyboard dismissal handled
- [ ] Button order consistent: Primary right, Secondary left
- [ ] Destructive actions in red
- [ ] Title and message text clear and concise

---

### 7. Chips (Compact Selectors)

#### Chip Variants

| Variant | Android | iOS | Use |
|---------|---------|-----|-----|
| **Input Chip** | `InputChip()` | Custom: `Button() { }.border()` | Selection with input |
| **Filter Chip** | `FilterChip()` | Custom: `Button() { }` | Multiple selection |
| **Choice Chip** | `ElevatedFilterChip()` | Custom: `Toggle()` style | Single selection |

#### Chip Implementation

**Android (Jetpack Compose):**
```kotlin
var selectedCategory by remember { mutableStateOf<String?>(null) }

LazyRow(
    horizontalArrangement = Arrangement.spacedBy(8.dp),
    modifier = Modifier.padding(horizontal = 16.dp)
) {
    items(categories) { category ->
        FilterChip(
            selected = selectedCategory == category.id,
            onClick = {
                selectedCategory = if (selectedCategory == category.id) null else category.id
            },
            label = { Text(category.name) },
            shape = RoundedCornerShape(20.dp),
            colors = FilterChipDefaults.filterChipColors(
                selectedContainerColor = MaterialTheme.colorScheme.primaryContainer,
                selectedLabelColor = MaterialTheme.colorScheme.onPrimaryContainer
            )
        )
    }
}
```

**iOS (SwiftUI):**
```swift
@State private var selectedCategory: String?

ScrollView(.horizontal, showsIndicators: false) {
    HStack(spacing: 8) {
        ForEach(categories, id: \.id) { category in
            Button(action: {
                selectedCategory = selectedCategory == category.id ? nil : category.id
            }) {
                Text(category.name)
                    .padding(.horizontal, 16)
                    .padding(.vertical, 8)
            }
            .background(
                selectedCategory == category.id ?
                Color.accentColor.opacity(0.2) : Color(.systemGray6)
            )
            .foregroundColor(
                selectedCategory == category.id ? .accentColor : .primary
            )
            .cornerRadius(20)
        }
    }
    .padding(.horizontal, 16)
}
```

**Parity Checklist:**
- [ ] Selected state shows with primary container color
- [ ] Unselected state is neutral grey
- [ ] Corner radius 20dp/pt (pill-shaped)
- [ ] Padding 16dp/pt horizontal, 8dp/pt vertical
- [ ] Horizontal scroll on both platforms
- [ ] Ripple feedback on tap (Android), opacity change (iOS)

---

### 8. Progress Indicators

#### Progress Types

| Type | Android | iOS | Use Case |
|------|---------|-----|----------|
| **Linear** | `LinearProgressIndicator()` | `ProgressView()` | File upload, form progress |
| **Circular** | `CircularProgressIndicator()` | `ProgressView(value: progress)` | Loading state |
| **Indeterminate** | No value parameter | Default `ProgressView()` | Unknown duration load |

#### Progress Implementation

**Android (Jetpack Compose):**
```kotlin
// Determinate (known progress)
LinearProgressIndicator(
    progress = { uploadProgress },  // 0.0 to 1.0
    modifier = Modifier
        .fillMaxWidth()
        .height(4.dp),
    color = MaterialTheme.colorScheme.primary,
    trackColor = MaterialTheme.colorScheme.surfaceVariant
)

// Indeterminate (unknown duration)
CircularProgressIndicator(
    modifier = Modifier.size(40.dp),
    color = MaterialTheme.colorScheme.primary,
    trackColor = MaterialTheme.colorScheme.surfaceVariant,
    strokeWidth = 4.dp
)
```

**iOS (SwiftUI):**
```swift
// Determinate
ProgressView(value: uploadProgress)
    .frame(height: 4)
    .tint(.accentColor)

// Indeterminate
ProgressView()
    .scaleEffect(1.5, anchor: .center)
    .tint(.accentColor)
```

**Parity Checklist:**
- [ ] Progress bar height 4dp/pt
- [ ] Color is primary on both platforms
- [ ] Determinate shows filled portion
- [ ] Indeterminate spins/animates continuously
- [ ] Track color is surface variant

---

## Phase 2: Decision Tree - Platform-Specific vs. Shared

```
┌─ Start: New Component ─┐
│                        │
├─ Is it a LAYOUT pattern?
│  ├─ YES → Use platform native (NavigationStack for iOS, NavController for Android)
│  └─ NO  → Continue
│
├─ Is it a FORM INPUT?
│  ├─ YES → Match Material 3 structure (label, error, hint on both)
│  └─ NO  → Continue
│
├─ Is it a BUTTON or INTERACTIVE?
│  ├─ YES → Use Material 3 color tokens + platform ripple/highlight
│  └─ NO  → Continue
│
├─ Is it a CONTAINER (Card, Surface)?
│  ├─ YES → Match corner radius (12dp), shadow (4dp), padding (16dp)
│  └─ NO  → Continue
│
├─ Is it TYPOGRAPHY or TEXT?
│  ├─ YES → Use Material 3 type scale on both
│  └─ NO  → Continue
│
├─ Is it UNIQUE to one platform?
│  ├─ YES → Implement platform-native version (e.g., iOS notch, Android back button)
│  └─ NO  → Implement shared design
│
└─ Result: Implementation path determined
```

### Decision Matrix

| Scenario | Decision | Rationale |
|----------|----------|-----------|
| New button in event detail | Use Material 3 filled/outlined on both | Consistent interaction model |
| Navigation between screens | Android: NavController, iOS: NavigationStack | Platform idioms matter for navigation |
| Form with email input | Material 3 outlined textfield on both | Users expect similar form patterns |
| Loading indicator | Use primary color on both, platform progress component | Consistent visual state |
| Chip filters | Custom Material 3 style on both | Not native in iOS, need custom |
| Bottom sheet menu | ModalBottomSheet (Android), .sheet() (iOS) | Platform native APIs are better |
| Card in list | Same corner radius, elevation, padding on both | Visual consistency across platforms |
| Error message color | Use Material 3 error color on both | Color token consistency |

---

## Phase 3: Accessibility Parity Checklist

### Semantic Labels & Descriptions

**Android (Jetpack Compose):**
```kotlin
Button(
    onClick = { viewModel.joinEvent() },
    modifier = Modifier.semantics {
        contentDescription = "Join event 'D&D Campaign' in Indianapolis on Friday"
        // Also set for icon buttons
    }
) {
    Text("Join")
}

// Image without visible text
Image(
    painter = painterResource(id = R.drawable.ic_favorite),
    contentDescription = "Add to favorites"  // Always required
)

// List items should announce index
LazyColumn {
    items(events.size) { index ->
        EventRow(
            event = events[index],
            modifier = Modifier.semantics {
                contentDescription = "Event ${index + 1} of ${events.size}: ${events[index].title}"
            }
        )
    }
}
```

**iOS (SwiftUI):**
```swift
Button(action: { viewModel.joinEvent() }) {
    Text("Join")
}
.accessibilityLabel("Join event")
.accessibilityHint("D&D Campaign in Indianapolis on Friday")

// Image without visible text
Image("favorite-icon")
    .accessibilityLabel("Add to favorites")

// List items
List(events, id: \.id) { event in
    EventRow(event: event)
        .accessibilityLabel(event.title)
        .accessibilityHint("Category: \(event.category), Players: \(event.currentPlayers)/\(event.maxPlayers)")
}
```

### Color Contrast Requirements

**WCAG AA Standard: 4.5:1 minimum**

| Color Pair | Example | Contrast Ratio | Status |
|-----------|---------|---|---|
| Primary text on background | Black text on white | 21:1 | PASS |
| Secondary text on background | Grey #666 on white | 7.2:1 | PASS |
| Button text on primary | White text on #6750A4 | 7.5:1 | PASS |
| Disabled text | Grey #999 on white | 3.2:1 | FAIL - needs adjustment |
| Error message | Red #B3261E on white | 5.5:1 | PASS |

**Tools for validation:**
- Android: Use `androidx.compose.ui.semantics` with accessibility checker
- iOS: Xcode Accessibility Inspector (Color Contrast Analyzer)

### State Announcements

**Android:**
```kotlin
var isLoading by remember { mutableStateOf(false) }

Button(
    onClick = { isLoading = true; loadData() },
    enabled = !isLoading,
    modifier = Modifier.semantics {
        // Screen readers will announce loading state
        liveRegion = LiveRegionMode.Assertive
    }
) {
    if (isLoading) {
        Row {
            CircularProgressIndicator(
                modifier = Modifier.size(16.dp),
                strokeWidth = 2.dp
            )
            Spacer(modifier = Modifier.width(8.dp))
            Text("Loading...")
        }
    } else {
        Text("Load Events")
    }
}
```

**iOS:**
```swift
@State private var isLoading = false

Button(action: {
    isLoading = true
    loadData()
}) {
    if isLoading {
        HStack {
            ProgressView()
            Text("Loading...")
        }
    } else {
        Text("Load Events")
    }
}
.disabled(isLoading)
.accessibilityElement(children: .combine)
.accessibilityLabel(isLoading ? "Loading events" : "Load events")
.accessibilityAddTraitIfNeeded(.isButton)
```

### Touch Target & Hit Area

**Minimum 44pt (iOS) / 48dp (Android)**

**Android:**
```kotlin
Button(
    onClick = { /* */ },
    modifier = Modifier
        .size(
            width = 64.dp,  // At least 48dp
            height = 48.dp  // At least 48dp
        )
)
```

**iOS:**
```swift
Button(action: { }) {
    Text("Action")
}
.frame(minWidth: 44, minHeight: 44)  // At least 44pt x 44pt
```

### Focus & Keyboard Navigation

**Android:**
```kotlin
OutlinedTextField(
    value = email,
    onValueChange = { email = it },
    label = { Text("Email") },
    modifier = Modifier.focusRequester(focusRequester),
    keyboardOptions = KeyboardOptions(
        keyboardType = KeyboardType.Email,
        imeAction = ImeAction.Next
    ),
    keyboardActions = KeyboardActions(
        onNext = { focusRequester.moveFocus(FocusDirection.Down) }
    )
)
```

**iOS:**
```swift
TextField("Email", text: $email)
    .textContentType(.emailAddress)
    .keyboardType(.emailAddress)
    .submitLabel(.next)
    .onSubmit {
        // Move focus to next field
        focusedField = .password
    }
```

### Accessibility Parity Checklist

- [ ] All buttons have labels (visible or `contentDescription`/`accessibilityLabel`)
- [ ] Images have alt text (icon buttons especially)
- [ ] Color isn't sole means of conveying info (use text + color for states)
- [ ] Text has 4.5:1 contrast minimum (WCAG AA)
- [ ] Touch targets are 44pt+ (iOS) / 48dp+ (Android)
- [ ] Form labels clearly associated with inputs
- [ ] Error messages announced to screen readers
- [ ] Keyboard navigation works (Tab key on Android with hardware keyboard)
- [ ] List items announce position ("3 of 10")
- [ ] Loading/state changes announced via live region

---

## Phase 4: Implementation Patterns

### Pattern 1: Shared ViewModel with Platform Views

**Shared (Kotlin):**
```kotlin
// Can be shared via Kotlin Multiplatform (common module)
class EventListViewModel {
    private val _events = MutableStateFlow<List<Event>>(emptyList())
    val events: StateFlow<List<Event>> = _events.asStateFlow()

    fun loadEvents() {
        // Implementation
    }

    fun selectEvent(id: String) {
        // Selection logic
    }
}
```

**Android:**
```kotlin
@Composable
fun EventListScreen() {
    val viewModel = hiltViewModel<EventListViewModel>()
    val events by viewModel.events.collectAsState()

    LazyColumn {
        items(events, key = { it.id }) { event ->
            EventRow(event = event) {
                viewModel.selectEvent(event.id)
            }
        }
    }
}
```

**iOS:**
```swift
class EventListViewModel: ObservableObject {
    @Published var events: [Event] = []

    func loadEvents() {
        // Implementation (same logic as shared)
    }

    func selectEvent(id: String) {
        // Selection logic
    }
}

struct EventListScreen: View {
    @StateObject var viewModel = EventListViewModel()

    var body: some View {
        List(viewModel.events, id: \.id) { event in
            EventRow(event: event)
                .onTapGesture {
                    viewModel.selectEvent(id: event.id)
                }
        }
    }
}
```

### Pattern 2: Conditional Platform-Specific UI

**Android:**
```kotlin
@Composable
fun EventDetailScreen(eventId: String) {
    // Android-specific: Use back button
    BackHandler(enabled = true) {
        navController.navigateUp()
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Event Details") },
                navigationIcon = {
                    IconButton(onClick = { navController.navigateUp() }) {
                        Icon(Icons.AutoMirrored.Filled.ArrowBack, "Back")
                    }
                }
            )
        }
    ) { padding ->
        // Content
    }
}
```

**iOS:**
```swift
struct EventDetailScreen: View {
    @Environment(\.dismiss) var dismiss

    var body: some View {
        NavigationView {
            content
                .navigationBarTitleDisplayMode(.inline)
                .toolbar {
                    ToolbarItem(placement: .navigationBarLeading) {
                        Button(action: { dismiss() }) {
                            Image(systemName: "chevron.left")
                        }
                    }
                }
        }
    }

    var content: some View {
        // Same content as Android
        Text("Event Details")
    }
}
```

### Pattern 3: Shared Design Tokens (Material 3)

**Create a shared color palette:**

**Android:**
```kotlin
object PartyOnColors {
    val primary = Color(0xFF6750A4)
    val onPrimary = Color(0xFFFFFFFF)
    val primaryContainer = Color(0xFFEADDFF)
    val onPrimaryContainer = Color(0xFF21005E)

    val secondary = Color(0xFF625B71)
    val onSecondary = Color(0xFFFFFFFF)
    val secondaryContainer = Color(0xFFE8DEF8)
    val onSecondaryContainer = Color(0xFF1E192B)

    val error = Color(0xFFB3261E)
    val onError = Color(0xFFFFFFFF)

    val surface = Color(0xFFFFFBFE)
    val onSurface = Color(0xFF1C1B1F)
    val outline = Color(0xFF79747E)
}
```

**iOS:**
```swift
extension Color {
    // Primary
    static let primaryMaterial = Color(red: 0.404, green: 0.318, blue: 0.643)
    static let onPrimaryMaterial = Color.white
    static let primaryContainerMaterial = Color(red: 0.920, green: 0.867, blue: 1.0)

    // Secondary
    static let secondaryMaterial = Color(red: 0.388, green: 0.357, blue: 0.443)
    static let onSecondaryMaterial = Color.white

    // Error
    static let errorMaterial = Color(red: 0.702, green: 0.149, blue: 0.118)

    // Surface
    static let surfaceMaterial = Color(red: 0.998, green: 0.984, blue: 0.998)
    static let onSurfaceMaterial = Color(red: 0.110, green: 0.107, blue: 0.122)
}
```

---

## Phase 5: Testing Cross-Platform Components

### Component Testing Checklist

**Visual Regression Testing:**
- [ ] Screenshot on multiple device sizes
- [ ] Test light and dark modes
- [ ] Verify scaling on different screen densities
- [ ] Check corner radius, shadows, spacing

**Interaction Testing:**
- [ ] Tap/click targets work correctly
- [ ] Disabled states prevent interaction
- [ ] Loading states show/hide appropriately
- [ ] Error messages appear/disappear

**State Management Testing:**
- [ ] Component state syncs with ViewModel
- [ ] State persists across navigation
- [ ] Memory doesn't leak on component destruction
- [ ] Concurrent state updates don't cause crashes

**Accessibility Testing:**
- [ ] Screen readers can read all content
- [ ] Keyboard navigation works (Android with hardware keyboard)
- [ ] Color contrast passes WCAG AA
- [ ] Focus order is logical

### Automated Testing Example

**Android (Instrumentation Test):**
```kotlin
@RunWith(AndroidJUnit4::class)
class EventListScreenTest {
    @get:Rule
    val composeRule = createComposeRule()

    @Test
    fun eventListRendersWithCorrectContent() {
        composeRule.setContent {
            EventListScreen()
        }

        composeRule
            .onNodeWithText("D&D Campaign")
            .assertIsDisplayed()
            .assertHasClickAction()
    }

    @Test
    fun eventListScrollsSmooth() {
        composeRule.setContent {
            EventListScreen()
        }

        composeRule
            .onNodeWithTag("eventList")
            .performScrollToIndex(10)
            .assertIsDisplayed()
    }
}
```

**iOS (XCUITest):**
```swift
class EventListScreenTest: XCTestCase {
    func testEventListRendersWithContent() {
        let app = XCUIApplication()
        app.launch()

        let eventTitle = app.staticTexts["D&D Campaign"]
        XCTAssertTrue(eventTitle.exists)
        XCTAssertTrue(eventTitle.isHittable)
    }

    func testEventListScrolls() {
        let app = XCUIApplication()
        app.launch()

        let listView = app.scrollViews.element
        listView.scroll(byDeltaX: 0, deltaY: 500)

        // Verify content updated after scroll
    }
}
```

---

## Quick Reference: Common Patterns

### Material 3 Color Tokens (Both Platforms)

```
Primary: #6750A4
OnPrimary: #FFFFFF
PrimaryContainer: #EADDFF
OnPrimaryContainer: #21005E

Secondary: #625B71
OnSecondary: #FFFFFF
SecondaryContainer: #E8DEF8
OnSecondaryContainer: #1E192B

Error: #B3261E
OnError: #FFFFFF

Surface: #FFFBFE
OnSurface: #1C1B1F
Outline: #79747E
```

### Spacing Scale (Both Platforms)

```
0dp/pt, 2dp/pt, 4dp/pt, 8dp/pt, 12dp/pt, 16dp/pt, 24dp/pt, 32dp/pt, 48dp/pt, 64dp/pt
```

### Corner Radii

```
Small: 4dp/pt
Medium: 8dp/pt
Large: 12dp/pt
Extra Large: 28dp/pt
```

### Typography (Material 3 Type Scale)

| Style | Size | Weight |
|-------|------|--------|
| displayLarge | 57sp | Bold |
| displayMedium | 45sp | Bold |
| displaySmall | 36sp | Bold |
| headlineLarge | 32sp | Bold |
| headlineMedium | 28sp | Bold |
| headlineSmall | 24sp | Bold |
| titleLarge | 22sp | Bold |
| titleMedium | 16sp | SemiBold |
| titleSmall | 14sp | SemiBold |
| bodyLarge | 16sp | Regular |
| bodyMedium | 14sp | Regular |
| bodySmall | 12sp | Regular |
| labelLarge | 14sp | SemiBold |
| labelMedium | 12sp | SemiBold |
| labelSmall | 11sp | SemiBold |

---

## Summary: Implementation Steps

1. **Choose component type** (button, text field, card, etc.)
2. **Reference the Matrix** for Material 3 specs (color, size, corner radius)
3. **Follow the Decision Tree** to determine if shared or platform-specific
4. **Implement on Android** using Jetpack Compose
5. **Implement on iOS** using SwiftUI
6. **Match Material 3 tokens** (colors, spacing, typography)
7. **Verify Accessibility** (labels, contrast, touch targets)
8. **Test on multiple devices** (size variations, light/dark mode)
9. **Update this guide** if new patterns emerge

---

## Resources

Based on 2025-2026 best practices from:
- [Material Design 3 Components](https://m3.material.io/components)
- [SwiftUI Documentation - Apple Developer](https://developer.apple.com/swiftui/)
- [Using Material Design Components in SwiftUI](https://jeremyleenz.medium.com/using-material-design-components-in-swiftui-45f745bb27c9)
- [Compose Multiplatform for iOS](https://medium.com/@rushabhprajapati20/compose-multiplatform-for-ios-is-now-stable-bf4e2fc35596)
- [Cross-Platform Design Systems - Lyft](https://www.designsystems.com/a-system-built-on-parity-how-to-treat-all-of-your-users-equally/)
- [Multi-Platform Design Systems](https://dbanks.design/blog/multi-platform/)

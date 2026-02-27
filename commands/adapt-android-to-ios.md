---
name: adapt-android-to-ios
description: Comprehensive guide for porting Jetpack Compose (Android) code to SwiftUI (iOS) with component mapping, state management translation, and code transformation examples
args:
  - name: compose_code
    type: string
    description: Kotlin/Jetpack Compose code to port to SwiftUI
    required: true
  - name: focus_area
    type: string
    enum: [components, state, modifiers, layout, gestures, navigation, theming, all]
    description: Specific porting area to focus on (default: all)
    required: false
  - name: output_format
    type: string
    enum: [checklist, code, guide, comparison]
    description: Format for the output (default: code)
    required: false
migration_phases:
  - preparation
  - component_mapping
  - state_management
  - layout_translation
  - gesture_handling
  - theming
  - testing
---

# Android Compose to iOS SwiftUI Porting Guide

Transform Jetpack Compose code into idiomatic SwiftUI with this comprehensive cross-platform porting reference. Based on 2025-2026 best practices for declarative UI migration.

## Overview

Jetpack Compose and SwiftUI share the same declarative UI philosophy (**UI = f(state)**), but differ significantly in:
- **Syntax**: Function composition vs. view structs
- **State management**: `remember`/`@State` vs. property wrappers
- **Modifiers**: Parameter-based vs. method chains
- **Layout system**: `weight()` vs. `GeometryReader`/`.frame(maxWidth: .infinity)`

This guide maps each concept with working code examples.

---

## Phase 1: Preparation & Analysis

Before porting, analyze your Compose codebase:

### Quick Assessment Checklist

{{#if (or (eq focus_area "all") (not focus_area))}}

- [ ] **Identify Composables**: List all `@Composable` functions (becomes `View` structs)
- [ ] **Map ViewModels**: Compose ViewModels → SwiftUI `@ObservedObject`/`@StateObject`
- [ ] **Trace State Flow**: Find `remember`, `State`, `MutableState` → converts to `@State`, `@Binding`, property wrappers
- [ ] **Document Navigation**: NavController/NavHost → `NavigationStack` or `NavigationLink`
- [ ] **Check Material Design 3**: Material 3 colors → SwiftUI semantic + custom environment keys
- [ ] **List Gestures**: Analyze all `.clickable()`, `.swipeGestures()` → swiftUI equivalents
- [ ] **Audit Modifiers**: Collect modifier chains for systematic translation
- [ ] **Size Hierarchy**: Document layout containers (Row/Column, weight distribution)

### File Structure Template

```kotlin
// Android Compose
MyScreen.kt (root @Composable)
├── SubComponent1.kt (@Composable)
├── SubComponent2.kt (@Composable)
└── ViewModel.kt (shared state, side effects)
```

Becomes in SwiftUI:

```swift
// iOS SwiftUI
MyScreen.swift (View struct)
├── SubComponent1.swift (View struct)
├── SubComponent2.swift (View struct)
└── ViewModel.swift (@ObservedObject, side effects)
```

{{/if}}

---

## Phase 2: Component Mapping (Compose Composables → SwiftUI Views)

### Basic Composable Structure

**Jetpack Compose:**
```kotlin
@Composable
fun EventCard(
    event: Event,
    onJoinClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxWidth()
            .padding(16.dp)
            .background(MaterialTheme.colorScheme.surface)
            .clickable(onClick = onJoinClick)
    ) {
        Text(event.title)
        Text(event.location)
    }
}
```

**SwiftUI Equivalent:**
```swift
struct EventCard: View {
    let event: Event
    var onJoinClick: () -> Void

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            Text(event.title)
            Text(event.location)
        }
        .frame(maxWidth: .infinity, alignment: .leading)
        .padding(16)
        .background(Color(.systemBackground))
        .onTapGesture(perform: onJoinClick)
    }
}
```

### Compose Component Mapping Table

| Compose | SwiftUI | Notes |
|---------|---------|-------|
| `@Composable fun` | `struct: View` | Every composable becomes a View struct with `var body: some View` |
| `Column()` | `VStack()` | Vertical layout container |
| `Row()` | `HStack()` | Horizontal layout container |
| `Box()` | `ZStack()` | Layering/overlay container |
| `Spacer()` | `.frame(height: 16)` or `Spacer()` | Space filler |
| `Text()` | `Text()` | Text display (similar API) |
| `Button()` | `Button()` | Interactive button |
| `Icon()` | `Image()` | Icon/image display |
| `LazyColumn()` | `List()` or `LazyVStack(in: ScrollView)` | Efficient lists |
| `TextField()` | `TextField()` | Text input |
| `Checkbox` | `Toggle()` | Boolean control |
| `Surface()` | `.background()` + `.cornerRadius()` | Card-like container |
| `Card()` | `VStack().background(...).cornerRadius(...)` | Elevated surface |

---

## Phase 3: State Management Translation

### State Hoisting Pattern

**Key Principle**: Both Compose and SwiftUI follow **unidirectional data flow**. State lives in the highest common parent and flows down as immutable data.

**Jetpack Compose State Management:**
```kotlin
@Composable
fun EventListScreen(
    viewModel: EventListViewModel = hiltViewModel()
) {
    val events by viewModel.events.collectAsState()
    val selectedEventId by viewModel.selectedEventId.collectAsState()

    Column {
        EventList(
            events = events,
            selectedEventId = selectedEventId,
            onEventSelect = { viewModel.selectEvent(it.id) }
        )
        if (selectedEventId != null) {
            EventDetail(eventId = selectedEventId)
        }
    }
}

class EventListViewModel : ViewModel() {
    private val _selectedEventId = MutableStateFlow<String?>(null)
    val selectedEventId: StateFlow<String?> = _selectedEventId.asStateFlow()

    fun selectEvent(id: String) {
        _selectedEventId.value = id
    }
}
```

**SwiftUI Equivalent:**
```swift
struct EventListScreen: View {
    @StateObject var viewModel = EventListViewModel()

    var body: some View {
        VStack {
            EventList(
                events: viewModel.events,
                selectedEventId: viewModel.selectedEventId,
                onEventSelect: { viewModel.selectEvent(id: $0.id) }
            )
            if let eventId = viewModel.selectedEventId {
                EventDetail(eventId: eventId)
            }
        }
    }
}

class EventListViewModel: ObservableObject {
    @Published var selectedEventId: String?
    @Published var events: [Event] = []

    func selectEvent(id: String) {
        selectedEventId = id
    }
}
```

### State Property Wrapper Mapping

| Compose | SwiftUI | Use Case | Lifetime |
|---------|---------|----------|----------|
| `remember { mutableStateOf(value) }` | `@State private var` | Local UI state owned by one view | View lifetime |
| `MutableState` passed to children | `@Binding` parameter | State owned by parent, edited by child | Parent's state |
| `StateFlow` in ViewModel | `@Published` property | Shared observable state | ViewModel lifetime |
| `remember(key) { ... }` memoization | Value stored in `@State` | Cache computed values | View lifetime |
| `derivedStateOf()` | `@State` + `.onChange()` | Computed state from other state | View lifetime |
| `rememberSaveable()` | Add to `UserDefaults` + `@AppStorage` | Persist across app restarts | App lifetime |

**Detailed Translations:**

#### 1. Local Mutable State

**Compose:**
```kotlin
@Composable
fun LoginForm() {
    var email by remember { mutableStateOf("") }
    var password by remember { mutableStateOf("") }

    TextField(value = email, onValueChange = { email = it })
    TextField(value = password, onValueChange = { password = it })
    Button(onClick = { submitLogin(email, password) })
}
```

**SwiftUI:**
```swift
struct LoginForm: View {
    @State private var email = ""
    @State private var password = ""

    var body: some View {
        VStack {
            TextField("Email", text: $email)
            SecureField("Password", text: $password)
            Button("Login") {
                submitLogin(email: email, password: password)
            }
        }
    }
}
```

#### 2. Parent-to-Child State Binding

**Compose:**
```kotlin
@Composable
fun ParentScreen() {
    var favoriteEventId by remember { mutableStateOf<String?>(null) }

    EventCard(
        event = event,
        isFavorite = favoriteEventId == event.id,
        onFavoriteToggle = { isNowFavorite ->
            if (isNowFavorite) favoriteEventId = event.id
            else favoriteEventId = null
        }
    )
}

@Composable
fun EventCard(
    event: Event,
    isFavorite: Boolean,
    onFavoriteToggle: (Boolean) -> Unit
) {
    Icon(
        Icons.Default.Favorite,
        tint = if (isFavorite) Color.Red else Color.Gray,
        modifier = Modifier.clickable {
            onFavoriteToggle(!isFavorite)
        }
    )
}
```

**SwiftUI:**
```swift
struct ParentScreen: View {
    @State private var favoriteEventId: String?

    var body: some View {
        EventCard(
            event: event,
            isFavorite: .constant(favoriteEventId == event.id),
            onFavoriteToggle: { isNowFavorite in
                favoriteEventId = isNowFavorite ? event.id : nil
            }
        )
    }
}

struct EventCard: View {
    let event: Event
    @Binding var isFavorite: Bool
    var onFavoriteToggle: (Bool) -> Void

    var body: some View {
        Image(systemName: isFavorite ? "heart.fill" : "heart")
            .foregroundColor(isFavorite ? .red : .gray)
            .onTapGesture {
                onFavoriteToggle(!isFavorite)
            }
    }
}
```

#### 3. Observable ViewModel Pattern

**Compose:**
```kotlin
class EventDetailViewModel(val eventId: String) : ViewModel() {
    private val _event = MutableStateFlow<Event?>(null)
    val event: StateFlow<Event?> = _event.asStateFlow()

    private val _isLoading = MutableStateFlow(false)
    val isLoading: StateFlow<Boolean> = _isLoading.asStateFlow()

    init {
        loadEvent()
    }

    private fun loadEvent() {
        viewModelScope.launch {
            _isLoading.value = true
            try {
                _event.value = eventRepository.getEvent(eventId)
            } finally {
                _isLoading.value = false
            }
        }
    }
}

@Composable
fun EventDetailScreen(eventId: String) {
    val viewModel = hiltViewModel<EventDetailViewModel>(
        creationCallback = { EventDetailViewModel(eventId) }
    )
    val event by viewModel.event.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()

    when {
        isLoading -> CircularProgressIndicator()
        event != null -> EventDetailContent(event = event!!)
        else -> Text("Event not found")
    }
}
```

**SwiftUI:**
```swift
class EventDetailViewModel: ObservableObject {
    @Published var event: Event?
    @Published var isLoading = false

    let eventId: String

    init(eventId: String) {
        self.eventId = eventId
        loadEvent()
    }

    private func loadEvent() {
        isLoading = true
        Task {
            do {
                event = try await EventRepository.shared.getEvent(eventId)
            } catch {
                print("Error loading event: \(error)")
            }
            isLoading = false
        }
    }
}

struct EventDetailScreen: View {
    let eventId: String
    @StateObject private var viewModel: EventDetailViewModel

    init(eventId: String) {
        self.eventId = eventId
        _viewModel = StateObject(wrappedValue: EventDetailViewModel(eventId: eventId))
    }

    var body: some View {
        if viewModel.isLoading {
            ProgressView()
        } else if let event = viewModel.event {
            EventDetailContent(event: event)
        } else {
            Text("Event not found")
        }
    }
}
```

---

## Phase 4: Modifier Chain Translation

### Modifier Execution Order (Critical)

**Jetpack Compose**: Modifiers are applied in order from first to last in the parameter list.

**SwiftUI**: Modifiers are applied in a **stack from bottom to top** of the chain. This matters!

```swift
// These are DIFFERENT:
Text("Hello")
    .background(Color.blue)      // 1. Background color
    .padding(16)                 // 2. Then add padding

Text("Hello")
    .padding(16)                 // 1. Padding first
    .background(Color.blue)      // 2. Then background (only behind text, not padding!)
```

### Comprehensive Modifier Mapping

| Compose Modifier | SwiftUI Equivalent | Example |
|-----------------|-------------------|---------|
| `.padding(16.dp)` | `.padding(16)` | Text().padding(16) |
| `.fillMaxWidth()` | `.frame(maxWidth: .infinity)` | VStack().frame(maxWidth: .infinity) |
| `.fillMaxHeight()` | `.frame(maxHeight: .infinity)` | VStack().frame(maxHeight: .infinity) |
| `.size(100.dp)` | `.frame(width: 100, height: 100)` | Image().frame(width: 100, height: 100) |
| `.width(100.dp)` | `.frame(width: 100)` | Text().frame(width: 100) |
| `.height(100.dp)` | `.frame(height: 100)` | Text().frame(height: 100) |
| `.background(Color.Blue)` | `.background(Color.blue)` | Text().background(Color.blue) |
| `.clip(RoundedCornerShape(8.dp))` | `.cornerRadius(8)` | VStack().cornerRadius(8) |
| `.border(Color.Red, 2.dp)` | `.border(2, Color.red)` | VStack().border(2, Color.red) |
| `.shadow(...)` | `.shadow(...)` | VStack().shadow(radius: 4) |
| `.opacity(0.5f)` | `.opacity(0.5)` | Text().opacity(0.5) |
| `.rotate(45.degrees)` | `.rotationEffect(.degrees(45))` | Image().rotationEffect(.degrees(45)) |
| `.scale(1.2f)` | `.scaleEffect(1.2)` | Text().scaleEffect(1.2) |
| `.offset(10.dp, 20.dp)` | `.offset(x: 10, y: 20)` | Text().offset(x: 10, y: 20) |
| `.clickable {}` | `.onTapGesture {}` | Text().onTapGesture { } |
| `.enabled()` | `.disabled()` | Button().disabled(false) |
| `.alpha(0.8f)` | `.opacity(0.8)` | Text().opacity(0.8) |
| `.testTag()` | `.accessibilityIdentifier()` | Text().accessibilityIdentifier("id") |

### Modifier Chain Examples

**Compose (padding order matters):**
```kotlin
Text("Event")
    .padding(horizontal = 16.dp, vertical = 8.dp)
    .background(MaterialTheme.colorScheme.surfaceVariant)
    .padding(8.dp)  // Additional padding around background
    .clip(RoundedCornerShape(8.dp))
    .clickable { /* ... */ }
```

**SwiftUI (stack order matters differently):**
```swift
Text("Event")
    .padding(.horizontal, 16)
    .padding(.vertical, 8)
    .background(Color(.systemGray5))
    .padding(8)
    .cornerRadius(8)
    .onTapGesture { /* ... */ }
```

**Complex Modifier Chain Example:**

**Compose:**
```kotlin
Button(onClick = { /* */ }) {
    Text("Join Event")
}
.modifier(
    Modifier
        .fillMaxWidth()
        .height(48.dp)
        .background(MaterialTheme.colorScheme.primary)
        .clip(RoundedCornerShape(8.dp))
        .shadow(elevation = 4.dp)
)
```

**SwiftUI:**
```swift
Button(action: { /* */ }) {
    Text("Join Event")
        .frame(maxWidth: .infinity)
        .frame(height: 48)
}
.background(Color.accentColor)
.cornerRadius(8)
.shadow(radius: 2)
```

---

## Phase 5: Layout System Translation

### Size Weight Distribution

**Key Difference**: Compose has explicit `weight()` modifier. SwiftUI uses `GeometryReader` or `.frame(maxWidth: .infinity)` for proportional layouts.

**Compose Weight System:**
```kotlin
Row(modifier = Modifier.fillMaxWidth()) {
    Text("Left", modifier = Modifier.weight(1f))
    Spacer(modifier = Modifier.width(8.dp))
    Text("Right", modifier = Modifier.weight(2f))  // 2x wider than left
}
```

**SwiftUI Equivalent (Method 1 - Manual):**
```swift
HStack(spacing: 8) {
    Text("Left")
        .frame(maxWidth: .infinity)
    Text("Right")
        .frame(maxWidth: .infinity)
}
// Problem: Both get equal space, not 1:2 ratio
```

**SwiftUI Equivalent (Method 2 - GeometryReader for Proportional Layout):**
```swift
GeometryReader { geometry in
    HStack(spacing: 8) {
        Text("Left")
            .frame(width: geometry.size.width * 1/3)  // 1/3 of space
        Text("Right")
            .frame(width: geometry.size.width * 2/3)  // 2/3 of space
    }
}
.frame(height: 40)  // Specify height for GeometryReader
```

**SwiftUI Equivalent (Method 3 - If weights are all equal):**
```swift
HStack(spacing: 8) {
    Text("Left")
        .frame(maxWidth: .infinity)
    Text("Right")
        .frame(maxWidth: .infinity)
}
```

### Alignment Properties

| Compose | SwiftUI | Notes |
|---------|---------|-------|
| `Alignment.Center` | `.center` | Center alignment |
| `Alignment.Start` | `.leading` | Left/start alignment |
| `Alignment.End` | `.trailing` | Right/end alignment |
| `Alignment.Top` | `.top` | Top alignment |
| `Alignment.Bottom` | `.bottom` | Bottom alignment |

**Compose Layout Example:**
```kotlin
Column(
    modifier = Modifier.fillMaxWidth(),
    horizontalAlignment = Alignment.CenterHorizontally,
    verticalArrangement = Arrangement.SpaceEvenly
) {
    Text("Item 1")
    Text("Item 2")
    Text("Item 3")
}
```

**SwiftUI Equivalent:**
```swift
VStack(alignment: .center, spacing: 0) {
    Text("Item 1")
    Spacer()
    Text("Item 2")
    Spacer()
    Text("Item 3")
}
.frame(maxWidth: .infinity)
```

---

## Phase 6: Gesture Handling

### Gesture Translation

| Compose | SwiftUI | Notes |
|---------|---------|-------|
| `.clickable {}` | `.onTapGesture {}` | Single tap |
| `.pointerInput { detectTapGestures {} }` | `.onLongPressGesture {}` | Long press |
| `.swipeGestures()` | `.gesture(DragGesture())` | Swipe detection |
| `.indication()` | `.buttonStyle(.plain)` | Visual feedback |
| `GestureScope.onPress()` | `.onLongPressGesture(onPress:)` | Press feedback |

### Gesture Examples

**Compose Single Tap:**
```kotlin
Text("Tap me")
    .clickable(
        indication = ripple(),
        interactionSource = remember { MutableInteractionSource() }
    ) {
        println("Tapped!")
    }
```

**SwiftUI Single Tap:**
```swift
Text("Tap me")
    .onTapGesture {
        print("Tapped!")
    }
```

**Compose Long Press:**
```kotlin
var isPressed by remember { mutableStateOf(false) }

Text("Hold me")
    .pointerInput(Unit) {
        detectTapGestures(
            onPress = {
                isPressed = true
                tryAwaitRelease()
                isPressed = false
            }
        )
    }
    .background(if (isPressed) Color.Gray else Color.White)
```

**SwiftUI Long Press:**
```swift
@State private var isPressed = false

Text("Hold me")
    .onLongPressGesture(minimumDuration: 0.5) {
        // Long press ended
    } onPressingChanged: { isPressing in
        isPressed = isPressing
    }
    .background(isPressed ? Color.gray : Color.white)
```

**Compose Swipe Gesture:**
```kotlin
var offset by remember { mutableStateOf(0f) }

Text("Swipe me")
    .pointerInput(Unit) {
        detectHorizontalDragGestures { change, dragAmount in
            offset += dragAmount
        }
    }
    .offset(x = offset.dp)
```

**SwiftUI Swipe Gesture:**
```swift
@State private var offset: CGFloat = 0

Text("Swipe me")
    .gesture(
        DragGesture()
            .onChanged { value in
                offset = value.translation.width
            }
            .onEnded { _ in
                withAnimation {
                    offset = 0
                }
            }
    )
    .offset(x: offset)
```

**SwiftUI Simultaneous Gestures:**
```swift
Text("Multi-gesture")
    .gesture(
        TapGesture()
            .onEnded {
                print("Tapped")
            }
    )
    .simultaneousGesture(
        DragGesture()
            .onChanged { value in
                print("Dragging: \(value.translation)")
            }
    )
```

---

## Phase 7: Navigation Translation

### Simple Navigation

**Compose with NavController:**
```kotlin
NavHost(navController = navController, startDestination = "events") {
    composable("events") {
        EventListScreen(
            onEventSelect = { eventId ->
                navController.navigate("event/$eventId")
            }
        )
    }
    composable("event/{eventId}") { backStackEntry ->
        val eventId = backStackEntry.arguments?.getString("eventId")
        EventDetailScreen(eventId = eventId!!)
    }
}
```

**SwiftUI with NavigationStack (iOS 16+):**
```swift
@State private var navigationPath = NavigationPath()

NavigationStack(path: $navigationPath) {
    EventListScreen(
        onEventSelect: { eventId in
            navigationPath.append(eventId)
        }
    )
    .navigationDestination(for: String.self) { eventId in
        EventDetailScreen(eventId: eventId)
    }
}
```

**SwiftUI Alternative (NavigationLink):**
```swift
NavigationView {
    List(events) { event in
        NavigationLink(destination: EventDetailScreen(eventId: event.id)) {
            EventRow(event: event)
        }
    }
}
```

### Navigation with Arguments

**Compose with Arguments:**
```kotlin
composable(
    route = "event/{eventId}?showShare={showShare}",
    arguments = listOf(
        navArgument("eventId") { type = NavType.StringType },
        navArgument("showShare") { type = NavType.BoolType; defaultValue = false }
    )
) { backStackEntry ->
    val eventId = backStackEntry.arguments?.getString("eventId")
    val showShare = backStackEntry.arguments?.getBoolean("showShare") ?: false
    EventDetailScreen(eventId = eventId!!, showShare = showShare)
}
```

**SwiftUI Equivalent:**
```swift
struct EventRoute: Hashable {
    let eventId: String
    let showShare: Bool = false
}

NavigationStack(path: $navigationPath) {
    EventListScreen()
        .navigationDestination(for: EventRoute.self) { route in
            EventDetailScreen(
                eventId: route.eventId,
                showShare: route.showShare
            )
        }
}

// Navigate:
navigationPath.append(EventRoute(eventId: "123", showShare: true))
```

---

## Phase 8: Theming & Colors

### Material 3 to SwiftUI Semantic Colors

**Compose Material 3 Theme:**
```kotlin
MaterialTheme(
    colorScheme = lightColorScheme(
        primary = Color(0xFF6750A4),
        onPrimary = Color(0xFFFFFFFF),
        surface = Color(0xFFFFFBFE),
        onSurface = Color(0xFF1C1B1F)
    )
) {
    // App content
}

// Usage:
Text(
    "Event",
    color = MaterialTheme.colorScheme.primary
)
```

**SwiftUI Semantic Colors (Built-in):**
```swift
// Use system colors directly
Text("Event")
    .foregroundColor(.accentColor)  // Primary equivalent

Button("Join") {
    // Action
}
.buttonStyle(.borderedProminent)  // Themed button

VStack {
    Text("Surface content")
}
.background(Color(.systemBackground))
```

**SwiftUI Custom Theme Environment (For custom colors):**
```swift
// Define theme tokens
struct ThemeColor {
    static let primary = Color(red: 0.4, green: 0.3, blue: 0.6)
    static let onPrimary = Color.white
    static let surface = Color(red: 1, green: 0.98, blue: 1)
    static let onSurface = Color(red: 0.11, green: 0.11, blue: 0.12)
}

// Or use custom environment key:
struct ThemeKey: EnvironmentKey {
    static let defaultValue = Theme()
}

extension EnvironmentValues {
    var theme: Theme {
        get { self[ThemeKey.self] }
        set { self[ThemeKey.self] = newValue }
    }
}

struct Theme {
    var primary: Color = Color(red: 0.4, green: 0.3, blue: 0.6)
    var onPrimary: Color = Color.white
}

// Usage:
struct EventCard: View {
    @Environment(\.theme) var theme

    var body: some View {
        Text("Event")
            .foregroundColor(theme.primary)
    }
}

// Apply theme:
EventCard()
    .environment(\.theme, customTheme)
```

### Typography Mapping

**Compose Material 3 Typography:**
```kotlin
MaterialTheme(
    typography = Typography(
        displayLarge = TextStyle(fontSize = 57.sp, fontWeight = FontWeight.Bold),
        headlineSmall = TextStyle(fontSize = 24.sp, fontWeight = FontWeight.Bold),
        bodyMedium = TextStyle(fontSize = 14.sp, fontWeight = FontWeight.Normal)
    )
) { }

// Usage:
Text("Title", style = MaterialTheme.typography.headlineSmall)
```

**SwiftUI Typography (Built-in styles):**
```swift
Text("Title")
    .font(.headline)  // Similar to headlineSmall

Text("Body")
    .font(.body)  // bodyMedium equivalent

// Or custom fonts:
Text("Title")
    .font(.system(size: 24, weight: .bold, design: .default))
```

---

## Phase 9: Common Patterns

### LazyColumn/List Pattern

**Compose:**
```kotlin
LazyColumn(modifier = Modifier.fillMaxSize()) {
    items(
        items = events,
        key = { it.id }
    ) { event ->
        EventRow(event = event)
    }
}
```

**SwiftUI:**
```swift
List(events, id: \.id) { event in
    EventRow(event: event)
}

// Or with LazyVStack for custom behavior:
ScrollView {
    LazyVStack(spacing: 8) {
        ForEach(events, id: \.id) { event in
            EventRow(event: event)
        }
    }
    .padding()
}
```

### Conditional Rendering

**Compose:**
```kotlin
when {
    isLoading -> CircularProgressIndicator()
    error != null -> ErrorMessage(error)
    events.isEmpty() -> Text("No events found")
    else -> EventList(events)
}
```

**SwiftUI:**
```swift
if isLoading {
    ProgressView()
} else if let error = error {
    ErrorMessage(error)
} else if events.isEmpty {
    Text("No events found")
} else {
    EventList(events: events)
}
```

### Side Effects & Lifecycle

**Compose (Effects):**
```kotlin
@Composable
fun EventDetailScreen(eventId: String) {
    LaunchedEffect(eventId) {
        viewModel.loadEvent(eventId)
    }

    DisposableEffect(Unit) {
        val listener = EventListener { /* */ }
        eventBus.register(listener)

        onDispose {
            eventBus.unregister(listener)
        }
    }
}
```

**SwiftUI (Task & onDisappear):**
```swift
struct EventDetailScreen: View {
    let eventId: String
    @StateObject var viewModel = EventDetailViewModel()

    var body: some View {
        EventDetailContent(event: viewModel.event)
            .task {
                await viewModel.loadEvent(eventId)
            }
            .onDisappear {
                viewModel.cleanup()
            }
    }
}
```

---

## Phase 10: Testing Checklist

{{#if (or (eq focus_area "all") (not focus_area))}}

After porting each component, verify:

### UI Rendering
- [ ] All text renders correctly (no truncation)
- [ ] Colors match Material Design 3 / app theme
- [ ] Spacing matches design system (8dp grid)
- [ ] Icons render correctly
- [ ] Corner radii match (8dp = .cornerRadius(8))

### State Management
- [ ] State changes trigger UI updates
- [ ] Binding changes propagate to parent
- [ ] ViewModel observing works (@StateObject)
- [ ] No infinite loops or unnecessary recomputations
- [ ] Async/await tasks complete correctly

### Gestures & Interactions
- [ ] Tap gestures work (no double-taps needed)
- [ ] Long press registers correctly
- [ ] Swipe gestures track user movement
- [ ] Buttons respond to touch
- [ ] Text selection works (if applicable)

### Navigation
- [ ] Forward navigation works
- [ ] Back button/gesture works
- [ ] Deep links resolve
- [ ] State persists across navigation
- [ ] Arguments pass correctly

### Performance
- [ ] List scrolling is smooth
- [ ] No jank on state changes
- [ ] Memory doesn't leak (check Instruments)
- [ ] Large lists render efficiently (LazyVStack)
- [ ] Images load without blocking UI

### Accessibility
- [ ] All interactive elements have labels
- [ ] Focus order is logical
- [ ] Colors have sufficient contrast (4.5:1)
- [ ] Voice over works correctly
- [ ] Text sizes are readable

{{/if}}

---

## Code Transformation Template

Use this template to systematically port each screen:

```swift
// BEFORE (Compose)
/*
@Composable
fun EventDetailScreen(eventId: String) {
    // ... implementation
}
*/

// AFTER (SwiftUI)
struct EventDetailScreen: View {
    let eventId: String
    @StateObject private var viewModel: EventDetailViewModel

    init(eventId: String) {
        self.eventId = eventId
        _viewModel = StateObject(
            wrappedValue: EventDetailViewModel(eventId: eventId)
        )
    }

    var body: some View {
        // Implementation matching original behavior
    }
}
```

---

## Quick Reference: Modifier Cheat Sheet

```swift
// Text styling
Text("Event")
    .font(.headline)
    .foregroundColor(.primary)
    .lineLimit(1)

// Button styling
Button("Join") { }
    .frame(maxWidth: .infinity)
    .frame(height: 48)
    .background(Color.accentColor)
    .foregroundColor(.white)
    .cornerRadius(8)

// Layout
VStack(alignment: .leading, spacing: 8) {
    // Content
}
.frame(maxWidth: .infinity, alignment: .leading)
.padding(16)
.background(Color(.systemBackground))
.cornerRadius(12)

// Gestures
Text("Tap me")
    .onTapGesture { print("Tapped") }
    .onLongPressGesture { print("Long pressed") }
    .gesture(
        DragGesture()
            .onChanged { print($0.translation) }
    )

// Conditional rendering
if isLoading {
    ProgressView()
} else if let event = viewModel.event {
    EventContent(event: event)
}

// Lists
List(items, id: \.id) { item in
    ItemRow(item: item)
}
```

---

## 2025-2026 Best Practices

### Do's
- ✅ Keep ViewModels as `@StateObject` (init once)
- ✅ Use `@Environment` for theme/global state
- ✅ Apply modifiers in correct order (affects layout)
- ✅ Use `Task` for async/await in SwiftUI
- ✅ Extract sub-views for large bodies
- ✅ Use `.sensoryFeedback()` for haptic feedback (iOS 17+)
- ✅ Test with both light and dark mode
- ✅ Use `GeometryReader` only when proportional sizing needed

### Don'ts
- ❌ Don't use `@ObservedObject` for ViewModels (use `@StateObject`)
- ❌ Don't mix `@State` and `@Binding` - choose one ownership pattern
- ❌ Don't apply modifiers in random order expecting same result
- ❌ Don't put heavy computation in view body (use `@State` + computed property)
- ❌ Don't force optional unwrapping in views (use if-let)
- ❌ Don't create new objects in view initialization (causes recreation)
- ❌ Don't nest GeometryReaders (performance hit)
- ❌ Don't hardcode colors (use semantic colors + theme)

---

## Resources

Based on 2025-2026 best practices from:
- [How I Mastered SwiftUI After Years of Jetpack Compose](https://dev.to/ibtiikhhn/how-i-mastered-swiftui-after-years-of-jetpack-compose-tips-for-android-devs-307n)
- [Mastering Jetpack Compose for Android and SwiftUI for iOS](https://200oksolutions.com/blog/mastering-jetpack-compose-for-android-and-swiftui-for-ios-declarative-ui-deep-dive/)
- [SwiftUI for Jetpack Compose developers - State](https://chrisbanes.me/posts/swiftui-for-jetpack-compose-devs-state/)
- [Managing user interface state - Apple Developer](https://developer.apple.com/documentation/swiftui/managing-user-interface-state)
- [Migrating to new navigation types - Apple Developer](https://developer.apple.com/documentation/swiftui/migrating-to-new-navigation-types)
- [Adapting Material Theming from Jetpack Compose to SwiftUI](https://medium.com/geekculture/adapting-material-theming-from-jetpack-compose-to-swiftui-e1bb3f40651c)
- [A guide to SwiftUI's state management system - Swift by Sundell](https://www.swiftbysundell.com/articles/swiftui-state-management-guide/)

---

## Example: Complete Screen Port

**Full Jetpack Compose Screen:**
```kotlin
@Composable
fun EventDetailScreen(
    eventId: String,
    onBackClick: () -> Unit
) {
    val viewModel = hiltViewModel<EventDetailViewModel>(
        creationCallback = { EventDetailViewModel(eventId) }
    )
    val event by viewModel.event.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()
    val error by viewModel.error.collectAsState()

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Event Details") },
                navigationIcon = {
                    IconButton(onClick = onBackClick) {
                        Icon(Icons.Default.ArrowBack, "Back")
                    }
                }
            )
        }
    ) { paddingValues ->
        when {
            isLoading -> {
                Box(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(paddingValues),
                    contentAlignment = Alignment.Center
                ) {
                    CircularProgressIndicator()
                }
            }
            error != null -> {
                Box(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(paddingValues),
                    contentAlignment = Alignment.Center
                ) {
                    Text(error!!)
                }
            }
            event != null -> {
                LazyColumn(
                    modifier = Modifier
                        .fillMaxSize()
                        .padding(paddingValues)
                        .padding(16.dp)
                ) {
                    item {
                        Text(
                            event!!.title,
                            style = MaterialTheme.typography.headlineSmall
                        )
                    }
                    item {
                        Text(event!!.location)
                    }
                    item {
                        Button(
                            onClick = { viewModel.joinEvent() },
                            modifier = Modifier.fillMaxWidth()
                        ) {
                            Text("Join Event")
                        }
                    }
                }
            }
        }
    }
}
```

**Equivalent SwiftUI Screen:**
```swift
struct EventDetailScreen: View {
    let eventId: String
    @Environment(\.dismiss) var dismiss
    @StateObject private var viewModel: EventDetailViewModel

    init(eventId: String) {
        self.eventId = eventId
        _viewModel = StateObject(
            wrappedValue: EventDetailViewModel(eventId: eventId)
        )
    }

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

    @ViewBuilder
    private var content: some View {
        if viewModel.isLoading {
            ProgressView()
        } else if let error = viewModel.error {
            Text(error)
        } else if let event = viewModel.event {
            List {
                Section {
                    Text(event.title)
                        .font(.headline)
                }
                Section {
                    Text(event.location)
                }
                Section {
                    Button(action: { viewModel.joinEvent() }) {
                        Text("Join Event")
                            .frame(maxWidth: .infinity)
                    }
                }
            }
        }
    }
}
```

---

## Summary: Phase Checklist

When porting, follow this order:

1. **Preparation**: List all composables, understand data flow
2. **Components**: Convert `@Composable` functions to `View` structs
3. **State**: Map `remember` → `@State`, ViewModels → `@StateObject`
4. **Modifiers**: Translate chains, verify order matters
5. **Layout**: Use `.frame(maxWidth: .infinity)` for fills, `GeometryReader` for weights
6. **Gestures**: Replace `.clickable()` with `.onTapGesture()`, etc.
7. **Navigation**: Convert NavHost/NavController to NavigationStack
8. **Theming**: Map Material 3 colors to custom environment keys
9. **Testing**: Verify UI, state, gestures, navigation, performance
10. **Polish**: Accessibility, dark mode, animations

Each phase builds on the previous. Complete one screen entirely before moving to the next. Refer to this guide's examples frequently - most patterns appear repeatedly across your codebase.

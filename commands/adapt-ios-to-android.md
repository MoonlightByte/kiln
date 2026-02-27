---
name: adapt-ios-to-android
description: Comprehensive guide for porting SwiftUI iOS code to Jetpack Compose Android
args:
  - name: code
    type: string
    description: SwiftUI/Swift code to port to Android
    required: true
  - name: focus
    type: string
    enum: [components, state, navigation, async, colors, lists, all]
    description: Specific porting area to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# SwiftUI to Jetpack Compose Adaptation Guide

Transform your iOS SwiftUI code to production-grade Jetpack Compose for Android using 2025-2026 best practices.

## Quick Reference: Side-by-Side Mapping

| iOS (SwiftUI) | Android (Jetpack Compose) | Notes |
|---------------|--------------------------|-------|
| `struct MyView: View` | `@Composable fun MyScreen()` | Core component definition |
| `@State private var` | `remember { mutableStateOf() }` | Local state management |
| `@StateObject` | `viewModel<MyViewModel>()` | Owned observable objects |
| `@ObservedObject` | `StateFlow<T>.collectAsState()` | External state changes |
| `@Binding` | `(value: T, onValueChange: (T) -> Unit)` | Two-way binding parameters |
| `@EnvironmentObject` | `CompositionLocal<T>` | Dependency injection |
| `NavigationStack` | `NavHost` + `NavController` | Navigation state management |
| `List { ForEach }` | `LazyColumn { items() }` | Efficient scrollable lists |
| `Combine.Publisher` | `Flow<T>` from Kotlin | Reactive streams |
| `async/await Task` | `launch { }` coroutine scope | Async operations |
| `.font(.title)` | `MaterialTheme.typography.titleLarge` | Typography tokens |
| `.foregroundColor()` | `color = MaterialTheme.colorScheme.primary` | Semantic colors |
| `.padding()` | `Modifier.padding()` | Spacing/layout |
| `.onAppear { }` | `LaunchedEffect(Unit) { }` | Lifecycle hooks |

---

## Part 1: Component Structure (Critical)

### iOS View Definition
```swift
struct EventDetailView: View {
    @StateObject private var viewModel = EventDetailViewModel()

    var body: some View {
        VStack(spacing: 16) {
            Text("Event Details")
                .font(.title)

            Button(action: { viewModel.joinEvent() }) {
                Text("Join Event")
            }
        }
    }
}
```

### Android Equivalent (Compose)
```kotlin
@Composable
fun EventDetailScreen(viewModel: EventDetailViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()

    Column(modifier = Modifier.padding(16.dp), verticalArrangement = Arrangement.spacedBy(16.dp)) {
        Text(
            text = "Event Details",
            style = MaterialTheme.typography.titleLarge
        )

        Button(onClick = { viewModel.joinEvent() }) {
            Text("Join Event")
        }
    }
}
```

**Key Differences:**
- Compose uses `@Composable` function instead of `View` struct
- Parameter-based dependency injection (viewModel as parameter)
- Layout containers: `VStack` → `Column`, `HStack` → `Row`
- Spacing: explicit `verticalArrangement` instead of modifier
- Styling applied via modifier chain (at end), not chained methods
- State observed directly in composable (don't use `@State` in parameters)

**Porting Checklist:**
- [ ] Replace all `struct X: View` with `@Composable fun XScreen()`
- [ ] Remove `@StateObject` - pass ViewModel as parameter
- [ ] Remove `@State` from composable parameters
- [ ] Replace `VStack`/`HStack` with `Column`/`Row`
- [ ] Move all styling to `Modifier` chain at composable level
- [ ] Verify all modifiers placed at end of composable call

---

## Part 2: State Management (Critical)

### Pattern 1: Local State (@State in SwiftUI)

**iOS:**
```swift
@State private var isExpanded = false

var body: some View {
    VStack {
        Text("Details")
            .onTapGesture {
                isExpanded.toggle()
            }

        if isExpanded {
            Text("Expanded content")
        }
    }
}
```

**Android:**
```kotlin
@Composable
fun ExpandableDetails() {
    var isExpanded by remember { mutableStateOf(false) }

    Column {
        Text(
            text = "Details",
            modifier = Modifier.clickable { isExpanded = !isExpanded }
        )

        if (isExpanded) {
            Text("Expanded content")
        }
    }
}
```

**Key Differences:**
- `@State` → `remember { mutableStateOf() }`
- SwiftUI: `.toggle()` method | Compose: Direct assignment `= !value`
- iOS: `.onTapGesture` | Android: `Modifier.clickable`

**Porting Checklist:**
- [ ] Replace `@State private var x = value` with `var x by remember { mutableStateOf(value) }`
- [ ] Use `by` delegate for cleaner syntax
- [ ] Replace `.toggle()` with `= !value`
- [ ] Replace `.onTapGesture` with `Modifier.clickable`

---

### Pattern 2: Owned Observable Objects (@StateObject)

**iOS:**
```swift
@MainActor
class EventViewModel: ObservableObject {
    @Published var events: [Event] = []
    @Published var isLoading = false

    func loadEvents() async {
        isLoading = true
        let fetched = await api.fetchEvents()
        events = fetched
        isLoading = false
    }
}

struct EventListView: View {
    @StateObject private var viewModel = EventViewModel()

    var body: some View {
        List(viewModel.events) { event in
            EventRow(event: event)
        }
        .onAppear {
            Task { await viewModel.loadEvents() }
        }
    }
}
```

**Android:**
```kotlin
class EventViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(EventUiState())
    val uiState: StateFlow<EventUiState> = _uiState.asStateFlow()

    fun loadEvents() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            val fetched = api.fetchEvents()
            _uiState.update { it.copy(
                events = fetched,
                isLoading = false
            ) }
        }
    }
}

data class EventUiState(
    val events: List<Event> = emptyList(),
    val isLoading: Boolean = false
)

@Composable
fun EventListScreen(viewModel: EventViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()

    LaunchedEffect(Unit) {
        viewModel.loadEvents()
    }

    LazyColumn {
        items(uiState.events, key = { it.id }) { event in
            EventRow(event = event)
        }
    }
}
```

**Key Differences:**
- SwiftUI: `@Published var x` | Compose: `MutableStateFlow` + backing private field
- SwiftUI: `@MainActor` for thread safety | Compose: `viewModelScope` for lifecycle
- SwiftUI: `Task { await }` in `.onAppear` | Compose: `LaunchedEffect` with `viewModelScope.launch`
- Data container: `@ObservedObject` implicitly unwraps | Compose: Explicit `data class UiState`
- List rendering: `List` with auto ID | Compose: `LazyColumn { items() }` with explicit key

**Porting Checklist:**
- [ ] Replace `ObservableObject` with `ViewModel()`
- [ ] Replace each `@Published var` with `private val _x = MutableStateFlow(initial)` + public `StateFlow`
- [ ] Remove `@MainActor` - use `viewModelScope.launch` instead
- [ ] Create `data class XxxUiState` to hold all observable state
- [ ] Use `.copy()` to update state immutably
- [ ] Replace `.onAppear { Task { } }` with `LaunchedEffect(Unit) { viewModelScope.launch { } }`
- [ ] Replace `List` with `LazyColumn { items() }`
- [ ] Always provide explicit `key` to `items()` function

---

### Pattern 3: Two-Way Binding (@Binding)

**iOS:**
```swift
@Binding var userName: String

TextField("Enter name", text: $userName)
```

**Android:**
```kotlin
TextField(
    value = userName,
    onValueChange = { userName = it },
    label = { Text("Enter name") }
)
```

**Key Differences:**
- SwiftUI: `$binding` syntax | Compose: Explicit `value` and `onValueChange` parameters
- No magic syntax in Compose - state flow is explicit

**Porting Checklist:**
- [ ] Remove `@Binding` parameter declarations
- [ ] Create mutable state in parent composable
- [ ] Pass state value + update callback to child
- [ ] Child calls callback: `onValueChange(newValue)`

---

### Pattern 4: Environment Objects (@EnvironmentObject)

**iOS:**
```swift
class ThemeSettings: ObservableObject {
    @Published var isDarkMode = false
}

@main
struct MyApp: App {
    @StateObject private var themeSettings = ThemeSettings()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(themeSettings)
        }
    }
}

struct SomeView: View {
    @EnvironmentObject var theme: ThemeSettings

    var body: some View {
        Text("Dark mode: \(theme.isDarkMode)")
    }
}
```

**Android:**
```kotlin
// Create as CompositionLocal
val LocalTheme = compositionLocalOf<ThemeSettings> {
    error("ThemeSettings not provided")
}

class ThemeSettings {
    var isDarkMode by mutableStateOf(false)
}

// Provide at root
val themeSettings = ThemeSettings()
CompositionLocalProvider(LocalTheme provides themeSettings) {
    MyApp()
}

// Consume in composable
@Composable
fun SomeScreen() {
    val theme = LocalTheme.current
    Text("Dark mode: ${theme.isDarkMode}")
}
```

**Key Differences:**
- SwiftUI: `.environmentObject()` modifier | Compose: `CompositionLocalProvider` wrapper
- Creation implicit (autowiring) vs explicit injection

**Porting Checklist:**
- [ ] Create `CompositionLocal<T>` for each environment object
- [ ] Wrap root composable with `CompositionLocalProvider`
- [ ] Replace `@EnvironmentObject var x` with `val x = LocalX.current`
- [ ] Provide all locals at app entry point

---

## Part 3: Navigation (Important)

### NavigationStack Pattern

**iOS (SwiftUI 16+):**
```swift
NavigationStack(path: $navigationPath) {
    List(events) { event in
        NavigationLink(value: event) {
            EventRow(event: event)
        }
    }
    .navigationDestination(for: Event.self) { event in
        EventDetailView(event: event)
    }
}
```

**Android (Compose with Jetpack Navigation):**
```kotlin
val navController = rememberNavController()

NavHost(navController = navController, startDestination = "events") {
    composable("events") {
        EventListScreen(
            onEventClick = { eventId ->
                navController.navigate("eventDetail/$eventId")
            }
        )
    }

    composable(
        route = "eventDetail/{eventId}",
        arguments = listOf(navArgument("eventId") { type = NavType.StringType })
    ) { backStackEntry ->
        val eventId = backStackEntry.arguments?.getString("eventId") ?: return@composable
        EventDetailScreen(eventId = eventId)
    }
}
```

**Key Differences:**
- SwiftUI: Value-based navigation with `NavigationStack` | Compose: String-based routes with `NavHost`
- SwiftUI: `.navigationDestination` modifier | Compose: Graph structure in `NavHost`
- iOS: Navigation state in variable | Compose: NavController manages state
- Parameter passing: Value objects vs URL parameters

**Porting Checklist:**
- [ ] Replace `NavigationStack` with `NavHost` + `rememberNavController()`
- [ ] Convert each `.navigationDestination` to `composable()` call
- [ ] Convert value-based navigation to string routes
- [ ] Extract route parameters: `backStackEntry.arguments?.getString(name)`
- [ ] Use `navController.navigate(route)` for navigation
- [ ] Add back button handling with `navController.popBackStack()`

---

## Part 4: Async/Reactive Programming (Important)

### Combine to Flow Conversion

**iOS (Combine):**
```swift
class DataViewModel: ObservableObject {
    @Published var data: String = ""

    func fetchData() async {
        for try await item in api.streamData() {
            DispatchQueue.main.async {
                self.data = item
            }
        }
    }
}

struct DataView: View {
    @StateObject private var viewModel = DataViewModel()

    var body: some View {
        Text(viewModel.data)
            .task {
                await viewModel.fetchData()
            }
    }
}
```

**Android (Kotlin Flow):**
```kotlin
class DataViewModel : ViewModel() {
    private val _dataFlow = MutableStateFlow("")
    val dataFlow: StateFlow<String> = _dataFlow.asStateFlow()

    fun fetchData() {
        viewModelScope.launch {
            try {
                api.streamData().collect { item ->
                    _dataFlow.value = item
                }
            } catch (e: Exception) {
                // Handle error
            }
        }
    }
}

@Composable
fun DataScreen(viewModel: DataViewModel = viewModel()) {
    val data by viewModel.dataFlow.collectAsState()

    LaunchedEffect(Unit) {
        viewModel.fetchData()
    }

    Text(data)
}
```

**Key Differences:**
- SwiftUI: Implicit main thread with `@MainActor` | Compose: Explicit with `viewModelScope`
- SwiftUI: `async/await Task` | Compose: `launch { }` coroutine builder
- SwiftUI: `.task` lifecycle | Compose: `LaunchedEffect`
- Data binding automatic vs explicit `.collectAsState()`

**Porting Checklist:**
- [ ] Replace `async/await` blocks with `viewModelScope.launch { }`
- [ ] Convert `Combine.Publisher` to `Flow<T>`
- [ ] Replace `.task { await }` with `LaunchedEffect(Unit) { launch { } }`
- [ ] Use `StateFlow<T>` for observable state (replaces `@Published`)
- [ ] Collect flows with `.collect { }` or `.collectAsState()` in composable
- [ ] Always use `viewModelScope` for lifecycle safety (cancels on ViewModel destroy)

---

## Part 5: Lists and Grids (Important)

### List Rendering

**iOS:**
```swift
List(events, id: \.id) { event in
    NavigationLink(value: event) {
        EventRow(event: event)
    }
}
```

**Android:**
```kotlin
LazyColumn {
    items(events, key = { it.id }) { event in
        EventRowButton(
            event = event,
            onClick = { navController.navigate("eventDetail/${event.id}") }
        )
    }
}
```

**Key Differences:**
- SwiftUI: `List` with implicit navigation | Compose: `LazyColumn` + explicit onClick handling
- ID specification: `id: \.id` vs `key = { }`
- Performance: Both lazy-load (render visible items only)

**Porting Checklist:**
- [ ] Replace `List` with `LazyColumn`
- [ ] Replace `ForEach` with `items()` lambda
- [ ] Move `NavigationLink` logic to item `onClick` callback
- [ ] Always provide explicit `key` for list items
- [ ] Use `Modifier.clickable` or `Surface(onClick = {})` for item interaction

### Grid Rendering

**iOS:**
```swift
let columns = [
    GridItem(.flexible()),
    GridItem(.flexible())
]

LazyVGrid(columns: columns, spacing: 16) {
    ForEach(events) { event in
        EventCard(event: event)
    }
}
```

**Android:**
```kotlin
LazyVerticalGrid(
    columns = GridCells.Fixed(2),
    horizontalArrangement = Arrangement.spacedBy(16.dp),
    verticalArrangement = Arrangement.spacedBy(16.dp)
) {
    items(events, key = { it.id }) { event in
        EventCard(event = event)
    }
}
```

**Key Differences:**
- SwiftUI: `LazyVGrid` with `GridItem` config | Compose: `LazyVerticalGrid` with `GridCells`
- iOS: `spacing` parameter | Compose: Explicit `horizontalArrangement`/`verticalArrangement`
- Column count: `GridItem(.flexible())` arrays vs `GridCells.Fixed(2)`

**Porting Checklist:**
- [ ] Replace `LazyVGrid` with `LazyVerticalGrid`
- [ ] Convert grid layout: fixed columns → `GridCells.Fixed(count)`
- [ ] Convert grid layout: flexible → `GridCells.Adaptive(minSize)`
- [ ] Use `Arrangement.spacedBy()` for consistent spacing
- [ ] Provide `key` in `items()` call

---

## Part 6: Typography & Semantic Colors (Important)

### Typography Tokens

**iOS:**
```swift
Text("Headline")
    .font(.headline)

Text("Body Text")
    .font(.body)

Text("Small")
    .font(.caption)

Text("Custom Size")
    .font(.system(size: 18, weight: .bold))  // AVOID - use styles instead
```

**Android:**
```kotlin
Text(
    text = "Headline",
    style = MaterialTheme.typography.headlineSmall
)

Text(
    text = "Body Text",
    style = MaterialTheme.typography.bodyLarge
)

Text(
    text = "Small",
    style = MaterialTheme.typography.labelSmall
)

// Never hardcode size - use tokens!
```

**iOS to Android Typography Mapping:**
| iOS Style | Size | Android Equivalent | Size |
|-----------|------|-------------------|------|
| `.largeTitle` | 34pt | `displayLarge` | 57sp |
| `.title` | 28pt | `titleLarge` | 22sp |
| `.headline` | 17pt (bold) | `headlineSmall` | 16sp |
| `.body` | 17pt | `bodyLarge` | 16sp |
| `.caption` | 12pt | `labelSmall` | 12sp |
| `.system(size:)` | custom | ❌ FORBIDDEN | Use token instead |

**Porting Checklist:**
- [ ] Replace all `.font(.style)` with `style = MaterialTheme.typography.token`
- [ ] Never use hardcoded font sizes in Compose
- [ ] Map iOS styles to closest Material 3 token
- [ ] Use `weight` parameter only if Material token doesn't match (bold text, etc.)

### Semantic Colors

**iOS:**
```swift
Text("Hello")
    .foregroundColor(.label)

Button(action: {}) {
    Text("Press me")
}
.foregroundColor(.white)
.background(Color.accentColor)

HStack {
    Image(systemName: "star")
        .foregroundColor(.systemOrange)
}
.background(Color.systemBackground)
```

**Android:**
```kotlin
Text(
    text = "Hello",
    color = MaterialTheme.colorScheme.onSurface
)

Button(onClick = {}) {
    Text("Press me")
}

Row(modifier = Modifier.background(MaterialTheme.colorScheme.surface)) {
    Icon(
        imageVector = Icons.Default.Star,
        contentDescription = null,
        tint = MaterialTheme.colorScheme.tertiary
    )
}
```

**iOS to Android Color Mapping:**
| iOS Semantic Color | Meaning | Android Equivalent |
|-------------------|---------|-------------------|
| `.label` | Primary text | `onSurface` |
| `.secondaryLabel` | Secondary text | `onSurfaceVariant` |
| `.systemBackground` | App background | `surface` |
| `.accentColor` | Interactive elements | `primary` |
| `.systemRed` | Error state | `error` |
| `.systemGreen` | Success state | `tertiary` / custom token |
| `.systemOrange` | Warning state | `tertiary` / custom token |
| `.systemGray` | Disabled state | `onSurface.copy(alpha = 0.38f)` |

**Porting Checklist:**
- [ ] Replace `.foregroundColor()` with `color = MaterialTheme.colorScheme.token`
- [ ] Replace `.background(Color)` with `Modifier.background(MaterialTheme.colorScheme.token)`
- [ ] Use Material 3 semantic tokens (primary, secondary, tertiary, error, surface)
- [ ] Never hardcode hex colors - use theme tokens always
- [ ] For custom colors (app-specific), add to theme's `colorScheme`

---

## Part 7: Modifiers & Layout (Important)

### Common Modifier Patterns

**iOS:**
```swift
VStack {
    Text("Hello")
        .font(.title)
        .padding(16)
        .background(Color.blue)
        .cornerRadius(8)
}
.frame(maxWidth: .infinity)
.navigationTitle("Screen")
```

**Android:**
```kotlin
Column(
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp)
        .background(
            color = MaterialTheme.colorScheme.primary,
            shape = RoundedCornerShape(8.dp)
        )
) {
    Text(
        text = "Hello",
        style = MaterialTheme.typography.titleLarge
    )
}
```

**iOS to Android Modifier Mapping:**
| iOS Modifier | iOS Example | Android Equivalent | Android Example |
|--------------|-------------|-------------------|-----------------|
| `.padding(x)` | `.padding(16)` | `Modifier.padding()` | `.padding(16.dp)` |
| `.frame(width:)` | `.frame(width: 100)` | `Modifier.width()` | `.width(100.dp)` |
| `.frame(maxWidth:)` | `.frame(maxWidth: .infinity)` | `Modifier.fillMaxWidth()` | `.fillMaxWidth()` |
| `.background(Color)` | `.background(.blue)` | `Modifier.background()` | `.background(colorScheme.primary)` |
| `.cornerRadius(x)` | `.cornerRadius(8)` | `Modifier.background(shape=)` | `.background(..., shape = RoundedCornerShape(8.dp))` |
| `.shadow()` | `.shadow(radius: 8)` | `Modifier.shadow()` | `.shadow(8.dp)` |
| `.opacity(x)` | `.opacity(0.5)` | `Modifier.alpha()` | `.alpha(0.5f)` |
| `.offset(x, y)` | `.offset(x: 10)` | `Modifier.offset()` | `.offset(x = 10.dp)` |
| `.rotation()` | `.rotationEffect(.degrees(45))` | `Modifier.rotate()` | `.rotate(45f)` |

**Porting Checklist:**
- [ ] All layout changes go in `Modifier` chain
- [ ] Modifiers are applied at composable level, not on parent
- [ ] `.padding()` → `Modifier.padding()` (works on self)
- [ ] `.background()` always wrap with `color` + `shape` parameters
- [ ] Use `dp` units consistently (never `px`)
- [ ] Replace `.frame(maxWidth: .infinity)` with `.fillMaxWidth()`

---

## Part 8: Conditional Layout

**iOS:**
```swift
var body: some View {
    VStack {
        if showHeader {
            HeaderView()
        }

        switch state {
        case .loading:
            ProgressView()
        case .success:
            ContentView()
        case .error:
            ErrorView()
        }
    }
}
```

**Android:**
```kotlin
@Composable
fun MyScreen(viewModel: ViewModel) {
    val uiState by viewModel.uiState.collectAsState()

    Column {
        if (uiState.showHeader) {
            HeaderView()
        }

        when (uiState.state) {
            UiState.Loading -> CircularProgressIndicator()
            UiState.Success -> ContentView()
            UiState.Error -> ErrorView()
        }
    }
}
```

**Key Differences:**
- Kotlin/Java use `when` instead of `switch`
- Sealed classes for state enums (type-safe)
- Conditional rendering same pattern (if/when in composable body)

**Porting Checklist:**
- [ ] Replace `if let` with simple `if`
- [ ] Replace `switch` with `when`
- [ ] Use sealed classes for state enums
- [ ] Keep conditional logic in composable body (prefer simple conditions)

---

## Full Porting Checklist

### Architecture Level
- [ ] Created `ViewModel` class extending `androidx.lifecycle.ViewModel`
- [ ] Created `UiState` data class holding all observable state
- [ ] Replaced all `@Published` with `MutableStateFlow` + public `StateFlow`
- [ ] Created app-level `CompositionLocal` for dependency injection
- [ ] Set up `NavHost` + `NavController` for navigation
- [ ] Created semantic color tokens in `Color.kt`
- [ ] Created typography tokens in `Type.kt`

### Component Level
- [ ] All view structs converted to `@Composable` functions
- [ ] Removed all `@State` from composable parameters
- [ ] Removed `@StateObject` - pass ViewModel as parameter
- [ ] Converted `VStack`/`HStack` to `Column`/`Row`
- [ ] Replaced all `.onAppear` with `LaunchedEffect`
- [ ] Replaced all `.onDisappear` with `DisposableEffect`

### State Management Level
- [ ] All state flows through ViewModel (single source of truth)
- [ ] Used `viewModelScope.launch` for all async operations
- [ ] Collected flows with `.collectAsState()` in composable
- [ ] Updated state immutably with `.copy()` or `.update()`
- [ ] No side effects in composable body (use `LaunchedEffect`)

### UI Level
- [ ] All text styles use `MaterialTheme.typography.*`
- [ ] All colors use `MaterialTheme.colorScheme.*`
- [ ] All spacing uses `dp` units via `Modifier`
- [ ] All interactive elements are 48×48dp minimum
- [ ] All icons have `contentDescription` for accessibility
- [ ] All hardcoded colors replaced with theme tokens

### Navigation Level
- [ ] Converted `NavigationStack` to `NavHost`
- [ ] Created route strings for each destination
- [ ] Set up `navArgument` for typed route parameters
- [ ] Navigation driven through callbacks, not value objects
- [ ] Back button handled with `navController.popBackStack()`

### Performance Level
- [ ] Using `LazyColumn` instead of `Column` for lists
- [ ] Provided explicit `key` to all `items()` calls
- [ ] No `remember` on mutable state (use ViewModel)
- [ ] No heavy computations in composable body
- [ ] Modifiers created outside composable (stable references)

---

## Common Porting Mistakes to Avoid

### ❌ Mistake 1: State in Composable Parameters
```kotlin
// WRONG
@Composable
fun EventRow(
    event: Event,
    @Suppress("UNUSED_PARAMETER") onSelect: (Event) -> Unit  // Hint: closure should be parameter!
) {
    var isSelected by remember { mutableStateOf(false) }  // State should be in ViewModel!

    Row(Modifier.clickable {
        isSelected = !isSelected  // Local state - lost on recomposition
        onSelect(event)
    }) { ... }
}

// CORRECT
@Composable
fun EventRow(
    event: Event,
    isSelected: Boolean,
    onSelect: (Event) -> Unit
) {
    Row(Modifier.clickable { onSelect(event) }) { ... }
}
```

### ❌ Mistake 2: Async Work in Composable
```kotlin
// WRONG
@Composable
fun EventListScreen() {
    var events by remember { mutableStateOf<List<Event>>(emptyList()) }

    // This runs on EVERY recomposition!
    viewModelScope.launch {
        events = api.fetchEvents()
    }

    LazyColumn {
        items(events) { event in EventRow(event) }
    }
}

// CORRECT
class EventViewModel : ViewModel() {
    private val _events = MutableStateFlow<List<Event>>(emptyList())
    val events: StateFlow<List<Event>> = _events.asStateFlow()

    init {
        viewModelScope.launch {
            _events.value = api.fetchEvents()
        }
    }
}

@Composable
fun EventListScreen(viewModel: EventViewModel = viewModel()) {
    val events by viewModel.events.collectAsState()

    LazyColumn {
        items(events) { event in EventRow(event) }
    }
}
```

### ❌ Mistake 3: Hardcoded Colors
```kotlin
// WRONG
Button(
    onClick = {},
    colors = ButtonDefaults.buttonColors(
        containerColor = Color(0xFF3F51B5),  // Hardcoded blue
        contentColor = Color.White
    )
) {
    Text("Click me")
}

// CORRECT
Button(
    onClick = {},
    colors = ButtonDefaults.buttonColors(
        containerColor = MaterialTheme.colorScheme.primary,
        contentColor = MaterialTheme.colorScheme.onPrimary
    )
) {
    Text("Click me")
}

// OR even simpler
Button(onClick = {}) {
    Text("Click me")
}  // Uses primary color by default
```

### ❌ Mistake 4: Using VStack/HStack Instead of Column/Row
```kotlin
// WRONG
@Composable
fun EventCard(event: Event) {
    Surface(
        modifier = Modifier
            .padding(8.dp)
            .clip(RoundedCornerShape(8.dp))
    ) {
        // Using older imperative API
        Box {
            // Manual layout
        }
    }
}

// CORRECT
@Composable
fun EventCard(event: Event) {
    Surface(
        modifier = Modifier
            .padding(8.dp)
            .clip(RoundedCornerShape(8.dp))
    ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            verticalArrangement = Arrangement.spacedBy(8.dp)
        ) {
            Text(
                text = event.title,
                style = MaterialTheme.typography.titleLarge
            )
            Text(
                text = event.description,
                style = MaterialTheme.typography.bodyMedium
            )
        }
    }
}
```

### ❌ Mistake 5: Missing contentDescription on Icons
```kotlin
// WRONG
IconButton(onClick = { share() }) {
    Icon(Icons.Default.Share, contentDescription = null)  // Invisible to screen readers!
}

// CORRECT
IconButton(onClick = { share() }) {
    Icon(
        imageVector = Icons.Default.Share,
        contentDescription = "Share event"
    )
}
```

### ❌ Mistake 6: Using List Instead of LazyColumn
```kotlin
// WRONG - loads ALL items into memory
Column {
    repeat(1000) { index in
        EventRow(events[index])
    }
}

// CORRECT - loads only visible items
LazyColumn {
    items(events, key = { it.id }) { event in
        EventRow(event)
    }
}
```

### ❌ Mistake 7: Mutable State Escaping ViewModel
```kotlin
// WRONG - state mutated outside ViewModel
@Composable
fun EventScreen(viewModel: EventViewModel = viewModel()) {
    var selectedEvent by remember { mutableStateOf<Event?>(null) }

    LazyColumn {
        items(viewModel.events) { event in
            EventRow(
                event = event,
                onClick = { selectedEvent = event }  // Local state!
            )
        }
    }

    if (selectedEvent != null) {
        EventDetail(event = selectedEvent!!)
    }
}

// CORRECT - state flows through ViewModel
class EventViewModel : ViewModel() {
    private val _selectedEvent = MutableStateFlow<Event?>(null)
    val selectedEvent: StateFlow<Event?> = _selectedEvent.asStateFlow()

    fun selectEvent(event: Event) {
        _selectedEvent.value = event
    }

    fun clearSelection() {
        _selectedEvent.value = null
    }
}

@Composable
fun EventScreen(viewModel: EventViewModel = viewModel()) {
    val selectedEvent by viewModel.selectedEvent.collectAsState()

    LazyColumn {
        items(viewModel.events) { event in
            EventRow(
                event = event,
                onClick = { viewModel.selectEvent(event) }
            )
        }
    }

    selectedEvent?.let { event in
        EventDetail(event = event)
    }
}
```

---

## Quick Conversion Examples

### Example 1: Simple Login Form

**iOS:**
```swift
struct LoginView: View {
    @State private var email = ""
    @State private var password = ""
    @StateObject private var viewModel = LoginViewModel()

    var body: some View {
        VStack(spacing: 12) {
            TextField("Email", text: $email)
                .textFieldStyle(.roundedBorder)

            SecureField("Password", text: $password)
                .textFieldStyle(.roundedBorder)

            Button(action: {
                Task {
                    await viewModel.login(email: email, password: password)
                }
            }) {
                Text("Login")
                    .frame(maxWidth: .infinity)
            }
            .buttonStyle(.borderedProminent)
        }
        .padding(16)
    }
}

class LoginViewModel: ObservableObject {
    @Published var isLoading = false
    @Published var errorMessage: String?

    func login(email: String, password: String) async {
        isLoading = true
        defer { isLoading = false }

        do {
            _ = try await AuthService.login(email: email, password: password)
        } catch {
            errorMessage = error.localizedDescription
        }
    }
}
```

**Android:**
```kotlin
data class LoginUiState(
    val email: String = "",
    val password: String = "",
    val isLoading: Boolean = false,
    val errorMessage: String? = null
)

class LoginViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(LoginUiState())
    val uiState: StateFlow<LoginUiState> = _uiState.asStateFlow()

    fun updateEmail(email: String) {
        _uiState.update { it.copy(email = email) }
    }

    fun updatePassword(password: String) {
        _uiState.update { it.copy(password = password) }
    }

    fun login() {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            try {
                AuthService.login(
                    email = _uiState.value.email,
                    password = _uiState.value.password
                )
                _uiState.update { it.copy(isLoading = false, errorMessage = null) }
            } catch (e: Exception) {
                _uiState.update { it.copy(
                    isLoading = false,
                    errorMessage = e.message
                ) }
            }
        }
    }
}

@Composable
fun LoginScreen(viewModel: LoginViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(12.dp)
    ) {
        OutlinedTextField(
            value = uiState.email,
            onValueChange = { viewModel.updateEmail(it) },
            label = { Text("Email") },
            modifier = Modifier.fillMaxWidth()
        )

        OutlinedTextField(
            value = uiState.password,
            onValueChange = { viewModel.updatePassword(it) },
            label = { Text("Password") },
            visualTransformation = PasswordVisualTransformation(),
            modifier = Modifier.fillMaxWidth()
        )

        Button(
            onClick = { viewModel.login() },
            modifier = Modifier.fillMaxWidth(),
            enabled = !uiState.isLoading
        ) {
            if (uiState.isLoading) {
                CircularProgressIndicator(
                    modifier = Modifier.size(20.dp),
                    color = MaterialTheme.colorScheme.onPrimary
                )
            } else {
                Text("Login")
            }
        }

        uiState.errorMessage?.let { error in
            Text(
                text = error,
                color = MaterialTheme.colorScheme.error,
                style = MaterialTheme.typography.bodySmall
            )
        }
    }
}
```

---

## Resources & References

### Official Documentation
- [Jetpack Compose Official Docs](https://developer.android.com/develop/ui/compose)
- [Material 3 Tokens System](https://m3.material.io/foundations/design-tokens/overview)
- [SwiftUI to Compose Mapping - Jetpack Playground](https://foso.github.io/Jetpack-Compose-Playground/compose_for/swiftui_devs/)
- [Kotlin Flow Documentation](https://kotlinlang.org/docs/flow.html)
- [Jetpack Navigation Documentation](https://developer.android.com/guide/navigation)

### 2025-2026 Articles
- [Compose Multiplatform for iOS Production Readiness 2026](https://vocal.media/01/compose-multiplatform-for-i-os-production-readiness-in-2026)
- [SwiftUI ↔ Jetpack Compose Reference - Medium](https://medium.com/@omz1990/swiftui-to-jetpack-compose-and-vice-vera-reference-guide-0b293e5a013f)
- [Same Grammar, Different Dialects - Medium 2025](https://medium.com/@sanchit115/diving-into-swiftui-and-jetpack-compose-together-par-1-f3447a79920d)
- [Mastering Material 3 in Jetpack Compose 2025](https://medium.com/@hiren6997/mastering-material-3-in-jetpack-compose-the-2025-guide-1c1bd5acc480)
- [SwiftUI NavigationStack Migration 2025](https://joelkingsley.com/2025/07/30/migrating-to-navigationstack-in-swiftui-a-step-by-step-guide/)

### Community Resources
- [Chris Banes on SwiftUI for Compose Devs](https://chrisbanes.me/posts/swiftui-for-jetpack-compose-devs-state/)
- [Compose for SwiftUI Developers](https://medium.com/@myshkinasasha/inside-swiftui-state-property-wrapper-3f3f1d1d9e33)
- [Kotlin Multiplatform Compose-SwiftUI Integration](https://kotlinlang.org/docs/multiplatform/compose-swiftui-integration.html)

---

## Execution Steps

1. **Analyze the provided iOS code**
2. **Identify all SwiftUI patterns** (state, navigation, async, etc.)
3. **Map each pattern** using the conversion tables above
4. **Flag any non-portable patterns** (platform-specific APIs, etc.)
5. **Provide complete Kotlin/Compose equivalent**
6. **Include Material 3 styling adjustments**
7. **Add accessibility considerations** (contentDescription, touch targets, contrast)
8. **Deliver ready-to-implement code** with clear comments

---

## Output Format

```markdown
## Porting Analysis: [Component/Screen Name]

### Code Structure
- [Structural analysis with pattern mapping]

### State Management Pattern
- [How state management was converted]

### Navigation Changes
- [Any navigation routing changes]

### Styling & Tokens
- [Color and typography token mapping]

### Async/Reactive Changes
- [Combine to Flow conversion]

### Complete Compose Implementation
[Full ready-to-implement code]

### Migration Checklist
- [ ] [Specific item]
- [ ] [Specific item]

### Potential Issues & Solutions
- [Any gotchas or challenges]

### Accessibility Review
- [ ] Touch targets are 48×48dp minimum
- [ ] Icons have contentDescription
- [ ] Colors use theme tokens
- [ ] Text styles use Material 3
```

---

**This guide ensures your SwiftUI code becomes high-quality, maintainable Jetpack Compose for Android while respecting Material 3 design principles and platform idioms.**

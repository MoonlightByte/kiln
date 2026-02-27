# Platform Adaptation Rules: Android ↔ iOS

**Purpose**: Provide clear rules for idiomatically adapting code between Android and iOS
**Version**: 1.0 (February 2025)
**Used by**: `/adapt-android-to-ios` and `/adapt-ios-to-android` skills

---

## Core Principle: Idiom Over Literal Translation

**Rule**: Never do literal code translation. Always adapt to the platform's idioms and design language.

### Example: Wrong vs. Right

#### WRONG (Literal Translation)
```
Android: Button with Material Design 3 elevation
iOS: Copy code, change Compose → SwiftUI, elevation → shadow
Result: Button looks wrong on iOS (doesn't follow HIG)
```

#### RIGHT (Idiomatic Adaptation)
```
Android: Material Design 3 button with elevation
iOS: SwiftUI button with accent color (HIG standard)
Result: Button looks native on each platform
```

---

## 1. Layout Adaptation Rules

### 1.1 Column/Row ↔ VStack/HStack

#### Android → iOS
```kotlin
// ANDROID - Jetpack Compose
Column(
    modifier = Modifier
        .fillMaxWidth()
        .padding(16.dp),
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
    Text("Title")
    Text("Subtitle")
}
```

**Adapts to:**
```swift
// iOS - SwiftUI
VStack(alignment: .leading, spacing: 8) {
    Text("Title")
    Text("Subtitle")
}
.padding(16)
```

**Rules**:
- [ ] `Column` → `VStack`
- [ ] `Row` → `HStack`
- [ ] `modifier = Modifier.fillMaxWidth()` → `.frame(maxWidth: .infinity)`
- [ ] `padding(16.dp)` → `.padding(16)`
- [ ] `verticalArrangement = Arrangement.spacedBy(8.dp)` → `spacing: 8`
- [ ] `horizontalArrangement = Arrangement.spacedBy(8.dp)` → `spacing: 8`

#### iOS → Android
```swift
// iOS - SwiftUI
HStack(alignment: .center, spacing: 12) {
    Image(systemName: "house")
    Text("Home")
}
.padding(.horizontal, 16)
```

**Adapts to:**
```kotlin
// ANDROID - Jetpack Compose
Row(
    modifier = Modifier
        .fillMaxWidth()
        .padding(horizontal = 16.dp),
    horizontalArrangement = Arrangement.spacedBy(12.dp),
    verticalAlignment = Alignment.CenterVertically
) {
    Icon(Icons.Default.Home, contentDescription = null)
    Text("Home")
}
```

**Rules**:
- [ ] `VStack` → `Column`
- [ ] `HStack` → `Row`
- [ ] `.frame(maxWidth: .infinity)` → `modifier = Modifier.fillMaxWidth()`
- [ ] `.padding(16)` → `padding(16.dp)`
- [ ] `spacing: 8` → `arrangement = Arrangement.spacedBy(8.dp)`

---

### 1.2 ScrollView ↔ LazyColumn/LazyRow

#### iOS → Android (Large Lists)
```swift
// iOS - ScrollView (inefficient for large lists)
ScrollView {
    VStack(spacing: 8) {
        ForEach(items, id: \.id) { item in
            ItemRow(item)
        }
    }
}
```

**SHOULD ADAPT TO:**
```kotlin
// ANDROID - LazyColumn (efficient rendering)
LazyColumn(
    modifier = Modifier.fillMaxSize(),
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
    items(items, key = { it.id }) { item ->
        ItemRow(item)
    }
}
```

**Rules**:
- [ ] `ScrollView { VStack { ForEach... } }` → `LazyColumn { items... }`
- [ ] `List(items) { item in }` → `LazyColumn { items(items) { item in } }`
- [ ] Always provide `key = { }` for stable identities

#### Android → iOS (Large Lists)
```kotlin
// ANDROID - LazyColumn (efficient)
LazyColumn(
    modifier = Modifier.fillMaxSize(),
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
    items(items, key = { it.id }) { item ->
        ItemRow(item)
    }
}
```

**Adapts to:**
```swift
// iOS - List (efficient, native feel)
List(items, id: \.id) { item in
    ItemRow(item)
}
.listStyle(.plain)
```

**Rules**:
- [ ] `LazyColumn { items(...) { } }` → `List(items) { }`
- [ ] `Arrangement.spacedBy(8.dp)` → List handles spacing natively
- [ ] Use `.listStyle(.plain)` for plain list (no grouped styling)

---

## 2. State Management Adaptation Rules

### 2.1 State ↔ @State/@StateObject

#### Android → iOS
```kotlin
// ANDROID - remember
@Composable
fun LoginScreen() {
    var email by remember { mutableStateOf("") }
    var isLoading by remember { mutableStateOf(false) }

    TextField(value = email, onValueChange = { email = it })
}
```

**Adapts to:**
```swift
// iOS - @State
struct LoginScreen: View {
    @State var email = ""
    @State var isLoading = false

    var body: some View {
        TextField("Email", text: $email)
    }
}
```

**Rules**:
- [ ] `var x by remember { mutableStateOf(value) }` → `@State var x = value`
- [ ] `x` (read) → `x` (same)
- [ ] `x = newValue` (write) → `$x` in bindings, or `x = newValue` in callbacks

#### iOS → Android
```swift
// iOS - @State
struct LoginForm: View {
    @State var password = ""
    @StateObject var viewModel = LoginViewModel()

    var body: some View {
        TextField("Password", text: $password)
    }
}
```

**Adapts to:**
```kotlin
// ANDROID - remember + ViewModel
@Composable
fun LoginForm(viewModel: LoginViewModel = viewModel()) {
    var password by remember { mutableStateOf("") }

    TextField(
        value = password,
        onValueChange = { password = it }
    )
}
```

**Rules**:
- [ ] `@State var x = value` → `var x by remember { mutableStateOf(value) }`
- [ ] `@StateObject var vm = ViewModel()` → `viewModel: ViewModel = viewModel()` parameter
- [ ] `$binding` → `onValueChange = { x = it }`

### 2.2 ObservedObject ↔ collectAsState()

#### iOS → Android
```swift
// iOS - @ObservedObject
class EventViewModel: ObservableObject {
    @Published var events: [Event] = []
}

struct EventList: View {
    @ObservedObject var viewModel: EventViewModel

    var body: some View {
        List(viewModel.events) { event in
            EventRow(event)
        }
    }
}
```

**Adapts to:**
```kotlin
// ANDROID - collectAsState() with Flow
class EventViewModel : ViewModel() {
    val events: StateFlow<List<Event>> = // ...
}

@Composable
fun EventList(viewModel: EventViewModel = viewModel()) {
    val events by viewModel.events.collectAsState()

    LazyColumn {
        items(events, key = { it.id }) { event ->
            EventRow(event)
        }
    }
}
```

**Rules**:
- [ ] `@ObservedObject var vm: ObservableObject` → `viewModel: ViewModel` parameter
- [ ] `@Published var x` → `val x: StateFlow<T>` or `val x: Flow<T>`
- [ ] `viewModel.x` → `val x by viewModel.x.collectAsState()`

#### Android → iOS
```kotlin
// ANDROID - Flow + collectAsState()
class EventViewModel : ViewModel() {
    val events: StateFlow<List<Event>> = // ...
}

@Composable
fun EventList(viewModel: EventViewModel = viewModel()) {
    val events by viewModel.events.collectAsState()

    LazyColumn {
        items(events) { event ->
            EventRow(event)
        }
    }
}
```

**Adapts to:**
```swift
// iOS - @ObservedObject + @Published
class EventViewModel: ObservableObject {
    @Published var events: [Event] = []
}

struct EventList: View {
    @ObservedObject var viewModel: EventViewModel

    var body: some View {
        List(viewModel.events) { event in
            EventRow(event)
        }
    }
}
```

**Rules**:
- [ ] `val x by viewModel.x.collectAsState()` → `@ObservedObject var viewModel`
- [ ] `viewModel.x` (read in compose) → `viewModel.x` (auto-subscribes)
- [ ] `StateFlow<T>` → `@Published var x: T`

---

## 3. Modifier/View Adaptation Rules

### 3.1 Modifier ↔ View Modifiers

#### Android → iOS

| Android (Modifier) | iOS (View Modifier) | Example |
|--------------------|-------------------|---------|
| `.background(color)` | `.background(color)` | Same |
| `.padding(16.dp)` | `.padding(16)` | dp → pt |
| `.size(48.dp)` | `.frame(width: 48, height: 48)` | Different API |
| `.border(2.dp, color)` | `.border(2, color)` | Same |
| `.clickable { }` | `Button(action: { })` | Button for interactivity |
| `.clip(shape)` | `.cornerRadius(8)` | RoundedCornerShape → cornerRadius |
| `.shadow(elevation: 4.dp)` | `.shadow(radius: 4)` | Similar |
| `.alpha(0.5f)` | `.opacity(0.5)` | Similar |

#### Examples

**ANDROID**:
```kotlin
Box(
    modifier = Modifier
        .size(100.dp)
        .background(Color.Blue)
        .padding(16.dp)
        .clip(RoundedCornerShape(8.dp))
        .border(2.dp, Color.Black)
)
```

**iOS**:
```swift
Box()
    .frame(width: 100, height: 100)
    .background(Color.blue)
    .padding(16)
    .cornerRadius(8)
    .border(Color.black, width: 2)
```

### 3.2 Semantics ↔ Accessibility Modifiers

#### Android → iOS

| Android (Semantics) | iOS (Accessibility) | Purpose |
|-------------------|-------------------|---------|
| `contentDescription = "..."` | `.accessibilityLabel(...)` | Element name |
| `contentDescription = null` | Implicit (system) | Icon-only, use system role |
| `semantics { role = Role.Button }` | `Button(action:) { }` | Define interaction type |
| `LiveRegionMode.Assertive` | `UIAccessibility.post(notification:)` | Announce changes |

#### Examples

**ANDROID**:
```kotlin
Icon(
    imageVector = Icons.Default.Close,
    contentDescription = "Close dialog",  // Accessibility label
    modifier = Modifier.clickable { closeDialog() }
)

Button(
    onClick = { action() },
    modifier = Modifier.semantics {
        contentDescription = "Submit form"
    }
)
```

**iOS**:
```swift
Button(action: { closeDialog() }) {
    Image(systemName: "xmark")
}
.accessibilityLabel("Close dialog")  // Accessibility label

Button(action: { action() }) {
    Text("Submit form")
}
.accessibilityLabel("Submit form")
```

---

## 4. Navigation Adaptation Rules

### 4.1 Navigation Stack

#### Android → iOS

**ANDROID** (NavController):
```kotlin
@Composable
fun MyApp() {
    val navController = rememberNavController()

    NavHost(navController = navController, startDestination = "login") {
        composable("login") { LoginScreen(navController) }
        composable("home") { HomeScreen(navController) }
    }
}

// Navigate
navController.navigate("home")
navController.popBackStack()
```

**iOS** (NavigationStack):
```swift
@main
struct MyApp: App {
    @State var navigationPath = NavigationPath()

    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $navigationPath) {
                LoginScreen(path: $navigationPath)
                    .navigationDestination(for: String.self) { route in
                        switch route {
                        case "home":
                            HomeScreen(path: $navigationPath)
                        default:
                            EmptyView()
                        }
                    }
            }
        }
    }
}

// Navigate
navigationPath.append("home")
navigationPath.removeLast()
```

**Rules**:
- [ ] `NavController` → `@State var navigationPath: NavigationPath`
- [ ] `NavHost + composable` → `NavigationStack + navigationDestination`
- [ ] `navigate(route)` → `navigationPath.append(route)`
- [ ] `popBackStack()` → `navigationPath.removeLast()`

### 4.2 Sheet/Dialog

#### Android → iOS

**ANDROID** (LaunchedEffect + Dialog):
```kotlin
@Composable
fun EventDetail(event: Event) {
    var showDeleteDialog by remember { mutableStateOf(false) }

    if (showDeleteDialog) {
        AlertDialog(
            onDismissRequest = { showDeleteDialog = false },
            title = { Text("Delete?") },
            confirmButton = {
                Button(onClick = { /* delete */ }) {
                    Text("Delete")
                }
            }
        )
    }

    Button(onClick = { showDeleteDialog = true }) {
        Text("Delete Event")
    }
}
```

**iOS** (Modals):
```swift
struct EventDetail: View {
    let event: Event
    @State var showDeleteDialog = false

    var body: some View {
        VStack {
            Button(action: { showDeleteDialog = true }) {
                Text("Delete Event")
            }
        }
        .confirmationDialog("Delete?", isPresented: $showDeleteDialog) {
            Button("Delete", role: .destructive) {
                // delete
            }
            Button("Cancel", role: .cancel) { }
        }
    }
}
```

**Rules**:
- [ ] `AlertDialog` → `.confirmationDialog()` or `.alert()`
- [ ] `if (condition) { Dialog }` → `.sheet(isPresented: $condition)`
- [ ] `onDismissRequest` → `isPresented` binding

---

## 5. Theme & Styling Adaptation Rules

### 5.1 Color System

#### Android (Material Design 3)
```kotlin
MaterialTheme(colorScheme = darkColorScheme) {
    Surface(color = MaterialTheme.colorScheme.surface) {
        Text(
            text = "Hello",
            color = MaterialTheme.colorScheme.onSurface
        )
    }
}
```

#### iOS (Semantic Colors)
```swift
ZStack {
    Color(.systemBackground)  // Adapts to dark mode automatically

    Text("Hello")
        .foregroundColor(.label)  // Adapts to dark mode
}
```

**Rules**:
- [ ] `MaterialTheme.colorScheme.primary` → `.accentColor` or `Color("Primary")`
- [ ] `MaterialTheme.colorScheme.onSurface` → `.label`
- [ ] `MaterialTheme.colorScheme.surface` → `Color(.systemBackground)`
- [ ] `MaterialTheme.colorScheme.surfaceVariant` → `Color(.systemGray6)`
- [ ] Dark mode is automatic in iOS (use semantic colors)

### 5.2 Typography

#### Android (Material Design 3)
```kotlin
Text(
    text = "Headline",
    style = MaterialTheme.typography.headlineMedium
)

Text(
    text = "Body",
    style = MaterialTheme.typography.bodyLarge
)
```

#### iOS (Dynamic Type)
```swift
Text("Headline")
    .font(.headline)

Text("Body")
    .font(.body)
```

**Rules**:
- [ ] `displayMedium` → `.title` or `.largeTitle`
- [ ] `headlineMedium` → `.headline`
- [ ] `bodyLarge` → `.body`
- [ ] `bodySmall` → `.subheadline` or `.caption`
- [ ] Always use semantic fonts (not `Font.system(size: 16)`)

---

## 6. Data Binding & Input Adaptation Rules

### 6.1 TextField ↔ TextField (Different APIs)

#### Android
```kotlin
@Composable
fun LoginScreen(viewModel: LoginViewModel = viewModel()) {
    val email by viewModel.email.collectAsState()

    TextField(
        value = email,
        onValueChange = { viewModel.updateEmail(it) },
        label = { Text("Email") },
        isError = email.isEmpty(),
        supportingText = {
            if (email.isEmpty()) Text("Email required")
        }
    )
}
```

#### iOS
```swift
struct LoginScreen: View {
    @ObservedObject var viewModel = LoginViewModel()

    var body: some View {
        VStack {
            TextField("Email", text: $viewModel.email)

            if viewModel.email.isEmpty {
                Text("Email required")
                    .font(.caption)
                    .foregroundColor(.red)
            }
        }
    }
}
```

**Rules**:
- [ ] `TextField(value:, onValueChange:)` → `TextField("label", text: $binding)`
- [ ] `supportingText` → Separate `Text` below field
- [ ] `isError` → Apply border color conditionally

### 6.2 Checkbox ↔ Toggle

#### Android
```kotlin
@Composable
fun SettingsScreen(viewModel: SettingsViewModel = viewModel()) {
    var notificationsEnabled by remember { mutableStateOf(true) }

    Checkbox(
        checked = notificationsEnabled,
        onCheckedChange = { notificationsEnabled = it },
        modifier = Modifier.semantics {
            contentDescription = "Enable notifications"
        }
    )
}
```

#### iOS
```swift
struct SettingsScreen: View {
    @State var notificationsEnabled = true

    var body: some View {
        Toggle("Enable notifications", isOn: $notificationsEnabled)
    }
}
```

**Rules**:
- [ ] `Checkbox` → `Toggle`
- [ ] `checked` → `isOn`
- [ ] `onCheckedChange` → Binding `$`

---

## 7. Image & Media Adaptation Rules

### 7.1 Icons

#### Android
```kotlin
Icon(
    imageVector = Icons.Default.Close,
    contentDescription = "Close",
    modifier = Modifier.size(24.dp),
    tint = Color.Black
)
```

#### iOS
```swift
Image(systemName: "xmark")
    .frame(width: 24, height: 24)
    .foregroundColor(.black)
```

**Rules**:
- [ ] `Icons.Default.*` → `Image(systemName: "...")`
- [ ] `tint` → `.foregroundColor()`
- [ ] `contentDescription` → `.accessibilityLabel()`
- [ ] Use SF Symbols on iOS (1:1 with Material Icons)

### 7.2 Images

#### Android
```kotlin
@Composable
fun UserAvatar(url: String) {
    AsyncImage(
        model = url,
        contentDescription = "User avatar",
        modifier = Modifier
            .size(48.dp)
            .clip(CircleShape),
        contentScale = ContentScale.Crop
    )
}
```

#### iOS
```swift
struct UserAvatar: View {
    let url: String

    var body: some View {
        AsyncImage(url: URL(string: url)) { image in
            image
                .resizable()
                .scaledToFill()
        } placeholder: {
            Circle().fill(Color.gray)
        }
        .frame(width: 48, height: 48)
        .clipShape(Circle())
        .accessibilityLabel("User avatar")
    }
}
```

**Rules**:
- [ ] `AsyncImage` → `AsyncImage` (iOS 15+)
- [ ] `clip(CircleShape)` → `.clipShape(Circle())`
- [ ] `contentScale = ContentScale.Crop` → `.scaledToFill()`

---

## 8. Animation & Gesture Adaptation Rules

### 8.1 Click Handling

#### Android
```kotlin
Box(
    modifier = Modifier
        .size(48.dp)
        .clickable { /* action */ }
)
```

#### iOS
```swift
Button(action: { /* action */ }) {
    Box()
        .frame(width: 48, height: 48)
}
```

**Rule**: Always use `Button` for interactive elements on iOS, not `.onTapGesture()`.

### 8.2 Animations

#### Android
```kotlin
@Composable
fun AnimatedBox(isExpanded: Boolean) {
    val animatedHeight by animateDpAsState(
        targetValue = if (isExpanded) 200.dp else 100.dp
    )

    Box(
        modifier = Modifier.height(animatedHeight)
    )
}
```

#### iOS
```swift
struct AnimatedBox: View {
    let isExpanded: Bool

    var body: some View {
        Box()
            .frame(height: isExpanded ? 200 : 100)
            .animation(.default, value: isExpanded)
    }
}
```

**Rules**:
- [ ] `animateDpAsState()` → `.animation(.default, value:)`
- [ ] Explicit animations → `.transition()`

---

## 9. Platform-Specific Feature Handling

### 9.1 Safe Area & Notch (iOS Only)

**NO EQUIVALENT IN ANDROID** - Android handles system insets automatically.

#### iOS Specific
```swift
VStack {
    HStack {
        Text("Centered Title")
    }
    .padding(.top, 16)
    .ignoresSafeArea(.keyboard)  // Allow keyboard to cover if needed

    Spacer()

    BottomBar()
        .ignoresSafeArea(.all)  // Extend to edges
}
```

**Rules**:
- [ ] Only use `.ignoresSafeArea()` when intentional (full-bleed designs)
- [ ] Default: Respect safe areas
- [ ] Android: No equivalent needed, system handles insets

### 9.2 Haptic Feedback (iOS)

**ANDROID**: Use `Vibrator` API

**iOS**: Use `UIImpactFeedbackGenerator`

#### Android
```kotlin
val vibrator = context.getSystemService(Context.VIBRATOR_SERVICE) as Vibrator
vibrator.vibrate(50)  // 50ms buzz
```

#### iOS
```swift
import UIKit

let impact = UIImpactFeedbackGenerator(style: .medium)
impact.impactOccurred()
```

---

## 10. Quick Checklist: Android to iOS

Before marking adaptation complete:

- [ ] **Layout**: Column → VStack, Row → HStack, Modifier → View modifiers
- [ ] **State**: `remember` → `@State`, `collectAsState` → `@ObservedObject`
- [ ] **Colors**: MaterialTheme → semantic colors (`.label`, `.systemBackground`)
- [ ] **Typography**: Material styles → Dynamic Type (`.headline`, `.body`)
- [ ] **Touch**: 48dp → 44pt minimum
- [ ] **Accessibility**: `contentDescription` → `accessibilityLabel`
- [ ] **Navigation**: NavController → NavigationStack
- [ ] **Dialogs**: AlertDialog → `.confirmationDialog()` or `.alert()`
- [ ] **Forms**: TextField API adapted for iOS binding style
- [ ] **Icons**: Material Icons → SF Symbols
- [ ] **Testing**: Screen reader set to VoiceOver

---

## 11. Quick Checklist: iOS to Android

Before marking adaptation complete:

- [ ] **Layout**: VStack → Column, HStack → Row, View modifiers → Modifier
- [ ] **State**: `@State` → `remember`, `@ObservedObject` → `collectAsState`
- [ ] **Colors**: Semantic colors → MaterialTheme tokens
- [ ] **Typography**: Dynamic Type → Material Design 3 styles
- [ ] **Touch**: 44pt → 48dp minimum
- [ ] **Accessibility**: `accessibilityLabel` → `contentDescription`
- [ ] **Navigation**: NavigationStack → NavController
- [ ] **Dialogs**: Confirmation dialog → AlertDialog
- [ ] **Forms**: TextField binding adapted for Compose API
- [ ] **Icons**: SF Symbols → Material Icons
- [ ] **Testing**: Screen reader set to TalkBack

---

## 12. Common Mistakes to Avoid

| Mistake | Why Bad | Fix |
|---------|--------|-----|
| Literal translation (copy paste) | Code doesn't follow platform idioms | Understand each platform's patterns |
| ScrollView + large lists on iOS | Poor performance | Use `List` or `LazyVStack` |
| LazyColumn on small lists on Android | Unnecessary complexity | Use `Column` for < 10 items |
| Hardcoded colors instead of semantic | Breaks dark mode, theming | Use `MaterialTheme` / `.label` |
| Mixing Material + HIG | Inconsistent, confusing UX | Stick to one design system per platform |
| Not testing with screen readers | Accessibility broken | Test with TalkBack/VoiceOver |
| Same spacing on both platforms | Looks wrong (different scales) | Adapt 48dp → 44pt mindfully |
| Button from Box.clickable on iOS | Loses accessibility features | Use `Button(action:)` |
| Ignoring Dynamic Type on iOS | Text doesn't scale for accessibility | Always use semantic font styles |


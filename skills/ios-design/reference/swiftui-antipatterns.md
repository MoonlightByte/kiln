# SwiftUI Anti-Patterns Database

LLM-generated code frequently contains these mistakes. Use this reference to catch and fix them.

---

## State Management (Critical)

### AP-001: @ObservedObject for Owned Instances

**Severity**: 🔴 Critical | **Confidence**: 95

**Description**: Using `@ObservedObject` for objects created within the view causes them to be recreated on every view update.

**Bad Example**:
```swift
struct ContentView: View {
    @ObservedObject var viewModel = ContentViewModel() // Recreated on every update!

    var body: some View {
        Text(viewModel.text)
        Button("Update") { viewModel.update() }
    }
}
```

**Good Example**:
```swift
struct ContentView: View {
    @StateObject private var viewModel = ContentViewModel() // Created once

    var body: some View {
        Text(viewModel.text)
        Button("Update") { viewModel.update() }
    }
}
```

**iOS 17+ Alternative**:
```swift
@Observable
final class ContentViewModel {
    var text = ""

    func update() { /* ... */ }
}

struct ContentView: View {
    @State private var viewModel = ContentViewModel() // @Observable works with @State!

    var body: some View {
        Text(viewModel.text)
        Button("Update") { viewModel.update() }
    }
}
```

**Rationale**: `@StateObject` preserves the object across view updates. Use `@ObservedObject` only for objects passed in from a parent. With iOS 17+, use `@Observable` macro and `@State` instead.

**Scanner Pattern**: `@ObservedObject\s+(?:var|let)\s+\w+\s*=`
- Flag any `@ObservedObject` with initialization (the `=` sign indicates creation)
- Suggest `@StateObject` replacement
- For iOS 17+ projects, suggest `@Observable` + `@State`

---

### AP-002: @State with Reference Types

**Severity**: 🔴 Critical | **Confidence**: 95

**Description**: Using `@State` with reference types (classes) doesn't trigger view updates when properties change.

**Bad Example**:
```swift
class UserData {
    var name: String = ""
}

struct ProfileView: View {
    @State private var userData = UserData() // Won't update UI!

    var body: some View {
        TextField("Name", text: $userData.name) // No binding works here
    }
}
```

**Good Example**:
```swift
// Option 1: Use struct
struct UserData {
    var name: String = ""
}

struct ProfileView: View {
    @State private var userData = UserData()

    var body: some View {
        TextField("Name", text: $userData.name)
    }
}

// Option 2: Use ObservableObject
class UserData: ObservableObject {
    @Published var name: String = ""
}

struct ProfileView: View {
    @StateObject private var userData = UserData()

    var body: some View {
        TextField("Name", text: $userData.name)
    }
}

// Option 3: iOS 17+ @Observable
@Observable
class UserData {
    var name: String = ""
}

struct ProfileView: View {
    @State private var userData = UserData() // Works with @Observable!

    var body: some View {
        TextField("Name", text: $userData.name)
    }
}
```

**Rationale**: `@State` is designed for value types. For reference types, use `@StateObject` with `ObservableObject` or `@Observable` macro (iOS 17+).

---

### AP-003: Missing @Published in ObservableObject

**Severity**: 🔴 Critical | **Confidence**: 90

**Description**: Properties in `ObservableObject` without `@Published` won't trigger view updates.

**Bad Example**:
```swift
class ViewModel: ObservableObject {
    var count = 0 // Missing @Published!

    func increment() {
        count += 1 // View won't update
    }
}
```

**Good Example**:
```swift
class ViewModel: ObservableObject {
    @Published var count = 0

    func increment() {
        count += 1 // View updates automatically
    }
}
```

**Rationale**: `@Published` triggers `objectWillChange` publisher. Without it, SwiftUI doesn't know the property changed.

---

### AP-004: Modifying @State During View Initialization

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Trying to modify `@State` properties during `init()` or in property initializers.

**Bad Example**:
```swift
struct ContentView: View {
    @State private var text: String

    init(initialText: String) {
        text = initialText // Warning: modifying state during init
    }
}
```

**Good Example**:
```swift
struct ContentView: View {
    @State private var text: String

    init(initialText: String) {
        _text = State(initialValue: initialText) // Use State wrapper
    }
}
```

**Rationale**: Use the underscore-prefixed property to access the underlying `State` wrapper during initialization.

---

## Performance (High)

### AP-005: AnyView Type Erasure Overuse

**Severity**: 🟠 Important | **Confidence**: 90

**Description**: Using `AnyView` to return different view types kills SwiftUI's ability to diff views efficiently. State is lost, animations break, and performance suffers.

**Bad Example**:
```swift
func makeContent() -> AnyView {
    if showImage {
        return AnyView(Image(systemName: "star"))
    } else {
        return AnyView(Text("No image"))
    }
}

// Every call rebuilds the view tree from scratch
var body: some View {
    makeContent() // View hierarchy is recreated
}
```

**Good Example**:
```swift
@ViewBuilder
func makeContent() -> some View {
    if showImage {
        Image(systemName: "star")
    } else {
        Text("No image")
    }
}

// Or use Group
var content: some View {
    Group {
        if showImage {
            Image(systemName: "star")
        } else {
            Text("No image")
        }
    }
}

// @ViewBuilder in properties
@ViewBuilder
var content: some View {
    if condition {
        VStack { /* ... */ }
    } else {
        HStack { /* ... */ }
    }
}
```

**Rationale**: `@ViewBuilder` lets SwiftUI see the actual view types and diff them efficiently. `AnyView` hides this information, destroying view identity and state.

**Scanner Pattern**: `AnyView\s*\(\s*`
- Flag any `AnyView(` usage
- Suggest `@ViewBuilder` replacement
- Check if conditionals could be moved to `@ViewBuilder` function or property
- Alert: This is a performance-critical pattern

---

### AP-006: VStack for Long Lists

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Using `VStack` inside `ScrollView` for lists of more than 20 items creates all views upfront.

**Bad Example**:
```swift
ScrollView {
    VStack {
        ForEach(items) { item in // All 500 items created at once!
            ItemRow(item: item)
        }
    }
}
```

**Good Example**:
```swift
ScrollView {
    LazyVStack {
        ForEach(items) { item in // Only visible items created
            ItemRow(item: item)
        }
    }
}

// Or use List for standard appearance
List(items) { item in
    ItemRow(item: item)
}
```

**Rationale**: `LazyVStack` and `List` only create views as they come on screen, enabling smooth scrolling.

---

### AP-007: Missing .id() for View Identity Issues

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: SwiftUI can't properly track view identity when data changes, causing animation glitches or state loss.

**Bad Example**:
```swift
ForEach(items.indices, id: \.self) { index in // Index-based ID breaks on reorder
    ItemRow(item: items[index])
}
```

**Good Example**:
```swift
ForEach(items, id: \.id) { item in // Stable ID
    ItemRow(item: item)
}

// For complex identity changes, use .id()
ScrollView {
    content
}
.id(refreshID) // Force recreation when refreshID changes
```

**Rationale**: Stable IDs let SwiftUI track which item is which across updates, enabling proper animations and state preservation.

---

### AP-008: Heavy Computation in View Body

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Performing expensive operations directly in the view body runs on every update.

**Bad Example**:
```swift
var body: some View {
    let sortedItems = items.sorted { $0.date > $1.date } // Every update!
    let grouped = Dictionary(grouping: sortedItems) { $0.category } // Every update!

    List {
        ForEach(grouped.keys.sorted(), id: \.self) { key in
            Section(key) {
                ForEach(grouped[key]!) { item in
                    ItemRow(item: item)
                }
            }
        }
    }
}
```

**Good Example**:
```swift
// Move to ViewModel
class ItemsViewModel: ObservableObject {
    @Published private(set) var groupedItems: [(String, [Item])] = []

    func processItems(_ items: [Item]) {
        let sorted = items.sorted { $0.date > $1.date }
        groupedItems = Dictionary(grouping: sorted) { $0.category }
            .sorted { $0.key < $1.key }
    }
}

// View is clean
var body: some View {
    List {
        ForEach(viewModel.groupedItems, id: \.0) { category, items in
            Section(category) {
                ForEach(items) { item in
                    ItemRow(item: item)
                }
            }
        }
    }
}
```

**Rationale**: View body runs frequently. Move computation to ViewModel or use `.task` modifier for async work.

---

### AP-009: Ignoring .task for Async Work

**Severity**: 🟡 Warning | **Confidence**: 75

**Description**: Using `onAppear` for async work instead of the built-in `.task` modifier.

**Bad Example**:
```swift
struct ContentView: View {
    @State private var data: [Item] = []

    var body: some View {
        List(data) { item in
            ItemRow(item: item)
        }
        .onAppear {
            Task {
                data = await fetchData() // Manual Task management
            }
        }
    }
}
```

**Good Example**:
```swift
struct ContentView: View {
    @State private var data: [Item] = []

    var body: some View {
        List(data) { item in
            ItemRow(item: item)
        }
        .task {
            data = await fetchData() // Automatic cancellation on disappear
        }
    }
}
```

**Rationale**: `.task` automatically cancels when the view disappears. `onAppear` + manual `Task` can leak.

---

## Accessibility (Critical)

### AP-010: Missing accessibilityLabel

**Severity**: 🔴 Critical | **Confidence**: 95

**Description**: Interactive elements without accessibility labels are invisible to VoiceOver.

**Bad Example**:
```swift
Button(action: { deleteItem() }) {
    Image(systemName: "trash")
}

Image(systemName: "star.fill")
    .onTapGesture { toggleFavorite() }
```

**Good Example**:
```swift
Button(action: { deleteItem() }) {
    Image(systemName: "trash")
}
.accessibilityLabel("Delete item")

Image(systemName: "star.fill")
    .onTapGesture { toggleFavorite() }
    .accessibilityLabel(isFavorite ? "Remove from favorites" : "Add to favorites")
    .accessibilityAddTraits(.isButton)
```

**Rationale**: VoiceOver reads accessibility labels. SF Symbols don't have automatic labels for all icons.

---

### AP-011: Small Touch Targets

**Severity**: 🔴 Critical | **Confidence**: 90

**Description**: Interactive elements smaller than 44x44 points fail Apple HIG requirements.

**Bad Example**:
```swift
Image(systemName: "xmark")
    .font(.system(size: 16))
    .onTapGesture { dismiss() }
```

**Good Example**:
```swift
Button(action: { dismiss() }) {
    Image(systemName: "xmark")
        .font(.system(size: 16))
}
.frame(width: 44, height: 44)

// Or use contentShape
Image(systemName: "xmark")
    .font(.system(size: 16))
    .frame(width: 44, height: 44)
    .contentShape(Rectangle())
    .onTapGesture { dismiss() }
```

**Rationale**: 44pt is Apple's minimum touch target. Smaller targets are hard to tap accurately.

---

### AP-012: Decorative Images Announced by VoiceOver

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: Decorative images that add no information are still announced by VoiceOver.

**Bad Example**:
```swift
HStack {
    Image(systemName: "chevron.right") // VoiceOver: "chevron right, image"
    Text("Settings")
}
```

**Good Example**:
```swift
HStack {
    Image(systemName: "chevron.right")
        .accessibilityHidden(true) // Purely decorative
    Text("Settings")
}

// Or group with the text
HStack {
    Image(systemName: "chevron.right")
    Text("Settings")
}
.accessibilityElement(children: .combine)
```

**Rationale**: Decorative elements clutter VoiceOver navigation. Hide them or combine with meaningful content.

---

## Styling (Important)

### AP-013: Hardcoded Colors

**Severity**: 🟠 Important | **Confidence**: 90

**Description**: Using explicit color values instead of semantic colors breaks dark mode.

**Bad Example**:
```swift
Text("Title")
    .foregroundColor(.black) // Invisible in dark mode!

Rectangle()
    .fill(Color(red: 0.95, green: 0.95, blue: 0.95))
```

**Good Example**:
```swift
Text("Title")
    .foregroundColor(.label) // Adapts to light/dark

Rectangle()
    .fill(Color(.systemBackground))

// Or use semantic system colors
Text("Warning")
    .foregroundColor(.orange)
```

**Rationale**: Semantic colors automatically adapt to light/dark mode and accessibility settings.

---

### AP-014: Hardcoded Font Sizes

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Using `.font(.system(size:))` instead of text styles breaks Dynamic Type.

**Bad Example**:
```swift
Text("Heading")
    .font(.system(size: 24, weight: .bold))

Text("Body text")
    .font(.system(size: 16))
```

**Good Example**:
```swift
Text("Heading")
    .font(.title)

Text("Body text")
    .font(.body)

// Custom sizes that scale
@ScaledMetric(relativeTo: .body) var customSize: CGFloat = 18

Text("Custom")
    .font(.system(size: customSize))
```

**Rationale**: Text styles scale with Dynamic Type. Hardcoded sizes don't, breaking accessibility.

---

### AP-015: Wrong Modifier Order

**Severity**: 🟡 Warning | **Confidence**: 80

**Description**: Modifier order matters in SwiftUI. Wrong order produces unexpected layouts.

**Bad Example**:
```swift
// Background only covers text, not padding
Text("Hello")
    .background(Color.blue)
    .padding()

// Tap area doesn't include full width
Text("Tap me")
    .onTapGesture { /* action */ }
    .frame(maxWidth: .infinity)
```

**Good Example**:
```swift
// Padding inside background
Text("Hello")
    .padding()
    .background(Color.blue)

// Tap area spans full width
Text("Tap me")
    .frame(maxWidth: .infinity)
    .contentShape(Rectangle())
    .onTapGesture { /* action */ }
```

**Rationale**: Modifiers wrap views. Order determines what's inside what.

---

## Navigation (Important)

### AP-016: NavigationLink Performance Issues

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: Using NavigationLink with destination that creates complex views eagerly.

**Bad Example**:
```swift
List(items) { item in
    NavigationLink(destination: DetailView(item: item)) { // All DetailViews created!
        ItemRow(item: item)
    }
}
```

**Good Example**:
```swift
// iOS 16+
List(items) { item in
    NavigationLink(value: item) {
        ItemRow(item: item)
    }
}
.navigationDestination(for: Item.self) { item in
    DetailView(item: item) // Created only when navigating
}

// Or use lazy loading
List(items) { item in
    NavigationLink {
        DetailView(item: item) // Trailing closure is lazy
    } label: {
        ItemRow(item: item)
    }
}
```

**Rationale**: NavigationLink's destination parameter is eager. Use value-based navigation or trailing closure for lazy creation.

---

## GeometryReader (Important)

### AP-017: GeometryReader Breaking Layout

**Severity**: 🟠 Important | **Confidence**: 75

**Description**: GeometryReader expands to fill available space, disrupting expected layout.

**Bad Example**:
```swift
VStack {
    Text("Header")
    GeometryReader { geo in // Pushes content, takes all space
        Image("photo")
            .resizable()
            .frame(width: geo.size.width)
    }
    Text("Footer") // Hidden or pushed off screen
}
```

**Good Example**:
```swift
VStack {
    Text("Header")
    Image("photo")
        .resizable()
        .aspectRatio(contentMode: .fit)
    Text("Footer")
}

// When GeometryReader is needed, constrain it
VStack {
    Text("Header")
    GeometryReader { geo in
        Image("photo")
            .resizable()
            .frame(width: geo.size.width, height: geo.size.width * 0.75)
    }
    .frame(height: 200) // Constrain GeometryReader
    Text("Footer")
}
```

**Rationale**: GeometryReader should be used sparingly and constrained. Prefer built-in layout modifiers.

---

## Async/Concurrency (Important)

### AP-018: @MainActor Missing for UI Updates

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Updating `@Published` properties from background threads causes crashes or undefined behavior.

**Bad Example**:
```swift
class ViewModel: ObservableObject {
    @Published var items: [Item] = []

    func fetchItems() async {
        let result = await api.getItems() // Background thread
        items = result // Publishing changes from background thread!
    }
}
```

**Good Example**:
```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []

    func fetchItems() async {
        let result = await api.getItems()
        items = result // Safe, we're @MainActor
    }
}

// Or await MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []

    func fetchItems() async {
        let result = await api.getItems()
        await MainActor.run {
            items = result
        }
    }
}
```

**Rationale**: UI updates must happen on the main thread. `@MainActor` ensures this.

---

### AP-019: Ignoring Task Cancellation

**Severity**: 🟡 Warning | **Confidence**: 70

**Description**: Long-running tasks that don't check for cancellation waste resources.

**Bad Example**:
```swift
func processAllItems() async {
    for item in items {
        await processItem(item) // Continues even if view disappeared
    }
}
```

**Good Example**:
```swift
func processAllItems() async throws {
    for item in items {
        try Task.checkCancellation() // Throws if cancelled
        await processItem(item)
    }
}

// Or check isCancelled
func processAllItems() async {
    for item in items {
        if Task.isCancelled { return }
        await processItem(item)
    }
}
```

**Rationale**: When views disappear, `.task` cancels its work. Respecting cancellation prevents wasted computation.

---

### AP-020: Force Unwrapping Optionals

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Using `!` to force unwrap optionals causes crashes when value is nil.

**Bad Example**:
```swift
struct UserView: View {
    var user: User?

    var body: some View {
        Text(user!.name) // Crash if user is nil!
    }
}
```

**Good Example**:
```swift
struct UserView: View {
    var user: User?

    var body: some View {
        if let user = user {
            Text(user.name)
        } else {
            Text("No user")
        }
    }
}

// Or use optional chaining with default
Text(user?.name ?? "Unknown")
```

**Rationale**: Optional binding and nil coalescing prevent runtime crashes.

---

### AP-021: Missing @Observable Macro (iOS 17+)

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: On iOS 17+, ObservableObject with @Published is deprecated. Classes should use `@Observable` macro for cleaner code and better performance.

**Bad Example (iOS 17+ projects)**:
```swift
// Outdated pattern
class ViewModel: ObservableObject {
    @Published var count = 0
    @Published var name = ""

    func increment() {
        count += 1
    }
}

struct ContentView: View {
    @StateObject private var viewModel = ViewModel()

    var body: some View {
        Text("\(viewModel.count)")
    }
}
```

**Good Example (iOS 17+)**:
```swift
// Modern pattern
@Observable
final class ViewModel {
    var count = 0
    var name = ""

    func increment() {
        count += 1
    }
}

struct ContentView: View {
    @State private var viewModel = ViewModel()

    var body: some View {
        Text("\(viewModel.count)")
    }
}
```

**Rationale**: `@Observable` macro (iOS 17+) replaces ObservableObject entirely. It provides automatic change tracking, reduces boilerplate, and performs better. Use for all new iOS 17+ projects.

**Scanner Pattern**: `class\s+\w+:\s*ObservableObject`
- In iOS 17+ projects, flag `ObservableObject` usage
- Suggest migration to `@Observable` macro
- Check for `@Published` properties - these become regular properties in `@Observable`
- Replace `@StateObject` with `@State` when using `@Observable`

---

### AP-022: Hardcoded Colors in Dark Mode Context

**Severity**: 🔴 Critical | **Confidence**: 85

**Description**: Using explicit color values (RGB, hex) instead of semantic colors breaks dark mode support and ignores user accessibility preferences.

**Bad Example**:
```swift
Text("Title")
    .foregroundColor(Color(red: 0.1, green: 0.1, blue: 0.1)) // Always dark gray

VStack {
    content
}
.background(Color(red: 1, green: 1, blue: 1)) // Always white - invisible in dark mode!

Image("icon")
    .tint(Color(red: 0.2, green: 0.6, blue: 1)) // Custom blue
```

**Good Example**:
```swift
Text("Title")
    .foregroundColor(.label) // Adapts to light/dark

VStack {
    content
}
.background(Color(.systemBackground)) // Automatic adaptation

Image("icon")
    .tint(.accentColor) // System blue, respects accessibility
```

**Rationale**: Semantic colors automatically adapt to light/dark mode, high-contrast mode, and reduce transparency mode. Use system colors always.

**Scanner Pattern**: `Color\s*\(\s*(?:red:|\.init)` or `Color\s*\(\s*#`
- Flag `Color(red:green:blue:)` and hex color patterns
- Verify it's not a custom app brand color
- Suggest semantic color replacement: `.label`, `.secondaryLabel`, `.systemBackground`, etc.

---

### AP-023: .onAppear with Task Instead of .task

**Severity**: 🟡 Warning | **Confidence**: 80

**Description**: Using `.onAppear { Task { ... } }` instead of `.task` modifier. The latter handles cancellation better.

**Bad Example**:
```swift
struct ContentView: View {
    @State private var data: [Item] = []

    var body: some View {
        List(data) { item in
            ItemRow(item: item)
        }
        .onAppear {
            Task {
                data = await fetchData() // Doesn't auto-cancel on disappear
            }
        }
    }
}
```

**Good Example**:
```swift
struct ContentView: View {
    @State private var data: [Item] = []

    var body: some View {
        List(data) { item in
            ItemRow(item: item)
        }
        .task {
            data = await fetchData() // Auto-cancels on disappear
        }
    }
}
```

**Rationale**: `.task` automatically cancels its work when the view disappears, preventing memory leaks and unnecessary API calls. `onAppear` + manual Task management is error-prone.

**Scanner Pattern**: `.onAppear\s*\{.*Task\s*\{`
- Flag any `.onAppear` that contains `Task {`
- Suggest `.task` modifier replacement
- Check if the work should be cancelled on disappear

---

### AP-024: @StateObject Dependency Injection

**Severity**: 🔴 Critical | **Confidence**: 90

**Description**: Using `@StateObject` with dependencies passed into the view's initializer. `@StateObject` only initializes once, so subsequent changes to dependencies are ignored.

**Bad Example**:
```swift
struct UserProfileView: View {
    @StateObject var viewModel = UserProfileViewModel(userId: userId) // userId changes ignored!
    let userId: String

    init(userId: String) {
        self.userId = userId
        // viewModel is only created once, won't update when userId changes
    }

    var body: some View {
        Text(viewModel.userName)
    }
}
```

**Good Example**:
```swift
// Option 1: Use @ObservedObject for injected dependencies
struct UserProfileView: View {
    @ObservedObject var viewModel: UserProfileViewModel

    init(userId: String) {
        viewModel = UserProfileViewModel(userId: userId)
    }

    var body: some View {
        Text(viewModel.userName)
    }
}

// Option 2: Force view recreation with .id()
struct ParentView: View {
    let userId: String

    var body: some View {
        UserProfileView(userId: userId)
            .id(userId) // Forces recreation when userId changes
    }
}

// Option 3: Update viewModel when dependency changes
struct UserProfileView: View {
    @StateObject private var viewModel = UserProfileViewModel()
    let userId: String

    var body: some View {
        Text(viewModel.userName)
            .task(id: userId) {
                await viewModel.loadUser(userId)
            }
    }
}
```

**Rationale**: `@StateObject` creates and owns the object only once per view lifetime. If the initializer depends on parameters that can change, the new values will be ignored. Use `@ObservedObject` for objects created elsewhere, or use `.id()` to force view recreation.

**Scanner Pattern**: `@StateObject\s+(?:var|let)\s+\w+\s*=\s*\w+\(`
- Flag `@StateObject` with initialization using constructor arguments
- Check if constructor arguments reference view parameters
- Suggest `@ObservedObject` or `.id()` modifier

---

### AP-025: Missing [weak self] in Task

**Severity**: 🔴 Critical | **Confidence**: 90

**Description**: Creating `Task` closures that capture `self` strongly, causing memory leaks when the view or view model is deallocated before the task completes.

**Bad Example**:
```swift
class SearchViewModel: ObservableObject {
    @Published var results: [Result] = []

    func search(query: String) {
        Task {
            let data = await api.search(query) // Retains self until complete
            self.results = data // If view dismissed, self is leaked
        }
    }
}

// Or in a View
struct SearchView: View {
    @State private var results: [Result] = []

    func performSearch() {
        Task {
            results = await fetchResults() // Captures self implicitly
        }
    }
}
```

**Good Example**:
```swift
class SearchViewModel: ObservableObject {
    @Published var results: [Result] = []

    func search(query: String) {
        Task { [weak self] in
            guard let self else { return }
            let data = await api.search(query)
            self.results = data
        }
    }
}

// Or use .task modifier in Views (auto-cancels on disappear)
struct SearchView: View {
    @State private var results: [Result] = []
    @State private var searchQuery = ""

    var body: some View {
        List(results) { result in
            ResultRow(result: result)
        }
        .task(id: searchQuery) {
            results = await fetchResults(searchQuery) // Auto-cancelled
        }
    }
}
```

**Rationale**: `Task` closures capture `self` strongly by default. If the task is long-running (network, processing) and the view is dismissed, the view model remains in memory until the task completes. Use `[weak self]` or prefer `.task` modifier which handles cancellation automatically.

**Scanner Pattern**: `Task\s*\{[^}]*(?<!weak\s)self\.[^}]*\}`
- Flag `Task {` containing `self.` without `[weak self]`
- Check for long-running operations (await, network calls)
- Suggest `[weak self]` capture or `.task` modifier

---

### AP-026: ScrollView vs List Misuse

**Severity**: 🟠 Important | **Confidence**: 75

**Description**: Using `ScrollView` with `ForEach` for large data sets loads all items eagerly, causing memory issues. Using `List` when custom styling is needed forces fighting against system appearance.

**Bad Example**:
```swift
// All 500 items created immediately - memory explosion!
ScrollView {
    ForEach(items) { item in // items.count = 500
        ExpensiveItemRow(item: item)
    }
}

// Or fighting List's default styling
List(items) { item in
    ItemRow(item: item)
}
.listStyle(.plain)
.scrollContentBackground(.hidden) // iOS 16+
.background(Color.clear)
// Still has separator lines, insets, etc.
```

**Good Example**:
```swift
// For large data sets: Use LazyVStack inside ScrollView
ScrollView {
    LazyVStack(spacing: 8) {
        ForEach(items) { item in // Only visible items created
            ItemRow(item: item)
        }
    }
}

// For standard list appearance with lazy loading
List(items) { item in
    ItemRow(item: item)
}

// For custom appearance with lazy loading
ScrollView {
    LazyVStack(spacing: 0) {
        ForEach(items) { item in
            CustomItemRow(item: item)
            Divider() // Custom divider if needed
        }
    }
}
```

**Rationale**: `ScrollView` + `VStack` + `ForEach` creates all views immediately regardless of visibility. `LazyVStack` defers creation until views scroll into view. Use `List` for standard iOS appearance with built-in lazy loading, `LazyVStack` when you need custom styling.

**Scanner Pattern**: `ScrollView\s*\{[^}]*(?:VStack|ForEach)`
- Flag `ScrollView` containing `VStack` or `ForEach` without `Lazy` prefix
- Check data source size if available
- Suggest `LazyVStack` or `List` based on styling needs

---

### AP-027: EnvironmentObject Crashes

**Severity**: 🔴 Critical | **Confidence**: 85

**Description**: Using `@EnvironmentObject` without ensuring the object is injected by a parent view causes a runtime crash with no helpful error message.

**Bad Example**:
```swift
struct SettingsView: View {
    @EnvironmentObject var settings: AppSettings // Crashes if not injected!

    var body: some View {
        Toggle("Dark Mode", isOn: $settings.isDarkMode)
    }
}

// Parent forgets to inject
struct ContentView: View {
    var body: some View {
        NavigationStack {
            SettingsView() // Runtime crash: no AppSettings in environment
        }
    }
}
```

**Good Example**:
```swift
// Option 1: Always inject in parent
struct ContentView: View {
    @StateObject private var settings = AppSettings()

    var body: some View {
        NavigationStack {
            SettingsView()
        }
        .environmentObject(settings) // Inject before any child accesses it
    }
}

// Option 2: Use @Environment with custom EnvironmentKey (has default)
struct AppSettingsKey: EnvironmentKey {
    static let defaultValue = AppSettings()
}

extension EnvironmentValues {
    var appSettings: AppSettings {
        get { self[AppSettingsKey.self] }
        set { self[AppSettingsKey.self] = newValue }
    }
}

struct SettingsView: View {
    @Environment(\.appSettings) var settings // Has default, won't crash

    var body: some View {
        Toggle("Dark Mode", isOn: $settings.isDarkMode)
    }
}

// Option 3: iOS 17+ @Observable with @Environment
@Observable
final class AppSettings {
    var isDarkMode = false
}

struct SettingsView: View {
    @Environment(AppSettings.self) var settings // Cleaner syntax

    var body: some View {
        @Bindable var settings = settings
        Toggle("Dark Mode", isOn: $settings.isDarkMode)
    }
}
```

**Rationale**: `@EnvironmentObject` requires explicit injection via `.environmentObject()` modifier. If forgotten, the app crashes at runtime with a confusing error. Prefer `@Environment` with a default value, or ensure all navigation paths inject the object.

**Scanner Pattern**: `@EnvironmentObject\s+(?:var|let)\s+\w+:\s*\w+`
- Flag all `@EnvironmentObject` declarations
- Check parent views for corresponding `.environmentObject()` injection
- Suggest `@Environment` with `EnvironmentKey` for fallback safety

---

## Image Loading & Memory (Important)

### AP-028: AsyncImage Without Placeholder

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Using `AsyncImage` without placeholder or error views creates a poor user experience with blank spaces during loading and failed loads showing nothing.

**Detection Pattern** (Regex):
```regex
AsyncImage\s*\(\s*url:[^)]*\)\s*(?!\s*\{)
```

**Detection Script** (grep):
```bash
# Find AsyncImage without content closure
grep -r "AsyncImage\s*(url:" --include="*.swift" Sources/ | grep -v "placeholder:\|content:"
# Find simple AsyncImage calls
grep -r "AsyncImage(url:" --include="*.swift" Sources/ -A3 | grep -v "\.frame\|placeholder\|case"
```

**Compound Check**:
```bash
# Flag AsyncImage without phase handling
for file in $(grep -r "AsyncImage" --include="*.swift" -l Sources/); do
  if grep -q "AsyncImage(url:" "$file" && ! grep -q "placeholder:\|case .empty\|case .failure" "$file"; then
    echo "WARN: AsyncImage without placeholder in $file"
  fi
done
```

**Bad Example**:
```swift
struct EventBanner: View {
    let imageURL: URL?

    var body: some View {
        AsyncImage(url: imageURL) // Shows nothing during load or on failure!
            .frame(height: 200)
    }
}

// Also bad: modifier-only approach without phases
struct UserAvatar: View {
    let url: URL?

    var body: some View {
        AsyncImage(url: url)
            .frame(width: 48, height: 48)
            .clipShape(Circle())
        // No loading indicator, no error handling
    }
}
```

**Good Example**:
```swift
struct EventBanner: View {
    let imageURL: URL?

    var body: some View {
        AsyncImage(url: imageURL) { phase in
            switch phase {
            case .empty:
                ProgressView()
                    .frame(maxWidth: .infinity, maxHeight: .infinity)
            case .success(let image):
                image
                    .resizable()
                    .aspectRatio(contentMode: .fill)
            case .failure:
                Image(systemName: "photo")
                    .font(.largeTitle)
                    .foregroundColor(.secondary)
            @unknown default:
                EmptyView()
            }
        }
        .frame(height: 200)
        .clipped()
    }
}

// Compact version with placeholder closure
struct UserAvatar: View {
    let url: URL?

    var body: some View {
        AsyncImage(url: url) { image in
            image
                .resizable()
                .aspectRatio(contentMode: .fill)
        } placeholder: {
            Circle()
                .fill(Color.gray.opacity(0.3))
                .overlay(
                    ProgressView()
                )
        }
        .frame(width: 48, height: 48)
        .clipShape(Circle())
    }
}
```

**Rationale**: AsyncImage has three phases: `.empty` (loading), `.success` (loaded), and `.failure` (error). Without handling these, users see blank space during slow loads and no indication of failures. Always provide visual feedback for each state.

**Scanner Pattern**: `AsyncImage\s*\(\s*url:`
- Flag `AsyncImage(url:)` without trailing closure or `placeholder:` parameter
- Check for `switch phase` or `case .empty` to confirm phase handling
- Suggest adding placeholder and error views

---

### AP-029: Missing Image Caching

**Severity**: 🟠 Important | **Confidence**: 75

**Description**: Not configuring `URLCache` or using a third-party caching library causes repeated network requests for the same images, wasting bandwidth and battery.

**Detection Pattern** (Regex):
```regex
AsyncImage\s*\([^)]*url:[^)]*\)(?![^}]*URLCache|cache)
```

**Detection Script** (grep):
```bash
# Find projects using AsyncImage without custom URLSession/cache setup
grep -r "AsyncImage" --include="*.swift" Sources/ | head -5
# Check if URLCache is configured
grep -r "URLCache\|urlCache" --include="*.swift" Sources/
# Check for image caching libraries
grep -r "Kingfisher\|SDWebImage\|Nuke" Package.swift
```

**Compound Check**:
```bash
# Flag projects with AsyncImage but no cache configuration
if grep -rq "AsyncImage" --include="*.swift" Sources/; then
  if ! grep -rq "URLCache\|Kingfisher\|SDWebImage\|Nuke" Sources/ Package.swift; then
    echo "WARN: AsyncImage used without explicit cache configuration"
  fi
fi
```

**Bad Example**:
```swift
// No cache configuration - uses system defaults
struct ImageList: View {
    let imageURLs: [URL]

    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(imageURLs, id: \.self) { url in
                    AsyncImage(url: url) { image in
                        image.resizable()
                    } placeholder: {
                        ProgressView()
                    }
                    // Each scroll past reloads the image!
                }
            }
        }
    }
}
```

**Good Example**:
```swift
// Option 1: Configure URLCache at app startup
@main
struct MyApp: App {
    init() {
        // Configure 100MB memory cache, 500MB disk cache
        let cache = URLCache(
            memoryCapacity: 100 * 1024 * 1024,
            diskCapacity: 500 * 1024 * 1024,
            diskPath: "image_cache"
        )
        URLCache.shared = cache
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

// Option 2: Use Kingfisher for advanced caching
import Kingfisher

struct CachedImageView: View {
    let url: URL?

    var body: some View {
        KFImage(url)
            .resizable()
            .placeholder {
                ProgressView()
            }
            .fade(duration: 0.25)
            .cacheMemoryOnly(false) // Enable disk cache
            .aspectRatio(contentMode: .fill)
    }
}

// Option 3: Custom caching with Nuke
import NukeUI

struct NukeCachedImage: View {
    let url: URL?

    var body: some View {
        LazyImage(url: url) { state in
            if let image = state.image {
                image.resizable()
            } else if state.error != nil {
                Image(systemName: "exclamationmark.triangle")
            } else {
                ProgressView()
            }
        }
        .pipeline(ImagePipeline.shared) // Uses configured cache
    }
}
```

**Rationale**: AsyncImage uses `URLSession.shared` which has limited default caching. For image-heavy apps, configure `URLCache` with appropriate sizes or use dedicated image libraries (Kingfisher, SDWebImage, Nuke) that provide memory + disk caching, prefetching, and automatic cache eviction.

**Scanner Pattern**: Multiple files with `AsyncImage`
- Check for `URLCache` configuration in App init or AppDelegate
- Check for image caching dependencies in Package.swift or Podfile
- Suggest configuring cache or using specialized library

---

### AP-030: UIImage from Data Without Downsampling

**Severity**: 🔴 Critical | **Confidence**: 90

**Description**: Creating `UIImage` directly from `Data` or file path loads the full-resolution image into memory, causing excessive memory usage and potential OOM crashes for large images.

**Detection Pattern** (Regex):
```regex
UIImage\s*\(\s*(data:|contentsOfFile:|named:)[^)]*\)(?![^;]*(preparingThumbnail|byPreparingThumbnail|CGImageSource))
```

**Detection Script** (grep):
```bash
# Find UIImage initialization without downsampling
grep -r "UIImage(data:" --include="*.swift" Sources/
grep -r "UIImage(contentsOfFile:" --include="*.swift" Sources/
# Check if downsampling is used
grep -r "preparingThumbnail\|byPreparingThumbnail\|CGImageSource" --include="*.swift" Sources/
```

**Compound Check**:
```bash
# Flag UIImage from data without downsampling
grep -r "UIImage(data:\|UIImage(contentsOfFile:" --include="*.swift" Sources/ | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  if ! grep -q "preparingThumbnail\|CGImageSource" "$file"; then
    echo "CRITICAL: Full-resolution image load in $file"
  fi
done
```

**Bad Example**:
```swift
// Loads entire 12MP image into memory!
func loadImage(from data: Data) -> UIImage? {
    return UIImage(data: data)
}

// Same problem with file path
func loadFromFile(_ path: String) -> UIImage? {
    return UIImage(contentsOfFile: path)
}

// Using in SwiftUI - still full resolution
struct PhotoView: View {
    let imageData: Data

    var body: some View {
        if let uiImage = UIImage(data: imageData) {
            Image(uiImage: uiImage)
                .resizable()
                .frame(width: 100, height: 100) // Still 48MB in memory!
        }
    }
}
```

**Good Example**:
```swift
// iOS 15+ thumbnail API
func loadThumbnail(from data: Data, maxSize: CGSize) async -> UIImage? {
    guard let image = UIImage(data: data) else { return nil }
    return await image.byPreparingThumbnail(ofSize: maxSize)
}

// Synchronous version
func loadThumbnailSync(from data: Data, maxSize: CGSize) -> UIImage? {
    guard let image = UIImage(data: data) else { return nil }
    return image.preparingThumbnail(of: maxSize)
}

// CGImageSource for more control (works on older iOS)
func downsampleImage(
    from data: Data,
    to pointSize: CGSize,
    scale: CGFloat = UIScreen.main.scale
) -> UIImage? {
    let maxDimensionInPixels = max(pointSize.width, pointSize.height) * scale

    let options: [CFString: Any] = [
        kCGImageSourceShouldCache: false
    ]

    guard let imageSource = CGImageSourceCreateWithData(data as CFData, options as CFDictionary) else {
        return nil
    }

    let downsampleOptions: [CFString: Any] = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceShouldCacheImmediately: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: maxDimensionInPixels
    ]

    guard let downsampledImage = CGImageSourceCreateThumbnailAtIndex(
        imageSource,
        0,
        downsampleOptions as CFDictionary
    ) else {
        return nil
    }

    return UIImage(cgImage: downsampledImage)
}

// SwiftUI usage with downsampling
struct PhotoView: View {
    let imageData: Data
    let displaySize: CGSize

    var body: some View {
        if let thumbnail = downsampleImage(from: imageData, to: displaySize) {
            Image(uiImage: thumbnail)
                .resizable()
                .frame(width: displaySize.width, height: displaySize.height)
        }
    }
}
```

**Rationale**: A 12MP photo (4000x3000) at 4 bytes per pixel uses 48MB of memory. Using `byPreparingThumbnail(ofSize:)` or `CGImageSource` downsamples during decode, loading only the pixels needed. This reduces memory from 48MB to under 1MB for a 100x100 thumbnail.

**Scanner Pattern**: `UIImage\s*\(\s*data:`
- Flag `UIImage(data:)` and `UIImage(contentsOfFile:)`
- Check for nearby `preparingThumbnail`, `byPreparingThumbnail`, or `CGImageSource`
- Suggest iOS 15+ thumbnail API or CGImageSource downsampling
- Alert: This is a memory-critical pattern that can cause OOM crashes

---

## Accessibility Traversal (Important)

### AP-031: Missing accessibilitySortPriority

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Elements announced in wrong order by VoiceOver. By default, SwiftUI traverses elements in layout order (top-to-bottom, left-to-right), which may not match the logical reading order or importance hierarchy.

**Detection Pattern** (Regex):
```regex
(HStack|ZStack)\s*\{[^}]*(Button|onTapGesture)[^}]*(Button|onTapGesture)
```

**Detection Script** (grep):
```bash
# Find layouts with multiple interactive elements
grep -r "HStack\|ZStack" --include="*.swift" Sources/ -A20 | grep -B10 "Button\|onTapGesture" | grep -B10 "Button\|onTapGesture"
# Check if accessibilitySortPriority is present
grep -r "accessibilitySortPriority" --include="*.swift" Sources/
# Manual review needed for complex layouts
```

**Compound Check**:
```bash
# Find files with multiple interactive elements but no sort priority
for file in $(grep -r "Button\|onTapGesture" --include="*.swift" -l Sources/); do
  count=$(grep -c "Button\|onTapGesture" "$file")
  if [ "$count" -gt 3 ] && ! grep -q "accessibilitySortPriority\|accessibilityElement(children:" "$file"; then
    echo "CHECK: $file has $count interactive elements without sort priority"
  fi
done
```

**Bad Example**:
```swift
struct EventCard: View {
    let event: Event
    let onJoin: () -> Void
    let onShare: () -> Void

    var body: some View {
        HStack {
            // Visual order: Image, Title, Share button, Join button
            // VoiceOver reads in source order, which may confuse users

            AsyncImage(url: event.imageURL) { image in
                image.resizable()
            } placeholder: {
                ProgressView()
            }
            .frame(width: 80, height: 80)

            VStack(alignment: .leading) {
                Text(event.title)
                    .font(.headline)
                Text(event.date)
                    .font(.caption)
            }

            Spacer()

            // Share appears before Join visually, but Join is primary action
            Button(action: onShare) {
                Image(systemName: "square.and.arrow.up")
            }
            .accessibilityLabel("Share event")

            Button(action: onJoin) {
                Text("Join")
            }
        }
    }
}
```

**Good Example**:
```swift
struct EventCard: View {
    let event: Event
    let onJoin: () -> Void
    let onShare: () -> Void

    var body: some View {
        HStack {
            AsyncImage(url: event.imageURL) { image in
                image.resizable()
            } placeholder: {
                ProgressView()
            }
            .frame(width: 80, height: 80)
            .accessibilityHidden(true) // Decorative

            VStack(alignment: .leading) {
                Text(event.title)
                    .font(.headline)
                Text(event.date)
                    .font(.caption)
            }
            .accessibilitySortPriority(3) // Read first (higher = earlier)

            Spacer()

            // Primary action gets higher priority (announced first among buttons)
            Button(action: onJoin) {
                Text("Join")
            }
            .accessibilitySortPriority(2)

            // Secondary action gets lower priority
            Button(action: onShare) {
                Image(systemName: "square.and.arrow.up")
            }
            .accessibilityLabel("Share event")
            .accessibilitySortPriority(1)
        }
        .accessibilityElement(children: .contain) // Maintain child accessibility
    }
}
```

**Alternative - Combine Elements for Simple Cards**:
```swift
struct EventCard: View {
    let event: Event
    let onJoin: () -> Void

    var body: some View {
        Button(action: onJoin) {
            HStack {
                AsyncImage(url: event.imageURL) { image in
                    image.resizable()
                } placeholder: {
                    ProgressView()
                }
                .frame(width: 80, height: 80)

                VStack(alignment: .leading) {
                    Text(event.title)
                    Text(event.date)
                }

                Spacer()

                Image(systemName: "chevron.right")
            }
        }
        .accessibilityElement(children: .combine) // Single focus target
        .accessibilityLabel("\(event.title), \(event.date)")
        .accessibilityHint("Double tap to join")
    }
}
```

**Rationale**: VoiceOver users navigate sequentially through focusable elements. If the visual layout doesn't match the logical focus order, users may struggle to find primary actions. Use `accessibilitySortPriority` (higher values = earlier in sequence) to control announcement order. For simple cards, `accessibilityElement(children: .combine)` creates a single focus target with combined labels.

**Scanner Pattern**: `HStack|ZStack` containing multiple `Button`
- Flag layouts with 2+ interactive elements
- Check for `accessibilitySortPriority` modifiers
- Suggest priority assignment or element combining

---

### AP-032: Missing Custom Accessibility Actions

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: Swipe gestures and complex interactions without accessible alternatives via `.accessibilityAction`. VoiceOver users cannot perform swipe-to-delete, drag gestures, or custom swipe actions.

**Detection Pattern** (Regex):
```regex
\.(swipeActions|onDelete|gesture\(DragGesture|simultaneousGesture|highPriorityGesture)
```

**Detection Script** (grep):
```bash
# Find gesture-based interactions
grep -r "swipeActions\|onDelete\|DragGesture\|gesture(" --include="*.swift" Sources/
# Check if accessibilityAction is defined
grep -r "accessibilityAction\|accessibilityCustomContent" --include="*.swift" Sources/
# Flag files with gestures but no custom actions
for file in $(grep -r "swipeActions\|DragGesture" --include="*.swift" -l Sources/); do
  if ! grep -q "accessibilityAction" "$file"; then
    echo "WARN: $file has gestures without accessibilityAction"
  fi
done
```

**Compound Check**:
```bash
# Find swipeActions without accessible alternative
grep -r "swipeActions\|onDelete" --include="*.swift" Sources/ | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  if ! grep -q "accessibilityAction" "$file"; then
    echo "ACCESSIBILITY: $file uses swipe actions without accessibilityAction"
  fi
done
```

**Bad Example**:
```swift
struct NotificationRow: View {
    let notification: Notification
    let onDismiss: () -> Void
    let onMarkRead: () -> Void

    var body: some View {
        HStack {
            Text(notification.message)
            Spacer()
        }
        .swipeActions(edge: .trailing, allowsFullSwipe: true) {
            Button(role: .destructive, action: onDismiss) {
                Label("Delete", systemImage: "trash")
            }

            Button(action: onMarkRead) {
                Label("Read", systemImage: "envelope.open")
            }
            .tint(.blue)
        }
        // VoiceOver users cannot access swipe actions!
    }
}

// Drag gesture without accessible alternative
struct ReorderableItem: View {
    let item: Item
    @GestureState private var dragOffset = CGSize.zero

    var body: some View {
        HStack {
            Image(systemName: "line.3.horizontal")
            Text(item.title)
        }
        .offset(dragOffset)
        .gesture(
            DragGesture()
                .updating($dragOffset) { value, state, _ in
                    state = value.translation
                }
        )
        // VoiceOver users cannot drag to reorder!
    }
}
```

**Good Example**:
```swift
struct NotificationRow: View {
    let notification: Notification
    let onDismiss: () -> Void
    let onMarkRead: () -> Void

    var body: some View {
        HStack {
            Text(notification.message)
            Spacer()
        }
        .swipeActions(edge: .trailing, allowsFullSwipe: true) {
            Button(role: .destructive, action: onDismiss) {
                Label("Delete", systemImage: "trash")
            }

            Button(action: onMarkRead) {
                Label("Read", systemImage: "envelope.open")
            }
            .tint(.blue)
        }
        // Provide accessible alternatives for swipe actions
        .accessibilityAction(named: "Delete") {
            onDismiss()
        }
        .accessibilityAction(named: "Mark as read") {
            onMarkRead()
        }
    }
}

// Drag gesture with accessible alternatives
struct ReorderableItem: View {
    let item: Item
    let onMoveUp: () -> Void
    let onMoveDown: () -> Void
    @GestureState private var dragOffset = CGSize.zero

    var body: some View {
        HStack {
            Image(systemName: "line.3.horizontal")
                .accessibilityHidden(true)
            Text(item.title)
        }
        .offset(dragOffset)
        .gesture(
            DragGesture()
                .updating($dragOffset) { value, state, _ in
                    state = value.translation
                }
        )
        .accessibilityElement(children: .combine)
        .accessibilityLabel(item.title)
        .accessibilityHint("Use actions to reorder")
        .accessibilityAction(named: "Move up") {
            onMoveUp()
        }
        .accessibilityAction(named: "Move down") {
            onMoveDown()
        }
    }
}

// Long press with accessible alternative
struct MessageBubble: View {
    let message: Message
    let onReact: () -> Void
    @State private var showReactions = false

    var body: some View {
        Text(message.content)
            .padding()
            .background(Color.blue.opacity(0.2))
            .cornerRadius(12)
            .onLongPressGesture {
                showReactions = true
                onReact()
            }
            .accessibilityAction(named: "Add reaction") {
                onReact()
            }
            .sheet(isPresented: $showReactions) {
                ReactionPicker()
            }
    }
}

// Context menu (automatically accessible, but custom actions can enhance)
struct EnhancedContextMenu: View {
    let item: Item
    let onEdit: () -> Void
    let onDelete: () -> Void
    let onShare: () -> Void

    var body: some View {
        Text(item.title)
            .contextMenu {
                Button(action: onEdit) {
                    Label("Edit", systemImage: "pencil")
                }
                Button(action: onShare) {
                    Label("Share", systemImage: "square.and.arrow.up")
                }
                Button(role: .destructive, action: onDelete) {
                    Label("Delete", systemImage: "trash")
                }
            }
            // Context menu actions are automatically accessible,
            // but adding explicit actions ensures discovery
            .accessibilityAction(named: "Edit") { onEdit() }
            .accessibilityAction(named: "Share") { onShare() }
            .accessibilityAction(named: "Delete") { onDelete() }
    }
}
```

**Rationale**: VoiceOver users interact through taps and the rotor (which exposes custom actions). Swipe gestures, long press, and drag-and-drop are inaccessible without alternatives. The `.accessibilityAction(named:)` modifier adds custom actions that appear in VoiceOver's Actions rotor, accessible by swiping up/down when focused on the element. Note that `contextMenu` actions are automatically accessible, but explicit `.accessibilityAction` modifiers can provide clearer names and ensure discoverability.

**Scanner Pattern**: `.swipeActions|.onDelete|DragGesture`
- Flag swipe actions, delete handlers, and drag gestures
- Check for corresponding `.accessibilityAction` modifiers
- Suggest adding named accessibility actions for each gesture
- Note: Context menus are automatically accessible but can be enhanced

---

## Summary Table

| ID | Anti-Pattern | Severity | Category |
|----|--------------|----------|----------|
| AP-001 | @ObservedObject for owned instances | 🔴 Critical | State |
| AP-002 | @State with reference types | 🔴 Critical | State |
| AP-003 | Missing @Published | 🔴 Critical | State |
| AP-004 | Modifying @State in init | 🟠 Important | State |
| AP-005 | AnyView overuse | 🟠 Important | Performance |
| AP-006 | VStack for long lists | 🟠 Important | Performance |
| AP-007 | Missing .id() | 🟠 Important | Performance |
| AP-008 | Heavy computation in body | 🟠 Important | Performance |
| AP-009 | Ignoring .task | 🟡 Warning | Performance |
| AP-010 | Missing accessibilityLabel | 🔴 Critical | Accessibility |
| AP-011 | Small touch targets | 🔴 Critical | Accessibility |
| AP-012 | Decorative images announced | 🟠 Important | Accessibility |
| AP-013 | Hardcoded colors | 🟠 Important | Styling |
| AP-014 | Hardcoded font sizes | 🟠 Important | Styling |
| AP-015 | Wrong modifier order | 🟡 Warning | Styling |
| AP-016 | NavigationLink performance | 🟠 Important | Navigation |
| AP-017 | GeometryReader breaking layout | 🟠 Important | Layout |
| AP-018 | @MainActor missing | 🟠 Important | Concurrency |
| AP-019 | Ignoring Task cancellation | 🟡 Warning | Concurrency |
| AP-020 | Force unwrapping optionals | 🟠 Important | Safety |
| AP-021 | Missing @Observable macro (iOS 17+) | 🟠 Important | State |
| AP-022 | Hardcoded colors in dark mode | 🔴 Critical | Styling |
| AP-023 | .onAppear with Task instead of .task | 🟡 Warning | Concurrency |
| AP-024 | @StateObject dependency injection | 🔴 Critical | State |
| AP-025 | Missing [weak self] in Task | 🔴 Critical | Concurrency |
| AP-026 | ScrollView vs List misuse | 🟠 Important | Performance |
| AP-027 | EnvironmentObject crashes | 🔴 Critical | State |
| AP-028 | AsyncImage without placeholder | 🟠 Important | Image Loading & Memory |
| AP-029 | Missing image caching | 🟠 Important | Image Loading & Memory |
| AP-030 | UIImage from Data without downsampling | 🔴 Critical | Image Loading & Memory |
| AP-031 | Missing accessibilitySortPriority | 🟠 Important | Accessibility Traversal |
| AP-032 | Missing Custom Accessibility Actions | 🟠 Important | Accessibility Traversal |

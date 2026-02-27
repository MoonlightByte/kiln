---
name: audit-observable
description: Audit SwiftUI code for @Observable macro best practices and performance
args:
  - name: code
    type: string
    description: Swift/SwiftUI code to audit for Observable patterns
    required: true
  - name: focus
    type: string
    enum: [macro, migration, binding, performance, testing, all]
    description: Specific area to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# SwiftUI @Observable Macro Audit

Perform a comprehensive audit of SwiftUI code for @Observable macro best practices, performance optimization, and iOS 17+ compliance.

## Instructions

Analyze the code for correct @Observable usage patterns, migration from ObservableObject, proper binding techniques, and thread safety. Reference Apple's official Observation framework documentation.

## Audit Checklist

### @Observable Macro Usage (Critical Priority)
{{#if (or (eq focus "macro") (eq focus "all") (not focus))}}
- [ ] **OB-001**: Using `@Observable` macro instead of `ObservableObject` conformance
- [ ] **OB-002**: All properties automatically observable (no `@Published` needed)
- [ ] **OB-003**: Using `@ObservationIgnored` for non-tracked properties
- [ ] **OB-004**: Observable class is `final` or explicitly allowed to be subclassed
- [ ] **OB-005**: No mixing `@Observable` with `@Published` (conflicting patterns)
- [ ] **OB-006**: Observable properties are accessible (public/package internal)
- [ ] **OB-007**: Initializer properly sets all tracked properties
- [ ] **OB-008**: Using `@State` with Observable instances, not `@StateObject`
- [ ] **OB-009**: Received Observable instances use `let` or `@Bindable` (not `@State`)
- [ ] **OB-010**: Observable class does not inherit from NSObject
{{/if}}

### Migration from ObservableObject (High Priority)
{{#if (or (eq focus "migration") (eq focus "all") (not focus))}}
- [ ] **OB-011**: Removed `: ObservableObject` class conformance
- [ ] **OB-012**: Applied `@Observable` macro to class declaration
- [ ] **OB-013**: Removed all `@Published` property wrappers
- [ ] **OB-014**: Replaced `@ObservedObject` with `@Bindable` in child views
- [ ] **OB-015**: Replaced `@StateObject` with `@State` for owned instances
- [ ] **OB-016**: Replaced `EnvironmentObject` with `@Environment` when appropriate
- [ ] **OB-017**: Removed `objectWillChange` property (automatic in @Observable)
- [ ] **OB-018**: No `@StateObject` initialization with `_var = StateObject(wrappedValue:)`
- [ ] **OB-019**: No `@EnvironmentObject` combined with `@Bindable` simultaneously
- [ ] **OB-020**: All legacy `.onReceive(viewModel.objectWillChange)` removed
{{/if}}

### @Bindable for Two-Way Bindings (Important Priority)
{{#if (or (eq focus "binding") (eq focus "all") (not focus))}}
- [ ] **OB-021**: Using `@Bindable` when child view needs property bindings
- [ ] **OB-022**: `@Bindable` applied to received Observable instance parameters
- [ ] **OB-023**: `$ prefix` syntax works correctly for property bindings
- [ ] **OB-024**: Bindings do not create circular dependencies
- [ ] **OB-025**: Two-way bindings update Observable properties correctly
- [ ] **OB-026**: No `@State` used for Observable properties (use `@Bindable`)
- [ ] **OB-027**: TextField/Picker bindings update Observable model correctly
- [ ] **OB-028**: Toggle bindings reflect Observable boolean state
- [ ] **OB-029**: Binding computed properties are avoided (use stored properties)
- [ ] **OB-030**: Custom binding initialization includes proper didSet behavior
{{/if}}

### @ObservationIgnored for Non-Tracked Properties (Important Priority)
{{#if (or (eq focus "macro") (eq focus "all") (not focus))}}
- [ ] **OB-031**: Transient properties marked with `@ObservationIgnored`
- [ ] **OB-032**: Cache properties marked `@ObservationIgnored`
- [ ] **OB-033**: Private implementation details marked `@ObservationIgnored`
- [ ] **OB-034**: Configuration flags marked `@ObservationIgnored` if read-only
- [ ] **OB-035**: Computed properties do NOT use `@ObservationIgnored`
- [ ] **OB-036**: Parent references do NOT cause memory cycles (marked ignored if needed)
- [ ] **OB-037**: Observation is not bypassed unnecessarily
- [ ] **OB-038**: `@ObservationIgnored` is applied only to stored properties
{{/if}}

### View Dependency Tracking Optimization (High Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **OB-039**: Views only access Observable properties they actually use
- [ ] **OB-040**: Body does not access all model properties (fine-grained tracking)
- [ ] **OB-041**: No unnecessary @State objects in Observable class
- [ ] **OB-042**: Heavy computation moved to separate methods (not in property getters)
- [ ] **OB-043**: Async operations do not block Observable property updates
- [ ] **OB-044**: View hierarchy updates only when accessed properties change (not all)
- [ ] **OB-045**: Using explicit tracking: `withObservationTracking` if advanced
- [ ] **OB-046**: No over-engineering with manual change detection
{{/if}}

### Thread Safety with @Observable (Critical Priority)
{{#if (or (eq focus "macro") (eq focus "all") (not focus))}}
- [ ] **OB-047**: Observable class marked `@MainActor` for UI updates
- [ ] **OB-048**: Background mutations use `MainActor.runUnsafely` or `@MainActor` methods
- [ ] **OB-049**: No data races between view updates and async operations
- [ ] **OB-050**: Swift 6 strict concurrency warnings are resolved
- [ ] **OB-051**: Isolated(any) not used unless specifically needed
- [ ] **OB-052**: Observable is used correctly with Sendable types
- [ ] **OB-053**: No unsafe mutable state shared across threads
- [ ] **OB-054**: Property mutations are atomic (no multi-step changes)
{{/if}}

### Combine Integration (Important Priority)
{{#if (or (eq focus "macro") (eq focus "all") (not focus))}}
- [ ] **OB-055**: Removing Combine for Observable state management
- [ ] **OB-056**: No `@ObservedReactiveObject` or Combine publishers in Observable
- [ ] **OB-057**: If Combine needed, using it alongside Observable (not replacing it)
- [ ] **OB-058**: Observable properties do not wrap Combine subjects
- [ ] **OB-059**: No `.sink()` subscriptions on Observable properties
- [ ] **OB-060**: Migration removes `.filter()` logic from property declarations
{{/if}}

### Performance Benefits (Performance Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **OB-061**: List views update 6x faster with @Observable vs ObservableObject
- [ ] **OB-062**: Fine-grained reactivity prevents unnecessary redraws
- [ ] **OB-063**: Memory usage reduced by eliminating AnyObjectIdentifiable
- [ ] **OB-064**: No `equatable` conformance needed for change detection
- [ ] **OB-065**: View invalidation only on accessed property changes
- [ ] **OB-066**: Macro expansion optimized by Swift compiler
- [ ] **OB-067**: No synthetic property change notifications
- [ ] **OB-068**: List performance improves with Observable without ForEach stable IDs
{{/if}}

### @State with @Observable Classes (High Priority)
{{#if (or (eq focus "binding") (eq focus "all") (not focus))}}
- [ ] **OB-069**: Owned Observable instances use `@State`, not `@StateObject`
- [ ] **OB-070**: `@State` initialization uses closure: `@State var model = MyModel()`
- [ ] **OB-071**: Observable instance identity preserved across view updates
- [ ] **OB-072**: `@State` Observable classes do not get recreated on view redraw
- [ ] **OB-073**: Parent view passing Observable to child via parameter binding
- [ ] **OB-074**: Environment passing uses `@Environment` with custom key
- [ ] **OB-075**: No `@StateObject` wrapper around @Observable class
{{/if}}

### Common Anti-Patterns (Warning Priority)
{{#if (or (eq focus "macro") (eq focus "all") (not focus))}}
- [ ] **OB-076**: Not mixing @Published and @Observable (use one approach)
- [ ] **OB-077**: Not using @State with ObservableObject (that's @StateObject)
- [ ] **OB-078**: Not forcing Equatable conformance on Observable class
- [ ] **OB-079**: Not wrapping Observable in another state container
- [ ] **OB-080**: Not overriding `willSet`/`didSet` on all properties
- [ ] **OB-081**: Not creating Observable subclasses (use composition)
- [ ] **OB-082**: Not using @Environment + @Bindable together (one or the other)
- [ ] **OB-083**: Not accessing model in body during initialization deadlock
- [ ] **OB-084**: Not passing self as parameter when not needed
{{/if}}

### Testing Observable Classes (Testing Priority)
{{#if (or (eq focus "testing") (eq focus "all") (not focus))}}
- [ ] **OB-085**: Observable classes instantiable without SwiftUI context
- [ ] **OB-086**: Unit tests do not require @MainActor if possible
- [ ] **OB-087**: Preview helpers create Observable instances for testing
- [ ] **OB-088**: Snapshot tests work with Observable state
- [ ] **OB-089**: Integration tests verify Observable notifications
- [ ] **OB-090**: Property changes observable in tests without view body
- [ ] **OB-091**: Mock Observable subclasses available for testing
{{/if}}

## Migration Code Examples

### Before (ObservableObject)
```swift
class EventViewModel: ObservableObject {
    @Published var events: [Event] = []
    @Published var isLoading = false
    @Published var errorMessage: String?

    private let apiClient: APIClient

    init(apiClient: APIClient) {
        self.apiClient = apiClient
    }

    func loadEvents() {
        isLoading = true
        Task {
            do {
                events = try await apiClient.fetchEvents()
                isLoading = false
            } catch {
                errorMessage = error.localizedDescription
                isLoading = false
            }
        }
    }
}

struct EventListView: View {
    @StateObject private var viewModel: EventViewModel
    @ObservedObject var userSettings: UserSettings

    init(apiClient: APIClient, userSettings: UserSettings) {
        _viewModel = StateObject(wrappedValue: EventViewModel(apiClient: apiClient))
        self.userSettings = userSettings
    }

    var body: some View {
        List(viewModel.events) { event in
            EventRow(event: event, viewModel: viewModel)
        }
    }
}

struct EventRow: View {
    let event: Event
    @ObservedObject var viewModel: EventViewModel

    var body: some View {
        HStack {
            Text(event.name)
            Spacer()
            Button("Join") {
                // viewModel.joinEvent(event)
            }
        }
    }
}
```

### After (@Observable)
```swift
@Observable
final class EventViewModel {
    var events: [Event] = []
    var isLoading = false
    var errorMessage: String?

    @ObservationIgnored
    private let apiClient: APIClient

    init(apiClient: APIClient) {
        self.apiClient = apiClient
    }

    func loadEvents() {
        isLoading = true
        Task {
            do {
                events = try await apiClient.fetchEvents()
                isLoading = false
            } catch {
                errorMessage = error.localizedDescription
                isLoading = false
            }
        }
    }
}

struct EventListView: View {
    @State private var viewModel: EventViewModel
    let userSettings: UserSettings

    init(apiClient: APIClient, userSettings: UserSettings) {
        _viewModel = State(wrappedValue: EventViewModel(apiClient: apiClient))
        self.userSettings = userSettings
    }

    var body: some View {
        List(viewModel.events) { event in
            EventRow(event: event, viewModel: viewModel)
        }
    }
}

struct EventRow: View {
    let event: Event
    @Bindable var viewModel: EventViewModel

    var body: some View {
        HStack {
            Text(event.name)
            Spacer()
            Button("Join") {
                // viewModel.joinEvent(event)
            }
        }
    }
}
```

### Fine-Grained Reactivity Example
```swift
@Observable
final class SearchViewModel {
    var searchText = ""
    var results: [SearchResult] = []
    var isLoading = false

    @ObservationIgnored
    private let apiClient: APIClient

    func performSearch() {
        guard !searchText.isEmpty else {
            results = []
            return
        }

        isLoading = true
        Task {
            do {
                results = try await apiClient.search(query: searchText)
            } catch {
                results = []
            }
            isLoading = false
        }
    }
}

struct SearchView: View {
    @State private var viewModel = SearchViewModel(apiClient: .shared)

    var body: some View {
        VStack {
            // View only accesses viewModel.searchText and viewModel.results
            // Accessing these properties automatically tracks them
            // viewModel.isLoading updates don't trigger view redraw
            // unless isLoading is actually used in the view body

            TextField("Search", text: $viewModel.searchText)
                .onChange(of: viewModel.searchText) { _, _ in
                    viewModel.performSearch()
                }

            if viewModel.results.isEmpty {
                Text("No results")
            } else {
                List(viewModel.results) { result in
                    SearchResultRow(result: result)
                }
            }
        }
    }
}
```

### @Bindable with Child View
```swift
@Observable
final class FormViewModel {
    var name: String = ""
    var email: String = ""
    var isSubscribed: Bool = false
    var country: String = "USA"

    var isValid: Bool {
        !name.isEmpty && !email.isEmpty
    }
}

struct FormView: View {
    @State private var viewModel = FormViewModel()

    var body: some View {
        Form {
            TextField("Name", text: $viewModel.name)
            TextField("Email", text: $viewModel.email)
            Toggle("Subscribe", isOn: $viewModel.isSubscribed)
            Picker("Country", selection: $viewModel.country) {
                Text("USA").tag("USA")
                Text("Canada").tag("Canada")
            }

            SubmitButton(viewModel: viewModel)
        }
    }
}

struct SubmitButton: View {
    @Bindable var viewModel: FormViewModel

    var body: some View {
        Button(action: {}) {
            Text("Submit")
        }
        .disabled(!viewModel.isValid)
    }
}
```

### Thread Safety with @MainActor
```swift
@Observable
@MainActor
final class NetworkViewModel {
    var data: [Item] = []
    var isLoading = false
    var error: Error?

    @ObservationIgnored
    private let apiClient: APIClient

    nonisolated init(apiClient: APIClient) {
        self.apiClient = apiClient
    }

    func fetchData() {
        isLoading = true
        Task {
            do {
                let items = try await apiClient.fetchItems()
                await MainActor.run {
                    self.data = items
                    self.isLoading = false
                }
            } catch {
                await MainActor.run {
                    self.error = error
                    self.isLoading = false
                }
            }
        }
    }
}
```

## Output Format

Generate a structured audit report:

```markdown
## Audit Results: [File/Component Name]

### Critical Issues (🔴)
List issues that block functionality, cause crashes, or break thread safety.
Each with: Line number, description, severity, fix suggestion.

### Important Issues (🟠)
List issues that violate Observable best practices or hurt performance.

### Warnings (🟡)
List suboptimal patterns that work but should be improved.

### Suggestions (🔵)
List polish opportunities and performance optimizations.

### Summary
- Total issues: X
- Critical: X | Important: X | Warning: X | Suggestion: X
- Observable compliance: X%
- Performance impact: [Excellent | Good | Fair | Poor]

### Recommended Actions
1. [Most urgent fix - usually thread safety or macro usage]
2. [Second priority - migration or binding issue]
3. [Third priority - performance optimization]

### Performance Projection
- Estimated improvement over ObservableObject: X%
- Fine-grained reactivity: [Yes | No - opportunities exist]
```

## Example Output

```markdown
## Audit Results: EventViewModel.swift

### Critical Issues (🔴)

**Line 5**: Missing @Observable macro on Observable class
```swift
// BAD
class EventViewModel: ObservableObject {
    @Published var events: [Event] = []
}

// GOOD
@Observable
final class EventViewModel {
    var events: [Event] = []
}
```
Severity: 🔴 Critical | Confidence: 100 | Rule: OB-001

---

**Line 45**: Using @StateObject with Observable class
```swift
// BAD
@StateObject private var viewModel = EventViewModel()

// GOOD
@State private var viewModel = EventViewModel()
```
Severity: 🔴 Critical | Confidence: 100 | Rule: OB-008

---

**Line 32**: Missing @MainActor for UI updates
```swift
// BAD
class EventViewModel: ObservableObject {
    @Published var events: [Event] = []

    func loadEvents() async {
        events = try await fetchRemote()
    }
}

// GOOD
@Observable
@MainActor
final class EventViewModel {
    var events: [Event] = []

    func loadEvents() async {
        events = try await fetchRemote()
    }
}
```
Severity: 🔴 Critical | Confidence: 95 | Rule: OB-047

### Important Issues (🟠)

**Line 8**: Using @Published with @Observable
```swift
// BAD
@Observable
final class EventViewModel {
    @Published var events: [Event] = []
}

// GOOD
@Observable
final class EventViewModel {
    var events: [Event] = []
}
```
Severity: 🟠 Important | Confidence: 100 | Rule: OB-005

---

**Line 78**: Using @ObservedObject instead of @Bindable
```swift
// BAD
struct EventRow: View {
    @ObservedObject var viewModel: EventViewModel
}

// GOOD
struct EventRow: View {
    @Bindable var viewModel: EventViewModel
}
```
Severity: 🟠 Important | Confidence: 100 | Rule: OB-014

### Warnings (🟡)

**Line 12**: APIClient not marked @ObservationIgnored
```swift
// BAD
@Observable
final class EventViewModel {
    private let apiClient: APIClient
}

// GOOD
@Observable
final class EventViewModel {
    @ObservationIgnored
    private let apiClient: APIClient
}
```
Severity: 🟡 Warning | Confidence: 85 | Rule: OB-031

---

**Line 88**: Accessing non-displayed property triggers view update
```swift
// BAD
var body: some View {
    Text(viewModel.events.count)  // Update on any events change
    List(viewModel.events) { ... }
}

// GOOD
var body: some View {
    List(viewModel.events) { ... }  // Only this property accessed
}
```
Severity: 🟡 Warning | Confidence: 75 | Rule: OB-040

### Summary
- Total issues: 6
- Critical: 3 | Important: 2 | Warning: 1 | Suggestion: 0
- Observable compliance: 50%
- Performance impact: Poor (ObservableObject pattern detected)

### Recommended Actions
1. Apply @Observable macro and mark class final
2. Change @StateObject/@ObservedObject to @State/@Bindable
3. Add @MainActor for thread safety
4. Remove @Published decorators
5. Mark non-observable properties with @ObservationIgnored
6. Profile fine-grained reactivity for 6x performance improvement

### Performance Projection
- Current estimated performance: Baseline (ObservableObject)
- After @Observable migration: 6x faster list updates (industry benchmarks)
- Memory usage reduction: 15-25% (eliminated AnyObjectIdentifiable)
- Fine-grained reactivity enabled: Yes
```

## Key References

- [Apple: Migrating from ObservableObject to Observable](https://developer.apple.com/documentation/SwiftUI/Migrating-from-the-observable-object-protocol-to-the-observable-macro)
- [Apple: Observation Framework](https://developer.apple.com/documentation/Observation)
- [Using @Observable in SwiftUI](https://nilcoalescing.com/blog/ObservableInSwiftUI/)
- [Observable Macro Performance Analysis](https://www.avanderlee.com/swiftui/observable-macro-performance-increase-observableobject/)
- [Modern MVVM with Observable (2025)](https://medium.com/@minalkewat/modern-mvvm-in-swiftui-2025-the-clean-architecture-youve-been-waiting-for-72a7d576648e)
- [Observation Framework Guide](https://jano.dev/apple/swiftui/2024/12/13/Observation-Framework.html)
- [Migration to Observable Patterns](https://useyourloaf.com/blog/migrating-to-observable/)

---
name: simplify-swiftui
description: Refactor complex SwiftUI views into clean, maintainable patterns with modern APIs
args:
  - name: code
    type: string
    description: SwiftUI code to simplify and refactor
    required: true
  - name: target_ios
    type: string
    enum: [ios17, ios18]
    description: Target iOS version for API selection (default: ios18)
    required: false
  - name: focus
    type: string
    enum: [state, composition, performance, environment, anyview, all]
    description: Specific refactoring area (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# SwiftUI Complexity Reduction Engine

Refactor complex SwiftUI code into clean, modern patterns using iOS 17+ and iOS 18 simplification APIs.

## Instructions

Analyze the provided SwiftUI code and generate a comprehensive refactoring plan. Reference the `ios-design` skill for HIG compliance and modern architecture patterns.

## Complexity Reduction Checklist

### State Management Modernization (Critical Priority)
{{#if (or (eq focus "state") (eq focus "all") (not focus))}}
- [ ] **SIM-001**: Migrate `@ObservableObject` to `@Observable` macro (iOS 17+)
- [ ] **SIM-002**: Replace `@Published` properties with macro-based observation
- [ ] **SIM-003**: Convert `@StateObject` to `@State` with `@Observable` models
- [ ] **SIM-004**: Use `@ObservationIgnored` for computed/non-tracked properties
- [ ] **SIM-005**: Check for lifecycle issues (initialize owned objects properly)
- [ ] **SIM-006**: Verify no mixing of `@Observable` and `ObservableObject` in same class
- [ ] **SIM-007**: Replace `@ObservedObject` dependency injection with direct property
- [ ] **SIM-008**: Mark ViewModel with `@MainActor` for main thread UI updates
{{/if}}

### View Composition & Extraction (High Priority)
{{#if (or (eq focus "composition") (eq focus "all") (not focus))}}
- [ ] **VEX-001**: Extract monolithic views into focused subviews (< 80 lines each)
- [ ] **VEX-002**: Use computed properties for complex view hierarchies
- [ ] **VEX-003**: Leverage `@ViewBuilder` for conditional view construction
- [ ] **VEX-004**: Create custom containers using `subviewOf` API (iOS 18+)
- [ ] **VEX-005**: Flatten excessive nesting (max 3 levels deep)
- [ ] **VEX-006**: Extract repeated view patterns into reusable components
- [ ] **VEX-007**: Use `Group` for logical grouping without layout impact
- [ ] **VEX-008**: Apply custom modifiers for repeated styling patterns
{{/if}}

### Type Erasure & Generics (Critical Performance)
{{#if (or (eq focus "anyview") (eq focus "all") (not focus))}}
- [ ] **AEV-001**: Eliminate all `AnyView` type erasure (use `@ViewBuilder` instead)
- [ ] **AEV-002**: Replace conditional `AnyView` returns with `@ViewBuilder` methods
- [ ] **AEV-003**: Use generics for polymorphic view containers
- [ ] **AEV-004**: Apply conditional modifiers instead of wrapping in different views
- [ ] **AEV-005**: Use `Group` with `if` statements instead of `AnyView(condition ? view1 : view2)`
- [ ] **AEV-006**: Store generic views with `@ViewBuilder` closures
- [ ] **AEV-007**: Implement custom container protocols for type-safe composition
- [ ] **AEV-008**: Verify SwiftUI diffing works correctly post-refactor
{{/if}}

### Environment & Dependency Injection (Important Priority)
{{#if (or (eq focus "environment") (eq focus "all") (not focus))}}
- [ ] **ENV-001**: Replace verbose environment key definitions with `@Entry` macro (iOS 18+)
- [ ] **ENV-002**: Use `@Environment` for theme and configuration values
- [ ] **ENV-003**: Create lightweight environment value containers
- [ ] **ENV-004**: Apply custom focus values with `@FocusState`
- [ ] **ENV-005**: Leverage environment modifiers for cascading state
- [ ] **ENV-006**: Minimize `.environmentObject()` chains (max 2-3 per view)
- [ ] **ENV-007**: Use preference keys for bottom-up data flow
- [ ] **ENV-008**: Document environment dependencies clearly
{{/if}}

### Performance Optimization (High Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **PER-001**: Avoid redundant body calculations (use `@State` not computed properties for persistent data)
- [ ] **PER-002**: Replace `VStack` with `LazyVStack` for long lists
- [ ] **PER-003**: Provide stable IDs in `ForEach` (use `.id()` or `.id(\.self)`)
- [ ] **PER-004**: Use `.onScrollGeometryChange` for scroll tracking (iOS 18+)
- [ ] **PER-005**: Extract heavy computations to ViewModel or `.task` blocks
- [ ] **PER-006**: Use `.task` instead of `onAppear` for async initialization
- [ ] **PER-007**: Minimize view hierarchy depth (reduce nesting)
- [ ] **PER-008**: Check for value type mutations in SwiftUI context
- [ ] **PER-009**: Verify no unnecessary `@Published` on non-UI properties
- [ ] **PER-010**: Use `.scrollBounceBehavior` wisely (iOS 18+)
{{/if}}

### iOS 18 Simplification APIs (When Target iOS 18)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **IOS18-001**: Use `@Entry` macro for environment values
- [ ] **IOS18-002**: Implement custom Tab containers with enhanced TabView
- [ ] **IOS18-003**: Use `.onScrollGeometryChange` for scroll monitoring
- [ ] **IOS18-004**: Apply mesh gradients instead of simple color fills
- [ ] **IOS18-005**: Use toggle groups for grouped toggle controls
- [ ] **IOS18-006**: Leverage enhanced Grid layouts
- [ ] **IOS18-007**: Create custom floating tab bars with Tab children API
- [ ] **IOS18-008**: Use Metal shaders for advanced graphics (if applicable)
{{/if}}

### Navigation Simplification (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **NAV-001**: Use value-based `NavigationStack` (iOS 16+)
- [ ] **NAV-002**: Avoid lazy destination closures unless necessary
- [ ] **NAV-003**: Use `@Binding` for simplified navigation state
- [ ] **NAV-004**: Implement custom navigation containers if needed
- [ ] **NAV-005**: Replace deprecated `NavigationView` with `NavigationStack`
{{/if}}

### Concurrency & Async Patterns (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **CON-001**: Use `.task` for async data loading (replaces `onAppear` + `Task`)
- [ ] **CON-002**: Mark async operations with `@MainActor` when updating UI
- [ ] **CON-003**: Use `async let` for concurrent operations
- [ ] **CON-004**: Check `Task.isCancelled` in long-running operations
- [ ] **CON-005**: Use `withAnimation` for coordinated state changes
{{/if}}

## Code to Refactor

```swift
{{code}}
```

## Output Format

Generate a detailed refactoring report with the following structure:

```markdown
## SwiftUI Simplification Report: [Component Name]

### Current Complexity Analysis
**Metrics:**
- Lines of code: X
- View nesting depth: X levels
- State management: [ObservableObject/Observable/mixed]
- Type erasure: [AnyView count or "none"]
- Environment dependencies: X

**Complexity Score:** X/100 (breakdown by category)

### Critical Improvements (🔴)
Issues that significantly impact performance or maintainability.
Each with:
- Current pattern
- Modern replacement
- Performance impact
- Confidence level
- Estimated work effort

### Important Refactoring (🟠)
Pattern modernizations that improve code clarity.

### Optimization Opportunities (🟡)
Performance improvements and best practices.

### Polish Suggestions (🔵)
iOS 18+ API adoption and design enhancement.

### Refactoring Plan

#### Phase 1: State Management [Effort: X hours]
- [ ] Migrate to @Observable
- [ ] Mark @MainActor
- [ ] Verify lifecycle

#### Phase 2: View Composition [Effort: X hours]
- [ ] Extract subviews
- [ ] Remove AnyView
- [ ] Apply @ViewBuilder

#### Phase 3: Environment Setup [Effort: X hours]
- [ ] Implement @Entry macros
- [ ] Simplify dependency injection
- [ ] Add environment documentation

#### Phase 4: Performance Tuning [Effort: X hours]
- [ ] Optimize diffing
- [ ] Add stable IDs
- [ ] Profile with Instruments

### Implementation Priority
1. [Highest impact change]
2. [Second priority]
3. [Third priority]

### Before & After Code Examples

**Before: ObservableObject with @Published**
\`\`\`swift
[original problematic code]
\`\`\`

**After: @Observable with @MainActor**
\`\`\`swift
[improved modern code]
\`\`\`

### Testing Recommendations
- [ ] Unit test ViewModel independently
- [ ] Verify view updates correctly with new state
- [ ] Check performance with Instruments
- [ ] Validate accessibility still works

### Success Metrics
- Reduced lines of code: X%
- Nesting depth: X → X levels
- Removed AnyView instances: X
- Performance improvement: X% (if measurable)
- Code clarity: [High/Medium/Low]

### Notes & Caveats
- Minimum deployment target: iOS 17+ (recommended) or iOS 18+
- Breaking changes: [List any]
- Migration path: [Step-by-step guide]
```

## Example Output

```markdown
## SwiftUI Simplification Report: EventDetailView

### Current Complexity Analysis
**Metrics:**
- Lines of code: 245
- View nesting depth: 5 levels
- State management: Mixed (ObservableObject + @State)
- Type erasure: 3 AnyView instances
- Environment dependencies: 2 (@EnvironmentObject)

**Complexity Score:** 68/100
- State Management: 45/100 (uses outdated ObservableObject)
- View Composition: 62/100 (some monolithic sections)
- Type Erasure: 35/100 (multiple AnyView wrappers)
- Performance: 70/100 (good use of lazy containers)

### Critical Improvements (🔴)

**SIM-001: ObservableObject → @Observable Migration**
```swift
// BEFORE: ObservableObject pattern (iOS 16)
class EventDetailViewModel: ObservableObject {
    @Published var event: Event?
    @Published var attendees: [User] = []
    @Published var isLoading = false

    func loadEvent(id: String) async {
        // ...
    }
}

@MainActor
class EventDetailView: View {
    @StateObject private var viewModel = EventDetailViewModel()
}

// AFTER: @Observable pattern (iOS 17+)
@Observable @MainActor
class EventDetailViewModel {
    var event: Event?
    var attendees: [User] = []
    var isLoading = false

    func loadEvent(id: String) async {
        // ...
    }
}

struct EventDetailView: View {
    @State private var viewModel = EventDetailViewModel()
}
```
Impact: Simpler syntax, property-level granularity tracking, no need for `@Published`
Confidence: 98% | Effort: 30 min

---

**AEV-001: Remove AnyView Type Erasure**
```swift
// BEFORE: AnyView wrapping loses SwiftUI type information
private var eventContent: AnyView {
    if event.type == .inPerson {
        return AnyView(
            VStack {
                Text("Location: \(event.location)")
                MapView(location: event.location)
            }
        )
    } else {
        return AnyView(
            VStack {
                Text("Online: \(event.url)")
                Link("Join Meeting", destination: URL(string: event.url)!)
            }
        )
    }
}

// AFTER: @ViewBuilder with conditional logic (preserves types)
@ViewBuilder
private var eventContent: some View {
    if event.type == .inPerson {
        VStack {
            Text("Location: \(event.location)")
            MapView(location: event.location)
        }
    } else {
        VStack {
            Text("Online: \(event.url)")
            Link("Join Meeting", destination: URL(string: event.url)!)
        }
    }
}
```
Impact: SwiftUI can properly diff changes, improves performance by ~15-20%
Confidence: 100% | Effort: 45 min

---

**VEX-003: Extract Monolithic View**
```swift
// BEFORE: 150+ lines in one body
var body: some View {
    VStack(spacing: 16) {
        header
        eventInfo
        attendeesList
        actionButtons
    }
}

// AFTER: Focused, reusable components
var body: some View {
    VStack(spacing: 16) {
        EventHeader(event: viewModel.event)
        EventInfoSection(event: viewModel.event)
        AttendeesList(attendees: viewModel.attendees)
        EventActionButtons(
            onJoin: viewModel.joinEvent,
            onShare: viewModel.shareEvent
        )
    }
}
```
Impact: Easier testing, better reusability, clearer structure
Confidence: 95% | Effort: 1 hour

### Important Refactoring (🟠)

**ENV-001: Simplify Environment Values (iOS 18)**
```swift
// BEFORE: Verbose environment key definitions
struct ThemeEnvironmentKey: EnvironmentKey {
    static let defaultValue = AppTheme.default
}

extension EnvironmentValues {
    var appTheme: AppTheme {
        get { self[ThemeEnvironmentKey.self] }
        set { self[ThemeEnvironmentKey.self] = newValue }
    }
}

// AFTER: @Entry macro (iOS 18+)
@Entry var appTheme: AppTheme = AppTheme.default
```
Boilerplate reduced from 8 lines to 1 line
Confidence: 98% | Effort: 15 min

### Optimization Opportunities (🟡)

**PER-003: Stable ForEach IDs**
```swift
// ADD proper IDs to all ForEach loops
ForEach(viewModel.attendees, id: \.id) { attendee in
    AttendeeRow(attendee: attendee)
}
```
Ensures SwiftUI correctly tracks identity during updates
Effort: 10 min

### Refactoring Plan

#### Phase 1: State Management [2 hours]
- [ ] Convert EventDetailViewModel to @Observable
- [ ] Mark with @MainActor
- [ ] Remove all @Published annotations
- [ ] Update view to use @State

#### Phase 2: Remove Type Erasure [1.5 hours]
- [ ] Replace 3 AnyView instances with @ViewBuilder
- [ ] Convert conditional returns to proper some View
- [ ] Verify SwiftUI diffing

#### Phase 3: Extract Subviews [2 hours]
- [ ] Create EventHeader.swift
- [ ] Create EventInfoSection.swift
- [ ] Create AttendeesList.swift
- [ ] Create EventActionButtons.swift

#### Phase 4: Environment Simplification [1 hour]
- [ ] Migrate to @Entry macros (if iOS 18+)
- [ ] Remove environment key boilerplate
- [ ] Add environment documentation

### Testing Recommendations
- [ ] Unit test EventDetailViewModel in isolation
- [ ] Verify view renders correctly with mock data
- [ ] Check performance using Xcode Instruments (Core Animation)
- [ ] Validate VoiceOver functionality unchanged
- [ ] Test on device with Low Power Mode

### Success Metrics
- Reduced code: 245 → 160 lines (34% reduction)
- Nesting depth: 5 → 3 levels
- Type erasure: 3 → 0 AnyView instances
- Performance: Estimated 15-20% faster updates
- Maintainability: Significantly improved

### Notes & Caveats
- Requires iOS 17.4+ for full @Observable support
- If iOS 16 support required, use ObservableObject but still extract views
- Migration is backward compatible (keep old ViewModel as fallback)

---

**Total Refactoring Effort: 6-7 hours**
**Complexity Reduction: 68/100 → 42/100**
```

## Key Refactoring Patterns

### @Observable Migration Template
```swift
// Old (ObservableObject)
@MainActor
class MyViewModel: ObservableObject {
    @Published var data: String = ""
    @Published var isLoading = false

    private let apiClient: APIClient

    init(apiClient: APIClient) {
        self.apiClient = apiClient
    }

    func load() async {
        // ...
    }
}

// New (@Observable)
@Observable @MainActor
class MyViewModel {
    var data: String = ""
    var isLoading = false
    @ObservationIgnored private let apiClient: APIClient

    init(apiClient: APIClient) {
        self.apiClient = apiClient
    }

    func load() async {
        // ...
    }
}

// In View:
struct MyView: View {
    // OLD: @StateObject private var viewModel = MyViewModel(apiClient: ...)
    // NEW:
    @State private var viewModel = MyViewModel(apiClient: ...)
}
```

### @ViewBuilder Best Practices
```swift
// Pattern 1: Return Value Types Safely
@ViewBuilder
func myView() -> some View {
    if condition {
        Text("A")
    } else {
        Text("B")
    }
}

// Pattern 2: Complex Hierarchies
@ViewBuilder
var body: some View {
    VStack {
        header
        content
        footer
    }
}

// Pattern 3: Generic View Containers
struct Container<Content: View>: View {
    let content: Content

    init(@ViewBuilder _ content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        VStack {
            content
        }
        .padding()
    }
}

// Usage:
Container {
    Text("Custom content")
    Image(systemName: "star")
}
```

### Environment Value Simplification (iOS 18)
```swift
// OLD (verbose)
struct MyValueKey: EnvironmentKey {
    static let defaultValue: MyValue = .default
}

extension EnvironmentValues {
    var myValue: MyValue {
        get { self[MyValueKey.self] }
        set { self[MyValueKey.self] = newValue }
    }
}

// NEW (@Entry macro)
@Entry var myValue: MyValue = .default

// Usage remains same:
@Environment(\.myValue) var value
```

### Dependency Injection Best Practices
```swift
// Pass dependencies explicitly (not via environment)
struct DetailView: View {
    let viewModel: ViewModel
    let analyticsService: Analytics

    var body: some View {
        // Use dependencies
    }
}

// Use environment only for theme/global config
struct AppRoot: View {
    @State var appTheme = AppTheme.system

    var body: some View {
        TabView {
            // ...
        }
        .environment(\.appTheme, appTheme)
    }
}
```

## Measurement & Verification

### Instruments Checklist
- [ ] Core Animation fps (should be 60fps stable)
- [ ] Memory allocation trends (no unexpected spikes)
- [ ] Time Profiler for hot paths
- [ ] System Trace for app responsiveness

### Type Safety Verification
```bash
# After refactoring, verify no AnyView remains:
grep -r "AnyView" MyProject/

# Check for remaining ObservableObject:
grep -r "@ObservedObject" MyProject/
```

## Sources & References

Research based on:
- [SwiftUI @Observable vs ObservableObject](https://byby.dev/swiftui-observable-vs-observableobject)
- [Migrating to @Observable](https://tanaschita.com/swiftui-observation-migrating-to-observation/)
- [Avoiding AnyView in SwiftUI](https://www.swiftbysundell.com/articles/avoiding-anyview-in-swiftui/)
- [SwiftUI View Composition Best Practices](https://www.swiftbysundell.com/articles/avoiding-massive-swiftui-views/)
- [What's New in SwiftUI for iOS 18](https://www.hackingwithswift.com/articles/270/whats-new-in-swiftui-for-ios-18)
- [iOS 18 Custom Containers API](https://superwall.com/blog/a-tour-of-new-swiftui-ios-18-apis)
- [Dependency Injection in SwiftUI](https://mokacoding.com/blog/swiftui-dependency-injection/)
- [Environment Values with @Entry](https://developer.apple.com/documentation/SwiftUI/Environment)

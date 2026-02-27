---
name: audit-navigation-ios18
description: Comprehensive audit of iOS 18 NavigationStack, type-safe routing, deep linking, and iPad navigation patterns
args:
  - name: code
    type: string
    description: SwiftUI navigation code to audit
    required: true
  - name: focus
    type: string
    enum: [routing, navigation-stack, split-view, deep-linking, modals, state, all]
    description: Specific navigation area to focus on (default: all)
    required: false
  - name: device_context
    type: string
    enum: [iphone, ipad, universal, all]
    description: Device context for audit (iPhone, iPad, or both) (default: universal)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# iOS 18 Navigation Audit (NavigationStack, Type-Safe Routing, Deep Linking, iPad)

Perform a comprehensive audit of SwiftUI navigation code against modern iOS 18 best practices, including type-safe routing patterns, NavigationStack usage, NavigationSplitView for iPad, deep linking, and state management.

## Instructions

Analyze the provided navigation code for compliance with Apple's latest Human Interface Guidelines and 2025 best practices. Reference the `ios-design` skill for detailed navigation guidelines.

Focus areas:
- **Type-safe routing** with enums instead of string-based routes
- **NavigationStack** with centralized path management
- **NavigationSplitView** for iPad multi-column layouts
- **Deep linking** with custom schemes and universal links
- **Modal presentation** (sheet, fullScreenCover) lifecycle
- **Navigation state persistence** across app lifecycle
- **Back button** customization and behavior
- **Tab-based navigation** integration with NavigationStack

## Navigation Audit Checklist

### Type-Safe Routing (Critical Priority)
{{#if (or (eq focus "routing") (eq focus "all") (not focus))}}
- [ ] **NV-001**: Navigation destinations defined as Hashable enum, not strings
- [ ] **NV-002**: Enum cases represent concrete screen types (not generic "detail" cases)
- [ ] **NV-003**: Route enum implements Codable for deep linking serialization
- [ ] **NV-004**: Each route case has associated values matching destination parameters
- [ ] **NV-005**: Route enum conforms to both Hashable and Equatable explicitly
- [ ] **NV-006**: Compiler forces handling of all route cases in navigationDestination
- [ ] **NV-007**: No string-based routes in NavigationPath or navigationLink
{{/if}}

### NavigationStack Fundamentals (Critical Priority)
{{#if (or (eq focus "navigation-stack") (eq focus "all") (not focus))}}
- [ ] **NV-010**: @State var navigationPath = NavigationPath() (not @Published in ViewModel)
- [ ] **NV-011**: NavigationStack anchored at single root view (not nested stacks)
- [ ] **NV-012**: NavigationStack(path:) binding to @State path property
- [ ] **NV-013**: path.append(route) used for programmatic navigation (not deprecated push)
- [ ] **NV-014**: path.removeLast() implemented for back button behavior
- [ ] **NV-015**: path.removeAll() clears navigation when returning to root
- [ ] **NV-016**: NavigationPath initialized with empty array, not optional
{{/if}}

### navigationDestination Modifier (Critical Priority)
{{#if (or (eq focus "routing") (eq focus "navigation-stack") (eq focus "all") (not focus))}}
- [ ] **NV-020**: Using .navigationDestination(for:destination:) for type matching
- [ ] **NV-021**: Not using deprecated NavigationView or NavigationLink (iOS 16+)
- [ ] **NV-022**: Destination closure is lazy (not evaluated until needed)
- [ ] **NV-023**: Each route type has exactly one .navigationDestination modifier
- [ ] **NV-024**: Multiple routes handled with separate modifiers, not @ViewBuilder chain
- [ ] **NV-025**: Destination views receive route value as parameter
- [ ] **NV-026**: Custom back button behavior defined with .navigationBackButtonHidden + custom button
{{/if}}

### Deep Linking & Path Restoration (Important Priority)
{{#if (or (eq focus "deep-linking") (eq focus "all") (not focus))}}
- [ ] **NV-030**: Route enum conforms to Codable for URL serialization
- [ ] **NV-031**: Custom URL scheme handler decodes paths from URLs
- [ ] **NV-032**: NavigationPath initialized from deep link URL at app launch
- [ ] **NV-033**: Universal link handling implemented (not just custom schemes)
- [ ] **NV-034**: Deep links tested with NSUserActivity and URLQueryItems
- [ ] **NV-035**: State restored on cold app launch with path: init(coder:)
- [ ] **NV-036**: Error handling for malformed deep links (invalid routes)
- [ ] **NV-037**: Deep link path validated (user has permission to access screens)
{{/if}}

### NavigationSplitView for iPad (Important Priority)
{{#if (or (eq focus "split-view") (eq focus "all") (not focus))}}
- [ ] **NV-040**: NavigationSplitView wraps entire iPad layout (not single screen)
- [ ] **NV-041**: Two-column layout: sidebar + detail (or three: sidebar + detail + secondary)
- [ ] **NV-042**: Selection state in @State var selectedItem (Hashable type matching routes)
- [ ] **NV-043**: NavigationSplitViewVisibility managed with @State var columnVisibility
- [ ] **NV-044**: Sidebar uses List with NavigationLink for item selection
- [ ] **NV-045**: Detail view updates when selectedItem changes (bindings work correctly)
- [ ] **NV-046**: Landscape orientation shows all columns; portrait collapses to NavigationStack
- [ ] **NV-047**: Sidebar button visible on iPadOS to toggle visibility
- [ ] **NV-048**: Swipe from left edge reveals sidebar on iPadOS
- [ ] **NV-049**: No NavigationStack nested inside NavigationSplitView (separate concerns)
{{/if}}

### Modal Presentation (Sheet, Fullscreen) (Important Priority)
{{#if (or (eq focus "modals") (eq focus "all") (not focus))}}
- [ ] **NV-050**: @State var sheetItem: RouteType? (optional enum case)
- [ ] **NV-051**: .sheet(item:) modifier used instead of isPresented: boolean
- [ ] **NV-052**: Sheet dismissed by setting sheetItem = nil (not boolean toggle)
- [ ] **NV-053**: Fullscreen modal uses .fullScreenCover(item:destination:)
- [ ] **NV-054**: Modal presentation doesn't use NavigationStack internally (flat hierarchy)
- [ ] **NV-055**: Modal environment has separate NavigationPath if nested navigation needed
- [ ] **NV-056**: Modal dismissed without causing navigation stack changes
- [ ] **NV-057**: Sheet height respected with .presentationDetents on iOS 16+
- [ ] **NV-058**: Background interaction disabled during modal with .interactiveDismissDisabled()
{{/if}}

### Navigation State Management (Critical Priority)
{{#if (or (eq focus "state") (eq focus "navigation-stack") (eq focus "all") (not focus))}}
- [ ] **NV-060**: Navigation state in @State, not @Published ViewModel property
- [ ] **NV-061**: NavigationPath held at view layer, not passed through deep hierarchies
- [ ] **NV-062**: Path updates don't trigger unnecessary view rebuilds
- [ ] **NV-063**: State restored on app backgrounding/foregrounding
- [ ] **NV-064**: Navigation persisted with SceneStorage on iOS 16+ (multiwindow apps)
- [ ] **NV-065**: NavigationPath cleared on logout (security consideration)
- [ ] **NV-066**: EnvironmentObject used for shared navigation state (not redundant copies)
- [ ] **NV-067**: No circular state where view changes route and route changes view
{{/if}}

### Tab-Based Navigation (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **NV-070**: Tab selection in @State var selectedTab: TabType
- [ ] **NV-071**: Each tab has independent NavigationPath (TabView doesn't share state)
- [ ] **NV-072**: NavigationStack created inside each tab, not wrapping TabView
- [ ] **NV-073**: Tab switching doesn't reset navigation path (tab state persists)
- [ ] **NV-074**: Returning to already-visited tab restores navigation state
- [ ] **NV-075**: No forced navigation when switching tabs
{{/if}}

### Programmatic Navigation (Important Priority)
{{#if (or (eq focus "navigation-stack") (eq focus "all") (not focus))}}
- [ ] **NV-080**: path.append() used for push navigation (type-safe)
- [ ] **NV-081**: Programmatic navigation only in response to user actions
- [ ] **NV-082**: Navigation not triggered in .onAppear or body evaluation
- [ ] **NV-083**: Navigation debounced (multiple appends prevented with flags)
- [ ] **NV-084**: Back button action explicitly clears path item when needed
- [ ] **NV-085**: Conditional navigation (e.g., auth check) uses onAppear().task()
- [ ] **NV-086**: Navigation callbacks (e.g., after form submit) handled with task/async
{{/if}}

### Back Button & Gesture Handling (Important Priority)
{{#if (or (eq focus "navigation-stack") (eq focus "all") (not focus))}}
- [ ] **NV-090**: Default back button behavior respected (swipe gesture works)
- [ ] **NV-091**: Custom back button with .navigationBackButtonHidden(true) + custom button
- [ ] **NV-092**: Back button action clears state before popping (onDisappear cleanup)
- [ ] **NV-093**: Back gesture doesn't trigger unintended side effects
- [ ] **NV-094**: Back button accessibility: .accessibilityLabel("Back")
- [ ] **NV-095**: Swipe back disabled with .gesture(DragGesture()) if needed
- [ ] **NV-096**: Confirmation alert shown before back if data would be lost
{{/if}}

### Performance & Lifecycle (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **NV-100**: Destination views created lazily (closure not evaluated immediately)
- [ ] **NV-101**: Large navigation paths don't cause memory leaks (no retained cycles)
- [ ] **NV-102**: View reuse doesn't cause stale navigation state
- [ ] **NV-103**: Navigation path not stored in Identifiable models (prevents memory issues)
- [ ] **NV-104**: ScenePhase monitored for proper state cleanup
- [ ] **NV-105**: Task cancellation on view disappear (async navigation operations)
{{/if}}

### iPad-Specific UX (Important Priority - iPad Only)
{{#if (or (eq device_context "ipad") (eq device_context "all") (not device_context))}}
- [ ] **NV-110**: Layout adapts to size classes (horizontal regular/compact detection)
- [ ] **NV-111**: NavigationSplitView hidden sidebar on iPhone (automatic via isCompact)
- [ ] **NV-112**: Landscape orientation tested on iPad (columns visible)
- [ ] **NV-113**: Sidebar width respects minimum (no cramped layouts)
- [ ] **NV-114**: Detail view has sensible default selection on iPad launch
- [ ] **NV-115**: Multitasking (Split View, Slide Over) handled gracefully
{{/if}}

### Universal App Patterns (Important Priority - Universal Only)
{{#if (or (eq device_context "universal") (eq device_context "all") (not device_context))}}
- [ ] **NV-120**: iPhone uses NavigationStack (single column)
- [ ] **NV-121**: iPad uses NavigationSplitView (multi-column)
- [ ] **NV-122**: Route enum shared between iPhone and iPad implementations
- [ ] **NV-123**: No duplicate route definitions per platform
- [ ] **NV-124**: Platform-specific layout in @Environment(\.horizontalSizeClass)
{{/if}}

### Common Mistakes (Pattern Matching)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **NV-200**: ❌ String-based routes ("user/123" in NavigationPath)
- [ ] **NV-201**: ❌ Nested NavigationStack inside NavigationStack
- [ ] **NV-202**: ❌ NavigationPath as @Published in ViewModel
- [ ] **NV-203**: ❌ State loss on app backgrounding (no SceneStorage)
- [ ] **NV-204**: ❌ Multiple navigationDestination per route type
- [ ] **NV-205**: ❌ NavigationSplitView nested inside NavigationStack
- [ ] **NV-206**: ❌ Modal presentation with NavigationStack at root
- [ ] **NV-207**: ❌ Tab-based navigation with shared NavigationPath
- [ ] **NV-208**: ❌ Deep link paths not validated for permissions
- [ ] **NV-209**: ❌ Back button behavior not customized (default may not fit UX)
- [ ] **NV-210**: ❌ Route enum not Hashable (runtime type-casting needed)
- [ ] **NV-211**: ❌ NavigationPath initialized with default values (should be empty)
{{/if}}

## Type-Safe Routing Implementation Examples

### Modern Pattern (2025 - Correct)

```swift
// Define routes as enum
enum AppRoute: Hashable, Codable {
    case event(id: String)
    case eventDetail(id: String, tab: EventTab)
    case userProfile(userId: String)
    case messages(conversationId: String)
    case shop(shopId: String)

    enum EventTab: String, Codable {
        case overview, attendees, reviews
    }
}

// Root navigation with centralized path
@main
struct PartyOnApp: App {
    @State var navigationPath = NavigationPath()

    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $navigationPath) {
                ExploreView()
                    .navigationDestination(for: AppRoute.self) { route in
                        switch route {
                        case .event(let id):
                            EventListView(eventId: id)
                        case .eventDetail(let id, let tab):
                            EventDetailView(eventId: id, selectedTab: tab)
                        case .userProfile(let userId):
                            ProfileView(userId: userId)
                        case .messages(let conversationId):
                            ConversationDetailView(conversationId: conversationId)
                        case .shop(let shopId):
                            ShopDetailView(shopId: shopId)
                        }
                    }
            }
            .environmentObject(NavigationManager(path: $navigationPath))
        }
    }
}

// View-level programmatic navigation
struct EventListView: View {
    @Environment(\.navigationManager) var navigator

    var body: some View {
        List(events) { event in
            Button(action: {
                navigator.navigate(to: .event(id: event.id))
            }) {
                EventRow(event: event)
            }
        }
    }
}

// Deep linking
struct AppDelegate: UIApplicationDelegate {
    func application(
        _ app: UIApplication,
        open url: URL,
        options: [UIApplication.OpenURLOptionsKey: Any] = [:]
    ) -> Bool {
        if let route = AppRoute.decode(from: url) {
            // Navigator handles path.append(route)
            return true
        }
        return false
    }
}
```

### Anti-Patterns (Deprecated/Incorrect)

```swift
// ❌ String-based routes
NavigationView {
    NavigationLink("Go to event", value: "event/123")  // String, not type-safe!
}

// ❌ Nested NavigationStack
NavigationStack {
    VStack {
        NavigationStack {  // WRONG - nested!
            DetailView()
        }
    }
}

// ❌ NavigationPath in ViewModel
class EventViewModel: ObservableObject {
    @Published var navigationPath = NavigationPath()  // WRONG location!
}

// ❌ Tab-based with shared path
TabView(selection: $selectedTab) {
    ForEach(tabs, id: \.self) { tab in
        NavigationStack(path: $sharedPath) {  // WRONG - shared path!
            TabViewContent(tab: tab)
        }
    }
}
```

## Deep Linking Example (Type-Safe)

```swift
extension AppRoute {
    // Decode from URL
    static func decode(from url: URL) -> AppRoute? {
        guard let components = URLComponents(url: url, resolvingAgainstBaseURL: true) else {
            return nil
        }

        let path = components.path
        let queryItems = components.queryItems ?? []

        if path == "/event" {
            guard let id = queryItems.first(where: { $0.name == "id" })?.value else {
                return nil
            }
            return .event(id: id)
        } else if path == "/event/detail" {
            guard let id = queryItems.first(where: { $0.name == "id" })?.value,
                  let tabString = queryItems.first(where: { $0.name == "tab" })?.value,
                  let tab = EventTab(rawValue: tabString) else {
                return nil
            }
            return .eventDetail(id: id, tab: tab)
        }

        return nil
    }

    // Encode to URL
    func encode() -> URL? {
        var components = URLComponents()
        components.scheme = "partyon"
        components.host = "app"

        switch self {
        case .event(let id):
            components.path = "/event"
            components.queryItems = [URLQueryItem(name: "id", value: id)]
        case .eventDetail(let id, let tab):
            components.path = "/event/detail"
            components.queryItems = [
                URLQueryItem(name: "id", value: id),
                URLQueryItem(name: "tab", value: tab.rawValue)
            ]
        default:
            break
        }

        return components.url
    }
}
```

## NavigationSplitView for iPad

```swift
struct MainView: View {
    @State var selectedItem: AppRoute?
    @State var columnVisibility = NavigationSplitViewVisibility.all

    var body: some View {
        NavigationSplitView(columnVisibility: $columnVisibility) {
            // Sidebar
            List(events, selection: $selectedItem) { event in
                NavigationLink(value: AppRoute.event(id: event.id)) {
                    EventRow(event: event)
                }
            }
            .navigationTitle("Events")
        } detail: {
            if let selectedItem {
                // Detail view based on selected route
                DetailContent(route: selectedItem)
                    .navigationTitle("Details")
            } else {
                Text("Select an event")
                    .foregroundColor(.secondary)
            }
        }
    }
}
```

## Tab-Based Navigation with Isolated Paths

```swift
struct MainTabView: View {
    @State var selectedTab = 0
    @State var exploreNavigation = NavigationPath()
    @State var mygamesNavigation = NavigationPath()
    @State var clubsNavigation = NavigationPath()

    var body: some View {
        TabView(selection: $selectedTab) {
            NavigationStack(path: $exploreNavigation) {
                ExploreView()
                    .navigationDestination(for: AppRoute.self) { $0.exploreDestination }
            }
            .tabItem { Label("Explore", systemImage: "map") }
            .tag(0)

            NavigationStack(path: $mygamesNavigation) {
                MyGamesView()
                    .navigationDestination(for: AppRoute.self) { $0.mygamesDestination }
            }
            .tabItem { Label("My Games", systemImage: "gamecontroller") }
            .tag(1)

            NavigationStack(path: $clubsNavigation) {
                ClubsView()
                    .navigationDestination(for: AppRoute.self) { $0.clubsDestination }
            }
            .tabItem { Label("Clubs", systemImage: "person.3") }
            .tag(2)
        }
    }
}
```

## Output Format

Generate a structured audit report:

```markdown
## Navigation Audit Results: [File/Component Name]

### Critical Issues (🔴)
List issues that break type-safety or prevent navigation.
Each with: Line number, description, severity, fix suggestion.

### Important Issues (🟠)
List issues that violate navigation best practices.

### Warnings (🟡)
List suboptimal patterns that work but should be improved.

### Suggestions (🔵)
List modern iOS 18 patterns for enhanced UX.

### Device-Specific Issues
#### iPhone
- [Issues specific to iPhone layouts]

#### iPad
- [Issues specific to iPad multi-column layouts]

### Summary
- Total issues: X
- Critical: X | Important: X | Warning: X | Suggestion: X
- Type-safe routing: [Score]%
- NavigationStack compliance: [Score]%
- iPad compatibility: [Score]%
- Overall compliance: X%

### Recommended Actions
1. [Most urgent: e.g., Convert string routes to enum]
2. [Second priority: e.g., Move NavigationPath to root]
3. [Third priority: e.g., Add NavigationSplitView for iPad]
4. [Fourth: e.g., Implement deep linking]

### Migration Path
If modernizing from NavigationView:
1. Replace NavigationView with NavigationStack
2. Extract routes to enum
3. Implement .navigationDestination(for:)
4. Add deep linking support
5. Test on iPad with NavigationSplitView
```

## Example Output

```markdown
## Navigation Audit Results: AppNavigation.swift

### Critical Issues (🔴)

**Line 15**: Using string-based routes in NavigationPath
```swift
// BAD
navigationPath.append("event/\(eventId)")

// GOOD
navigationPath.append(AppRoute.event(id: eventId))
```
Severity: 🔴 Critical | Confidence: 98 | Rule: NV-001

---

**Line 28**: NavigationPath stored in ViewModel instead of view
```swift
// BAD
class EventViewModel: ObservableObject {
    @Published var navigationPath = NavigationPath()
}

// GOOD
struct EventView: View {
    @State var navigationPath = NavigationPath()
}
```
Severity: 🔴 Critical | Confidence: 95 | Rule: NV-060

### Important Issues (🟠)

**Line 42**: Nested NavigationStack in detail view
```swift
// BAD
NavigationStack(path: $rootPath) {
    VStack {
        NavigationStack(path: $detailPath) {  // NESTED!
            DetailView()
        }
    }
}

// GOOD
NavigationStack(path: $rootPath) {
    DetailView()
        .navigationDestination(for: AppRoute.self) { route in
            NextView(route: route)
        }
}
```
Severity: 🟠 Important | Confidence: 92 | Rule: NV-201

---

**Line 61**: No NavigationSplitView support for iPad
```swift
// Current: iPhone only with NavigationStack
NavigationStack(path: $navigationPath) {
    ExploreView()
}

// Add iPad support:
NavigationSplitView {
    SidebarView(selection: $selectedItem)
} detail: {
    DetailView(selectedItem: selectedItem)
}
```
Severity: 🟠 Important | Confidence: 85 | Rule: NV-040

### Warnings (🟡)

**Line 35**: navigationDestination destination not lazy
```swift
// SUBOPTIMAL (destination evaluated immediately)
.navigationDestination(for: AppRoute.self) { route in
    let viewModel = expensiveInit(route)  // Computed now!
    return ExpensiveView(viewModel: viewModel)
}

// BETTER (lazy evaluation)
.navigationDestination(for: AppRoute.self) { route in
    ExpensiveView(route: route)  // Init only when pushed
}
```
Severity: 🟡 Warning | Confidence: 78 | Rule: NV-100

### Suggestions (🔵)

**State Restoration**: Add SceneStorage for multi-window apps
```swift
@SceneStorage("navigation.path") var navigationPath: NavigationPath = NavigationPath()
```
Severity: 🔵 Suggestion | Confidence: 88 | Rule: NV-064

**Deep Linking**: Implement URL decoding for universal links
Severity: 🔵 Suggestion | Confidence: 90 | Rule: NV-030

### iPad-Specific Findings

- No NavigationSplitView (iPhone-only layout)
- Tab navigation uses shared path (should be isolated per tab)
- Sidebar not implemented for large screens

### Summary
- Total issues: 7
- Critical: 2 | Important: 2 | Warning: 2 | Suggestion: 1
- Type-safe routing: 15%
- NavigationStack compliance: 50%
- iPad compatibility: 0%
- Overall compliance: 25%

### Recommended Actions
1. **URGENT**: Convert string routes to Hashable enum (NV-001, NV-007)
2. **HIGH**: Move NavigationPath from ViewModel to View layer (NV-060)
3. **HIGH**: Remove nested NavigationStack, use navigationDestination instead (NV-201)
4. **MEDIUM**: Add NavigationSplitView for iPad support (NV-040)
5. **MEDIUM**: Implement deep linking with URL decoding (NV-030)
6. **LOW**: Optimize lazy destination evaluation (NV-100)

### Migration Path
```
Week 1: Define AppRoute enum, move NavigationPath to views
Week 2: Replace NavigationView with NavigationStack + navigationDestination
Week 3: Implement NavigationSplitView for iPad
Week 4: Add deep linking support and test
```
```

## Key 2025 References

**NavigationStack & Type-Safe Routing:**
- [Mastering Navigation in SwiftUI: The 2025 Guide](https://medium.com/@dinaga119/mastering-navigation-in-swiftui-the-2025-guide-to-clean-scalable-routing-bbcb6dbce929)
- [Advanced NavigationStack Patterns](https://buczel.com/blog/swift-navigation-stack-patterns/)
- [Building a Type-Safe Routing System](https://edoardo.fyi/blog/2025/07/swiftui-routing/)

**navigationDestination Best Practices:**
- [Handling navigation the smart way](https://www.hackingwithswift.com/books/ios-swiftui/handling-navigation-the-smart-way-with-navigationdestination)
- [Programmatic Navigation with NavigationPath](https://www.donnywals.com/programmatic-navigation-in-swiftui-with-navigationpath-and-navigationdestination/)

**NavigationSplitView & iPad:**
- [NavigationSplitView Multi-Column iPad Layouts](https://medium.com/@mhamdouchi/split-view-and-multi-column-layouts-for-ipad-apps-775e0237d7a8)
- [How to create multi-column layouts](https://www.hackingwithswift.com/quick-start/swiftui/how-to-create-a-two-column-or-three-column-layout-with-navigationsplitview)

**Deep Linking & Universal Links:**
- [NavigationStack + Deep Linking in Large Apps](https://medium.com/@wesleymatlock/%EF%B8%8F-navigationstack-deep-linking-in-large-swiftui-apps-439a1ce77337)

## Navigation Checklist Summary

**Before going to production, verify:**

1. ✅ All routes defined as Hashable enum
2. ✅ NavigationPath at view root (not ViewModel)
3. ✅ No nested NavigationStack
4. ✅ All navigationDestination use lazy closures
5. ✅ iPad support with NavigationSplitView
6. ✅ Deep linking implemented and tested
7. ✅ Tab navigation uses isolated paths
8. ✅ Modal presentation doesn't affect navigation stack
9. ✅ State restored on app backgrounding (SceneStorage)
10. ✅ Back button behavior customized as needed
11. ✅ No circular state dependencies
12. ✅ Navigation paths persisted for multi-window apps


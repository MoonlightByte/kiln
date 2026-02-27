---
name: polish-ios
description: Transform good SwiftUI code into excellent, HIG-compliant production-ready code
args:
  - name: code
    type: string
    description: SwiftUI code to polish
    required: true
  - name: focus
    type: string
    enum: [state, animations, performance, accessibility, design, concurrency, all]
    description: Specific area to focus on (default: all)
    required: false
  - name: target_ios
    type: string
    enum: [ios17, ios18, ios26, latest]
    description: Target iOS version (default: ios18)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
  - polish
---

# SwiftUI iOS Code Polish Command

Elevate your SwiftUI code from good to production-excellent using cutting-edge best practices from iOS 18+ and the latest Apple design patterns. This command applies the 2025 HIG standards, visionOS-inspired design, and performance optimizations from WWDC 2025.

## Instructions

Analyze the SwiftUI code and provide a comprehensive polish report that transforms it into excellent production-ready code. Focus on:

1. **Modern State Management** - @Observable macro, @Bindable, Observation framework
2. **Navigation Excellence** - NavigationStack, type-safe routing, deep linking
3. **Animation & Motion** - Gesture-driven animations, physics-based motion, reduced motion respect
4. **Performance Optimization** - View body efficiency, lazy loading, GPU-heavy effect management
5. **Accessibility Excellence** - WCAG compliance, VoiceOver optimization, inclusive design
6. **Liquid Glass Design** - iOS 26 "Solarium" design system, translucency effects
7. **Concurrency Best Practices** - @MainActor, Task management, cancellation safety

Reference the latest WWDC 2025 guidance and Apple Design Resources.

---

## Polish Checklist

### State Management Excellence (Critical Priority)

{{#if (or (eq focus "state") (eq focus "all") (not focus))}}

**iOS 18+ (@Observable Macro)**
- [ ] **PS-001**: Using `@Observable` macro instead of `ObservableObject` protocol
- [ ] **PS-002**: Properties correctly marked as observable within `@Observable` class
- [ ] **PS-003**: Using `@Bindable` with `@State` for child view bindings
- [ ] **PS-004**: `@State` only used for local view state (never reference types)
- [ ] **PS-005**: Proper initialization: `@State private var count = 0`
- [ ] **PS-006**: No unnecessary `@StateObject` for simple value types
- [ ] **PS-007**: ViewModel marked `@MainActor` for UI updates
- [ ] **PS-008**: Using environment for dependency injection, not `@EnvironmentObject` for mutable state

**Migration Pattern (iOS 17 -> iOS 18)**
```swift
// BEFORE (iOS 17 - ObservableObject)
final class EventViewModel: ObservableObject {
    @Published var events: [Event] = []
}

// AFTER (iOS 18+ - @Observable)
@Observable
final class EventViewModel {
    var events: [Event] = []
}
```

{{/if}}

### Navigation Architecture (Important Priority)

{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}

**NavigationStack Best Practices**
- [ ] **PS-009**: Using `NavigationStack` instead of deprecated `NavigationView`
- [ ] **PS-010**: Type-safe routing with enum-based path management
- [ ] **PS-011**: Deep linking handled via `@State private var path = NavigationPath()`
- [ ] **PS-012**: Lazy destination closures to avoid unnecessary view creation
- [ ] **PS-013**: Router/Coordinator pattern for centralized navigation logic
- [ ] **PS-014**: Proper back button behavior with `.navigationBackButtonHidden` when needed
- [ ] **PS-015**: NavigationSplitView for iPad/macOS multi-column layouts

**Type-Safe Navigation Pattern**
```swift
// NavigationStack with type-safe routing
NavigationStack(path: $router.path) {
    HomeView()
        .navigationDestination(for: Route.self) { route in
            switch route {
            case .eventDetail(let id):
                EventDetailView(eventID: id)
            case .profile:
                ProfileView()
            }
        }
}
```

{{/if}}

### Animation & Motion Excellence (Important Priority)

{{#if (or (eq focus "animations") (eq focus "all") (not focus))}}

**Physics-Based Motion**
- [ ] **PS-016**: Using `.spring()` animations for natural, bouncy motion
- [ ] **PS-017**: Respecting `@Environment(\.accessibilityReduceMotion)` in all animations
- [ ] **PS-018**: Using `.animation(_:value:)` modifier with explicit value bindings
- [ ] **PS-019**: Physics parameters tuned (response: 0.3-0.5, dampingFraction: 0.7-0.9)
- [ ] **PS-020**: GPU-heavy effects (.shadow, .blur) combined into single modifier when possible

**iOS 18 Advanced Animation Features**
- [ ] **PS-021**: Using `keyframeAnimator` for multi-stage sequential animations
- [ ] **PS-022**: Using `PhaseAnimator` for looping animations with phases
- [ ] **PS-023**: Using `matchedGeometryEffect` for shared element transitions
- [ ] **PS-024**: Gesture velocity tracked for gesture-driven animations
- [ ] **PS-025**: Custom animations respect `.easeInOut`, `.snappy`, `.smooth` timing

**Reduced Motion Respect Pattern**
```swift
// GOOD - Respects accessibility preference
@Environment(\.accessibilityReduceMotion) var reduceMotion

var animationDuration: Double {
    reduceMotion ? 0 : 0.35
}

.withAnimation(.spring(response: 0.35, dampingFraction: 0.8)) {
    isExpanded.toggle()
}
```

{{/if}}

### Performance Optimization (Critical Priority)

{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}

**View Body Efficiency**
- [ ] **PS-026**: View body only contains layout, no computation or data fetching
- [ ] **PS-027**: Using `@ViewBuilder` instead of `AnyView` for conditional views
- [ ] **PS-028**: Extracting complex views into separate components (< 50 lines per view)
- [ ] **PS-029**: Stable `id(_:)` provided in all `ForEach` loops
- [ ] **PS-030**: Using `EquatableView` to skip expensive view redraws
- [ ] **PS-031**: Heavy computation moved to ViewModel or computed properties

**List Performance**
- [ ] **PS-032**: Using `List` instead of `ScrollView + VStack`
- [ ] **PS-033**: Using `LazyVStack`, `LazyHStack`, `LazyVGrid` for long lists
- [ ] **PS-034**: `.onAppear` avoided; using `.task(id:)` for async work
- [ ] **PS-035**: List items are lightweight (< 5 views per row)
- [ ] **PS-036**: Images lazy-loaded with caching strategy

**GPU-Heavy Effects Management**
- [ ] **PS-037**: `.shadow`, `.blur`, `.opacity` combined when possible
- [ ] **PS-038**: Visual effects only applied when needed, not in loops
- [ ] **PS-039**: Using `.scaleEffect` instead of `.frame(width:height:)` for minor adjustments
- [ ] **PS-040**: Custom shapes optimized with `.fill()` not `.background()`

**Environment Performance**
- [ ] **PS-041**: Frequently-changing values NOT stored in environment
- [ ] **PS-042**: Environment objects are immutable when possible
- [ ] **PS-043**: Using `@Binding` for local mutations, not environment

**WWDC 2025 Profiling**
- [ ] Performance measured with SwiftUI Instruments (new in Instruments 26)
- [ ] View body recompilations tracked and minimized
- [ ] State updates optimized (target: single recompilation per change)

{{/if}}

### Accessibility Excellence (Critical Priority)

{{#if (or (eq focus "accessibility") (eq focus "all") (not focus))}}

**WCAG Compliance (iOS 18+)**
- [ ] **PS-044**: All interactive elements have `.accessibilityLabel` (descriptive, not redundant)
- [ ] **PS-045**: All elements have 44x44pt minimum tap target (HIG standard)
- [ ] **PS-046**: Decorative images marked `.accessibilityHidden(true)`
- [ ] **PS-047**: Focus order logical with `.accessibilityElement(children:)` when needed
- [ ] **PS-048**: Custom controls implement proper focus indicators
- [ ] **PS-049**: Text contrast meets WCAG AA: 4.5:1 (normal), 3:1 (large)
- [ ] **PS-050**: Using `.accessibilityEnabled(condition:)` for conditional attributes (iOS 18+)

**VoiceOver Optimization**
- [ ] **PS-051**: All interactive elements are VoiceOver-traversable
- [ ] **PS-052**: Button/Toggle actions clearly described in accessibility label
- [ ] **PS-053**: Complex UI grouped with `.accessibilityElement(children: .combine)`
- [ ] **PS-054**: Status updates announced with `.accessibilityAnnouncement()`
- [ ] **PS-055**: Custom sliders implement `.accessibilityAdjustableAction` correctly

**Dynamic Type Support**
- [ ] **PS-056**: Using text styles (`.font(.body)`) not hardcoded sizes
- [ ] **PS-057**: Testing with 200% text scaling (Settings > Accessibility > Display & Text Size)
- [ ] **PS-058**: Layout doesn't break with large text
- [ ] **PS-059**: Line height properly scaled (no hardcoded line height values)

**Inclusive Color Design**
- [ ] **PS-060**: Not relying on color alone to convey information
- [ ] **PS-061**: Using semantic colors (`.label`, `.secondaryLabel`) not hardcoded hex
- [ ] **PS-062**: Dark mode tested and verified
- [ ] **PS-063**: Supporting `.colorSchemeContrast(.increased)` for visibility

{{/if}}

### Liquid Glass & Modern Design (Important Priority)

{{#if (or (eq focus "design") (eq focus "all") (not focus))}}

**iOS 26 "Solarium" Design System**
- [ ] **PS-064**: Using `.glassEffect()` modifier for iOS 26+ design aesthetic
- [ ] **PS-065**: Liquid Glass applied to controls, menus, and navigational elements
- [ ] **PS-066**: Translucency respects background content with proper opacity
- [ ] **PS-067**: Material design elements use `.material(.thin)`, `.material(.regular)`, `.material(.thick)`
- [ ] **PS-068**: Glass effects don't obscure critical information

**Adaptive Lighting (visionOS Inspiration)**
- [ ] **PS-069**: Light/dark mode adapted with `@Environment(\.colorScheme)`
- [ ] **PS-070**: Subtle gradient effects for depth (respecting reduce motion)
- [ ] **PS-071**: Shadows use system shadows, not hardcoded values
- [ ] **PS-072**: Color palette simplified and cohesive

**HIG Compliance (2025)**
- [ ] **PS-073**: Using semantic colors from ColorSet (not hex hardcoded)
- [ ] **PS-074**: Button styles use `.buttonStyle(.bordered)` or `.buttonStyle(.plain)`
- [ ] **PS-075**: Spacing follows 4pt grid (multiples of 4: 8, 12, 16, 20, 24)
- [ ] **PS-076**: Safe areas respected with `.ignoresSafeArea()` used consciously

{{/if}}

### Concurrency & Safety (Important Priority)

{{#if (or (eq focus "concurrency") (eq focus "all") (not focus))}}

**@MainActor Isolation**
- [ ] **PS-077**: ViewModel class marked `@MainActor`
- [ ] **PS-078**: All UI updates happen on main thread (no `DispatchQueue.main` needed)
- [ ] **PS-079**: Background work properly isolated with `@BackgroundActor` or explicit `Task.detached`

**Task Management**
- [ ] **PS-080**: Using `.task(id:)` modifier instead of `.onAppear` with `Task { }`
- [ ] **PS-081**: Checking `Task.isCancelled` in long-running operations
- [ ] **PS-082**: Proper cleanup in `.task` via `defer` or `onDisappear`
- [ ] **PS-083**: Using `async let` for concurrent operations
- [ ] **PS-084**: No force-unwrapping (`!`) of optionals; using `if let` or `??`

**Error Handling**
- [ ] **PS-085**: Proper error handling in async operations (no silent failures)
- [ ] **PS-086**: Using `Result<Success, Failure>` for explicit error handling
- [ ] **PS-087**: User-facing errors localized and actionable

**NetworkService Pattern (iOS 18+)**
```swift
@MainActor
final class EventViewModel {
    @Observable
    final class State {
        var events: [Event] = []
        var isLoading = false
        var error: String?
    }

    private let networkService: NetworkService
    @MainActor var state = State()

    func loadEvents() async {
        state.isLoading = true
        state.error = nil

        do {
            state.events = try await networkService.fetchEvents()
        } catch {
            state.error = error.localizedDescription
        }

        state.isLoading = false
    }
}
```

{{/if}}

### Data Handling (Important Priority)

{{#if (or (eq focus "all") (not focus))}}

**SwiftData Integration (iOS 17+)**
- [ ] **PS-088**: Using `@Query` macro for data fetching instead of manual observation
- [ ] **PS-089**: Model classes properly decorated with `@Model` macro
- [ ] **PS-090**: Relationships configured with proper cascade delete/update rules
- [ ] **PS-091**: Using `.modelContainer` environment modifier

**Codable Compliance**
- [ ] **PS-092**: Model conforming to `Codable` when needed
- [ ] **PS-093**: Custom `Decodable` implementation for complex JSON structures
- [ ] **PS-094**: Proper error handling for decoding failures

{{/if}}

---

## Code Polish Template

### Input Code

```swift
{{code}}
```

---

## Output Format

Generate a comprehensive polish report following this structure:

```markdown
## SwiftUI Code Polish Report

### Summary
- **Current State**: [Brief assessment]
- **Target Level**: Production Excellence
- **Polish Opportunities**: [Count] issues found

### Critical Issues (🔴)
[Issues that block accessibility, crash, or violate HIG]

Each with:
- Line number and description
- Current code snippet
- Polished code snippet
- Explanation of improvement
- Rule: PS-XXX

### Important Issues (🟠)
[Issues that reduce code quality or UX]

### Warnings (🟡)
[Suboptimal patterns that work but should improve]

### Polish Suggestions (💎)
[Excellence opportunities - cutting-edge patterns]

### Modern Architecture Recommendations

**State Management Pattern**
```swift
[Recommended @Observable pattern]
```

**Navigation Pattern**
```swift
[Type-safe NavigationStack pattern]
```

**Async/Concurrency Pattern**
```swift
[@MainActor + .task pattern]
```

### Performance Profile
- View recompilations: [Analysis]
- GPU-heavy effects: [List and recommendations]
- List performance: [LazyVStack vs VStack recommendation]

### Accessibility Score
- Label coverage: X%
- VoiceOver readiness: [Good/Needs work]
- Dynamic Type support: [Good/Needs work]
- WCAG Compliance: X/100

### iOS 18+ Features Applied
- [ ] @Observable macro
- [ ] NavigationStack (type-safe)
- [ ] keyframeAnimator
- [ ] @Bindable
- [ ] accessibilityEnabled modifier

### iOS 26 "Solarium" Design Upgrades
- [ ] Liquid Glass effects
- [ ] Adaptive lighting
- [ ] Simplified color palette
- [ ] Translucency applied correctly

### Recommended Next Steps
1. [Most critical polish]
2. [Performance optimization]
3. [Accessibility improvement]
4. [Design modernization]

### Code Quality Metrics
- **Before**: [Lines, complexity]
- **After**: [Lines, complexity]
- **Improvement**: [Clarity, performance, maintainability]
```

---

## Example Polish Report

### Input Code
```swift
class EventViewModel: ObservableObject {
    @Published var events: [Event] = []
    @Published var isLoading = false

    func loadEvents() {
        isLoading = true
        DispatchQueue.main.async {
            // Network call
            self.isLoading = false
        }
    }
}

struct EventListView: View {
    @ObservedObject var viewModel = EventViewModel()

    var body: some View {
        NavigationView {
            ScrollView {
                VStack {
                    ForEach(viewModel.events) { event in
                        HStack {
                            VStack(alignment: .leading) {
                                Text(event.title)
                                    .font(.system(size: 18, weight: .bold))
                                Text(event.date)
                                    .font(.system(size: 14))
                            }
                            Spacer()
                        }
                        .padding(12)
                        .background(Color(red: 0.95, green: 0.95, blue: 0.95))
                        .cornerRadius(8)
                    }
                }
            }
            .navigationTitle("Events")
        }
    }
}
```

### Polish Report Output

```markdown
## SwiftUI Code Polish Report: EventListView

### Summary
- **Current State**: Functional iOS 16-era code using deprecated patterns
- **Target Level**: iOS 18+ Production Excellence
- **Polish Opportunities**: 12 modernizations available

### Critical Issues (🔴)

**Issue 1: Using @ObservedObject for owned instance (Line 29)**

```swift
// CURRENT - Recreates ViewModel on every parent update!
@ObservedObject var viewModel = EventViewModel()

// POLISHED - Use @StateObject for owned instances
@StateObject private var viewModel = EventViewModel()
```

Severity: 🔴 Critical | Rule: PS-002
The ViewModel is recreated whenever the parent View redraws, losing all state. This is a performance and correctness issue.

---

**Issue 2: Using deprecated @ObservableObject protocol (Line 1)**

```swift
// CURRENT (iOS 17)
class EventViewModel: ObservableObject {
    @Published var events: [Event] = []
}

// POLISHED (iOS 18+)
@Observable
final class EventViewModel {
    var events: [Event] = []
}
```

Severity: 🔴 Critical | Rule: PS-001
The @Observable macro provides 6x faster list updates (WWDC 2025), cleaner syntax, and precise property-level invalidation. This is the modern pattern.

---

**Issue 3: Missing accessibility labels on text elements (Line 34-37)**

```swift
// CURRENT - No context for VoiceOver
Text(event.title)
    .font(.system(size: 18, weight: .bold))

// POLISHED
Text(event.title)
    .font(.headline)
    .accessibilityElement(children: .combine)
```

Severity: 🔴 Critical | Rule: PS-044
VoiceOver users hear uncontextualized text. Add semantic labels.

---

### Important Issues (🟠)

**Issue 4: Hardcoded font sizes instead of text styles (Line 34)**

```swift
// CURRENT - Won't scale with Dynamic Type
.font(.system(size: 18, weight: .bold))

// POLISHED - Scales automatically
.font(.headline)
```

Severity: 🟠 Important | Rule: PS-056
Hardcoded sizes fail for users with accessibility text scaling enabled. Text styles automatically adapt.

---

**Issue 5: Using VStack for potentially long list (Line 26)**

```swift
// CURRENT - Renders ALL items at once (poor performance)
ScrollView {
    VStack {
        ForEach(viewModel.events) { event in
            EventRow(event: event)
        }
    }
}

// POLISHED - Renders only visible items
List(viewModel.events) { event in
    EventRow(event: event)
}
```

Severity: 🟠 Important | Rule: PS-032
VStack renders all 1000 events even if only 5 are visible. List uses lazy rendering. WWDC 2025 shows 16x performance improvement with List.

---

**Issue 6: Using main queue dispatch instead of async/await (Line 15)**

```swift
// CURRENT - Outdated concurrency pattern
DispatchQueue.main.async {
    self.isLoading = false
}

// POLISHED - Modern Swift Concurrency
@MainActor
final class EventViewModel {
    func loadEvents() async {
        isLoading = true
        events = try await networkService.fetchEvents()
        isLoading = false
    }
}
```

Severity: 🟠 Important | Rule: PS-077
@MainActor eliminates manual main queue dispatch. Cleaner, safer, faster.

---

### Warnings (🟡)

**Issue 7: Using NavigationView (deprecated) (Line 23)**

```swift
// CURRENT (iOS 16)
NavigationView {
    // content
}

// POLISHED (iOS 16+, recommended iOS 18)
NavigationStack(path: $path) {
    // content
        .navigationDestination(for: Route.self) { route in
            // handle routes
        }
}
```

Severity: 🟡 Warning | Rule: PS-009
NavigationView is deprecated. NavigationStack offers type-safe routing and better performance.

---

### Polish Suggestions (💎)

**Enhancement 1: Apply Liquid Glass Design (iOS 26)**

```swift
// iOS 26 - Solarium Design System
EventRow(event: event)
    .glassEffect()  // New iOS 26 feature
```

The Solarium project brings visionOS-inspired translucency to iOS 26, creating an airier, more modern look.

---

**Enhancement 2: Add gesture-driven animations**

```swift
// iOS 18 - Physics-based motion
@State private var isExpanded = false

var body: some View {
    EventRow()
        .onTapGesture {
            withAnimation(.spring(response: 0.35, dampingFraction: 0.8)) {
                isExpanded.toggle()
            }
        }
}
```

Spring animations feel natural. WWDC 2025 emphasizes gesture-velocity tracking for fluid interactions.

---

### Modern Architecture Recommendations

**@Observable Pattern (iOS 18+)**
```swift
@Observable
final class EventViewModel {
    var events: [Event] = []
    var isLoading = false
    var error: String?

    @MainActor
    func loadEvents() async {
        isLoading = true
        error = nil

        do {
            events = try await networkService.fetchEvents()
        } catch {
            error = error.localizedDescription
        }

        isLoading = false
    }
}

struct EventListView: View {
    @State var viewModel = EventViewModel()

    var body: some View {
        List(viewModel.events) { event in
            EventRow(event: event)
        }
        .navigationTitle("Events")
        .task {
            await viewModel.loadEvents()
        }
    }
}
```

This eliminates @Published, uses @MainActor properly, leverages .task for async, and uses List for performance.

---

### Performance Profile

- **View recompilations**: Current: 500+ (every event change). Polished: 2-3 (only affected rows)
- **GPU-heavy effects**: Current: None. Polished: Add glassEffect() with translucency
- **List rendering**: Current: All items (poor). Polished: Lazy rendering (excellent)
- **Memory impact**: Current: ~10MB for 1000 items. Polished: ~1MB (10x improvement)

### Accessibility Score
- **Label coverage**: 30% → 95% ✅
- **VoiceOver readiness**: ❌ Needs work → ✅ Ready
- **Dynamic Type support**: ❌ Broken at 200% → ✅ Perfect scaling
- **WCAG Compliance**: 45/100 → 92/100

### iOS 18+ Features Applied
- ✅ @Observable macro
- ✅ NavigationStack (type-safe)
- ✅ keyframeAnimator (optional)
- ✅ @MainActor isolation
- ✅ .task modifier
- ✅ accessibilityEnabled modifier

### iOS 26 "Solarium" Design Upgrades
- ✅ Liquid Glass effects (.glassEffect())
- ✅ Simplified semantic colors
- ✅ Adaptive lighting patterns
- ✅ Translucency for depth

### Recommended Next Steps
1. Migrate to @Observable and @MainActor (critical for performance)
2. Replace VStack with List for 10x performance gain
3. Add .accessibilityLabel to all interactive elements
4. Apply .glassEffect() for iOS 26 "Solarium" aesthetic
5. Implement physics-based spring animations
6. Add proper error state handling UI

### Code Quality Metrics
- **Before**: 45 lines, 3 issues, poor performance, inaccessible
- **After**: 52 lines (more readable), 0 issues, 10x performance, WCAG 92/100
- **Improvement**: Modern, maintainable, accessible, performant iOS 18+ code
```

---

## Resources & References

**WWDC 2025 Guidance**
- [Catch up on accessibility in SwiftUI - WWDC24](https://developer.apple.com/videos/play/wwdc2024/10073/)
- [Optimize SwiftUI performance with Instruments - WWDC25](https://developer.apple.com/videos/play/wwdc2025/306/)
- [Build a SwiftUI app with the new design - WWDC25](https://developer.apple.com/videos/play/wwdc2025/323/)

**Apple Documentation**
- [Migrating from ObservableObject to @Observable](https://developer.apple.com/documentation/swiftui/migrating-from-the-observable-object-protocol-to-the-observable-macro)
- [Understanding and improving SwiftUI performance](https://developer.apple.com/documentation/Xcode/understanding-and-improving-swiftui-performance)
- [Accessibility modifiers](https://developer.apple.com/documentation/swiftui/view-accessibility)
- [Applying Liquid Glass to custom views](https://developer.apple.com/documentation/SwiftUI/Applying-Liquid-Glass-to-custom-views)

**Modern Patterns & Best Practices**
- [Modern SwiftUI Navigation: Best Practices for 2025 Apps](https://medium.com/@dinaga119/mastering-navigation-in-swiftui-the-2025-guide-to-clean-scalable-routing-bbcb6dbce929)
- [Modern MVVM in SwiftUI 2025: Clean Architecture](https://medium.com/@minalkewat/modern-mvvm-in-swiftui-2025-the-clean-architecture-youve-been-waiting-for-72a7d576648e)
- [24 SwiftUI Performance Tips Every iOS Developer Should Know (2025)](https://medium.com/@ravisolankice12/24-swiftui-performance-tips-every-ios-developer-should-know-2025-edition-723340d9bd79)
- [SwiftUI Animations 2025: Physics, Gestures & Advanced Techniques](https://medium.com/@bhumibhuva18/swiftui-animations-in-2025-beyond-basic-transitions-f63db40c7c46)

**Accessibility Excellence**
- [CVS Health iOS SwiftUI Accessibility Techniques](https://github.com/cvs-health/ios-swiftui-accessibility-techniques)
- [WCAG Applicability to Native iOS Apps](https://knowbility.org/programs/john-slatin-accessu-2025/swiftui-accessibility-techniques-for-native-ios-apps/)
- [AccessLint — iOS Accessibility Analysis at Build Time](https://accesslint.app/)

**Design System & visionOS Inspiration**
- [visionOS Design Patterns & iOS 26 Design Overhaul](https://applemagazine.com/apples-ios-26-to-introduce-visionos-inspired/)
- [Liquid Glass & Solarium Design System - Apple](https://www.apple.com/newsroom/2025/06/apple-introduces-a-delightful-and-elegant-new-software-design/)

---

## Implementation Strategy

1. **Phase 1 - State (Critical)**: Migrate to @Observable + @MainActor
2. **Phase 2 - Performance**: Replace VStack with List, optimize view bodies
3. **Phase 3 - Accessibility**: Add labels, test with VoiceOver
4. **Phase 4 - Design**: Apply Liquid Glass, modern colors, animations
5. **Phase 5 - Polish**: Gesture interactions, physics-based animations

---

**Key Insight**: iOS 18+ code using @Observable and @MainActor is dramatically simpler, faster, and more accessible than pre-iOS 18 patterns. This polish command helps you leverage 2025's best practices.

---
name: simplify-compose
description: Refactor complex Jetpack Compose code into clean, maintainable patterns with performance optimization
args:
  - name: code
    type: string
    description: Kotlin Compose code to refactor and simplify
    required: true
  - name: focus
    type: string
    enum: [recomposition, state-hoisting, slot-api, performance, extraction, all]
    description: Specific refactoring focus area (default: all)
    required: false
  - name: complexity_target
    type: string
    enum: [minimal, moderate, advanced]
    description: Target complexity level after refactoring (default: moderate)
    required: false
optimization_areas:
  - compiler-stability
  - recomposition-minimization
  - state-management
  - component-extraction
  - memory-efficiency
---

# Jetpack Compose Code Simplification and Refactoring

Refactor complex Compose code into clean, maintainable patterns aligned with 2025-2026 best practices. Focus on compiler optimizations, state management, and performance profiling.

## Instructions

Analyze the provided Compose code and suggest simplifications using modern Compose patterns. Reference the Compose compiler optimizations, stability analysis, state hoisting best practices, and performance optimization techniques.

This command applies research-backed patterns from:
- Jetpack Compose December 2025 performance parity milestone (pausable composition, background text prefetching)
- Compose Compiler Plugin stability analysis and metadata inference
- State hoisting and recomposition optimization best practices
- Slot API patterns for flexible, reusable components
- CompositionLocal usage for cross-cutting concerns
- `derivedStateOf` and `remember` for performance optimization

## Refactoring Checklist

### Recomposition Minimization (Critical Priority)
{{#if (or (eq focus "recomposition") (eq focus "all") (not focus))}}
- [ ] **REC-001**: Not reading state in composition scope unnecessarily
- [ ] **REC-002**: Using `derivedStateOf` for computed values that rarely change
- [ ] **REC-003**: Wrapping heavy operations in `remember` with correct dependencies
- [ ] **REC-004**: Using `compositionLocalOf` (not `staticCompositionLocalOf`) for frequently changing values
- [ ] **REC-005**: Lambdas wrapped in `remember` to prevent unnecessary recomposition of children
- [ ] **REC-006**: LazyColumn/LazyRow with stable keys to prevent spurious recomposition
- [ ] **REC-007**: Not passing `MutableState` directly to children (hoist state up instead)
- [ ] **REC-008**: Using `skipToLookaheadSize()` in LazyLayout to reduce layout passes
{{/if}}

### State Hoisting (Critical Priority)
{{#if (or (eq focus "state-hoisting") (eq focus "all") (not focus))}}
- [ ] **SH-001**: All state lifted to correct parent composable
- [ ] **SH-002**: State holder class created for logically related state values
- [ ] **SH-003**: Events passed downward as lambdas (State Up, Events Down pattern)
- [ ] **SH-004**: Child composables are stateless UI building blocks
- [ ] **SH-005**: Minimal state at composable level (ViewModels for app-level state)
- [ ] **SH-006**: State restored correctly on process death (use `rememberSaveable` for UI state)
- [ ] **SH-007**: Scroll state hoisted to parent with lambda for lazy reading
- [ ] **SH-008**: No state initialization inside composition (use `remember { ... }` or `rememberSaveable`)
{{/if}}

### Slot API Patterns (High Priority)
{{#if (or (eq focus "slot-api") (eq focus "all") (not focus))}}
- [ ] **SLOT-001**: Components accept `@Composable` lambda parameters for flexibility
- [ ] **SLOT-002**: Scaffold pattern used for pluggable layout structure
- [ ] **SLOT-003**: Multiple slots defined for different UI regions
- [ ] **SLOT-004**: Single responsibility maintained (one component = one purpose)
- [ ] **SLOT-005**: Prop-drilling eliminated via slot parameters
- [ ] **SLOT-006**: Type constraints enforced for slot parameters where needed
- [ ] **SLOT-007**: Default slot content provided when applicable
- [ ] **SLOT-008**: Slot API replaces multiple specialized component variants
{{/if}}

### Compiler Stability and Optimization (Critical Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **COMP-001**: Data classes marked `@Immutable` or `@Stable` for compiler inference
- [ ] **COMP-002**: Collection types use `ImmutableList` from Kotlin collections
- [ ] **COMP-003**: Unstable types wrapped in value holders for stability
- [ ] **COMP-004**: Using Compose Compiler 1.5.0+ with pausable composition enabled
- [ ] **COMP-005**: Custom objects implement structural equality (`data class` or custom `equals`)
- [ ] **COMP-006**: No var properties in data that's used in composition (use val only)
- [ ] **COMP-007**: Background text measurement enabled for offloaded prefetching
- [ ] **COMP-008**: Modifier.onPlaced and Modifier.onVisibilityChanged optimized
{{/if}}

### CompositionLocal Patterns (High Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **CL-001**: `compositionLocalOf` used for values that change frequently
- [ ] **CL-002**: `staticCompositionLocalOf` used only for rarely-changing foundational values (theme)
- [ ] **CL-003**: CompositionLocals scoped to smallest needed subtree
- [ ] **CL-004**: Good default values provided for all CompositionLocals
- [ ] **CL-005**: CompositionLocal not used for app logic (use parameters or ViewModels)
- [ ] **CL-006**: Cross-cutting concerns (theming, spacing) use CompositionLocal
- [ ] **CL-007**: Performance impact measured with Compose Stability Analyzer
{{/if}}

### Component Extraction (Important Priority)
{{#if (or (eq focus "extraction") (eq focus "all") (not focus))}}
- [ ] **EXT-001**: Complex composables broken into smaller, reusable pieces
- [ ] **EXT-002**: Each extracted composable has single responsibility
- [ ] **EXT-003**: Extracted composables are previewed independently
- [ ] **EXT-004**: Extracted composables are testable in isolation
- [ ] **EXT-005**: Layout logic separated from state management
- [ ] **EXT-006**: Display logic separated from business logic
- [ ] **EXT-007**: Conditional branches extracted to separate composables
- [ ] **EXT-008**: Repetitive composable patterns extracted to reusable factory
{{/if}}

### Memory and Lifecycle Efficiency (High Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **MEM-001**: `DisposableEffect` used for resources needing cleanup
- [ ] **MEM-002**: API calls in `LaunchedEffect`, not in composition body
- [ ] **MEM-003**: No memory leaks from long-lived lambdas capturing this
- [ ] **MEM-004**: Animations completed with `animateValueAsState` (not manual updates)
- [ ] **MEM-005**: Image loading uses Coil/Glide with memory caching
- [ ] **MEM-006**: Heavy objects held in `remember` with correct dependency tracking
- [ ] **MEM-007**: Coroutine scope cancelled on composable disposal
{{/if}}

## Code to Refactor

```kotlin
{{code}}
```

## Output Format

Generate a structured refactoring report:

```markdown
## Refactoring Analysis: [Component Name]

### Current State Assessment
- Complexity level: [Low/Moderate/High]
- Recomposition count: [estimate based on code]
- Performance bottlenecks: [3-5 specific issues]
- State management: [assessment]

### Recommended Refactorings (Priority Order)

#### 1. [First Priority Issue]
**Category**: [Recomposition/State Hoisting/Slot API/Compiler/etc]
**Severity**: [Critical/Important/Warning]
**Current Code**:
\`\`\`kotlin
// Problem code snippet
\`\`\`

**Issues**:
- [Specific issue 1]
- [Specific issue 2]

**Refactored Code**:
\`\`\`kotlin
// Improved code snippet
\`\`\`

**Benefits**:
- Recomposition reduced by ~X%
- [Other performance benefit]
- Improved testability/maintainability

---

#### 2. [Second Priority Issue]
...

### Compiler Stability Analysis

**Type Stability Assessment**:
- [Data class/type A]: [Stable/Unstable] - [reason]
- [Function parameter B]: [Stable/Unstable] - [reason]

**Recommendations**:
- Mark types with `@Immutable` or `@Stable` where applicable
- Replace unstable types with stable wrappers
- Use `ImmutableList` instead of `MutableList`

### State Hoisting Improvements

**Current State Tree**:
- Local state: [list current hoisted state and where it lives]
- Lifting opportunity: [describe what can be hoisted]

**Improved State Tree**:
- [New state structure diagram or description]
- [Explanation of benefits]

### Slot API Opportunities

**Reusable Components**:
- Component A currently has N variants → can be unified with Slot API
- Component B accepts fixed content → can be flexible with lambdas

**Refactoring Path**:
\`\`\`kotlin
// Before: Multiple specialized components
class EventCard(isFavorite: Boolean) { ... }
class FeaturedEventCard(isFavorite: Boolean) { ... }

// After: Single flexible component with slots
fun EventCard(
    isFavorite: Boolean,
    header: @Composable () -> Unit = { DefaultHeader() },
    actions: @Composable () -> Unit = { DefaultActions() }
) { ... }
\`\`\`

### Memory and Lifecycle

**Potential Issues**:
- [Memory leak or inefficiency A]
- [Lifecycle issue B]

**Fixes**:
- [Solution 1]
- [Solution 2]

### Performance Metrics (Post-Refactor Estimates)

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Recomposition Count (per state change) | X | Y | Z% |
| Memory Usage | Xmb | Ymb | Z% |
| Initial Composition Time | Xms | Yms | Z% |
| Scroll Frame Drops | X | Y | Z% |

### Testing Checklist

- [ ] Component previews still display correctly
- [ ] State preservation works on config change
- [ ] No crashes on rapid state changes
- [ ] Memory profiler shows no leaks
- [ ] Compose recomposition profiler confirms reduced unnecessary recomposition
- [ ] Accessibility maintained (semantics unchanged)

### Implementation Steps

1. [First step]
2. [Second step]
3. [Verification step]

### Additional Recommendations

- Use Compose Stability Analyzer IDE plugin for real-time stability feedback
- Profile with Compose Recomposition Highlighter during development
- Consider extracting ViewModels for complex state management
- Document state flow with diagrams for team reference
```

## Example Refactoring Output

```markdown
## Refactoring Analysis: EventListScreen

### Current State Assessment
- Complexity level: High
- Recomposition count: ~15 per item click (should be ~1)
- Performance bottlenecks:
  1. Search query read at composition level (not in derivedStateOf)
  2. Filter list created inline (not in remember)
  3. Event items not memoized with keys
  4. State hoisted only to composable level, not to ViewModel

### Recommended Refactorings (Priority Order)

#### 1. Use derivedStateOf for Filtered Events
**Category**: Recomposition Minimization
**Severity**: Critical
**Current Code**:
```kotlin
@Composable
fun EventListScreen(viewModel: EventListViewModel) {
    val searchQuery by viewModel.searchQuery.collectAsState()
    val allEvents by viewModel.events.collectAsState()

    // Filters every recomposition, even when searchQuery doesn't change
    val filteredEvents = allEvents.filter {
        it.title.contains(searchQuery, ignoreCase = true)
    }

    LazyColumn {
        items(filteredEvents) { event ->
            EventCard(event)
        }
    }
}
```

**Issues**:
- `filteredEvents` recomputed on every composition frame
- Inefficient with large event lists
- Child recomposition caused by parent reading state directly

**Refactored Code**:
```kotlin
@Composable
fun EventListScreen(viewModel: EventListViewModel) {
    val searchQuery by viewModel.searchQuery.collectAsState()
    val allEvents by viewModel.events.collectAsState()

    // Only recompute when search query actually changes
    val filteredEvents by remember {
        derivedStateOf {
            allEvents.filter {
                it.title.contains(searchQuery, ignoreCase = true)
            }
        }
    }

    LazyColumn {
        items(filteredEvents, key = { it.id }) { event ->
            EventCard(event)
        }
    }
}
```

**Benefits**:
- Recomposition reduced by ~80% (only when filter result changes)
- Scroll performance improved
- List items use stable keys to prevent spurious recomposition

---

#### 2. Extract Event Cards as Stable Memoized Components
**Category**: Compiler Stability & Component Extraction
**Severity**: Important

**Current Code**:
```kotlin
LazyColumn {
    items(events) { event ->
        EventCard(
            event = event,
            onFavorite = { viewModel.toggleFavorite(event.id) },
            onJoin = { viewModel.joinEvent(event.id) },
            onNavigate = { navigation.navigate("detail/${event.id}") }
        )
    }
}
```

**Issues**:
- Lambda passed inline, not stable
- Event object may be mutable
- No key provided for list items

**Refactored Code**:
```kotlin
@Composable
internal fun MemoizedEventCard(
    event: Event,
    onFavorite: (eventId: String) -> Unit,
    onJoin: (eventId: String) -> Unit,
    onNavigate: (route: String) -> Unit,
    modifier: Modifier = Modifier
) {
    EventCard(
        event = event,
        onFavorite = { onFavorite(event.id) },
        onJoin = { onJoin(event.id) },
        onNavigate = { onNavigate("detail/${event.id}") },
        modifier = modifier
    )
}

// In list:
LazyColumn {
    items(
        events,
        key = { it.id }  // Stable key prevents spurious recomposition
    ) { event ->
        MemoizedEventCard(
            event = event.copy(),  // Ensure immutable copy
            onFavorite = { viewModel.toggleFavorite(it) },
            onJoin = { viewModel.joinEvent(it) },
            onNavigate = navigation::navigate
        )
    }
}
```

**Benefits**:
- Compiler can infer stability of extracted component
- List reordering no longer causes full re-layout
- Child composables memoized by list key

---

#### 3. Use Slot API for Flexible Header
**Category**: Slot API & Component Extraction
**Severity**: Important

**Current Code**:
```kotlin
@Composable
fun EventListScreen(showHeader: Boolean) {
    if (showHeader) {
        Text("Events Near You", style = MaterialTheme.typography.headlineMedium)
    }

    LazyColumn {
        items(events) { ... }
    }
}

// Multiple variants created:
fun EventListScreenWithSearch() { ... }
fun EventListScreenWithFilter() { ... }
fun EventListScreenWithSort() { ... }
```

**Issues**:
- Multiple component variants for single idea
- Prop-drilling to control header
- Not reusable

**Refactored Code**:
```kotlin
@Composable
fun EventListScreen(
    events: List<Event>,
    modifier: Modifier = Modifier,
    header: @Composable (() -> Unit)? = {
        Text("Events Near You", style = MaterialTheme.typography.headlineMedium)
    },
    emptyState: @Composable (() -> Unit)? = { DefaultEmptyState() }
) {
    Column(modifier = modifier.fillMaxSize()) {
        header?.invoke()

        if (events.isEmpty()) {
            emptyState?.invoke()
        } else {
            LazyColumn {
                items(events, key = { it.id }) { event ->
                    EventCard(event)
                }
            }
        }
    }
}

// Usage - single component, multiple configurations:
EventListScreen(events = events)  // Default header
EventListScreen(events = events, header = null)  // No header
EventListScreen(
    events = events,
    header = {
        SearchBar(onSearch = { viewModel.search(it) })
    }
)
```

**Benefits**:
- Single reusable component replaces 3 variants
- Caller controls content, not component
- Fully flexible without breaking changes

### Compiler Stability Analysis

**Type Stability Assessment**:
- `Event` data class: **Unstable** - Contains `DateTime` and mutable metadata fields
- `List<Event>`: **Unstable** - Mutable lists not recognized by compiler
- Filter lambda: **Unstable** - Lambdas are always unstable

**Recommendations**:
```kotlin
// Before: Unstable data class
data class Event(
    val id: String,
    val title: String,
    val startTime: DateTime,  // Unstable type
    var isFavorite: Boolean   // Mutable field
)

// After: Stable data class
@Immutable
data class Event(
    val id: String,
    val title: String,
    val startTime: Long,  // Use Long instead of DateTime
    val isFavorite: Boolean  // Immutable (val)
)

// Use immutable list
val events: ImmutableList<Event> = persistentListOf(...)
```

### State Hoisting Improvements

**Current State Tree**:
- `searchQuery` and `events` live in ViewModel (correct)
- `filteredEvents` computed in composable (should be in ViewModel or derivedStateOf)
- `selectedFilter` passed through 3 levels (prop-drilling)

**Improved State Tree**:
```
ViewModel
├── events: StateFlow<List<Event>>
├── searchQuery: StateFlow<String>
├── selectedFilter: StateFlow<FilterType>
└── filteredEvents: Flow<List<Event>>  // Computed in ViewModel

Composable (stateless UI)
├── Read: filteredEvents, selectedFilter
├── Emit: onSearch, onFilterChange
```

### Memory and Lifecycle

**Potential Issues**:
- Search query Flow not cancelled when screen closed
- Event callbacks captured in composable scope (potential memory leak)

**Fixes**:
```kotlin
// Use .collectAsStateWithLifecycle() instead of .collectAsState()
val events by viewModel.events.collectAsStateWithLifecycle()

// Memoize callbacks to prevent recreation
val onFavorite = remember { { id: String -> viewModel.toggleFavorite(id) } }
```

### Performance Metrics (Post-Refactor Estimates)

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Recomposition Count | 15 per action | 1 per action | 93% |
| Memory Usage | 2.4 MB | 1.8 MB | 25% |
| Scroll Frame Drops | 8 per second | 0 per second | 100% |
| Initial Load Time | 450ms | 280ms | 38% |

### Testing Checklist

- [x] Component previews render correctly
- [x] Search and filter still work
- [x] No recomposition on state that doesn't affect item
- [x] Memory profile shows ~25% reduction
- [x] Scroll is buttery smooth
- [ ] Accessibility: Search field still announced correctly (verify with TalkBack)

### Implementation Steps

1. Mark `Event` with `@Immutable` annotation
2. Move `filteredEvents` to `derivedStateOf` in composable
3. Add `key = { it.id }` to LazyColumn items
4. Extract memoized item composable
5. Refactor multiple screen variants into single component with Slot API
6. Replace `.collectAsState()` with `.collectAsStateWithLifecycle()`
7. Run Compose Recomposition Highlighter to verify improvements
```

## 2025-2026 Best Practices Reference

### Compiler Optimizations (Dec 2025)
- **Pausable Composition**: Enabled by default - runtime pauses composition if frame budget exceeded
- **Background Text Prefetching**: Offloads text measurement to background threads via `LocalBackgroundTextMeasurementExecutor`
- **Modifier Optimization**: Reduced overhead in `Modifier.onPlaced` and `Modifier.onVisibilityChanged`
- **Stability Analyzer IDE Plugin**: Real-time feedback on type stability in Android Studio

### State Management Pattern (2025)
1. **Move state to correct level**: Only in composables that need it
2. **Hoist to ViewModel for app-level state**: Use `StateFlow` or `LiveData`
3. **Use `derivedStateOf` for computed values**: Prevent unnecessary recomposition
4. **Encapsulate related state**: Create dedicated state holder classes
5. **Pass events downward**: State Up, Events Down - unidirectional data flow

### Slot API Pattern (2025)
- **Define "slots"**: Accept `@Composable` lambda parameters for flexible areas
- **Provide defaults**: Sensible defaults allow simple use without configuration
- **Multiple slots**: Different regions can have different content
- **Example**: Scaffold with header, content, footer slots
- **Replaces variants**: Single component with slots = many specialized components

### Performance Profiling Tools (2025)
- **Compose Stability Analyzer**: IDE plugin for real-time stability analysis
- **Recomposition Highlighter**: Debug tool highlighting unnecessary recompositions
- **Modifier Tracing**: Trace modifier creation and application
- **Memory Profiler**: Detect memory leaks in composition tree

## References and Further Reading

Research and patterns sourced from:
- [Jetpack Compose Complete Roadmap for 2025 - Medium](https://medium.com/@ami0275/jetpack-compose-complete-roadmap-for-2025-eec23b780d84)
- [The State of Jetpack Compose in 2025 - DVT Software Engineering](https://medium.com/dvt-engineering/the-state-of-jetpack-compose-in-2025-987145a773fb)
- [Jetpack Compose Performance Matches Views: December 2025 - ByteIOTA](https://byteiota.com/jetpack-compose-performance-matches-views-december-2025/)
- [Compose Stability Inference - GitHub](https://github.com/skydoves/compose-stability-inference)
- [Jetpack Compose Performance - Android Developers](https://developer.android.com/develop/ui/compose/performance)
- [Slot APIs in Jetpack Compose - Dharma Kshetri](https://medium.com/@dharmakshetri/slot-apis-in-jetpack-compose-bacd7e4fe43c)
- [State and Jetpack Compose - Android Developers](https://developer.android.com/develop/ui/compose/state)
- [CompositionLocal Patterns and Performance - Monika Bhasin](https://medium.com/@monikabhasin.sd17/compositionlocal-explained-patterns-pitfalls-and-performance-tips-for-jetpack-compose-099c6654358a)
- [Jetpack Compose derivedStateOf Explained - Tailorviren](https://medium.com/@tailorviren96/jetpack-compose-derivedstateof-explained-with-real-ui-example-performance-optimization-guide-38df0b8173d4)
- [Jetpack Compose in 2025: Architecture Patterns - Jamshidbek Boynazarov](https://jamshidbekboynazarov.medium.com/jetpack-compose-in-2025-architecture-patterns-for-scalable-ui-7b98457938ac)

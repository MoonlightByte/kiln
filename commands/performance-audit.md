---
name: performance-audit
description: Comprehensive mobile app performance audit covering startup, memory, scrolling, and rendering
args:
  - name: code
    type: string
    description: Mobile code (Kotlin/Compose or Swift/SwiftUI) to audit
    required: true
  - name: platform
    type: string
    enum: [android, ios, auto]
    description: Target platform (default: auto-detect based on syntax)
    required: false
  - name: focus_area
    type: string
    enum: [startup, memory, scrolling, rendering, network, battery, baseline-profiles, all]
    description: Specific area to focus on (default: all)
    required: false
  - name: target_metrics
    type: string
    enum: [aggressive, balanced, conservative]
    description: Performance target level (default: balanced)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Mobile Performance Audit

Comprehensive performance audit for mobile applications covering startup time, memory management, rendering, scrolling, network efficiency, and battery impact.

## Instructions

Analyze the code for performance compliance. Auto-detect platform based on syntax (`@Composable` = Android, `@State` + `some View` = iOS). Report specific metrics and optimization opportunities based on industry benchmarks for 2025.

## Performance Targets

### Target Metrics (by level)

**Aggressive (Premium Devices)**
- Cold start: < 300ms
- Scroll FPS: 60fps consistently
- Memory footprint: < 80MB baseline
- Compose recompositions: < 10% unnecessary

**Balanced (Mid-Range Devices) - DEFAULT**
- Cold start: < 400ms (2-3 sec threshold for user retention)
- Scroll FPS: 60fps (drops acceptable in complex lists)
- Memory footprint: < 150MB baseline
- Compose recompositions: < 20% unnecessary

**Conservative (Budget Devices)**
- Cold start: < 800ms
- Scroll FPS: 30-60fps mixed
- Memory footprint: < 250MB baseline
- Compose recompositions: < 30% unnecessary

## Audit Checklist

### PA-001 to PA-015: Startup Time Optimization

#### Cold Start Measurements
- [ ] **PA-001**: Measuring cold-start time (onCreate + initial composition)
- [ ] **PA-002**: Not blocking main thread during app initialization
- [ ] **PA-003**: Lazy-loading non-critical features (analytics, ads, secondary APIs)
- [ ] **PA-004**: Deferring heavy computations to background thread
- [ ] **PA-005**: Using `rememberSaveable` to restore UI state quickly

#### Baseline Profiles (Android)
- [ ] **PA-006**: Baseline Profile module created (macrobenchmark)
- [ ] **PA-007**: Profile includes critical user journey (login → list view)
- [ ] **PA-008**: Profile targets top 3 features (30%+ startup improvement expected)
- [ ] **PA-009**: Baseline Profile deployed with app binary
- [ ] **PA-010**: Measuring AOT compilation benefit (must show >= 25% improvement)

#### iOS Startup Optimization
- [ ] **PA-011**: App delegate minimal - no heavy initialization
- [ ] **PA-012**: Scene delegate delays API calls until view appears
- [ ] **PA-013**: Using Instruments.app to measure cold-start phases
- [ ] **PA-014**: Lazy-loading SwiftUI views with `.task` modifier
- [ ] **PA-015**: Launch screen matches main UI theme (perceived speed)

### PA-016 to PA-035: Memory Management & Leak Detection

#### Memory Profiling
- [ ] **PA-016**: Using Android Profiler to capture baseline memory
- [ ] **PA-017**: Memory not growing after 10 navigate/back cycles
- [ ] **PA-018**: Large bitmaps loaded in `thumbnail` size, not full resolution
- [ ] **PA-019**: Using Coil/Glide for image caching (disk + memory)
- [ ] **PA-020**: Memory peaks during scroll don't exceed 50MB above baseline

#### Retain Cycles & Leaks
- [ ] **PA-021**: No lambda captures in `remember { }` that reference whole Activity
- [ ] **PA-022**: ViewModels don't hold View references
- [ ] **PA-023**: Listeners unregistered in `DisposableEffect` or `onCleared()`
- [ ] **PA-024**: Using `weak` references for listeners in Kotlin
- [ ] **PA-025**: iOS: No strong reference cycles between view controllers

#### Memory-Efficient Immutable Lists
- [ ] **PA-026**: Using `ImmutableList` or `persistentListOf()` for state
- [ ] **PA-027**: Data classes marked `@Immutable` or `@Stable`
- [ ] **PA-028**: Not recreating list objects in every recomposition
- [ ] **PA-029**: Paginated loading (not loading entire 10K items into memory)
- [ ] **PA-030**: Not storing large objects in LazyColumn items

#### Object Allocation Monitoring
- [ ] **PA-031**: Memory Profiler shows < 100MB allocations per frame
- [ ] **PA-032**: No continuous allocations during idle scrolling
- [ ] **PA-033**: String concatenation uses StringBuilder in loops
- [ ] **PA-034**: Object pools for frequently-created objects (e.g., animations)
- [ ] **PA-035**: iOS: Using value types (struct) instead of class for data models

### PA-036 to PA-060: Jetpack Compose Performance

#### Recomposition Control (Critical for Compose)
- [ ] **PA-036**: Stable data classes (`@Stable`, `@Immutable`)
- [ ] **PA-037**: Not passing MutableState to child composables
- [ ] **PA-038**: Using `key()` for LazyColumn items (stable IDs required)
- [ ] **PA-039**: Using `derivedStateOf` for calculated state (not in composition)
- [ ] **PA-040**: Heavy computation in `remember { }` or `LaunchedEffect`
- [ ] **PA-041**: Not calling `mutableStateOf()` inside loops or conditions

#### Modifier & Layout Optimization
- [ ] **PA-042**: Modifiers created outside composable body (except dynamic ones)
- [ ] **PA-043**: Using `Modifier.then()` instead of chaining in loops
- [ ] **PA-044**: Size/constraint modifiers placed first in modifier chain
- [ ] **PA-045**: Custom layouts avoid unnecessary measure/layout passes
- [ ] **PA-046**: Using `LazyColumn` for lists > 10 items (never `Column` with scroll)
- [ ] **PA-047**: LazyColumn key is unique and stable (not list index)

#### Compose Metrics & Debugging
- [ ] **PA-048**: Compose Compiler Metrics enabled (`compose.metricsDestination`)
- [ ] **PA-049**: Skippable composables marked (reduces recomposition)
- [ ] **PA-050**: Restartable composables analyzed (hot-reload performance)
- [ ] **PA-051**: debugInspectorInfo used for custom modifiers
- [ ] **PA-052**: Recomposition counts < 5 for stable parameters
- [ ] **PA-053**: No "deferred" composables blocking main thread

#### State & Side Effects
- [ ] **PA-054**: Side effects in `LaunchedEffect`, not composition body
- [ ] **PA-055**: API calls in `LaunchedEffect(Unit)` with key dependencies
- [ ] **PA-056**: Using `DisposableEffect` for cleanup (listeners, timers)
- [ ] **PA-057**: No Flow.collectAsState in nested composables
- [ ] **PA-058**: ViewModels use `StateFlow`, not `MutableState`
- [ ] **PA-059**: `.cache()` used on frequently-collected Flows
- [ ] **PA-060**: Pausable composition supported (Compose Dec 2025+)

### PA-061 to PA-080: SwiftUI Performance & Rendering

#### View Body Optimization
- [ ] **PA-061**: View body is lightweight (< 5ms to compute)
- [ ] **PA-062**: Heavy computation moved to ViewModel, not view body
- [ ] **PA-063**: Using `@State` for simple state, `@ObservedObject` for complex
- [ ] **PA-064**: No `.constant()` bindings that hide mutations
- [ ] **PA-065**: View identifiers stable (not changing on each render)

#### Instruments Profiling (WWDC 2025)
- [ ] **PA-066**: Xcode 26 Instruments SwiftUI tool used for baseline
- [ ] **PA-067**: Long View Body Updates identified and fixed
- [ ] **PA-068**: Unnecessary View Updates minimized (< 20%)
- [ ] **PA-069**: Core Animation tool shows < 16.7ms frame time (60fps)
- [ ] **PA-070**: Time Profiler shows no functions > 5ms in main thread

#### Conditional View Rendering
- [ ] **PA-071**: Using `@ViewBuilder` to defer expensive views
- [ ] **PA-072**: Optional views wrapped in `if let` inside body
- [ ] **PA-073**: List views have `.id()` or `.hashable()`
- [ ] **PA-074**: ScrollView contains LazyVStack for large lists
- [ ] **PA-075**: Gesture handlers don't trigger view recreations

#### Animation & Transition
- [ ] **PA-076**: Animations have explicit `.animation(_:value:)` modifiers
- [ ] **PA-077**: Transition times < 400ms (M3 standard)
- [ ] **PA-078**: No expensive redraws inside animation blocks
- [ ] **PA-079**: Using CADisplayLink for smooth custom animations
- [ ] **PA-080**: Respecting Motion accessibility setting (`.preferredMotionSetting`)

### PA-081 to PA-100: Scroll & List Performance

#### 60fps Scroll Guarantee
- [ ] **PA-081**: LazyColumn/LazyVStack renders frames in < 16.7ms
- [ ] **PA-082**: Item composables lightweight (no images in every item)
- [ ] **PA-083**: Image loading doesn't block scroll (async with Coil)
- [ ] **PA-084**: Scroll gesture responsiveness < 50ms
- [ ] **PA-085**: No janky momentum scroll (constant frame drops)

#### List Item Optimization
- [ ] **PA-086**: Items have stable `key()` and don't recreate
- [ ] **PA-087**: Item height fixed or predictable (helps scroller)
- [ ] **PA-088**: Images loaded with `contentScale = ContentScale.Crop`
- [ ] **PA-089**: Text strings truncated early (large bodies block rendering)
- [ ] **PA-090**: No heavy Shadows or blur effects per item

#### Prefetching & Pagination
- [ ] **PA-091**: Implementing pagination (load next 20 items on scroll)
- [ ] **PA-092**: Placeholder content shown during image load
- [ ] **PA-093**: Using `windowInsets` to keep list bounds clean
- [ ] **PA-094**: Scroll velocity tracked to load ahead-of-time
- [ ] **PA-095**: LazyColumn with `rememberLazyListState()` for scroll position

#### Complex Lists
- [ ] **PA-096**: Multi-column grids use `LazyVerticalGrid` (not nested LazyRows)
- [ ] **PA-097**: Grid items sized for exact fit (no jank on orientation change)
- [ ] **PA-098**: Sticky headers not remeasuring on every scroll
- [ ] **PA-099**: No horizontal scroll inside vertical list item
- [ ] **PA-100**: Fling scroll doesn't load 100+ items ahead

### PA-101 to PA-120: Image & Asset Loading

#### Image Optimization
- [ ] **PA-101**: Images compressed to device screen size (not 4K originals)
- [ ] **PA-102**: Using WebP format (30% smaller than PNG/JPG)
- [ ] **PA-103**: Thumbnails loaded first, detail images lazy
- [ ] **PA-104**: Image cache size capped (Android: 50MB default)
- [ ] **PA-105**: Disk cache enabled (Coil default: 2GB)

#### Network Image Loading
- [ ] **PA-106**: Using Coil (Android) or URLSession (iOS) with caching
- [ ] **PA-107**: Placeholder color shown while loading
- [ ] **PA-108**: Failed images show retry button (not silent failure)
- [ ] **PA-109**: Local cache checked before network request
- [ ] **PA-110**: Image decoding on background thread (not main)

#### Icon & Asset Size
- [ ] **PA-111**: Material Design icons 24dp (not 48dp for small uses)
- [ ] **PA-112**: SVG vectors used for scalable assets (vs PNG atlases)
- [ ] **PA-113**: No loading 200+ drawable resources on startup
- [ ] **PA-114**: VectorDrawable complexity < 500 nodes per icon
- [ ] **PA-115**: Using `rememberAsyncImagePainter` with lifecycle cleanup

#### Animation Assets
- [ ] **PA-116**: Lottie animations < 100KB per animation
- [ ] **PA-117**: Frame buffer size limited (not loading full sequence)
- [ ] **PA-118**: Playing animations only when visible (use lifecycle)
- [ ] **PA-119**: SVG-based animations preferred over Lottie
- [ ] **PA-120**: No auto-playing videos in backgrounds

### PA-121 to PA-140: Network Efficiency

#### API Request Optimization
- [ ] **PA-121**: Implementing request batching (not 10 API calls per screen)
- [ ] **PA-122**: Using HTTP caching headers (Cache-Control)
- [ ] **PA-123**: Request timeout set (5-10 seconds, not infinite)
- [ ] **PA-124**: Response compression enabled (gzip, brotli)
- [ ] **PA-125**: Retry logic with exponential backoff (3 attempts max)

#### Data Serialization
- [ ] **PA-126**: Using Kotlin Serialization or Moshi (not JSON reflection)
- [ ] **PA-127**: Serialization uses @Serializable (code-generated, not reflection)
- [ ] **PA-128**: Response parsing on background thread
- [ ] **PA-129**: Parsing large responses in chunks (not load all in memory)
- [ ] **PA-130**: Unused JSON fields skipped (@SerialName for mapping)

#### Connection Management
- [ ] **PA-131**: Using persistent HTTP connections (not creating new per request)
- [ ] **PA-132**: DNS caching enabled
- [ ] **PA-133**: Connection pooling configured (max 5-10 connections)
- [ ] **PA-134**: Network state monitored (wifi vs cellular)
- [ ] **PA-135**: Throttling API calls during poor signal

#### Request Deferral
- [ ] **PA-136**: Analytics logged asynchronously (not blocking)
- [ ] **PA-137**: Non-critical requests deferred until wifi
- [ ] **PA-138**: Background sync used for post-login requests
- [ ] **PA-139**: Request queue implemented for offline mode
- [ ] **PA-140**: Duplicate prevention (request deduping)

### PA-141 to PA-160: Battery Impact

#### CPU Usage
- [ ] **PA-141**: Not spinning tight loops on main thread
- [ ] **PA-142**: Using `Dispatchers.IO` for I/O, `Dispatchers.Default` for compute
- [ ] **PA-143**: Work scheduled with WorkManager (batched, not frequent)
- [ ] **PA-144**: Location updates requested at minimal frequency
- [ ] **PA-145**: Bluetooth scanning disabled when not active

#### Battery Profiler Metrics
- [ ] **PA-146**: Using Android Battery Historian for power analysis
- [ ] **PA-147**: App wake-locks < 1 per minute
- [ ] **PA-148**: No partial wake-locks held during normal use
- [ ] **PA-149**: GPS disabled when not in active use
- [ ] **PA-150**: Screen off operations minimal (sync, data)

#### Power-Aware Design
- [ ] **PA-151**: Throttling location updates (1 update per 30 sec, not 1 per sec)
- [ ] **PA-152**: Disabling animations when low-power mode active
- [ ] **PA-153**: Reducing frame rate in background (10fps vs 60fps)
- [ ] **PA-154**: Not continuously querying sensors
- [ ] **PA-155**: Push notifications preferred over polling

#### Thermal Management
- [ ] **PA-156**: App doesn't exceed 40°C during normal use
- [ ] **PA-157**: Video rendering at device capability (not forcing 4K)
- [ ] **PA-158**: No continuous heavy computation during active use
- [ ] **PA-159**: CPU frequency scaling not prevented
- [ ] **PA-160**: Thermal throttling gracefully handled

### PA-161 to PA-180: Rendering & UI Polish

#### Frame Budget
- [ ] **PA-161**: Frame rendering < 16.7ms (60fps target)
- [ ] **PA-162**: No frames > 34ms (jank threshold)
- [ ] **PA-163**: Janky frames < 1% of total frames
- [ ] **PA-164**: Preframe costs measured (draw calls, vertices)
- [ ] **PA-165**: Using `skipFrame()` sparingly (not on every frame)

#### GPU Optimization
- [ ] **PA-166**: Not using excessive transparency overlays
- [ ] **PA-167**: Avoiding blur/shadow effects per item in lists
- [ ] **PA-168**: RenderEffect used instead of custom shaders
- [ ] **PA-169**: Clipping bounds set for offscreen content
- [ ] **PA-170**: Canvas drawing uses hardware-accelerated paths

#### Layout Stability
- [ ] **PA-171**: Layout doesn't shift during image load (set explicit sizes)
- [ ] **PA-172**: Text doesn't reflow (line clamp or fixed width)
- [ ] **PA-173**: Using ConstraintLayout for complex layouts
- [ ] **PA-174**: Avoiding nested `Box` layouts (flatten hierarchy)
- [ ] **PA-175**: Measure/layout passes < 5 per frame

#### VSync & Rendering
- [ ] **PA-176**: Rendering synchronized to display refresh rate
- [ ] **PA-177**: No tearing artifacts (frame buffer double-buffering)
- [ ] **PA-178**: HDR rendering if display supports (not always enabled)
- [ ] **PA-179**: RenderThread not competing with main thread
- [ ] **PA-180**: Using `Choreographer` for frame-aligned animations

### PA-181 to PA-200: Monitoring & Profiling Setup

#### Profiling Tools Setup (Android)
- [ ] **PA-181**: Android Profiler enabled in build (debuggable = true for dev)
- [ ] **PA-182**: Layout Inspector configured for real-time debugging
- [ ] **PA-183**: Compose Compiler Metrics generated (`./gradlew composeMetrics`)
- [ ] **PA-184**: `debugInspectorInfo` added to custom composables
- [ ] **PA-185**: Frame metrics captured with `adb shell dumpsys gfxinfo`

#### Profiling Tools Setup (iOS)
- [ ] **PA-186**: Instruments Time Profiler target configured
- [ ] **PA-187**: Core Animation tool enabled for frame timing
- [ ] **PA-188**: Allocations instrument configured for memory leaks
- [ ] **PA-189**: SwiftUI Instrument (Xcode 26+) baseline captured
- [ ] **PA-190**: System Trace for multi-thread analysis

#### Performance Monitoring in Production
- [ ] **PA-191**: Firebase Performance Monitoring enabled
- [ ] **PA-192**: Custom performance traces for critical user journeys
- [ ] **PA-193**: Crash Reporting integrated (detect ANRs)
- [ ] **PA-194**: Cold-start time tracked in analytics
- [ ] **PA-195**: Scroll jank reported (dropped frame percentage)

#### Baseline & Regression Detection
- [ ] **PA-196**: Performance baseline established (startup, memory, FPS)
- [ ] **PA-197**: Performance budget defined (target metrics)
- [ ] **PA-198**: Regression tests in CI (macrobenchmark on every build)
- [ ] **PA-199**: Alert setup for > 10% regression
- [ ] **PA-200**: Historical performance trends tracked

## Code Examples

### Example 1: Android Startup with Baseline Profile

```kotlin
// GOOD: Minimal App.onCreate()
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        // Only critical initialization here
        initializeFirebase()  // Moved to lazy

        // Defer non-critical work
        Handler(Looper.getMainLooper()).postDelayed({
            initializeAnalytics()
        }, 1000)  // 1 second after app shows
    }
}

// Baseline Profile (macrobenchmark)
class StartupBenchmark : AbstractStartupBenchmark() {
    @get:Rule
    val benchmarkRule = BenchmarkRule()

    @Test
    fun startup() = benchmarkRule.measureRepeated {
        // Critical user journey: launch app → show list
        loginAndNavigateToList()
    }
}

// Build configuration
android {
    compileSdk 35

    dependenciesInfo {
        includeInApk = false
        includeInBundle = false
    }

    // Enable Baseline Profiles
    baselineProfile {
        enable = true
    }
}
```

### Example 2: Compose Recomposition Control

```kotlin
// BAD: Recomposes every time parent changes
@Composable
fun EventList(events: List<Event>) {
    LazyColumn {
        items(events) { event ->  // No key!
            EventCard(event = event)
        }
    }
}

// GOOD: Stable keys prevent recomposition
@Stable
data class Event(val id: String, val title: String)

@Composable
fun EventList(events: List<Event>) {
    LazyColumn {
        items(events, key = { it.id }) { event ->  // Stable key
            EventCard(event = event, modifier = Modifier.animateItem())
        }
    }
}

// GOOD: Derived state doesn't recompose children
@Composable
fun FilteredEventList(events: List<Event>, filter: String) {
    val filteredEvents = remember(events, filter) {
        events.filter { it.title.contains(filter, ignoreCase = true) }
    }
    EventList(events = filteredEvents)
}

// EXCELLENT: Using derivedStateOf for scroll position
@Composable
fun ScrollTracking() {
    val scrollState = rememberLazyListState()
    val isScrolled = remember {
        derivedStateOf { scrollState.firstVisibleItemIndex > 0 }
    }

    LazyColumn(state = scrollState) {
        // Header shows/hides based on scroll
        if (isScrolled.value) {
            item { Header() }
        }
        items(100) { EventCard() }
    }
}
```

### Example 3: Image Loading with Caching

```kotlin
// GOOD: Coil with memory + disk caching
@Composable
fun EventImage(imageUrl: String, modifier: Modifier = Modifier) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(imageUrl)
            .crossfade(true)
            .diskCachePolicy(CachePolicy.ENABLED)
            .memoryCachePolicy(CachePolicy.ENABLED)
            .build(),
        contentDescription = "Event image",
        contentScale = ContentScale.Crop,
        modifier = modifier
            .fillMaxWidth()
            .height(200.dp)
            .clip(RoundedCornerShape(8.dp))
    )
}

// Setup in Application
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()

        // Configure Coil caching
        val imageLoader = ImageLoader.Builder(this)
            .diskCache {
                DiskCache.Builder()
                    .directory(File(cacheDir, "image_cache"))
                    .maxSizePercent(0.02)  // 2% of app cache
                    .build()
            }
            .memoryCache {
                MemoryCache.Builder(this)
                    .maxSizePercent(0.25)  // 25% of available memory
                    .build()
            }
            .build()
        Coil.setImageLoader(imageLoader)
    }
}
```

### Example 4: Network Request with Caching Headers

```kotlin
// GOOD: Retrofit with HTTP caching
@GET("api/v1/events")
suspend fun getEvents(
    @Query("limit") limit: Int = 20,
    @Header("Cache-Control") cacheControl: String = "public, max-age=600"  // 10 min cache
): EventResponse

// Setup interceptor
val httpClient = OkHttpClient.Builder()
    .cache(Cache(File(context.cacheDir, "http_cache"), 10 * 1024 * 1024))  // 10MB
    .addNetworkInterceptor { chain ->
        val response = chain.proceed(chain.request())
        response.newBuilder()
            .header("Cache-Control", "public, max-age=600")
            .build()
    }
    .connectTimeout(10, TimeUnit.SECONDS)
    .readTimeout(10, TimeUnit.SECONDS)
    .writeTimeout(10, TimeUnit.SECONDS)
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("http://api.example.com/")
    .addConverterFactory(Json.asConverterFactory("application/json".toMediaType()))
    .client(httpClient)
    .build()
```

### Example 5: SwiftUI Performance with Instruments

```swift
// GOOD: Lightweight view body
struct EventCard: View {
    let event: Event
    @State var isExpanded = false

    var body: some View {
        // Simple, computed quickly
        VStack(alignment: .leading, spacing: 8) {
            Text(event.title)
                .font(.headline)
            Text(event.location)
                .font(.caption)
                .foregroundColor(.secondary)

            if isExpanded {
                DetailView(event: event)  // Only expanded when needed
            }
        }
        .contentShape(Rectangle())
        .onTapGesture {
            withAnimation {
                isExpanded.toggle()
            }
        }
    }
}

// GOOD: Using Instruments
// Product > Profile (Cmd+I) in Xcode
// Select "Core Animation" to measure FPS
// Check "Color Misaligned Images" and "Color Offscreen Rendered"

// Checking for long view body:
// Use SwiftUI instrument (Xcode 26+) to see view update duration
struct EventList: View {
    @StateObject var viewModel = EventViewModel()

    var body: some View {
        // This entire body runs on every state change
        List(viewModel.events) { event in
            EventCard(event: event)
        }
        .task {
            // Non-blocking: don't load here
            await viewModel.loadEvents()
        }
    }
}
```

## Profiling Workflow

### Android: Using Compose Metrics

```bash
# Generate Compose Compiler Metrics
./gradlew composeMetrics

# Find results
cat build/compose_metrics/app-release-composables.txt
cat build/compose_metrics/app-release-recompositions.txt

# Look for:
# - Unstable parameters (recompose even if unchanged)
# - Skippable composables (fast recompose)
# - Restartable composables (can pause)
```

### Android: Layout Inspector

```bash
# In Android Studio:
# 1. Run app on device/emulator
# 2. Tools > Layout Inspector
# 3. Select process
# 4. Inspect view hierarchy
# 5. Check "Render bounds" to see layout passes
```

### Android: Frame Metrics

```bash
adb shell dumpsys gfxinfo com.partyonapp.android.debug > /tmp/gfx.txt
# Look for:
# - VSYNC_TIMESTAMP: frame timing
# - Frame_Start_Time and Frame_Complete_Time
# - Frames with > 16.7ms duration (jank)
```

### iOS: Instruments

```bash
# In Xcode:
# 1. Product > Profile (Cmd+I)
# 2. Select "Core Animation" or "Time Profiler"
# 3. Start recording
# 4. Interact with app
# 5. Stop and analyze
#
# Core Animation shows:
# - FPS (target 60)
# - Color Misaligned Images (render off-screen)
# - Color Offscreen Rendered (GPU inefficiency)
```

### iOS: SwiftUI Instrument (Xcode 26+)

```swift
// Enable debug printing
struct EventList: View {
    var body: some View {
        List {
            ForEach(events) { event in
                EventCard(event: event)
            }
        }
        ._printChanges()  // Print recompositions to console
    }
}

// Use in Instruments:
// Product > Profile > SwiftUI
// Captures view update duration, identifies long body calculations
```

## Performance Budgets & Metrics

| Metric | Aggressive | Balanced | Conservative |
|--------|-----------|----------|--------------|
| Cold Start | < 300ms | < 400ms | < 800ms |
| Warm Start | < 100ms | < 150ms | < 300ms |
| Scroll FPS | 60 solid | 60 (avg) | 30-60 |
| Memory Baseline | < 80MB | < 150MB | < 250MB |
| First Frame | < 500ms | < 800ms | < 1.5s |
| Image Load | < 200ms | < 500ms | < 1s |
| API Response | < 500ms | < 1s | < 2s |
| Battery/hour | < 5% | < 10% | < 20% |

## Output Format

Generate a structured performance audit report:

```markdown
## Performance Audit: [Component/App Name]

### Executive Summary
- **Target Level**: Balanced (mid-range devices)
- **Platform**: Android / iOS / Both
- **Focus Area**: Startup time, memory, scrolling, rendering, network, battery, all
- **Overall Score**: X% (0-100, based on target metrics)

### Critical Issues (🔴 - Blocks Performance)

**PA-XXX**: Issue description
- **Evidence**: Measured data or code pattern
- **Impact**: Specific metric affected (startup +200ms, memory +50MB)
- **Fix**: Code change or configuration update
- **Effort**: Low/Medium/High

### Important Issues (🟠 - Degraded Performance)

**PA-XXX**: Issue description
- **Evidence**: Details
- **Impact**: Metric impact
- **Fix**: Recommendation
- **Effort**: Time estimate

### Warnings (🟡 - Suboptimal Patterns)

**PA-XXX**: Issue description
- **Details**: Context
- **Recommendation**: How to improve
- **Effort**: Minimal/Small/Medium

### Suggestions (🔵 - Polish Opportunities)

**PA-XXX**: Issue description
- **Details**: Why this matters
- **Recommendation**: Optional improvement
- **Effort**: Small/Polish

### Platform-Specific Findings

#### Android
- [ ] Compose recomposition analysis
- [ ] Memory pressure points
- [ ] Baseline Profile recommendations

#### iOS
- [ ] SwiftUI body computation times
- [ ] Instruments profiling results
- [ ] Core Animation frame drops

### Performance Metrics Summary

| Metric | Current | Target | Status |
|--------|---------|--------|--------|
| Cold Start | Xms | <400ms | 🟡 |
| Scroll FPS | 45avg | 60 | 🟠 |
| Memory | 180MB | 150MB | 🟠 |
| Recompositions | 35% | 20% | 🔴 |

### Profiling Data

**Traces Captured:**
- Startup flow: launch → first interactive frame
- Scroll: 100 item list, 3 sec of continuous scrolling
- Memory: baseline + peak during normal use
- Network: API response timing with real data

**Tools Used:**
- Android: Profiler, Layout Inspector, Compose Metrics
- iOS: Instruments (Core Animation, Time Profiler, SwiftUI)

### Recommended Priority Actions

1. **URGENT** [Issue with highest impact]
   - Action: Specific code change
   - Expected improvement: X% faster / Y MB less
   - Time estimate: Z hours

2. **IMPORTANT** [Second priority]
   - Action: Details
   - Expected improvement: Details
   - Time estimate: Hours

3. **NICE-TO-HAVE** [Polish]
   - Action: Details
   - Expected improvement: Details
   - Time estimate: Hours

### Implementation Checklist

- [ ] Run baseline profiling before changes
- [ ] Implement fixes in priority order
- [ ] Re-profile after each fix
- [ ] Set up continuous monitoring (Firebase)
- [ ] Document performance baseline
- [ ] Create regression tests (macrobenchmark)

### Next Steps

1. [First action]
2. [Second action]
3. [Third action]
```

## References & Tools

### Android Performance
- [Jetpack Compose Performance Documentation](https://developer.android.com/develop/ui/compose/performance)
- [Compose Best Practices](https://developer.android.com/develop/ui/compose/performance/bestpractices)
- [Baseline Profiles Overview](https://developer.android.com/topic/performance/baselineprofiles/overview)
- [Create Baseline Profiles](https://developer.android.com/topic/performance/baselineprofiles/create-baselineprofile)
- [Android Profiler Guide](https://developer.android.com/studio/profile/android-profiler)
- [Layout Inspector](https://developer.android.com/studio/debug/layout-inspector)

### iOS Performance
- [Optimize SwiftUI performance with Instruments (WWDC 2025)](https://developer.apple.com/videos/play/wwdc2025/306/)
- [Understanding and improving SwiftUI performance](https://developer.apple.com/documentation/Xcode/understanding-and-improving-swiftui-performance)
- [Core Animation Performance](https://developer.apple.com/library/archive/documentation/3DDrawing/Conceptual/CoreAnimationGuide/Introduction/Introduction.html)
- [Instruments User Guide](https://help.apple.com/instruments/mac/current/)

### Network & Image Loading
- [Coil Image Loader (Android)](https://coil-kt.github.io/coil/)
- [Retrofit HTTP Client](https://square.github.io/retrofit/)
- [OkHttp Caching](https://square.github.io/okhttp/)

### Monitoring & Analytics
- [Firebase Performance Monitoring](https://firebase.google.com/docs/perf-mod)
- [Firebase Crashlytics](https://firebase.google.com/docs/crashlytics)

---

**Note:** All performance targets are based on 2025 industry standards. Adjust targets based on your specific app requirements and device tiers.

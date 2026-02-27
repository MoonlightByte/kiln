<!-- SPDX-License-Identifier: Apache-2.0 -->
<!-- Patterns extracted from Now in Android, Copyright Google LLC, licensed under Apache-2.0 -->

# Now in Android: Reference Architecture Patterns

> **License Notice:** This document analyzes architectural patterns from Google's
> [Now in Android](https://github.com/android/nowinandroid) reference implementation,
> licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).
> These patterns are presented for educational purposes.

Extracted best practices from Google's official reference implementation: https://github.com/android/nowinandroid

---

## Overview

**Now in Android (NiA)** is Google's flagship Jetpack Compose reference app demonstrating production-grade architectural patterns, Material Design 3 implementation, and Compose stability optimization.

**Key Achievement**: Baseline Profiles (AOT compilation) + compiler metrics-driven optimization = smooth launch and scrolling on all devices.

---

## Material 3 & Dynamic Color Implementation

### Design System Architecture

```kotlin
// core/designsystem/src/main/kotlin/com/google/samples/apps/nia/core/designsystem/theme/Color.kt

val LightColorScheme = lightColorScheme(
    primary = NiaGreen80,
    onPrimary = NiaGreen20,
    primaryContainer = NiaGreen30,
    onPrimaryContainer = NiaGreen90,
    secondary = NiaDarkerDitsyBlue80,
    onSecondary = NiaDarkerDitsyBlue20,
    secondaryContainer = NiaDarkerDitsyBlue30,
    onSecondaryContainer = NiaDarkerDitsyBlue90,
    tertiary = NiaBlue80,
    onTertiary = NiaBlue20,
    tertiaryContainer = NiaBlue30,
    onTertiaryContainer = NiaBlue90,
    // ... remaining colors
)

val DarkColorScheme = darkColorScheme(
    primary = NiaGreen20,
    onPrimary = NiaGreen80,
    // ... mirror of light scheme
)
```

### Dynamic Color Support (Android 12+)

```kotlin
// core/designsystem/src/main/kotlin/com/google/samples/apps/nia/core/designsystem/theme/Theme.kt

@Composable
fun NiaTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,  // Enable dynamic theming
    content: @Composable () -> Unit,
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = NiaTypography,
        shapes = NiaShapes,
        content = content,
    )
}
```

**Key Pattern**: Fallback to predefined scheme when dynamic color unavailable, ensures consistent experience across Android versions.

---

## Typography System

### Semantic Tokens Approach

```kotlin
// core/designsystem/src/main/kotlin/com/google/samples/apps/nia/core/designsystem/theme/Type.kt

val NiaTypography = Typography(
    displayLarge = TextStyle(
        fontFamily = FiraSans,
        fontSize = 57.sp,
        fontWeight = FontWeight.Normal,
        lineHeight = 64.sp,
        letterSpacing = 0.sp,
    ),
    displayMedium = TextStyle(
        fontFamily = FiraSans,
        fontSize = 45.sp,
        fontWeight = FontWeight.Normal,
        lineHeight = 52.sp,
        letterSpacing = 0.sp,
    ),
    displaySmall = TextStyle(
        fontFamily = FiraSans,
        fontSize = 36.sp,
        fontWeight = FontWeight.Normal,
        lineHeight = 44.sp,
        letterSpacing = 0.sp,
    ),
    // ... all 15 M3 typography levels
)
```

**Best Practice**: Define typography at theme level, use `MaterialTheme.typography.*` throughout app instead of hardcoded sizes.

---

## Modularization Strategy

### Feature Module Pattern

```
now-in-android/
├── app/                          # Entry point
├── core/
│   ├── designsystem/            # Theme, colors, typography, shapes
│   ├── ui/                       # Shared UI components
│   ├── data/                     # Repository pattern
│   ├── datastore/                # Encrypted preferences
│   ├── network/                  # Retrofit + OkHttp
│   └── common/                   # Utilities
├── feature/
│   ├── for-you/                  # Feature module (isolated)
│   │   ├── src/main/kotlin/...
│   │   └── build.gradle.kts
│   ├── interests/
│   ├── bookmarks/
│   └── search/
└── ...
```

**Advantages**:
- Each feature can be built, tested, and deployed independently
- Clear ownership boundaries (team can own one feature module)
- Faster builds (only rebuild changed modules)
- Reusable core components across features

### Module Dependencies (Uni-directional)

```
┌─────────────────────┐
│  :feature:for-you   │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│   :core:ui          │
└──────────┬──────────┘
           │
┌──────────▼──────────┐
│  :core:designsystem │
└─────────────────────┘
```

**Rule**: Features depend on core modules. Core modules NEVER depend on features. This prevents circular dependencies.

---

## Stability & Performance Optimization

### Baseline Profiles (AOT Compilation)

```gradle
// app/build.gradle.kts

android {
    // Enable baseline profile for optimized compilation
    baselineProfile {
        baselineProfileOutputDir = "src"
    }
}

dependencies {
    // Baseline profile library
    implementation("androidx.profileinstaller:profileinstaller:1.3.1")
}
```

**Baseline Profile File** (`app/src/main/baseline-prof.txt`):

```
# Precompile critical app launch paths
com/google/samples/apps/nia/ui/interests/InterestsScreenKt
com/google/samples/apps/nia/ui/foryou/ForYouScreenKt
com/google/samples/apps/nia/ui/search/SearchScreenKt

# Mark classes for JIT profiling
com.google.samples.apps.nia.core.ui.**.* -> com.google.samples.apps.nia.feature.**.* _INCLUDED_
```

**Impact**:
- 60-80% faster app launch time
- Smoother initial scrolling
- Precompiled hot paths before user interaction

### Compiler Metrics Configuration

```gradle
// build.gradle.kts (project level)

tasks.register("inspectComposerMetrics") {
    doLast {
        val composeMetrics = file("build/compose_metrics")
        val outputFile = file("build/reports/compose-metrics-summary.txt")

        outputFile.printWriter().use { writer ->
            composeMetrics.walk().forEach { file ->
                if (file.name.endsWith("_composables.csv")) {
                    writer.println("=== ${file.parentFile.name} ===")
                    file.forEachLine { line ->
                        writer.println(line)
                    }
                }
            }
        }
    }
}
```

**Usage**:
```bash
# Generate metrics
./gradlew assembleRelease -Pandroidx.enableComposeCompilerMetrics=true

# View summary
cat build/reports/compose-metrics-summary.txt
```

---

## Edge-to-Edge & Insets

### Window Insets Handling (Android 15+)

```kotlin
// core/ui/src/main/kotlin/com/google/samples/apps/nia/core/ui/WindowInsets.kt

@Composable
fun EdgeToEdgeScreen(content: @Composable () -> Unit) {
    val view = LocalView.current

    // Enable edge-to-edge rendering
    SideEffect {
        val window = (view.context as? Activity)?.window ?: return@SideEffect
        WindowCompat.setDecorFitsSystemWindows(window, false)

        // Set up insets
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            window.statusBarColor = Color.TRANSPARENT
            window.navigationBarColor = Color.TRANSPARENT
        }
    }

    Box(
        modifier = Modifier
            .fillMaxSize()
            .systemBarsPadding()  // Respects status/nav bar areas
    ) {
        content()
    }
}
```

**Key Functions**:
- `systemBarsPadding()`: Applies insets for status + nav bars
- `statusBarsPadding()`: Status bar only
- `navigationBarsPadding()`: Navigation bar only
- `displayCutoutPadding()`: Display notch/punch-hole

---

## Predictive Back Navigation

### Gesture Navigation Integration

```kotlin
// core/ui/src/main/kotlin/com/google/samples/apps/nia/core/ui/PredictiveBack.kt

@Composable
fun BackPressHandler(
    enabled: Boolean = true,
    onBackPress: () -> Unit,
) {
    val backCallback = remember {
        object : OnBackPressedCallback(enabled) {
            override fun handleOnBackPressed() {
                onBackPress()
            }
        }
    }

    val backDispatcher = LocalOnBackPressedDispatcherOwner.current
        ?.onBackPressedDispatcher

    LaunchedEffect(enabled) {
        backDispatcher?.addCallback(backCallback)
    }

    DisposableEffect(Unit) {
        onDispose {
            backCallback.remove()
        }
    }
}
```

**Usage**:
```kotlin
@Composable
fun DetailScreen(onBack: () -> Unit) {
    BackPressHandler(onBackPress = onBack)

    Column {
        TopAppBar(
            navigationIcon = {
                IconButton(onClick = onBack) {
                    Icon(Icons.AutoMirrored.Filled.ArrowBack, "Back")
                }
            },
        )
        // Content
    }
}
```

---

## Data Flow & MVVM

### ViewModel + StateFlow Pattern

```kotlin
// feature/for-you/src/main/kotlin/com/google/samples/apps/nia/feature/foryou/ForYouViewModel.kt

@HiltViewModel
class ForYouViewModel @Inject constructor(
    private val newsRepository: NewsRepository,
    savedStateHandle: SavedStateHandle,
) : ViewModel() {

    // State exposed as StateFlow
    private val _feedState = MutableStateFlow<FeedUIState>(FeedUIState.Loading)
    val feedState: StateFlow<FeedUIState> = _feedState.asStateFlow()

    private val _selectedTopics = MutableStateFlow<Set<String>>(emptySet())
    val selectedTopics: StateFlow<Set<String>> = _selectedTopics.asStateFlow()

    init {
        viewModelScope.launch {
            combine(
                newsRepository.getNews(),
                _selectedTopics,
            ) { news, topics ->
                FeedUIState.Success(
                    news = news.filter { topics.contains(it.topic) }
                )
            }.catch { error ->
                emit(FeedUIState.Error(error.message ?: "Unknown error"))
            }.collect { uiState ->
                _feedState.value = uiState
            }
        }
    }

    fun onSelectTopic(topic: String) {
        _selectedTopics.update { it + topic }
    }
}
```

### Composable Integration

```kotlin
@Composable
fun ForYouScreen(
    viewModel: ForYouViewModel = hiltViewModel(),
) {
    val feedState by viewModel.feedState.collectAsStateWithLifecycle()
    val selectedTopics by viewModel.selectedTopics.collectAsStateWithLifecycle()

    when (feedState) {
        FeedUIState.Loading -> LoadingIndicator()
        is FeedUIState.Success -> {
            FeedContent(
                news = (feedState as FeedUIState.Success).news,
                onSelectTopic = viewModel::onSelectTopic,
                selectedTopics = selectedTopics,
            )
        }
        is FeedUIState.Error -> ErrorMessage((feedState as FeedUIState.Error).message)
    }
}
```

**Best Practices**:
- Expose immutable `StateFlow<T>` (not mutable `MutableStateFlow<T>`)
- Use `combine()` for complex state transformations
- Leverage `collectAsStateWithLifecycle()` for lifecycle-aware collection

---

## Composable Component Pattern

### Reusable Component Example

```kotlin
// core/ui/src/main/kotlin/com/google/samples/apps/nia/core/ui/NewsItem.kt

@Composable
fun NewsItemCard(
    title: String,
    description: String,
    resourceUrl: String,
    modifier: Modifier = Modifier,
    onClick: () -> Unit = {},
) {
    ElevatedCard(
        modifier = modifier
            .fillMaxWidth()
            .clickable { onClick() },
        shape = RoundedCornerShape(8.dp),
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            // Always load images asynchronously
            AsyncImage(
                model = resourceUrl,
                contentDescription = title,
                modifier = Modifier
                    .fillMaxWidth()
                    .height(200.dp),
                contentScale = ContentScale.Crop,
            )

            Spacer(modifier = Modifier.height(8.dp))

            Text(
                text = title,
                style = MaterialTheme.typography.headlineSmall,
            )

            Spacer(modifier = Modifier.height(4.dp))

            Text(
                text = description,
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant,
            )
        }
    }
}

// Preview for development
@Preview
@Composable
fun NewsItemCardPreview() {
    NiaTheme {
        NewsItemCard(
            title = "Breaking News",
            description = "This is a test news item",
            resourceUrl = "https://example.com/image.jpg",
        )
    }
}
```

**Design Principles**:
- `Modifier` as first parameter (allows composition)
- Sensible defaults (empty lambdas, `Modifier.Companion`)
- Fully self-contained (doesn't expect external state)
- Preview provided for IDE/manual testing

---

## Testing Patterns

### Screen-Level Testing

```kotlin
// feature/for-you/src/androidTest/kotlin/ForYouScreenTest.kt

@HiltAndroidTest
class ForYouScreenTest {

    @get:Rule
    val hiltRule = HiltAndroidRule(this)

    @get:Rule
    val composeTestRule = createAndroidComposeRule<MainActivity>()

    @Before
    fun setup() {
        hiltRule.inject()
    }

    @Test
    fun forYouScreen_showsLoadingThenContent() {
        composeTestRule.setContent {
            ForYouScreen()
        }

        // Verify loading state
        composeTestRule.onNodeWithContentDescription("Loading").assertExists()

        // Wait for content
        composeTestRule.waitUntil(5000) {
            composeTestRule.onAllNodesWithTag("news_item").fetchSemanticsNodes().size > 0
        }

        // Verify news items appear
        composeTestRule.onNodeWithTag("news_item").assertExists()
    }

    @Test
    fun forYouScreen_clickingTopicFiltersNews() {
        val testTopic = "Technology"

        composeTestRule.setContent {
            ForYouScreen()
        }

        composeTestRule.onNodeWithText(testTopic).performClick()
        composeTestRule.onAllNodesWithTag("news_item").assertCountEquals(5)
    }
}
```

---

## Macro Benchmark Testing

### Startup Performance Baseline

```kotlin
// benchmark/src/main/kotlin/com/google/samples/apps/nia/benchmark/StartupBenchmark.kt

@RunWith(AndroidBenchmarkRunner::class)
@BenchmarkRule.IncludeLoops(500)
class StartupBenchmark {
    @get:Rule
    val benchmarkRule = MacrobenchmarkRule()

    @Test
    fun startup() = benchmarkRule.measureRepeated(
        packageName = "com.google.samples.apps.nia",
        metrics = listOf(StartupTimingMetric()),
        iterations = 10,
        startupMode = StartupMode.COLD,
    ) {
        pressHome()
        startActivityAndWait()
    }

    @Test
    fun scrollNewsFeed() = benchmarkRule.measureRepeated(
        packageName = "com.google.samples.apps.nia",
        metrics = listOf(FrameTimingMetric()),
        iterations = 10,
    ) {
        startActivityAndWait()

        // Scroll to bottom
        device.findObject(By.res("feed_list")).fling(Direction.DOWN)
        device.waitForIdle()
    }
}
```

---

## Code Generation & Build Time

### BuildConfig for Feature Flags

```gradle
// app/build.gradle.kts

android {
    buildTypes {
        debug {
            buildConfigField("Boolean", "ENABLE_COMPOSE_METRICS", "true")
            buildConfigField("Boolean", "ENABLE_BASELINE_PROFILE", "false")
        }
        release {
            buildConfigField("Boolean", "ENABLE_COMPOSE_METRICS", "false")
            buildConfigField("Boolean", "ENABLE_BASELINE_PROFILE", "true")
        }
    }
}
```

### Build Time Optimization

```gradle
// gradle/libs.versions.toml

[versions]
kotlinVersion = "1.9.10"
composeVersion = "2023.10.01"

[plugins]
ksp = { id = "com.google.devtools.ksp", version = "1.9.10-1.0.13" }
hilt = { id = "com.google.dagger.hilt.android", version = "2.48" }
```

---

## Summary Table: NiA Best Practices

| Category | Pattern | Benefit |
|----------|---------|---------|
| **Theme** | Dynamic color + fallback | Works on all Android versions |
| **Typography** | Semantic tokens at theme level | Consistent, centralized styling |
| **Module** | Feature modules (uni-directional deps) | Faster builds, clear ownership |
| **Performance** | Baseline profiles + metrics | 60-80% faster launch |
| **Insets** | `systemBarsPadding()` edge-to-edge | Modern look, full-screen UX |
| **Navigation** | Predictive back with callbacks | Smooth gesture navigation |
| **State** | `StateFlow` + `combine()` | Reactive, testable data flow |
| **Components** | Composable with `Modifier` first | Reusable, composable UI |
| **Testing** | Macro benchmarks + E2E | Data-driven performance |

---

## References

- **Repository**: https://github.com/android/nowinandroid
- **Material 3 Docs**: https://m3.material.io/
- **Compose Docs**: https://developer.android.com/develop/ui/compose
- **Modularization Guide**: https://developer.android.com/guide/modularization
- **Baseline Profiles**: https://developer.android.com/guide/baseline-profiles

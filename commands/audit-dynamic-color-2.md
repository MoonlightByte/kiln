---
name: audit-dynamic-color-2
description: Comprehensive Material 3 Dynamic Color and color harmonization audit for Android 12+ personalization
args:
  - name: code
    type: string
    description: Kotlin/Compose code to audit for dynamic color and harmonization
    required: true
  - name: focus
    type: string
    enum: [api_setup, color_roles, harmonization, fallbacks, testing, all]
    description: Specific area to focus on (default: all)
    required: false
  - name: min_api
    type: number
    enum: [31, 32, 33]
    description: Minimum API level (31=Android 12, 32=Android 12L, 33=Android 13; default: 31)
    required: false
    default: 31
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Material 3 Dynamic Color & Harmonization Audit (2025)

Comprehensive audit of Material 3 Dynamic Color implementation, color scheme extraction, brand color harmonization with MaterialColors.harmonize(), tonal palette generation, and fallback patterns for Android 12+ personalization features.

## Instructions

Analyze the code for Material 3 Dynamic Color compliance. Reference the latest Material 3 specifications and Android 12+ dynamic color APIs:

- **DynamicColors.applyToActivitiesIfAvailable()** - System-wide dynamic color enablement
- **dynamicLightColorScheme() / dynamicDarkColorScheme()** - Compose color scheme generation
- **MaterialColors.harmonize()** - Brand color harmonization with primary palette
- **HarmonizedColors / HarmonizedColorAttributes** - Custom color harmonization
- **ColorScheme color roles** - Primary, secondary, tertiary, error, surface
- **Tonal elevation & surfaces** - Surface1-5 for elevation-based coloring
- **Platform detection & fallbacks** - Pre-Android 12 graceful degradation

## Material 3 Dynamic Color Fundamentals (2025)

### What is Dynamic Color?

**Definition**: Dynamic Color adapts app color schemes from the user's device wallpaper (Android 12+), enabling system-wide color personalization without requiring app updates.

**Key Benefits**:
- System-wide personalization (Material You)
- Wallpaper-based color extraction
- Automatic light/dark theme variants
- User preference respect
- Visual cohesion across apps

**History**:
- Android 12 (API 31): Initial dynamic color support with Tonal Spot
- Android 12L (API 32): Expanded dynamic color availability
- Android 13 (API 33): Additional palette styles (Neutral, Vibrant, Expressive)

### Core APIs (Android 12+)

| API | Purpose | Min API | Notes |
|-----|---------|---------|-------|
| `DynamicColors` | Apply dynamic colors system-wide | 31 | Must call in Application.onCreate() |
| `dynamicLightColorScheme()` | Generate light scheme from wallpaper | 31 | Compose M3 theming |
| `dynamicDarkColorScheme()` | Generate dark scheme from wallpaper | 31 | Compose M3 theming |
| `MaterialColors.harmonize()` | Harmonize brand color with primary | 31 | For custom color preservation |
| `HarmonizedColors` | Container for harmonized color set | 31 | Advanced custom colors |
| `HarmonizedColorAttributes` | Define custom color harmonization rules | 31 | Per-color harmonization config |

## Audit Checklist

### DC-001 to DC-035+: Complete Dynamic Color Coverage

#### API Setup & Initialization (Critical Priority)
{{#if (or (eq focus "api_setup") (eq focus "all") (not focus))}}

**DC-001**: DynamicColors enabled in Application.onCreate()
- [ ] `DynamicColors.applyToActivitiesIfAvailable(this)` called
- [ ] Called in Application or MainApplication class
- [ ] No conditional logic preventing execution
- [ ] Executes before any activities created

**DC-002**: Application class extends Application
- [ ] `public class MyApplication extends Application { }`
- [ ] Registered in AndroidManifest.xml: `<application android:name="...MyApplication">`
- [ ] No errors in initialization chain

**DC-003**: AndroidManifest.xml references Application class
- [ ] `<application android:name="com.example.MyApplication">`
- [ ] Class path is correct (matches source tree)
- [ ] No typos or missing imports

**DC-004**: API level checks for pre-Android 12 devices
- [ ] `if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S)` for Android 12 checks
- [ ] Fallback theme applied for SDK < 31
- [ ] No crashes on Android 11 or lower

**DC-005**: Dynamic color disabled for testing/forced themes
- [ ] Dark mode not forced with `setTheme(R.style.Theme_Light)`
- [ ] No MODE_NIGHT_NO or MODE_NIGHT_YES forcing
- [ ] Using MODE_NIGHT_FOLLOW_SYSTEM or MODE_NIGHT_AUTO

{{/if}}

#### Color Scheme Generation (Critical Priority)
{{#if (or (eq focus "color_roles") (eq focus "all") (not focus))}}

**DC-006**: dynamicLightColorScheme() used for light theme
- [ ] `if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) { dynamicLightColorScheme(context) } else { lightColorScheme() }`
- [ ] Fallback lightColorScheme() defined for API < 31
- [ ] Context passed correctly (non-null)

**DC-007**: dynamicDarkColorScheme() used for dark theme
- [ ] `if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) { dynamicDarkColorScheme(context) } else { darkColorScheme() }`
- [ ] Fallback darkColorScheme() defined for API < 31
- [ ] Context passed correctly

**DC-008**: ColorScheme contains all required color roles
- [ ] `primary` - brand color (usually harmonized)
- [ ] `onPrimary` - contrast text/icons on primary
- [ ] `secondary` - supporting color
- [ ] `tertiary` - accent color
- [ ] `error` - error/validation states
- [ ] `surface` - main background
- [ ] `onSurface` - text on background

**DC-009**: Tonal elevation surfaces (Surface1-5) defined
- [ ] `surface` - base (0dp elevation)
- [ ] `surfaceContainer` or `surface1` - 1dp tones
- [ ] `surface2` - higher elevation tones (if using Material 3.1+)
- [ ] `surface3`, `surface4`, `surface5` - deepest elevations
- [ ] Each gets progressively lighter tint from primary

**DC-010**: Color scheme applied to MaterialTheme
- [ ] `MaterialTheme(colorScheme = colorScheme, ...)`
- [ ] Color scheme computed at app startup
- [ ] No hardcoded colors replacing dynamic scheme

{{/if}}

#### Color Role Usage (Important Priority)
{{#if (or (eq focus "color_roles") (eq focus "all") (not focus))}}

**DC-011**: Primary color used for main UI elements
- [ ] Buttons, active tabs, highlights use `MaterialTheme.colorScheme.primary`
- [ ] No hardcoded color replacing primary
- [ ] Primary respects dynamic color on Android 12+

**DC-012**: OnPrimary for text/icons on primary backgrounds
- [ ] `Text(color = MaterialTheme.colorScheme.onPrimary)` on primary buttons
- [ ] Icons on primary use `tint = MaterialTheme.colorScheme.onPrimary`
- [ ] Sufficient contrast (4.5:1+ WCAG AA)

**DC-013**: Secondary and OnSecondary used correctly
- [ ] Secondary chips, filters use `MaterialTheme.colorScheme.secondary`
- [ ] Text on secondary uses `MaterialTheme.colorScheme.onSecondary`
- [ ] Distinct from primary (not duplicated)

**DC-014**: Tertiary for accent/highlight purposes
- [ ] Accent elements use `MaterialTheme.colorScheme.tertiary`
- [ ] Examples: favorite icons, special badges, highlights
- [ ] Harmonized with primary for cohesion

**DC-015**: Error color for validation/warnings
- [ ] Error states use `MaterialTheme.colorScheme.error`
- [ ] No hardcoded red (#FF0000)
- [ ] Error text uses `onError` for contrast

**DC-016**: Surface colors for containers/backgrounds
- [ ] Default surface: `MaterialTheme.colorScheme.surface`
- [ ] Elevated surfaces: `MaterialTheme.colorScheme.surface1` through `surface5`
- [ ] Surface tones increase with elevation
- [ ] No shadows - use tonal elevation instead

**DC-017**: OnSurface/OnSurfaceVariant for text
- [ ] Primary text: `onSurface`
- [ ] Secondary text: `onSurfaceVariant`
- [ ] Sufficient contrast for both (4.5:1+ for body text)

{{/if}}

#### Brand Color Harmonization (Important Priority)
{{#if (or (eq focus "harmonization") (eq focus "all") (not focus))}}

**DC-018**: MaterialColors.harmonize() for brand colors
- [ ] `MaterialColors.harmonize(brandColor, primaryColor)`
- [ ] Brand color shifted to match primary hue
- [ ] Output used consistently throughout app
- [ ] Brand remains recognizable but cohesive

**DC-019**: Harmonization preserves brand recognition
- [ ] Harmonized color visibly related to original brand color
- [ ] Hue shifted but saturation/tone preserved
- [ ] A/B comparison shows clear similarity
- [ ] No color unrecognizable after harmonization

**DC-020**: HarmonizedColors for custom color sets
- [ ] `HarmonizedColors.Companion.from(context)` for system colors (if using extended colors)
- [ ] Contains pre-computed harmonized palette
- [ ] Extends beyond 5 standard color roles
- [ ] Used for brand/custom color preservation

**DC-021**: HarmonizedColorAttributes for per-color rules
- [ ] Define which custom color harmonizes with which role (primary/secondary/tertiary)
- [ ] Example: brand blue harmonizes with primary, brand accent with tertiary
- [ ] Applied to HarmonizedColors generation
- [ ] Ensures consistent custom color treatment

**DC-022**: No color hardcoding alongside harmonized colors
- [ ] All app colors either from MaterialTheme OR harmonized values
- [ ] No `Color(0xHHHHHH)` for core UI
- [ ] No mix of dynamic and static colors in same component
- [ ] Consistency throughout codebase

**DC-023**: Harmonization tested on multiple devices
- [ ] Tested on Pixel 6/7/8 (OLED, official Android)
- [ ] Tested on Samsung Galaxy S21+ (OLED, OneUI)
- [ ] Tested with different wallpaper colors
- [ ] Brand color harmonization consistent across devices

{{/if}}

#### Tonal Palette Generation (Important Priority)
{{#if (or (eq focus "color_roles") (eq focus "all") (not focus))}}

**DC-024**: Tonal palettes generated correctly
- [ ] Tonal Spot (default): balanced hue/saturation across tones
- [ ] Tone range: 0 (darkest) to 100 (lightest)
- [ ] Each tone is accessible color (meets WCAG standards)
- [ ] No skipped or compressed tone ranges

**DC-025**: Surface colors use tonal elevation
- [ ] Surface color increases in lightness with elevation
- [ ] Example: surface < surface1 < surface2 (in lightness)
- [ ] Tones derived from primary color's hue
- [ ] No shadow-based elevation overlays (old Material 2 pattern)

**DC-026**: Palette styles (Android 13+) tested
- [ ] If supporting Android 13+: Tonal Spot, Neutral, Vibrant Tonal, Expressive available
- [ ] User can switch styles (if supporting preference)
- [ ] Each style generates complete color scheme
- [ ] Fallback to Tonal Spot if device doesn't support style

**DC-027**: Tone contrast verification
- [ ] All text tones meet 4.5:1 WCAG AA minimum
- [ ] Dark surfaces use light text (high tone contrast)
- [ ] Light surfaces use dark text
- [ ] No tone inversion breaking readability

{{/if}}

#### Dark Theme Dynamic Color (Important Priority)
{{#if (or (eq focus "color_roles") (eq focus "all") (not focus))}}

**DC-028**: Dark theme dynamic colors applied correctly
- [ ] `dynamicDarkColorScheme(context)` used in dark mode
- [ ] Tones inverted from light (lighter colors for dark surfaces)
- [ ] Primary remains recognizable in dark
- [ ] Surface colors appear darker in dark theme

**DC-029**: Dark theme surface tones verified
- [ ] Dark surface: high tone number (80-95)
- [ ] Dark surface1: slightly lighter tone
- [ ] Dark background: very high tone (90-99)
- [ ] No pure black (#000000) - use surfaces instead

**DC-030**: System dark mode respected
- [ ] `AppCompatDelegate.setDefaultNightMode(MODE_NIGHT_FOLLOW_SYSTEM)`
- [ ] App theme switches when system dark mode toggles
- [ ] No forced light-only theming
- [ ] Scheduled dark mode (sunset/sunrise) works

{{/if}}

#### Fallback & Pre-Android 12 Support (Important Priority)
{{#if (or (eq focus "fallbacks") (eq focus "all") (not focus))}}

**DC-031**: Fallback color schemes defined (API < 31)
- [ ] `lightColorScheme()` defined with hardcoded colors
- [ ] `darkColorScheme()` defined with hardcoded colors
- [ ] All required color roles included
- [ ] Fallback colors match Material Design 3 spec

**DC-032**: Graceful degradation on old devices
- [ ] Android 11 and lower: no crashes when dynamic API called
- [ ] SDK check prevents API misuse: `if (SDK_INT >= S)`
- [ ] Fallback applied automatically
- [ ] No visual glitches or missing colors

**DC-033**: Testing on Android 11 emulator/device
- [ ] App launches successfully on API 30
- [ ] Colors appear reasonable (Material 3 defaults)
- [ ] No hardcoded dynamic color leaks to fallback
- [ ] User experience acceptable on older devices

**DC-034**: Version detection logic clean
- [ ] No repeated SDK checks (use constants)
- [ ] `Build.VERSION_CODES.S` (31) used consistently
- [ ] Refactored into utility function if repeated

{{/if}}

#### Testing & Validation (Important Priority)
{{#if (or (eq focus "testing") (eq focus "all") (not focus))}}

**DC-035**: Visual regression tests for dynamic color
- [ ] Screenshots taken on multiple wallpapers
- [ ] Color consistency verified programmatically
- [ ] Automated contrast ratio validation
- [ ] Light/dark mode both tested

**DC-036**: Unit tests for color harmonization
- [ ] `MaterialColors.harmonize()` output validated
- [ ] Harmonized color tone/hue within expected range
- [ ] Brand color recognizability metric (if custom)
- [ ] Fallback colors assigned correctly

**DC-037**: E2E tests on real devices
- [ ] Pixel 6/7/8 with system dynamic color on/off
- [ ] Samsung Galaxy with OneUI theming
- [ ] Multiple wallpaper colors tested
- [ ] App appears as intended on each setup

**DC-038**: Documentation of dynamic color behavior
- [ ] Documented which colors are dynamic vs. static
- [ ] Brand color harmonization strategy documented
- [ ] Fallback color choices justified
- [ ] Design rationale recorded

{{/if}}

## Key Implementation Patterns

### Basic Dynamic Color Setup

**GOOD: Application.kt**
```kotlin
import android.app.Application
import com.google.android.material.color.DynamicColors

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        // Enable dynamic color for all activities
        DynamicColors.applyToActivitiesIfAvailable(this)
    }
}
```

**GOOD: Theme.kt (Compose)**
```kotlin
import android.os.Build
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.dynamicDarkColorScheme
import androidx.compose.material3.dynamicLightColorScheme
import androidx.compose.material3.lightColorScheme
import androidx.compose.material3.darkColorScheme

@Composable
fun MyAppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        darkTheme -> darkColorScheme()
        else -> lightColorScheme()
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        shapes = Shapes,
        content = content
    )
}
```

**GOOD: Fallback Color Scheme**
```kotlin
private val LightColors = lightColorScheme(
    primary = Color(0xFF6200EE),
    onPrimary = Color(0xFFFFFFFF),
    secondary = Color(0xFF03DAC6),
    onSecondary = Color(0xFF000000),
    tertiary = Color(0xFFFFC100),
    onTertiary = Color(0xFF000000),
    error = Color(0xFFB3261E),
    onError = Color(0xFFFFFFFF),
    background = Color(0xFFFFFBFE),
    onBackground = Color(0xFF1C1B1F),
    surface = Color(0xFFFFFBFE),
    onSurface = Color(0xFF1C1B1F),
)

private val DarkColors = darkColorScheme(
    primary = Color(0xFFBB86FC),
    onPrimary = Color(0xFF370B1E),
    secondary = Color(0xFF03DAC6),
    onSecondary = Color(0xFF000000),
    tertiary = Color(0xFFFFC100),
    onTertiary = Color(0xFF000000),
    error = Color(0xFFF2B8B5),
    onError = Color(0xFF601410),
    background = Color(0xFF1C1B1F),
    onBackground = Color(0xFFE6E1E6),
    surface = Color(0xFF1C1B1F),
    onSurface = Color(0xFFE6E1E6),
)
```

### Brand Color Harmonization

**GOOD: Harmonize Brand Color with Primary**
```kotlin
import com.google.android.material.color.MaterialColors

// Get primary color from dynamic scheme
val primary = MaterialTheme.colorScheme.primary

// Harmonize brand color (e.g., custom accent) with primary
val brandColor = Color(0xFF00C853)  // Original green
val harmonizedBrand = MaterialColors.harmonize(
    brandColor.toArgb(),
    primary.toArgb()
).let { Color(it) }

// Use harmonized brand throughout app
Surface(
    color = harmonizedBrand,
    modifier = Modifier.fillMaxWidth()
) {
    Text("Custom accent that harmonizes with primary")
}
```

**GOOD: HarmonizedColors for Extended Color Sets**
```kotlin
import com.google.android.material.color.HarmonizedColors
import com.google.android.material.color.HarmonizedColorAttributes

// Get system harmonized colors (if API 31+)
val harmonizedColors = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
    HarmonizedColors.from(context)
} else {
    null
}

// Access harmonized values
val customBrandDynamic = harmonizedColors?.let {
    // Access harmonized custom color
    it.colorRoles.primary  // Already harmonized
} ?: Color(0xFF6200EE)  // Fallback

Surface(color = customBrandDynamic) {
    Text("Harmonized custom color")
}
```

### Testing Dynamic Color

**GOOD: Unit Test for Harmonization**
```kotlin
@RunWith(AndroidJUnit4::class)
class DynamicColorTest {

    @Test
    fun harmonizePreservesColorRecognition() {
        val brandBlue = Color(0xFF2196F3).toArgb()
        val primaryPurple = Color(0xFF6200EE).toArgb()

        val harmonized = MaterialColors.harmonize(brandBlue, primaryPurple)

        // Verify harmonized color is still recognizable as blue-ish
        assert(Color(harmonized).saturation > 0.3f)
    }

    @Test
    fun fallbackColorsDefinedForAllRoles() {
        val fallbackLight = lightColorScheme()

        assertNotNull(fallbackLight.primary)
        assertNotNull(fallbackLight.secondary)
        assertNotNull(fallbackLight.tertiary)
        assertNotNull(fallbackLight.error)
        assertNotNull(fallbackLight.surface)
    }
}
```

**GOOD: E2E Test on Multiple Devices**
```kotlin
@RunWith(AndroidJUnit4::class)
class DynamicColorE2ETest {

    @Test
    fun dynamicColorAppliedOnAndroid12Plus() {
        val scenario = activityScenarioRule.scenario

        scenario.onActivity { activity ->
            val colorScheme = MaterialTheme.colorScheme

            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
                // Verify colors are from dynamic palette (not all-default)
                // Check against known wallpaper colors
                assertThat(colorScheme.primary).isNotEqualTo(DEFAULT_PRIMARY)
            }
        }
    }
}
```

## Anti-Patterns to Avoid

**BAD: Hardcoded colors bypassing dynamic color**
```kotlin
// WRONG: Dynamic color enabled but colors hardcoded
Text(
    text = "Hello",
    color = Color(0xFF6200EE)  // Ignores dynamic primary
)
```

**GOOD: Use theme tokens**
```kotlin
Text(
    text = "Hello",
    color = MaterialTheme.colorScheme.primary
)
```

---

**BAD: Missing fallback colors for pre-Android 12**
```kotlin
// WRONG: No fallback if dynamicColorScheme fails
val colorScheme = dynamicLightColorScheme(context)  // Crashes on API < 31
```

**GOOD: With API check and fallback**
```kotlin
val colorScheme = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
    dynamicLightColorScheme(context)
} else {
    lightColorScheme()
}
```

---

**BAD: No tonal elevation for surfaces**
```kotlin
// WRONG: Using shadow instead of tonal elevation
Surface(
    shadowElevation = 8.dp,
    color = Color.White  // Hardcoded, not from scheme
) {
    Text("Content")
}
```

**GOOD: Tonal elevation**
```kotlin
Surface(
    modifier = Modifier.padding(8.dp),
    color = MaterialTheme.colorScheme.surface,
    tonalElevation = 8.dp  // Increases tone, not shadow
) {
    Text("Content")
}
```

---

**BAD: Brand color clashing with dynamic primary**
```kotlin
// WRONG: Brand color unharmonized, doesn't match theme
Surface(
    color = Color(0xFF00AA00)  // Fixed green, clashes with dynamic purple
) {
    Text("Brand element")
}
```

**GOOD: Harmonized brand color**
```kotlin
val primary = MaterialTheme.colorScheme.primary
val harmonizedBrand = MaterialColors.harmonize(
    Color(0xFF00AA00).toArgb(),
    primary.toArgb()
).let { Color(it) }

Surface(color = harmonizedBrand) {
    Text("Brand element harmonized with theme")
}
```

## Output Format

```markdown
## Dynamic Color Audit: [Component/App Name]

### Compliance Summary

| Check | Status | Issues | Priority |
|-------|--------|--------|----------|
| DC-001-005: API Setup | ✅/⚠️/❌ | None / Warnings / Critical | — |
| DC-006-010: Color Scheme | ✅/⚠️/❌ | Issues | — |
| DC-011-017: Color Roles | ✅/⚠️/❌ | Issues | — |
| DC-018-023: Harmonization | ✅/⚠️/❌ | Issues | — |
| DC-024-030: Palettes & Dark | ✅/⚠️/❌ | Issues | — |
| DC-031-034: Fallbacks | ✅/⚠️/❌ | Issues | — |
| DC-035-038: Testing | ✅/⚠️/❌ | Issues | — |

### Critical Issues (🔴)

**[DC-###] Issue Title**
```kotlin
// Current code (BAD)
// Issue description, severity, confidence, rule ID

// Fixed code (GOOD)
// Solution
```

### Important Issues (🟠)

**[DC-###] Issue Title**
- Description
- Impact
- Fix

### Warnings (🟡)

**[DC-###] Issue Title**
- Description
- Suggestion

### Suggestions (🔵)

**[DC-###] Optimization Opportunity**
- Description
- Benefit

### Summary

- **Overall Readiness**: X%
- **Critical**: X | **Important**: X | **Warning**: X | **Suggestion**: X
- **Min API Supported**: 31+ (Android 12+) / Lower with fallbacks

### Action Plan (Priority Order)

1. **IMMEDIATE (Critical)**
   - [Most urgent fix]

2. **BEFORE RELEASE (Important)**
   - [Second priority]

3. **NICE TO HAVE (Suggestions)**
   - [Polish/optimization]

### Recommended Resources

- [Material Design 3 in Compose - Android Developers](https://developer.android.com/develop/ui/compose/designsystems/material3)
- [Material 3 Dynamic Color - Material Design](https://m3.material.io/styles/color/dynamic-color/overview)
- [Enable Users to Personalize Color - Android Developers](https://developer.android.com/develop/ui/views/theming/dynamic-colors)
- [Color Harmonization Guide - Material Design 3](https://m3.material.io/blog/dynamic-color-harmony)
- [MaterialColors API Reference - Android Developers](https://developer.android.com/reference/com/google/android/material/color/MaterialColors)
- [Mastering Material 3 in Jetpack Compose — The 2025 Guide - Medium](https://medium.com/@hiren6997/mastering-material-3-in-jetpack-compose-the-2025-guide-1c1bd5acc480)
```

## Code to Audit

```
{{code}}
```

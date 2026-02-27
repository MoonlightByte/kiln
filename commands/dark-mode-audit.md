---
name: dark-mode-audit
description: Comprehensive dark mode audit for Material 3 and iOS semantic colors compliance
args:
  - name: code
    type: string
    description: Mobile code (Compose or SwiftUI) to audit for dark mode
    required: true
  - name: platform
    type: string
    enum: [android, ios, auto]
    description: Target platform (default: auto-detect)
    required: false
  - name: include_images
    type: boolean
    description: Check image and media handling in dark mode
    required: false
    default: true
  - name: contrast_level
    type: string
    enum: [aa, aaa]
    description: WCAG contrast target (AA=4.5:1 normal, AAA=7:1 normal)
    required: false
    default: aa
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Dark Mode Compliance Audit

Comprehensive dark mode audit covering Material 3 color tokens, iOS semantic colors, dynamic color, OLED optimization, WCAG contrast compliance, and image handling best practices for 2025-2026.

## Instructions

Analyze the code for dark mode compliance. Auto-detect platform based on syntax (`@Composable` = Android, `some View` = iOS).

Review against the latest Material 3 and iOS dark mode standards covering:
- Correct use of color tokens and tonal elevation
- Dynamic color support (Android 12+)
- OLED optimization strategies
- Contrast ratio compliance (WCAG AA/AAA)
- Image and media handling
- System appearance detection
- Testing readiness

## Material 3 Dark Theme Standards (Android)

### Color Tokens & Tonal Elevation (Critical)

**2025 Material 3 Changes:**
- Surface colors now use **tonal elevation** instead of shadow-based elevation
- Surface color takes tonal tint from primary color based on elevation
- Five predefined surface tonal variations: Surface, Surface1-Surface5
- Background color remains consistent; surface color varies with elevation

**Audit Points:**
- [ ] Using `MaterialTheme.colorScheme.surface*` tokens, not custom colors
- [ ] Tonal elevation applied correctly (`Surface`, `Surface1` through `Surface5`)
- [ ] Surface colors get lighter/more colorful at higher elevations (approaching light source)
- [ ] No shadow-based elevation overlays in dark theme
- [ ] Primary color controls tonal tint across surface variants

**Reference:**
- [Material Design 3 in Compose - Android Developers](https://developer.android.com/develop/ui/compose/designsystems/material3)
- [Theming in Compose with Material 3 - Codelabs](https://codelabs.developers.google.com/jetpack-compose-theming)
- [Color roles and tokens - Wear Android](https://developer.android.com/design/ui/wear/guides/styles/color/roles-tokens)

### Dynamic Color Support (Android 12+) (Important)

**Implementation:**
- Enable `DynamicColors.applyToActivitiesIfAvailable(application)` in Application.onCreate()
- Supports Tonal Spot, Neutral, Vibrant Tonal, and Expressive variants (Android 13+)
- Personalizes app colors to user's wallpaper/system theme
- Design tokens make dynamic color seamless

**Audit Points:**
- [ ] Dynamic Color API enabled for Android 12+
- [ ] Theme uses semantic color roles (not hardcoded hex values)
- [ ] Works correctly with Material3.DayNight theme
- [ ] Testing on multiple dynamic color schemes
- [ ] Fallback colors defined for pre-Android 12 devices

**Reference:**
- [Enable personalized color experience - Android Views](https://developer.android.com/develop/ui/views/theming/dynamic-colors)
- [Implementing Dynamic Color - Chrome team lessons](https://android-developers.googleblog.com/2022/05/implementing-dynamic-color-lessons-from.html)
- [Apply Dynamic Color - Codelabs](https://codelabs.developers.google.com/codelabs/apply-dynamic-color)

### Dark Theme Implementation Checklist (Android)

**Required:**
- [ ] Strings use `android:colorBackground` and `android:colorForeground` (API 29+)
- [ ] Using `Theme.Material3.DayNight` or custom DayNight variant
- [ ] `<item name="android:forceDarkAllowed">false</item>` if custom dark implemented
- [ ] No hardcoded `#FFF` white or `#000` black in color resources
- [ ] All color resources have day/night variants
- [ ] Testing in system dark/light mode (Settings > Display > Dark theme)

## iOS Dark Mode Standards (SwiftUI & UIKit)

### Semantic Colors (Critical)

**iOS Color System (Auto-adapting):**
- Use semantic colors that work in both light AND dark automatically
- System colors include: `label`, `secondaryLabel`, `systemBackground`, `secondarySystemBackground`, `systemGray`, `systemFill`
- Custom colors in Asset Catalog can have light/dark variants
- Do NOT use explicit `Color(red:, green:, blue:)` without dark variant

**Audit Points:**
- [ ] Using `Color(UIColor.label)`, `Color(UIColor.secondaryLabel)`, not custom RGB
- [ ] `Color(UIColor.systemBackground)` for backgrounds in both modes
- [ ] Custom colors defined in Asset Catalog with Appearance set to Light/Dark
- [ ] No `@Environment(\.colorScheme)` workarounds for standard UI
- [ ] Testing with Environment Override: Light/Dark Appearance

**SwiftUI Implementation:**
```swift
// Correct: semantic color
Text("Hello").foregroundColor(.primary)  // Uses label in dark mode

// Correct: custom light/dark variant
Color("CustomBackground")  // Defined in Asset Catalog with Both appearances

// Wrong: hardcoded color
Text("Hello").foregroundColor(.white)  // Breaks in dark mode
```

**Reference:**
- [Supporting Dark Mode - Apple Developer Docs](https://developer.apple.com/documentation/uikit/supporting-dark-mode-in-your-interface)
- [Dark Mode - Apple HIG](https://developer.apple.com/design/human-interface-guidelines/dark-mode)
- [Dark Theme Mobile UI Best Practices 2025](https://www.mindinventory.com/blog/how-to-design-dark-mode-for-mobile-apps/)

### iOS Color Desaturation Rule (Important)

**Design Principle:**
- Saturated colors vibrate against dark surfaces visually
- Desaturate main colors in dark mode for sufficient contrast
- Example: Brand red `#FF0000` → `#CC3333` in dark mode

**Audit Points:**
- [ ] Brand colors desaturated in dark mode
- [ ] Saturation reduced 15-25% compared to light mode colors
- [ ] Testing in low-light and bright environments
- [ ] No pure saturation (100%) colors in dark surfaces

## WCAG Contrast Ratio Standards (Critical)

**WCAG 2.2 Level AA (Minimum):**
- Normal text: **4.5:1** ratio (applies in dark mode)
- Large text (24sp+/18pt+): **3:1** ratio
- UI components and graphics: **3:1** ratio
- Dark mode requires same ratios - NOT stricter, but often harder to achieve

**WCAG 2.2 Level AAA:**
- Normal text: **7:1** ratio
- Large text: **4.5:1** ratio

**Dark Mode Contrast Pitfalls:**
- Pure black `#000000` creates > 15:1 contrast, causes halation effect and eye strain
- Use softer dark grays instead (e.g., `#1C1C1E`, `#121212`)
- Soft grays often need testing to confirm they meet 4.5:1

**Audit Points:**
- [ ] Text tested against actual background color (not assumed)
- [ ] No pure black (#000000) - use `#1C1C1E` or Material `surfaceDim`
- [ ] All text meets 4.5:1 minimum (or 3:1 for large text)
- [ ] Icons and decorative graphics meet 3:1 against background
- [ ] Testing on actual devices (simulators may vary)

**Tools:**
- Contrast ratio checkers: [WCAG Color Contrast Checker](https://accessibleweb.com/color-contrast-checker/)
- [WebAIM Contrast and Color Accessibility](https://webaim.org/articles/contrast/)
- [Complete Accessibility Guide WCAG 2.1 AA - 2026](https://blog.greeden.me/en/2026/02/23/complete-accessibility-guide-for-dark-mode-and-high-contrast-color-design-contrast-validation-respecting-os-settings-icons-images-and-focus-visibility-wcag-2-1-aa/)

**Reference:**
- [Contrast requirements for WCAG 2.2 Level AA](https://www.makethingsaccessible.com/guides/contrast-requirements-for-wcag-2-2-level-aa/)
- [Color contrast - MDN Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Guides/Understanding_WCAG/Perceivable/Color_contrast)

## OLED Optimization Strategy

### True Black Implementation (Important)

**OLED Physics:**
- OLED pixels turn OFF completely at black, reducing power consumption 20-40%
- Enables perfect black without backlight bleed
- "True Black" `#000000` is often too extreme (eye strain, halation)
- Sweet spot: `#0A0E27` (near-black) or Material `surface` colors

**Audit Points:**
- [ ] Using near-black instead of `#000000` for backgrounds
- [ ] Dark theme reduces blue sub-pixel load (battery savings)
- [ ] Testing on actual OLED devices (not LCD)
- [ ] Pure black used only for special emphasis (rare)
- [ ] Verified dark mode appearance on Android OLED phones

**Reference:**
- [OLED Dark Theme Optimization 2026](https://www.tech-rz.com/blog/dark-mode-design-best-practices-in-2026/)
- [OLED Display Technology - High Contrast & Power Efficiency](https://www.displaymodule.com/blogs/knowledge/oled-display-technology-high-contrast-fast-response-energy-efficiency)

### Micro-contrast Optimization (Suggestion)

**OLED Advantage:**
- Infinite contrast ratio (0 current when pixels off)
- Enhanced micro-contrast on character edges
- Reduced PPI needed for clarity vs LCD

**Audit Points:**
- [ ] Character edges are crisp in dark mode
- [ ] Text anti-aliasing renders correctly on dark
- [ ] No gray text (use `#B0BCCC` or higher for readability)

## Image & Media Handling in Dark Mode

### Adaptive Image Strategies (Important)

**Problem:** Images that look balanced in light mode may feel too bright in dark mode, pulling focus away from UI.

**Solutions:**
1. **Separate image versions** for light/dark (PNG)
2. **SVG with CSS styling** (change fill colors per theme)
3. **Adaptive containers** (subtle scrims, toned borders around images)
4. **Fallback logic** for older devices

**Audit Points:**
- [ ] All images have dark-mode versions (or adaptive styling)
- [ ] Photos don't overwhelm dark UI (toned borders/scrims)
- [ ] SVG icons styled with theme colors, not hardcoded
- [ ] Screenshots in dark-themed UX use dark backgrounds
- [ ] Testing on actual devices (simulator images may differ)

**Reference:**
- [How to Handle Images in Dark Mode](https://thisisglance.com/learning-centre/how-do-i-handle-images-and-graphics-in-dark-mode/)
- [Rethinking Dark Mode Image Strategies with Media Queries](https://www.sarasoueidan.com/newsletter/issue-2025-10-31/)
- [Dark Mode Image Handling 2025](https://ui-deploy.com/blog/complete-dark-mode-design-guide-ui-patterns-and-implementation-best-practices-2025/)

### Icon & Graphic Audit (Important)

**Common Issues:**
- Thin white line icons disappear on light backgrounds
- Dark text in screenshots vanishes against dark themes
- Decorative graphics (e.g., illustrations) may need desaturation

**Audit Points:**
- [ ] Thin-line icons have stroke weight optimized for both modes
- [ ] All screenshots use appropriate theme context
- [ ] Illustrations desaturated slightly in dark mode
- [ ] Emoji and colored graphics render correctly in dark
- [ ] Testing on various device backgrounds

### CSS/Layout-level Image Handling (Web-applicable)

**Recommended Patterns:**
```swift
// SwiftUI: Conditional image based on color scheme
@Environment(\.colorScheme) var colorScheme

Image(colorScheme == .dark ? "imageDark" : "imageLight")

// Or use symbol variants
Image(systemName: "sun.max").foregroundColor(.yellow)
```

```kotlin
// Android Compose: Use color filter for images
Image(
    painter = painterResource(id = R.drawable.icon),
    modifier = Modifier.colorFilter(
        if (isDarkMode) ColorFilter.tint(Color.White) else ColorFilter.tint(Color.Black)
    )
)
```

## System Appearance Detection & Preferences

### Respecting User Settings (Important)

**Android:**
- [ ] No hardcoded light/dark forcing
- [ ] `AppCompatDelegate.setDefaultNightMode(MODE_NIGHT_FOLLOW_SYSTEM)`
- [ ] Testing in Settings > Display > Dark theme (on/off/auto)

**iOS:**
- [ ] Using `@Environment(\.colorScheme) var colorScheme` in SwiftUI
- [ ] UITraitCollection on UIKit
- [ ] Testing with Appearance Override in preview and device

**Audit Points:**
- [ ] App respects system appearance preference
- [ ] No settings forced to light or dark mode
- [ ] Scheduled dark mode (sunset to sunrise) works if enabled
- [ ] Transitions between light/dark are smooth (no flashing)

## Dark Mode Testing Checklist

### Visual Testing (Important)

- [ ] **Device Testing**
  - Test on actual Android OLED device (Pixel, Samsung)
  - Test on actual iOS device (iPhone 13+)
  - NOT just simulators (colors may differ significantly)

- [ ] **Lighting Conditions**
  - Low-light environment (dimly lit room)
  - Bright sunlight
  - Medium indoor lighting
  - Confirm readability in all conditions

- [ ] **Color Scheme Combinations**
  - System light + app light
  - System dark + app dark
  - Scheduled transitions (sunrise/sunset)
  - Dynamic colors (Android 12+) on multiple devices

- [ ] **Contrast Verification**
  - Text on all background colors (4.5:1 minimum)
  - Focus indicators visible
  - Buttons and interactive elements distinct
  - Icons against both light and dark backgrounds

- [ ] **Image & Media Review**
  - Photos in dark mode appearance appropriate
  - Screenshots match current theme context
  - SVG icons render correctly in dark
  - No images too bright or too dark

### Automated Testing (Important)

- [ ] Visual regression tests for dark mode
  - Screenshot comparison: light vs dark
  - Automated contrast ratio validation
  - Icon visibility checks

- [ ] Unit Tests
  - Color token correctness
  - Dynamic color fallbacks
  - Contrast ratio calculations

**Reference:**
- [How to Test Dark Mode Effectively - BrowserStack 2026](https://www.browserstack.com/guide/how-to-test-apps-in-dark-mode/)
- [Dark Mode UX 2025: Testing Strategies](https://www.influencers-time.com/dark-mode-ux-in-2025-design-tips-for-comfort-and-control/)

## Code Patterns to Audit

### Android (Jetpack Compose)

**GOOD:**
```kotlin
// Using theme tokens
Text(
    text = "Hello",
    color = MaterialTheme.colorScheme.onSurface,
    style = MaterialTheme.typography.bodyMedium
)

// Tonal elevation
Surface(
    modifier = Modifier.padding(16.dp),
    color = MaterialTheme.colorScheme.surface,
    tonalElevation = 8.dp
) {
    // Content
}

// Dynamic color enabled
class MyApplication: Application() {
    override fun onCreate() {
        DynamicColors.applyToActivitiesIfAvailable(this)
    }
}

// Respecting system dark mode
<item name="android:colorBackground">@color/background</item>
```

**BAD:**
```kotlin
// Hardcoded colors (breaks dynamic color)
Text(text = "Hello", color = Color(0xFF6B7280))

// Shadow-based elevation (old style)
Surface(shadowElevation = 8.dp) { }

// Pure white or pure black
Text(color = Color.White)
Surface(color = Color.Black)

// Ignoring system dark mode
setTheme(R.style.Theme_Light)  // Always light
```

### iOS (SwiftUI)

**GOOD:**
```swift
// Semantic colors
Text("Hello")
    .foregroundColor(.primary)  // Adapts to light/dark

// Custom light/dark variant in Asset Catalog
Text("Title")
    .foregroundColor(Color("CustomText"))

// Dynamic color scheme support
@Environment(\.colorScheme) var colorScheme

// Respecting system appearance
Text(colorScheme == .dark ? "🌙" : "☀️")

// Semantic backgrounds
ZStack {
    Color(UIColor.systemBackground)
    // Content
}
```

**BAD:**
```swift
// Hardcoded white or black
Text("Hello").foregroundColor(.white)
Text("Hello").foregroundColor(.black)

// Not using Asset Catalog variants
Color(red: 0.5, green: 0.5, blue: 0.5)

// Hardcoded colors that don't adapt
Color(UIColor(red: 1, green: 1, blue: 1, alpha: 1))

// Ignoring colorScheme
Text(colorScheme == .dark ? Color.white : Color.black)  // Wrong - just use .primary
```

## Platform-Specific Checks

### Android Deep Dive

- [ ] `values-night/colors.xml` exists with dark variants
- [ ] No `android:forceDarkAllowed="true"` (forced dark breaks custom themes)
- [ ] `Theme.Material3.DayNight` base theme
- [ ] `DynamicColors` enabled in Application
- [ ] Night mode resources: `values-night/`, `values-night-v31/` for API 31+
- [ ] No WebView forcing dark/light (let system decide)

### iOS Deep Dive

- [ ] All colors defined in Asset Catalog with Appearance = "Light & Dark"
- [ ] `@Environment(\.colorScheme)` used sparingly (prefer semantic colors)
- [ ] Preview includes `.preferredColorScheme(.dark)` modifier
- [ ] No `UIColor.white` / `UIColor.black` (use semantic instead)
- [ ] Testing both `overrideUserInterfaceStyle = .light` and `.dark`

## Checklist for Complete Dark Mode Compliance

```markdown
## Dark Mode Audit Checklist

### Material 3 Color Tokens
- [ ] Surface colors use tonal elevation (not shadows)
- [ ] All colors from MaterialTheme.colorScheme.*
- [ ] Surface1-5 used appropriately for elevation
- [ ] No hardcoded hex values for core UI

### Dynamic Color (Android 12+)
- [ ] DynamicColors.applyToActivitiesIfAvailable() enabled
- [ ] Theme.Material3.DayNight base theme in use
- [ ] Tested on multiple Android devices with dynamic color
- [ ] Fallback colors defined for pre-Android 12

### iOS Semantic Colors
- [ ] Using Color(UIColor.label), .secondary, .systemBackground
- [ ] All custom colors in Asset Catalog with light/dark variants
- [ ] @Environment(\.colorScheme) used only when necessary
- [ ] Previews include .preferredColorScheme(.dark)

### WCAG Contrast (AA/AAA)
- [ ] All text >= 4.5:1 ratio (normal) or 3:1 (large)
- [ ] Icons/graphics >= 3:1 against background
- [ ] No pure black (#000000), using #1C1C1E or darker gray
- [ ] Tested on actual devices

### OLED Optimization
- [ ] Near-black colors reduce power consumption
- [ ] Character micro-contrast verified on OLED devices
- [ ] No excessive brightness in dark mode
- [ ] Tested on OLED display (Pixel, Samsung)

### Image & Media
- [ ] All images have dark-mode variants or adaptive styling
- [ ] Photos don't overwhelm dark UI
- [ ] SVG icons themed with color filters
- [ ] Screenshots use appropriate theme context

### System Appearance
- [ ] Respects system light/dark preference
- [ ] No forced light-only or dark-only mode
- [ ] Smooth transitions between modes
- [ ] Scheduled dark mode works (if enabled)

### Testing
- [ ] Tested on actual devices in multiple lighting
- [ ] Low-light, bright sunlight, medium indoor tested
- [ ] Dynamic color schemes tested
- [ ] Visual regression tests for both modes
- [ ] Contrast ratio automated validation

### Code Quality
- [ ] No @Deprecated color properties
- [ ] Night mode resources organized (values-night/)
- [ ] Custom theme extends Material3 base
- [ ] No WebView dark mode forcing
```

## Output Format

```markdown
## Dark Mode Audit: [Component/App Name]

### Compliance Summary

| Category | Status | Issues | Priority |
|----------|--------|--------|----------|
| Material 3 Tokens | ⚠️ | 3 hardcoded colors | Critical |
| Dynamic Color | ✅ | None | — |
| iOS Semantics | ❌ | 5 Color.white references | Critical |
| WCAG Contrast | ⚠️ | 2 text colors < 4.5:1 | Important |
| OLED Optimization | ✅ | None | — |
| Image Handling | ⚠️ | 4 images not dark-adapted | Important |
| System Detection | ✅ | None | — |
| Testing | ⚠️ | No device testing logged | Important |

### Critical Issues (Platform-Specific)

**Android: Hardcoded Colors Instead of Material3 Tokens**
```kotlin
// Line 45
Text(
    text = "Hello",
    color = Color(0xFF6B7280)  // WRONG: hardcoded gray
)

// Fix: Use MaterialTheme token
Text(
    text = "Hello",
    color = MaterialTheme.colorScheme.onSurface
)
```

**iOS: Using Color.white Instead of Semantic**
```swift
// Line 82
Text("Hello")
    .foregroundColor(.white)  // WRONG: breaks in dark mode

// Fix: Use semantic color
Text("Hello")
    .foregroundColor(.primary)  // Adapts automatically
```

**WCAG Contrast Violation**
```
Text: #BBBEC3 on Surface: #121212
Ratio: 3.2:1 (FAIL - need 4.5:1)

Fix: Use #FFFFFF (#FFFFFF on #121212 = 18.5:1)
```

### Important Issues

**Image Not Dark-Mode Adapted**
```
Screenshot at Line 124 shows light-mode UI
Dark mode appearance would be overwhelming/illegible

Fix: Create dark variant or add adaptive scrim
```

**Missing Dynamic Color Support**
```
Android 12+ devices won't personalize colors
DynamicColors.applyToActivitiesIfAvailable() not called

Fix: Add to Application.onCreate()
```

### Warnings

**No Dark Mode Testing Logged**
- Verify on actual OLED device (Pixel 7+, Samsung S23)
- Test in low-light and bright sunlight
- Validate dynamic color schemes

### Suggestions

**OLED Optimization Opportunity**
- Change background from #1C1C1E to #0A0E27 (deeper black)
- Reduces power consumption by additional 15-20% on OLED
- Verify text contrast still meets 4.5:1

### Summary

**Overall Dark Mode Readiness: 70%**
- Critical: 2 (hardcoded colors, iOS semantics)
- Important: 3 (contrast, images, dynamic color)
- Warning: 1 (testing verification)

### Action Plan (Priority Order)

1. **IMMEDIATE** (Critical)
   - Replace all `Color(0xHHHHHH)` with `MaterialTheme.colorScheme.*`
   - Replace `Color.white`/`Color.black` with semantic colors
   - Add `DynamicColors.applyToActivitiesIfAvailable()` in Android

2. **BEFORE RELEASE** (Important)
   - Fix text contrast: test all text on actual dark backgrounds
   - Create dark-mode image variants or adaptive containers
   - Test on actual OLED devices (not just emulator)

3. **NICE TO HAVE** (Suggestion)
   - Implement OLED micro-optimization with #0A0E27
   - Add automated contrast ratio validation tests
   - Document dark mode testing checklist in QA docs

### Device Testing Checklist

- [ ] Android Pixel 7/8 (OLED) - system dark on/off
- [ ] Samsung Galaxy S23+ (OLED) - system dark on/off
- [ ] iPhone 13+ - Appearance override dark
- [ ] Low-light environment (< 50 lux)
- [ ] Bright sunlight (> 1000 lux)
- [ ] Medium indoor (200-500 lux)
- [ ] Dynamic color schemes (Android 12+)
- [ ] Scheduled dark mode (sunset to sunrise)

### Recommended Resources

- [Material Design 3 in Compose](https://developer.android.com/develop/ui/compose/designsystems/material3)
- [Apple Dark Mode HIG](https://developer.apple.com/design/human-interface-guidelines/dark-mode)
- [WCAG 2.2 Contrast Checker](https://accessibleweb.com/color-contrast-checker/)
- [2025 Dark Mode Best Practices](https://ui-deploy.com/blog/complete-dark-mode-design-guide-ui-patterns-and-implementation-best-practices-2025/)
- [OLED Optimization Guide 2026](https://www.tech-rz.com/blog/dark-mode-design-best-practices-in-2026/)
```

## Code to Audit

```
{{code}}
```

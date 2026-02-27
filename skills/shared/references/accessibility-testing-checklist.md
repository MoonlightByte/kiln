# Accessibility Testing Checklist

**Standard**: WCAG 2.2 Level AA, Axe-Core Mobile
**Platforms**: Android (Jetpack Compose), iOS (SwiftUI)
**Version**: 1.0 (February 2025)

---

## Phase 1: Automated Testing

### 1.1 Static Code Analysis (Before Runtime)

#### Android - Lint Checks
```bash
# Run accessibility lint checks
cd android
./gradlew lint

# Check for specific a11y issues
grep -r "contentDescription=\"\"" app/src/main/
# Should return: EMPTY - all interactive elements need descriptions
```

**Required Checks:**
- [ ] No `contentDescription = ""` in Images/Icons
- [ ] No `contentDescription = null` in interactive elements
- [ ] All `clickable` modifiers have semantic labels
- [ ] No hardcoded colors without contrast verification

#### iOS - Xcode Accessibility Audit
```bash
# In Xcode, run accessibility audit
xcodebuild test -scheme PartyOn -derivedDataPath build \
  -only-testing:PartyOnTests/AccessibilityTests
```

**Required Checks:**
- [ ] No missing `accessibilityLabel` on interactive elements
- [ ] No missing `accessibilityHint` where needed
- [ ] All images have descriptions
- [ ] No orphaned text without context

### 1.2 Axe-Core Mobile Testing

#### Android - Axe-Core for Android Integration

```gradle
// Add to build.gradle.kts
dependencies {
    androidTestImplementation("com.deque.android.axe:axe-android:2.3.0")
}
```

**Test Class**:
```kotlin
@RunWith(AndroidJUnit4::class)
class AccessibilityAuditTest {
    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun auditLoginScreen() {
        composeTestRule.setContent {
            LoginScreen()
        }

        // Take screenshot for manual review
        composeTestRule.captureToImage().asAndroidBitmap()

        // Check all interactive elements
        composeTestRule.onAllNodes(
            isEnabled() and (isClickable() or isToggleable())
        ).assertAll(hasContentDescription())

        // Verify touch target sizes
        composeTestRule.onAllNodes(isClickable()).forEach { node ->
            val bounds = node.fetchSemanticsNode().boundsInRoot
            assert(bounds.width >= 48.dp && bounds.height >= 48.dp) {
                "Touch target too small: ${bounds.width}x${bounds.height}"
            }
        }
    }

    private fun hasContentDescription() = SemanticsMatcher.expectValue(
        SemanticsProperties.ContentDescription,
        listOf<String>()  // Non-empty required
    )
}
```

#### iOS - Axe-Core for iOS Integration

```swift
// Add to Package.swift or Podfile
// pod 'AxeCore', '~> 1.0'

import AxeCore
import XCTest

class AccessibilityAuditTest: XCTestCase {
    func testAppAccessibility() async throws {
        let app = XCUIApplication()
        app.launch()

        // Run axe scan on current screen
        let axe = AxeDevTools(application: app)
        let results = await axe.scan()

        // Filter by severity
        let criticalIssues = results.violations.filter { $0.impact == .critical }
        let seriousIssues = results.violations.filter { $0.impact == .serious }

        // Assert no critical violations
        XCTAssertEqual(criticalIssues.count, 0)

        // Report on serious issues
        if !seriousIssues.isEmpty {
            print("Found \(seriousIssues.count) serious accessibility issues:")
            for issue in seriousIssues {
                print("  - \(issue.description)")
            }
        }
    }
}
```

### 1.3 Automated Accessibility Scans (CI/CD)

#### GitHub Actions - Automated Accessibility Testing

```yaml
# .github/workflows/accessibility-audit.yml
name: Accessibility Audit

on: [pull_request, push]

jobs:
  android-a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: android-actions/setup-android@v2

      - name: Run Android accessibility tests
        run: |
          cd android
          ./gradlew :app:connectedAndroidTest -Dtests.accessibility=true

      - name: Upload accessibility report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: accessibility-report-android
          path: android/app/build/reports/androidTests/connected/

  ios-a11y:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run iOS accessibility tests
        run: |
          cd ios
          xcodebuild test -scheme PartyOn \
            -derivedDataPath build \
            -only-testing:PartyOnTests/AccessibilityTests

      - name: Upload accessibility report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: accessibility-report-ios
          path: ios/build/Logs/Test/
```

---

## Phase 2: Manual Testing with Screen Readers

### 2.1 TalkBack Testing (Android)

#### Enable TalkBack
```
Settings > Accessibility > TalkBack > Toggle ON
Shortcut: Volume Up + Volume Down (3 sec)
```

#### Test Checklist
- [ ] **Navigation**: Can swipe left/right to navigate all interactive elements
- [ ] **Labels**: All buttons/inputs have clear, descriptive labels (not "Button")
- [ ] **Feedback**: Actions announce their result (e.g., "Notification posted")
- [ ] **Forms**: Error messages announce clearly and suggest corrections
- [ ] **Lists**: Item count announced, scrollable content clear
- [ ] **Images**: All meaningful images have descriptions
- [ ] **Reading Order**: Content flows logically (top to bottom, left to right)

#### TalkBack Testing Script
```kotlin
@Test
fun testTalkBackNavigation() {
    // 1. Open app
    composeTestRule.setContent { LoginScreen() }

    // 2. Verify each interactive element
    composeTestRule.onNodeWithContentDescription("Email input field")
        .assertExists()
        .assert(hasSetTextAction())

    composeTestRule.onNodeWithContentDescription("Password input field")
        .assertExists()
        .assert(hasSetTextAction())

    composeTestRule.onNodeWithContentDescription("Login button")
        .assertExists()
        .assert(isClickable())

    // 3. Verify error announcements
    // Type invalid email, verify error announced
}
```

#### Common TalkBack Issues & Fixes

| Issue | Symptoms | Fix |
|-------|----------|-----|
| **Missing description** | TalkBack says "Button" instead of action | Add `contentDescription` |
| **Generic label** | "Image" instead of "User avatar" | Replace with specific text |
| **Non-interactive element focusable** | TalkBack stops on spacing elements | Remove `focusable()` modifier |
| **Focus order wrong** | Navigation jumps around illogically | Use `semantics { role = Role.Button }` |
| **Dynamic content not announced** | TalkBack doesn't announce "3 new messages" | Use `LiveRegionMode.Assertive` |

### 2.2 VoiceOver Testing (iOS)

#### Enable VoiceOver
```
Settings > Accessibility > VoiceOver > Toggle ON
Shortcut: Triple-tap home/side button
```

#### Test Checklist
- [ ] **Navigation**: Can use rotor (2-finger rotate) to navigate by type
- [ ] **Labels**: All buttons/inputs have clear, descriptive labels
- [ ] **Feedback**: Actions announce their result
- [ ] **Forms**: Error messages announce and suggest corrections
- [ ] **Lists**: Item count announced, swipe to navigate
- [ ] **Images**: All meaningful images have descriptions
- [ ] **Safe Areas**: Notch/home indicator don't interfere

#### VoiceOver Testing Script
```swift
@MainActor
func testVoiceOverNavigation() {
    let app = XCUIApplication()
    app.launch()

    // 1. Verify email field exists and is accessible
    let emailField = app.textFields["email_field"]
    XCTAssertTrue(emailField.exists)
    XCTAssertTrue(emailField.isAccessibilityElement)

    // 2. Tap and verify announcement
    emailField.tap()
    // VoiceOver announces: "Email input field, text field"

    // 3. Verify password field
    let passwordField = app.secureTextFields["password_field"]
    XCTAssertTrue(passwordField.exists)

    // 4. Verify login button
    let loginButton = app.buttons["submit_button"]
    XCTAssertTrue(loginButton.exists)
    loginButton.tap()
    // VoiceOver announces: "Login successful" or error message
}
```

---

## Phase 3: Keyboard Navigation Testing

### 3.1 Android - External Keyboard Navigation

#### Enable External Keyboard
```
Device: Attach Bluetooth keyboard or USB keyboard
In Emulator: Extended controls > Virtual Sensors > Keyboard input
```

#### Navigation Test
```kotlin
@Test
fun testKeyboardNavigation() {
    composeTestRule.setContent { LoginScreen() }

    // Tab through elements
    // Expected: Focus moves Email → Password → Login → OK

    // Shift+Tab backwards
    // Expected: Focus moves Login → Password → Email

    // Enter/Space on focused button
    // Expected: Button activated (color change + action)

    // Escape/Back key
    // Expected: Dialog closes or app returns to previous screen
}
```

#### Expected Behavior
- [ ] Tab moves focus forward (logical order)
- [ ] Shift+Tab moves focus backward
- [ ] Enter/Space activates focused button
- [ ] Arrow keys navigate list/grid items
- [ ] Esc/Back key exits modals/dialogs
- [ ] Focus indicator visible at all times (3:1 contrast)

### 3.2 iOS - Software Keyboard Navigation

#### Full Keyboard Access
```
Settings > Accessibility > Keyboards > Full Keyboard Access > Toggle ON
```

#### Navigation Test
```swift
@MainActor
func testKeyboardNavigation() {
    let app = XCUIApplication()
    app.launch()

    // Tab through elements
    // Expected: Focus moves Email → Password → Login

    // Shift+Tab backwards
    // Expected: Focus moves Login → Password → Email

    // Space/Enter on focused button
    // Expected: Button activated

    // Escape key
    // Expected: Dialog closes
}
```

#### Expected Behavior
- [ ] Tab moves focus forward
- [ ] Shift+Tab moves focus backward
- [ ] Space/Enter activates focused element
- [ ] Arrow keys navigate lists/tables
- [ ] Return key submits forms
- [ ] Escape closes modals

---

## Phase 4: Contrast & Color Testing

### 4.1 Automated Contrast Checking

#### Android - Runtime Contrast Verification
```kotlin
@Test
fun testContrastRatios() {
    val textColor = Color(0xFF212121)  // Dark gray
    val backgroundColor = Color(0xFFFFFFFF)  // White

    val contrast = calculateContrast(textColor, backgroundColor)
    assert(contrast >= 4.5) { "Contrast $contrast is below 4.5:1" }
}

fun calculateContrast(foreground: Color, background: Color): Double {
    val fgLum = relativeLuminance(foreground)
    val bgLum = relativeLuminance(background)
    val lighter = maxOf(fgLum, bgLum)
    val darker = minOf(fgLum, bgLum)
    return (lighter + 0.05) / (darker + 0.05)
}
```

#### iOS - Runtime Contrast Verification
```swift
@Test
func testContrastRatios() {
    let textColor = UIColor(red: 0.13, green: 0.13, blue: 0.13, alpha: 1.0)
    let backgroundColor = UIColor(red: 1.0, green: 1.0, blue: 1.0, alpha: 1.0)

    let contrast = calculateContrast(textColor, backgroundColor)
    XCTAssertGreaterThanOrEqual(contrast, 4.5, "Contrast \(contrast) below 4.5:1")
}
```

### 4.2 Manual Contrast Testing

#### Browser DevTools (Screenshots)
```
1. Take screenshot of app UI
2. Upload to WebAIM Contrast Checker
3. Verify: Normal text ≥ 4.5:1, Large text ≥ 3:1
```

#### Tools
- **WebAIM Contrast Checker**: https://webaim.org/resources/contrastchecker/
- **Deque Accessibility Checker**: https://www.deque.com/axe/devtools/
- **Google Lighthouse**: https://developers.google.com/web/tools/lighthouse

### 4.3 Color Blindness Simulation

#### Android - Simulator Color Blindness
```
Android Studio > AVD Manager > [Device] > Edit
> Advanced Settings > OpenGL ES Renderer: [Enable] Color Blindness
```

#### Color Blindness Types Test
- [ ] Protanopia (red-blind): Cannot distinguish red/green
- [ ] Deuteranopia (green-blind): Cannot distinguish green/red
- [ ] Tritanopia (blue-blind): Cannot distinguish blue/yellow
- [ ] Achromatopsia (colorblind): No color vision

#### Test Strategy
```kotlin
// GOOD - Information not conveyed by color alone
Row {
    Icon(
        imageVector = Icons.Default.CheckCircle,
        contentDescription = "Complete",
        tint = Color(0xFF4CAF50)  // Green icon
    )
    Text("Completed")  // Also labeled with text
}

// BAD - Color only
Box(
    modifier = Modifier
        .size(24.dp)
        .background(Color(0xFF4CAF50))  // Green only, no label
)
```

---

## Phase 5: Text Scaling & Dynamic Type

### 5.1 Android - Font Scaling Test

#### Enable Large Font Size
```
Settings > Accessibility > Display > Font size
Choose: Large / Extra Large
```

#### Test Checklist
```kotlin
@Test
fun testFontScaling() {
    // Test with default font size
    composeTestRule.setContent { EventDetailScreen() }
    assertScreenLayoutCorrect()

    // TODO: Test with large font size (requires configuration)
    // Expected: Text enlarges, no clipping, layout adapts
}
```

#### Expected Behavior
- [ ] Text enlarges proportionally
- [ ] No text clipping or overflow
- [ ] Layout adapts (single column if needed)
- [ ] Buttons remain accessible (48x48dp)
- [ ] Line length stays readable (< 80 chars)

### 5.2 iOS - Dynamic Type Test

#### Enable Largest Accessibility Size
```
Settings > Accessibility > Display & Text Size
Choose: AX5, AX6, AX7 (Largest)
```

#### Test Strategy
```swift
@MainActor
func testDynamicType() {
    let app = XCUIApplication()

    // Add launch argument to simulate large font
    app.launchArguments = ["-com.apple.accessibility.accessibility-bundle", "true"]
    app.launch()

    // Verify text scales properly
    let titleLabel = app.staticTexts["screen_title"]
    XCTAssertTrue(titleLabel.exists)

    // Verify layout adapts
    let contentArea = app.otherElements["content_area"]
    XCTAssertTrue(contentArea.isAccessibilityElement)
}
```

#### Expected Behavior
- [ ] Text scales to AX7 (largest accessibility size)
- [ ] No text clipping
- [ ] Layout reflows to single column if needed
- [ ] Buttons remain accessible (44x44pt minimum)
- [ ] Line length stays readable

---

## Phase 6: Interactive Element Testing

### 6.1 Focus & Active States

#### Focus Indicator Requirements
- [ ] **Visible**: Clear outline or color change
- [ ] **Contrast**: 3:1 minimum against background
- [ ] **Size**: ≥ 3:1 outline width
- [ ] **Consistency**: Same style across all interactive elements

#### Android - Focus Indicator Test
```kotlin
@Test
fun testFocusIndicator() {
    composeTestRule.setContent {
        Button(onClick = { }) {
            Text("Submit")
        }
    }

    // Simulate keyboard focus
    composeTestRule.onNodeWithText("Submit").performKeyInput {
        keyDown(Key.Tab)
    }

    // Verify focus outline exists and has sufficient contrast
    composeTestRule.onNodeWithText("Submit")
        .assertIsDisplayed()
        // Visual verification required: outline should be visible
}
```

#### iOS - Focus Ring Test
```swift
@MainActor
func testFocusIndicator() {
    let app = XCUIApplication()
    app.launch()

    let button = app.buttons["submit_button"]
    button.tap()

    // Focus ring should be visible on keyboard navigation
    XCTAssertTrue(button.isFocused)
    // Visual verification: 2pt outline should appear
}
```

### 6.2 Disabled States

#### Disabled Element Requirements
- [ ] **Visual**: Clearly different from enabled state
- [ ] **Semantic**: Marked as disabled in accessibility tree
- [ ] **Contrast**: Can be 2:1 (lower for disabled)
- [ ] **Announcement**: Screen reader announces "disabled"

#### Android - Disabled State Test
```kotlin
@Test
fun testDisabledState() {
    composeTestRule.setContent {
        Button(
            onClick = { },
            enabled = false  // Disabled
        ) {
            Text("Submit")
        }
    }

    composeTestRule.onNodeWithText("Submit")
        .assert(isNotEnabled())
        .assertTextEquals("Submit")  // Text should be present
        // Visual check: Should appear grayed out (38% opacity)
}
```

#### iOS - Disabled State Test
```swift
@MainActor
func testDisabledState() {
    let app = XCUIApplication()
    app.launch()

    let button = app.buttons["submit_button"]
    XCTAssertFalse(button.isEnabled)
    // Visual check: Should appear grayed out (50% opacity)
}
```

---

## Phase 7: Error Handling & Feedback

### 7.1 Form Error Announcements

#### Test Case: Invalid Email
```kotlin
@Test
fun testFormErrorAnnouncement() {
    composeTestRule.setContent { SignupForm() }

    // Enter invalid email
    composeTestRule.onNodeWithTag("email_field")
        .performTextInput("invalid")

    // Verify error message appears and is announced
    composeTestRule.onNodeWithText("Invalid email format")
        .assertIsDisplayed()
        // TalkBack will announce: "Error: Invalid email format"
}
```

#### Test Case: Confirmation Dialog
```swift
@MainActor
func testConfirmationDialog() {
    let app = XCUIApplication()
    app.launch()

    // Perform destructive action
    app.buttons["delete_button"].tap()

    // Verify dialog appears
    let alertTitle = app.staticTexts["Delete this item?"]
    XCTAssertTrue(alertTitle.exists)

    // Verify VoiceOver announces options
    let cancelButton = app.buttons["Cancel"]
    let deleteButton = app.buttons["Delete"]
    XCTAssertTrue(cancelButton.exists)
    XCTAssertTrue(deleteButton.exists)
}
```

### 7.2 Status Message Announcements

#### Loading States
```kotlin
@Composable
fun ListScreen(viewModel: ListViewModel) {
    when (val state = viewModel.state.collectAsState().value) {
        is Loading -> {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .semantics {
                        // Announce loading to TalkBack
                        liveRegion = LiveRegionMode.Assertive
                        contentDescription = "Loading items..."
                    },
                contentAlignment = Alignment.Center
            ) {
                CircularProgressIndicator()
            }
        }
        is Success -> {
            // Display list
        }
        is Error -> {
            // Show error message with live region
        }
    }
}
```

#### Success Notifications
```swift
struct ListView: View {
    @State var items: [Item] = []
    @State var statusMessage: String?

    var body: some View {
        VStack {
            List(items) { item in
                ItemRow(item)
            }
        }
        .onChange(of: items.count) { oldValue, newValue in
            if newValue > oldValue {
                statusMessage = "Added \(newValue - oldValue) new items"
                UIAccessibility.post(
                    notification: .announcement,
                    argument: statusMessage
                )
            }
        }
    }
}
```

---

## Phase 8: Touch Target Testing

### 8.1 Minimum Target Size (48dp / 44pt)

#### Android - Touch Target Verification
```kotlin
@Test
fun testTouchTargets() {
    composeTestRule.setContent { LoginScreen() }

    // Get all clickable elements
    composeTestRule.onAllNodes(isClickable()).forEach { node ->
        val bounds = node.fetchSemanticsNode().boundsInRoot

        assert(bounds.width >= 48.dp && bounds.height >= 48.dp) {
            "Touch target too small: ${bounds.width}x${bounds.height} (min 48x48dp)"
        }
    }
}
```

#### iOS - Touch Target Verification
```swift
@MainActor
func testTouchTargets() {
    let app = XCUIApplication()
    app.launch()

    // Get all buttons
    for button in app.buttons.allElementsBoundByIndex {
        let frame = button.frame
        let width = frame.width
        let height = frame.height

        XCTAssertGreaterThanOrEqual(width, 44, "Button too narrow: \(width)pt")
        XCTAssertGreaterThanOrEqual(height, 44, "Button too short: \(height)pt")
    }
}
```

### 8.2 Target Spacing (8dp / 8pt minimum)

#### Test Strategy
```kotlin
// GOOD - 8dp spacing
Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
    Button(modifier = Modifier.size(48.dp)) { }
    Button(modifier = Modifier.size(48.dp)) { }
}

// BAD - 4dp spacing (targets may be too close)
Row(horizontalArrangement = Arrangement.spacedBy(4.dp)) {
    Button(modifier = Modifier.size(48.dp)) { }
    Button(modifier = Modifier.size(48.dp)) { }
}
```

---

## Phase 9: Test Report Template

### A11y Test Report

```markdown
# Accessibility Testing Report

## Project: [App Name]
## Date: [Date]
## Tester: [Name]

### Summary
- **Pass Rate**: 95% (19/20 criteria met)
- **Critical Issues**: 0
- **Serious Issues**: 1
- **Minor Issues**: 3

### Critical Issues (Must Fix)
None

### Serious Issues (Fix ASAP)
1. **Missing Touch Target Size on Search Button**
   - Component: SearchBar > Search button
   - Issue: 32x32dp (min 48x48dp)
   - Fix: Increase to 48x48dp using `minimumInteractiveComponentSize()`

### Minor Issues (Polish)
1. **Weak Contrast on Secondary Text**
   - Component: EventCard > Subtitle
   - Contrast: 3.2:1 (min 4.5:1)
   - Fix: Darker gray or larger font

2. **Missing Error Announcement**
   - Component: LoginForm > Error state
   - Issue: No live region announcement
   - Fix: Add `liveRegion = LiveRegionMode.Assertive`

3. **Focus Indicator Not Visible**
   - Component: All buttons
   - Issue: Focus outline barely visible
   - Fix: Increase border width from 1dp to 2dp

### Testing Details

#### Manual Testing
- [ ] TalkBack: Fully navigable, all labels present
- [ ] VoiceOver: Fully navigable, all labels present
- [ ] Keyboard: Tab navigation works, focus visible
- [ ] Contrast: All text meets 4.5:1 requirement
- [ ] Font Scaling: Layout adapts to 200% scale

#### Automated Testing
- [ ] Android Lint: 0 accessibility warnings
- [ ] Axe-Core Android: 0 violations
- [ ] iOS Xcode: 0 accessibility issues
- [ ] Axe-Core iOS: 0 violations

### Recommendations
1. Implement live region announcements for status changes
2. Increase focus indicator visibility across all interactive elements
3. Add touch target spacing validation to unit tests
4. Integrate automated accessibility testing into CI/CD pipeline
```

---

## Appendix: Testing Tools Reference

| Tool | Platform | Purpose | Link |
|------|----------|---------|------|
| **TalkBack** | Android | Screen reader | Built-in |
| **VoiceOver** | iOS | Screen reader | Built-in |
| **Axe-Core** | Both | Automated audits | https://github.com/dequelabs |
| **Xcode A11y Audit** | iOS | Built-in checker | Xcode > Product > Perform Action > Accessibility Audit |
| **Android Lint** | Android | Static analysis | `./gradlew lint` |
| **WAVE Browser** | Both | Web accessibility | https://wave.webaim.org/ |
| **Color Contrast** | Both | Contrast checker | https://webaim.org/resources/contrastchecker/ |
| **Lighthouse** | Both | Performance & a11y | https://developers.google.com/web/tools/lighthouse |


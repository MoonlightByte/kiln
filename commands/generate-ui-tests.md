---
name: generate-ui-tests
description: Generate comprehensive UI test suites for Compose and SwiftUI components with semantic tags and state coverage
args:
  - name: component_path
    type: string
    description: Path to component file to generate tests for (e.g., EventCard.kt, CustomButton.swift)
    required: true
  - name: platform
    type: string
    enum: [android, ios, both]
    description: Target platform for test generation (default: auto-detect from file extension)
    required: false
  - name: include_accessibility
    type: boolean
    description: Generate accessibility interaction tests (WCAG 2.2 Level AA coverage)
    required: false
  - name: include_semantic
    type: boolean
    description: Extract and generate semantic test tag definitions for E2E automation
    required: false
  - name: test_style
    type: string
    enum: [snapshot, interaction, comprehensive]
    description: Test generation style - snapshot (visual regression), interaction (state changes), or both
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# UI Test Generation for Mobile Components

Generate production-ready UI test suites for Compose (Android) and SwiftUI (iOS) components with comprehensive state coverage, accessibility validation, and semantic test tags for E2E automation.

## Overview

This command analyzes component structure and generates:

1. **Interaction Tests** - State changes, user interactions, callbacks
2. **Snapshot Tests** - Visual regression testing (Roborazzi/SnapshotTesting)
3. **Semantic Tags** - Element discovery for MCP UI automation tools
4. **Accessibility Tests** - WCAG 2.2 Level AA compliance
5. **Test Data Factories** - Deterministic test data builders
6. **State Variations** - Coverage for all component states (enabled, disabled, loading, error)

## Instructions

Analyze the provided component file and generate:

1. **Test Skeleton** - Structure with setup/teardown, test discovery patterns
2. **State Coverage Matrix** - All possible UI states and property combinations
3. **Interaction Flows** - User gestures, callbacks, event handling
4. **Semantic Analysis** - Test tags and accessibility identifiers
5. **Accessibility Suite** - Touch targets, contrast, focus, announcements
6. **Output Templates** - Copy-paste-ready test code for both platforms
7. **CI/CD Checklist** - Integration points for GitHub Actions

## Test Generation Checklist

### Component Analysis (Critical Priority)

- [ ] **TG-001**: Component structure analyzed (Composable/View hierarchy)
- [ ] **TG-002**: All state properties identified (data, user interactions, loading)
- [ ] **TG-003**: Callbacks and event handlers enumerated
- [ ] **TG-004**: Slot/content parameters mapped
- [ ] **TG-005**: Default parameter values documented

### Test Skeleton Setup (Critical Priority)

**Android (Jetpack Compose):**
- [ ] **TG-010**: ComposeTestRule configured with proper composition
- [ ] **TG-011**: Test class extends BaseComposeTest or uses @RunWith(RobolectricTestRunner)
- [ ] **TG-012**: @get:Rule decorator applied to test rule
- [ ] **TG-013**: setUp() and tearDown() methods defined
- [ ] **TG-014**: Theme wrapped around component under test

**iOS (SwiftUI):**
- [ ] **TG-020**: XCTestCase subclass created
- [ ] **TG-021**: Accessibility Inspector configured for testing
- [ ] **TG-022**: XCUIApplication launched in setUp
- [ ] **TG-023**: Accessibility identifiers resolvable in tests
- [ ] **TG-024**: Device orientation tests included

### State Variation Coverage (Important Priority)

- [ ] **TG-030**: Default state tested
- [ ] **TG-031**: Enabled/disabled states tested
- [ ] **TG-032**: Loading state tested (spinners, skeletons)
- [ ] **TG-033**: Error state tested (error messages, retry)
- [ ] **TG-034**: Empty state tested (no data)
- [ ] **TG-035**: Dark mode variant tested
- [ ] **TG-036**: Large text (accessibility) tested
- [ ] **TG-037**: Landscape orientation tested (mobile)

### Interaction Tests (Important Priority)

**Jetpack Compose:**
- [ ] **TG-040**: performClick() on clickable elements
- [ ] **TG-041**: performTextInput() for text fields
- [ ] **TG-042**: performScroll() for scrollable content
- [ ] **TG-043**: performSwipe() for swipe gestures
- [ ] **TG-044**: Callback verification with mockito/fake implementations
- [ ] **TG-045**: State updates after user actions

**SwiftUI:**
- [ ] **TG-050**: tap() on buttons and interactive elements
- [ ] **TG-051**: typeText() for text input
- [ ] **TG-052**: swipeUp/Down/Left/Right for gestures
- [ ] **TG-053**: XCTAssertTrue/False for state verification
- [ ] **TG-054**: waitForElement() for async operations
- [ ] **TG-055**: Closure/callback verification

### Semantic Test Tags (Important Priority)

**Android (testTag modifier):**
- [ ] **TG-060**: All clickable elements have testTag
- [ ] **TG-061**: Text fields have testTag for input
- [ ] **TG-062**: Containers have parent testTag for hierarchy
- [ ] **TG-063**: Test tags follow naming: `component_element_action` (e.g., `event_card_join_button`)
- [ ] **TG-064**: Tags resolvable via `onNodeWithTag("tag")`
- [ ] **TG-065**: Semantic tree provides parent context

**iOS (accessibilityIdentifier):**
- [ ] **TG-070**: All interactive elements have accessibilityIdentifier
- [ ] **TG-071**: Identifiers follow naming: `component_element_action`
- [ ] **TG-072**: Identifiers resolvable via `.descendants(matching: .any)[id]`
- [ ] **TG-073**: Custom containers have accessibility roles assigned
- [ ] **TG-074**: Accessibility labels don't contain redundant technical terms

### Snapshot/Visual Regression Tests (Important Priority)

**Compose (Roborazzi):**
- [ ] **TG-080**: Screenshot test for default state
- [ ] **TG-081**: Screenshot test for dark mode
- [ ] **TG-082**: Screenshot test for accessibility (large text)
- [ ] **TG-083**: Golden files committed to repo
- [ ] **TG-084**: Test naming: `test_ComponentName_state`

**SwiftUI (SnapshotTesting):**
- [ ] **TG-090**: assertSnapshot() for light mode
- [ ] **TG-091**: assertSnapshot() for dark mode (.dark scheme)
- [ ] **TG-092**: assertSnapshot() for large size category
- [ ] **TG-093**: Multiple device previews (iPhone, iPad)
- [ ] **TG-094**: Reference snapshots stored in `__Snapshots__/`

### Accessibility Tests (Important Priority)

- [ ] **TG-100**: Touch target size verified (24×24 minimum, 48×48 recommended)
- [ ] **TG-101**: Color contrast checked (4.5:1 normal text, 3:1 large text)
- [ ] **TG-102**: Content descriptions present on all images/icons
- [ ] **TG-103**: Focus indicators visible and properly contrasted
- [ ] **TG-104**: Text resizes at 200% without overflow
- [ ] **TG-105**: VoiceOver/TalkBack announcements tested
- [ ] **TG-106**: No decorative elements announced to screen readers

### Test Data Factories (Important Priority)

- [ ] **TG-110**: Factory pattern for test data creation
- [ ] **TG-111**: Builder methods for state variations
- [ ] **TG-112**: Deterministic fake data (no randomness)
- [ ] **TG-113**: Default values reflect production defaults
- [ ] **TG-114**: Complex nested objects handled
- [ ] **TG-115**: Optional/nullable fields tested

### CI/CD Integration (Important Priority)

- [ ] **TG-120**: Test class discoverable by test runner
- [ ] **TG-121**: Tests run in CI pipeline (GitHub Actions)
- [ ] **TG-122**: Screenshots/snapshots uploaded as artifacts
- [ ] **TG-123**: Snapshot diffs visible in PR checks
- [ ] **TG-124**: Test naming follows convention (test_*)
- [ ] **TG-125**: Timeout handling for async operations

## Component Analysis Template

```markdown
## Component Structure Analysis: [ComponentName]

### State Properties
- Property name: Type (default value)
- Example: `title: String = ""`
- Example: `isLoading: Boolean = false`
- Example: `onSubmit: () -> Unit = {}`

### User Interactions
- Click/tap on [element]
- Text input in [field]
- Swipe/scroll on [container]

### Slot/Content Parameters
- If Compose: @Composable content slots
- If SwiftUI: View builder closures
- If neither: fixed content

### Visual States
- Default appearance
- Disabled (grayed out, not clickable)
- Loading (spinner, skeleton)
- Error (error text, red border)
- Empty (placeholder message)
- Dark mode variant
```

## Test Skeleton Generation

### Jetpack Compose Template

```kotlin
package com.partyonapp.android.ui.components

import androidx.compose.ui.test.*
import androidx.compose.ui.test.junit4.ComposeContentTestRule
import androidx.compose.ui.test.junit4.createComposeRule
import androidx.test.ext.junit.runners.AndroidJUnit4
import org.junit.Before
import org.junit.Rule
import org.junit.Test
import org.junit.runner.RunWith

@RunWith(AndroidJUnit4::class)
class [ComponentName]Test {
    @get:Rule
    val composeTestRule = createComposeRule()

    @Before
    fun setUp() {
        // Setup theme, test data, mocks
    }

    // Tests organized by category:
    // 1. Rendering Tests (verifies UI displays correctly)
    // 2. Interaction Tests (user events)
    // 3. State Change Tests (verify internal state updates)
    // 4. Accessibility Tests (WCAG 2.2 compliance)
    // 5. Edge Cases (empty data, errors)
}
```

**Key Setup Points:**
- Use `createComposeRule()` for modern Compose testing
- Wrap component in `composeTestRule.setContent { AppTheme { ... } }`
- Use `onNode()` with `hasTestTag()` for element selection
- Mock callbacks with `mockito.mock()` or `FakeLambda` classes
- For snapshots, add Roborazzi dependency to build.gradle

### SwiftUI Template

```swift
import XCTest
@testable import PartyOn

final class [ComponentName]Tests: XCTestCase {

    var app: XCUIApplication!

    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app = XCUIApplication()
        app.launch()

        // Navigate to component or present in isolation
    }

    override func tearDown() {
        super.tearDown()
        app.terminate()
    }

    // Tests organized by category (same as Compose)
    // Use .descendants(matching:) to find by accessibility ID
    // Use .isHittable to verify touch targets
    // Use waitForElement() for animations/async
}
```

**Key Setup Points:**
- Use `XCUIApplication()` for full app testing
- Define constants for accessibility IDs in shared file
- Use `XCTWaiter()` instead of `sleep()` for deterministic waits
- For snapshots, add `swift-snapshot-testing` SPM dependency
- Test on multiple device sizes (iPhone 14, 15 Pro, iPad)

## State Variation Coverage Matrix

```markdown
### State Matrix for [ComponentName]

| State | Data | Expected UI | Test Case |
|-------|------|-------------|-----------|
| Default | title="Hello" | Text displayed | test_default_rendering |
| Disabled | enabled=false | Grayed out, not clickable | test_disabled_state |
| Loading | isLoading=true | Spinner shown | test_loading_state |
| Error | error="Failed" | Error message shown | test_error_state |
| Empty | items=[] | Empty placeholder | test_empty_state |
| Dark Mode | isDark=true | Colors adjusted | test_dark_mode |
| Large Text | contentScale=2.0 | Text scales correctly | test_large_text_a11y |
| Landscape | orientation=landscape | Layout reflows | test_landscape_orientation |

### Combination Tests (if multiple independent states)
- Loading + Error (show error if loading fails)
- Disabled + Dark (verify disabled contrast in dark mode)
- Empty + Dark (empty message readable in dark mode)
```

## Semantic Test Tag Generation

### Android (Compose) Pattern

```kotlin
// Define tags in your component file or separate file:
object TestTags {
    // Button identifiers
    const val EVENT_CARD_JOIN_BUTTON = "event_card_join_button"
    const val EVENT_CARD_MENU_BUTTON = "event_card_menu_button"

    // Text field identifiers
    const val EVENT_TITLE_INPUT = "event_title_input"
    const val EVENT_DESCRIPTION_INPUT = "event_description_input"

    // Container identifiers
    const val EVENT_CARD = "event_card"
    const val EVENT_HEADER = "event_card_header"

    // Semantic markers
    const val LOADING_SPINNER = "loading_spinner"
    const val ERROR_MESSAGE = "error_message"
}

// In component:
@Composable
fun EventCard(
    event: Event,
    onJoin: () -> Unit,
    modifier: Modifier = Modifier
) {
    Surface(modifier = modifier.testTag(TestTags.EVENT_CARD)) {
        Column {
            Header(event, modifier = Modifier.testTag(TestTags.EVENT_HEADER))

            Button(
                onClick = onJoin,
                modifier = Modifier.testTag(TestTags.EVENT_CARD_JOIN_BUTTON)
            ) {
                Text("Join")
            }

            if (isLoading) {
                CircularProgressIndicator(modifier = Modifier.testTag(TestTags.LOADING_SPINNER))
            }
        }
    }
}

// In test:
@Test
fun test_joinButton_clickable() {
    composeTestRule.onNodeWithTag(TestTags.EVENT_CARD_JOIN_BUTTON)
        .assertIsDisplayed()
        .performClick()

    // Verify callback was called
}
```

**Naming Convention:**
- Format: `component_element_action`
- Examples:
  - `event_card_join_button` (button in event card)
  - `event_title_input` (input field for event title)
  - `event_card` (root container)
  - `loading_spinner` (shared across components)

### iOS (SwiftUI) Pattern

```swift
// Define identifiers in shared enum:
enum AccessibilityIds {
    static let eventCard = "event_card"
    static let eventCardJoinButton = "event_card_join_button"
    static let eventCardMenuButton = "event_card_menu_button"
    static let eventTitleInput = "event_title_input"
    static let eventDescriptionInput = "event_description_input"
    static let loadingSpinner = "loading_spinner"
    static let errorMessage = "error_message"
}

// In component:
struct EventCard: View {
    let event: Event
    let onJoin: () -> Void

    var body: some View {
        VStack {
            EventHeader(event)
                .accessibilityIdentifier(AccessibilityIds.eventCard)

            Button(action: onJoin) {
                Text("Join")
            }
            .accessibilityIdentifier(AccessibilityIds.eventCardJoinButton)

            if isLoading {
                ProgressView()
                    .accessibilityIdentifier(AccessibilityIds.loadingSpinner)
            }
        }
    }
}

// In test:
func test_joinButton_clickable() {
    let app = XCUIApplication()
    app.launch()

    let joinButton = app.buttons[AccessibilityIds.eventCardJoinButton]
    XCTAssertTrue(joinButton.isHittable)
    joinButton.tap()

    // Verify state change
    XCTAssertFalse(app.buttons[AccessibilityIds.loadingSpinner].exists)
}
```

**Naming Convention:** Same as Android (`component_element_action`)

## Test Data Factory Examples

### Compose Factory Pattern

```kotlin
// File: EventTestDataFactory.kt
object EventTestDataFactory {
    fun createDefaultEvent(): Event = Event(
        id = "test-event-1",
        title = "D&D Campaign",
        description = "Weekly campaign session",
        date = "2025-02-26T19:00:00Z",
        spotsLeft = 5,
        isLoading = false,
        error = null
    )

    fun createLoadingEvent(): Event = createDefaultEvent().copy(
        isLoading = true
    )

    fun createErrorEvent(message: String = "Failed to load"): Event =
        createDefaultEvent().copy(
            error = message,
            isLoading = false
        )

    fun createEmptyEvent(): Event = createDefaultEvent().copy(
        spotsLeft = 0
    )

    fun createBuilder(): EventBuilder = EventBuilder()
}

// Builder for complex setups:
class EventBuilder {
    private var id = "test-event-1"
    private var title = "D&D Campaign"
    private var isLoading = false
    private var error: String? = null

    fun withId(id: String) = apply { this.id = id }
    fun withTitle(title: String) = apply { this.title = title }
    fun loading() = apply { this.isLoading = true }
    fun withError(message: String) = apply { this.error = message }

    fun build(): Event = Event(id, title, isLoading = isLoading, error = error)
}

// In test:
@Test
fun test_errorState() {
    val errorEvent = EventTestDataFactory.createErrorEvent("Network failed")

    composeTestRule.setContent {
        AppTheme {
            EventCard(event = errorEvent, onJoin = {})
        }
    }

    composeTestRule.onNodeWithText("Network failed").assertIsDisplayed()
}
```

### SwiftUI Factory Pattern

```swift
// File: EventTestDataFactory.swift
enum EventTestDataFactory {
    static func createDefaultEvent() -> Event {
        Event(
            id: "test-event-1",
            title: "D&D Campaign",
            description: "Weekly session",
            date: Date(),
            spotsLeft: 5,
            isLoading: false,
            error: nil
        )
    }

    static func createLoadingEvent() -> Event {
        var event = createDefaultEvent()
        event.isLoading = true
        return event
    }

    static func createErrorEvent(_ message: String = "Failed") -> Event {
        var event = createDefaultEvent()
        event.error = message
        event.isLoading = false
        return event
    }

    static func createEmptyEvent() -> Event {
        var event = createDefaultEvent()
        event.spotsLeft = 0
        return event
    }
}

// In test:
func test_errorState() {
    let errorEvent = EventTestDataFactory.createErrorEvent("Network failed")

    let view = EventCard(event: errorEvent, onJoin: {})

    // Assert error message visible
    XCTAssertTrue(
        view.children.contains { $0 is Text } // Verify error text present
    )
}
```

## Interaction Test Examples

### Compose Interaction Tests

```kotlin
@Test
fun test_joinButton_clickable_triggersCallback() {
    var callbackInvoked = false
    val event = EventTestDataFactory.createDefaultEvent()

    composeTestRule.setContent {
        AppTheme {
            EventCard(
                event = event,
                onJoin = { callbackInvoked = true }
            )
        }
    }

    // Find and click button
    composeTestRule
        .onNodeWithTag(TestTags.EVENT_CARD_JOIN_BUTTON)
        .assertIsDisplayed()
        .assertIsEnabled()
        .performClick()

    // Verify callback was invoked
    assertTrue(callbackInvoked)
}

@Test
fun test_scrollContent_reachesEnd() {
    val longEvent = EventTestDataFactory.createDefaultEvent().copy(
        description = "Lorem ipsum dolor sit amet. ".repeat(50)
    )

    composeTestRule.setContent {
        AppTheme {
            EventCard(event = longEvent, onJoin = {})
        }
    }

    // Scroll to end
    composeTestRule
        .onNodeWithTag(TestTags.EVENT_CARD)
        .performScrollToIndex(100)

    // Verify end of content visible
}

@Test
fun test_disabledState_notClickable() {
    val disabledEvent = EventTestDataFactory.createDefaultEvent().copy(
        spotsLeft = 0  // No spots, so button disabled
    )

    composeTestRule.setContent {
        AppTheme {
            EventCard(event = disabledEvent, onJoin = {})
        }
    }

    composeTestRule
        .onNodeWithTag(TestTags.EVENT_CARD_JOIN_BUTTON)
        .assertIsNotEnabled()  // Button should be disabled
        .performClick()  // This should fail gracefully
}
```

### SwiftUI Interaction Tests

```swift
func test_joinButton_clickable_triggersCallback() {
    let app = XCUIApplication()
    app.launch()

    let joinButton = app.buttons[AccessibilityIds.eventCardJoinButton]

    XCTAssertTrue(joinButton.exists)
    XCTAssertTrue(joinButton.isHittable)

    joinButton.tap()

    // Verify callback effect (e.g., state change, navigation)
    let successMessage = app.staticTexts["Successfully joined"]
    XCTAssertTrue(successMessage.waitForExistence(timeout: 2))
}

func test_scrollContent_reachesEnd() {
    let app = XCUIApplication()
    app.launch()

    let scrollView = app.scrollViews.firstMatch

    // Scroll to bottom
    scrollView.swipeUp()

    let endMarker = app.staticTexts["End of content"]
    XCTAssertTrue(endMarker.exists)
}

func test_disabledState_notClickable() {
    let app = XCUIApplication()
    app.launch()

    let disabledButton = app.buttons[AccessibilityIds.eventCardJoinButton]

    XCTAssertFalse(disabledButton.isHittable, "Button should be disabled")
}
```

## Snapshot Testing Setup

### Compose Snapshot Tests (Roborazzi)

```kotlin
// build.gradle.kts dependencies:
dependencies {
    testImplementation("com.github.takahirom.roborazzi:roborazzi:x.x.x")
    testImplementation("com.github.takahirom.roborazzi:roborazzi-compose:x.x.x")
}

// Test code:
@RunWith(RobolectricTestRunner::class)
class EventCardScreenshotTest {
    @get:Rule
    val roborazziRule = RoborazziRule()

    @Test
    fun test_eventCard_default() {
        roborazziRule.captureRoboImage("EventCard/default.png") {
            AppTheme(useDarkTheme = false) {
                EventCard(
                    event = EventTestDataFactory.createDefaultEvent(),
                    onJoin = {},
                    modifier = Modifier.wrapContentSize()
                )
            }
        }
    }

    @Test
    fun test_eventCard_darkMode() {
        roborazziRule.captureRoboImage("EventCard/dark_mode.png") {
            AppTheme(useDarkTheme = true) {
                EventCard(
                    event = EventTestDataFactory.createDefaultEvent(),
                    onJoin = {},
                    modifier = Modifier.wrapContentSize()
                )
            }
        }
    }

    @Test
    fun test_eventCard_loadingState() {
        roborazziRule.captureRoboImage("EventCard/loading.png") {
            AppTheme {
                EventCard(
                    event = EventTestDataFactory.createLoadingEvent(),
                    onJoin = {},
                    modifier = Modifier.wrapContentSize()
                )
            }
        }
    }

    @Test
    fun test_eventCard_errorState() {
        roborazziRule.captureRoboImage("EventCard/error.png") {
            AppTheme {
                EventCard(
                    event = EventTestDataFactory.createErrorEvent("Connection failed"),
                    onJoin = {},
                    modifier = Modifier.wrapContentSize()
                )
            }
        }
    }
}
```

**Workflow:**
1. First run: Generates `.png` files in `src/test/snapshots/`
2. Review images for correctness
3. Commit `.png` files to version control
4. Future runs: Compare new images against golden copies
5. On changes: Approve updates or investigate regressions

### SwiftUI Snapshot Tests (SnapshotTesting)

```swift
// Package.swift dependencies:
.package(url: "https://github.com/pointfreeco/swift-snapshot-testing.git", from: "1.15.0")

// Test code:
import SnapshotTesting

final class EventCardSnapshotTests: XCTestCase {

    func test_eventCard_default() {
        let view = EventCard(
            event: EventTestDataFactory.createDefaultEvent(),
            onJoin: {}
        )

        assertSnapshot(matching: view, as: .image)
    }

    func test_eventCard_darkMode() {
        let view = EventCard(
            event: EventTestDataFactory.createDefaultEvent(),
            onJoin: {}
        )
        .preferredColorScheme(.dark)

        assertSnapshot(matching: view, as: .image)
    }

    func test_eventCard_largeText() {
        let view = EventCard(
            event: EventTestDataFactory.createDefaultEvent(),
            onJoin: {}
        )
        .environment(\.sizeCategory, .extraExtraLarge)

        assertSnapshot(matching: view, as: .image)
    }

    func test_eventCard_loadingState() {
        let view = EventCard(
            event: EventTestDataFactory.createLoadingEvent(),
            onJoin: {}
        )

        assertSnapshot(matching: view, as: .image)
    }

    func test_eventCard_errorState() {
        let view = EventCard(
            event: EventTestDataFactory.createErrorEvent("Connection failed"),
            onJoin: {}
        )

        assertSnapshot(matching: view, as: .image)
    }
}
```

**Workflow:** Same as Roborazzi (golden images stored and compared)

## Accessibility Test Examples

```kotlin
// Android Compose Accessibility Tests
@Test
fun test_touchTarget_joinButton_sufficientSize() {
    val event = EventTestDataFactory.createDefaultEvent()

    composeTestRule.setContent {
        AppTheme {
            EventCard(event = event, onJoin = {})
        }
    }

    composeTestRule
        .onNodeWithTag(TestTags.EVENT_CARD_JOIN_BUTTON)
        .assertWidthIsAtLeast(48.dp)
        .assertHeightIsAtLeast(48.dp)
}

@Test
fun test_contentDescription_presentOnIcon() {
    val event = EventTestDataFactory.createDefaultEvent()

    composeTestRule.setContent {
        AppTheme {
            EventCard(event = event, onJoin = {})
        }
    }

    composeTestRule
        .onNode(hasContentDescription() and hasText("Join"))
        .assertExists()
}

@Test
fun test_textSize_scales_withA11ySettings() {
    val event = EventTestDataFactory.createDefaultEvent()

    // Test at 200% text size
    composeTestRule.setContent {
        AppTheme(fontScale = 2.0f) {
            EventCard(event = event, onJoin = {})
        }
    }

    // Verify text is still readable (no truncation)
    composeTestRule
        .onNodeWithText(event.title)
        .assertIsDisplayed()
}
```

```swift
// iOS SwiftUI Accessibility Tests
func test_touchTarget_joinButton_sufficientSize() {
    let view = EventCard(
        event: EventTestDataFactory.createDefaultEvent(),
        onJoin: {}
    )

    let button = view
        .descendants(matching: .button)
        .matching(identifier: AccessibilityIds.eventCardJoinButton)
        .firstMatch

    XCTAssertTrue(button.frame.height >= 44) // Minimum iOS touch target
    XCTAssertTrue(button.frame.width >= 44)
}

func test_accessibilityLabel_present() {
    let view = EventCard(
        event: EventTestDataFactory.createDefaultEvent(),
        onJoin: {}
    )

    let joinButton = view.buttons[AccessibilityIds.eventCardJoinButton]

    XCTAssertNotNil(joinButton.label)
    XCTAssertFalse(joinButton.label.isEmpty)
}

func test_voiceOver_announces_buttonCorrectly() {
    let app = XCUIApplication()
    app.launch()

    // Enable VoiceOver
    app.launchArguments = ["-com.apple.accessibility.bundle", "VoiceOver"]

    let joinButton = app.buttons[AccessibilityIds.eventCardJoinButton]

    // Verify button has proper accessibility traits
    XCTAssertTrue(joinButton.isAccessibilityElement)
}
```

## CI/CD Integration Checklist

### GitHub Actions Integration

```yaml
name: UI Tests

on: [push, pull_request]

jobs:
  android-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Android SDK
        uses: android-actions/setup-android@v2

      - name: Run Compose UI Tests
        run: ./gradlew testDebug

      - name: Run Screenshot Tests (Roborazzi)
        run: ./gradlew testDebugRoborazzi

      - name: Upload snapshots as artifact
        uses: actions/upload-artifact@v3
        with:
          name: roborazzi-snapshots
          path: app/build/outputs/roborazzi/
          if-no-files-found: ignore

  ios-tests:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run SwiftUI Tests
        run: |
          xcodebuild test \
            -scheme PartyOn \
            -configuration Debug \
            -derivedDataPath build

      - name: Upload snapshots as artifact
        uses: actions/upload-artifact@v3
        with:
          name: snapshot-tests
          path: PartyOnTests/__Snapshots__/
          if-no-files-found: ignore
```

### Local Test Execution

**Android:**
```bash
# Run all UI tests
./gradlew testDebug

# Run specific test class
./gradlew testDebug --tests "com.partyonapp.android.ui.components.EventCardTest"

# Generate Roborazzi snapshots
./gradlew testDebugRoborazzi

# View coverage
./gradlew jacocoTestReport
```

**iOS:**
```bash
# Run all UI tests
xcodebuild test -scheme PartyOn -configuration Debug

# Run specific test
xcodebuild test -scheme PartyOn -test-configurationsDebug \
  PartyOnTests/EventCardSnapshotTests/test_eventCard_default

# Update snapshots
xcodebuild test -scheme PartyOn -testPlan SnapshotUpdate
```

## Common Pitfalls & Best Practices

### DO:

- **Use deterministic test data** - No random values, timestamps must be fixed
- **Test one thing per test** - Single responsibility for clear failures
- **Use semantic tags** - Avoid hardcoded coordinates or indices
- **Test accessibility** - WCAG 2.2 Level AA isn't optional
- **Snapshot on multiple states** - Default, loading, error, dark mode, a11y sizes
- **Mock external dependencies** - Network calls, analytics, device sensors
- **Keep tests fast** - Unit tests < 100ms, UI tests < 1s per test
- **Commit snapshots** - Golden images are part of version control
- **Test on real devices/simulators** - CI must run on actual hardware simulators

### DON'T:

- Don't use `sleep()` or fixed delays - Use `waitForElement()` or `composeTestRule.waitUntil()`
- Don't hardcode screen coordinates - Use accessibility IDs and semantic tags
- Don't mix business logic with UI tests - Keep tests focused on UI behavior
- Don't test third-party libraries - Mock them instead
- Don't ignore accessibility - It's part of quality, not optional
- Don't skip error states - Test network failures, validation errors, edge cases
- Don't snapshot flaky content - Animations, async operations must be deterministic
- Don't create brittle tests - Tests should survive UI refactors (semantic tags help)
- Don't test implementation details - Focus on observable behavior

## Output Format

```markdown
## UI Test Suite Generation: [ComponentName]

### Component Analysis Summary
- States identified: X (default, loading, error, disabled, dark mode, a11y)
- Interactions: X (clicks, scrolls, text input)
- Callbacks: X (onJoin, onDismiss, etc.)

### Test Structure
**Location:**
- Android: `android/app/src/androidTest/java/com/partyonapp/android/ui/components/[ComponentName]Test.kt`
- iOS: `ios/PartyOnTests/Components/[ComponentName]Tests.swift`

**Test Classes:**
- Interaction Tests: `[ComponentName]InteractionTest`
- Snapshot Tests: `[ComponentName]SnapshotTest`
- Accessibility Tests: `[ComponentName]AccessibilityTest`

### Semantic Tags Defined
| Tag Name | Element | Purpose |
|----------|---------|---------|
| component_element_action | Button | Click trigger |
| component_input_field | TextField | Text entry |
| component_loading_spinner | Progress | Loading state |

### State Coverage
- [ ] Default state
- [ ] Enabled state
- [ ] Disabled state
- [ ] Loading state
- [ ] Error state
- [ ] Empty state
- [ ] Dark mode
- [ ] Large text (a11y)
- [ ] Landscape orientation

### Accessibility Compliance
| Criterion | Status | Details |
|-----------|--------|---------|
| WCAG 1.1.1 (Text Alt) | ✅ | All icons have descriptions |
| WCAG 2.5.8 (Touch Size) | ✅ | 48×48dp minimum |
| WCAG 1.4.3 (Contrast) | ✅ | 4.5:1 ratio verified |
| WCAG 2.4.7 (Focus) | ✅ | Focus indicators visible |

### Generated Test Files

**Android (Compose):**
1. `[ComponentName]Test.kt` - Interaction tests (XX test methods)
2. `[ComponentName]SnapshotTest.kt` - Visual regression (XX snapshots)
3. `[ComponentName]AccessibilityTest.kt` - A11y compliance (XX tests)

**iOS (SwiftUI):**
1. `[ComponentName]Tests.swift` - Interaction tests (XX test methods)
2. `[ComponentName]SnapshotTests.swift` - Visual regression (XX snapshots)
3. `[ComponentName]AccessibilityTests.swift` - A11y compliance (XX tests)

### Test Data Factories
- `[ComponentName]TestDataFactory` - Factory with builder pattern
- `create Default[Component]()` - Standard test object
- `create[State][Component]()` - State variants

### CI/CD Integration
- ✅ Tests discoverable in GitHub Actions
- ✅ Artifacts: Screenshots/snapshots uploaded
- ✅ Timeout: X seconds per test
- ✅ Platforms: Android + iOS (if applicable)

### Next Steps
1. Copy generated test files to appropriate directories
2. Implement mocked callbacks/dependencies
3. Run tests locally: `./gradlew testDebug` or `xcodebuild test`
4. Commit test code and snapshot files
5. Verify CI pipeline runs tests on PR

### Maintenance Notes
- Update tests if component props change
- Regenerate snapshots if visual changes are intentional
- Add new state variations as feature requirements evolve
- Keep semantic tags synchronized with component changes
```

## References

This command synthesizes best practices from:

- [Jetpack Compose Testing Guide](https://developer.android.com/develop/ui/compose/testing) - Official Android testing patterns
- [XCUITest Fundamentals](https://developer.apple.com/documentation/xctest) - Apple's XCTest framework
- [Semantic UI Testing](https://infinum.com/handbook/devproc/test-automation/mobile-element-ids) - Infinum mobile test automation handbook
- [Roborazzi Screenshot Testing](https://github.com/takahirom/roborazzi) - Compose visual regression
- [SnapshotTesting for SwiftUI](https://github.com/pointfreeco/swift-snapshot-testing) - Point-Free Snapshot Testing library
- [WCAG 2.2 Mobile Testing](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html) - Web Accessibility Guidelines
- [Compose Testing Best Practices (2025)](https://medium.com/@anandgaur2207/jetpack-compose-chapter-10-testing-in-compose-7a8dce1db157)
- [XCUITest with SwiftUI (2025)](https://www.swiftyplace.com/blog/xcuitest-ui-testing-swiftui)

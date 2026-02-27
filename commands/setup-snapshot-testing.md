---
name: setup-snapshot-testing
description: Comprehensive setup and configuration for mobile snapshot testing (Roborazzi/Compose + swift-snapshot-testing/SwiftUI)
args:
  - name: platform
    type: string
    enum: [android, ios, both]
    description: Target platform for snapshot testing setup
    required: true
  - name: devices
    type: string
    enum: [phone, tablet, foldable, all]
    description: Device types to generate baselines for (comma-separated)
    required: false
  - name: dark_mode
    type: boolean
    description: Include dark mode variant baselines (default: true)
    required: false
  - name: output_ci
    type: boolean
    description: Generate GitHub Actions workflow configuration (default: true)
    required: false
  - name: diff_threshold
    type: number
    description: Pixel-level diff threshold before marking test as failed (0.0-1.0, default: 0.01 = 1%)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Mobile Snapshot Testing Setup & Configuration

Comprehensive guide for implementing visual regression testing on iOS and Android using industry-standard tools: **Roborazzi** (Compose) and **swift-snapshot-testing** (SwiftUI).

## Overview

Snapshot testing captures visual output and compares against baseline images to detect unintended UI changes. This guide covers:

- **Android (Jetpack Compose)**: Roborazzi with Robolectric (JVM-based, fast CI)
- **iOS (SwiftUI)**: swift-snapshot-testing with Prefire integration
- **Multi-device testing**: Phone, tablet, foldable device matrices
- **Dark mode variants**: Automated baseline generation for light/dark themes
- **CI/CD integration**: GitHub Actions workflows with artifact management
- **Baseline management**: Approval workflow for new/updated baselines
- **Diff visualization**: Side-by-side comparisons with threshold configuration

## Tools & Latest Standards (2025)

### Roborazzi (Android)
- **Repository**: [takahirom/roborazzi](https://github.com/takahirom/roborazzi)
- **Key Features**:
  - Runs on JVM with Robolectric (no emulator required)
  - Compose Preview scanner auto-generates tests
  - Fast CI/CD execution (2-3 sec per snapshot)
  - Supports interaction testing (click, scroll, state changes)
  - Flexible diff thresholds and filtering

### swift-snapshot-testing (iOS)
- **Repository**: [pointfreeco/swift-snapshot-testing](https://github.com/pointfreeco/swift-snapshot-testing)
- **Key Features**:
  - Multi-format support (images, PDFs, View hierarchies)
  - SwiftUI native integration
  - Prefire plugin auto-generates tests from Previews
  - Fast compilation and execution
  - Custom precision/diff strategies

### Prefire (iOS)
- **Purpose**: Auto-generates snapshot tests from SwiftUI Previews at build time
- **Integration**: Works seamlessly with swift-snapshot-testing
- **Benefit**: No test duplication - Previews → Tests automatically

---

## Part 1: Android Setup (Jetpack Compose + Roborazzi)

### 1.1 Gradle Configuration

#### Add Roborazzi to build.gradle.kts (app module)

```kotlin
plugins {
    // ... existing plugins
    id("io.github.takahirom.roborazzi") version "1.15.0"
}

dependencies {
    // Roborazzi core
    testImplementation("io.github.takahirom.roborazzi:roborazzi:1.15.0")
    testImplementation("io.github.takahirom.roborazzi:roborazzi-compose:1.15.0")
    testImplementation("io.github.takahirom.roborazzi:roborazzi-junit4:1.15.0")

    // Robolectric runtime
    testImplementation("org.robolectric:robolectric:4.13")

    // Compose test utilities
    testImplementation("androidx.compose.ui:ui-test-junit4")
    testImplementation("androidx.compose.ui:ui-test-manifest")

    // JUnit 4
    testImplementation("junit:junit:4.13.2")
}

roborazzi {
    // Enable Compose Preview test generation
    generateComposePreviewRobolectricTests {
        enable = true
        packages = listOf(
            "com.partyonapp.android.ui.screens",
            "com.partyonapp.android.ui.components"
        )
    }

    // Diff configuration
    compareOptions {
        resultBehavior = io.github.takahirom.roborazzi.RobolectricDeviceQualifier.PRINT_DIFF_ON_FAILURE
        thresholdPixelCount = 100  // Allow up to 100 pixels difference
        thresholdPercentage = 1.0f  // Or 1% threshold
    }
}
```

#### Enable Snapshot Testing in build.gradle.kts (root)

```kotlin
tasks.register("updateRoborazziSnapshots") {
    doLast {
        println("Run: ./gradlew updateRoborazziDebugSnapshots")
    }
}
```

### 1.2 Directory Structure

```
android/app/src/
├── main/
│   ├── java/com/partyonapp/android/
│   │   ├── ui/
│   │   │   ├── screens/
│   │   │   │   ├── EventDetailScreen.kt
│   │   │   │   └── ...@Preview...
│   │   │   └── components/
│   │   │       ├── EventCard.kt
│   │   │       └── ...
│   │   └── ...
│   └── res/
│
├── test/
│   └── java/com/partyonapp/android/
│       ├── roborazzi/
│       │   ├── snapshots/
│       │   │   ├── images/
│       │   │   │   ├── EventDetailScreenPreview_phone.png
│       │   │   │   ├── EventDetailScreenPreview_phone_dark.png
│       │   │   │   ├── EventDetailScreenPreview_tablet.png
│       │   │   │   ├── EventCardPreview_phone.png
│       │   │   │   └── ...
│       │   │   └── diffs/
│       │   │       └── EventDetailScreenPreview_phone_diffs.png
│       │   └── RoborazziConfig.kt
│       └── ...
│
└── testScreenshots/
    ├── metadata.json
    └── artifacts.json
```

### 1.3 Roborazzi Configuration & Helpers

Create `android/app/src/test/java/com/partyonapp/android/roborazzi/RoborazziConfig.kt`:

```kotlin
package com.partyonapp.android.roborazzi

import io.github.takahirom.roborazzi.RobolectricDeviceQualifier
import io.github.takahirom.roborazzi.RoborazziConfig
import io.github.takahirom.roborazzi.RoborazziRule

object RoborazziConfigProvider {

    enum class DeviceConfig(val qualifier: RobolectricDeviceQualifier) {
        PHONE(RobolectricDeviceQualifier.Pixel5),
        TABLET(RobolectricDeviceQualifier.TabletLandscape),
        FOLDABLE(RobolectricDeviceQualifier.FoldableOpen);

        val displayName: String
            get() = when (this) {
                PHONE -> "phone"
                TABLET -> "tablet"
                FOLDABLE -> "foldable"
            }
    }

    fun configureRule(
        deviceConfig: DeviceConfig,
        isDarkMode: Boolean = false
    ): RoborazziRule {
        return RoborazziRule(
            options = RoborazziRule.Options(
                recordOptions = RoborazziRule.RecordOptions(
                    resizeScale = 1.0f,
                    roborazziEnabled = true
                ),
                compareOptions = RoborazziRule.CompareOptions(
                    resultBehavior = if (isDarkMode) {
                        "snapshot_dark"
                    } else {
                        "snapshot"
                    },
                    thresholdPixelCount = 100,
                    thresholdPercentage = 1.0f
                ),
                screenshotQualifiers = listOf(
                    deviceConfig.qualifier,
                    if (isDarkMode) "dark" else "light"
                )
            )
        )
    }

    fun getSuffix(deviceConfig: DeviceConfig, isDarkMode: Boolean): String {
        return "_${deviceConfig.displayName}${if (isDarkMode) "_dark" else ""}"
    }
}
```

### 1.4 Example Snapshot Test (EventCard)

Create `android/app/src/test/java/com/partyonapp/android/ui/components/EventCardSnapshotTest.kt`:

```kotlin
package com.partyonapp.android.ui.components

import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import com.partyonapp.android.roborazzi.RoborazziConfigProvider
import io.github.takahirom.roborazzi.RoborazziRule
import org.junit.Rule
import org.junit.Test

class EventCardSnapshotTest {

    @get:Rule
    val roborazziRule = RoborazziRule(
        options = RoborazziRule.Options(
            recordOptions = RoborazziRule.RecordOptions(
                resizeScale = 1.0f
            )
        )
    )

    @Test
    fun eventCard_phone_light() {
        captureSnapshot(
            RoborazziConfigProvider.DeviceConfig.PHONE,
            isDarkMode = false
        )
    }

    @Test
    fun eventCard_phone_dark() {
        captureSnapshot(
            RoborazziConfigProvider.DeviceConfig.PHONE,
            isDarkMode = true
        )
    }

    @Test
    fun eventCard_tablet_light() {
        captureSnapshot(
            RoborazziConfigProvider.DeviceConfig.TABLET,
            isDarkMode = false
        )
    }

    @Composable
    private fun captureSnapshot(
        deviceConfig: RoborazziConfigProvider.DeviceConfig,
        isDarkMode: Boolean
    ) {
        val config = RoborazziConfigProvider.configureRule(deviceConfig, isDarkMode)

        Box(
            modifier = Modifier
                .fillMaxSize()
                .background(
                    if (isDarkMode) {
                        MaterialTheme.colorScheme.background
                    } else {
                        MaterialTheme.colorScheme.surface
                    }
                )
        ) {
            EventCard(
                event = sampleEvent(),
                onClick = {},
                modifier = Modifier.fillMaxSize()
            )
        }
    }

    private fun sampleEvent() = Event(
        id = "event-001",
        title = "D&D Campaign Night",
        description = "Join us for an epic 5e campaign",
        datetime = "2025-03-15 19:00",
        location = "Comic Book Store",
        attendeeCount = 4,
        maxAttendees = 6
    )
}
```

### 1.5 Baseline Generation & Approval Workflow

#### Generate Initial Baselines

```bash
# Generate all phone baseline snapshots (light mode)
./gradlew recordRoborazziDebugSnapshots \
  -Proborazzi.record=true

# Generate dark mode variants
./gradlew recordRoborazziDebugSnapshots \
  -Proborazzi.record=true \
  -Proborazzi.qualifiers=dark

# Generate tablet & foldable variants
./gradlew recordRoborazziDebugSnapshots \
  -Proborazzi.record=true \
  -Proborazzi.qualifiers=landscape,fold_open
```

#### Run Snapshot Tests (Compare Mode)

```bash
# Compare against existing baselines
./gradlew testDebugRoborazziSnapshots

# With verbose diff output
./gradlew testDebugRoborazziSnapshots \
  -Proborazzi.printDiff=true
```

#### Update Baselines After Intentional Changes

```bash
# Review the diffs, then approve updates:
./gradlew updateRoborazziDebugSnapshots

# Git workflow:
git add android/app/src/test/roborazzi/snapshots/images/
git commit -m "chore(snapshot): update EventCard baselines for new styling"
```

---

## Part 2: iOS Setup (SwiftUI + swift-snapshot-testing + Prefire)

### 2.1 CocoaPods/SPM Installation

#### Option A: Swift Package Manager (Recommended)

Add to Package.swift or Xcode Project Settings:

```
File > Add Packages
URL: https://github.com/pointfreeco/swift-snapshot-testing
Version: 1.17.5 or later
```

#### Option B: CocoaPods

Add to Podfile:

```ruby
target 'PartyOnTests' do
  pod 'SnapshotTesting', '~> 1.17'
  pod 'Prefire', '~> 1.1'
end
```

Run `pod install`.

### 2.2 Directory Structure

```
ios/PartyOn/
├── PartyOn/
│   ├── Screens/
│   │   ├── EventDetailView.swift
│   │   └── EventDetailView+Previews.swift
│   ├── Components/
│   │   ├── EventCard.swift
│   │   └── EventCard+Previews.swift
│   └── ...
│
├── PartyOnTests/
│   ├── SnapshotTests/
│   │   ├── __Snapshots__/
│   │   │   ├── EventDetailViewTests.SnapshotTests/
│   │   │   │   ├── testEventDetail_iPhone_light.1.png
│   │   │   │   ├── testEventDetail_iPhone_dark.1.png
│   │   │   │   ├── testEventDetail_iPad_light.1.png
│   │   │   │   └── ...
│   │   │   └── EventCardTests.SnapshotTests/
│   │   │       └── ...
│   │   │
│   │   ├── EventDetailViewTests.swift
│   │   ├── EventCardTests.swift
│   │   └── SnapshotTestConfig.swift
│   │
│   └── PrefireTests.swift
│
└── ...
```

### 2.3 Snapshot Testing Configuration

Create `ios/PartyOnTests/SnapshotTests/SnapshotTestConfig.swift`:

```swift
import SwiftUI
import SnapshotTesting

enum DeviceConfig {
    case iPhonePhone_light
    case iPhonePhone_dark
    case iPadPro_light
    case iPadPro_dark
    case iPhoneLandscape_light
    case iPhoneLandscape_dark

    var viewImageConfig: ViewImageConfig {
        switch self {
        case .iPhonePhone_light, .iPhonePhone_dark:
            return .iPhone13Pro
        case .iPadPro_light, .iPadPro_dark:
            return .iPad(orientation: .portrait)
        case .iPhoneLandscape_light, .iPhoneLandscape_dark:
            return .iPhone13Pro(orientation: .landscape)
        }
    }

    var isDark: Bool {
        switch self {
        case .iPhonePhone_dark, .iPadPro_dark, .iPhoneLandscape_dark:
            return true
        default:
            return false
        }
    }

    var nameSuffix: String {
        switch self {
        case .iPhonePhone_light:
            return "iPhone_light"
        case .iPhonePhone_dark:
            return "iPhone_dark"
        case .iPadPro_light:
            return "iPad_light"
        case .iPadPro_dark:
            return "iPad_dark"
        case .iPhoneLandscape_light:
            return "iPhone_landscape_light"
        case .iPhoneLandscape_dark:
            return "iPhone_landscape_dark"
        }
    }
}

func assertSnapshot<V: View>(
    of view: V,
    config: DeviceConfig,
    file: StaticString = #file,
    testName: String = #function,
    line: UInt = #line
) {
    let viewController = UIHostingController(rootView: view)
    viewController.overrideUserInterfaceStyle = config.isDark ? .dark : .light
    viewController.view.frame = CGRect(
        x: 0,
        y: 0,
        width: config.viewImageConfig.size.width,
        height: config.viewImageConfig.size.height
    )

    assertSnapshot(
        matching: viewController,
        as: .image(config: config.viewImageConfig, precision: 0.99),
        testName: testName + "_" + config.nameSuffix,
        file: file,
        line: line
    )
}
```

### 2.4 Example Snapshot Test (EventCard)

Create `ios/PartyOnTests/SnapshotTests/EventCardTests.swift`:

```swift
import SwiftUI
import SnapshotTesting
import XCTest

@testable import PartyOn

class EventCardTests: XCTestCase {

    override func setUp() {
        super.setUp()
        isRecording = false
    }

    func testEventCard_iPhoneLight() {
        assertSnapshot(
            of: sampleEventCardPreview(),
            config: .iPhonePhone_light
        )
    }

    func testEventCard_iPhoneDark() {
        assertSnapshot(
            of: sampleEventCardPreview(),
            config: .iPhonePhone_dark
        )
    }

    func testEventCard_iPadLight() {
        assertSnapshot(
            of: sampleEventCardPreview(),
            config: .iPadPro_light
        )
    }

    func testEventCard_iPadDark() {
        assertSnapshot(
            of: sampleEventCardPreview(),
            config: .iPadPro_dark
        )
    }

    @ViewBuilder
    private func sampleEventCardPreview() -> some View {
        ZStack {
            Color(uiColor: UIColor.systemBackground)

            EventCard(
                event: Event(
                    id: "event-001",
                    title: "D&D Campaign Night",
                    description: "Join us for an epic 5e campaign adventure",
                    datetime: Date().addingTimeInterval(86400),
                    location: "Comic Book Store Downtown",
                    attendeeCount: 4,
                    maxAttendees: 6
                ),
                action: {}
            )
            .padding()
        }
        .frame(height: 200)
    }
}
```

### 2.5 Using Prefire for Auto-Generated Tests

Add `Prefire` plugin configuration to `ios/PartyOn/Components/EventCard+Previews.swift`:

```swift
import SwiftUI

extension EventCard {
    @available(iOS 16.0, *)
    static var previews: some View {
        Group {
            EventCard(
                event: sampleEvent,
                action: {}
            )
            .preferredColorScheme(.light)
            .previewDisplayName("Default")

            EventCard(
                event: sampleEvent,
                action: {}
            )
            .preferredColorScheme(.dark)
            .previewDisplayName("Dark")

            EventCard(
                event: sampleEvent,
                action: {}
            )
            .frame(maxWidth: 600)
            .previewDevice(PreviewDevice(rawValue: "iPad Pro (12.9-inch)"))
            .previewDisplayName("iPad")
        }
    }

    private static var sampleEvent: Event {
        Event(
            id: "event-001",
            title: "D&D Campaign Night",
            description: "Join us for an epic 5e campaign",
            datetime: Date().addingTimeInterval(86400),
            location: "Comic Book Store",
            attendeeCount: 4,
            maxAttendees: 6
        )
    }
}
```

Then enable Prefire in Build Settings:

```
Build Phases > [+] New Run Script Phase

Script:
/usr/local/bin/prefire
```

### 2.6 Baseline Generation & Approval

#### Generate Initial Baselines (Recording Mode)

```bash
cd ios

xcodebuild test \
  -scheme PartyOn \
  -configuration Debug \
  -derivedDataPath .build \
  -enableCodeCoverage NO \
  -testPlan "SnapshotTests" \
  PREFIRE_RECORD=1

ls PartyOnTests/SnapshotTests/__Snapshots__/
```

#### Run Tests in Compare Mode

```bash
xcodebuild test \
  -scheme PartyOn \
  -configuration Debug \
  -derivedDataPath .build \
  -enableCodeCoverage NO \
  -testPlan "SnapshotTests"
```

#### Update Baselines After Changes

```bash
xcodebuild test \
  -scheme PartyOn \
  -configuration Debug \
  -derivedDataPath .build \
  -enableCodeCoverage NO \
  -testPlan "SnapshotTests" \
  PREFIRE_RECORD=1

git add ios/PartyOnTests/SnapshotTests/__Snapshots__/
git commit -m "chore(snapshot): update EventCard baselines for new styling"
```

---

## Part 3: CI/CD Integration (GitHub Actions)

### 3.1 Android Snapshot Testing Workflow

Create `.github/workflows/android-snapshot-tests.yml`:

```yaml
name: Android Snapshot Tests

on:
  pull_request:
    paths:
      - 'android/**'
      - '.github/workflows/android-snapshot-tests.yml'
  push:
    branches:
      - main
      - develop

jobs:
  snapshot-tests:
    runs-on: ubuntu-latest
    timeout-minutes: 30

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: gradle

      - name: Run Roborazzi snapshot tests
        run: |
          cd android
          ./gradlew testDebugRoborazziSnapshots \
            -Proborazzi.printDiff=true \
            --stacktrace \
            --no-daemon

      - name: Upload snapshot diffs
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: roborazzi-diffs
          path: android/app/build/outputs/roborazzi/debug/**/*_diff.png
          retention-days: 7

      - name: Comment PR with snapshot failures
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '❌ Snapshot tests failed. Check the diff artifacts for visual changes.'
            })
```

### 3.2 iOS Snapshot Testing Workflow

Create `.github/workflows/ios-snapshot-tests.yml`:

```yaml
name: iOS Snapshot Tests

on:
  pull_request:
    paths:
      - 'ios/**'
      - '.github/workflows/ios-snapshot-tests.yml'
  push:
    branches:
      - main
      - develop

jobs:
  snapshot-tests:
    runs-on: macos-14
    timeout-minutes: 45

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Xcode
        uses: maxim-lobanov/setup-xcode@v1
        with:
          xcode-version: latest-stable

      - name: Install dependencies
        run: |
          cd ios
          pod install --repo-update

      - name: Run snapshot tests
        run: |
          cd ios
          xcodebuild test \
            -workspace PartyOn.xcworkspace \
            -scheme PartyOn \
            -configuration Debug \
            -derivedDataPath .build \
            -testPlan SnapshotTests \
            -enableCodeCoverage NO \
            -resultBundlePath TestResults.xcresult \
            CODE_SIGN_IDENTITY="" \
            CODE_SIGNING_REQUIRED=NO

      - name: Upload test results
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: ios-snapshot-results
          path: ios/TestResults.xcresult
          retention-days: 7

      - name: Comment PR with results
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '❌ iOS snapshot tests failed. Review the test results artifact.'
            })
```

---

## Acceptance Checklist

### Android Implementation
- [ ] Roborazzi configured in build.gradle.kts
- [ ] RoborazziConfig.kt supports phone/tablet/foldable devices
- [ ] Dark mode variants auto-generated with _dark suffix
- [ ] At least 3 snapshot tests implemented
- [ ] GitHub Actions workflow runs tests on every PR
- [ ] Baseline update process documented and tested
- [ ] Diff artifacts uploaded on failure
- [ ] Snapshot images committed to repo under src/test/roborazzi/snapshots/images/

### iOS Implementation
- [ ] swift-snapshot-testing integrated via SPM/CocoaPods
- [ ] Prefire configured to auto-generate tests from Previews
- [ ] SnapshotTestConfig.swift supports iPhone/iPad/landscape/dark
- [ ] At least 3 snapshot tests implemented
- [ ] GitHub Actions workflow runs tests on macOS-14
- [ ] Baseline update process documented
- [ ] Test results uploaded on failure
- [ ] Snapshot images committed to repo under __Snapshots__/

### CI/CD Integration
- [ ] Android workflow triggers on android/** path changes
- [ ] iOS workflow triggers on ios/** path changes
- [ ] Both workflows run in parallel
- [ ] Diff artifacts retained for 7 days
- [ ] PR comments notify of snapshot failures

### Documentation
- [ ] README documents baseline generation commands
- [ ] Troubleshooting guide addresses common issues
- [ ] Diff threshold configuration documented (1% default)
- [ ] Device matrix expansion guide provided

---

## Quick Reference Commands

### Android
```bash
./gradlew recordRoborazziDebugSnapshots
./gradlew testDebugRoborazziSnapshots
./gradlew updateRoborazziDebugSnapshots
open android/app/build/outputs/roborazzi/debug/
```

### iOS
```bash
xcodebuild test -workspace PartyOn.xcworkspace -scheme PartyOn \
  -testPlan SnapshotTests PREFIRE_RECORD=1
xcodebuild test -workspace PartyOn.xcworkspace -scheme PartyOn \
  -testPlan SnapshotTests
open TestResults.xcresult
```

---

## Sources & References

- [GitHub - takahirom/roborazzi](https://github.com/takahirom/roborazzi)
- [GitHub - pointfreeco/swift-snapshot-testing](https://github.com/pointfreeco/swift-snapshot-testing)
- [Medium - Snapshot Testing Jetpack Compose with Roborazzi](https://medium.com/@prnksingh829/snapshot-testing-jetpack-compose-with-roborazzi-a-quickstart-guide-911358662d9c)
- [Kodeco - Snapshot Testing Tutorial for SwiftUI](https://www.kodeco.com/24426963-snapshot-testing-tutorial-for-swiftui-getting-started)


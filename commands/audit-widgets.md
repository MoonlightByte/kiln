---
name: audit-widgets
description: Comprehensive audit of iOS WidgetKit, Live Activities, and Dynamic Island implementation
args:
  - name: code
    type: string
    description: Swift/SwiftUI code for widgets or Live Activities to audit
    required: true
  - name: focus
    type: string
    enum: [widget, live-activity, dynamic-island, timeline, configuration, performance, all]
    description: Specific area to focus on (default: all)
    required: false
  - name: targets
    type: string
    enum: [home-screen, lock-screen, dynamic-island, watch, tvos, visionos, all]
    description: Widget target platforms (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# iOS WidgetKit & Live Activities Audit

Perform a comprehensive audit of iOS Widget extensions, Live Activities, and Dynamic Island implementations against current best practices (iOS 18+).

## Instructions

Analyze the provided code for compliance with Apple WidgetKit specifications, App Intents integration, and Live Activities patterns. Reference latest WWDC 2025 guidelines for interactive widgets and cross-platform expansion.

## Audit Checklist

### Widget Extension Setup (Critical Priority)
{{#if (or (eq focus "widget") (eq focus "all") (not focus))}}
- [ ] **WG-001**: Widget target created in `Widget.swift` bundle identifier
- [ ] **WG-002**: `WidgetBundle` defined for multiple widget families
- [ ] **WG-003**: Info.plist configured with `NSExtensionAttributes`
- [ ] **WG-004**: `@main` attribute on widget entry point
- [ ] **WG-005**: Development target minimum iOS 16+ (iOS 17+ for Live Activities)
- [ ] **WG-006**: Code signing configured for widget target
- [ ] **WG-007**: Widget targets not linked to main app binary
- [ ] **WG-008**: Development Team ID matches main app
{{/if}}

### Timeline Provider Implementation (Critical Priority)
{{#if (or (eq focus "timeline") (eq focus "all") (not focus))}}
- [ ] **WG-010**: `TimelineProvider` protocol fully implemented
- [ ] **WG-011**: `getTimeline()` returns next refresh date within bounds
- [ ] **WG-012**: Placeholder view is lightweight and instantly available
- [ ] **WG-013**: `getSnapshot()` provides accurate current state
- [ ] **WG-014**: Timeline entries have correct `relevance` scores
- [ ] **WG-015**: Minimum 30 minutes between refresh intervals
- [ ] **WG-016**: Using `TimelineReloadPolicy.after(_:)` not `.never`
- [ ] **WG-017**: Error handling in timeline provider with fallback view
- [ ] **WG-018**: Background app refresh permission checked before updates
- [ ] **WG-019**: `IntervalTimelineProvider` for regular updates (iOS 17+)
{{/if}}

### Widget Families & Layout (Important Priority)
{{#if (or (eq focus "widget") (eq focus "all") (not focus))}}
- [ ] **WG-020**: Small widget (3x2 grid) - maximum 2 lines of text
- [ ] **WG-021**: Medium widget (3x3 grid) - balanced content layout
- [ ] **WG-022**: Large widget (3x4 grid) - detailed information display
- [ ] **WG-023**: Extra Large widget (4x4 grid) - iOS 17+ support
- [ ] **WG-024**: Lock Screen widget (circular/rectangular) minimum 20pt font
- [ ] **WG-025**: StandBy widgets with large text hierarchy
- [ ] **WG-026**: `containerBackground()` modifier for semantic backgrounds
- [ ] **WG-027**: Respecting widget content margins (safe area insets)
- [ ] **WG-028**: Using `@Environment(\.widgetFamily)` for responsive layouts
- [ ] **WG-029**: `widgetAccentable()` on interactive elements
- [ ] **WG-030**: No hardcoded colors - using `.label`, `.secondaryLabel`, `.systemBackground`
{{/if}}

### Live Activities API (Critical Priority)
{{#if (or (eq focus "live-activity") (eq focus "all") (not focus))}}
- [ ] **WG-035**: `ActivityAttributes` protocol defines immutable data structure
- [ ] **WG-036**: `ActivityState` defines mutable content updates
- [ ] **WG-037**: `@available(iOS 16.1, *)` on all Live Activity code
- [ ] **WG-038**: `ActivityKit` framework imported and linked
- [ ] **WG-039**: `startActivity()` called with valid `ActivityAttributes`
- [ ] **WG-040**: Updates use `.update(using:_:)` with animation context
- [ ] **WG-041**: Activity dismissed with `.end()` or auto-expires
- [ ] **WG-042**: Push notifications via APNs for remote updates
- [ ] **WG-043**: Alert type set to `.default` (not `.critical` unless urgent)
- [ ] **WG-044**: Testing with `ActivityKitSimulator` (Xcode 15+)
{{/if}}

### Dynamic Island Implementation (Important Priority)
{{#if (or (eq focus "dynamic-island") (eq focus "all") (not focus))}}
- [ ] **WG-050**: Compact presentation with `@ViewBuilder` closure
- [ ] **WG-051**: Expanded presentation without overflow
- [ ] **WG-052**: Minimal text (1-2 lines) in compact view
- [ ] **WG-053**: Leading/trailing content properly aligned
- [ ] **WG-054**: 4:1 aspect ratio for compact view (typical)
- [ ] **WG-055**: No hardcoded colors in Dynamic Island - using semantic
- [ ] **WG-056**: `.contentMargins()` applied for padding
- [ ] **WG-057**: Transitions between compact/expanded are smooth
- [ ] **WG-058**: Tap target minimum 44pt in expanded view
- [ ] **WG-059**: Long-press behavior well-defined or disabled
{{/if}}

### Lock Screen Widgets (Important Priority)
{{#if (or (eq focus "lock-screen") (eq focus "all") (not focus))}}
- [ ] **WG-060**: Lock Screen widget minimum 20pt font size
- [ ] **WG-061**: High contrast for readability in low light
- [ ] **WG-062**: `widgetLabel()` for context information
- [ ] **WG-063**: Circular gauge or progress indicators (iOS 17+)
- [ ] **WG-064**: `.systemSmall` family for single-line locks screen
- [ ] **WG-065**: Using `AccessoryWidgetBackground()` for consistency
- [ ] **WG-066**: No interactive buttons on Lock Screen (iOS 16-17)
- [ ] **WG-067**: Interactive Lock Screen widgets (iOS 18+) use App Intents
- [ ] **WG-068**: Ambient mode colors approved by Apple
{{/if}}

### StandBy & Watch (Important Priority)
{{#if (or (eq focus "widget") (eq focus "all") (not focus))}}
- [ ] **WG-070**: StandBy widget layout uses full screen (390x844 iPhone)
- [ ] **WG-071**: Large typography for StandBy readability
- [ ] **WG-072**: Minimal color palette (white/black) for battery life
- [ ] **WG-073**: `.containerBackground(for: .standby)` applied
- [ ] **WG-074**: watchOS widget minimum iOS 9 + watchOS 9+ compatibility
- [ ] **WG-075**: Watch widget size 170x170 or 182x182 points
{{/if}}

### Widget Configuration & App Intents (Important Priority)
{{#if (or (eq focus "configuration") (eq focus "all") (not focus))}}
- [ ] **WG-080**: `AppIntentTimelineProvider` for user-configurable widgets
- [ ] **WG-081**: `WidgetConfigurationIntent` defines settings
- [ ] **WG-082**: User can modify intent parameters via long-press
- [ ] **WG-083**: Intent values persisted with `@AppStorage` or `UserDefaults`
- [ ] **WG-084**: Configuration preview shows in lock screen settings
- [ ] **WG-085**: Intent validation with error handling
- [ ] **WG-086**: Sensible defaults if intent missing
- [ ] **WG-087**: `IntentConfiguration` deprecation warning handled (iOS 18+)
- [ ] **WG-088**: Custom intent types conform to `AppIntent` protocol
{{/if}}

### Widget URL Handling (Important Priority)
{{#if (or (eq focus "widget") (eq focus "all") (not focus))}}
- [ ] **WG-090**: Widget links use custom URL schemes or `deeplink://` format
- [ ] **WG-091**: Main app registers URL scheme in Info.plist
- [ ] **WG-092**: `onOpenURL()` modifier handles widget deep links
- [ ] **WG-093**: Navigation context persisted across app restart
- [ ] **WG-094**: Fallback screen if deep link invalid
- [ ] **WG-095**: URL encoding for special characters in parameters
- [ ] **WG-096**: `ScenePhase` monitored to detect widget tap navigation
{{/if}}

### Spatial Widgets for visionOS (Important Priority)
{{#if (or (eq focus "visionos") (eq focus "all") (not focus))}}
- [ ] **WG-100**: Widgets automatically available in visionOS 2+
- [ ] **WG-101**: iPhone/iPad widgets work without visionOS-specific code
- [ ] **WG-102**: Three-dimensional visual hierarchy for spatial presence
- [ ] **WG-103**: Using `Image(systemName:)` for 3D depth effects
- [ ] **WG-104**: Material design applied with depth and shadows
- [ ] **WG-105**: Large widget size (200+ points) recommended for spatial
- [ ] **WG-106**: Elevated appearance option in room placement settings
{{/if}}

### Interactive Widgets (iOS 18+) (Important Priority)
{{#if (or (eq focus "widget") (eq focus "all") (not focus))}}
- [ ] **WG-110**: `Button` with `App Intent` action on widget
- [ ] **WG-111**: Toggle state visible after button tap (no app launch)
- [ ] **WG-112**: Multiple interactive buttons supported per widget
- [ ] **WG-113**: App Intent requires `AppIntent` protocol conformance
- [ ] **WG-114**: `perform()` method handles action asynchronously
- [ ] **WG-115**: State updated via timeline provider refresh
- [ ] **WG-116**: 340% higher engagement than static widgets (verified metric)
- [ ] **WG-117**: Loading state shown during action execution
- [ ] **WG-118**: Error handling with user-friendly messages
{{/if}}

### Push Widget Updates (iOS 18+) (Important Priority)
{{#if (or (eq focus "widget") (eq focus "all") (not focus))}}
- [ ] **WG-120**: Widget push notification capability enabled
- [ ] **WG-121**: APNs payload includes `aps-content-available`
- [ ] **WG-122**: `appIntentClassName` in APNs to trigger update
- [ ] **WG-123**: Push notification limit respected (50 per user per month)
- [ ] **WG-124**: Remote update initiated from server/backend
- [ ] **WG-125**: Local push as fallback if remote unavailable
{{/if}}

### Performance Optimization (Critical Priority)
{{#if (or (eq focus "performance") (eq focus "all") (not focus))}}
- [ ] **WG-130**: Widget memory footprint < 30 MB
- [ ] **WG-131**: No main app code execution in widget bundle
- [ ] **WG-132**: Heavy computations in shared framework with compilation flags
- [ ] **WG-133**: Data fetching uses `URLSession` with timeout (max 10 sec)
- [ ] **WG-134**: Background refresh scheduled during low-power periods
- [ ] **WG-135**: Placeholder view loads instantly (< 100ms render)
- [ ] **WG-136**: Images pre-sized (no runtime scaling)
- [ ] **WG-137**: No synchronous network calls on main thread
- [ ] **WG-138**: Using `Task` for async operations with cancellation
- [ ] **WG-139**: View hierarchy depth limited to 4-5 levels
- [ ] **WG-140**: Avoiding `@ObservedObject` in widgets - use `@State` + `.task`
- [ ] **WG-141**: Metrics: timeline refresh < 30 sec, widget update < 5 sec
{{/if}}

### Accessibility (Important Priority)
{{#if (or (eq focus "widget") (eq focus "all") (not focus))}}
- [ ] **WG-150**: All interactive elements have `.accessibilityLabel`
- [ ] **WG-151**: Using `.accessibilityElement(children: .combine)` for groups
- [ ] **WG-152**: Text contrast 4.5:1 minimum in all widget sizes
- [ ] **WG-153**: Dynamic Type support with `.lineLimit(1)` or `.truncationMode(.tail)`
- [ ] **WG-154**: No color-only indicators (icon + text required)
- [ ] **WG-155**: Lock Screen widgets readable at arm's length
- [ ] **WG-156**: Reduce Motion respected with static variants
{{/if}}

### Testing Checklist (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **WG-160**: Timeline provider unit tests with mock `TimelineContext`
- [ ] **WG-161**: Widget UI tests with `XCTestWidgetUITest` framework
- [ ] **WG-162**: Live Activity UI tests with `ActivityKitSimulator`
- [ ] **WG-163**: Dynamic Island UI validated on iPhone 14 Pro simulator
- [ ] **WG-164**: Preview provider for each widget family
- [ ] **WG-165**: Placeholder rendering tests for instant availability
- [ ] **WG-166**: Intent configuration tests with valid/invalid inputs
- [ ] **WG-167**: URL deep link testing for all navigation paths
- [ ] **WG-168**: Memory profiling with Instruments (allocations, leaks)
- [ ] **WG-169**: Live Activity dismissal tests (timeout, manual, error)
- [ ] **WG-170**: visionOS spatial rendering validated
- [ ] **WG-171**: Localization testing for all supported languages
{{/if}}

## Code Examples

### Example: TimelineProvider Implementation

**GOOD - iOS 17+ Pattern:**
```swift
struct EventTimelineProvider: TimelineProvider {
    typealias Entry = EventEntry

    func placeholder(in context: Context) -> EventEntry {
        // Instant placeholder - no network
        EventEntry(date: Date(), event: nil)
    }

    func getSnapshot(in context: Context, completion: @escaping (EventEntry) -> ()) {
        // Quick snapshot for preview
        Task {
            let event = await fetchNextEvent()
            let entry = EventEntry(date: Date(), event: event)
            completion(entry)
        }
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<EventEntry>) -> ()) {
        Task {
            let event = await fetchNextEvent()
            let entry = EventEntry(date: Date(), event: event)

            // Refresh 5 minutes before event starts
            let refreshDate = event?.startTime.addingTimeInterval(-300) ?? Date().addingTimeInterval(3600)

            let timeline = Timeline(entries: [entry], policy: .after(refreshDate))
            completion(timeline)
        }
    }

    private func fetchNextEvent() async -> Event? {
        // Timeout-protected fetch
        try? await Task.sleep(nanoseconds: 10_000_000_000)  // 10 sec max
        // Fetch from app group or shared database
        return nil
    }
}

struct EventEntry: TimelineEntry {
    let date: Date
    let event: Event?
}

// Widget View
struct EventWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(
            kind: "com.party.event",
            provider: EventTimelineProvider()
        ) { entry in
            EventWidgetView(entry: entry)
                .containerBackground(for: .widget) {
                    Color(uiColor: .systemBackground)
                }
        }
        .configurationDisplayName("Event Widget")
        .description("Shows your next event")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}

struct EventWidgetView: View {
    let entry: EventEntry

    var body: some View {
        if let event = entry.event {
            VStack(alignment: .leading, spacing: 8) {
                Text(event.name)
                    .font(.headline)
                    .lineLimit(1)

                HStack(spacing: 12) {
                    Image(systemName: "calendar")
                    Text(event.startTime, style: .time)
                        .font(.caption)
                }
                .foregroundColor(.secondary)
            }
            .padding()
        } else {
            Text("No upcoming events")
                .font(.body)
                .foregroundColor(.secondary)
        }
    }
}
```

### Example: Live Activity with Dynamic Island

**GOOD - iOS 17+ Pattern:**
```swift
import ActivityKit
import SwiftUI

// Define attributes for Live Activity
struct GameSessionAttributes: ActivityAttributes {
    struct ContentState: Codable, Hashable {
        var currentRound: Int
        var playersRemaining: Int
        var timeRemaining: Int  // seconds
    }

    let gameId: String
    let gameName: String
    let gm: String
}

// Implement Live Activity view
struct GameSessionLiveActivity: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: GameSessionAttributes.self) { context in
            // Lock Screen presentation
            VStack(alignment: .leading, spacing: 4) {
                Text("Round \(context.state.currentRound)")
                    .font(.title3.bold())

                HStack {
                    Text("\(context.state.playersRemaining) players")
                        .font(.caption)
                    Spacer()
                    Text(formatTime(context.state.timeRemaining))
                        .font(.caption.monospaced())
                }
                .foregroundColor(.secondary)
            }
            .padding()
            .activityBackgroundTint(.indigo)

        } dynamicIsland: { context in
            DynamicIsland {
                // Expanded view
                DynamicIslandExpandedRegion(.leading) {
                    Label("Round \(context.state.currentRound)", systemImage: "gamecontroller.fill")
                        .font(.title3.bold())
                }

                DynamicIslandExpandedRegion(.trailing) {
                    Text(formatTime(context.state.timeRemaining))
                        .font(.title3.monospaced())
                        .contentTransition(.numericText())
                }

                DynamicIslandExpandedRegion(.bottom) {
                    VStack(spacing: 8) {
                        ProgressView(
                            value: Double(context.state.playersRemaining),
                            total: 6.0
                        )

                        HStack {
                            Text("Players: \(context.state.playersRemaining)/6")
                            Spacer()
                            Label("GM: \(context.attributes.gm)", systemImage: "person.fill")
                        }
                        .font(.caption)
                    }
                }

            } compactLeading: {
                // Compact leading
                Text("R\(context.state.currentRound)")
                    .font(.title3.bold())
            } compactTrailing: {
                // Compact trailing
                Text(formatTime(context.state.timeRemaining))
                    .contentTransition(.numericText())
            } minimal: {
                // Minimal presentation
                Image(systemName: "gamecontroller.fill")
            }
        }
    }

    private func formatTime(_ seconds: Int) -> String {
        let minutes = seconds / 60
        let secs = seconds % 60
        return String(format: "%d:%02d", minutes, secs)
    }
}

// Start Live Activity
@available(iOS 16.1, *)
func startGameSession(attributes: GameSessionAttributes, state: GameSessionAttributes.ContentState) {
    do {
        let activity = try Activity<GameSessionAttributes>.request(
            attributes: attributes,
            contentState: state,
            pushType: .token
        )

        // Get push token for remote updates
        let token = activity.pushToken
        // Send token to server for remote updates
    } catch {
        print("Failed to start activity: \(error)")
    }
}

// Update Live Activity
@available(iOS 16.1, *)
func updateGameSession(newState: GameSessionAttributes.ContentState) async {
    for activity in Activity<GameSessionAttributes>.all {
        await activity.update(using: newState)
    }
}

// End Live Activity
@available(iOS 16.1, *)
func endGameSession() {
    Task {
        for activity in Activity<GameSessionAttributes>.all {
            await activity.end()
        }
    }
}
```

### Example: Interactive Widget with App Intent

**GOOD - iOS 18+ Pattern:**
```swift
struct JoinGameIntent: AppIntent {
    static var title: LocalizedStringResource = "Join Game"

    @Parameter(title: "Event ID")
    var eventId: String

    func perform() async throws -> some IntentResult {
        // Perform action without launching app
        let success = try await joinGame(eventId: eventId)

        return .result(
            dialog: success ? "Joined game!" : "Failed to join game"
        )
    }

    private func joinGame(eventId: String) async throws -> Bool {
        // Hit backend API
        let url = URL(string: "https://api.party.com/join/\(eventId)")!
        let (data, response) = try await URLSession.shared.data(from: url)
        return (response as? HTTPURLResponse)?.statusCode == 200
    }
}

struct JoinGameWidget: Widget {
    var body: some WidgetConfiguration {
        AppIntentConfiguration(
            kind: "com.party.join-game",
            provider: GameTimelineProvider()
        ) { entry in
            VStack(alignment: .leading, spacing: 12) {
                Text("Next Game")
                    .font(.headline)

                if let game = entry.game {
                    VStack(alignment: .leading, spacing: 8) {
                        Text(game.name)
                            .lineLimit(1)

                        HStack(spacing: 8) {
                            Image(systemName: "person.2.fill")
                            Text("\(game.spotsLeft) spots left")
                                .font(.caption)
                        }
                        .foregroundColor(.secondary)

                        Button(intent: JoinGameIntent(eventId: game.id)) {
                            Label("Join", systemImage: "hand.thumbsup.fill")
                                .frame(maxWidth: .infinity)
                        }
                        .buttonStyle(.bordered)
                        .widgetAccentable()
                    }
                } else {
                    Text("No upcoming games")
                        .font(.caption)
                        .foregroundColor(.secondary)
                }
            }
            .padding()
            .containerBackground(for: .widget) {
                Color(uiColor: .systemBackground)
            }
        }
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}
```

## Performance Metrics

| Metric | Target | Critical Threshold |
|--------|--------|-------------------|
| Widget memory footprint | < 30 MB | > 50 MB fails |
| Placeholder render time | < 100 ms | > 500 ms noticeable |
| Timeline fetch | < 30 sec | > 30 sec system kills |
| Widget update latency | < 5 sec | > 10 sec appears hung |
| Live Activity push delay | < 2 sec | > 5 sec poor UX |
| Dynamic Island transition | < 300 ms | > 500 ms janky |

## Testing Patterns

```swift
// Unit test timeline provider
final class EventTimelineProviderTests: XCTestCase {
    func testPlaceholderInstant() {
        let provider = EventTimelineProvider()
        let placeholder = provider.placeholder(in: .init(
            isPreview: true,
            family: .systemSmall,
            displaySize: .init(width: 170, height: 170)
        ))

        XCTAssertNotNil(placeholder)
        XCTAssertEqual(placeholder.event, nil)
    }

    func testTimelineRefreshSchedule() {
        let provider = EventTimelineProvider()
        let exp = expectation(description: "Timeline completion")

        provider.getTimeline(in: .init(isPreview: false, family: .systemMedium, displaySize: .init(width: 340, height: 360))) { timeline in
            XCTAssertGreater(timeline.entries.count, 0)
            XCTAssertNotNil(timeline.policy)
            exp.fulfill()
        }

        wait(for: [exp], timeout: 15)
    }
}

// Live Activity test
@available(iOS 16.1, *)
final class GameSessionLiveActivityTests: XCTestCase {
    func testActivityStartsSuccessfully() async {
        let attributes = GameSessionAttributes(
            gameId: "test-123",
            gameName: "Curse of Strahd",
            gm: "Matt"
        )

        let state = GameSessionAttributes.ContentState(
            currentRound: 1,
            playersRemaining: 5,
            timeRemaining: 3600
        )

        do {
            let activity = try Activity<GameSessionAttributes>.request(
                attributes: attributes,
                contentState: state
            )
            XCTAssertNotNil(activity)
        } catch {
            XCTFail("Failed to start activity: \(error)")
        }
    }
}
```

## Output Format

Generate a structured audit report:

```markdown
## Widget Audit Results: [Widget Name]

### Critical Issues (🔴)
List configuration problems, timeline failures, memory issues.
Each with: Component, description, severity, fix suggestion.

### Important Issues (🟠)
List guideline violations, performance concerns, accessibility issues.

### Warnings (🟡)
List suboptimal patterns that work but should be improved.

### Suggestions (🔵)
List polish opportunities and modernization suggestions.

### Summary
- Total issues: X
- Critical: X | Important: X | Warning: X | Suggestion: X
- Overall compliance: X%
- Target platforms: [iOS, watchOS, visionOS, etc.]

### Performance Analysis
- Memory footprint: X MB
- Timeline refresh time: X sec
- Widget update latency: X sec
- Recommended optimizations: [list]

### Recommended Actions
1. [Most urgent fix]
2. [Second priority]
3. [Third priority]

### Testing Recommendations
- [ ] Add timeline provider unit tests
- [ ] Test all widget families
- [ ] Validate Live Activity dismissal
- [ ] Check memory under load (Instruments)
- [ ] Test Deep Links navigation
```

## References

Latest iOS WidgetKit and Live Activities documentation:
- [What's new in widgets - WWDC2025](https://developer.apple.com/videos/play/wwdc2025/278/)
- [WidgetKit Documentation](https://developer.apple.com/documentation/widgetkit)
- [Live Activities and Dynamic Island](https://developer.apple.com/news/?id=bkm73839)
- [App Intents Framework](https://developer.apple.com/documentation/appintents)
- [Interactive Widgets iOS 18](https://developer.apple.com/videos/play/wwdc2024/10199/)
- [ActivityKit Framework iOS 16.1+](https://developer.apple.com/documentation/activitykit)

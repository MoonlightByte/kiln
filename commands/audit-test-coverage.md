---
name: audit-test-coverage
description: Audit mobile app test coverage across unit, integration, UI, accessibility, and snapshot tests
args:
  - name: path
    type: string
    description: Path to scan for tests (e.g., android/, ios/, or specific test directory)
    required: true
  - name: target
    type: string
    enum: [android, ios, both, auto]
    description: Target platform(s) (default: auto-detect)
    required: false
  - name: report_format
    type: string
    enum: [markdown, json, csv, html]
    description: Output format for coverage report (default: markdown)
    required: false
  - name: include_accessibility
    type: boolean
    description: Include accessibility test coverage analysis (default: true)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Mobile Test Coverage Audit

Comprehensive test coverage analysis across unit tests, integration tests, UI tests, snapshot tests, and accessibility tests for mobile applications.

## Instructions

Analyze test coverage in the provided path. Auto-detect platform based on file structure (Kotlin/XML = Android, Swift = iOS). Generate actionable coverage report with gap identification and trend tracking.

## Coverage Categories

### TC-001: Unit Test Coverage

**What to measure:**
- [ ] All business logic tested (>85% code coverage)
- [ ] ViewModels/Services have unit tests
- [ ] Repository layer has unit tests
- [ ] Utility functions and extensions tested
- [ ] Data models have equality/serialization tests
- [ ] Error handling paths covered

**Gaps to identify:**
- [ ] Untested classes (especially ViewModels)
- [ ] Missing null safety checks in tests
- [ ] No negative/error case tests
- [ ] Mock objects not configured correctly

### TC-002: Integration Test Coverage

**What to measure:**
- [ ] API client integration tested
- [ ] Database integration tested
- [ ] Local storage (SharedPreferences/UserDefaults) tested
- [ ] Authentication flow tested end-to-end
- [ ] Network layer retry logic tested
- [ ] Dependency injection verified

**Gaps to identify:**
- [ ] No database integration tests
- [ ] API mocking incomplete
- [ ] Network timeout scenarios untested
- [ ] Missing middleware testing

### TC-003: UI/Component Test Coverage

**What to measure:**
- [ ] All screens have component tests
- [ ] Navigation flows tested
- [ ] User interactions tested (tap, scroll, input)
- [ ] State changes reflected in UI
- [ ] Loading/error states verified
- [ ] Empty states rendered correctly

**Gaps to identify:**
- [ ] Screens without component tests
- [ ] Navigation untested
- [ ] Button/field interactions missing
- [ ] Dialog dismissal flows untested

### TC-004: Snapshot Test Coverage

**What to measure:**
- [ ] Visual regressions caught
- [ ] Layout consistency verified
- [ ] Text rendering correct
- [ ] Color/styling maintained
- [ ] Component variants covered
- [ ] Dark mode variants tested

**Gaps to identify:**
- [ ] No snapshot tests for screens
- [ ] New component variants not snapshotted
- [ ] Outdated snapshots not reviewed
- [ ] Dark mode snapshots missing

### TC-005: Accessibility (a11y) Test Coverage

**What to measure:**
- [ ] Touch target sizes verified (48dp/44pt minimum)
- [ ] Color contrast tested (4.5:1 normal, 3:1 large)
- [ ] Content descriptions on interactive elements
- [ ] Focus visibility tested
- [ ] Screen reader compatibility
- [ ] Keyboard navigation working

**Gaps to identify:**
- [ ] No automated a11y tests
- [ ] Missing TalkBack/VoiceOver validation
- [ ] Touch target detection gaps
- [ ] Focus order incorrect

### TC-006: State Variation Coverage

**What to measure:**
- [ ] Loading states tested
- [ ] Error states tested
- [ ] Empty states tested
- [ ] Success states tested
- [ ] Disabled/inactive states tested
- [ ] Transition states tested

**Gaps to identify:**
- [ ] Loading UI never tested
- [ ] Error messages untested
- [ ] Empty state not designed/tested
- [ ] State transitions missing

### TC-007: Component Test Readiness

**What to measure:**
- [ ] Reusable components have isolated tests
- [ ] Props/parameters validated
- [ ] Default values work correctly
- [ ] Component composition verified
- [ ] Styling themes tested
- [ ] Custom modifiers/extensions work

**Gaps to identify:**
- [ ] Composable functions without tests
- [ ] No props documentation
- [ ] Theme customization untested
- [ ] Modifier composition broken

### TC-008: Critical Path Coverage

**What to measure:**
- [ ] Signup flow end-to-end tested
- [ ] Login flow end-to-end tested
- [ ] Primary user journey tested
- [ ] Data mutations tested
- [ ] User permissions respected
- [ ] Payment/booking flows verified

**Gaps to identify:**
- [ ] Signup missing E2E test
- [ ] Login edge cases untested
- [ ] Primary feature untested
- [ ] Permission denied paths missing

### TC-009: Error Path Coverage

**What to measure:**
- [ ] Network errors handled
- [ ] Validation errors caught
- [ ] Server errors gracefully handled
- [ ] Retry logic tested
- [ ] Timeouts managed
- [ ] Offline scenarios tested

**Gaps to identify:**
- [ ] No network error tests
- [ ] Validation untested
- [ ] Timeout behavior undefined
- [ ] Offline mode not tested

### TC-010: Platform-Specific Coverage (Android)

**Android-Specific Checks:**
- [ ] Fragment lifecycle tests
- [ ] ViewModel state survival (configuration change)
- [ ] LazyColumn/Box/Row composition tested
- [ ] Material 3 components validated
- [ ] TalkBack accessibility tested
- [ ] Back navigation works
- [ ] Permission handling tested
- [ ] Jetpack libraries (Room, Flow, etc.) tested

**Gaps to identify:**
- [ ] No ViewModel state tests
- [ ] Fragment transactions broken
- [ ] Material 3 theme misapplied
- [ ] Permission requests untested

### TC-011: Platform-Specific Coverage (iOS)

**iOS-Specific Checks:**
- [ ] SwiftUI view state tests
- [ ] Navigation stack tested
- [ ] VoiceOver compatibility
- [ ] Safe area handling
- [ ] Large text support (Dynamic Type)
- [ ] Memory leak detection
- [ ] Sheet/Navigation dismissal
- [ ] App delegate lifecycle

**Gaps to identify:**
- [ ] View state not observable
- [ ] Navigation presentation issues
- [ ] VoiceOver labels missing
- [ ] Memory leaks in tests

### TC-012: Performance Test Coverage

**What to measure:**
- [ ] Scroll performance (60fps target)
- [ ] List rendering with >100 items
- [ ] Image loading/caching
- [ ] Animation smoothness
- [ ] Memory usage under load
- [ ] Startup time measured

**Gaps to identify:**
- [ ] No performance benchmarks
- [ ] Large list rendering slow
- [ ] Memory leaks detected
- [ ] Animation jank

### TC-013: Data Coverage

**What to measure:**
- [ ] Serialization/deserialization tested
- [ ] Null handling verified
- [ ] Large dataset handling
- [ ] Data persistence verified
- [ ] Cache invalidation tested
- [ ] Data transformation logic

**Gaps to identify:**
- [ ] No JSON serialization tests
- [ ] Null crashes possible
- [ ] Large dataset untested
- [ ] Cache corruption possible

### TC-014: Localization Coverage

**What to measure:**
- [ ] All strings translated
- [ ] RTL layout support
- [ ] Date/time formatting per locale
- [ ] Number formatting per locale
- [ ] Plural forms tested
- [ ] String length edge cases

**Gaps to identify:**
- [ ] Hardcoded English strings
- [ ] RTL not tested
- [ ] Missing translations
- [ ] Locale-specific formatting broken

### TC-015: Security Test Coverage

**What to measure:**
- [ ] No hardcoded credentials
- [ ] Secure token storage tested
- [ ] HTTPS enforced
- [ ] SQL injection prevention
- [ ] Input validation tested
- [ ] Sensitive data not logged

**Gaps to identify:**
- [ ] Secrets in code
- [ ] Tokens in SharedPreferences (not encrypted)
- [ ] HTTP used for sensitive data
- [ ] No input validation tests

### TC-016: Deeplink Coverage

**What to measure:**
- [ ] Deeplink parsing tested
- [ ] Invalid deeplinks handled
- [ ] Deeplink navigation verified
- [ ] App restoration from deeplink
- [ ] Share flows work

**Gaps to identify:**
- [ ] Deeplinks untested
- [ ] Malformed links crash app
- [ ] Share button broken
- [ ] Navigation fails from deeplink

### TC-017: Dependency Mock Coverage

**What to measure:**
- [ ] API clients mocked
- [ ] Database mocked
- [ ] File system mocked
- [ ] Location services mocked
- [ ] Notification mocked
- [ ] Camera/media mocked

**Gaps to identify:**
- [ ] Real API calls in tests
- [ ] Database not mocked
- [ ] File operations in tests
- [ ] Permission prompts in tests

### TC-018: Edge Case Coverage

**What to measure:**
- [ ] Empty strings handled
- [ ] Very long strings handled
- [ ] Unicode characters handled
- [ ] Zero/negative numbers
- [ ] Boundary values (min/max)
- [ ] Special characters

**Gaps to identify:**
- [ ] No edge case tests
- [ ] Text truncation broken
- [ ] Emoji rendering fails
- [ ] Large numbers overflow

### TC-019: Async/Concurrency Coverage

**What to measure:**
- [ ] Coroutines tested (Android)
- [ ] Async/await patterns tested (iOS)
- [ ] Race conditions tested
- [ ] Cancellation behavior
- [ ] Timeout handling
- [ ] Main thread violations caught

**Gaps to identify:**
- [ ] No async tests
- [ ] Race conditions possible
- [ ] Cancellation not working
- [ ] Main thread hangs possible

### TC-020: Mutation Testing Coverage

**What to measure:**
- [ ] Tests catch operator changes (+/-, >/>=)
- [ ] Tests catch boolean flips
- [ ] Tests catch condition removals
- [ ] Mutation score >70%
- [ ] Trivial mutations caught

**Gaps to identify:**
- [ ] Tests skip boolean branches
- [ ] Conditional checks not validated
- [ ] Low mutation score
- [ ] Tests too permissive

### TC-021: Behavior-Driven Tests (BDD)

**What to measure:**
- [ ] Given-When-Then format used
- [ ] Readable test scenarios
- [ ] User perspective tested
- [ ] Feature files linked to tests
- [ ] Gherkin syntax valid
- [ ] Test coverage traceable

**Gaps to identify:**
- [ ] No BDD tests
- [ ] Tests hard to read
- [ ] No feature files
- [ ] Gherkin not used

### TC-022: Test Organization

**What to measure:**
- [ ] Tests grouped by feature
- [ ] Test naming consistent
- [ ] Setup/teardown proper
- [ ] No duplicate tests
- [ ] Test isolation verified
- [ ] Dependencies manageable

**Gaps to identify:**
- [ ] Chaotic test structure
- [ ] Unclear test names
- [ ] Global state issues
- [ ] Flaky tests due to ordering

### TC-023: Test Documentation

**What to measure:**
- [ ] Test purposes documented
- [ ] Complex scenarios explained
- [ ] Setup requirements noted
- [ ] Edge cases documented
- [ ] Known limitations noted
- [ ] Maintenance notes present

**Gaps to identify:**
- [ ] Tests lack comments
- [ ] Purpose unclear
- [ ] Brittle setup undocumented
- [ ] Hard to maintain

### TC-024: Visual Regression Coverage

**What to measure:**
- [ ] UI changes caught by snapshots
- [ ] Layout shifts detected
- [ ] Typography changes tracked
- [ ] Color changes verified
- [ ] Spacing consistency checked
- [ ] Component rendering verified

**Gaps to identify:**
- [ ] No visual regression tests
- [ ] Layout broken undetected
- [ ] Typography inconsistent
- [ ] Color drift uncaught

### TC-025: Form Validation Coverage

**What to measure:**
- [ ] Empty field validation
- [ ] Format validation (email, phone, etc.)
- [ ] Length validation
- [ ] Pattern matching
- [ ] Cross-field validation
- [ ] Error messages displayed

**Gaps to identify:**
- [ ] No validation tests
- [ ] Invalid data accepted
- [ ] Validation too strict
- [ ] Error messages missing

## Platform-Specific Coverage (Android)

### Android Unit Tests
- [ ] ViewModel tests with `InstantTaskExecutorRule`
- [ ] Repository tests with mock API client
- [ ] StateFlow emissions verified
- [ ] LiveData transformations tested
- [ ] Jetpack Compose state logic
- [ ] Room database queries

### Android Integration Tests
- [ ] Navigation tests with Hilt
- [ ] Database migration tests
- [ ] API integration with OkHttp mocks
- [ ] SharedPreferences encrypted correctly
- [ ] Fragment/Activity transitions

### Android UI Tests (Instrumentation)
- [ ] Compose testing library used
- [ ] Semantics matchers for elements
- [ ] Typography/colors verified
- [ ] Animations not checked directly (too brittle)
- [ ] Material 3 components render

### Android Accessibility
- [ ] TalkBack tested
- [ ] Touch targets 48x48dp minimum
- [ ] Contrast ratios 4.5:1
- [ ] `contentDescription` on icons
- [ ] Focus order logical
- [ ] Magnification support

## Platform-Specific Coverage (iOS)

### iOS Unit Tests
- [ ] MVVM ViewModel tests
- [ ] Combine publishers tested
- [ ] @Published properties verified
- [ ] CoreData model tests
- [ ] Extensions/helpers tested
- [ ] Decodable/Encodable

### iOS Integration Tests
- [ ] URLSession mocked with URLProtocol
- [ ] FileManager sandboxed
- [ ] Keychain mocking
- [ ] Navigation stack tested
- [ ] SwiftUI environment

### iOS UI Tests (XCUITest)
- [ ] SwiftUI view accessibility IDs
- [ ] Navigation flows E2E
- [ ] User interactions (tap, scroll, type)
- [ ] Alert/Sheet dismissal
- [ ] View hierarchy assertions

### iOS Accessibility
- [ ] VoiceOver tested
- [ ] `.accessibilityLabel` on elements
- [ ] Touch targets 44x44pt minimum
- [ ] Contrast ratios 4.5:1
- [ ] `.accessibilityValue` for dynamic content
- [ ] Dynamic Type support

## CI/CD Trend Tracking

### Metrics to Track

**Coverage Metrics:**
- [ ] Code coverage trend (weekly)
- [ ] Test count trend
- [ ] Test execution time trend
- [ ] Flaky test percentage
- [ ] Test pass rate by platform

**Quality Metrics:**
- [ ] Critical issue detection rate
- [ ] Regression rate
- [ ] Time to fix failures
- [ ] Coverage gaps identified per sprint

**Accessibility Metrics:**
- [ ] a11y test coverage percentage
- [ ] Touch target violations
- [ ] Contrast ratio violations
- [ ] Missing labels/descriptions

### GitHub Actions Integration

```yaml
# Coverage report job
- name: Test Coverage
  run: |
    # Android
    ./gradlew test jacocoTestReport
    # iOS
    xcodebuild test -scheme PartyOn -derivedDataPath Build

    # Generate reports
    mkdir -p coverage-reports
    cp android/build/reports/jacoco coverage-reports/android
    cp ios/Build/coverage coverage-reports/ios

    # Upload artifacts
    - uses: actions/upload-artifact@v3
      with:
        name: coverage-reports
        path: coverage-reports/
        retention-days: 30
```

## Output Format

```markdown
# Test Coverage Audit: [App/Feature Name]

## Coverage Summary

| Category | Coverage | Target | Status |
|----------|----------|--------|--------|
| Unit Tests | 65% | 85% | ⚠️ |
| Integration Tests | 40% | 60% | ❌ |
| UI/Component Tests | 55% | 75% | ⚠️ |
| Snapshot Tests | 30% | 50% | ❌ |
| Accessibility Tests | 25% | 100% | ❌ |
| Critical Path | 90% | 95% | ✅ |
| Error Paths | 45% | 80% | ⚠️ |

## Coverage Gaps (High Priority)

### Critical Gaps (❌)

**TC-004: No Snapshot Tests**
- Issue: Visual regression detection missing
- Impact: UI changes go undetected to production
- Components affected:
  - `EventCard.kt` - No snapshot baseline
  - `PartyDetailScreen.kt` - 3 layout variants untested
  - `ConversationBubble.kt` - Dark/light mode untested
- Recommendation: Add snapshot tests for all screen variants (30 min)

**TC-005: Limited Accessibility Testing**
- Issue: Only 25% a11y coverage
- Impact: WCAG 2.2 Level AA compliance at risk
- Gaps:
  - No TalkBack testing (Android)
  - VoiceOver coverage: 0%
  - Touch target violations: 8 instances
  - Missing content descriptions: 12 icons
  - Color contrast failures: 3 components
- Recommendation: Implement axe DevTools + manual VoiceOver validation

### Important Gaps (⚠️)

**TC-001: Unit Test Coverage Below Target**
- Current: 65% | Target: 85%
- Missing tests:
  - `EventViewModel.kt` - 0% coverage
  - `BookingViewModel.kt` - 40% coverage
  - `ChatViewModel.kt` - 55% coverage
- Recommendation: Add ViewModel state tests for State/UI events

**TC-003: Component Test Coverage**
- Screens without tests: 4
  - PartyDetailScreen
  - ConversationDetailScreen
  - ClubDashboardScreen
  - TavernChatScreen
- Navigation flows untested: 6
- Recommendation: Prioritize critical path (PartyDetail + ConversationDetail)

### Accessibility Violations

**Touch Target Issues (TC-006 subtask)**
```
- EventCard favorite icon: 32×32dp (needs 48×48dp)
- CloseButton in dialogs: 28×28dp (needs 44×44pt iOS)
- Chat input emoji button: 36×36dp (acceptable)
```

**Content Description Missing (TC-006 subtask)**
```
- NavigationBar.kt:
  Line 15: Icon(Icons.Default.Explore, null) ← needs contentDescription
  Line 18: Icon(Icons.Default.Convention, null) ← needs contentDescription

- EventCard.kt:
  Line 42: Icon(Icons.Default.Favorite) ← needs conditional description
  Line 55: Icon(Icons.Default.Location) ← needs "Location" description
```

**Color Contrast Failures (TC-006 subtask)**
```
- Event title (14sp) on light background: 3.2:1 (needs 4.5:1)
  Location: EventCard.kt line 20
  Fix: Use darker shade of primary color

- Secondary text on tonal background: 2.8:1 (needs 3:1)
  Location: PartyDetail.kt line 45
  Fix: Increase opacity from 70% to 85%
```

## Detailed Analysis

### Unit Test Coverage by Module

| Module | Coverage | Trend | Notes |
|--------|----------|-------|-------|
| `events/` | 72% | ↑ +5% | EventViewModel needs 10% more |
| `bookings/` | 68% | ↔ Stable | All critical paths covered |
| `messaging/` | 55% | ↓ -3% | New ChatViewModel not tested |
| `auth/` | 85% | ✅ | Fully covered |
| `location/` | 40% | ❌ | LFG beacon logic untested |

### Test Execution Performance

| Suite | Count | Duration | Avg/Test |
|-------|-------|----------|----------|
| Unit Tests | 284 | 2m 15s | 0.48s |
| Integration | 42 | 1m 30s | 2.14s |
| Component | 31 | 4m 20s | 8.39s |
| **Total** | **357** | **8m 5s** | **1.35s** |

### Flaky Tests Detected

- `ChatViewModel_loadMessages_Test` - Fails intermittently (timing)
- `BookingFlow_Integration_Test` - Fails on network timeout
- `EventCard_Snapshot_Test` - Font rendering variance

### Recommended Priority Actions

1. **Critical (Complete This Sprint)**
   - [ ] Add snapshot tests for 5 core screens (4 hours)
   - [ ] Fix accessibility violations: touch targets + descriptions (3 hours)
   - [ ] EventViewModel unit test coverage from 0% to 80% (2 hours)

2. **Important (Next Sprint)**
   - [ ] Component tests for PartyDetail + Conversation screens (4 hours)
   - [ ] Color contrast fixes + accessibility audit pass (2 hours)
   - [ ] Integration tests for booking flow (3 hours)

3. **Nice-to-Have (Future)**
   - [ ] Performance benchmarks for scroll + image loading
   - [ ] Mutation testing implementation
   - [ ] BDD scenarios for user journeys

## Coverage Trend Report

### Monthly Trend (Last 3 Months)

```
Unit Coverage:       65% → 68% → 72% ✅ Improving
Component Coverage:  40% → 45% → 55% ✅ Improving
a11y Coverage:       15% → 20% → 25% ⚠️ Slow progress
Snapshot Coverage:   20% → 25% → 30% ⚠️ Needs acceleration
```

### Regression Detection Rate

- Bugs caught by tests: 47/52 (90% catch rate)
- Bugs escaping to production: 5 (9.6% escape rate)
- Average time to fix: 2.3 hours

## Accessibility Compliance Summary

**WCAG 2.2 Level AA Status: 68%**

| Criterion | Status | Issues |
|-----------|--------|--------|
| 2.5.8 Target Size | ⚠️ Partial | 8 violations |
| 1.4.3 Contrast | ⚠️ Partial | 3 violations |
| 1.1.1 Non-text Content | ❌ | 12 missing descriptions |
| 2.4.7 Focus Visible | ✅ | 0 issues |
| 2.5.1 Pointer Gestures | ✅ | 0 issues |
| 1.4.4 Text Resize | ⚠️ | Needs Dynamic Type/sp testing |

**Automated Coverage: 30%** (consistent with industry standard of 20-30%)
- Automated tools catch: Structure, missing labels, contrast ratio
- Manual testing needed: Context, readability, content quality

## Next Steps

1. Review critical gaps with team
2. Assign developers to test coverage tasks
3. Schedule accessibility audit with domain expert
4. Implement CI/CD coverage reporting
5. Monthly trend tracking dashboard

```

## Example Output (JSON Format)

```json
{
  "timestamp": "2025-02-26T15:30:00Z",
  "app_name": "PartyOn",
  "summary": {
    "overall_coverage": 68.5,
    "target_coverage": 80,
    "compliance_status": "PARTIAL",
    "critical_gaps": 3,
    "important_gaps": 5
  },
  "coverage_by_category": {
    "unit_tests": {
      "coverage": 72,
      "target": 85,
      "status": "BELOW_TARGET"
    },
    "accessibility": {
      "coverage": 25,
      "target": 100,
      "status": "CRITICAL_GAP"
    }
  },
  "accessibility": {
    "wcag_level": "AA",
    "compliance_percentage": 68,
    "violations": {
      "touch_targets": 8,
      "contrast": 3,
      "missing_descriptions": 12
    }
  },
  "recommendations": [
    {
      "priority": "CRITICAL",
      "category": "Snapshot Tests",
      "effort_hours": 4,
      "impact": "Catch visual regressions"
    }
  ]
}
```

## Coverage Analysis Checklist

When analyzing coverage, verify:

- [ ] Test infrastructure configured (unittest/XCTest/Espresso)
- [ ] Coverage tools installed (Jacoco/Coverage.py/llvm-cov)
- [ ] Baseline coverage metrics established
- [ ] Test naming conventions consistent
- [ ] Tests organized by layer (unit/integration/ui)
- [ ] CI/CD coverage reporting enabled
- [ ] Flaky tests identified and quarantined
- [ ] Accessibility testing framework chosen
- [ ] Performance benchmarks in place
- [ ] Test review process documented

## References

**Coverage Best Practices:**
- [Automation Test Coverage: Metrics, Strategies, and Best Practices - Ranorex](https://www.ranorex.com/blog/automation-test-coverage/)
- [How to Increase Test Coverage for Mobile and Web Apps - BrowserStack](https://www.browserstack.com/guide/how-to-increase-test-coverage)
- [Test Coverage Metrics: What is, Types & Examples - PractiTest](https://www.practitest.com/resource-center/blog/test-coverage-metrics/)

**Accessibility Testing:**
- [Automating Accessibility Testing in 2026 - BrowserStack](https://www.browserstack.com/guide/automate-accessibility-testing)
- [Automated Tools for Testing Accessibility - Harvard Digital Accessibility Services](https://accessibility.huit.harvard.edu/auto-tools-testing)
- [Web Accessibility Evaluation Tools List - W3C WAI](https://www.w3.org/WAI/test-evaluate/tools/list/)

**Mobile Testing:**
- [Latest Mobile App Testing Trends in 2025 - 42Gears](https://www.42gears.com/blog/top-mobile-app-testing-trends/)
- [Top 6 Mobile Testing Tools in 2025 - Autify](https://autify.com/blog/mobile-testing-tools)

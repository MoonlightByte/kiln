# Kiln Plugin: Shared Cross-Platform Reference

**Purpose**: Central hub for cross-platform accessibility and design token resources used by Android Design and iOS Design skills.

**Scope**: Material Design 3 (Android/Jetpack Compose) ↔ Human Interface Guidelines (iOS/SwiftUI)

---

## Reference Documents

### 1. WCAG 2.2 Mobile Accessibility Rules
**File**: `./references/wcag-22-mobile-rules.md`
**Use When**: Implementing accessibility features, preparing for WCAG AA compliance, conducting accessibility audits

**Covers**:
- Level AA requirements for mobile (contrast, touch targets, keyboard navigation)
- Exact metrics: 48dp touch targets (Android), 44pt (iOS)
- 4.5:1 contrast ratio for normal text, 3:1 for large text
- Screen reader requirements (contentDescription, accessibilityLabel)
- Automated testing with Axe-Core Mobile
- Manual testing workflows with TalkBack (Android) and VoiceOver (iOS)

**Key Sections**:
- Contrast ratios with code examples for both platforms
- Touch target sizing with Android Material & iOS HIG requirements
- Screen reader semantic information (4.1.2 Name, Role, Value)
- WCAG 2.2 criteria summary table for quick reference

**Example Use Case**: "Ensure all interactive elements meet WCAG AA contrast requirements"

---

### 2. Cross-Platform Token Mapping
**File**: `./references/cross-platform-token-mapping.md`
**Use When**: Converting design between Android and iOS, implementing consistent design tokens

**Covers**:
- Color token mappings (Material Design 3 ↔ HIG semantic colors)
- Typography mappings (Material styles ↔ Dynamic Type)
- Spacing scale (4dp base on both platforms)
- Corner radius, elevation, shadow mappings
- Touch target sizes (48dp/44pt)
- Complete component token mappings (buttons, text fields)

**Key Tables**:
- Primary colors: `MaterialTheme.colorScheme.primary` → `.accentColor`
- Secondary colors: `MaterialTheme.colorScheme.secondary` → `.secondary`
- Surface colors: `MaterialTheme.colorScheme.surface` → `Color(.systemBackground)`
- Typography: `headlineSmall` (24sp) → `.headline` (17pt)
- Spacing: `16.dp` → `16pt` (1:1 for base unit)

**Example Use Case**: "What's the iOS equivalent of `MaterialTheme.colorScheme.primary`?"
**Answer**: `.accentColor` or custom `Color("Primary")`

---

### 3. Accessibility Testing Checklist
**File**: `./references/accessibility-testing-checklist.md`
**Use When**: Writing test plans, conducting manual accessibility testing, setting up automated a11y audits

**Covers**:
- Automated testing (Axe-Core Mobile, Lint checks, GitHub Actions setup)
- Manual testing with screen readers (TalkBack, VoiceOver)
- Keyboard navigation testing
- Contrast verification (automated + manual)
- Touch target verification (48dp/44pt)
- Color blindness simulation
- Text scaling tests (200% on Android, Dynamic Type AX1-AX7 on iOS)
- Form error announcements
- Test report template

**Key Workflows**:
- **Phase 1**: Static analysis (Lint, Xcode audit)
- **Phase 2**: Screen reader testing (TalkBack/VoiceOver)
- **Phase 3**: Keyboard navigation
- **Phase 4**: Contrast & color testing
- **Phase 5**: Text scaling
- **Phase 6**: Interactive elements
- **Phase 7**: Error handling & feedback
- **Phase 8**: Touch targets
- **Phase 9**: Test report

**Example Use Case**: "Set up automated accessibility testing in CI/CD"
**Answer**: See Phase 1.3 for GitHub Actions workflow

---

### 4. Platform Adaptation Rules
**File**: `./references/platform-adaptation-rules.md`
**Use When**: Adapting code from one platform to another (not literal translation)

**Covers**:
- Layout adaptation (Column/Row ↔ VStack/HStack)
- ScrollView/LazyColumn efficiency patterns
- State management (remember ↔ @State, StateFlow ↔ @ObservedObject)
- Modifier/View modifier patterns
- Accessibility semantics (contentDescription ↔ accessibilityLabel)
- Navigation (NavController ↔ NavigationStack)
- Theme & styling (MaterialTheme ↔ semantic colors)
- Data binding (TextField APIs)
- Image & media handling
- Animation patterns
- Platform-specific features (Safe Area, haptic feedback)

**Core Principle**: **Idiomatic Adaptation, Not Literal Translation**
- Wrong: Copy Compose code, change to SwiftUI syntax
- Right: Adapt code to follow each platform's idioms and design language

**Quick Checklists**:
- Android → iOS checklist (11 items)
- iOS → Android checklist (11 items)
- Common mistakes to avoid (8 items)

**Example Use Case**: "How do I convert a Compose Column to SwiftUI?"
**Answer**: `Column` → `VStack`, `.padding(16.dp)` → `.padding(16)`, `Arrangement.spacedBy(8.dp)` → `spacing: 8`

---

### 5. UI State Patterns (Loading, Empty, Error)
**File**: `./references/ui-states.md`
**Use When**: Implementing data-driven screens, handling API responses, designing "unhappy path" UX

**Covers**:
- Why LLMs consistently miss the "unhappy path" (assuming instant API success)
- Loading state with skeleton/shimmer patterns (NOT just "Loading..." text)
- Empty state design (illustration + title + subtitle + CTA)
- Error state with retry mechanism
- State machine approach using sealed class/enum (NOT multiple booleans)
- Pull-to-refresh integration that preserves existing data
- Snackbar/toast patterns for non-blocking errors

**Key Sections**:
- Skeleton/Shimmer UI code for both Android (Compose) and iOS (SwiftUI)
- EmptyState component with context-specific examples (My Events, Messages, Search)
- ErrorState component with retry button
- UiState sealed class pattern (Loading, Success, Error, Empty)
- Why `sealed class` over multiple booleans (prevents impossible states)

**Checklist**:
- [ ] Loading state shows skeleton/shimmer (not just spinner)
- [ ] Empty state has illustration + title + subtitle + CTA
- [ ] Error state has retry button and clear message
- [ ] State handled via sealed class/enum (not booleans)
- [ ] Pull-to-refresh preserves existing data

**Example Use Case**: "My screen only shows content when API succeeds - what am I missing?"
**Answer**: You need Loading (skeleton), Empty, and Error states. See ui-states.md for the state machine pattern.

---

## Skill Integration

### Android Design Skill
**Location**: `/source/skills/android-design/`
**Uses Shared References**:
- WCAG 22 Mobile Rules (for accessibility requirements)
- Cross-Platform Token Mapping (Android column of mappings)
- Platform Adaptation Rules (iOS to Android conversions)

### iOS Design Skill
**Location**: `/source/skills/ios-design/`
**Uses Shared References**:
- WCAG 22 Mobile Rules (for accessibility requirements)
- Cross-Platform Token Mapping (iOS column of mappings)
- Platform Adaptation Rules (Android to iOS conversions)

### Future Skills
**Potential Skills Using Shared References**:
- `/adapt-android-to-ios` - Automated code adaptation
- `/adapt-ios-to-android` - Automated code adaptation
- `/a11y-audit` - Cross-platform accessibility audits

---

## Key Metrics Reference

### Spacing Scale (Base 4dp/pt)
```
xs: 4dp/pt    - Inline spacing
sm: 8dp/pt    - Related elements
md: 12dp/pt   - Group spacing
lg: 16dp/pt   - Section spacing
xl: 24dp/pt   - Major divisions
2xl: 32dp/pt  - Page margins
3xl: 48dp/pt  - Large gaps
4xl: 64dp/pt  - Maximum margins
```

### Touch Targets
```
Android: 48x48dp minimum (Material Design 3)
iOS: 44x44pt minimum (Human Interface Guidelines)
Spacing: 8dp/pt minimum between targets
WCAG: 24x24 CSS px (equivalent to 48dp/44pt)
```

### Contrast Ratios (WCAG 2.2 AA)
```
Normal text: 4.5:1
Large text (18pt+, 14pt bold+): 3:1
UI components: 3:1
Disabled: 2:1 (acceptable)
Focus indicators: 3:1
```

### Typography Scale Mapping
```
Material displayMedium (45sp) → iOS .title (28pt)
Material bodyLarge (16sp) → iOS .body (17pt)
Material bodySmall (12sp) → iOS .caption (12pt)
Material labelLarge (14sp) → iOS .body (bold)
```

---

## How to Use These References

### For Android Developers
1. **Accessibility**: Read "WCAG 2.2 Mobile Rules" → Section 2.1 (Android-specific)
2. **Colors**: Check "Cross-Platform Token Mapping" → Section 1 (right column)
3. **Layouts**: Reference "Platform Adaptation Rules" → Section 1.1 (for iOS context)
4. **Testing**: Use "Accessibility Testing Checklist" → Phase 2 (TalkBack section)

### For iOS Developers
1. **Accessibility**: Read "WCAG 2.2 Mobile Rules" → Section 2.2 (iOS-specific)
2. **Colors**: Check "Cross-Platform Token Mapping" → Section 1 (right column)
3. **Layouts**: Reference "Platform Adaptation Rules" → Section 1.1 (iOS side)
4. **Testing**: Use "Accessibility Testing Checklist" → Phase 2.2 (VoiceOver section)

### For Cross-Platform Teams
1. **Token Consistency**: Use "Cross-Platform Token Mapping" for all design decisions
2. **Accessibility Parity**: Use "WCAG 2.2 Mobile Rules" as acceptance criteria
3. **Code Adaptation**: Follow "Platform Adaptation Rules" (not literal translation)
4. **Quality**: Use "Accessibility Testing Checklist" for all platforms before merge

---

## File Structure

```
/shared/
├── INDEX.md (this file)
└── /references/
    ├── wcag-22-mobile-rules.md
    ├── cross-platform-token-mapping.md
    ├── accessibility-testing-checklist.md
    ├── platform-adaptation-rules.md
    └── ui-states.md
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.1 | Feb 2026 | Added UI State Patterns reference (loading, empty, error states) |
| 1.0 | Feb 2025 | Initial release: WCAG rules, token mappings, testing checklist, adaptation rules |

---

## Related Resources

### External References
- **Axe-Core Android**: https://github.com/dequelabs/axe-core-android
- **Axe-Core iOS**: https://github.com/dequelabs/axe-core-ios
- **Material Design 3**: https://m3.material.io/
- **Apple HIG**: https://developer.apple.com/design/human-interface-guidelines/
- **WCAG 2.2**: https://www.w3.org/WAI/WCAG22/quickref/

### Internal Skills
- `/android-design` - Material Design 3 audit & enforcement
- `/ios-design` - HIG audit & enforcement

---

## Maintenance

### When to Update These References
- New WCAG 2.2 AA criteria released
- Material Design 3 specification changes
- Apple HIG updates
- New platform versions (Android OS, iOS) with accessibility implications
- Axe-Core updates

### How to Contribute
1. Update relevant reference file
2. Cross-check with external standards (WCAG, Material Design, HIG)
3. Include code examples for both platforms
4. Update version history above
5. Notify Kiln plugin maintainers


---
name: audit-accessibility
description: Audit mobile code for WCAG 2.2 Level AA accessibility compliance
args:
  - name: code
    type: string
    description: Mobile code (Compose or SwiftUI) to audit
    required: true
  - name: platform
    type: string
    enum: [android, ios, auto]
    description: Target platform (default: auto-detect)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# WCAG 2.2 Level AA Accessibility Audit

Comprehensive accessibility audit for mobile applications targeting WCAG 2.2 Level AA compliance.

## Instructions

Analyze the code for accessibility compliance. Auto-detect platform based on syntax (`@Composable` = Android, `some View` = iOS).

## WCAG 2.2 Criteria (Mobile-Relevant)

### Touch Target Size (Critical)

**2.5.8 Target Size (Minimum)** - Level AA
- Minimum 24×24 CSS pixels (24×24dp Android, 24×24pt iOS)
- Recommended: 48×48dp (Android), 44×44pt (iOS)

**Audit Points:**
- [ ] All interactive elements meet minimum 24×24 size
- [ ] Recommended targets are 48×48dp / 44×44pt
- [ ] Minimum 8dp/pt spacing between adjacent targets

### Color Contrast (Critical)

**1.4.3 Contrast (Minimum)** - Level AA
- Normal text: 4.5:1 ratio against background
- Large text (24sp+/18pt bold+): 3:1 ratio
- UI components and graphics: 3:1 ratio

**Audit Points:**
- [ ] Text on colored backgrounds meets contrast ratio
- [ ] Icons and graphics have sufficient contrast
- [ ] Interactive element boundaries are visible

### Text Alternatives (Critical)

**1.1.1 Non-text Content** - Level A
- All images have text alternatives
- Decorative images are hidden from assistive technology

**Audit Points:**
- [ ] All `Image`/`Icon` have content description (Android) or accessibilityLabel (iOS)
- [ ] Decorative images use null description or `.accessibilityHidden(true)`
- [ ] Complex images have extended descriptions

### Focus Visibility (Important)

**2.4.7 Focus Visible** - Level AA
- Focus indicator is visible on all interactive elements
- **2.4.11 Focus Not Obscured (Minimum)** - Level AA (WCAG 2.2)
- Focus indicator must not be completely hidden

**Audit Points:**
- [ ] Focus indicators are visible
- [ ] Focus is not obscured by sticky headers or overlays
- [ ] Custom focus indicators meet 3:1 contrast

### Input Modalities (Important)

**2.5.1 Pointer Gestures** - Level A
- Multi-point or path-based gestures have single-pointer alternatives

**2.5.7 Dragging Movements** - Level AA (WCAG 2.2)
- Drag operations have single-pointer alternatives

**Audit Points:**
- [ ] Swipe actions have button alternatives
- [ ] Drag-to-reorder has accessible alternative
- [ ] Pinch-to-zoom is not required (zoom controls available)

### Text Resizing (Important)

**1.4.4 Resize Text** - Level AA
- Content readable at 200% text size
- No loss of functionality at large sizes

**Audit Points:**
- [ ] Using `sp` (Android) / Dynamic Type (iOS) for text
- [ ] Layout doesn't break at largest accessibility sizes
- [ ] No text truncation that hides meaning

### Accessible Authentication (Important)

**3.3.8 Accessible Authentication (Minimum)** - Level AA (WCAG 2.2)
- No cognitive function tests required for authentication
- Alternatives available (copy-paste, password managers)

**Audit Points:**
- [ ] Text fields allow paste
- [ ] No CAPTCHA without accessible alternative
- [ ] Biometric authentication available as alternative

### Consistent Help (Important)

**3.2.6 Consistent Help** - Level A (WCAG 2.2)
- Help mechanisms appear in same relative order across pages

**Audit Points:**
- [ ] Help button in consistent location
- [ ] Contact information in consistent location

### Redundant Entry (Important)

**3.3.7 Redundant Entry** - Level A (WCAG 2.2)
- Auto-fill previously entered information
- Don't ask for same info twice in same process

**Audit Points:**
- [ ] Form fields support autofill
- [ ] Previously entered data auto-populated

## Platform-Specific Checks

### Android (Jetpack Compose)
- [ ] `contentDescription` on all interactive `Icon` and `Image`
- [ ] `Modifier.semantics` for custom accessibility
- [ ] `Modifier.minimumInteractiveComponentSize()` on small clickables
- [ ] Proper `Role` assignments (Button, Tab, Checkbox, etc.)
- [ ] `stateDescription` for toggle states
- [ ] Using `sp` for text sizes
- [ ] Testing with TalkBack

### iOS (SwiftUI)
- [ ] `.accessibilityLabel` on all interactive elements
- [ ] `.accessibilityHidden(true)` on decorative elements
- [ ] `.accessibilityAddTraits` for roles (.isButton, .isHeader)
- [ ] `.accessibilityValue` for dynamic content
- [ ] Using `.font(.body)` not `.font(.system(size:))`
- [ ] 44×44pt minimum touch targets
- [ ] Testing with VoiceOver

## Code to Audit

```
{{code}}
```

## Output Format

```markdown
## Accessibility Audit: [Component Name]

### WCAG 2.2 Compliance Summary

| Criterion | Status | Issues |
|-----------|--------|--------|
| 2.5.8 Target Size | ⚠️ | 2 violations |
| 1.4.3 Contrast | ✅ | 0 violations |
| 1.1.1 Non-text Content | ❌ | 3 violations |
| 2.4.7 Focus Visible | ✅ | 0 violations |
| 2.5.7 Dragging | N/A | No drag operations |

### Critical Violations (🔴)

**WCAG 1.1.1**: Image without text alternative
```kotlin
// Line 24
Icon(Icons.Default.Settings, contentDescription = null)

// Fix:
Icon(Icons.Default.Settings, contentDescription = "Settings")
```

**WCAG 2.5.8**: Touch target too small (32×32dp, minimum 48×48dp recommended)
```kotlin
// Line 45
Icon(
    modifier = Modifier.size(32.dp).clickable { }
)

// Fix:
IconButton(onClick = { }) {
    Icon(Icons.Default.Close, contentDescription = "Close")
}
```

### Important Issues (🟠)

**WCAG 1.4.4**: Hardcoded text size won't scale
```kotlin
// Line 18
Text(fontSize = 14.sp)

// Fix:
Text(style = MaterialTheme.typography.bodyMedium)
```

### Screen Reader Experience

**Reading Order:**
1. ✅ Header announced first
2. ⚠️ Decorative icon announced unnecessarily
3. ✅ Content announced correctly
4. ❌ Action buttons missing labels

**Recommendations:**
1. Add semantics role to card: `Modifier.semantics { role = Role.Button }`
2. Group related elements: `Modifier.semantics(mergeDescendants = true)`

### Summary
- **Level AA Compliance**: 75%
- Critical: 2 | Important: 1 | Warning: 0
- Recommended for screen reader testing before release

### Priority Actions
1. Add contentDescription to all interactive icons
2. Ensure 48×48dp minimum touch targets
3. Replace hardcoded text sizes with typography tokens
4. Test complete flow with TalkBack/VoiceOver
```

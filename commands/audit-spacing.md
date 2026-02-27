---
name: audit-spacing
description: Audit touch targets and spacing scale compliance
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
---

# Spacing & Touch Targets Audit

Comprehensive audit of spacing, touch targets, and layout compliance.

## Touch Target Requirements

| Standard | Minimum | Recommended | Notes |
|----------|---------|-------------|-------|
| **WCAG 2.2 AA** | 24×24 CSS px | 44×44 CSS px | Level AA requirement |
| **Material 3** | 48×48 dp | 48×48 dp | Hard requirement |
| **iOS HIG** | 44×44 pt | 44×44 pt | Minimum for comfortable tapping |
| **Spacing** | 8dp/pt | 8dp/pt | Minimum between targets |

## Spacing Scale

### Android (4dp Base Grid)
| Name | Value | Use Case |
|------|-------|----------|
| xs | 4dp | Inline elements |
| sm | 8dp | Related elements |
| md | 12dp | Group spacing |
| lg | 16dp | Section spacing |
| xl | 24dp | Major divisions |
| 2xl | 32dp | Page margins |
| 3xl | 48dp | Large gaps |
| 4xl | 64dp | Hero spacing |

### iOS (4pt Base Grid)
| Name | Value | Use Case |
|------|-------|----------|
| xs | 4pt | Inline elements |
| sm | 8pt | Related elements |
| md | 12pt | Group spacing |
| lg | 16pt | Section spacing (compact) |
| xl | 20pt | Section spacing (regular) |
| 2xl | 24pt | Major divisions |
| 3xl | 32pt | Large gaps |

## Audit Checklist

### Touch Targets (Critical)
- [ ] All buttons are 48×48dp / 44×44pt minimum
- [ ] All icons with click handlers meet minimum size
- [ ] IconButton or equivalent used for icon taps
- [ ] `minimumInteractiveComponentSize()` applied when needed
- [ ] Adjacent targets have 8dp/pt minimum spacing

### Spacing Grid (Important)
- [ ] Using 4dp/pt base unit
- [ ] No "magic numbers" (17dp, 13pt, etc.)
- [ ] Consistent horizontal margins (16-24dp/pt)
- [ ] Consistent vertical spacing between elements

### Safe Areas (iOS)
- [ ] Content respects Safe Area insets
- [ ] Background colors extend into Safe Areas
- [ ] Bottom navigation accounts for home indicator
- [ ] Top content accounts for notch/Dynamic Island

### Layout Patterns
- [ ] Cards have consistent internal padding (16dp typical)
- [ ] List items have consistent height
- [ ] Dividers aligned properly

## Code to Audit

```
{{code}}
```

## Output Format

```markdown
## Spacing Audit: [Component Name]

### Touch Target Analysis

| Element | Size | Required | Status | Fix |
|---------|------|----------|--------|-----|
| Close button | 24×24dp | 48×48dp | ❌ | Add IconButton |
| Favorite icon | 48×48dp | 48×48dp | ✅ | - |
| Menu item | 48×64dp | 48×48dp | ✅ | - |

### Issues

**Line 24**: Touch target too small (24×24dp)
```kotlin
// BAD
Icon(
    imageVector = Icons.Default.Close,
    modifier = Modifier
        .size(24.dp)
        .clickable { onClose() }
)

// GOOD
IconButton(onClick = onClose) {
    Icon(
        imageVector = Icons.Default.Close,
        contentDescription = "Close"
    )
}
```
Severity: 🔴 Critical | WCAG 2.5.8 violation

**Line 45**: Non-standard spacing (17dp)
```kotlin
// BAD
.padding(17.dp)

// GOOD
.padding(16.dp)  // Standard M3 spacing
```
Severity: 🟡 Warning

**Line 62**: Adjacent targets too close (4dp spacing)
```kotlin
// BAD
Row {
    IconButton { /* ... */ }
    Spacer(Modifier.width(4.dp))  // Too close!
    IconButton { /* ... */ }
}

// GOOD
Row {
    IconButton { /* ... */ }
    Spacer(Modifier.width(8.dp))  // Minimum 8dp
    IconButton { /* ... */ }
}
```
Severity: 🟠 Important

### Spacing Grid Compliance

| Usage | Value | Standard | Status |
|-------|-------|----------|--------|
| Card padding | 16dp | 16dp | ✅ |
| Section gap | 24dp | 24dp | ✅ |
| Inline spacing | 17dp | 16dp | ⚠️ |

### Recommendations

1. Wrap all clickable icons in `IconButton` for proper touch targets
2. Replace 17dp with 16dp to follow 4dp grid
3. Increase icon button spacing from 4dp to 8dp
4. Test with accessibility touch targets enabled
```

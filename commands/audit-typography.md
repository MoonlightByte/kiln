---
name: audit-typography
description: Audit typography scale and text accessibility compliance
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

# Typography Audit

Comprehensive audit of typography usage against Material Design 3 and Human Interface Guidelines.

## Instructions

Analyze the code for proper typography usage. Auto-detect platform from syntax.

## Android (Material Design 3) Typography Scale

| Token | Size | Line Height | Weight | Use Case |
|-------|------|-------------|--------|----------|
| `displayLarge` | 57sp | 64sp | 400 | Hero headlines |
| `displayMedium` | 45sp | 52sp | 400 | Page titles |
| `displaySmall` | 36sp | 44sp | 400 | Section headers |
| `headlineLarge` | 32sp | 40sp | 400 | Card titles |
| `headlineMedium` | 28sp | 36sp | 400 | Dialog titles |
| `headlineSmall` | 24sp | 32sp | 400 | List headers |
| `titleLarge` | 22sp | 28sp | 400 | App bar titles |
| `titleMedium` | 16sp | 24sp | 500 | Tab labels |
| `titleSmall` | 14sp | 20sp | 500 | Chip labels |
| `bodyLarge` | 16sp | 24sp | 400 | Body text |
| `bodyMedium` | 14sp | 20sp | 400 | Secondary text |
| `bodySmall` | 12sp | 16sp | 400 | Captions |
| `labelLarge` | 14sp | 20sp | 500 | Buttons |
| `labelMedium` | 12sp | 16sp | 500 | Small buttons |
| `labelSmall` | 11sp | 16sp | 500 | Badges |

## iOS (Dynamic Type) Typography Scale

| Style | Default Size | Supports Scaling | Use Case |
|-------|--------------|------------------|----------|
| `.largeTitle` | 34pt | ✅ | Hero headlines |
| `.title` | 28pt | ✅ | Page titles |
| `.title2` | 22pt | ✅ | Section headers |
| `.title3` | 20pt | ✅ | Subsection headers |
| `.headline` | 17pt (semibold) | ✅ | Emphasized body |
| `.body` | 17pt | ✅ | Main content |
| `.callout` | 16pt | ✅ | Secondary content |
| `.subheadline` | 15pt | ✅ | Supporting text |
| `.footnote` | 13pt | ✅ | Metadata |
| `.caption` | 12pt | ✅ | Captions |
| `.caption2` | 11pt | ✅ | Small labels |

## Audit Checklist

### Semantic Tokens (Critical)
- [ ] Using typography tokens instead of hardcoded sizes
- [ ] Correct token for context (headings vs body vs labels)
- [ ] Consistent usage across similar elements

### Accessibility (Critical)
- [ ] Text uses scalable units (sp/Dynamic Type)
- [ ] Minimum 11sp/11pt for readable text
- [ ] Large text (24sp+) for critical information
- [ ] Proper contrast ratio (4.5:1 normal, 3:1 large)

### Platform Compliance
**Android:**
- [ ] Using `MaterialTheme.typography.*`
- [ ] Not using `fontSize = X.sp` directly
- [ ] Not using `fontWeight` without typography token

**iOS:**
- [ ] Using `.font(.body)` style text styles
- [ ] Not using `.font(.system(size:))`
- [ ] Using `@ScaledMetric` for custom sizes

### Layout
- [ ] Proper line height (M3: ~120% of font size)
- [ ] Consistent vertical rhythm
- [ ] No truncation that hides critical information

## Code to Audit

```
{{code}}
```

## Output Format

```markdown
## Typography Audit: [Component Name]

### Token Usage Summary

| Element | Current | Should Be | Status |
|---------|---------|-----------|--------|
| Screen title | 24.sp hardcoded | headlineSmall | ⚠️ |
| Body text | bodyMedium | bodyMedium | ✅ |
| Button label | 12.sp hardcoded | labelLarge | ⚠️ |

### Issues

**Line 15**: Hardcoded font size
```kotlin
// BAD
Text(fontSize = 24.sp, fontWeight = FontWeight.Bold)

// GOOD
Text(style = MaterialTheme.typography.headlineSmall)
```
Severity: 🟠 Important

**Line 32**: Wrong typography token for context
```kotlin
// BAD - Using body style for a heading
Text("Section Title", style = MaterialTheme.typography.bodyLarge)

// GOOD - Use appropriate heading style
Text("Section Title", style = MaterialTheme.typography.titleLarge)
```
Severity: 🟡 Warning

### Accessibility Check

- ✅ Minimum font size: 12sp (meets 11sp minimum)
- ⚠️ Scaling support: 2 hardcoded sizes won't scale
- ✅ Contrast: Verified (requires visual inspection)

### Recommendations

1. Replace all hardcoded font sizes with typography tokens
2. Use `titleMedium` for section headers, not `bodyLarge`
3. Test with 200% font scale to verify layout doesn't break
```

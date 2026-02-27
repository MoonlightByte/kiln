---
name: verify-design-tokens
description: Comprehensive design token verification and linting for design system compliance
args:
  - name: path
    type: string
    description: Path to code directory or file to analyze (supports glob patterns like **/*.kt)
    required: true
  - name: tokens_path
    type: string
    description: Path to design tokens JSON file (W3C DTCG format) or token definitions
    required: false
  - name: report_format
    type: string
    enum: [json, html, markdown, sarif]
    description: Output format for verification report (default: markdown)
    required: false
  - name: strict_mode
    type: boolean
    description: Fail on warnings (not just errors) - enforces strict compliance
    required: false
  - name: fix
    type: boolean
    description: Attempt auto-fixes for common violations
    required: false
  - name: platforms
    type: string
    enum: [all, android, ios, web]
    description: Target platforms to verify (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Design Token Verification & Linting

Verify design system compliance by detecting hardcoded values, validating token naming conventions, ensuring cross-platform consistency, and measuring token usage coverage.

## Instructions

Analyze provided code against design token standards. Use the `android-design` and `ios-design` skills for platform-specific token definitions. Validate against W3C Design Tokens Community Group (DTCG) specification where applicable.

## Verification Checklist

### Color Token Compliance (VT-001 to VT-010)

- [ ] **VT-001**: No hardcoded RGB/hex colors outside token definitions
- [ ] **VT-002**: All color.* tokens follow semantic naming (primary, secondary, error, success, etc.)
- [ ] **VT-003**: Color values match W3C DTCG color spec format
- [ ] **VT-004**: Colors have accessibility contrast pairs (WCAG AA minimum)
- [ ] **VT-005**: Dark mode variants exist for all color tokens
- [ ] **VT-006**: No direct color references (e.g., `#FF0000`) in component code
- [ ] **VT-007**: Color naming follows intent-based convention (not `blue-100`, use `color.primary`)
- [ ] **VT-008**: All custom colors in Figma linked to design tokens
- [ ] **VT-009**: Alpha/opacity variants documented with parent token
- [ ] **VT-010**: Color contrast verified for light/dark themes

### Spacing Token Compliance (VT-011 to VT-015)

- [ ] **VT-011**: All spacing values use token scale (4dp, 8dp, 12dp, 16dp, 24dp, 32dp, etc.)
- [ ] **VT-012**: No hardcoded pixel/dp/rem values in margin/padding
- [ ] **VT-013**: Spacing tokens follow naming pattern: `spacing.xs`, `spacing.sm`, `spacing.md`, etc.
- [ ] **VT-014**: Line height uses token scale (120%, 140%, 160%, 180%)
- [ ] **VT-015**: Gap/gutter values use token scale consistently

### Typography Token Compliance (VT-016 to VT-022)

- [ ] **VT-016**: All text uses typography.* token (displayLarge, headlineSmall, bodyMedium, etc.)
- [ ] **VT-017**: No hardcoded font sizes in code
- [ ] **VT-018**: Font families defined as tokens (only system fonts or licensed)
- [ ] **VT-019**: Font weights follow token scale (regular: 400, medium: 500, bold: 700)
- [ ] **VT-020**: Letter spacing defined as tokens where needed (tracking)
- [ ] **VT-021**: Line height linked to typography token
- [ ] **VT-022**: Typography tokens include fallback font stacks

### Border & Radius Compliance (VT-023 to VT-028)

- [ ] **VT-023**: No hardcoded border radius values
- [ ] **VT-024**: Border radius uses token scale (4dp, 8dp, 12dp, 16dp, 28dp max)
- [ ] **VT-025**: Border width uses token scale (1dp, 2dp, 4dp)
- [ ] **VT-026**: Border colors use color tokens
- [ ] **VT-027**: Corner radius tokens follow naming: `shape.none`, `shape.extraSmall`, `shape.large`
- [ ] **VT-028**: Semantic shape tokens defined (e.g., `shape.button`, `shape.card`, `shape.dialog`)

### Shadow & Elevation Compliance (VT-029 to VT-032)

- [ ] **VT-029**: All shadows use elevation/shadow tokens
- [ ] **VT-030**: Elevation scale consistent across platforms (0dp to 24dp)
- [ ] **VT-031**: Shadow tokens include color, blur, offset, opacity
- [ ] **VT-032**: Material 3 tonal elevation colors match platform

### Token Usage & Coverage (VT-033 to VT-035)

- [ ] **VT-033**: Token usage coverage >= 95% (measure hardcoded vs. token references)
- [ ] **VT-034**: All token definitions have usage examples documented
- [ ] **VT-035**: Unused tokens identified and marked for removal

### Theme Variant Coverage (VT-036 to VT-040)

- [ ] **VT-036**: Light theme tokens defined and complete
- [ ] **VT-037**: Dark theme tokens defined and complete
- [ ] **VT-038**: High contrast theme (if applicable) tokens defined
- [ ] **VT-039**: Token variants for all supported color modes (light, dark, auto)
- [ ] **VT-040**: Theme token switching logic verified (no hardcoded fallbacks)

### Cross-Platform Consistency (VT-041 to VT-045)

- [ ] **VT-041**: Android Material 3 colors match iOS semantic colors
- [ ] **VT-042**: Spacing scales match across platforms (4dp base)
- [ ] **VT-043**: Typography scales match (both platforms use same point sizes conceptually)
- [ ] **VT-044**: Border radius scales consistent (no platform-specific overrides without doc)
- [ ] **VT-045**: Component token mappings documented (e.g., button uses `color.primary`)

---

## Lint Rules & Regex Patterns

### Critical Violations (Auto-Detect)

**Hardcoded RGB Colors**
```regex
# Kotlin
\b(?:Color|0x)[Ff](?:[0-9A-Fa-f]{6}|[0-9A-Fa-f]{8})\b
# Swift
UIColor\(red:|Color\(red:
# CSS/Web
#[0-9A-Fa-f]{3,8}\b|rgb\(|rgba\(
```

**Hardcoded Spacing Values (pixels)**
```regex
# Kotlin
\.padding\((?!.*dp).*\)|\.margin\((?!.*dp).*\)|Modifier\.size\(\d+[^d]
# Swift
\.padding\(\d+\)|.frame\(width: \d+\)|.frame\(height: \d+\)
# CSS
padding:\s*\d+px|margin:\s*\d+px
```

**Hardcoded Font Sizes**
```regex
# Kotlin
fontSize = (?!.*sp).*\d+|TextStyle\(fontSize = \d+\.sp\)
# Swift
\.font\(\.system\(size:\s*\d+\)|UIFont\(size:
# CSS
font-size:\s*\d+px
```

### Important Violations

**Semantic Color Naming Violations**
```regex
# Avoid technical color names in tokens
blue|red|green|yellow|purple|orange|pink|teal|cyan|magenta|indigo
# Instead use semantic names
primary|secondary|tertiary|error|success|warning|info|surface|background|outline
```

**Token Reference Patterns**

Valid token references:
```
# Kotlin (Material 3)
MaterialTheme.colorScheme.primary
MaterialTheme.typography.bodyMedium
MaterialTheme.shapes.medium

# Swift
Color(.primary)
Font.system(size: ...)

# CSS
var(--color-primary)
var(--spacing-md)
```

### Auto-Fix Suggestions

**Pattern**: `Color(0xFF1976D2)` → `MaterialTheme.colorScheme.primary`
```
1. Match hardcoded color hex
2. Lookup nearest token color in design tokens file
3. Replace with token reference
4. Log replacement for manual review (no silent fixes)
```

**Pattern**: `.padding(16)` → `.padding(16.dp)` → `.padding(dimensionResource(R.dimen.spacing_md))`
```
1. Ensure pixel value has dp suffix
2. Map numeric value to closest spacing token
3. Reference token instead of raw value
```

**Pattern**: `fontSize = 14.sp` → `MaterialTheme.typography.bodyMedium`
```
1. Match font size in sp
2. Lookup typography token with matching size
3. Replace with token reference
```

---

## Allowed Exceptions

These patterns are acceptable and should not be flagged:

```kotlin
// Platform-specific values (documented)
actual val platformSpacing = 8.dp  // iOS uses 8pt, Android uses 8dp

// Generated code (from code generators)
@Generated("design-token-codegen")

// Comments and documentation
// BAD: hardcoded color #FF0000
// GOOD: use MaterialTheme.colorScheme.error

// Constants defined in token file
object DesignTokens {
    val colorPrimary = Color(0xFF6200EE)
}

// Third-party library colors (with rationale comment)
// Using Mapbox default: color/0xFF3BB2D9
val mapboxBlue = Color(0xFF3BB2D9)  // Mapbox SDK requires this color

// Opacity/alpha modifiers on tokens
Color.primary.copy(alpha = 0.38f)  // Allowed: modifier on token, not hardcoded

// Feature flags for experimental tokens (temporary)
if (useExperimentalTokens) {
    MaterialTheme.colorScheme.primary
} else {
    Color(0xFF6200EE)  // Deprecated, scheduled for removal
}
```

---

## Report Output Formats

### Markdown (Default)

```markdown
# Design Token Verification Report

**Generated**: 2025-02-26 15:30 UTC
**Platform**: Android | iOS | Web
**Compliance Score**: 94%

## Summary
- Total violations: 12
- Critical: 1 | Important: 4 | Warning: 7 | Suggestion: 0
- Token usage coverage: 96%
- Unused tokens: 3
- Cross-platform inconsistencies: 2

## Critical Violations (🔴)

**VT-001**: Hardcoded color in EventCard.kt:45
```kotlin
// BAD
Text("Sold Out", color = Color(0xFFE53935))

// GOOD
Text("Sold Out", color = MaterialTheme.colorScheme.error)
```
Rule: VT-001 | Confidence: 98% | Severity: 🔴 Critical

---

## Important Violations (🟠)

**VT-006**: Direct color value in theme extension - SearchBar.kt:78
Rule: VT-006 | Confidence: 92% | Severity: 🟠 Important

---

## Warnings (🟡)

**VT-014**: Line height not linked to typography token - Dialog.kt:120
Rule: VT-014 | Confidence: 85% | Severity: 🟡 Warning

---

## Suggestions (🔵)

**VT-035**: Unused token detected: `color.onSurfaceDisabled`
Rule: VT-035 | Confidence: 100% | Severity: 🔵 Suggestion

---

## Token Usage Analysis

| Token Category | Defined | Used | Coverage |
|---|---|---|---|
| Colors | 45 | 43 | 95.6% |
| Spacing | 12 | 12 | 100% |
| Typography | 8 | 8 | 100% |
| Shapes | 5 | 5 | 100% |
| **Total** | **70** | **68** | **97.1%** |

---

## Cross-Platform Consistency

### Android vs iOS

| Token | Android | iOS | Match |
|---|---|---|---|
| color.primary | #6200EE | 6200EE | ✓ |
| spacing.md | 16dp | 16pt | ✓ |
| typography.bodyMedium | 14sp | 17pt | ⚠️ Different scales |

**Issue**: Typography not scaled consistently (14sp ≠ 17pt). Recommend: Use relative scaling or document platform differences.

---

## Theme Coverage

### Light Theme
- Status: ✓ Complete (45/45 tokens)
- Missing variants: None
- Accessibility: WCAG AA verified

### Dark Theme
- Status: ✓ Complete (45/45 tokens)
- Missing variants: None
- Accessibility: WCAG AA verified

### High Contrast Theme
- Status: ⚠️ Incomplete (38/45 tokens)
- Missing: color.tertiary, color.onTertiary, color.tertiaryContainer

---

## Recommended Actions

1. **Critical** (Fix immediately):
   - Replace hardcoded colors in EventCard.kt with theme tokens
   - Document exception for Mapbox library color

2. **Important** (Fix in next sprint):
   - Update SearchBar.kt to use theme color tokens
   - Audit all theme extensions for hardcoded values

3. **Nice to have**:
   - Complete high contrast theme variant
   - Remove 3 unused tokens
   - Document platform-specific typography scaling

---

## Files Analyzed

- EventCard.kt (15 issues)
- SearchBar.kt (3 issues)
- Dialog.kt (2 issues)
- Theme.kt (1 issue - passed)
- ...and 45 more files (clean)

---

## Token Definitions Reference

**Source**: tokens.json (W3C DTCG format)
**Last updated**: 2025-02-20
**Validation**: ✓ Compliant with DTCG spec v1.0

```json
{
  "color": {
    "primary": {
      "$type": "color",
      "$value": "#6200EE",
      "$description": "Primary brand color"
    }
  }
}
```

---

## CI/CD Integration Status

- [ ] GitHub Actions: Pre-commit linting enabled
- [ ] Figma sync: Token definitions auto-pulled
- [ ] Automated reports: Generated on every PR
- [ ] Slack notifications: Design system violations reported

---

## Related Documentation

- [W3C Design Tokens Specification](https://design-tokens.github.io/community-group/format/)
- Design System Token Guidelines: `.claude/plugins/kiln/docs/design-tokens.md`
- Platform-specific guides: `android-design`, `ios-design` skills
```

### JSON Format

```json
{
  "report": {
    "timestamp": "2025-02-26T15:30:00Z",
    "compliance_score": 94,
    "summary": {
      "total_violations": 12,
      "critical": 1,
      "important": 4,
      "warning": 7,
      "suggestion": 0,
      "token_usage_coverage_percent": 96,
      "unused_tokens": 3,
      "cross_platform_inconsistencies": 2
    },
    "violations": [
      {
        "id": "VT-001",
        "type": "hardcoded_color",
        "severity": "critical",
        "file": "EventCard.kt",
        "line": 45,
        "column": 12,
        "code": "Text(\"Sold Out\", color = Color(0xFFE53935))",
        "message": "Hardcoded color value found",
        "suggestion": "Use MaterialTheme.colorScheme.error",
        "confidence": 98,
        "auto_fix": "color = MaterialTheme.colorScheme.error"
      }
    ],
    "token_coverage": {
      "colors": { "defined": 45, "used": 43, "coverage_percent": 95.6 },
      "spacing": { "defined": 12, "used": 12, "coverage_percent": 100 },
      "typography": { "defined": 8, "used": 8, "coverage_percent": 100 }
    },
    "cross_platform": {
      "inconsistencies": [
        {
          "token": "typography.bodyMedium",
          "android": "14sp",
          "ios": "17pt",
          "note": "Platform-specific scaling - documented"
        }
      ]
    },
    "themes": {
      "light": { "status": "complete", "coverage_percent": 100 },
      "dark": { "status": "complete", "coverage_percent": 100 },
      "high_contrast": { "status": "incomplete", "coverage_percent": 84 }
    }
  }
}
```

### HTML Format

Interactive HTML report with:
- Color previews (light/dark swatches side-by-side)
- Violation highlighting with line numbers
- Token usage charts (pie/bar graphs)
- Theme comparison tables
- Clickable file navigation
- Severity filtering
- Export as PDF option

### SARIF Format (GitHub Code Scanning)

```json
{
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "design-token-linter",
          "version": "1.0.0",
          "rules": [
            {
              "id": "VT-001",
              "shortDescription": { "text": "Hardcoded color value" },
              "help": { "text": "Use design token reference instead" }
            }
          ]
        }
      },
      "results": [
        {
          "ruleId": "VT-001",
          "message": { "text": "Hardcoded color #FFE53935" },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": { "uri": "EventCard.kt" },
                "region": { "startLine": 45, "startColumn": 12 }
              }
            }
          ],
          "level": "error"
        }
      ]
    }
  ]
}
```

---

## CI/CD Integration Examples

### GitHub Actions Workflow

```yaml
name: Design Token Verification

on: [pull_request, push]

jobs:
  token-verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Verify Design Tokens
        run: |
          # Run verification with strict mode
          claude kiln verify-design-tokens \
            --path "android/app/src/**/*.kt" \
            --tokens_path "design-tokens.json" \
            --report_format sarif \
            --strict_mode true

      - name: Upload SARIF
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: design-tokens-report.sarif

      - name: Comment PR with Results
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const report = JSON.parse(fs.readFileSync('design-tokens-report.json'));
            const comment = `
            ## Design Token Verification
            - Compliance: ${report.summary.compliance_score}%
            - Issues: ${report.summary.total_violations} (Critical: ${report.summary.critical})
            [View full report]({{ github.server_url }}/{{ github.repository }}/actions/runs/{{ github.run_id }})
            `;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

### Pre-Commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Verify design tokens before committing
claude kiln verify-design-tokens \
  --path "android/app/src/**/*.kt" \
  --tokens_path "design-tokens.json" \
  --strict_mode true \
  --report_format json

if [ $? -ne 0 ]; then
  echo "Design token violations detected. Fix before committing."
  exit 1
fi
```

---

## Example Output

```markdown
## Audit Results: Event System Design Tokens

### Critical Issues (🔴)

**VT-001**: Hardcoded color in EventCard.kt:45
```kotlin
// BAD
Text("Sold Out", color = Color(0xFFE53935))

// GOOD
Text("Sold Out", color = MaterialTheme.colorScheme.error)
```
Severity: 🔴 Critical | Confidence: 98 | Rule: VT-001
Auto-fix: `color = MaterialTheme.colorScheme.error`

---

**VT-011**: Hardcoded spacing value in EventList.kt:120
```kotlin
// BAD
modifier = Modifier.padding(16)

// GOOD
modifier = Modifier.padding(
    dimensionResource(id = R.dimen.spacing_md)
)
```
Severity: 🔴 Critical | Confidence: 95 | Rule: VT-011
Auto-fix: Replace with spacing token reference

### Important Issues (🟠)

**VT-006**: Color reference not using theme token - SearchBar.kt:78
Severity: 🟠 Important | Confidence: 92 | Rule: VT-006

### Warnings (🟡)

**VT-014**: Typography line-height not linked to token - Dialog.kt:120
Severity: 🟡 Warning | Confidence: 85 | Rule: VT-014

### Summary

- Total issues: 12
- Critical: 1 | Important: 4 | Warning: 7 | Suggestion: 0
- Overall compliance: 94%
- Token usage coverage: 96%
- Cross-platform consistency: 100%
- Theme coverage: Light 100%, Dark 100%, High Contrast 84%

### Recommended Actions

1. Replace 1 hardcoded color (EventCard.kt:45)
2. Update 3 spacing references (use token scale)
3. Document platform-specific typography differences
4. Complete high-contrast theme variant (7 tokens)
5. Remove 3 unused tokens
```

---

## Notes

- **W3C Compliance**: Reports validate tokens against W3C Design Tokens Community Group specification
- **Figma Integration**: If using Tokens Studio, sync token definitions automatically
- **Token Lens**: Pair with [Token Lens](https://francescoimprota.com/2025/11/17/token-lens/) tool for CSS coverage analysis
- **Buoy Tool**: Consider [Buoy](https://buoy.design/) for PR-based drift detection
- **Anima Validator**: Use [Anima's Design Token Validator](https://animaapp.github.io/design-token-validator-site/) for JSON validation
- **SARIF Support**: Enables integration with GitHub Code Scanning and other SAST tools

Sources:
- [Design Token Management Tools 2025](https://cssauthor.com/design-token-management-tools/)
- [W3C Design Tokens Specification](https://www.w3.org/community/design-tokens/)
- [Design Token Enforcement Guide](https://medium.com/@barshaya97_76274/design-tokens-enforcement-977310b2788e)
- [Token Lens - Token Usage Analysis](https://francescoimprota.com/2025/11/17/token-lens/)
- [Buoy - Design Drift Detection](https://buoy.design/)
- [Anima Design Token Validator](https://animaapp.github.io/design-token-validator-site/)
- [Tokens Studio](https://tokens.studio/)

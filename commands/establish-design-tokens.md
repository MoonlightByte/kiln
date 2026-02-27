---
name: establish-design-tokens
description: Establish W3C-compliant design tokens with cross-platform theming and export pipelines
args:
  - name: output_dir
    type: string
    description: Output directory for tokens.json and generated files
    required: true
  - name: platforms
    type: string
    enum: [android, ios, web, all]
    description: Target platforms for token export
    required: false
  - name: themes
    type: string
    description: Comma-separated theme names (e.g., light,dark,high-contrast)
    required: false
---

# Establish Design Tokens

Establish a W3C Design Tokens Community Group (DTCG) compliant design system with semantic tokens, theme variants, and cross-platform code generation.

## Overview

This command guides you through creating a production-ready design token system following the official [W3C Design Tokens Format Module 2025.10](https://www.designtokens.org/tr/drafts/format/) specification. The system provides:

- **Single source of truth** for design decisions across platforms
- **Semantic naming** that scales with your design system
- **Token aliases and references** for managing complex relationships
- **Multi-theme support** (light/dark/high-contrast/brand variants)
- **Cross-platform code generation** for Compose, SwiftUI, CSS, and Figma
- **CI/CD validation** for consistency and completeness

### Why Design Tokens?

Design tokens solve the "cascade problem":
- Design changes in Figma require manual code updates
- Different platforms re-implement colors/spacing differently
- Theming becomes a copy-paste nightmare
- Onboarding new designers is error-prone

With design tokens, your design system is **declarative, version-controlled, and automatically synchronized** across all platforms.

## File Structure

```
design-tokens/
├── tokens.json                 # Main token file (W3C DTCG format)
├── tokens/
│   ├── color.json             # Color token definitions
│   ├── typography.json        # Font/text tokens
│   ├── spacing.json           # Spacing and sizing
│   ├── shadow.json            # Elevation and shadow
│   ├── motion.json            # Animation/transition tokens
│   └── border.json            # Border and corner radius
├── themes/
│   ├── light.json             # Light theme overrides
│   ├── dark.json              # Dark theme overrides
│   └── high-contrast.json     # Accessibility theme
├── build/
│   ├── android/
│   │   └── Theme.kt           # Generated Compose theme (auto)
│   ├── ios/
│   │   └── Theme.swift        # Generated SwiftUI environment (auto)
│   └── web/
│       └── tokens.css         # Generated CSS custom properties (auto)
└── scripts/
    ├── validate.js            # Validate tokens against schema
    ├── build.js               # Generate platform-specific code
    └── sync-figma.js          # Sync from Tokens Studio to repo
```

## Token Categories

### 1. Color Tokens

**Structure:** `{category}-{semantic}-{state}`

```json
{
  "color": {
    "$description": "Color palette",
    "primary": {
      "$description": "Brand primary color",
      "default": {
        "$type": "color",
        "$value": "#6200EA",
        "$description": "Primary action color"
      },
      "light": {
        "$type": "color",
        "$value": "#7C3DED",
        "$description": "Light variant for hover"
      },
      "dark": {
        "$type": "color",
        "$value": "#3700B3",
        "$description": "Dark variant for pressed"
      },
      "container": {
        "$type": "color",
        "$value": "#EADDFF",
        "$description": "Container background"
      }
    },
    "secondary": {
      "$description": "Secondary brand color",
      "default": {
        "$type": "color",
        "$value": "#03DAC6"
      },
      "container": {
        "$type": "color",
        "$value": "#84F8F0"
      }
    },
    "success": {
      "$type": "color",
      "$value": "#4CAF50",
      "$description": "Success state (green)"
    },
    "error": {
      "$type": "color",
      "$value": "#B3261E",
      "$description": "Error state (red)"
    },
    "warning": {
      "$type": "color",
      "$value": "#F9A825",
      "$description": "Warning state (amber)"
    },
    "surface": {
      "$type": "color",
      "$value": "#FFFBFE",
      "$description": "Default background"
    },
    "surface-variant": {
      "$type": "color",
      "$value": "#E7E0EC",
      "$description": "Secondary background"
    },
    "outline": {
      "$type": "color",
      "$value": "#79747E",
      "$description": "Border/divider color"
    },
    "on-surface": {
      "$type": "color",
      "$value": "#1C1B1F",
      "$description": "Text on surface"
    }
  }
}
```

**Token Naming Convention:**
| Pattern | Example | Usage |
|---------|---------|-------|
| `{brand}-{state}` | `primary-default`, `primary-light` | Brand colors |
| `{semantic}-{variant}` | `error-container`, `success-light` | Semantic colors |
| `on-{surface}` | `on-surface`, `on-primary-container` | Text/content colors |
| `surface` or `background` | `surface`, `surface-variant` | Background layers |

**Color Specification:** Support Display P3, Oklch, and CSS Color Module 4:
```json
{
  "color": {
    "accent": {
      "$type": "color",
      "$value": "oklch(66% 0.15 120)"  // Modern Oklch format
    }
  }
}
```

### 2. Typography Tokens

**Structure:** Defines font families, sizes, weights, and line heights

```json
{
  "typography": {
    "font-family": {
      "display": {
        "$type": "fontFamily",
        "$value": ["Roboto", "system-ui", "sans-serif"]
      },
      "body": {
        "$type": "fontFamily",
        "$value": ["Roboto", "system-ui", "sans-serif"]
      },
      "mono": {
        "$type": "fontFamily",
        "$value": ["Roboto Mono", "Courier", "monospace"]
      }
    },
    "font-size": {
      "display-large": {
        "$type": "dimension",
        "$value": "57sp"
      },
      "display-medium": {
        "$type": "dimension",
        "$value": "45sp"
      },
      "headline-large": {
        "$type": "dimension",
        "$value": "32sp"
      },
      "title-large": {
        "$type": "dimension",
        "$value": "22sp"
      },
      "body-large": {
        "$type": "dimension",
        "$value": "16sp"
      },
      "body-medium": {
        "$type": "dimension",
        "$value": "14sp"
      },
      "body-small": {
        "$type": "dimension",
        "$value": "12sp"
      },
      "label-large": {
        "$type": "dimension",
        "$value": "14sp"
      }
    },
    "font-weight": {
      "regular": {
        "$type": "number",
        "$value": 400
      },
      "medium": {
        "$type": "number",
        "$value": 500
      },
      "bold": {
        "$type": "number",
        "$value": 700
      }
    },
    "line-height": {
      "tight": {
        "$type": "number",
        "$value": 1.2
      },
      "normal": {
        "$type": "number",
        "$value": 1.5
      },
      "loose": {
        "$type": "number",
        "$value": 1.8
      }
    },
    "body-large": {
      "$type": "typography",
      "$value": {
        "fontFamily": "{typography.font-family.body}",
        "fontSize": "{typography.font-size.body-large}",
        "fontWeight": "{typography.font-weight.regular}",
        "lineHeight": "{typography.line-height.normal}"
      },
      "$description": "Primary body text style"
    }
  }
}
```

**Composite Typography Token:** Groups font properties together
```json
{
  "heading-1": {
    "$type": "typography",
    "$value": {
      "fontFamily": "{typography.font-family.display}",
      "fontSize": "32sp",
      "fontWeight": 700,
      "lineHeight": 1.2,
      "letterSpacing": "0sp"
    }
  }
}
```

### 3. Spacing Tokens

**Structure:** 4dp base grid for consistency

```json
{
  "spacing": {
    "0": {
      "$type": "dimension",
      "$value": "0px"
    },
    "xs": {
      "$type": "dimension",
      "$value": "4dp",
      "$description": "Extra small (inline elements)"
    },
    "sm": {
      "$type": "dimension",
      "$value": "8dp",
      "$description": "Small (related elements)"
    },
    "md": {
      "$type": "dimension",
      "$value": "12dp",
      "$description": "Medium (grouped elements)"
    },
    "lg": {
      "$type": "dimension",
      "$value": "16dp",
      "$description": "Large (sections)"
    },
    "xl": {
      "$type": "dimension",
      "$value": "24dp",
      "$description": "Extra large (major divisions)"
    },
    "2xl": {
      "$type": "dimension",
      "$value": "32dp"
    },
    "3xl": {
      "$type": "dimension",
      "$value": "48dp"
    }
  },
  "border-radius": {
    "none": {
      "$type": "dimension",
      "$value": "0px"
    },
    "xs": {
      "$type": "dimension",
      "$value": "4dp",
      "$description": "Minimal rounding"
    },
    "sm": {
      "$type": "dimension",
      "$value": "8dp",
      "$description": "Small corners"
    },
    "md": {
      "$type": "dimension",
      "$value": "12dp",
      "$description": "Medium corners"
    },
    "lg": {
      "$type": "dimension",
      "$value": "16dp",
      "$description": "Large corners (cards)"
    },
    "xl": {
      "$type": "dimension",
      "$value": "24dp",
      "$description": "Extra large corners"
    },
    "full": {
      "$type": "dimension",
      "$value": "999px",
      "$description": "Fully rounded (pills)"
    }
  }
}
```

### 4. Shadow/Elevation Tokens

**Structure:** Material Design 3 elevation system

```json
{
  "shadow": {
    "elevation": {
      "level-0": {
        "$type": "shadow",
        "$value": {
          "offsetX": "0px",
          "offsetY": "0px",
          "blur": "0px",
          "spread": "0px",
          "color": "rgba(0, 0, 0, 0)"
        },
        "$description": "No elevation (flat)"
      },
      "level-1": {
        "$type": "shadow",
        "$value": {
          "offsetX": "0px",
          "offsetY": "1px",
          "blur": "3px",
          "spread": "0px",
          "color": "rgba(0, 0, 0, 0.12)"
        },
        "$description": "Subtle lift"
      },
      "level-2": {
        "$type": "shadow",
        "$value": {
          "offsetX": "0px",
          "offsetY": "3px",
          "blur": "6px",
          "spread": "0px",
          "color": "rgba(0, 0, 0, 0.16)"
        },
        "$description": "Light card elevation"
      },
      "level-3": {
        "$type": "shadow",
        "$value": {
          "offsetX": "0px",
          "offsetY": "6px",
          "blur": "12px",
          "spread": "0px",
          "color": "rgba(0, 0, 0, 0.20)"
        },
        "$description": "Medium elevation"
      },
      "level-4": {
        "$type": "shadow",
        "$value": {
          "offsetX": "0px",
          "offsetY": "8px",
          "blur": "16px",
          "spread": "0px",
          "color": "rgba(0, 0, 0, 0.24)"
        },
        "$description": "High elevation (modals)"
      },
      "level-5": {
        "$type": "shadow",
        "$value": {
          "offsetX": "0px",
          "offsetY": "12px",
          "blur": "24px",
          "spread": "0px",
          "color": "rgba(0, 0, 0, 0.28)"
        },
        "$description": "Maximum elevation"
      }
    }
  }
}
```

### 5. Motion Tokens

**Structure:** Durations and easing for animations

```json
{
  "motion": {
    "duration": {
      "short": {
        "$type": "duration",
        "$value": "100ms",
        "$description": "Micro-interactions"
      },
      "medium": {
        "$type": "duration",
        "$value": "200ms",
        "$description": "Standard transitions"
      },
      "long": {
        "$type": "duration",
        "$value": "300ms",
        "$description": "Page transitions"
      },
      "extra-long": {
        "$type": "duration",
        "$value": "500ms",
        "$description": "Extended animations"
      }
    },
    "easing": {
      "standard": {
        "$type": "cubic-bezier",
        "$value": [0.2, 0, 0.8, 1],
        "$description": "Default easing"
      },
      "emphasized": {
        "$type": "cubic-bezier",
        "$value": [0.05, 0.7, 0.1, 1],
        "$description": "Emphasis/deceleration"
      },
      "decelerated": {
        "$type": "cubic-bezier",
        "$value": [0, 0, 0.2, 1],
        "$description": "Deceleration curve"
      },
      "accelerated": {
        "$type": "cubic-bezier",
        "$value": [0.3, 0, 1, 1],
        "$description": "Acceleration curve"
      }
    },
    "transition": {
      "standard": {
        "$type": "transition",
        "$value": {
          "duration": "{motion.duration.medium}",
          "delay": "0ms",
          "timingFunction": "{motion.easing.standard}"
        },
        "$description": "Standard transition"
      }
    }
  }
}
```

### 6. Border Tokens

```json
{
  "border": {
    "width": {
      "thin": {
        "$type": "dimension",
        "$value": "1dp"
      },
      "medium": {
        "$type": "dimension",
        "$value": "2dp"
      },
      "thick": {
        "$type": "dimension",
        "$value": "4dp"
      }
    },
    "style": {
      "solid": {
        "$type": "strokeStyle",
        "$value": "solid"
      },
      "dashed": {
        "$type": "strokeStyle",
        "$value": "dashed"
      }
    }
  }
}
```

## Semantic Naming Conventions

Use semantic names that describe **purpose**, not visual appearance:

| Anti-Pattern | Semantic Name | Usage |
|--------------|---------------|-------|
| `purple-500` | `primary` | Brand primary |
| `gray-200` | `surface-variant` | Secondary background |
| `16px-font` | `body-large` | Body text |
| `8px-margin` | `spacing-sm` | Related elements |
| `blue-on-hover` | `primary-light` | Interactive states |

**Semantic Naming Structure:**
```
{category}-{concept}-{state}
```

Example:
- `color-primary-default` (category-concept-state)
- `color-primary-light` (hover state)
- `spacing-md` (medium spacing)
- `motion-duration-medium` (animation duration)

## Token Aliases and References

Use aliases to build complex relationships and enable theme variants:

```json
{
  "color": {
    "primary": {
      "$type": "color",
      "$value": "#6200EA"
    },
    "primary-light": {
      "$type": "color",
      "$value": "#7C3DED"
    }
  },
  "component": {
    "button": {
      "background-default": {
        "$type": "color",
        "$value": "{color.primary}",
        "$description": "Button default background uses primary"
      },
      "background-hover": {
        "$type": "color",
        "$value": "{color.primary-light}",
        "$description": "Button hover uses light variant"
      }
    }
  }
}
```

**Cross-Token References:**
```json
{
  "button": {
    "$type": "typography",
    "$value": {
      "fontFamily": "{typography.font-family.body}",
      "fontSize": "{typography.font-size.label-large}",
      "fontWeight": "{typography.font-weight.medium}",
      "lineHeight": 1.5
    }
  }
}
```

## Theme Variants

Define theme overrides for light, dark, and high-contrast modes:

### Light Theme (base.json)

```json
{
  "color": {
    "surface": {
      "$type": "color",
      "$value": "#FFFBFE"
    },
    "on-surface": {
      "$type": "color",
      "$value": "#1C1B1F"
    },
    "surface-variant": {
      "$type": "color",
      "$value": "#E7E0EC"
    }
  }
}
```

### Dark Theme (themes/dark.json)

```json
{
  "color": {
    "surface": {
      "$type": "color",
      "$value": "#1C1B1F"
    },
    "on-surface": {
      "$type": "color",
      "$value": "#E7E0EC"
    },
    "surface-variant": {
      "$type": "color",
      "$value": "#49454E"
    }
  }
}
```

### High-Contrast Theme (themes/high-contrast.json)

```json
{
  "color": {
    "primary": {
      "$type": "color",
      "$value": "#0D0D0D",
      "$description": "Maximum contrast black"
    },
    "on-surface": {
      "$type": "color",
      "$value": "#FFFFFF",
      "$description": "Pure white for contrast"
    },
    "outline": {
      "$type": "color",
      "$value": "#000000",
      "$description": "Thicker outline for visibility"
    }
  },
  "border": {
    "width": {
      "medium": {
        "$type": "dimension",
        "$value": "3dp",
        "$description": "Increased for contrast"
      }
    }
  }
}
```

## Platform-Specific Export

### Android (Jetpack Compose)

**Generated:** `/build/android/Theme.kt`

```kotlin
package com.example.designsystem

import androidx.compose.material3.*
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

// Colors (from tokens.json)
val PrimaryColor = Color(0xFF6200EA)
val PrimaryLightColor = Color(0xFF7C3DED)
val SecondaryColor = Color(0xFF03DAC6)
val SurfaceColor = Color(0xFFFFFBFE)
val OnSurfaceColor = Color(0xFF1C1B1F)

// Spacing (from tokens.json)
val SpacingXs = 4.dp
val SpacingSm = 8.dp
val SpacingMd = 12.dp
val SpacingLg = 16.dp
val SpacingXl = 24.dp

// Border radius
val CornerRadiusSm = 8.dp
val CornerRadiusMd = 12.dp
val CornerRadiusLg = 16.dp
val CornerRadiusFull = 999.dp

// Typography
val HeadingLarge = TextStyle(
    fontFamily = FontFamily.Default,
    fontSize = 32.sp,
    fontWeight = FontWeight.Bold,
    lineHeight = 40.sp
)

val BodyLarge = TextStyle(
    fontFamily = FontFamily.Default,
    fontSize = 16.sp,
    fontWeight = FontWeight.Normal,
    lineHeight = 24.sp
)

// Elevation/Shadow
fun shadowElevation1() = ShadowElevation(
    elevation = 1.dp,
    shape = RoundedCornerShape(8.dp)
)

// Theme
private val LightColorScheme = lightColorScheme(
    primary = PrimaryColor,
    secondary = SecondaryColor,
    surface = SurfaceColor,
    onSurface = OnSurfaceColor
)

private val DarkColorScheme = darkColorScheme(
    primary = PrimaryLightColor,
    secondary = SecondaryColor,
    surface = Color(0xFF1C1B1F),
    onSurface = Color(0xFFF5EFF7)
)

@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit
) {
    val colorScheme = if (darkTheme) DarkColorScheme else LightColorScheme

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography(
            headlineLarge = HeadingLarge,
            bodyLarge = BodyLarge
        ),
        content = content
    )
}
```

### iOS (SwiftUI)

**Generated:** `/build/ios/Theme.swift`

```swift
import SwiftUI

// MARK: - Colors
struct Colors {
    static let primary = Color(red: 0.38, green: 0, blue: 0.92)
    static let primaryLight = Color(red: 0.49, green: 0.24, blue: 0.93)
    static let secondary = Color(red: 0.03, green: 0.85, blue: 0.78)
    static let surface = Color(red: 1.0, green: 0.98, blue: 0.99)
    static let onSurface = Color(red: 0.11, green: 0.10, blue: 0.12)
    static let error = Color(red: 0.70, green: 0.15, blue: 0.12)
}

// MARK: - Spacing
struct Spacing {
    static let xs: CGFloat = 4
    static let sm: CGFloat = 8
    static let md: CGFloat = 12
    static let lg: CGFloat = 16
    static let xl: CGFloat = 24
    static let xxl: CGFloat = 32
}

// MARK: - Corner Radius
struct CornerRadius {
    static let sm: CGFloat = 8
    static let md: CGFloat = 12
    static let lg: CGFloat = 16
    static let full: CGFloat = 999
}

// MARK: - Typography
struct Typography {
    static let headingLarge = Font.system(size: 32, weight: .bold)
    static let headingMedium = Font.system(size: 28, weight: .bold)
    static let bodyLarge = Font.system(size: 16, weight: .regular)
    static let bodyMedium = Font.system(size: 14, weight: .regular)
    static let labelLarge = Font.system(size: 14, weight: .semibold)
}

// MARK: - Elevation/Shadow
struct Elevation {
    static let level1 = Shadow(
        color: .black.opacity(0.12),
        radius: 3,
        x: 0,
        y: 1
    )
    static let level2 = Shadow(
        color: .black.opacity(0.16),
        radius: 6,
        x: 0,
        y: 3
    )
    static let level3 = Shadow(
        color: .black.opacity(0.20),
        radius: 12,
        x: 0,
        y: 6
    )
}

// MARK: - Theme Environment
struct ThemeKey: EnvironmentKey {
    static let defaultValue = Theme()
}

extension EnvironmentValues {
    var theme: Theme {
        get { self[ThemeKey.self] }
        set { self[ThemeKey.self] = newValue }
    }
}

struct Theme {
    var colors = Colors()
    var spacing = Spacing()
    var cornerRadius = CornerRadius()
    var isDarkMode = false
}

// MARK: - Theme Provider
struct ThemeProvider<Content: View>: View {
    @Environment(\.colorScheme) var colorScheme
    let content: Content

    var body: some View {
        let isDark = colorScheme == .dark
        let theme = Theme(isDarkMode: isDark)

        content
            .environment(\.theme, theme)
    }
}
```

### Web (CSS)

**Generated:** `/build/web/tokens.css`

```css
/* Design Tokens - CSS Custom Properties */

:root {
  /* Colors */
  --color-primary: #6200EA;
  --color-primary-light: #7C3DED;
  --color-primary-dark: #3700B3;
  --color-secondary: #03DAC6;
  --color-surface: #FFFBFE;
  --color-on-surface: #1C1B1F;
  --color-error: #B3261E;
  --color-success: #4CAF50;

  /* Spacing */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 12px;
  --spacing-lg: 16px;
  --spacing-xl: 24px;
  --spacing-2xl: 32px;

  /* Border Radius */
  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;
  --radius-full: 999px;

  /* Typography */
  --font-family-display: "Roboto", system-ui, sans-serif;
  --font-size-display-large: 57px;
  --font-size-heading-large: 32px;
  --font-size-body-large: 16px;
  --font-size-body-medium: 14px;
  --font-weight-regular: 400;
  --font-weight-medium: 500;
  --font-weight-bold: 700;
  --line-height-tight: 1.2;
  --line-height-normal: 1.5;

  /* Shadow/Elevation */
  --shadow-level-1: 0 1px 3px rgba(0, 0, 0, 0.12);
  --shadow-level-2: 0 3px 6px rgba(0, 0, 0, 0.16);
  --shadow-level-3: 0 6px 12px rgba(0, 0, 0, 0.20);

  /* Motion */
  --duration-short: 100ms;
  --duration-medium: 200ms;
  --duration-long: 300ms;
  --easing-standard: cubic-bezier(0.2, 0, 0.8, 1);
  --easing-emphasized: cubic-bezier(0.05, 0.7, 0.1, 1);
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-surface: #1C1B1F;
    --color-on-surface: #E7E0EC;
    --color-surface-variant: #49454E;
  }
}

@media (prefers-contrast: more) {
  :root {
    --color-primary: #0D0D0D;
    --color-on-surface: #FFFFFF;
    --border-width-medium: 3px;
  }
}

/* Sample Usage */
.button {
  background-color: var(--color-primary);
  color: #FFFFFF;
  padding: var(--spacing-md) var(--spacing-lg);
  border-radius: var(--radius-md);
  font-family: var(--font-family-display);
  font-size: var(--font-size-body-large);
  font-weight: var(--font-weight-medium);
  transition: background-color var(--duration-medium) var(--easing-standard);
  box-shadow: var(--shadow-level-2);
}
```

## Figma Integration

### Using Tokens Studio Plugin

1. **Install** [Tokens Studio for Figma](https://www.tokens.studio/)
2. **Configure GitHub sync:**
   - Settings → GitHub → Connect
   - Select your repo and branch
3. **Link tokens.json:**
   - Plugins → Tokens Studio → Settings
   - Set tokens file path: `design-tokens/tokens.json`
4. **Create components using tokens:**
   ```
   Fill color = {color.primary}
   Text style = {typography.body-large}
   Spacing = {spacing.md}
   ```
5. **Auto-sync to code:**
   - Studio → Export → GitHub commit
   - Tokens update in `tokens.json`

### Figma Variable Naming (Match DTCG)

```
color/primary
color/primary/light
color/primary/dark
spacing/md
radius/lg
typography/body-large
motion/duration/medium
```

## CI/CD Validation

### Validation Script (validate.js)

```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

// Load tokens.json
const tokensPath = path.join(__dirname, '../tokens.json');
const tokens = JSON.parse(fs.readFileSync(tokensPath, 'utf-8'));

const errors = [];
const warnings = [];

// Validate token structure
function validateToken(token, path = '') {
  if (!token.$type) {
    errors.push(`Missing $type at ${path}`);
  }
  if (!token.$value) {
    errors.push(`Missing $value at ${path}`);
  }

  // Validate color values
  if (token.$type === 'color') {
    if (!/^#[0-9A-Fa-f]{6}$/.test(token.$value) &&
        !/^oklch\(/.test(token.$value) &&
        !/^rgb/.test(token.$value)) {
      warnings.push(`Invalid color format at ${path}: ${token.$value}`);
    }
  }

  // Validate dimension values
  if (token.$type === 'dimension') {
    if (!/^\d+($|dp|px|pt|sp)$/.test(token.$value)) {
      errors.push(`Invalid dimension format at ${path}: ${token.$value}`);
    }
  }

  // Validate references
  if (token.$value && typeof token.$value === 'string' &&
      token.$value.startsWith('{')) {
    const refPath = token.$value.slice(1, -1);
    if (!getTokenByPath(refPath)) {
      errors.push(`Broken reference at ${path}: {${refPath}}`);
    }
  }
}

// Recursively validate all tokens
function walkTokens(obj, basePath = '') {
  for (const [key, value] of Object.entries(obj)) {
    const currentPath = basePath ? `${basePath}.${key}` : key;

    if (value.$type && value.$value) {
      validateToken(value, currentPath);
    } else if (typeof value === 'object' && !Array.isArray(value)) {
      walkTokens(value, currentPath);
    }
  }
}

function getTokenByPath(path) {
  const parts = path.split('.');
  let current = tokens;
  for (const part of parts) {
    current = current[part];
    if (!current) return null;
  }
  return current;
}

walkTokens(tokens);

// Report results
console.log('\n=== Token Validation Results ===\n');

if (errors.length > 0) {
  console.error(`❌ ${errors.length} errors found:`);
  errors.forEach(err => console.error(`   - ${err}`));
} else {
  console.log('✅ No structural errors');
}

if (warnings.length > 0) {
  console.warn(`⚠️  ${warnings.length} warnings:`);
  warnings.forEach(warn => console.warn(`   - ${warn}`));
}

process.exit(errors.length > 0 ? 1 : 0);
```

### Build Script (build.js)

```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

const tokensPath = path.join(__dirname, '../tokens.json');
const tokens = JSON.parse(fs.readFileSync(tokensPath, 'utf-8'));

// Generate Compose theme
function generateComposeTheme() {
  let output = `package com.example.designsystem

import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp

`;

  // Colors
  for (const [name, token] of Object.entries(tokens.color || {})) {
    if (token.$type === 'color') {
      const hex = token.$value.toUpperCase();
      output += `val ${pascalCase(name)}Color = Color(0xFF${hex.slice(1)})\n`;
    }
  }

  fs.writeFileSync(
    path.join(__dirname, '../build/android/Theme.kt'),
    output
  );
}

// Generate SwiftUI theme
function generateSwiftUITheme() {
  let output = `import SwiftUI

struct Colors {
`;

  for (const [name, token] of Object.entries(tokens.color || {})) {
    if (token.$type === 'color') {
      const hex = token.$value;
      const r = parseInt(hex.slice(1, 3), 16) / 255;
      const g = parseInt(hex.slice(3, 5), 16) / 255;
      const b = parseInt(hex.slice(5, 7), 16) / 255;
      output += `    static let ${camelCase(name)} = Color(red: ${r.toFixed(2)}, green: ${g.toFixed(2)}, blue: ${b.toFixed(2)})\n`;
    }
  }

  output += `}\n`;

  fs.writeFileSync(
    path.join(__dirname, '../build/ios/Theme.swift'),
    output
  );
}

// Generate CSS
function generateCSS() {
  let output = `:root {\n`;

  for (const [name, token] of Object.entries(tokens.color || {})) {
    if (token.$type === 'color') {
      output += `  --color-${kebabCase(name)}: ${token.$value};\n`;
    }
  }

  for (const [name, token] of Object.entries(tokens.spacing || {})) {
    if (token.$type === 'dimension') {
      output += `  --spacing-${kebabCase(name)}: ${token.$value};\n`;
    }
  }

  output += `}\n`;

  fs.writeFileSync(
    path.join(__dirname, '../build/web/tokens.css'),
    output
  );
}

// Utility functions
const pascalCase = str => str.split('-').map(w => w.charAt(0).toUpperCase() + w.slice(1)).join('');
const camelCase = str => str.split('-').map((w, i) => i === 0 ? w : w.charAt(0).toUpperCase() + w.slice(1)).join('');
const kebabCase = str => str.replace(/([a-z])([A-Z])/g, '$1-$2').toLowerCase();

console.log('Building design tokens...');
generateComposeTheme();
console.log('✅ Generated Android theme');
generateSwiftUITheme();
console.log('✅ Generated iOS theme');
generateCSS();
console.log('✅ Generated CSS tokens');
```

### GitHub Actions Workflow

**File:** `.github/workflows/validate-tokens.yml`

```yaml
name: Validate Design Tokens

on:
  pull_request:
    paths:
      - 'design-tokens/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install dependencies
        run: npm install
        working-directory: design-tokens

      - name: Validate tokens.json
        run: node scripts/validate.js
        working-directory: design-tokens

      - name: Generate platform code
        run: node scripts/build.js
        working-directory: design-tokens

      - name: Check for changes
        run: git diff --exit-code build/
        working-directory: design-tokens
```

## Cross-Platform Consistency Checklist

### Token Definition
- [ ] All tokens have `$type` and `$value`
- [ ] Token names follow semantic conventions (not color codes)
- [ ] Descriptions explain purpose, not appearance
- [ ] No hardcoded values in code (all use token references)
- [ ] References validate (no broken `{token.path}` links)

### Color Consistency
- [ ] Light theme and dark theme are defined
- [ ] Contrast ratios meet WCAG AA (4.5:1 normal, 3:1 large)
- [ ] Color palette supports accessibility (not red/green only)
- [ ] All platforms use same color values (no platform overrides)

### Typography Consistency
- [ ] Font families defined once (referenced everywhere)
- [ ] Font sizes match platform conventions (sp for Android, pt for iOS)
- [ ] Line heights are consistent across platforms
- [ ] Weight values are standardized (400, 500, 700)

### Spacing Consistency
- [ ] Spacing uses 4dp/4pt base grid
- [ ] Touch targets are 48dp/44pt minimum
- [ ] No magic numbers (all values from tokens)
- [ ] Padding/margin use consistent scale (xs, sm, md, lg, xl)

### Motion Consistency
- [ ] Duration tokens are used for all animations
- [ ] Easing curves are semantic (standard, emphasized, etc.)
- [ ] Transitions use consistent duration + easing pairs
- [ ] No hardcoded animation values in code

### Figma Integration
- [ ] Figma components use token variables
- [ ] Tokens Studio plugin is configured
- [ ] GitHub sync is enabled (auto-commit on changes)
- [ ] Token names match code (no manual translation)

### Code Generation
- [ ] `validate.js` runs without errors
- [ ] `build.js` generates all platforms
- [ ] Generated code matches hand-written version
- [ ] CI/CD validates on every PR

### Documentation
- [ ] README.md explains token structure
- [ ] Examples show how to use tokens in each platform
- [ ] Theme variants are documented
- [ ] Maintenance guide covers adding/updating tokens

## Next Steps

1. **Create tokens.json** following the W3C DTCG format
2. **Define base tokens** (colors, typography, spacing)
3. **Create theme variants** (light, dark, high-contrast)
4. **Generate platform code** using validation + build scripts
5. **Integrate with Figma** using Tokens Studio plugin
6. **Set up CI/CD** to validate and auto-generate on PR
7. **Migrate code** to use generated theme tokens
8. **Document** usage in team wiki or Storybook

## References

- [W3C Design Tokens Format Module 2025.10](https://www.designtokens.org/tr/drafts/format/)
- [Design Tokens Community Group](https://www.w3.org/community/design-tokens/)
- [Tokens Studio for Figma](https://www.tokens.studio/)
- [Style Dictionary (Adobe open-source)](https://styledictionary.com/)
- [Material Design 3 Tokens](https://m3.material.io/foundations/design-tokens/overview)
- [Specify Design Token Format](https://docs.specifyapp.com/concepts/specify-design-token-format)

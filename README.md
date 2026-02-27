# Kiln Mobile Design Plugin

**Harden your mobile designs into production-ready quality**

Kiln is a Claude Code plugin that brings Material Design 3 and Apple Human Interface Guidelines expertise directly into your development workflow. It catches common LLM-generated anti-patterns, enforces WCAG 2.2 accessibility compliance, and helps you build beautiful, performant mobile applications.

> *Like a kiln fires and hardens clay into durable ceramics, this plugin fires your UI code through rigorous design audits to produce hardened, production-ready mobile applications.*

## Features

- **18 Audit & Polish Commands** - Comprehensive design review capabilities
- **40+ Anti-Pattern Database** - Catches LLM-specific mobile code mistakes
- **WCAG 2.2 Level AA** - Full accessibility compliance checking
- **Multi-Provider Support** - Works with Claude Code, Cursor, Gemini, and Codex
- **Platform-Specific Skills** - Deep expertise for both Android and iOS

## Installation

### Claude Code
```bash
# Clone or download the plugin, then copy the dist to your plugins folder
mkdir -p ~/.claude/plugins/kiln
cp -r dist/claude-code/* ~/.claude/plugins/kiln/

# Or add to installed_plugins.json for local development:
# "kiln@local": [{ "scope": "project", "installPath": "/path/to/kiln/dist/claude-code", "version": "1.0.0" }]
```

### Cursor
```bash
cp -r dist/cursor/.cursor ~/.cursor/rules/kiln
```

### Other Providers
See `dist/gemini/` and `dist/codex/` for provider-specific formats.

## Commands

### Audit Commands
| Command | Description |
|---------|-------------|
| `/audit-material` | Audit Compose code for Material Design 3 compliance |
| `/audit-swiftui` | Audit SwiftUI code for HIG compliance |
| `/audit-accessibility` | WCAG 2.2 Level AA accessibility audit |
| `/audit-spacing` | Touch targets and spacing scale validation |
| `/audit-typography` | Typography scale and text accessibility |
| `/audit-motion` | Animation compliance and reduce motion |

### Polish Commands
| Command | Description |
|---------|-------------|
| `/polish-android` | Refine Compose code to M3 excellence |
| `/polish-ios` | Refine SwiftUI to iOS-native polish |
| `/polish-motion` | Refine animations for platform feel |

### Simplify Commands
| Command | Description |
|---------|-------------|
| `/simplify-compose` | Reduce Compose complexity |
| `/simplify-swiftui` | Reduce SwiftUI complexity |

### Adapt Commands
| Command | Description |
|---------|-------------|
| `/adapt-android-to-ios` | Port Compose to SwiftUI |
| `/adapt-ios-to-android` | Port SwiftUI to Compose |

### Specialized Commands
| Command | Description |
|---------|-------------|
| `/dark-mode-audit` | Dark mode and contrast compliance |
| `/responsive-audit` | Multi-device layout support |
| `/component-extract` | Extract reusable components |
| `/critique-design` | Visual hierarchy review |
| `/teach-mobile-design` | One-time design principles setup |

## Skills

### android-design
Material Design 3 expertise including:
- Typography scale (15 styles)
- Color system with dynamic color
- Touch targets (48dp minimum)
- State management patterns
- 20+ anti-pattern detection

### ios-design
Human Interface Guidelines expertise including:
- Dynamic Type (11 styles)
- Semantic colors
- Safe Areas
- Property wrappers
- 20+ anti-pattern detection

## Agents

### design-reviewer
Autonomous agent that runs multi-pass design review:
1. Accessibility audit
2. Design system compliance
3. Performance review
4. Consistency check
5. Synthesized report

### accessibility-auditor
WCAG 2.2 Level AA compliance auditor:
- Touch target validation
- Contrast checking
- Screen reader support
- Focus management
- Full compliance report

## Anti-Patterns Detected

### Android (Compose)
- Passing MutableState to children
- Reading scroll state in composition
- Creating modifiers in composable body
- Missing contentDescription
- Touch targets under 48dp
- Hardcoded colors and fonts
- And 14+ more...

### iOS (SwiftUI)
- @ObservedObject for owned instances
- @State with reference types
- AnyView overuse
- Missing accessibilityLabel
- Touch targets under 44pt
- Wrong modifier order
- And 14+ more...

## WCAG 2.2 Coverage

| Criterion | Level | Covered |
|-----------|-------|---------|
| 1.1.1 Non-text Content | A | ✅ |
| 1.4.3 Contrast (Minimum) | AA | ✅ |
| 1.4.4 Resize Text | AA | ✅ |
| 2.4.7 Focus Visible | AA | ✅ |
| 2.5.7 Dragging Movements | AA | ✅ |
| 2.5.8 Target Size | AA | ✅ |
| 3.3.7 Redundant Entry | A | ✅ |
| 3.3.8 Accessible Auth | AA | ✅ |

## Development

### Build
```bash
npm install
npm run build
```

### Test
```bash
npm test
```

### Project Structure
```
kiln/
├── plugin.json              # Plugin manifest
├── package.json             # Build dependencies
├── README.md                # This file
├── source/                  # Source files (edit these)
│   ├── commands/            # 18 audit commands
│   ├── skills/              # Platform skills
│   │   ├── android-design/
│   │   │   ├── SKILL.md
│   │   │   └── reference/   # 7 reference files
│   │   └── ios-design/
│   │       ├── SKILL.md
│   │       └── reference/   # 7 reference files
│   └── agents/              # Autonomous agents
├── scripts/                 # Build system
│   └── build.js
└── dist/                    # Generated (don't edit)
    ├── claude-code/
    ├── cursor/
    ├── gemini/
    └── codex/
```

## Contributing

1. Edit files in `source/`
2. Run `npm run build`
3. Test with your preferred provider
4. Submit PR

## License

MIT

## Disclaimer

This plugin provides audit capabilities against Material Design 3 and Human Interface
Guidelines standards but is **not an official Google or Apple product**. It is an
independent tool created by the PartyOn Team to help developers build better mobile
applications.

References to Material Design 3, Human Interface Guidelines, and other design systems
are for educational purposes only and do not imply endorsement or affiliation.

## Acknowledgments

- [Material Design 3](https://m3.material.io/) - Google's design system (referenced for educational purposes)
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/) - Apple's design standards (referenced for educational purposes)
- [WCAG 2.2](https://www.w3.org/TR/WCAG22/) - W3C accessibility guidelines
- [Jetpack Compose](https://developer.android.com/jetpack/compose) - Android UI toolkit
- [SwiftUI](https://developer.apple.com/xcode/swiftui/) - Apple's declarative UI framework
- [Now in Android](https://github.com/android/nowinandroid) - Google's reference app (Apache 2.0, patterns analyzed)
- [IceCubesApp](https://github.com/Dimillian/IceCubesApp) - Thomas Ricouard's Mastodon client (AGPL-3.0, patterns analyzed)
- [compose-rules](https://github.com/mrmans0n/compose-rules) - Compose linting rules (Apache 2.0)

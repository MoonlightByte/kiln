# Kiln Mobile Design Plugin

**Harden your mobile designs into production-ready quality**

Kiln is a Claude Code plugin that brings Material Design 3 and Apple Human Interface Guidelines expertise directly into your development workflow. It catches common LLM-generated anti-patterns, enforces WCAG 2.2 accessibility compliance, and helps you build beautiful, performant mobile applications.

> *Like a kiln fires and hardens clay into durable ceramics, this plugin fires your UI code through rigorous design audits to produce hardened, production-ready mobile applications.*

## Features

- **38 Audit & Polish Commands** - Comprehensive design review capabilities
- **40+ Anti-Pattern Database** - Catches LLM-specific mobile code mistakes
- **WCAG 2.2 Level AA** - Full accessibility compliance checking
- **Platform-Specific Skills** - Deep expertise for both Android and iOS
- **Edit Guards** - Auto-inject anti-pattern checklists when editing code

## Installation

### Claude Code

```bash
# Install from this repository
git clone https://github.com/MoonlightByte/kiln.git
mv kiln ~/.claude/plugins/
```

Or add to your Claude Code settings:
```json
{
  "plugins": [
    "https://github.com/MoonlightByte/kiln"
  ]
}
```

## Commands

### Core Audit Commands
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

### Platform Adaptation
| Command | Description |
|---------|-------------|
| `/adapt-android-to-ios` | Port Compose to SwiftUI |
| `/adapt-ios-to-android` | Port SwiftUI to Compose |

### Specialized Audits
| Command | Description |
|---------|-------------|
| `/audit-edge-to-edge` | Android edge-to-edge display compliance |
| `/audit-predictive-back` | Android predictive back gesture support |
| `/audit-liquid-glass` | iOS 26+ Liquid Glass design language |
| `/audit-observable` | SwiftUI @Observable macro patterns |
| `/audit-dynamic-color-2` | Material Dynamic Color v2 |
| `/audit-shape-morphing` | M3 shape morphing animations |
| `/dark-mode-audit` | Dark mode and contrast compliance |
| `/responsive-audit` | Multi-device layout support |
| `/performance-audit` | UI performance optimization |

### Testing & Tokens
| Command | Description |
|---------|-------------|
| `/generate-ui-tests` | Generate accessibility-focused UI tests |
| `/setup-snapshot-testing` | Configure screenshot testing |
| `/establish-design-tokens` | Create design token system |
| `/verify-design-tokens` | Validate token usage consistency |

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

### material-auditor
Material Design 3 compliance auditor for Android/Compose code.

### swiftui-auditor
Human Interface Guidelines compliance auditor for iOS/SwiftUI code.

### accessibility-auditor
WCAG 2.2 Level AA compliance auditor:
- Touch target validation
- Contrast checking
- Screen reader support
- Focus management
- Full compliance report

## Edit Guards (Hooks)

Kiln automatically injects anti-pattern checklists when you edit mobile code:

- **compose-edit-guard** - Triggers on `*.kt` files, injects M3 checklist
- **swiftui-edit-guard** - Triggers on `*.swift` files, injects HIG checklist

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

## Project Structure

```
kiln/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── commands/                # 38 slash commands
│   ├── audit-material.md
│   ├── audit-swiftui.md
│   ├── audit-accessibility.md
│   └── ...
├── agents/                  # 4 autonomous agents
│   ├── design-reviewer.md
│   ├── material-auditor.md
│   ├── swiftui-auditor.md
│   └── accessibility-auditor.md
├── skills/                  # Platform expertise
│   ├── android-design/
│   │   ├── SKILL.md
│   │   └── reference/       # 7 reference files
│   ├── ios-design/
│   │   ├── SKILL.md
│   │   └── reference/       # 7 reference files
│   └── shared/
│       └── references/      # Cross-platform references
├── hooks/
│   ├── hooks.json           # Hook configuration
│   ├── compose-edit-guard.md
│   └── swiftui-edit-guard.md
├── LICENSE
└── README.md
```

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

---
name: teach-mobile-design
description: One-time setup to teach mobile design principles for this session
args: []
---

# Mobile Design Principles

This command loads comprehensive mobile design knowledge into the current session. Run once at the start of a mobile development session.

## Core Philosophy

### Platform Authenticity
- **Android**: Follow Material Design 3 - expressive, accessible, adaptive
- **iOS**: Follow Human Interface Guidelines - clear, deferential, depth
- **Never mix**: Don't use iOS patterns on Android or vice versa

### Accessibility First
Build accessible from the start, not as an afterthought:
- WCAG 2.2 Level AA compliance
- Screen reader support (TalkBack/VoiceOver)
- Dynamic Type / sp scaling
- Sufficient contrast and touch targets

### Performance Matters
Mobile users have limited battery and patience:
- Smooth 60fps scrolling
- Efficient state management
- Lazy loading for lists
- Minimal recomposition/view updates

---

## Android (Jetpack Compose) Quick Reference

### Typography (Use MaterialTheme.typography.*)
```
displayLarge (57sp)  → Hero headlines
titleLarge (22sp)    → App bar, screen titles
bodyLarge (16sp)     → Main content
labelLarge (14sp)    → Buttons
```

### Colors (Use MaterialTheme.colorScheme.*)
```
primary        → Main interactive elements
onPrimary      → Text/icons on primary
surface        → Background
onSurface      → Text/icons on surface
error          → Error states
```

### Touch Targets
```kotlin
// ALWAYS use IconButton for icons
IconButton(onClick = { }) {
    Icon(Icons.Default.Star, contentDescription = "Favorite")
}

// Or ensure 48×48dp minimum
Modifier.minimumInteractiveComponentSize()
```

### State Management
```kotlin
// State in ViewModel
class MyViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(UiState())
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()
}

// Composable is stateless
@Composable
fun MyScreen(viewModel: MyViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()
    MyContent(uiState = uiState, onAction = viewModel::handleAction)
}
```

### Common Mistakes to Avoid
1. ❌ Passing MutableState to children
2. ❌ Creating modifiers inside composable body
3. ❌ Using Column for long lists (use LazyColumn)
4. ❌ Missing contentDescription on icons
5. ❌ Hardcoded colors instead of theme tokens

---

## iOS (SwiftUI) Quick Reference

### Typography (Use .font(.style))
```
.largeTitle (34pt)   → Hero headlines
.title (28pt)        → Page titles
.body (17pt)         → Main content
.caption (12pt)      → Supporting text
```

### Colors (Use semantic colors)
```swift
.label           → Primary text
.secondaryLabel  → Secondary text
.systemBackground → Background
.accentColor     → Interactive elements
```

### Touch Targets
```swift
// System Button provides 44×44pt automatically
Button(action: { }) {
    Image(systemName: "star.fill")
}

// Custom elements need explicit sizing
Image(systemName: "star")
    .frame(width: 44, height: 44)
    .contentShape(Rectangle())
    .onTapGesture { }
```

### State Management
```swift
// ObservableObject for complex state
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []
}

// @StateObject for owned instances
struct ContentView: View {
    @StateObject private var viewModel = ViewModel()
}

// @State for simple local state
struct ToggleView: View {
    @State private var isOn = false
}
```

### Common Mistakes to Avoid
1. ❌ @ObservedObject for owned objects (use @StateObject)
2. ❌ AnyView for conditional content (use @ViewBuilder)
3. ❌ VStack for long lists (use LazyVStack)
4. ❌ Missing accessibilityLabel on interactive elements
5. ❌ Hardcoded colors instead of semantic colors

---

## Accessibility Checklist

### Every Screen
- [ ] All interactive elements are 48×48dp / 44×44pt minimum
- [ ] All icons have text alternatives (contentDescription / accessibilityLabel)
- [ ] Text contrast is 4.5:1 minimum
- [ ] Text scales with system font size settings
- [ ] Focus order is logical

### Forms
- [ ] All fields have visible labels
- [ ] Error messages are announced
- [ ] Required fields are indicated
- [ ] Paste is allowed in text fields

### Images
- [ ] Informative images have descriptions
- [ ] Decorative images are hidden from screen readers
- [ ] Complex images have extended descriptions

---

## Design Tokens Summary

### Spacing (4dp/pt base)
```
4   → Inline
8   → Related elements
16  → Section spacing
24  → Major divisions
```

### Elevation (Android)
```
0dp  → Flat surface
1dp  → Cards, sheets
3dp  → FAB resting
6dp  → Dialogs
```

### Shape (Android M3)
```
4dp  → extraSmall (badges)
8dp  → small (chips)
12dp → medium (cards)
16dp → large (sheets)
28dp → extraLarge (FAB)
```

---

## Session Commands

After running this, use these commands for specific audits:

| Command | Use When |
|---------|----------|
| `/audit-material` | Reviewing Compose code |
| `/audit-swiftui` | Reviewing SwiftUI code |
| `/audit-accessibility` | Checking WCAG compliance |
| `/audit-typography` | Checking text styles |
| `/audit-spacing` | Checking touch targets and layout |
| `/polish-android` | Refining Compose to M3 excellence |
| `/polish-ios` | Refining SwiftUI to native polish |

---

**You are now configured with mobile design expertise for this session.**

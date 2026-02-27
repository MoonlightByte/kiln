# WCAG 2.2 Level AA Mobile Accessibility Rules

**Standards**: WCAG 2.2 Level AA, ADA Compliance
**Updated**: February 2025
**For**: Android (Jetpack Compose), iOS (SwiftUI)

---

## 1. Perceivable - Users Must See & Hear Content

### 1.4.3 Contrast (Minimum)
**Level AA Requirement**: 4.5:1 for normal text, 3:1 for large text

#### Color Contrast Rules

| Context | Requirement | Measurement |
|---------|------------|-------------|
| Normal Text (< 18pt) | 4.5:1 | Foreground / Background |
| Large Text (18pt+, 14pt bold+) | 3:1 | Foreground / Background |
| UI Components & Borders | 3:1 | Component / Adjacent color |
| Disabled States | 2:1 | Acceptable for disabled |
| Icons | 3:1 | Icon / Background |
| Focus Indicators | 3:1 | Focus outline / Surface |

#### Android Implementation
```kotlin
// BAD - 2:1 contrast (fails WCAG AA)
Text(
    text = "Secondary label",
    color = Color(0xFF999999),  // Gray on white = ~2:1
    style = MaterialTheme.typography.bodySmall
)

// GOOD - 4.5:1 contrast (meets WCAG AA)
Text(
    text = "Secondary label",
    color = MaterialTheme.colorScheme.onSurfaceVariant,  // ~7:1 ratio
    style = MaterialTheme.typography.bodySmall
)

// Verify contrast ratio (rough guide)
fun contrastRatio(foreground: Color, background: Color): Double {
    val fgLum = relativeLuminance(foreground)
    val bgLum = relativeLuminance(background)
    val lighter = maxOf(fgLum, bgLum)
    val darker = minOf(fgLum, bgLum)
    return (lighter + 0.05) / (darker + 0.05)
}

fun relativeLuminance(color: Color): Double {
    val r = color.red.let { if (it <= 0.03928) it / 12.92 else ((it + 0.055) / 1.055).pow(2.4) }
    val g = color.green.let { if (it <= 0.03928) it / 12.92 else ((it + 0.055) / 1.055).pow(2.4) }
    val b = color.blue.let { if (it <= 0.03928) it / 12.92 else ((it + 0.055) / 1.055).pow(2.4) }
    return 0.2126 * r + 0.7152 * g + 0.0722 * b
}
```

#### iOS Implementation
```swift
// BAD - 2:1 contrast (fails WCAG AA)
Text("Secondary label")
    .foregroundColor(Color(red: 0.6, green: 0.6, blue: 0.6))

// GOOD - 4.5:1 contrast (meets WCAG AA)
Text("Secondary label")
    .foregroundColor(.secondary)  // iOS semantic color ~7:1

// Verify at runtime
func contrastRatio(foreground: UIColor, background: UIColor) -> Double {
    let fgLum = foreground.relativeLuminance
    let bgLum = background.relativeLuminance
    let lighter = max(fgLum, bgLum)
    let darker = min(fgLum, bgLum)
    return (lighter + 0.05) / (darker + 0.05)
}
```

---

### 1.4.4 Resize Text
**Requirement**: Users can resize text up to 200% without loss of content

#### Android - Support Font Scaling
```kotlin
@Composable
fun ResponsiveText(text: String) {
    val fontSize = MaterialTheme.typography.bodyLarge.fontSize

    Text(
        text = text,
        fontSize = fontSize,  // Uses density and user scale settings
        style = MaterialTheme.typography.bodyLarge
    )
}

// Test: Settings > Display > Font size = Large
// Expected: Text enlarges, no clipping, horizontal scroll only if needed
```

#### iOS - Support Dynamic Type (AX1-AX7)
```swift
Text("Hello")
    .font(.body)  // Respects user's Dynamic Type setting (AX1 to AX7)
    .lineLimit(nil)  // Allow wrapping

// Test: Settings > Accessibility > Display & Text Size = Largest
// Expected: Text scales to AX7, no clipping
```

---

### 1.4.5 Images of Text
**Requirement**: Use real text instead of images containing text (except logos, charts)

#### Good
```kotlin
Text(
    text = "Welcome to PartyOn",
    style = MaterialTheme.typography.headlineSmall
)
```

#### Bad (Fails WCAG)
```kotlin
Image(
    painter = painterResource(id = R.drawable.welcome_banner),  // Contains text
    contentDescription = null  // Also missing description
)
```

---

## 2. Operable - Users Must Navigate & Control

### 2.1.1 Keyboard
**Requirement**: All functionality must be accessible via keyboard (mobile: VoiceOver/TalkBack)

#### Android - Keyboard Navigation
```kotlin
@Composable
fun InteractiveButton() {
    Button(
        onClick = { /* action */ },
        modifier = Modifier
            .semantics {
                contentDescription = "Submit form"  // TalkBack reads this
            }
    ) {
        Text("Submit")
    }
}

// Test with TalkBack enabled: Settings > Accessibility > TalkBack
// Expected: Can navigate with swipe, all buttons reachable
```

#### iOS - VoiceOver Navigation
```swift
Button(action: { /* action */ }) {
    Text("Submit")
}
.accessibilityLabel("Submit form")  // VoiceOver reads this
.accessibility(hint: Text("Sends your form"))  // Additional context

// Test: Settings > Accessibility > VoiceOver (on/off)
// Expected: Can navigate with rotor, all buttons reachable
```

---

### 2.5.5 Target Size (Enhanced)
**Level AAA (Level AA minimum below)**: 44x44dp (Android) / 44x44pt (iOS) minimum

#### Android Touch Targets - 48x48dp (M3 Standard)
```kotlin
// GOOD - Uses minimumInteractiveComponentSize()
IconButton(
    onClick = { /* action */ },
    modifier = Modifier.minimumInteractiveComponentSize()  // Ensures 48x48dp
) {
    Icon(Icons.Default.Close, contentDescription = "Close")
}

// GOOD - Explicit size
Button(
    onClick = { /* action */ },
    modifier = Modifier
        .width(48.dp)
        .height(48.dp)
) {
    Icon(Icons.Default.Home, contentDescription = "Home")
}

// BAD - Only 24x24dp (too small, fails WCAG)
Icon(
    imageVector = Icons.Default.Close,
    contentDescription = "Close",
    modifier = Modifier.size(24.dp)
)

// BAD - Padding not counted in touch target
Surface(
    modifier = Modifier
        .size(24.dp)
        .padding(12.dp)  // Reduces actual touch area
        .clickable { /* action */ }
) { }
```

#### Touch Target Spacing (M3 Standard)
- **Minimum**: 48x48dp
- **Recommended**: 48x48dp with 8dp spacing
- **Measuring**: From edge to edge of interactive area

```kotlin
// GOOD spacing - 8dp gap between buttons
Row(
    modifier = Modifier.fillMaxWidth(),
    horizontalArrangement = Arrangement.spacedBy(8.dp)  // 8dp gap
) {
    Button(modifier = Modifier.size(48.dp)) { }  // 48x48dp
    Button(modifier = Modifier.size(48.dp)) { }  // 48x48dp
}

// BAD spacing - buttons overlap (fail)
Row {
    Button(
        modifier = Modifier
            .size(48.dp)
            .offset(x = 32.dp)  // Overlaps previous button
    ) { }
}
```

#### iOS Touch Targets - 44x44pt Minimum
```swift
// GOOD - 44x44pt
Button(action: { /* action */ }) {
    Text("Submit")
}
.frame(minWidth: 44, minHeight: 44)

// GOOD - Implicit via HStack padding
HStack(spacing: 8) {
    Button(action: { }) { Image(systemName: "house") }
    Button(action: { }) { Image(systemName: "heart") }
}
.padding(8)  // Creates touch target with padding

// BAD - Only 24x24pt
Button(action: { }) {
    Image(systemName: "xmark")
        .frame(width: 24, height: 24)
}
```

---

### 2.4.7 Focus Visible
**Requirement**: Keyboard focus indicator is always visible (3:1 contrast minimum)

#### Android - Focus Indicator
```kotlin
Button(
    onClick = { /* action */ },
    modifier = Modifier
        .size(48.dp)
        .focusable()  // Enables focus state
) {
    Text("OK")
}

// Focus outline is automatically visible when navigating via keyboard
// Expected: 2-3dp blue/primary outline appears on focus
```

#### iOS - Focus Ring
```swift
Button(action: { }) {
    Text("OK")
}
.focusable()  // iOS 15+
.frame(minWidth: 44, minHeight: 44)

// Focus ring automatically appears for keyboard navigation
// Expected: 2pt system color outline appears on focus
```

---

## 3. Understandable - Clear Language & Predictable Behavior

### 3.2.2 On Input
**Requirement**: No unexpected changes when user interacts with UI elements

#### Android - Predictable State Changes
```kotlin
// BAD - Toggle switches without confirmation
var notifications by remember { mutableStateOf(true) }

Switch(
    checked = notifications,
    onCheckedChange = { notifications = it }  // Changes immediately, no confirmation
)

// GOOD - Explicit callback, optional confirmation
var notifications by remember { mutableStateOf(true) }

Switch(
    checked = notifications,
    onCheckedChange = {
        // Show confirmation dialog before switching
        showConfirmationDialog = true
    }
)
```

#### iOS - Predictable Navigation
```swift
// BAD - NavigationLink changes page unexpectedly
NavigationLink(destination: DetailView()) {
    Text("Item")
}  // Changes page immediately on tap

// GOOD - Explicit action handler
Button(action: {
    navigateTo(.detail)
}) {
    Text("Item")
}  // Clear action, predictable behavior
```

---

### 3.3.1 Error Identification
**Requirement**: Form errors must be identified and user is informed how to correct them

#### Android - Clear Error Messages
```kotlin
@Composable
fun LoginForm() {
    var email by remember { mutableStateOf("") }
    var emailError by remember { mutableStateOf("") }

    Column {
        TextField(
            value = email,
            onValueChange = {
                email = it
                emailError = validateEmail(it)
            },
            label = { Text("Email") },
            isError = emailError.isNotEmpty(),
            supportingText = if (emailError.isNotEmpty()) {
                { Text(emailError) }  // "Please enter a valid email"
            } else null
        )
    }
}

// Test with TalkBack: Should announce error AND correction instructions
```

#### iOS - Clear Error Messages
```swift
@State var email = ""
@State var emailError = ""

VStack(alignment: .leading) {
    TextField("Email", text: $email)
        .accessibilityHint(Text(emailError))  // Announce error
        .border(emailError.isEmpty ? Color.gray : Color.red)

    if !emailError.isEmpty {
        Text(emailError)
            .font(.caption)
            .foregroundColor(.red)
            .accessibilityAddTraits(.isStaticText)
    }
}

// Test with VoiceOver: Should announce error AND correction instructions
```

---

### 3.3.4 Error Prevention
**Requirement**: High-risk operations require confirmation or allow reversal

#### Android - Confirmation for Destructive Actions
```kotlin
@Composable
fun DeleteButton() {
    var showConfirm by remember { mutableStateOf(false) }

    Button(
        onClick = { showConfirm = true },
        colors = ButtonDefaults.buttonColors(
            containerColor = MaterialTheme.colorScheme.error
        )
    ) {
        Text("Delete")
    }

    if (showConfirm) {
        AlertDialog(
            onDismissRequest = { showConfirm = false },
            title = { Text("Delete this item?") },
            text = { Text("This action cannot be undone.") },
            confirmButton = {
                Button(
                    onClick = {
                        performDelete()
                        showConfirm = false
                    },
                    colors = ButtonDefaults.buttonColors(
                        containerColor = MaterialTheme.colorScheme.error
                    )
                ) {
                    Text("Delete")
                }
            },
            dismissButton = {
                Button(onClick = { showConfirm = false }) {
                    Text("Cancel")
                }
            }
        )
    }
}
```

#### iOS - Confirmation for Destructive Actions
```swift
@State var showConfirm = false

Button(role: .destructive, action: {
    showConfirm = true
}) {
    Text("Delete")
}
.confirmationDialog("Delete?", isPresented: $showConfirm) {
    Button("Delete", role: .destructive) {
        performDelete()
    }
    Button("Cancel", role: .cancel) { }
} message: {
    Text("This action cannot be undone.")
}
```

---

## 4. Robust - Works with Assistive Technology

### 4.1.2 Name, Role, Value
**Requirement**: All UI elements must have accessible name, role, and current value

#### Android - Semantic Information
```kotlin
// GOOD - All semantic info provided
IconButton(
    onClick = { deleteItem() },
    modifier = Modifier.semantics {
        contentDescription = "Delete this event"  // Accessible name
        // Role: button (implicit from IconButton)
        // Value: (implicit from action)
    }
) {
    Icon(Icons.Default.Delete, contentDescription = null)  // null because semantics block above
}

// BAD - Missing contentDescription
Icon(
    imageVector = Icons.Default.Delete,
    contentDescription = null  // TalkBack won't announce what this does
)

// BAD - Vague description
Icon(
    imageVector = Icons.Default.Delete,
    contentDescription = "Icon"  // Too vague, should be "Delete item"
)
```

#### iOS - Accessibility Labels
```swift
// GOOD - All semantic info provided
Button(action: { deleteItem() }) {
    Image(systemName: "trash")
}
.accessibilityLabel("Delete")  // Accessible name
.accessibility(hint: Text("Removes this event"))  // Additional context

// BAD - Missing accessibilityLabel
Button(action: { deleteItem() }) {
    Image(systemName: "trash")  // VoiceOver says "image" or "trash.fill"
}

// BAD - Vague label
Button(action: { deleteItem() }) {
    Image(systemName: "trash")
}
.accessibilityLabel("Icon")  // Too vague
```

---

### 4.1.3 Status Messages
**Requirement**: Status messages announced to assistive technology users

#### Android - Announce Important Changes
```kotlin
@Composable
fun ItemList(items: List<Item>) {
    var message by remember { mutableStateOf("") }

    LazyColumn {
        items(items, key = { it.id }) { item ->
            ItemRow(item)
        }
    }

    // Announce when items load
    LaunchedEffect(items.size) {
        if (items.isNotEmpty()) {
            message = "Loaded ${items.size} items"
            // TalkBack will announce this change
        }
    }
}

// Using announcement modifier
modifier = Modifier.semantics {
    liveRegion = LiveRegionMode.Assertive  // Announce immediately
}
```

#### iOS - Announce Important Changes
```swift
@State var message = ""

VStack {
    List(items) { item in
        ItemRow(item)
    }
    .onChange(of: items.count) { oldValue, newValue in
        if newValue > oldValue {
            message = "Loaded \(newValue) items"
            UIAccessibility.post(notification: .announcement, argument: message)
        }
    }
}
```

---

## 5. Quick Reference - Testing Checklist

### Automated Testing (Axe-Core Mobile)

#### Android (Using Axe-Core Android)
```bash
# In unit/instrumentation test
@RunWith(AndroidJUnit4::class)
class AccessibilityTest {
    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun testLoginScreenAccessibility() {
        composeTestRule.setContent {
            LoginScreen()
        }

        // Check touch targets
        composeTestRule.onAllNodes(isEnabled())
            .forEach { node ->
                val size = node.fetchSemanticsNode().boundsInRoot
                assertTrue("Touch target too small", size.width >= 48.dp && size.height >= 48.dp)
            }

        // Check contrast ratios
        // (requires custom implementation or axe-core-android integration)
    }
}
```

#### iOS (Using Axe-Core iOS)
```swift
import AxeCore

@MainActor
class AccessibilityTests: XCTestCase {
    func testLoginScreenAccessibility() async {
        let app = XCUIApplication()
        app.launch()

        // Run axe scan
        let axe = AxeDevTools()
        let results = await axe.scan()

        let violations = results.violations.filter { $0.impact == .critical }
        XCTAssertEqual(violations.count, 0, "Found critical accessibility violations")
    }
}
```

### Manual Testing (Always Required)

1. **Screen Reader Test**
   - Android: Enable TalkBack (Settings > Accessibility > TalkBack)
   - iOS: Enable VoiceOver (Settings > Accessibility > VoiceOver)
   - Verify: All buttons are reachable, all text is readable, error messages are clear

2. **Keyboard Navigation Test**
   - Android: Use Tab/Shift+Tab with external keyboard
   - iOS: Use keyboard navigation (Hardware Keyboard Enabled)
   - Verify: Focus indicators visible, all buttons reachable without mouse

3. **Contrast Test**
   - Use browser dev tools or accessibility checker
   - Verify: All text ≥ 4.5:1, large text ≥ 3:1, UI components ≥ 3:1

4. **Text Scaling Test**
   - Android: Settings > Display > Font size = Large
   - iOS: Settings > Accessibility > Display & Text Size = Largest
   - Verify: Text scales, no clipping, layout adapts

5. **Color Blindness Test**
   - Use simulator: Android Studio > AVD > Extended controls > Color blindness
   - Verify: Information conveyed not by color alone

---

## 6. WCAG 2.2 Criteria Summary Table

| Criterion | Level | Platforms | Requirement |
|-----------|-------|-----------|-------------|
| 1.4.3 Contrast | AA | Both | 4.5:1 normal, 3:1 large |
| 1.4.4 Resize Text | AA | Both | User can scale to 200% |
| 1.4.5 Images of Text | AA | Both | Use text not images |
| 2.1.1 Keyboard | A | Both | All features via keyboard |
| 2.5.5 Target Size (Enhanced) | AAA | Both | 44x44dp/pt minimum |
| 2.4.7 Focus Visible | AA | Both | Focus indicator 3:1 contrast |
| 3.2.2 On Input | A | Both | Predictable state changes |
| 3.3.1 Error ID | A | Both | Identify & explain errors |
| 3.3.4 Error Prevention | AA | Both | Confirm destructive actions |
| 4.1.2 Name, Role, Value | A | Both | Semantic information |
| 4.1.3 Status Messages | AAA | Both | Announce changes |

---

## 7. Resources

- **Axe DevTools**: https://www.deque.com/axe/core/
- **Axe-Core Android**: https://github.com/dequelabs/axe-core-android
- **Axe-Core iOS**: https://github.com/dequelabs/axe-core-ios
- **WCAG 2.2 Official**: https://www.w3.org/WAI/WCAG22/quickref/
- **Google a11y Checker**: https://www.google.com/accessibility/
- **Apple HIG Accessibility**: https://developer.apple.com/design/human-interface-guidelines/accessibility

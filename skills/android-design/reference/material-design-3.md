# Material Design 3 Foundation

Comprehensive reference for Material Design 3 implementation in Jetpack Compose.

---

## Core Principles

### 1. Dynamic Color
Material 3 extracts colors from the user's wallpaper (Android 12+) to create personalized themes.

```kotlin
@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context)
            else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        shapes = Shapes,
        content = content
    )
}
```

### 2. Expressive Components
M3 components have updated designs with more rounded corners, better elevation, and improved accessibility.

### 3. Accessible by Default
- 48dp minimum touch targets
- 4.5:1 contrast ratio for normal text
- 3:1 contrast ratio for large text and UI components

---

## Color System

### Key Colors
| Role | Light Theme | Dark Theme | Use Case |
|------|-------------|------------|----------|
| Primary | primary | primary | Main interactive elements |
| OnPrimary | onPrimary | onPrimary | Content on primary surfaces |
| PrimaryContainer | primaryContainer | primaryContainer | Tinted containers |
| OnPrimaryContainer | onPrimaryContainer | onPrimaryContainer | Content on primary containers |
| Secondary | secondary | secondary | Secondary actions |
| OnSecondary | onSecondary | onSecondary | Content on secondary |
| SecondaryContainer | secondaryContainer | secondaryContainer | Secondary tinted containers |
| Tertiary | tertiary | tertiary | Accent elements |
| Error | error | error | Error states |
| Surface | surface | surface | Background surfaces |
| OnSurface | onSurface | onSurface | Content on surfaces |
| SurfaceVariant | surfaceVariant | surfaceVariant | Variant backgrounds |
| Outline | outline | outline | Borders, dividers |
| OutlineVariant | outlineVariant | outlineVariant | Subtle borders |

### Surface Elevation (Tonal Elevation)
M3 uses tonal elevation instead of shadow elevation for surfaces.

```kotlin
// Elevation levels
Surface(tonalElevation = 0.dp) // Level 0 - No tint
Surface(tonalElevation = 1.dp) // Level 1 - Slight tint
Surface(tonalElevation = 3.dp) // Level 2 - More tint
Surface(tonalElevation = 6.dp) // Level 3 - Prominent tint
Surface(tonalElevation = 8.dp) // Level 4 - Strong tint
Surface(tonalElevation = 12.dp) // Level 5 - Maximum tint
```

---

## Component Specifications

### Buttons

| Type | Min Width | Height | Shape | Elevation |
|------|-----------|--------|-------|-----------|
| Filled | 48dp | 40dp | Full (20dp) | 0dp (1dp pressed) |
| Outlined | 48dp | 40dp | Full (20dp) | 0dp |
| Text | 48dp | 40dp | Full (20dp) | 0dp |
| Elevated | 48dp | 40dp | Full (20dp) | 1dp (2dp pressed) |
| FAB | 56dp | 56dp | Large (16dp) | 3dp (6dp pressed) |
| Extended FAB | 80dp | 56dp | Large (16dp) | 3dp (6dp pressed) |

```kotlin
// Primary action
Button(onClick = { /* action */ }) {
    Text("Submit")
}

// Secondary action
OutlinedButton(onClick = { /* action */ }) {
    Text("Cancel")
}

// Low emphasis
TextButton(onClick = { /* action */ }) {
    Text("Learn more")
}

// Floating action
FloatingActionButton(onClick = { /* action */ }) {
    Icon(Icons.Default.Add, contentDescription = "Add")
}
```

### Cards

| Type | Shape | Elevation | Use Case |
|------|-------|-----------|----------|
| Filled | 12dp | 0dp | Primary content containers |
| Elevated | 12dp | 1dp | Lifted containers |
| Outlined | 12dp | 0dp | Bordered containers |

```kotlin
// Elevated card (most common)
ElevatedCard(
    modifier = Modifier.fillMaxWidth()
) {
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Card Title", style = MaterialTheme.typography.titleLarge)
        Text("Card content here", style = MaterialTheme.typography.bodyMedium)
    }
}

// Outlined card
OutlinedCard(
    modifier = Modifier.fillMaxWidth()
) {
    // content
}
```

### Text Fields

| Type | Height | Label Position | Helper Text |
|------|--------|----------------|-------------|
| Filled | 56dp | Inside (floats up) | Below field |
| Outlined | 56dp | Border (floats up) | Below field |

```kotlin
var text by remember { mutableStateOf("") }

OutlinedTextField(
    value = text,
    onValueChange = { text = it },
    label = { Text("Email") },
    supportingText = { Text("Enter your email address") },
    isError = !text.contains("@"),
    singleLine = true,
    modifier = Modifier.fillMaxWidth()
)
```

### Navigation

| Component | Items | Height | Use Case |
|-----------|-------|--------|----------|
| Navigation Bar | 3-5 | 80dp | Primary navigation |
| Navigation Rail | 3-7 | Full | Tablet/desktop |
| Navigation Drawer | Unlimited | Full | Secondary navigation |

```kotlin
NavigationBar {
    items.forEachIndexed { index, item ->
        NavigationBarItem(
            icon = { Icon(item.icon, contentDescription = item.label) },
            label = { Text(item.label) },
            selected = selectedItem == index,
            onClick = { onItemSelected(index) }
        )
    }
}
```

### Dialogs

| Type | Max Width | Shape | Scrim |
|------|-----------|-------|-------|
| Basic | 280dp | 28dp | 32% black |
| Full-screen | 100% | 0dp | None |

```kotlin
AlertDialog(
    onDismissRequest = { /* dismiss */ },
    title = { Text("Confirm Action") },
    text = { Text("Are you sure you want to proceed?") },
    confirmButton = {
        TextButton(onClick = { /* confirm */ }) {
            Text("Confirm")
        }
    },
    dismissButton = {
        TextButton(onClick = { /* dismiss */ }) {
            Text("Cancel")
        }
    }
)
```

---

## Shape Scale

| Token | Corner Radius | Use Case |
|-------|---------------|----------|
| extraSmall | 4dp | Badges, small chips |
| small | 8dp | Chips, small cards |
| medium | 12dp | Cards, dialogs |
| large | 16dp | Large cards, sheets |
| extraLarge | 28dp | FABs, full sheets |

```kotlin
val Shapes = Shapes(
    extraSmall = RoundedCornerShape(4.dp),
    small = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large = RoundedCornerShape(16.dp),
    extraLarge = RoundedCornerShape(28.dp)
)
```

---

## Icons

### Sizing
| Context | Size | Touch Target |
|---------|------|--------------|
| Navigation | 24dp | 48dp |
| Button | 18dp | 40dp |
| FAB | 24dp | 56dp |
| Dense list | 24dp | 48dp |

### Filled vs Outlined
- **Filled icons**: Selected state, emphasis
- **Outlined icons**: Unselected state, secondary

```kotlin
// Toggle between filled/outlined based on state
Icon(
    imageVector = if (selected) Icons.Filled.Favorite else Icons.Outlined.FavoriteBorder,
    contentDescription = if (selected) "Remove from favorites" else "Add to favorites"
)
```

---

## State Layers

Interactive elements show state through overlay opacity:

| State | Opacity |
|-------|---------|
| Enabled | 0% |
| Hovered | 8% |
| Focused | 12% |
| Pressed | 12% |
| Dragged | 16% |
| Disabled | 38% (content opacity) |

---

## Motion

### Duration
| Type | Duration | Use Case |
|------|----------|----------|
| Short1 | 50ms | Micro-interactions |
| Short2 | 100ms | Small elements |
| Short3 | 150ms | Icons, badges |
| Short4 | 200ms | Standard |
| Medium1 | 250ms | Cards, dialogs |
| Medium2 | 300ms | Transitions |
| Medium3 | 350ms | Complex animations |
| Medium4 | 400ms | Large elements |
| Long1 | 450ms | Page transitions |
| Long2 | 500ms | Complex sequences |

### Easing
| Type | Use Case |
|------|----------|
| LinearOutSlowIn | Entering |
| FastOutSlowIn | Standard |
| FastOutLinearIn | Exiting |
| Emphasized | Hero animations |
| EmphasizedDecelerate | Incoming focus |
| EmphasizedAccelerate | Outgoing focus |

```kotlin
val enterTransition = fadeIn(
    animationSpec = tween(
        durationMillis = 300,
        easing = FastOutSlowInEasing
    )
)
```

---

## Accessibility Checklist

- [ ] All interactive elements have 48x48dp minimum touch target
- [ ] Color contrast meets 4.5:1 for normal text
- [ ] Color contrast meets 3:1 for large text (24sp+)
- [ ] All icons have contentDescription
- [ ] Form fields have visible labels
- [ ] Error states have both color and text indicators
- [ ] Focus indicators are visible
- [ ] Content is readable at 200% font scale

# iOS 18 Materials and Glass Effects API

Comprehensive guide to iOS 18's new Material effects, glass morphism, and semantic color updates. Enables modern "liquid glass" designs seen in iOS 18 interface.

---

## Materials API Overview (iOS 15+, Enhanced iOS 18)

### What Are Materials?

Materials are system-provided blur/transparency effects that:
- Adapt to light/dark mode
- Respect Dynamic Type
- Handle elevation hierarchy
- Work across all devices (iPhone, iPad, Mac)

---

## Material Types

| Material | Density | Use Case | Blur Radius |
|----------|---------|----------|-------------|
| `.ultraThinMaterial` | Lightest | Floating badges, subtle overlays | ~5px |
| `.thinMaterial` | Light | Floating buttons, secondary elements | ~10px |
| `.regularMaterial` | Medium | Card backgrounds, modals | ~20px |
| `.thickMaterial` | Heavy | Navigation bars, major grouping | ~30px |
| `.ultraThickMaterial` | Heaviest | Full-screen overlays, alerts | ~40px |

### Transparency + Blur Combination

```
┌─────────────────────────────────────────────────┐
│  Color Layer                                    │
│  + Semi-transparent overlay (70-90% opacity)   │
│  + Gaussian blur (varies by material)           │
│  + Vibrancy effect (color shift in dark mode)   │
└─────────────────────────────────────────────────┘
```

---

## Basic Usage

### Material as Background

```swift
// BASIC: Material as direct background
VStack {
    Text("Floating Content")
        .font(.title)
}
.frame(height: 200)
.background(.regularMaterial) // Blurred background

// With shape
VStack {
    Text("Floating Content")
}
.frame(height: 200)
.background(.thickMaterial)
.cornerRadius(12)

// Nested materials (layering)
ZStack {
    Color.blue.opacity(0.3)

    VStack {
        Text("Content")
    }
    .background(.regularMaterial) // Material over color
}
```

---

## Advanced: Glass Morphism

### Pattern 1: Floating Card Over Blurred Background

```swift
struct GlassCard: View {
    var body: some View {
        ZStack {
            // Full-bleed blur effect
            Image("background")
                .resizable()
                .scaledToFill()
                .blur(radius: 30)
                .ignoresSafeArea()

            // Glass card floating over blur
            VStack(spacing: 16) {
                HStack {
                    VStack(alignment: .leading) {
                        Text("Title")
                            .font(.headline)
                        Text("Subtitle")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }

                    Spacer()

                    Image(systemName: "star.fill")
                        .font(.title2)
                        .foregroundColor(.yellow)
                }

                Divider()

                HStack {
                    Button("Cancel") { }
                    Spacer()
                    Button("Confirm") { }
                }
            }
            .padding(16)
            .background(.regularMaterial)
            .cornerRadius(16)
            .padding(16)
        }
    }
}
```

**Key elements**:
- `.blur(radius: 30)` on background image
- `.regularMaterial` for the card itself
- Shadows optional (modern design often skips)
- Corner radius 12-16pt typical

---

### Pattern 2: Navigation Bar with Material

```swift
struct NavigationWithMaterial: View {
    var body: some View {
        ZStack(alignment: .top) {
            // Content with background
            ScrollView {
                VStack(spacing: 16) {
                    ForEach(0..<20, id: \.self) { i in
                        Text("Item \(i)")
                            .frame(height: 100)
                            .frame(maxWidth: .infinity)
                            .background(Color.blue.opacity(0.1))
                            .cornerRadius(8)
                    }
                }
                .padding(16)
            }

            // Glass nav bar floating
            VStack {
                HStack {
                    Button(action: {}) {
                        Image(systemName: "chevron.left")
                    }

                    Spacer()

                    Text("Title")
                        .font(.headline)

                    Spacer()

                    Button(action: {}) {
                        Image(systemName: "ellipsis")
                    }
                }
                .padding(12)
                .frame(height: 44)
            }
            .background(.ultraThickMaterial)
            .shadow(radius: 8) // Optional subtle shadow
        }
    }
}
```

---

### Pattern 3: Floating Action Button with Material

```swift
struct FABWithMaterial: View {
    @State private var isExpanded = false

    var body: some View {
        ZStack(alignment: .bottomTrailing) {
            // Main content
            List {
                ForEach(0..<10, id: \.self) { i in
                    Text("Item \(i)")
                }
            }

            // Floating action button with material
            VStack(spacing: 12) {
                if isExpanded {
                    Button(action: {}) {
                        Image(systemName: "square.and.pencil")
                            .foregroundColor(.blue)
                    }
                    .frame(width: 44, height: 44)
                    .background(.thinMaterial)
                    .clipShape(Circle())

                    Button(action: {}) {
                        Image(systemName: "plus")
                            .foregroundColor(.blue)
                    }
                    .frame(width: 44, height: 44)
                    .background(.thinMaterial)
                    .clipShape(Circle())
                }

                Button(action: { withAnimation { isExpanded.toggle() } }) {
                    Image(systemName: "plus")
                        .font(.system(size: 20, weight: .semibold))
                        .foregroundColor(.white)
                }
                .frame(width: 52, height: 52)
                .background(.regularMaterial)
                .clipShape(Circle())
            }
            .padding(16)
        }
    }
}
```

---

## Combining Colors with Materials

### Material + Tint Color

```swift
struct MaterialWithTint: View {
    var body: some View {
        VStack {
            Text("Tinted Glass Card")
                .font(.headline)
        }
        .frame(height: 150)
        // Option 1: Base color + material
        .background(Color.blue.opacity(0.2))
        .background(.regularMaterial)

        // Option 2: Foreground tint + material
        .foregroundColor(.white)
        .background(
            ZStack {
                Color.blue.opacity(0.4)
                    .ignoresSafeArea()

                Color.blue.opacity(0.1)
                    .background(.regularMaterial)
            }
        )

        // Option 3: Using @Environment color
        .tint(.blue)
        .background(.regularMaterial)
    }
}
```

---

## Elevation Hierarchy

### Material Stacking

```swift
struct ElevationHierarchy: View {
    var body: some View {
        ZStack {
            // Layer 1: Base background
            Color(.systemBackground)

            VStack(spacing: 16) {
                // Layer 2: Card (thinner material)
                VStack {
                    Text("Level 1 Card")
                }
                .padding(12)
                .background(.thinMaterial)
                .cornerRadius(8)

                // Layer 3: Floating element (thicker material)
                VStack {
                    Text("Level 2 Floating")
                }
                .padding(12)
                .background(.regularMaterial)
                .cornerRadius(8)

                // Layer 4: Modal (thickest material)
                VStack {
                    Text("Level 3 Modal")
                }
                .padding(12)
                .background(.ultraThickMaterial)
                .cornerRadius(8)
            }
            .padding(16)
        }
    }
}
```

**Rule**: More elevation = thicker material (blur more)

---

## Dark Mode and Vibrancy

### Automatic Adaptation

```swift
struct DarkModeMaterial: View {
    var body: some View {
        VStack {
            Text("Light mode: Subtle, transparent")
                .font(.caption)
                .foregroundColor(.secondary)

            Divider()

            Text("Dark mode: Brighter, more opaque")
                .font(.caption)
                .foregroundColor(.secondary)
        }
        .padding(12)
        .background(.regularMaterial) // Auto-adapts!
        .cornerRadius(12)
    }
}

// Materials automatically:
// - Add tint/vibrancy in dark mode
// - Reduce transparency in light mode
// - Match system colors
// - NO CUSTOM COLOR NEEDED
```

---

## Performance Considerations

### When to Use Materials

**Good**:
```swift
// Light overlays on static content
VStack {
    Text("Content")
}
.background(.thinMaterial)

// Floating UI elements
Button { }
    .background(.regularMaterial)
    .cornerRadius(8)

// Navigation bars
VStack { }
    .background(.ultraThickMaterial)
```

**Avoid**:
```swift
// ❌ Material on every view (performance hit)
List {
    ForEach(items) { item in
        Text(item.name)
            .background(.regularMaterial) // Too much blur
    }
}

// ❌ Nested materials (visual noise)
VStack {
    HStack {
        Text("Content")
            .background(.thinMaterial)
    }
    .background(.regularMaterial) // Too many layers
}

// ❌ Material + complex animation (jank)
VStack { }
    .background(.regularMaterial)
    .offset(x: animatingValue) // May stutter
```

---

## iOS 18 New Features

### Chromatic Blur (iOS 18+)

```swift
// Experimental: Color separation effect in blur
VStack {
    Text("Chromatic effect")
}
.background(.regularMaterial)
.blur(radius: 10, opaque: false) // Opaque: false enables color shift
```

### Enhanced Vibrancy (iOS 18+)

```swift
// More pronounced color interaction in dark mode
VStack(spacing: 12) {
    Text("Title")
        .font(.headline)
        .foregroundColor(.primary)

    Text("Subtitle")
        .font(.caption)
        .foregroundColor(.secondary)
}
.padding(12)
.background(.regularMaterial)
.containerShape(RoundedRectangle(cornerRadius: 12))
```

---

## Complete Example: iOS 18 Glass Design

```swift
struct iOS18GlassDesign: View {
    @State private var selectedTab = 0

    var body: some View {
        ZStack {
            // Background gradient
            LinearGradient(
                gradient: Gradient(colors: [
                    Color.blue.opacity(0.3),
                    Color.purple.opacity(0.3)
                ]),
                startPoint: .topLeading,
                endPoint: .bottomTrailing
            )
            .ignoresSafeArea()

            VStack(spacing: 0) {
                // Glass header
                HStack {
                    VStack(alignment: .leading) {
                        Text("Welcome")
                            .font(.title2)
                            .bold()

                        Text("iOS 18 Glass Design")
                            .font(.caption)
                            .foregroundColor(.secondary)
                    }

                    Spacer()

                    Image(systemName: "bell.badge")
                        .font(.title2)
                        .foregroundColor(.accentColor)
                }
                .padding(12)
                .background(.ultraThickMaterial)
                .cornerRadius(12)
                .padding(12)

                // Content area
                ScrollView {
                    VStack(spacing: 12) {
                        // Glass card 1
                        VStack(alignment: .leading, spacing: 8) {
                            HStack {
                                Image(systemName: "star.fill")
                                    .foregroundColor(.yellow)

                                Text("Featured")
                                    .font(.headline)

                                Spacer()

                                Image(systemName: "chevron.right")
                                    .foregroundColor(.secondary)
                            }

                            Text("Explore the new glass morphism design system")
                                .font(.caption)
                                .foregroundColor(.secondary)
                                .lineLimit(2)
                        }
                        .padding(12)
                        .background(.regularMaterial)
                        .cornerRadius(12)

                        // Glass card 2
                        VStack(alignment: .leading, spacing: 8) {
                            HStack {
                                Image(systemName: "sparkles")
                                    .foregroundColor(.pink)

                                Text("What's New")
                                    .font(.headline)

                                Spacer()

                                Image(systemName: "chevron.right")
                                    .foregroundColor(.secondary)
                            }

                            Text("iOS 18 introduces enhanced materials and blur effects")
                                .font(.caption)
                                .foregroundColor(.secondary)
                                .lineLimit(2)
                        }
                        .padding(12)
                        .background(.regularMaterial)
                        .cornerRadius(12)

                        // Glass card 3
                        VStack(alignment: .leading, spacing: 8) {
                            HStack {
                                Image(systemName: "lightbulb.fill")
                                    .foregroundColor(.yellow)

                                Text("Tips")
                                    .font(.headline)

                                Spacer()

                                Image(systemName: "chevron.right")
                                    .foregroundColor(.secondary)
                            }

                            Text("Use materials to create hierarchy without heavy shadows")
                                .font(.caption)
                                .foregroundColor(.secondary)
                                .lineLimit(2)
                        }
                        .padding(12)
                        .background(.regularMaterial)
                        .cornerRadius(12)
                    }
                    .padding(12)
                }

                // Glass bottom bar
                HStack(spacing: 16) {
                    Spacer()

                    Button(action: {}) {
                        Image(systemName: "house.fill")
                            .font(.system(size: 20))
                    }

                    Button(action: {}) {
                        Image(systemName: "magnifyingglass")
                            .font(.system(size: 20))
                    }

                    Button(action: {}) {
                        Image(systemName: "heart.fill")
                            .font(.system(size: 20))
                    }

                    Spacer()
                }
                .padding(12)
                .frame(height: 56)
                .background(.ultraThickMaterial)
                .cornerRadius(12)
                .padding(12)
            }
        }
    }
}
```

---

## Design Guidelines

### Color Contrast with Materials

```swift
// ✅ GOOD: High contrast text on material
Text("Clear text")
    .foregroundColor(.primary)
    .background(.regularMaterial)

// ❌ BAD: Low contrast
Text("Hard to read")
    .foregroundColor(.gray)
    .background(.regularMaterial)
```

### Touch Target Sizing

```swift
// Material buttons always 44x44 minimum
Button(action: {}) {
    Image(systemName: "plus")
}
.frame(width: 44, height: 44)
.background(.regularMaterial)
.clipShape(Circle())
```

### Spacing with Materials

```swift
// Material cards benefit from padding/spacing
VStack(spacing: 12) {
    // Spacing between glass cards
    VStack {
        Text("Card 1")
    }
    .background(.regularMaterial)

    VStack {
        Text("Card 2")
    }
    .background(.regularMaterial)
}
.padding(16) // Outer padding
```

---

## References

- Apple Materials Documentation: https://developer.apple.com/documentation/swiftui/material
- WWDC 2024 Session 10147: "What's new in SwiftUI"
- iOS 18 Design Updates: https://developer.apple.com/design/ios18/
- Semantic Colors: https://developer.apple.com/design/human-interface-guidelines/color/overview

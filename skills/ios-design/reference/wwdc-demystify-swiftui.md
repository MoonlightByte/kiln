# WWDC 2021: Demystify SwiftUI (Session 10022)

Key insights from Apple's definitive SwiftUI architecture talk. This explains the fundamental mechanisms SwiftUI uses to manage view identity, lifetime, and performance.

---

## View Identity

### What is View Identity?

SwiftUI uses **identity** to track views across updates. Identity is NOT the view instance - it's a logical identifier that tells SwiftUI "this is the same view as before."

**Two types of identity:**

1. **Explicit Identity**: Provided by the developer (e.g., `id: \.id` in ForEach)
2. **Structural Identity**: Derived from position in the view hierarchy (fallback)

### Explicit Identity in ForEach

```swift
// GOOD - Stable IDs survive reordering
ForEach(items, id: \.id) { item in
    ItemRow(item: item)
}

// BAD - Index-based IDs break on reorder
ForEach(items.indices, id: \.self) { index in
    ItemRow(item: items[index]) // State lost if items reorder!
}

// WORST - No ID at all (structural fallback)
ForEach(items) { item in // Assumes items never reorder
    ItemRow(item: item)
}
```

**Rule**: Always provide stable IDs with `id:` parameter. Structural identity is fragile.

---

## View Lifetime

### Creation vs. Identity

SwiftUI distinguishes between:

- **View Type**: The Swift struct (e.g., `ContentView`)
- **View Instance**: A specific occurrence in the hierarchy
- **View Identity**: The logical identifier that connects instances across updates

```swift
struct ContentView: View {
    @State private var counter = 0

    var body: some View {
        VStack {
            Text("Count: \(counter)")
            Button("Inc") { counter += 1 }
        }
    }
}

// Same struct TYPE
// Same view INSTANCE across body re-evaluations
// SwiftUI uses @State to preserve lifetime
```

**Key insight**: `@State` preserves the instance lifetime WITHIN a single view. When the parent view is destroyed, so is the `@State`.

---

## Property Wrapper Lifetime Rules

### @StateObject vs. @ObservedObject

```swift
// @StateObject: Preserves object across view updates
struct ParentView: View {
    @StateObject private var viewModel = ViewModel() // Created ONCE

    var body: some View {
        ChildView(viewModel: viewModel)
    }
}

// @ObservedObject: No preservation, recreated if not passed from parent
struct ChildView: View {
    @ObservedObject var viewModel: ViewModel // Passed from parent

    var body: some View {
        Text(viewModel.text)
    }
}

// ANTI-PATTERN: @ObservedObject with initialization
struct BadView: View {
    @ObservedObject var viewModel = ViewModel() // Recreated on every view update!

    var body: some View {
        Text(viewModel.text)
    }
}
```

**Rule**:
- Use `@StateObject` for objects YOU create in the view
- Use `@ObservedObject` for objects passed from a parent
- Never use `@ObservedObject var x = Something()` - that defeats the purpose

---

## Dependency Rules

### Forward vs. Backward Dependencies

```swift
// Forward dependency (Parent → Child): OK
struct ParentView: View {
    @StateObject private var model = Model()

    var body: some View {
        ChildView(model: model) // Passes model down
    }
}

struct ChildView: View {
    @ObservedObject var model: Model // Receives model

    var body: some View {
        Text(model.value)
    }
}

// Backward dependency (Child → Parent): NOT OK
struct ParentView: View {
    @State private var value = ""

    var body: some View {
        ChildView(binding: $value) // OK - binding is special
    }
}

struct ChildView: View {
    @Binding var binding: String // Receives binding

    var body: some View {
        TextField("", text: $binding)
    }
}

// Sideways dependency via EnvironmentObject: Also OK
@main
struct App: App {
    @StateObject var appState = AppState()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState) // Inject into tree
        }
    }
}

struct ContentView: View {
    @EnvironmentObject var appState: AppState // Any descendant can access

    var body: some View {
        Text(appState.value)
    }
}
```

---

## Why AnyView Destroys View Hierarchy

### The Problem with Type Erasure

```swift
// This seems fine, but SwiftUI can't see the actual types
func makeView() -> AnyView {
    if condition {
        return AnyView(Text("A"))
    } else {
        return AnyView(Image("B"))
    }
}

// What's happening internally:
// SwiftUI sees: AnyView { some opaque view }
// SwiftUI can't tell when the content CHANGES
// Result: View is destroyed and recreated from scratch
```

**The cost**:
- State is lost (or reset)
- Animations don't work
- Identity is lost
- No diff capability

### The Solution: @ViewBuilder

```swift
// SwiftUI sees the actual types at compile time
@ViewBuilder
func makeView() -> some View {
    if condition {
        Text("A")
    } else {
        Image("B")
    }
}

// @ViewBuilder enables:
// - Conditional compilation into TupleView<>
// - SwiftUI knows exact types
// - Efficient diffing
// - State preservation
```

---

## View Diffing and Reconciliation

### How SwiftUI Updates Views

1. **Type Check**: Is this the same view type?
2. **Identity Check**: Is the structural/explicit identity the same?
3. **Diff Properties**: Did any @State, bindings, or environment values change?
4. **Update/Recreate**: Reuse view if type+identity match, recreate otherwise

```swift
struct ItemRow: View {
    let item: Item // Property change → body re-eval
    @State private var isExpanded = false // Preserved across body re-evals

    var body: some View {
        // When item changes, body is re-evaluated
        // When isExpanded toggles, body is re-evaluated
        // But @State lifetime is preserved
        VStack {
            Text(item.title)
            if isExpanded {
                Text(item.details)
            }
        }
    }
}
```

---

## Performance Implications

### View Body Evaluation

```swift
// Body is re-evaluated frequently
struct ContentView: View {
    @State private var count = 0

    var body: some View { // Called on every state/property change
        VStack {
            Text("\(count)") // Quick - just rendering
            ExpensiveView() // Problem! Entire view re-created
            Button("Inc") { count += 1 }
        }
    }
}

// Solution 1: Extract to property
struct ContentView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("\(count)")
            expensiveContent // Computed property, but same pattern
            Button("Inc") { count += 1 }
        }
    }

    var expensiveContent: some View {
        ExpensiveView()
    }
}

// Solution 2: Extract to subview with @ViewBuilder
struct ContentView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("\(count)")
            makeExpensive()
            Button("Inc") { count += 1 }
        }
    }

    @ViewBuilder
    func makeExpensive() -> some View {
        ExpensiveView()
    }
}
```

**Key insight**: Body is re-evaluated, but views with stable identity are reused efficiently.

---

## Lifetime of State Containers

### When @State is Created/Destroyed

```swift
struct ParentView: View {
    @State private var showChild = false

    var body: some View {
        VStack {
            Button("Toggle") { showChild.toggle() }

            if showChild {
                ChildView() // Created when showChild=true, destroyed when false
            }
        }
    }
}

struct ChildView: View {
    @State private var text = ""

    var body: some View {
        TextField("", text: $text) // Each time ChildView appears, @State is fresh
    }
}

// RESULT: User's input in TextField is LOST when showChild toggles to false!
```

**Solution**: Move state to parent or use data model

```swift
struct ParentView: View {
    @State private var childText = ""
    @State private var showChild = false

    var body: some View {
        VStack {
            Button("Toggle") { showChild.toggle() }

            if showChild {
                ChildView(text: $childText) // Binding preserves value
            }
        }
    }
}

struct ChildView: View {
    @Binding var text: String

    var body: some View {
        TextField("", text: $text)
    }
}
```

---

## Summary: The Mental Model

```
┌─────────────────────────────────────────────────────────┐
│           SwiftUI View Lifetime Model                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  View Type (struct ContentView)                        │
│     ↓                                                   │
│  View Instance (in hierarchy)                          │
│     ↓                                                   │
│  View Identity (explicit ID or structural)            │
│     ↓                                                   │
│  State Containers (@State, @StateObject)              │
│     ↓                                                   │
│  Lifetime (as long as identity exists in tree)        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Key rules**:
1. **Identity first**: Stable ID = stable lifetime
2. **State ownership**: @StateObject for owned, @ObservedObject for received
3. **No type erasure**: Use @ViewBuilder instead of AnyView
4. **Understand dependencies**: Pass down, don't reach up
5. **Respect conditionals**: View disappearing = state destroyed

---

## References

- WWDC 2021, Session 10022: "Demystify SwiftUI"
- https://developer.apple.com/videos/play/wwdc2021/10022/
- Related: WWDC 2021 Session 10019 "Demystify SwiftUI" (layout system)

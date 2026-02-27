# Property Wrapper Decision Tree

Decision matrix for auditing and selecting correct property wrappers in SwiftUI. Use this to identify wrapper mismatches automatically.

---

## Quick Reference Table

| Wrapper | Owns Object? | Value Type | Reference Type | Use Case |
|---------|------|------|------|----------|
| `@State` | YES | ✅ Yes | ❌ No* | Simple value state in view |
| `@StateObject` | YES | ❌ No | ✅ Yes | ViewModel/model objects created here |
| `@ObservedObject` | NO | ❌ No | ✅ Yes | Objects passed from parent |
| `@Binding` | NO | ✅ Yes | N/A | Value binding from parent |
| `@Environment` | NO | ✅ Yes | N/A | System environment values |
| `@EnvironmentObject` | NO | ❌ No | ✅ Yes | Custom objects injected via `.environmentObject()` |
| `@Observable` | YES | - | ✅ Yes | iOS 17+: Modern replacement for ObservableObject |

*`@State<ReferenceType>` works if the type is `@Observable` (iOS 17+), but NOT with standard ObservableObject

---

## Decision Flow

### Step 1: Do YOU Create This Object?

```
┌─ YES ─→ Go to Step 2
│
Is this object created
in this view's init?
│
└─ NO ──→ Is it passed from parent?
    │
    ├─ YES: Use @ObservedObject
    │
    └─ NO: Is it from .environmentObject()?
        └─ YES: Use @EnvironmentObject
        └─ NO: Is it a system value?
            └─ YES: Use @Environment
```

### Step 2: Object Creation - What Type?

```
┌─ Value Type (struct) ──→ Use @State
│                          (if <iOS 17)
│
Is it a value type
or reference type?
│
└─ Reference Type (class)
    │
    ├─ iOS 17+? Mark with @Observable
    │   └─ Use @State
    │
    └─ iOS 16 or earlier?
        └─ Make it ObservableObject
        └─ Use @StateObject
```

---

## Pattern Detection: Scanning Rules

### Scanner Pattern 1: @ObservedObject with Initialization

**Regex**: `@ObservedObject\s+(?:var|let)\s+\w+\s*=`

**Bad**:
```swift
@ObservedObject var viewModel = ContentViewModel()
```

**Why**: Object is recreated on every view update.

**Fix**: Use `@StateObject` instead.

```swift
@StateObject private var viewModel = ContentViewModel()
```

---

### Scanner Pattern 2: @State with ObservableObject

**Pattern**: `@State\s+(?:var|let)\s+\w+\s*:\s*\w+(?:ViewModel|Model)?\s*=\s*\w+\(`

**Bad**:
```swift
@State private var userData = UserViewModel()
```

**Why**: `@State` doesn't trigger updates when ObservableObject properties change.

**Fix**: Check if the class is `@Observable` (iOS 17+). If not, use `@StateObject`.

```swift
// iOS 17+
@Observable
class UserViewModel: ObservableObject { /* ... */ }

@State private var userData = UserViewModel() // Now works!

// iOS 16 or earlier
@StateObject private var userData = UserViewModel()
```

---

### Scanner Pattern 3: Missing @Published in ObservableObject

**Pattern**: `class\s+\w+:\s*ObservableObject\s*\{[^}]*\n\s*var\s+\w+\s*[:=]`

**Bad**:
```swift
class ViewModel: ObservableObject {
    var count = 0 // Missing @Published!

    func increment() {
        count += 1 // Won't notify SwiftUI
    }
}
```

**Why**: ObservableObject needs `@Published` to trigger view updates.

**Fix**: Add `@Published` to properties that should trigger updates.

```swift
class ViewModel: ObservableObject {
    @Published var count = 0

    func increment() {
        count += 1 // View updates automatically
    }
}
```

---

### Scanner Pattern 4: @EnvironmentObject Access Without Injection

**Pattern**: `@EnvironmentObject\s+var\s+\w+:\s+\w+`

**Bad** (compile-time issue if parent didn't inject):
```swift
struct DetailsView: View {
    @EnvironmentObject var appState: AppState

    var body: some View {
        Text(appState.title)
    }
}

// Parent forgot to inject
NavigationLink(destination: DetailsView()) { } // Runtime crash!
```

**Why**: If parent doesn't call `.environmentObject(appState)`, the view crashes.

**Fix**: Verify parent injects, or provide default.

```swift
// Option 1: Inject in parent
@main
struct App: App {
    @StateObject var appState = AppState()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState) // Inject
        }
    }
}

// Option 2: Use optional fallback (iOS 17+)
@EnvironmentObject private var appState: AppState?

var body: some View {
    if let appState = appState {
        Text(appState.title)
    } else {
        Text("State not available")
    }
}
```

---

### Scanner Pattern 5: Modifying @State in init()

**Pattern**: `init\s*\([^)]*\)\s*\{[^}]*\b\w+\s*=`

**Bad**:
```swift
struct ProfileView: View {
    @State private var userName: String

    init(initialName: String) {
        userName = initialName // Warning: modifying state during init
    }
}
```

**Why**: SwiftUI complains, and the value may not be preserved correctly.

**Fix**: Use the underscore-prefixed State wrapper.

```swift
struct ProfileView: View {
    @State private var userName: String

    init(initialName: String) {
        _userName = State(initialValue: initialName) // Correct way
    }
}
```

---

### Scanner Pattern 6: @Binding Misuse (Wrong Direction)

**Pattern**: `@Binding\s+var\s+\w+\s*=` (binding created locally)

**Bad**:
```swift
struct ChildView: View {
    @Binding var value = "" // Can't initialize binding here!
}
```

**Why**: Bindings are only created from parent `@State` or other sources with `$` prefix.

**Fix**: Accept binding from parent or create local @State.

```swift
// If you need to pass binding from parent:
struct ParentView: View {
    @State private var value = ""

    var body: some View {
        ChildView(value: $value) // Pass binding with $
    }
}

struct ChildView: View {
    @Binding var value: String // No initialization

    var body: some View {
        TextField("", text: $value)
    }
}
```

---

## Comprehensive Audit Checklist

When auditing a SwiftUI file, check these in order:

### State Management (Critical)
- [ ] No `@ObservedObject var x = Something()` patterns
- [ ] All owned objects use `@StateObject`
- [ ] All received objects use `@ObservedObject` or `@EnvironmentObject`
- [ ] ObservableObject properties marked with `@Published`
- [ ] `@State` only used with value types (unless iOS 17+ @Observable)
- [ ] No binding initialization in child views (`@Binding var x = ...`)

### Property Wrappers (iOS 17+ Modernization)
- [ ] Classes are marked `@Observable` when possible
- [ ] `@StateObject` replaced with `@State` when using `@Observable`
- [ ] No unnecessary `ObservableObject` + `@Published` if `@Observable` is available

### Initialization
- [ ] `@State` modified in init() uses `_state = State(initialValue:)`
- [ ] No @StateObject/ObservableObject without proper initialization

### Environment
- [ ] All `@EnvironmentObject` injections verified in parent
- [ ] No missing `.environmentObject()` calls in NavigationLinks

---

## iOS 17+ Modernization Guide

### Before (iOS 16)

```swift
class ViewModel: ObservableObject {
    @Published var count = 0
    @Published var name = ""

    func increment() {
        count += 1
    }
}

struct ContentView: View {
    @StateObject private var viewModel = ViewModel()

    var body: some View {
        VStack {
            Text("Count: \(viewModel.count)")
            TextField("Name", text: $viewModel.name)
            Button("Inc") { viewModel.increment() }
        }
    }
}
```

### After (iOS 17+)

```swift
@Observable
final class ViewModel {
    var count = 0
    var name = ""

    func increment() {
        count += 1
    }
}

struct ContentView: View {
    @State private var viewModel = ViewModel()

    var body: some View {
        VStack {
            Text("Count: \(viewModel.count)")
            TextField("Name", text: $viewModel.name)
            Button("Inc") { viewModel.increment() }
        }
    }
}
```

**Key changes**:
- No `ObservableObject`, `@Published` needed
- `@State` works directly with `@Observable` classes
- Cleaner, less boilerplate
- Automatic change tracking

---

## Common Mistakes and Fixes

### Mistake 1: Observable Class Without @Observable Macro

```swift
// ❌ iOS 17+: Forgot macro
class UserModel {
    var name: String = ""
}

@State var user = UserModel() // Won't work!

// ✅ Add @Observable
@Observable
class UserModel {
    var name: String = ""
}

@State var user = UserModel() // Now works
```

### Mistake 2: Passing Observable Without Binding

```swift
// ❌ Changes in child don't update parent
struct Parent: View {
    @State var user = UserModel()

    var body: some View {
        Child(user: user) // Passing value, not binding
    }
}

struct Child: View {
    var user: UserModel // Received value

    var body: some View {
        TextField("Name", text: $user.name) // Can't bind!
    }
}

// ✅ Pass as binding or @ObservedObject
struct Parent: View {
    @State var user = UserModel()

    var body: some View {
        Child(user: $user) // Pass binding
    }
}

struct Child: View {
    @Binding var user: UserModel // Receive binding

    var body: some View {
        TextField("Name", text: $user.name)
    }
}
```

### Mistake 3: Stale ObservableObject Instance

```swift
// ❌ Each view gets different instance
struct Parent: View {
    var body: some View {
        VStack {
            Child1(model: UserModel())
            Child2(model: UserModel()) // Different instance!
        }
    }
}

// ✅ Share single instance
struct Parent: View {
    @StateObject var model = UserModel()

    var body: some View {
        VStack {
            Child1(model: model)
            Child2(model: model) // Same instance
        }
    }
}
```

---

## Scanning Implementation

For automated audits, scan for these patterns:

```python
# Python regex patterns for CI/linting

PATTERNS = {
    "observed_with_init": r"@ObservedObject\s+(?:var|let)\s+\w+\s*=",
    "state_with_class": r"@State\s+(?:var|let)\s+\w+\s*:\s*\w+(?:ViewModel|Model)?\s*=\s*\w+\(",
    "missing_published": r"class\s+\w+:\s*ObservableObject\s*{[^}]*\n\s*var\s+\w+",
    "state_modification_in_init": r"init\s*\([^)]*\)\s*{[^}]*\b\w+\s*=",
    "binding_initialization": r"@Binding\s+(?:var|let)\s+\w+\s*=",
}

# For each match, flag as error and suggest fix
```

---

## References

- Apple SwiftUI Documentation: Property Wrappers
- WWDC 2021 Session 10022: Demystify SwiftUI (view identity and lifetime)
- WWDC 2023 Session 10149: Build an app with SwiftUI (iOS 17 @Observable)

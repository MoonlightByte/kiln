<!-- SPDX-License-Identifier: AGPL-3.0-only -->
<!-- Patterns analyzed from IceCubesApp, Copyright Thomas Ricouard, licensed under AGPL-3.0 -->

# IceCubesApp: Modern SwiftUI Patterns

> **License Notice:** This document analyzes architectural patterns from
> [IceCubesApp](https://github.com/Dimillian/IceCubesApp) by Thomas Ricouard,
> licensed under the [GNU Affero General Public License v3.0](https://www.gnu.org/licenses/agpl-3.0.en.html).
>
> **Copyleft Notice:** AGPL-3.0 is a strong copyleft license. This documentation is
> for educational purposes only. Any code derived from IceCubesApp must comply with
> AGPL-3.0 licensing requirements, including source code availability.

Extracted architectural patterns from IceCubesApp (https://github.com/Dimillian/IceCubesApp), the largest open-source SwiftUI application on GitHub. Uses iOS 16-18 features, semantic colors, and production-grade patterns.

---

## Architecture Overview

IceCubesApp is a Mastodon client with:
- 20,000+ lines of SwiftUI
- Supports iOS 16+, macOS 13+
- MVVM with SwiftUI's native tools
- Extensive use of modern APIs (iOS 17+)

**Key folders**:
- `/App/` - App entry point, environment setup
- `/Views/` - SwiftUI views organized by feature
- `/Models/` - Data models, API responses
- `/Services/` - Networking, data persistence
- `/Utilities/` - Helper functions, extensions

---

## Patterns

### Pattern 1: MVVM with Observable (iOS 17+)

**File**: `Views/Timelines/TimelineViewModel.swift`

**Pattern**:
```swift
import Foundation

@Observable
final class TimelineViewModel {
    @MainActor var posts: [Post] = []
    @MainActor var isLoading = false
    @MainActor var error: Error?

    private var postFetcher: PostFetcher
    private var cancellables = Set<AnyCancellable>()

    init(fetcher: PostFetcher = .shared) {
        self.postFetcher = fetcher
    }

    @MainActor
    func fetchPosts() async {
        isLoading = true
        error = nil

        do {
            posts = try await postFetcher.fetch()
        } catch {
            self.error = error
        }

        isLoading = false
    }
}
```

**Key insights**:
- Uses `@Observable` (iOS 17) instead of ObservableObject
- Marks main-thread properties with `@MainActor`
- No `@Published` needed - auto-tracking
- Clean, minimal boilerplate
- Error handling bundled in class

**Usage in view**:
```swift
struct TimelineView: View {
    @State private var viewModel = TimelineViewModel()

    var body: some View {
        List(viewModel.posts) { post in
            PostRow(post: post)
        }
        .task {
            await viewModel.fetchPosts()
        }
    }
}
```

---

### Pattern 2: Hierarchical Environment Setup

**Pattern**: Inject shared services at app level, access anywhere via `@Environment`

```swift
@main
struct App: App {
    @StateObject private var appState = AppState()
    @StateObject private var apiService = APIService()
    @StateObject private var imageCache = ImageCache()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(appState)
                .environment(apiService)
                .environment(imageCache)
        }
    }
}

// Deep in view hierarchy
struct DetailView: View {
    @Environment(APIService.self) var apiService
    @Environment(ImageCache.self) var imageCache

    var body: some View {
        // Access injected services
        Image(url: imageCache.cachedURL(for: id))
    }
}
```

**Benefits**:
- No prop drilling through 5+ view layers
- Dependency injection at app level
- Clean, testable architecture

---

### Pattern 3: Computed @State with Debounce

**Pattern**: Search field with debounced API calls

```swift
@Observable
final class SearchViewModel {
    @MainActor var searchText: String = "" {
        didSet {
            Task {
                await handleSearchTextChange(searchText)
            }
        }
    }

    @MainActor var results: [SearchResult] = []
    private var searchTask: Task<Void, Never>?

    @MainActor
    private func handleSearchTextChange(_ text: String) async {
        // Cancel previous search
        searchTask?.cancel()

        // Debounce: wait 300ms before searching
        try? await Task.sleep(nanoseconds: 300_000_000)

        if !Task.isCancelled {
            results = try await searchAPI.search(text)
        }
    }
}
```

**Usage**:
```swift
struct SearchView: View {
    @State private var viewModel = SearchViewModel()

    var body: some View {
        TextField("Search", text: $viewModel.searchText)
            .onChange(of: viewModel.searchText) { _, newValue in
                // Debounce handled in ViewModel
            }

        List(viewModel.results) { result in
            ResultRow(result: result)
        }
    }
}
```

**Key insights**:
- Property didSet triggers async work
- `Task.sleep()` for debounce (not Timer)
- Task cancellation respects view disappearance

---

### Pattern 4: SF Symbols with Semantic Rendering

**Pattern**: Icons with color, size, and weight flexibility

```swift
struct IconButton: View {
    let icon: String
    let size: CGFloat = 24
    let weight: Font.Weight = .regular
    let color: Color? = nil
    let action: () -> Void

    var body: some View {
        Button(action: action) {
            Image(systemName: icon)
                .font(.system(size: size, weight: weight))
                .foregroundColor(color ?? .accentColor)
        }
        .frame(width: 44, height: 44)
        .contentShape(Rectangle())
        .accessibilityLabel(icon)
    }
}

// Usage
IconButton(
    icon: "heart.fill",
    color: .red,
    action: { toggleLike() }
)
```

**Key insights**:
- Semantic colors (`.accentColor`, not hardcoded)
- Configurable without subclasses
- Minimum 44x44 touch target
- `contentShape()` for accurate tap area

---

### Pattern 5: Dynamic Type Support

**Pattern**: Responsive sizing that respects Accessibility settings

```swift
struct PostRow: View {
    @Environment(\.sizeCategory) var sizeCategory

    var body: some View {
        VStack(alignment: .leading, spacing: dynamicSpacing) {
            HStack {
                VStack(alignment: .leading, spacing: 2) {
                    Text(post.author)
                        .font(.headline)
                    Text(post.timestamp)
                        .font(.caption)
                }

                Spacer()

                Image(url: post.avatar)
                    .frame(width: avatarSize, height: avatarSize)
            }

            Text(post.content)
                .font(.body)
                .lineLimit(contentLineLimit)
        }
        .padding(dynamicPadding)
    }

    private var dynamicSpacing: CGFloat {
        sizeCategory.isAccessibilityCategory ? 8 : 4
    }

    private var dynamicPadding: CGFloat {
        sizeCategory.isAccessibilityCategory ? 16 : 12
    }

    private var avatarSize: CGFloat {
        sizeCategory.isAccessibilityCategory ? 48 : 36
    }

    private var contentLineLimit: Int {
        sizeCategory.isAccessibilityCategory ? 6 : 3
    }
}
```

**Key insights**:
- `@Environment(\.sizeCategory)` detects accessibility category
- Adjust spacing, sizes, line limits dynamically
- Avoid hardcoded spacing/sizes

---

### Pattern 6: Semantic Colors with Dark Mode

**Pattern**: Theme-aware colors without custom color sets

```swift
struct SemanticColor {
    static var primaryText: Color {
        Color(.label) // Black in light, white in dark
    }

    static var secondaryText: Color {
        Color(.secondaryLabel)
    }

    static var background: Color {
        Color(.systemBackground)
    }

    static var cardBackground: Color {
        Color(.secondarySystemBackground)
    }

    static var separator: Color {
        Color(.separator)
    }

    static var accentBlue: Color {
        Color(.systemBlue)
    }

    static var successGreen: Color {
        Color(.systemGreen)
    }

    static var destructiveRed: Color {
        Color(.systemRed)
    }
}

// Usage
struct CardView: View {
    var body: some View {
        VStack {
            Text("Title")
                .foregroundColor(SemanticColor.primaryText)
        }
        .background(SemanticColor.cardBackground)
    }
}
```

**Benefits**:
- Automatic light/dark mode support
- Respects accessibility high-contrast settings
- No custom color assets needed

---

### Pattern 7: Image Loading with Async/Await

**Pattern**: AsyncImage with fallback and error states

```swift
struct RemoteImage: View {
    let url: URL?
    let placeholder: Image

    var body: some View {
        if let url = url {
            AsyncImage(url: url) { phase in
                switch phase {
                case .empty:
                    placeholder
                        .resizable()
                        .scaledToFit()

                case .success(let image):
                    image
                        .resizable()
                        .scaledToFill()

                case .failure:
                    placeholder
                        .overlay(alignment: .center) {
                            Image(systemName: "photo.slash")
                                .foregroundColor(.gray)
                        }

                @unknown default:
                    placeholder
                }
            }
        } else {
            placeholder
        }
    }
}

// Usage
RemoteImage(
    url: user.avatarURL,
    placeholder: Image(systemName: "person.crop.circle")
)
.frame(width: 48, height: 48)
.clipShape(Circle())
```

**Key insights**:
- AsyncImage native (no third-party image library needed)
- Proper state handling (loading, success, failure)
- Placeholder support
- `@unknown default` for future cases

---

### Pattern 8: List Performance with LazyVStack

**Pattern**: Efficient long lists with lazy evaluation

```swift
struct PostsFeed: View {
    @State private var viewModel = FeedViewModel()

    var body: some View {
        ScrollView {
            LazyVStack(alignment: .leading, spacing: 0) {
                ForEach(viewModel.posts, id: \.id) { post in
                    PostRow(post: post)
                        .onAppear {
                            // Pagination
                            if post.id == viewModel.posts.last?.id {
                                Task {
                                    await viewModel.loadMore()
                                }
                            }
                        }

                    Divider()
                }
            }
        }
    }
}
```

**Benefits**:
- Only visible views created
- Smooth scrolling with 100+ items
- Pagination on-demand

---

### Pattern 9: Error Handling with .task and Alerts

**Pattern**: Centralized error display

```swift
@Observable
final class ViewModel {
    @MainActor var items: [Item] = []
    @MainActor var error: AppError?

    @MainActor
    func fetchItems() async {
        do {
            items = try await api.fetchItems()
        } catch {
            self.error = AppError(from: error)
        }
    }
}

struct ContentView: View {
    @State private var viewModel = ViewModel()

    var body: some View {
        List(viewModel.items) { item in
            ItemRow(item: item)
        }
        .task {
            await viewModel.fetchItems()
        }
        .alert("Error", isPresented: .constant(viewModel.error != nil)) {
            Button("OK") {
                viewModel.error = nil
            }
        } message: {
            if let error = viewModel.error {
                Text(error.localizedDescription)
            }
        }
    }
}
```

**Key insights**:
- Error stored in ViewModel
- `.task` calls async fetch
- Alert binding uses `.constant()` with error presence check
- Dismissal clears error

---

### Pattern 10: Safe Area and Notch Handling

**Pattern**: Proper insets for Dynamic Island and notches

```swift
struct HeaderView: View {
    var body: some View {
        VStack(spacing: 16) {
            HStack {
                Text("Title")
                    .font(.largeTitle)
                    .bold()

                Spacer()

                Image(systemName: "gear")
                    .font(.title2)
            }
            .padding(.horizontal, 16)
            .padding(.vertical, 12)
        }
        .background(Color(.systemBackground))
        .safeAreaInset(edge: .top) {
            Color.clear.frame(height: 0) // No extra space
        }
    }
}

// OR use ignoresSafeArea for full-bleed
struct FullBleedBackground: View {
    var body: some View {
        ZStack {
            Color.blue
                .ignoresSafeArea(edges: .all)

            VStack {
                Text("Content respects safe area")
                    .padding()
                    .foregroundColor(.white)
            }
        }
    }
}
```

**Key insights**:
- `.ignoresSafeArea()` for full-bleed backgrounds
- Content still respects safe areas
- Never hardcode insets; use `.safeAreaInset()`

---

## Architecture Takeaways

1. **@Observable over ObservableObject** (iOS 17+)
   - Less boilerplate
   - Better performance
   - Auto change tracking

2. **Hierarchical environment injection**
   - No prop drilling
   - Dependency injection at app level

3. **Async/await everywhere**
   - `.task` for async work
   - Task cancellation automatic
   - Clean error handling

4. **Semantic colors and styles**
   - Dark mode automatic
   - Accessibility built-in
   - No custom color assets

5. **Proper accessibility**
   - Minimum 44x44 touch targets
   - SF Symbols with labels
   - Dynamic Type support

6. **Performance discipline**
   - LazyVStack for lists
   - Stable IDs in ForEach
   - No AnyView abuse

---

## Migration Path from IceCubesApp Patterns

If you're currently using older SwiftUI patterns:

### Old Pattern (iOS 14-16)
```swift
class ViewModel: ObservableObject {
    @Published var items: [Item] = []

    func fetch() {
        // ...
    }
}

struct View: View {
    @StateObject var viewModel = ViewModel()

    var body: some View {
        List(viewModel.items) { item in
            Text(item.name)
        }
        .onAppear { viewModel.fetch() }
    }
}
```

### New Pattern (iOS 17+)
```swift
@Observable
final class ViewModel {
    var items: [Item] = []

    func fetch() async {
        // ...
    }
}

struct View: View {
    @State private var viewModel = ViewModel()

    var body: some View {
        List(viewModel.items) { item in
            Text(item.name)
        }
        .task {
            await viewModel.fetch()
        }
    }
}
```

**Migration checklist**:
- [ ] Replace `ObservableObject` classes with `@Observable`
- [ ] Remove `@Published` properties
- [ ] Add `@MainActor` to UI properties
- [ ] Replace `.onAppear { }` with `.task { }`
- [ ] Use `async/await` throughout
- [ ] Test on iOS 17+ simulator

---

## References

- IceCubesApp: https://github.com/Dimillian/IceCubesApp
- WWDC 2023 Session 10149: "Build an app with SwiftUI"
- WWDC 2023 Session 10160: "SwiftUI Essentials"
- Apple @Observable documentation: https://developer.apple.com/documentation/Observation

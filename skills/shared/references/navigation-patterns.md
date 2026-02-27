# Navigation Type Safety Patterns

Type-safe navigation patterns for Android (Jetpack Compose) and iOS (SwiftUI) to eliminate runtime crashes from malformed routes and enable compile-time argument validation.

**Updated**: February 2025
**Applies to**: Android (Navigation Compose 2.8+), iOS (SwiftUI 4+/iOS 16+)

---

## Core Principle: Compile-Time Safety Over Runtime Parsing

Navigation arguments should be validated at compile time, not runtime. String-based route construction is error-prone and loses type information.

---

## 1. Android Navigation Type Safety

### 1.1 kotlinx.serialization Integration (Navigation 2.8+)

The modern approach uses `@Serializable` route classes with Navigation Compose for full type safety.

#### BAD: String-Based Route Arguments

```kotlin
// Anti-pattern: String interpolation with manual parsing
sealed class Screen(val route: String) {
    object EventList : Screen("events")
    object EventDetail : Screen("events/{eventId}") {
        fun createRoute(eventId: String) = "events/$eventId"
    }
    object UserProfile : Screen("users/{userId}?tab={tab}") {
        fun createRoute(userId: String, tab: String? = null): String {
            return "users/$userId" + (tab?.let { "?tab=$it" } ?: "")
        }
    }
}

// Fragile: Manual argument extraction
NavHost(navController, startDestination = Screen.EventList.route) {
    composable(
        route = Screen.EventDetail.route,
        arguments = listOf(
            navArgument("eventId") { type = NavType.StringType }
        )
    ) { backStackEntry ->
        val eventId = backStackEntry.arguments?.getString("eventId")
            ?: throw IllegalStateException("eventId required")  // Runtime crash!
        EventDetailScreen(eventId)
    }
}

// Navigation call - typos compile but crash at runtime
navController.navigate("events/abc123")  // Works
navController.navigate("event/abc123")   // Typo - crashes at runtime!
```

**Problems**:
- Typos in route strings compile but crash at runtime
- No type checking for arguments (String vs Int vs UUID)
- Optional parameters require manual query string building
- Argument extraction is verbose and error-prone

#### GOOD: Serializable Route Classes

```kotlin
import kotlinx.serialization.Serializable

// Type-safe route definitions
@Serializable
object EventListRoute

@Serializable
data class EventDetailRoute(
    val eventId: String,
    val showComments: Boolean = false  // Optional with default
)

@Serializable
data class UserProfileRoute(
    val userId: String,
    val tab: ProfileTab = ProfileTab.OVERVIEW  // Enum support
)

@Serializable
enum class ProfileTab {
    OVERVIEW, EVENTS, REVIEWS
}

// NavHost with type-safe composables
NavHost(navController, startDestination = EventListRoute) {
    composable<EventListRoute> {
        EventListScreen(
            onEventClick = { eventId ->
                navController.navigate(EventDetailRoute(eventId))
            }
        )
    }

    composable<EventDetailRoute> { backStackEntry ->
        // Type-safe argument extraction - compiler validates types
        val route: EventDetailRoute = backStackEntry.toRoute()
        EventDetailScreen(
            eventId = route.eventId,
            showComments = route.showComments
        )
    }

    composable<UserProfileRoute> { backStackEntry ->
        val route: UserProfileRoute = backStackEntry.toRoute()
        UserProfileScreen(
            userId = route.userId,
            initialTab = route.tab
        )
    }
}

// Navigation is now type-safe
navController.navigate(EventDetailRoute(eventId = "abc123"))  // Compiler checks types
navController.navigate(EventDetailRoute(eventId = 123))       // Compile error: Int vs String
```

**Benefits**:
- Compile-time type checking for all arguments
- IDE autocomplete for route parameters
- Default values for optional parameters
- Enum support out of the box
- No string interpolation or manual parsing

---

### 1.2 Deep Link Handling with NavDeepLink

#### BAD: Manual Deep Link Parsing

```kotlin
// Anti-pattern: Manual URI parsing scattered across code
composable(
    route = "events/{eventId}",
    deepLinks = listOf(
        navDeepLink { uriPattern = "https://partyOn.app/events/{eventId}" }
    )
) { backStackEntry ->
    // Duplicate parsing logic, no validation
    val eventId = backStackEntry.arguments?.getString("eventId") ?: return@composable
    EventDetailScreen(eventId)
}
```

#### GOOD: Type-Safe Deep Links

```kotlin
@Serializable
data class EventDetailRoute(val eventId: String)

// Deep links work automatically with serializable routes
composable<EventDetailRoute>(
    deepLinks = listOf(
        navDeepLink<EventDetailRoute>(basePath = "https://partyon.app/events")
    )
) { backStackEntry ->
    val route: EventDetailRoute = backStackEntry.toRoute()
    EventDetailScreen(route.eventId)
}

// For complex patterns, use explicit URI templates
@Serializable
data class ConventionEventRoute(
    val conventionId: String,
    val eventId: String,
    val day: Int? = null
)

composable<ConventionEventRoute>(
    deepLinks = listOf(
        navDeepLink<ConventionEventRoute>(
            basePath = "https://partyon.app/conventions/{conventionId}/events"
        )
    )
)
```

---

### 1.3 SavedStateHandle Integration

#### BAD: Manual SavedStateHandle Extraction

```kotlin
@HiltViewModel
class EventDetailViewModel @Inject constructor(
    savedStateHandle: SavedStateHandle
) : ViewModel() {
    // Runtime crash if key doesn't exist or wrong type
    val eventId: String = savedStateHandle.get<String>("eventId")
        ?: throw IllegalStateException("eventId required")

    // No type safety - could be Int in route but String here
    val showComments: Boolean = savedStateHandle.get<Boolean>("showComments") ?: false
}
```

#### GOOD: Type-Safe SavedStateHandle with toRoute()

```kotlin
@Serializable
data class EventDetailRoute(
    val eventId: String,
    val showComments: Boolean = false
)

@HiltViewModel
class EventDetailViewModel @Inject constructor(
    savedStateHandle: SavedStateHandle
) : ViewModel() {
    // Type-safe extraction - matches route definition exactly
    private val route: EventDetailRoute = savedStateHandle.toRoute<EventDetailRoute>()

    val eventId: String = route.eventId
    val showComments: Boolean = route.showComments
}
```

---

### 1.4 Detection Patterns for Android

| Anti-Pattern | Regex/Pattern | Fix |
|--------------|---------------|-----|
| String route construction | `navController\.navigate\("[^"]+/\$` | Use serializable route class |
| Manual argument extraction | `arguments\?\.getString\(` | Use `backStackEntry.toRoute()` |
| NavArgument definitions | `navArgument\("[^"]+"\)` | Use `@Serializable` data class |
| String route sealed class | `sealed class.*route: String` | Convert to `@Serializable` objects |
| Deep link manual parsing | `intent\.data\?.getQueryParameter` | Use `navDeepLink<T>()` |

---

## 2. iOS Navigation Type Safety

### 2.1 NavigationPath with Codable Destinations

#### BAD: Loosely Typed NavigationPath

```swift
// Anti-pattern: Any Hashable in path, no type safety
struct ContentView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            EventListView()
                .navigationDestination(for: String.self) { eventId in
                    // No validation - any string works
                    EventDetailView(eventId: eventId)
                }
                .navigationDestination(for: Int.self) { userId in
                    // Mixing types in same path - confusing
                    UserProfileView(userId: userId)
                }
        }
    }

    func navigateToEvent(_ id: String) {
        path.append(id)  // Works
        path.append(123) // Also works - but is this an event or user?
    }
}
```

**Problems**:
- No distinction between different String routes
- Easy to append wrong type to path
- No compile-time validation of navigation calls

#### GOOD: Typed Navigation Destinations

```swift
// Define strongly-typed destinations
enum AppDestination: Hashable, Codable {
    case eventDetail(eventId: String, showComments: Bool = false)
    case userProfile(userId: String, tab: ProfileTab = .overview)
    case conventionSchedule(conventionId: String, day: Int?)
    case settings
}

enum ProfileTab: String, Hashable, Codable {
    case overview, events, reviews
}

struct ContentView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            EventListView(onEventTap: navigateToEvent)
                .navigationDestination(for: AppDestination.self) { destination in
                    destinationView(for: destination)
                }
        }
    }

    @ViewBuilder
    private func destinationView(for destination: AppDestination) -> some View {
        switch destination {
        case .eventDetail(let eventId, let showComments):
            EventDetailView(eventId: eventId, showComments: showComments)
        case .userProfile(let userId, let tab):
            UserProfileView(userId: userId, initialTab: tab)
        case .conventionSchedule(let conventionId, let day):
            ConventionScheduleView(conventionId: conventionId, selectedDay: day)
        case .settings:
            SettingsView()
        }
    }

    func navigateToEvent(_ eventId: String) {
        path.append(AppDestination.eventDetail(eventId: eventId))
    }

    func navigateToUserProfile(_ userId: String, tab: ProfileTab = .overview) {
        path.append(AppDestination.userProfile(userId: userId, tab: tab))
    }
}
```

**Benefits**:
- Exhaustive switch ensures all destinations handled
- Associated values provide type-safe arguments
- Codable enables navigation state persistence
- Clear distinction between different navigation targets

---

### 2.2 NavigationPath Persistence

```swift
// Save and restore navigation state across app launches
struct ContentView: View {
    @State private var path = NavigationPath()
    @AppStorage("navigationPath") private var savedPath: Data?

    var body: some View {
        NavigationStack(path: $path) {
            // ... content
        }
        .onAppear {
            restoreNavigationPath()
        }
        .onChange(of: path) { _, newPath in
            saveNavigationPath(newPath)
        }
    }

    private func saveNavigationPath(_ path: NavigationPath) {
        guard let representation = path.codable else { return }
        savedPath = try? JSONEncoder().encode(representation)
    }

    private func restoreNavigationPath() {
        guard let data = savedPath,
              let representation = try? JSONDecoder().decode(
                  NavigationPath.CodableRepresentation.self,
                  from: data
              )
        else { return }
        path = NavigationPath(representation)
    }
}
```

---

### 2.3 Deep Link Handling with onOpenURL

#### BAD: Scattered URL Parsing

```swift
// Anti-pattern: Manual URL parsing with string matching
struct ContentView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            // ...
        }
        .onOpenURL { url in
            // Fragile string parsing
            if url.host == "events",
               let eventId = url.pathComponents.dropFirst().first {
                path.append(eventId)  // Loosely typed
            } else if url.host == "users" {
                // Duplicate parsing logic...
            }
        }
    }
}
```

#### GOOD: Centralized Deep Link Router

```swift
// Centralized deep link parsing with type safety
enum DeepLinkRouter {
    static func destination(from url: URL) -> AppDestination? {
        guard let components = URLComponents(url: url, resolvingAgainstBaseURL: true),
              let host = components.host
        else { return nil }

        let pathComponents = components.path.split(separator: "/").map(String.init)
        let queryItems = components.queryItems ?? []

        switch host {
        case "events":
            guard let eventId = pathComponents.first else { return nil }
            let showComments = queryItems.first { $0.name == "comments" }?.value == "true"
            return .eventDetail(eventId: eventId, showComments: showComments)

        case "users":
            guard let userId = pathComponents.first else { return nil }
            let tabString = queryItems.first { $0.name == "tab" }?.value
            let tab = tabString.flatMap(ProfileTab.init(rawValue:)) ?? .overview
            return .userProfile(userId: userId, tab: tab)

        case "conventions":
            guard pathComponents.count >= 2,
                  let conventionId = pathComponents.first,
                  pathComponents[1] == "schedule"
            else { return nil }
            let day = queryItems.first { $0.name == "day" }?.value.flatMap(Int.init)
            return .conventionSchedule(conventionId: conventionId, day: day)

        default:
            return nil
        }
    }
}

struct ContentView: View {
    @State private var path = NavigationPath()

    var body: some View {
        NavigationStack(path: $path) {
            // ...
        }
        .onOpenURL { url in
            if let destination = DeepLinkRouter.destination(from: url) {
                path.append(destination)
            }
        }
    }
}
```

---

### 2.4 Detection Patterns for iOS

| Anti-Pattern | Pattern | Fix |
|--------------|---------|-----|
| String-based navigationDestination | `navigationDestination\(for: String\.self\)` | Use typed enum |
| Loosely typed path append | `path\.append\([^A-Z]` | Use destination enum |
| Manual URL parsing in view | `\.onOpenURL.*pathComponents` | Use DeepLinkRouter |
| Environment for navigation data | `@Environment.*navigationData` | Use NavigationPath with typed destinations |
| Scattered destination views | Multiple `.navigationDestination` for primitives | Single enum with @ViewBuilder switch |

---

## 3. Cross-Platform Considerations

### 3.1 Consistent Deep Link URL Schemes

Define a shared URL scheme that works across both platforms:

```
# URL Scheme Convention
partyon://events/{eventId}                    -> Event detail
partyon://events/{eventId}?comments=true      -> Event detail with comments
partyon://users/{userId}                      -> User profile (overview)
partyon://users/{userId}?tab=reviews          -> User profile (reviews tab)
partyon://conventions/{id}/schedule?day=2     -> Convention schedule, day 2

# Universal Links (HTTPS)
https://partyon.app/events/{eventId}
https://partyon.app/users/{userId}
```

### 3.2 Universal Links (iOS) / App Links (Android)

#### Android: App Links Configuration

```xml
<!-- AndroidManifest.xml -->
<activity android:name=".MainActivity">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data
            android:scheme="https"
            android:host="partyon.app"
            android:pathPrefix="/events" />
        <data
            android:scheme="https"
            android:host="partyon.app"
            android:pathPrefix="/users" />
    </intent-filter>
</activity>
```

#### iOS: Universal Links Configuration

```json
// apple-app-site-association (hosted at https://partyon.app/.well-known/)
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "TEAMID.com.partyon.ios",
        "paths": [
          "/events/*",
          "/users/*",
          "/conventions/*"
        ]
      }
    ]
  }
}
```

### 3.3 Navigation State Persistence

Both platforms should persist navigation state for seamless app restoration:

| Platform | Mechanism | Storage |
|----------|-----------|---------|
| Android | `SavedStateHandle` + `rememberSaveable` | Process death survival |
| iOS | `NavigationPath.codable` + `@AppStorage` | UserDefaults |

---

## 4. Migration Guide

### 4.1 Android: String Routes to Serializable

1. Add kotlinx.serialization dependency:
   ```gradle
   plugins {
       id("org.jetbrains.kotlin.plugin.serialization")
   }
   dependencies {
       implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.0")
   }
   ```

2. Convert sealed class routes to `@Serializable` objects/data classes

3. Update NavHost composables:
   ```kotlin
   // Before
   composable("events/{eventId}") { ... }

   // After
   composable<EventDetailRoute> { ... }
   ```

4. Update navigation calls:
   ```kotlin
   // Before
   navController.navigate("events/$eventId")

   // After
   navController.navigate(EventDetailRoute(eventId))
   ```

### 4.2 iOS: String Destinations to Enum

1. Define `AppDestination` enum with Codable conformance

2. Replace primitive `navigationDestination(for:)` with enum version

3. Update path operations:
   ```swift
   // Before
   path.append(eventId)

   // After
   path.append(AppDestination.eventDetail(eventId: eventId))
   ```

---

## 5. Quick Reference

### Android (Navigation Compose 2.8+)

```kotlin
// Route definition
@Serializable
data class EventDetailRoute(val eventId: String, val showComments: Boolean = false)

// NavHost setup
composable<EventDetailRoute> { backStackEntry ->
    val route: EventDetailRoute = backStackEntry.toRoute()
    EventDetailScreen(route.eventId, route.showComments)
}

// Navigation
navController.navigate(EventDetailRoute(eventId = "abc123"))

// ViewModel access
val route: EventDetailRoute = savedStateHandle.toRoute<EventDetailRoute>()
```

### iOS (SwiftUI 4+)

```swift
// Route definition
enum AppDestination: Hashable, Codable {
    case eventDetail(eventId: String, showComments: Bool = false)
}

// NavigationStack setup
NavigationStack(path: $path) {
    ContentView()
        .navigationDestination(for: AppDestination.self) { destination in
            switch destination {
            case .eventDetail(let eventId, let showComments):
                EventDetailView(eventId: eventId, showComments: showComments)
            }
        }
}

// Navigation
path.append(AppDestination.eventDetail(eventId: "abc123"))
```

---

## 6. Audit Checklist

### Android
- [ ] No string interpolation in `navigate()` calls
- [ ] No `navArgument()` definitions (use `@Serializable`)
- [ ] No `arguments?.getString()` extraction (use `toRoute()`)
- [ ] All routes are `@Serializable` objects or data classes
- [ ] Deep links use `navDeepLink<T>()`
- [ ] ViewModels use `savedStateHandle.toRoute<T>()`

### iOS
- [ ] No `navigationDestination(for: String.self)`
- [ ] No `navigationDestination(for: Int.self)` for IDs
- [ ] AppDestination enum is Codable
- [ ] NavigationPath used with typed destinations
- [ ] Deep link parsing centralized in router
- [ ] Navigation state persisted if app supports restoration

---

## References

- **Android Navigation Type Safety**: https://developer.android.com/guide/navigation/design/type-safety
- **kotlinx.serialization**: https://github.com/Kotlin/kotlinx.serialization
- **iOS NavigationStack**: https://developer.apple.com/documentation/swiftui/navigationstack
- **Universal Links**: https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app
- **App Links**: https://developer.android.com/training/app-links

# UI State Patterns - Loading, Empty, Error

## Overview
LLMs consistently assume the "happy path" where API calls succeed instantly. Production apps must handle all three states.

## The Three States
Every data-driven screen MUST implement:
1. **Loading State** - While fetching data
2. **Empty State** - When data exists but is empty
3. **Error State** - When fetch fails

## Loading State Patterns

### BANNED: Text-Only Loading
```kotlin
// NEVER do this
Text("Loading...")
CircularProgressIndicator() // alone, without context
```

```swift
// NEVER do this
Text("Loading...")
ProgressView() // alone, without context
```

### REQUIRED: Skeleton/Shimmer UI

**Android (Compose):**
```kotlin
@Composable
fun SkeletonRow(modifier: Modifier = Modifier) {
    Row(modifier.shimmer()) {
        Box(
            Modifier
                .size(48.dp)
                .clip(CircleShape)
                .background(Color.Gray.copy(alpha = 0.3f))
        )
        Column(Modifier.padding(start = 12.dp)) {
            Box(
                Modifier
                    .size(width = 120.dp, height = 16.dp)
                    .background(Color.Gray.copy(alpha = 0.3f))
            )
            Box(
                Modifier
                    .padding(top = 4.dp)
                    .size(width = 80.dp, height = 12.dp)
                    .background(Color.Gray.copy(alpha = 0.2f))
            )
        }
    }
}

// Shimmer modifier using Accompanist or custom implementation
fun Modifier.shimmer(): Modifier = composed {
    val transition = rememberInfiniteTransition(label = "shimmer")
    val alpha by transition.animateFloat(
        initialValue = 0.3f,
        targetValue = 0.7f,
        animationSpec = infiniteRepeatable(
            animation = tween(1000),
            repeatMode = RepeatMode.Reverse
        ),
        label = "shimmer_alpha"
    )
    this.graphicsLayer { this.alpha = alpha }
}
```

**iOS (SwiftUI):**
```swift
struct SkeletonRow: View {
    @State private var isAnimating = false

    var body: some View {
        HStack {
            Circle()
                .fill(Color.gray.opacity(0.3))
                .frame(width: 48, height: 48)
            VStack(alignment: .leading, spacing: 4) {
                RoundedRectangle(cornerRadius: 4)
                    .fill(Color.gray.opacity(0.3))
                    .frame(width: 120, height: 16)
                RoundedRectangle(cornerRadius: 4)
                    .fill(Color.gray.opacity(0.2))
                    .frame(width: 80, height: 12)
            }
        }
        .opacity(isAnimating ? 0.5 : 1.0)
        .animation(.easeInOut(duration: 1.0).repeatForever(), value: isAnimating)
        .onAppear { isAnimating = true }
    }
}

// Alternative using redacted modifier
struct SkeletonContent: View {
    var body: some View {
        ContentRow(data: .placeholder)
            .redacted(reason: .placeholder)
    }
}
```

### Loading State for Lists

**Android:**
```kotlin
@Composable
fun SkeletonList(itemCount: Int = 5) {
    LazyColumn {
        items(itemCount) {
            SkeletonRow(modifier = Modifier.padding(16.dp))
        }
    }
}
```

**iOS:**
```swift
struct SkeletonList: View {
    var itemCount: Int = 5

    var body: some View {
        List {
            ForEach(0..<itemCount, id: \.self) { _ in
                SkeletonRow()
            }
        }
        .listStyle(.plain)
    }
}
```

## Empty State Pattern

### REQUIRED Structure
Every empty state MUST have:
1. **Illustration** - Icon or image (not just emoji)
2. **Title** - What's empty (bold, larger text)
3. **Subtitle** - Why it's empty or what to do
4. **Call to Action** - Button to fix it (when applicable)

**Android:**
```kotlin
@Composable
fun EmptyState(
    icon: ImageVector,
    title: String,
    subtitle: String,
    actionText: String? = null,
    onAction: (() -> Unit)? = null,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(32.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Icon(
            imageVector = icon,
            contentDescription = null,
            modifier = Modifier.size(64.dp),
            tint = MaterialTheme.colorScheme.outline
        )
        Spacer(Modifier.height(16.dp))
        Text(
            text = title,
            style = MaterialTheme.typography.titleLarge,
            textAlign = TextAlign.Center
        )
        Spacer(Modifier.height(8.dp))
        Text(
            text = subtitle,
            style = MaterialTheme.typography.bodyMedium,
            color = MaterialTheme.colorScheme.onSurfaceVariant,
            textAlign = TextAlign.Center
        )
        if (actionText != null && onAction != null) {
            Spacer(Modifier.height(24.dp))
            Button(onClick = onAction) {
                Text(actionText)
            }
        }
    }
}

// Usage examples
EmptyState(
    icon = Icons.Default.Event,
    title = "No events yet",
    subtitle = "Events you join will appear here",
    actionText = "Browse Events",
    onAction = { navController.navigate("explore") }
)

EmptyState(
    icon = Icons.Default.Search,
    title = "No results found",
    subtitle = "Try adjusting your filters or search terms"
    // No CTA for search empty states - user adjusts filters
)
```

**iOS:**
```swift
struct EmptyState: View {
    let icon: String
    let title: String
    let subtitle: String
    var actionText: String? = nil
    var action: (() -> Void)? = nil

    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: icon)
                .font(.system(size: 64))
                .foregroundStyle(.secondary)

            Text(title)
                .font(.title2.bold())
                .multilineTextAlignment(.center)

            Text(subtitle)
                .font(.body)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)

            if let actionText = actionText, let action = action {
                Button(actionText, action: action)
                    .buttonStyle(.borderedProminent)
                    .padding(.top, 8)
            }
        }
        .padding(32)
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}

// Usage examples
EmptyState(
    icon: "calendar",
    title: "No events yet",
    subtitle: "Events you join will appear here",
    actionText: "Browse Events",
    action: { /* navigate */ }
)

EmptyState(
    icon: "magnifyingglass",
    title: "No results found",
    subtitle: "Try adjusting your filters or search terms"
)
```

### Context-Specific Empty States

| Screen | Icon | Title | Subtitle | CTA |
|--------|------|-------|----------|-----|
| My Events (Hosting) | `event`/`calendar.badge.plus` | "You're not hosting any events" | "Create an event to start gathering players" | "Host an Event" |
| My Events (Joined) | `event`/`calendar` | "No events joined yet" | "Browse events and join games that interest you" | "Browse Events" |
| Messages | `chat`/`message` | "No conversations yet" | "Start chatting with other players" | "Find Players" |
| Search Results | `search`/`magnifyingglass` | "No results found" | "Try different keywords or adjust filters" | (none) |
| Notifications | `notifications`/`bell` | "All caught up!" | "You'll see new notifications here" | (none) |

## Error State Pattern

### REQUIRED: Non-blocking + Retry

**Android:**
```kotlin
@Composable
fun ErrorState(
    message: String,
    onRetry: () -> Unit,
    modifier: Modifier = Modifier
) {
    Column(
        modifier = modifier
            .fillMaxSize()
            .padding(32.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Icon(
            imageVector = Icons.Default.Warning,
            contentDescription = null,
            modifier = Modifier.size(48.dp),
            tint = MaterialTheme.colorScheme.error
        )
        Spacer(Modifier.height(16.dp))
        Text(
            text = "Something went wrong",
            style = MaterialTheme.typography.titleMedium
        )
        Spacer(Modifier.height(8.dp))
        Text(
            text = message,
            style = MaterialTheme.typography.bodySmall,
            color = MaterialTheme.colorScheme.onSurfaceVariant,
            textAlign = TextAlign.Center
        )
        Spacer(Modifier.height(24.dp))
        OutlinedButton(onClick = onRetry) {
            Icon(Icons.Default.Refresh, contentDescription = null)
            Spacer(Modifier.width(8.dp))
            Text("Try Again")
        }
    }
}
```

**iOS:**
```swift
struct ErrorState: View {
    let message: String
    let onRetry: () -> Void

    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "exclamationmark.triangle")
                .font(.system(size: 48))
                .foregroundStyle(.red)

            Text("Something went wrong")
                .font(.headline)

            Text(message)
                .font(.caption)
                .foregroundStyle(.secondary)
                .multilineTextAlignment(.center)

            Button(action: onRetry) {
                Label("Try Again", systemImage: "arrow.clockwise")
            }
            .buttonStyle(.bordered)
        }
        .padding(32)
        .frame(maxWidth: .infinity, maxHeight: .infinity)
    }
}
```

### Snackbar/Toast for Non-Blocking Errors

For errors that don't prevent the entire screen from rendering (e.g., failed action on existing data):

**Android:**
```kotlin
// In ViewModel
sealed class UiEvent {
    data class ShowSnackbar(
        val message: String,
        val actionLabel: String? = null,
        val action: (() -> Unit)? = null
    ) : UiEvent()
}

private val _events = Channel<UiEvent>()
val events = _events.receiveAsFlow()

fun onActionFailed(error: Throwable) {
    viewModelScope.launch {
        _events.send(UiEvent.ShowSnackbar(
            message = error.message ?: "Action failed",
            actionLabel = "Retry",
            action = { retry() }
        ))
    }
}

// In Screen
val snackbarHostState = remember { SnackbarHostState() }

LaunchedEffect(Unit) {
    viewModel.events.collect { event ->
        when (event) {
            is UiEvent.ShowSnackbar -> {
                val result = snackbarHostState.showSnackbar(
                    message = event.message,
                    actionLabel = event.actionLabel
                )
                if (result == SnackbarResult.ActionPerformed) {
                    event.action?.invoke()
                }
            }
        }
    }
}

Scaffold(snackbarHost = { SnackbarHost(snackbarHostState) }) { ... }
```

**iOS:**
```swift
// Using toast/banner pattern
struct ContentView: View {
    @StateObject var viewModel = MyViewModel()
    @State private var showError = false
    @State private var errorMessage = ""

    var body: some View {
        content
            .overlay(alignment: .top) {
                if showError {
                    ErrorBanner(message: errorMessage) {
                        viewModel.retry()
                    }
                    .transition(.move(edge: .top))
                }
            }
            .onReceive(viewModel.$error) { error in
                if let error = error {
                    errorMessage = error.localizedDescription
                    withAnimation { showError = true }
                    DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
                        withAnimation { showError = false }
                    }
                }
            }
    }
}

struct ErrorBanner: View {
    let message: String
    let onRetry: () -> Void

    var body: some View {
        HStack {
            Image(systemName: "exclamationmark.circle.fill")
                .foregroundStyle(.white)
            Text(message)
                .foregroundStyle(.white)
                .font(.subheadline)
            Spacer()
            Button("Retry", action: onRetry)
                .foregroundStyle(.white)
                .font(.subheadline.bold())
        }
        .padding()
        .background(Color.red)
    }
}
```

## State Machine Pattern

### REQUIRED: Sealed Class/Enum for UI State

Never use multiple booleans like `isLoading`, `hasError`, `isEmpty`. Use a single sealed class/enum.

**Android:**
```kotlin
sealed class UiState<out T> {
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
    object Empty : UiState<Nothing>()
}

// In ViewModel
class EventsViewModel : ViewModel() {
    private val _uiState = MutableStateFlow<UiState<List<Event>>>(UiState.Loading)
    val uiState: StateFlow<UiState<List<Event>>> = _uiState.asStateFlow()

    fun loadEvents() {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            try {
                val events = repository.getEvents()
                _uiState.value = if (events.isEmpty()) {
                    UiState.Empty
                } else {
                    UiState.Success(events)
                }
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }

    fun retry() = loadEvents()
}

// In Screen
@Composable
fun EventsScreen(viewModel: EventsViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsState()

    when (val state = uiState) {
        is UiState.Loading -> SkeletonList()
        is UiState.Success -> EventList(events = state.data)
        is UiState.Error -> ErrorState(
            message = state.message,
            onRetry = viewModel::retry
        )
        is UiState.Empty -> EmptyState(
            icon = Icons.Default.Event,
            title = "No events found",
            subtitle = "Check back later for new events",
            actionText = "Refresh",
            onAction = viewModel::retry
        )
    }
}
```

**iOS:**
```swift
enum ViewState<T> {
    case loading
    case success(T)
    case error(String)
    case empty
}

// In ViewModel
@MainActor
class EventsViewModel: ObservableObject {
    @Published var viewState: ViewState<[Event]> = .loading

    func loadEvents() async {
        viewState = .loading
        do {
            let events = try await repository.getEvents()
            viewState = events.isEmpty ? .empty : .success(events)
        } catch {
            viewState = .error(error.localizedDescription)
        }
    }

    func retry() {
        Task { await loadEvents() }
    }
}

// In View
struct EventsView: View {
    @StateObject var viewModel = EventsViewModel()

    var body: some View {
        Group {
            switch viewModel.viewState {
            case .loading:
                SkeletonList()
            case .success(let events):
                EventList(events: events)
            case .error(let message):
                ErrorState(message: message, onRetry: viewModel.retry)
            case .empty:
                EmptyState(
                    icon: "calendar",
                    title: "No events found",
                    subtitle: "Check back later for new events",
                    actionText: "Refresh",
                    action: viewModel.retry
                )
            }
        }
        .task { await viewModel.loadEvents() }
    }
}
```

### Why Sealed Classes Over Booleans

```kotlin
// BAD - Multiple booleans create impossible states
class BadViewModel {
    var isLoading = false
    var hasError = false
    var isEmpty = false
    var data: List<Event>? = null
    // What if isLoading AND hasError are both true? Undefined behavior!
}

// GOOD - Only one state at a time is possible
sealed class UiState<out T> {
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
    object Empty : UiState<Nothing>()
}
```

## Pull-to-Refresh Integration

When implementing pull-to-refresh, preserve existing data during refresh:

**Android:**
```kotlin
sealed class UiState<out T> {
    object Loading : UiState<Nothing>()
    data class Success<T>(val data: T, val isRefreshing: Boolean = false) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
    object Empty : UiState<Nothing>()
}

// In Screen
val state = uiState as? UiState.Success
val pullRefreshState = rememberPullRefreshState(
    refreshing = state?.isRefreshing == true,
    onRefresh = viewModel::refresh
)

Box(Modifier.pullRefresh(pullRefreshState)) {
    // content
    PullRefreshIndicator(
        refreshing = state?.isRefreshing == true,
        state = pullRefreshState,
        modifier = Modifier.align(Alignment.TopCenter)
    )
}
```

**iOS:**
```swift
List(events) { event in
    EventRow(event: event)
}
.refreshable {
    await viewModel.refresh()
}
```

## Checklist

Before submitting any data-driven screen, verify:

- [ ] **Loading state** shows skeleton/shimmer (not just spinner or "Loading...")
- [ ] **Empty state** has illustration + title + subtitle + CTA (where appropriate)
- [ ] **Error state** has retry button and clear message
- [ ] All three states render without crashes
- [ ] State is handled via sealed class/enum (not booleans)
- [ ] Pull-to-refresh preserves existing data during refresh
- [ ] Non-blocking errors use snackbar/toast instead of full-screen error
- [ ] Skeleton matches the shape of actual content (same layout structure)

## Common Mistakes to Avoid

1. **Showing spinner over empty screen** - Use skeleton that matches content shape
2. **Generic "Error" message** - Include actionable context ("Network unavailable - check connection")
3. **No retry mechanism** - Users should never be stuck; always provide retry
4. **Boolean state soup** - `isLoading && !hasError && data != null` is unmaintainable
5. **Empty state without context** - "No items" is useless; explain why and what to do
6. **Blocking errors for non-critical failures** - Use snackbar for action failures, full-screen only for load failures

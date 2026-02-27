# Jetpack Compose Anti-Patterns Database

LLM-generated code frequently contains these mistakes. Use this reference to catch and fix them.

---

## State Management (Critical)

### AP-001: Passing MutableState to Children

**Severity**: 🔴 Critical | **Confidence**: 95

**Description**: Passing `MutableState<T>` directly to child composables breaks unidirectional data flow and makes state changes unpredictable.

**Detection Pattern** (Regex):
```regex
fun\s+\w+\([^)]*:\s*MutableState<[^>]+>[^)]*\)
```

**Detection Script** (grep):
```bash
grep -r "MutableState<.*>" --include="*.kt" src/ | grep "fun.*("
# Flags any function parameter typed as MutableState
```

**Bad Example**:
```kotlin
@Composable
fun Parent() {
    val count = remember { mutableStateOf(0) }
    Child(count) // Passing MutableState directly
}

@Composable
fun Child(count: MutableState<Int>) {
    Button(onClick = { count.value++ }) {
        Text("Count: ${count.value}")
    }
}
```

**Good Example**:
```kotlin
@Composable
fun Parent() {
    var count by remember { mutableStateOf(0) }
    Child(count = count, onIncrement = { count++ })
}

@Composable
fun Child(count: Int, onIncrement: () -> Unit) {
    Button(onClick = onIncrement) {
        Text("Count: $count")
    }
}
```

**Rationale**: State hoisting enables testing, preview, and reuse. Child composables should be stateless functions.

---

### AP-002: Reading Scroll State in Composition

**Severity**: 🔴 Critical | **Confidence**: 90

**Description**: Reading scroll position during composition causes recomposition on every scroll frame (60+ times per second).

**Detection Pattern** (Regex):
```regex
val\s+\w+\s*=\s*scrollState\.(firstVisibleItemIndex|layoutInfo|canScrollForward|canScrollBackward)
```

**Detection Script** (grep):
```bash
grep -r "scrollState\.\(firstVisibleItemIndex\|layoutInfo\|canScrollForward\|canScrollBackward\)" --include="*.kt" src/
# Flags scroll state accessed directly in composition body
```

**Bad Example**:
```kotlin
@Composable
fun ScrollingList(scrollState: LazyListState) {
    val firstVisible = scrollState.firstVisibleItemIndex // Reads every frame!

    LazyColumn(state = scrollState) {
        items(100) { index ->
            Text("Item $index, first visible: $firstVisible")
        }
    }
}
```

**Good Example**:
```kotlin
@Composable
fun ScrollingList(scrollState: LazyListState) {
    val showButton by remember {
        derivedStateOf { scrollState.firstVisibleItemIndex > 10 }
    }

    Box {
        LazyColumn(state = scrollState) {
            items(100) { index -> Text("Item $index") }
        }
        if (showButton) {
            ScrollToTopButton()
        }
    }
}
```

**Rationale**: `derivedStateOf` only triggers recomposition when the derived value changes, not on every scroll frame.

---

### AP-003: Creating State in Loops

**Severity**: 🔴 Critical | **Confidence**: 95

**Description**: Using `remember` inside loops or conditionals creates unpredictable state behavior.

**Detection Pattern** (Regex):
```regex
(forEach|repeat|for\s*\()\s*\{[^}]*remember\s*\{
```

**Detection Script** (grep):
```bash
# Complex detection: look for remember inside loop blocks
awk '/forEach|repeat|for \(/{p=1} p && /remember \{/{print FILENAME":"NR":"$0} /\}/{p=0}' --include="*.kt" -r src/
```

**Simpler Script**:
```bash
grep -r "forEach\|repeat\|for (" --include="*.kt" src/ | grep -l "remember"
# Manual review of these files needed to confirm
```

**Bad Example**:
```kotlin
@Composable
fun ItemList(items: List<Item>) {
    Column {
        items.forEach { item ->
            val expanded = remember { mutableStateOf(false) } // Wrong!
            ItemRow(item, expanded.value) { expanded.value = !expanded.value }
        }
    }
}
```

**Good Example**:
```kotlin
@Composable
fun ItemList(items: List<Item>) {
    val expandedStates = remember(items) {
        mutableStateMapOf<String, Boolean>()
    }

    Column {
        items.forEach { item ->
            val expanded = expandedStates[item.id] ?: false
            ItemRow(item, expanded) {
                expandedStates[item.id] = !expanded
            }
        }
    }
}
```

**Rationale**: `remember` is keyed by call site, not loop iteration. Use a map or `LazyColumn` with keys.

---

### AP-004: Missing rememberSaveable

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Using `remember` for user input that should survive configuration changes.

**Detection Pattern** (Regex):
```regex
(var|val)\s+\w+\s+by\s+remember\s*\{\s*mutableStateOf
```

**Detection Script** (grep):
```bash
grep -r "remember\s*{\s*mutableStateOf" --include="*.kt" src/ | grep -v "rememberSaveable"
# Flags all remember-based state (manual review needed)
# Filter for TextField/TextInput components to confirm user input
grep -B5 "TextField\|OutlinedTextField" --include="*.kt" -r src/ | grep -B5 "remember.*mutableStateOf"
```

**Bad Example**:
```kotlin
@Composable
fun SearchScreen() {
    var query by remember { mutableStateOf("") } // Lost on rotation!
    TextField(value = query, onValueChange = { query = it })
}
```

**Good Example**:
```kotlin
@Composable
fun SearchScreen() {
    var query by rememberSaveable { mutableStateOf("") }
    TextField(value = query, onValueChange = { query = it })
}
```

**Rationale**: `rememberSaveable` uses the saved instance state bundle to preserve values across configuration changes.

---

## Performance (High)

### AP-005: Creating Modifiers in Composition

**Severity**: 🟠 Important | **Confidence**: 90

**Description**: Creating modifier chains inside composable functions allocates new objects on every recomposition.

**Detection Pattern** (Regex):
```regex
modifier\s*=\s*Modifier\s*\n\s*\.(fillMaxWidth|padding|clip|size|background|border|alpha)
```

**Detection Script** (grep):
```bash
# Flag Modifier chains inside composables
grep -A10 "@Composable" --include="*.kt" -r src/ | grep "modifier\s*=\s*Modifier"
# Then check if it's inside function (no private val outside)
grep -r "modifier\s*=\s*Modifier" --include="*.kt" src/ | grep -v "^.*private.*val"
```

**Compound Check** (using ast-grep style pseudo-code):
```
Flags any Modifier(...) allocation inside a @Composable body
(not in top-level or companion object scope)
```

**Bad Example**:
```kotlin
@Composable
fun MyCard(item: Item) {
    Card(
        modifier = Modifier // New chain every recomposition
            .fillMaxWidth()
            .padding(16.dp)
            .clip(RoundedCornerShape(12.dp))
    ) {
        Text(item.title)
    }
}
```

**Good Example**:
```kotlin
private val cardModifier = Modifier
    .fillMaxWidth()
    .padding(16.dp)
    .clip(RoundedCornerShape(12.dp))

@Composable
fun MyCard(item: Item) {
    Card(modifier = cardModifier) {
        Text(item.title)
    }
}
```

**Rationale**: Static modifiers are created once. Dynamic modifiers should use `Modifier.then()` with remembered parts.

---

### AP-006: Unstable Collections in State

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Using standard Kotlin collections in state classes makes them unstable for recomposition.

**Detection Pattern** (Regex):
```regex
(data\s+class|class)\s+\w+.*\{\s*.*:\s*(List|Set|Map)<[^>]+>\s*=
```

**Detection Script** (grep):
```bash
# Find data classes with mutable collections
grep -r "val\s\+\w\+:\s\*(List\|Set\|Map)<" --include="*.kt" src/ | grep -v "ImmutableList\|ImmutableSet\|ImmutableMap\|persistentListOf"
# Should also check for missing @Immutable annotation
grep -B2 "val.*:\s*List<\|Set<\|Map<" --include="*.kt" -r src/ | grep -v "@Immutable"
```

**Compiler Metrics Check**:
```bash
# After building with metrics enabled:
grep "UNSTABLE:.*List\|Set\|Map" build/compose_metrics/*_classes.txt
```

**Bad Example**:
```kotlin
data class UiState(
    val items: List<Item> = emptyList(), // Unstable!
    val tags: Set<String> = emptySet()   // Unstable!
)
```

**Good Example**:
```kotlin
@Immutable
data class UiState(
    val items: ImmutableList<Item> = persistentListOf(),
    val tags: ImmutableSet<String> = persistentSetOf()
)
```

**Rationale**: Compose compiler can't prove `List<T>` is stable. Use `kotlinx.collections.immutable` or mark class `@Immutable`.

---

### AP-007: Heavy Computation Without remember

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: Performing expensive operations in composition without caching.

**Detection Pattern** (Regex):
```regex
val\s+\w+\s*=\s*(sortedBy|sorted|groupBy|filter|map|distinctBy|partition)\s*\{
```

**Detection Script** (grep):
```bash
# Find expensive operations in composable bodies
grep -r "sortedBy\|sorted\|groupBy\|filter\|map\|distinctBy\|partition" --include="*.kt" src/ | grep -v "remember\s*{"
# Verify they're not inside remember() calls
# Manual review: check context (should be wrapped in remember)
```

**Compound Check** (pseudo-code):
```
Flag if found:
- .sortedBy/.sorted/.groupBy/.filter/.distinctBy/.partition
- NOT inside remember { } block
- Inside @Composable function body
```

**Bad Example**:
```kotlin
@Composable
fun ExpensiveScreen(data: List<Item>) {
    val sortedData = data.sortedBy { it.timestamp } // Runs every recomposition
    val grouped = sortedData.groupBy { it.category } // Runs every recomposition

    LazyColumn {
        grouped.forEach { (category, items) ->
            // ...
        }
    }
}
```

**Good Example**:
```kotlin
@Composable
fun ExpensiveScreen(data: List<Item>) {
    val grouped = remember(data) {
        data.sortedBy { it.timestamp }.groupBy { it.category }
    }

    LazyColumn {
        grouped.forEach { (category, items) ->
            // ...
        }
    }
}
```

**Rationale**: `remember(key)` caches the result until the key changes.

---

### AP-008: Using Column Instead of LazyColumn

**Severity**: 🟡 Warning | **Confidence**: 75

**Description**: Using `Column` for lists longer than 10-20 items wastes memory.

**Detection Pattern** (Regex):
```regex
Column\s*\([^)]*verticalScroll\s*\(.*\)\s*\)\s*\{[^}]*(forEach|items\.map|items\.forEach)
```

**Detection Script** (grep):
```bash
# Find Column with verticalScroll + iteration
grep -r "Column.*verticalScroll" --include="*.kt" -r src/
# Then inspect those files for forEach/map iterations
grep -B5 -A10 "Column.*verticalScroll" --include="*.kt" -r src/ | grep -A10 "forEach\|\.map\s*{"
```

**Automated Check**:
```bash
# If using Column + scroll + forEach:
for file in $(grep -r "forEach\|\.map" --include="*.kt" -l src/); do
  if grep -q "Column.*verticalScroll" "$file"; then
    echo "WARN: $file may have Column + scroll + iteration"
  fi
done
```

**Bad Example**:
```kotlin
@Composable
fun EventList(events: List<Event>) {
    Column(modifier = Modifier.verticalScroll(rememberScrollState())) {
        events.forEach { event ->
            EventCard(event) // All 100+ items composed at once
        }
    }
}
```

**Good Example**:
```kotlin
@Composable
fun EventList(events: List<Event>) {
    LazyColumn {
        items(events, key = { it.id }) { event ->
            EventCard(event) // Only visible items composed
        }
    }
}
```

**Rationale**: `LazyColumn` only composes visible items, enabling smooth scrolling for large lists.

---

### AP-009: Missing Keys in LazyColumn

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Not providing stable keys causes state loss and poor animation.

**Detection Pattern** (Regex):
```regex
(LazyColumn|LazyRow|LazyVerticalGrid|LazyHorizontalGrid)\s*\{[^}]*items\s*\([^,)]+\)\s*\{
```

**Detection Script** (grep):
```bash
# Find items() calls without key= parameter
grep -r "items\s*(" --include="*.kt" src/ | grep -v "key\s*="
# Manual review: all items() without key= parameter should be flagged
# Context check: should be inside LazyColumn/LazyRow/LazyGrid
grep -B10 "items\s*([^,)]" --include="*.kt" -r src/ | grep -B10 "LazyColumn\|LazyRow"
```

**Compound Check**:
```
Flag if found in LazyColumn/LazyRow/LazyGrid:
- items(list) { ... }           <- No key parameter
- items(list.size) { ... }      <- Index-based (breaks on reorder)
```

**Bad Example**:
```kotlin
LazyColumn {
    items(events) { event -> // No keys!
        EventCard(event)
    }
}
```

**Good Example**:
```kotlin
LazyColumn {
    items(events, key = { it.id }) { event ->
        EventCard(event)
    }
}
```

**Rationale**: Keys enable correct state preservation during reordering and efficient diffing.

---

## Accessibility (Critical)

### AP-010: Missing contentDescription

**Severity**: 🔴 Critical | **Confidence**: 95

**Description**: Images and icons without content descriptions are invisible to screen readers.

**Detection Pattern** (Regex):
```regex
(Icon|Image|AsyncImage)\s*\([^)]*contentDescription\s*=\s*null
```

**Detection Script** (grep):
```bash
# Find Icon/Image with contentDescription = null
grep -r "Icon\|Image\|AsyncImage" --include="*.kt" src/ | grep "contentDescription\s*=\s*null"
# OR find Icon/Image missing contentDescription entirely
grep -r "^\s*Icon\|^\s*Image\|^\s*AsyncImage" --include="*.kt" src/ | grep -v "contentDescription"
```

**Compound Check**:
```bash
# Find all Icon/Image calls and check for contentDescription
awk '/Icon\(|Image\(|AsyncImage\(/{flag=1} flag && /contentDescription/{flag=0} flag && /\)/{print FILENAME":"NR": Missing contentDescription"}' --include="*.kt" -r src/
```

**Bad Example**:
```kotlin
Icon(
    imageVector = Icons.Default.Favorite,
    contentDescription = null // Screen reader skips this!
)

Image(
    painter = painterResource(R.drawable.event_banner),
    contentDescription = null // What is this image?
)
```

**Good Example**:
```kotlin
Icon(
    imageVector = Icons.Default.Favorite,
    contentDescription = "Add to favorites"
)

Image(
    painter = painterResource(R.drawable.event_banner),
    contentDescription = "Event banner showing players at a gaming table"
)
```

**Rationale**: TalkBack reads `contentDescription` to describe UI elements. Decorative images should use `contentDescription = null` with `Modifier.semantics { invisibleToUser() }`.

---

### AP-011: Small Touch Targets

**Severity**: 🔴 Critical | **Confidence**: 90

**Description**: Interactive elements smaller than 48x48dp fail WCAG and Material guidelines.

**Detection Pattern** (Regex):
```regex
\.size\s*\(\s*(\d+)\s*\.dp\s*\)\s*.*\.clickable|Icon\s*\([^)]*\.size\s*\(\s*([0-9]+)\s*\.dp
```

**Detection Script** (grep):
```bash
# Find clickable elements with explicit size < 48dp
grep -r "\.size\s*(\s*\(1[0-9]\|2[0-9]\|3[0-9]\|4[0-7]\)\.dp" --include="*.kt" src/ | grep "clickable"
# Find Icon with size parameter < 48dp
grep -r "Icon\|Image" --include="*.kt" src/ | grep "\.size\s*(\s*\(1[0-9]\|2[0-9]\|3[0-9]\|4[0-7]\)\.dp"
```

**Better Detection** (manual):
```bash
# Look for custom clickables without minimumInteractiveComponentSize
grep -r "\.clickable" --include="*.kt" src/ | grep -v "IconButton\|Button\|minimumInteractiveComponentSize"
# Check if size is hardcoded < 48dp
```

**Bad Example**:
```kotlin
Icon(
    imageVector = Icons.Default.Close,
    contentDescription = "Close",
    modifier = Modifier
        .size(24.dp)
        .clickable { onClose() }
)
```

**Good Example**:
```kotlin
IconButton(onClick = onClose) {
    Icon(
        imageVector = Icons.Default.Close,
        contentDescription = "Close"
    )
}

// Or manually:
Icon(
    imageVector = Icons.Default.Close,
    contentDescription = "Close",
    modifier = Modifier
        .clickable { onClose() }
        .minimumInteractiveComponentSize() // Ensures 48x48dp
)
```

**Rationale**: `IconButton` automatically provides 48x48dp touch target. Use `minimumInteractiveComponentSize()` for custom clickables.

---

### AP-012: Wrong Semantics Role

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: Using wrong or missing semantic roles confuses screen readers.

**Detection Pattern** (Regex):
```regex
\.clickable\s*\{[^}]*\}(?!.*\.semantics|Role\.|toggleable)
```

**Detection Script** (grep):
```bash
# Find clickable elements without semantics role
grep -r "\.clickable" --include="*.kt" src/ | grep -v "\.semantics\|Role\.\|toggleable\|checkbox\|switch"
# Manual review: check if these should have role annotations
grep -B5 "\.clickable" --include="*.kt" -r src/ | grep -B5 "Row\|Column\|Box" | head -20
```

**Compound Check** (AST-like):
```
Flag if found:
- .clickable { ... } on Row/Column/Box
- NO .semantics { role = ... }
- NOT using RadioButton/Checkbox/Switch
```

**Bad Example**:
```kotlin
Row(
    modifier = Modifier.clickable { onSelect() }
) {
    Text("Option A")
    if (selected) Icon(Icons.Default.Check, "Selected")
}
```

**Good Example**:
```kotlin
Row(
    modifier = Modifier
        .clickable { onSelect() }
        .semantics {
            role = Role.RadioButton
            selected = isSelected
        }
) {
    Text("Option A")
    if (selected) Icon(Icons.Default.Check, null) // Decorative when selected state announced
}
```

**Rationale**: Semantic roles tell assistive technologies how to interact with elements.

---

## Styling (Important)

### AP-013: Hardcoded Colors

**Severity**: 🟠 Important | **Confidence**: 90

**Description**: Using hex colors or `Color.*` instead of theme tokens breaks theming and dark mode.

**Detection Pattern** (Regex):
```regex
color\s*=\s*(Color\(0x|Color\.(White|Black|Red|Green|Blue|Gray|Cyan|Magenta|Yellow))
```

**Detection Script** (grep):
```bash
# Find hardcoded Color() values
grep -r "color\s*=\s*Color\s*(" --include="*.kt" src/ | grep -v "MaterialTheme.colorScheme"
# Find Color.* constants
grep -r "color\s*=\s*Color\.\(White\|Black\|Red\|Green\|Blue\)" --include="*.kt" src/
```

**Compound Check**:
```bash
# Flag any color assignment not using MaterialTheme
grep -r "\.color\s*=" --include="*.kt" src/ | grep -v "MaterialTheme\|colorScheme"
# Manual review for theme tokens
```

**Bad Example**:
```kotlin
Text(
    text = "Title",
    color = Color(0xFF1976D2) // Hardcoded blue
)

Surface(color = Color.White) { // Hardcoded white
    // content
}
```

**Good Example**:
```kotlin
Text(
    text = "Title",
    color = MaterialTheme.colorScheme.primary
)

Surface(color = MaterialTheme.colorScheme.surface) {
    // content
}
```

**Rationale**: Theme tokens adapt to light/dark mode and dynamic color.

---

### AP-014: Hardcoded Font Sizes

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Using explicit `sp` values instead of typography tokens.

**Detection Pattern** (Regex):
```regex
fontSize\s*=\s*\d+\s*\.sp|fontWeight\s*=\s*FontWeight\.(Bold|SemiBold|Medium|Light|Thin)
```

**Detection Script** (grep):
```bash
# Find hardcoded fontSize
grep -r "fontSize\s*=\s*[0-9]\+\s*\.sp" --include="*.kt" src/
# Find hardcoded fontWeight
grep -r "fontWeight\s*=\s*FontWeight\." --include="*.kt" src/
# Both should use MaterialTheme.typography instead
grep -r "style\s*=\s*MaterialTheme" --include="*.kt" src/ # Good examples
```

**Better Check** (AST-like):
```
Flag if Text() uses:
- fontSize = [0-9]+.sp (instead of style = MaterialTheme.typography.*)
- fontWeight = FontWeight.Bold (instead of using typography token)
```

**Bad Example**:
```kotlin
Text(
    text = "Title",
    fontSize = 24.sp,
    fontWeight = FontWeight.Bold
)
```

**Good Example**:
```kotlin
Text(
    text = "Title",
    style = MaterialTheme.typography.headlineSmall
)
```

**Rationale**: Typography tokens include proper line height and letter spacing, and scale correctly.

---

### AP-015: Hardcoded Dimensions

**Severity**: 🟡 Warning | **Confidence**: 75

**Description**: Using magic numbers for spacing and sizing.

**Detection Pattern** (Regex):
```regex
\.(padding|margin|size|offset)\s*\(\s*([0-9]{2}|[1-9][0-9]|[0-9]{3,})\s*\.dp
```

**Detection Script** (grep):
```bash
# Find non-standard dimension values (not 4, 8, 12, 16, 24, 32, 48, 64)
grep -r "\.padding\|\.size\|\.offset" --include="*.kt" src/ | grep -v "\.padding\s*(\s*\(4\|8\|12\|16\|24\|32\|48\|64\)\.dp"
# Manual review needed: check if non-standard values are justified
grep -r "\.\(padding\|size\).*\(17\|13\|21\|31\|37\|42\|51\|57\)\.dp" --include="*.kt" src/
```

**Standard Spacing Grid** (base 4dp):
```
4dp, 8dp, 12dp, 16dp, 24dp, 32dp, 48dp, 64dp
```

**Bad Example**:
```kotlin
Column(modifier = Modifier.padding(17.dp)) { // Why 17?
    Text(modifier = Modifier.padding(bottom = 13.dp)) // Why 13?
}
```

**Good Example**:
```kotlin
Column(modifier = Modifier.padding(16.dp)) { // Standard M3 spacing
    Text(modifier = Modifier.padding(bottom = 12.dp)) // Standard M3 spacing
}
```

**Rationale**: Use 4dp base grid (4, 8, 12, 16, 24, 32, 48, 64).

---

## Side Effects (Critical)

### AP-016: API Calls in Composition

**Severity**: 🔴 Critical | **Confidence**: 95

**Description**: Making network requests or database calls inside composable functions.

**Detection Pattern** (Regex):
```regex
@Composable.*\{[^}]*(api\.|repo\.|database\.)[a-zA-Z]+\s*\([^)]*\)
```

**Detection Script** (grep):
```bash
# Find direct API/repo calls inside @Composable
grep -B2 "api\.\|repo\.\|database\." --include="*.kt" -r src/ | grep -B2 "@Composable"
# Look for common patterns
grep -r "api\.get\|api\.post\|database\.query" --include="*.kt" src/ | grep -v "LaunchedEffect\|viewModel"
```

**Compound Check**:
```bash
# Flag suspicious patterns
grep -r "api\.\|repository\." --include="*.kt" src/ | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  if grep -q "@Composable" "$file"; then
    echo "SUSPICIOUS: $file has API calls (check if in LaunchedEffect)"
  fi
done
```

**Bad Example**:
```kotlin
@Composable
fun EventDetail(eventId: String) {
    val event = api.getEvent(eventId) // Blocks UI! Called on every recomposition!
    Text(event.title)
}
```

**Good Example**:
```kotlin
@Composable
fun EventDetail(eventId: String, viewModel: EventViewModel = viewModel()) {
    val event by viewModel.event.collectAsState()

    LaunchedEffect(eventId) {
        viewModel.loadEvent(eventId)
    }

    event?.let { Text(it.title) }
}
```

**Rationale**: Composables should be pure functions. Side effects belong in `LaunchedEffect`, `DisposableEffect`, or ViewModel.

---

### AP-017: Missing DisposableEffect Cleanup

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Resources that need cleanup (listeners, observers) without proper disposal.

**Detection Pattern** (Regex):
```regex
LaunchedEffect\s*\([^)]*\)\s*\{[^}]*(startUpdates|registerListener|addEventListener|subscribe)[^}]*\}(?!.*DisposableEffect|onDispose)
```

**Detection Script** (grep):
```bash
# Find LaunchedEffect with resource setup but no cleanup
grep -r "LaunchedEffect" --include="*.kt" src/ -A10 | grep "startUpdates\|registerListener\|addEventListener\|subscribe"
# Check if corresponding onDispose/DisposableEffect exists
grep -r "startUpdates\|registerListener\|addEventListener" --include="*.kt" src/ | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  linenum=$(echo "$line" | cut -d: -f2)
  if ! grep -q -A20 "startUpdates\|registerListener" "$file" | grep -q "onDispose\|DisposableEffect"; then
    echo "WARN: Missing cleanup in $file:$linenum"
  fi
done
```

**Pattern Check**:
```
Flag if found:
- startUpdates/registerListener/addEventListener/subscribe
- Inside LaunchedEffect or DisposableEffect
- NO onDispose {} block following
```

**Bad Example**:
```kotlin
@Composable
fun LocationScreen(locationManager: LocationManager) {
    LaunchedEffect(Unit) {
        locationManager.startUpdates() // Never stopped!
    }
}
```

**Good Example**:
```kotlin
@Composable
fun LocationScreen(locationManager: LocationManager) {
    DisposableEffect(Unit) {
        locationManager.startUpdates()
        onDispose {
            locationManager.stopUpdates()
        }
    }
}
```

**Rationale**: `DisposableEffect` ensures cleanup when composable leaves composition.

---

## Security (Critical)

### AP-018: Hardcoded API Keys

**Severity**: 🔴 Critical | **Confidence**: 95

**Description**: Embedding secrets directly in code.

**Detection Pattern** (Regex):
```regex
(const\s+val|val)\s+(API_KEY|SECRET|PASSWORD|TOKEN|KEY)\s*=\s*"(sk-|api_|secret_|password_|token_|key_)[a-zA-Z0-9]+"
```

**Detection Script** (grep):
```bash
# Find hardcoded secrets (exact patterns)
grep -r "API_KEY\s*=\s*\"" --include="*.kt" src/
grep -r "SECRET\s*=\s*\"" --include="*.kt" src/
grep -r "PASSWORD\s*=\s*\"" --include="*.kt" src/
grep -r "sk-\|api_key\|secret_" --include="*.kt" src/
# Scan all strings with suspicious prefixes
grep -r "\"sk-\|\"api_\|\"secret_\|\"token_" --include="*.kt" src/
```

**Pattern Check**:
```
Flag if found:
- Variable name: API_KEY, SECRET, PASSWORD, TOKEN, APIKEY
- String value with patterns: sk-, api_, secret_, token_, password_
- In .kt files (buildConfig is OK, source code is NOT)
```

**Bad Example**:
```kotlin
object Config {
    const val API_KEY = "sk-abc123..." // Leaked in APK!
}
```

**Good Example**:
```kotlin
// build.gradle.kts
android {
    buildConfigField("String", "API_KEY", "\"${System.getenv("API_KEY")}\"")
}

// Usage
BuildConfig.API_KEY
```

**Rationale**: APKs can be decompiled. Use build config with CI secrets or Android Keystore.

---

### AP-019: Storing Secrets in SharedPreferences

**Severity**: 🔴 Critical | **Confidence**: 90

**Description**: Using plain SharedPreferences for sensitive data.

**Detection Pattern** (Regex):
```regex
sharedPrefs\.edit\(\)\.put(String|Int|Boolean|Long|Float)\s*\(\s*"(token|password|secret|apikey|auth)"
```

**Detection Script** (grep):
```bash
# Find plain SharedPreferences with sensitive keys
grep -r "putString.*token\|putString.*password\|putString.*secret" --include="*.kt" src/
# Find SharedPreferences (not EncryptedSharedPreferences)
grep -r "SharedPreferences\|getSharedPreferences" --include="*.kt" src/ | grep -v "Encrypted"
# Check if using correct encryption
grep -r "EncryptedSharedPreferences" --include="*.kt" src/ # Good examples
```

**Compound Check** (manual):
```bash
grep -r "sharedPrefs\.edit()" --include="*.kt" src/ | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  if ! grep -q "EncryptedSharedPreferences" "$file"; then
    echo "WARN: Plain SharedPreferences in $file - should use EncryptedSharedPreferences"
  fi
done
```

**Bad Example**:
```kotlin
sharedPrefs.edit().putString("auth_token", token).apply()
```

**Good Example**:
```kotlin
// Use EncryptedSharedPreferences
val encryptedPrefs = EncryptedSharedPreferences.create(
    context,
    "secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
encryptedPrefs.edit().putString("auth_token", token).apply()
```

**Rationale**: SharedPreferences are stored in plain XML. Use EncryptedSharedPreferences or DataStore with encryption.

---

## ViewModel Integration (Important)

### AP-020: Creating ViewModel in Composable

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Manually instantiating ViewModels instead of using the `viewModel()` helper.

**Detection Pattern** (Regex):
```regex
val\s+\w*[Vv]iew[Mm]odel\s*=\s*\w+ViewModel\s*\(\s*\)
```

**Detection Script** (grep):
```bash
# Find manual ViewModel instantiation
grep -r "val\s\+\w*[Vv]iew[Mm]odel\s*=\s*\w*ViewModel()" --include="*.kt" src/
# Should use viewModel() instead
grep -r "viewModel()" --include="*.kt" src/ # Good examples
# Check inside @Composable context
grep -B5 "ViewModel()" --include="*.kt" -r src/ | grep -B5 "@Composable"
```

**Better Detection**:
```bash
grep -r "ViewModel()" --include="*.kt" src/ | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  if grep -q "ViewModel()" "$file" && ! grep -q "viewModel()" "$file"; then
    echo "WARN: Manual ViewModel instantiation in $file"
  fi
done
```

**Bad Example**:
```kotlin
@Composable
fun EventScreen() {
    val viewModel = EventViewModel() // New instance every recomposition!
    // ...
}
```

**Good Example**:
```kotlin
@Composable
fun EventScreen(viewModel: EventViewModel = viewModel()) {
    // Same instance preserved across recompositions
}
```

**Rationale**: The `viewModel()` function provides proper lifecycle management and process death survival.

---

## Scroll & Derived State (Important)

### AP-021: Missing derivedStateOf

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: Reading scroll state directly in composition without `derivedStateOf` causes 60+ recompositions per second during scrolling.

**Detection Pattern** (Regex):
```regex
val\s+\w+\s*=\s*(scrollState\.value|lazyListState\.firstVisibleItemIndex|scrollState\.firstVisibleItemScrollOffset)(?!.*derivedStateOf)
```

**Detection Script** (grep):
```bash
# Find scroll state read directly without derivedStateOf
grep -r "scrollState\.value\|lazyListState\.firstVisibleItemIndex" --include="*.kt" src/ | grep -v "derivedStateOf"
# Check for scroll offset reads
grep -r "firstVisibleItemScrollOffset" --include="*.kt" src/ | grep -v "derivedStateOf"
```

**Compound Check**:
```bash
# Flag scroll reads outside derivedStateOf blocks
grep -B3 "scrollState\.value\|firstVisibleItemIndex" --include="*.kt" -r src/ | grep -v "derivedStateOf"
# Manual review: ensure these are inside derivedStateOf { }
```

**Bad Example**:
```kotlin
@Composable
fun ScrollHeader(scrollState: ScrollState) {
    val isAtTop = scrollState.value == 0 // Recomposes 60+ times/sec during scroll!

    if (isAtTop) {
        ExpandedHeader()
    } else {
        CollapsedHeader()
    }
}
```

**Good Example**:
```kotlin
@Composable
fun ScrollHeader(scrollState: ScrollState) {
    val isAtTop by remember {
        derivedStateOf { scrollState.value == 0 }
    }

    if (isAtTop) {
        ExpandedHeader()
    } else {
        CollapsedHeader()
    }
}
```

**Rationale**: `derivedStateOf` only triggers recomposition when the derived boolean changes, not on every scroll frame. This reduces 60+ recompositions to just 2 (when crossing the threshold).

---

### AP-022: Nested Scrolling Conflicts

**Severity**: 🔴 Critical | **Confidence**: 95

**Description**: Placing `LazyColumn` or `LazyRow` inside a scrollable `Column` causes crashes or infinite height calculation errors.

**Detection Pattern** (Regex):
```regex
Column\s*\([^)]*verticalScroll\s*\([^)]*\)[^)]*\)\s*\{[^}]*(LazyColumn|LazyRow|LazyVerticalGrid|LazyHorizontalGrid)
```

**Detection Script** (grep):
```bash
# Find Column with verticalScroll containing lazy composables
grep -r "Column.*verticalScroll" --include="*.kt" -r src/ -A20 | grep "LazyColumn\|LazyRow\|LazyVerticalGrid"
# Find Row with horizontalScroll containing lazy composables
grep -r "Row.*horizontalScroll" --include="*.kt" -r src/ -A20 | grep "LazyColumn\|LazyRow"
```

**Compound Check**:
```bash
# List files with both verticalScroll and LazyColumn
for file in $(grep -r "verticalScroll" --include="*.kt" -l src/); do
  if grep -q "LazyColumn\|LazyRow" "$file"; then
    echo "WARN: Potential nested scroll conflict in $file"
  fi
done
```

**Bad Example**:
```kotlin
@Composable
fun BrokenScreen(items: List<Item>) {
    Column(
        modifier = Modifier.verticalScroll(rememberScrollState()) // Outer scroll
    ) {
        Text("Header")
        LazyColumn { // Crash! Infinite height measurement
            items(items) { item ->
                ItemRow(item)
            }
        }
    }
}
```

**Good Example**:
```kotlin
@Composable
fun FixedScreen(items: List<Item>) {
    LazyColumn { // Single lazy container handles all scrolling
        item {
            Text("Header")
        }
        items(items) { item ->
            ItemRow(item)
        }
    }
}

// OR use nestedScroll connection for complex cases:
@Composable
fun NestedScrollScreen(items: List<Item>) {
    val nestedScrollConnection = remember {
        object : NestedScrollConnection {
            // Handle nested scroll properly
        }
    }
    Column(modifier = Modifier.nestedScroll(nestedScrollConnection)) {
        // ...
    }
}
```

**Rationale**: Lazy composables need unbounded height to calculate visible items. Placing them in a scrollable container that expects finite height causes measurement failures. Use a single `LazyColumn` with `item {}` blocks for headers/footers.

---

## Focus Management (Important)

### AP-023: Un-remembered FocusRequester

**Severity**: 🟠 Important | **Confidence**: 90

**Description**: Creating `FocusRequester` without `remember` causes focus loss, keyboard glitches, and lost ripple states on every recomposition.

**Detection Pattern** (Regex):
```regex
val\s+\w+\s*=\s*FocusRequester\s*\(\s*\)(?!.*remember)
```

**Detection Script** (grep):
```bash
# Find FocusRequester without remember
grep -r "val\s\+\w\+\s*=\s*FocusRequester()" --include="*.kt" src/ | grep -v "remember"
# Check for remember wrapping
grep -B2 "FocusRequester()" --include="*.kt" -r src/ | grep -v "remember\s*{"
```

**Compound Check**:
```bash
# Flag FocusRequester not in remember block
awk '/FocusRequester\(\)/{if(!/remember/) print FILENAME":"NR":"$0}' $(find src -name "*.kt")
```

**Bad Example**:
```kotlin
@Composable
fun SearchField(onSearch: (String) -> Unit) {
    var text by remember { mutableStateOf("") }
    val focusRequester = FocusRequester() // New instance every recomposition!

    TextField(
        value = text,
        onValueChange = { text = it },
        modifier = Modifier.focusRequester(focusRequester)
    )

    LaunchedEffect(Unit) {
        focusRequester.requestFocus() // May fail or cause glitches
    }
}
```

**Good Example**:
```kotlin
@Composable
fun SearchField(onSearch: (String) -> Unit) {
    var text by remember { mutableStateOf("") }
    val focusRequester = remember { FocusRequester() }

    TextField(
        value = text,
        onValueChange = { text = it },
        modifier = Modifier.focusRequester(focusRequester)
    )

    LaunchedEffect(Unit) {
        focusRequester.requestFocus()
    }
}
```

**Rationale**: `FocusRequester` maintains focus state internally. Creating a new instance on every recomposition breaks the connection between the composable and focus system.

---

## Effect Keys (Important)

### AP-024: Missing LaunchedEffect Keys

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Using `LaunchedEffect(true)` or `LaunchedEffect(Unit)` when the effect depends on dynamic values that should restart the effect when they change.

**Detection Pattern** (Regex):
```regex
LaunchedEffect\s*\(\s*(true|Unit)\s*\)\s*\{[^}]*(userId|eventId|itemId|id|key|param)[^}]*\}
```

**Detection Script** (grep):
```bash
# Find LaunchedEffect with true/Unit key
grep -r "LaunchedEffect\s*(true)\|LaunchedEffect\s*(Unit)" --include="*.kt" src/
# Check if they reference dynamic IDs
grep -A10 "LaunchedEffect(true)\|LaunchedEffect(Unit)" --include="*.kt" -r src/ | grep "Id\|key\|param"
```

**Compound Check**:
```bash
# Flag LaunchedEffect(true/Unit) with ID references
grep -A10 "LaunchedEffect(true)\|LaunchedEffect(Unit)" --include="*.kt" -r src/ | while read line; do
  if echo "$line" | grep -q "Id\|\.id\|key"; then
    echo "WARN: LaunchedEffect may need dynamic key: $line"
  fi
done
```

**Bad Example**:
```kotlin
@Composable
fun UserProfile(userId: String) {
    var user by remember { mutableStateOf<User?>(null) }

    LaunchedEffect(true) { // Never restarts when userId changes!
        user = api.getUser(userId)
    }

    user?.let { ProfileContent(it) }
}
```

**Good Example**:
```kotlin
@Composable
fun UserProfile(userId: String) {
    var user by remember { mutableStateOf<User?>(null) }

    LaunchedEffect(userId) { // Restarts when userId changes
        user = api.getUser(userId)
    }

    user?.let { ProfileContent(it) }
}
```

**Rationale**: `LaunchedEffect(true)` or `LaunchedEffect(Unit)` only runs once when the composable enters composition. If the effect depends on parameters that can change (like IDs), those parameters must be keys so the effect restarts.

---

## Text Input Performance (Important)

### AP-025: Keystroke Hoisting

**Severity**: 🟠 Important | **Confidence**: 75

**Description**: Hoisting text state to parent composables causes the entire form to recompose on every keystroke, leading to visible lag in complex forms.

**Detection Pattern** (Regex):
```regex
onValueChange\s*=\s*\{\s*\w+\s*->\s*on\w+\(\w+\)\s*\}
```

**Detection Script** (grep):
```bash
# Find TextField with callback that updates parent
grep -r "onValueChange\s*=\s*{.*->.*on[A-Z]" --include="*.kt" src/
# Check for mutableStateOf in parent passed to children
grep -B10 "TextField\|OutlinedTextField" --include="*.kt" -r src/ | grep "by\s\+mutableStateOf.*\"\""
```

**Compound Check**:
```bash
# Flag forms where text state is hoisted
grep -r "var\s\+\w*[Tt]ext\s\+by" --include="*.kt" src/ | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  # Check if this state is passed to child TextField
  if grep -q "value\s*=\s*\w*[Tt]ext" "$file"; then
    echo "CHECK: Potential keystroke hoisting in $file"
  fi
done
```

**Bad Example**:
```kotlin
@Composable
fun ParentForm() {
    var name by remember { mutableStateOf("") }
    var email by remember { mutableStateOf("") }
    var address by remember { mutableStateOf("") }

    Column {
        // Every keystroke recomposes ALL of ParentForm!
        NameField(name) { name = it }
        EmailField(email) { email = it }
        AddressField(address) { address = it }
        SubmitButton(name, email, address)
    }
}

@Composable
fun NameField(value: String, onValueChange: (String) -> Unit) {
    TextField(value = value, onValueChange = onValueChange)
}
```

**Good Example**:
```kotlin
@Composable
fun ParentForm(onSubmit: (FormData) -> Unit) {
    // State stays local to child until submit
    val formState = remember { FormState() }

    Column {
        NameField(onNameReady = { formState.name = it })
        EmailField(onEmailReady = { formState.email = it })
        AddressField(onAddressReady = { formState.address = it })
        SubmitButton { onSubmit(formState.toData()) }
    }
}

@Composable
fun NameField(onNameReady: (String) -> Unit) {
    var text by remember { mutableStateOf("") }

    TextField(
        value = text,
        onValueChange = { text = it },
        modifier = Modifier.onFocusChanged {
            if (!it.isFocused) onNameReady(text) // Hoist only on blur
        }
    )
}

// OR use TextFieldValue for more control:
@Composable
fun ControlledNameField() {
    var textFieldValue by remember { mutableStateOf(TextFieldValue()) }
    TextField(value = textFieldValue, onValueChange = { textFieldValue = it })
}
```

**Rationale**: When parent holds text state, every keystroke triggers `onValueChange` -> parent recomposition -> all children recompose. Keep text state local to the TextField, hoist only on focus loss or explicit submit.

---

## Image Loading & Memory (Important)

### AP-026: Missing Image Sizing

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Loading full-resolution images without `contentScale` or size constraints causes excessive memory usage and poor performance. A 4K image loaded into a 48dp avatar wastes 99% of memory.

**Detection Pattern** (Regex):
```regex
AsyncImage\s*\([^)]*model\s*=[^)]*\)(?![^)]*contentScale)
```

**Detection Script** (grep):
```bash
# Find AsyncImage without contentScale
grep -r "AsyncImage\s*(" --include="*.kt" src/ | grep -v "contentScale"
# Find Image without size constraints
grep -r "Image\s*(" --include="*.kt" src/ -A5 | grep -v "\.size\|\.width\|\.height\|\.aspectRatio"
```

**Compound Check**:
```bash
# Flag AsyncImage calls missing contentScale
for file in $(grep -r "AsyncImage" --include="*.kt" -l src/); do
  if grep -q "AsyncImage" "$file" && ! grep -q "contentScale\s*=" "$file"; then
    echo "WARN: AsyncImage without contentScale in $file"
  fi
done
```

**Bad Example**:
```kotlin
@Composable
fun UserAvatar(imageUrl: String) {
    AsyncImage(
        model = imageUrl, // Loads full 4K image for 48dp avatar!
        contentDescription = "Avatar",
        modifier = Modifier.size(48.dp)
    )
}

@Composable
fun EventBanner(imageUrl: String) {
    AsyncImage(
        model = imageUrl,
        contentDescription = "Event banner"
        // No size constraints - could be infinite!
    )
}
```

**Good Example**:
```kotlin
@Composable
fun UserAvatar(imageUrl: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(imageUrl)
            .size(Size(144, 144)) // Request 3x for density (48dp * 3)
            .build(),
        contentDescription = "Avatar",
        contentScale = ContentScale.Crop,
        modifier = Modifier
            .size(48.dp)
            .clip(CircleShape)
    )
}

@Composable
fun EventBanner(imageUrl: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(imageUrl)
            .size(1080, 400) // Match expected display size
            .build(),
        contentDescription = "Event banner",
        contentScale = ContentScale.Crop,
        modifier = Modifier
            .fillMaxWidth()
            .height(200.dp)
    )
}
```

**Rationale**: Coil's `size()` parameter in `ImageRequest` tells the decoder to downsample the image before loading it into memory. Combined with `contentScale`, this prevents loading multi-megabyte images for thumbnail displays.

---

### AP-027: No Coil/Glide Cache Policy

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: Missing `diskCachePolicy` or `memoryCachePolicy` causes repeated network requests and redundant decoding for the same images.

**Detection Pattern** (Regex):
```regex
ImageRequest\.Builder\([^)]*\)[^}]*\.build\(\)(?![^}]*(diskCachePolicy|memoryCachePolicy))
```

**Detection Script** (grep):
```bash
# Find ImageRequest without cache policy
grep -r "ImageRequest.Builder" --include="*.kt" src/ -A10 | grep -v "diskCachePolicy\|memoryCachePolicy"
# Find AsyncImage with simple string model (no ImageRequest)
grep -r "AsyncImage\s*(\s*model\s*=\s*[\"']" --include="*.kt" src/
grep -r "AsyncImage\s*(\s*model\s*=\s*\w\+\s*," --include="*.kt" src/ | grep -v "ImageRequest"
```

**Compound Check**:
```bash
# Flag AsyncImage using raw URLs without ImageRequest
awk '/@Composable/,/^fun\s+\w+\s*\(/{if(/AsyncImage.*model\s*=\s*[a-zA-Z]+[^I]/ && !/ImageRequest/) print FILENAME":"NR":"$0}' $(find src -name "*.kt")
```

**Bad Example**:
```kotlin
@Composable
fun EventImage(url: String) {
    AsyncImage(
        model = url, // Uses default cache - may not be optimal
        contentDescription = "Event"
    )
}

@Composable
fun ProfileImage(url: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(url)
            .build(), // Missing cache policies
        contentDescription = "Profile"
    )
}
```

**Good Example**:
```kotlin
@Composable
fun EventImage(url: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(url)
            .diskCachePolicy(CachePolicy.ENABLED) // Cache to disk
            .memoryCachePolicy(CachePolicy.ENABLED) // Cache in memory
            .crossfade(true)
            .build(),
        contentDescription = "Event"
    )
}

// For frequently changing images (avatars that update):
@Composable
fun ProfileImage(url: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(url)
            .diskCachePolicy(CachePolicy.ENABLED)
            .memoryCachePolicy(CachePolicy.ENABLED)
            .diskCacheKey(url) // Use URL as cache key
            .memoryCacheKey(url)
            .build(),
        contentDescription = "Profile"
    )
}

// For images that should always be fresh:
@Composable
fun LiveFeedImage(url: String) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(url)
            .diskCachePolicy(CachePolicy.DISABLED) // No disk cache
            .memoryCachePolicy(CachePolicy.WRITE_ONLY) // Cache after load
            .build(),
        contentDescription = "Live feed"
    )
}
```

**Rationale**: Explicit cache policies ensure consistent behavior across app restarts and configuration changes. Default caching may not suit all use cases (e.g., user avatars that update, live feeds, static assets).

---

### AP-028: Bitmap Decode Without Subsampling

**Severity**: 🔴 Critical | **Confidence**: 90

**Description**: Using `BitmapFactory.decodeStream()` or `decodeFile()` without `inSampleSize` loads full-resolution images into memory, causing OOM crashes on large images.

**Detection Pattern** (Regex):
```regex
BitmapFactory\.decode(Stream|File|ByteArray|Resource)\s*\([^)]*\)(?![^;]*(inSampleSize|Options))
```

**Detection Script** (grep):
```bash
# Find BitmapFactory decode calls
grep -r "BitmapFactory\.decode" --include="*.kt" src/
# Check if Options with inSampleSize is used
grep -r "BitmapFactory\.decode" --include="*.kt" src/ -B5 -A5 | grep -v "inSampleSize\|BitmapFactory.Options"
```

**Compound Check**:
```bash
# Flag decode calls without Options parameter
grep -r "BitmapFactory\.decode" --include="*.kt" src/ | while read line; do
  if ! echo "$line" | grep -q "Options"; then
    echo "CRITICAL: Unsampled bitmap decode - $line"
  fi
done
```

**Bad Example**:
```kotlin
// Decodes full 12MP photo into memory - likely OOM crash!
fun loadImage(path: String): Bitmap? {
    return BitmapFactory.decodeFile(path)
}

// Same problem with streams
fun loadFromStream(stream: InputStream): Bitmap? {
    return BitmapFactory.decodeStream(stream)
}

// Resource loading without sampling
fun loadDrawable(context: Context, resId: Int): Bitmap? {
    return BitmapFactory.decodeResource(context.resources, resId)
}
```

**Good Example**:
```kotlin
fun loadImage(path: String, reqWidth: Int, reqHeight: Int): Bitmap? {
    // First decode bounds only (no memory allocation)
    val options = BitmapFactory.Options().apply {
        inJustDecodeBounds = true
    }
    BitmapFactory.decodeFile(path, options)

    // Calculate sample size
    options.apply {
        inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight)
        inJustDecodeBounds = false
    }

    // Decode with subsampling
    return BitmapFactory.decodeFile(path, options)
}

private fun calculateInSampleSize(
    options: BitmapFactory.Options,
    reqWidth: Int,
    reqHeight: Int
): Int {
    val (height, width) = options.outHeight to options.outWidth
    var inSampleSize = 1

    if (height > reqHeight || width > reqWidth) {
        val halfHeight = height / 2
        val halfWidth = width / 2

        while (halfHeight / inSampleSize >= reqHeight &&
               halfWidth / inSampleSize >= reqWidth) {
            inSampleSize *= 2
        }
    }
    return inSampleSize
}

// Better approach: Use Coil for all image loading
@Composable
fun SafeImage(path: String, modifier: Modifier = Modifier) {
    AsyncImage(
        model = ImageRequest.Builder(LocalContext.current)
            .data(File(path))
            .size(Size.ORIGINAL) // Let Coil handle sizing
            .build(),
        contentDescription = null,
        modifier = modifier
    )
}
```

**Rationale**: A 12MP photo (4000x3000) uses 48MB of memory at ARGB_8888. Setting `inSampleSize = 4` reduces this to 3MB while still providing a 1000x750 image. Always use Coil/Glide in Compose instead of manual BitmapFactory decoding.

---

## Accessibility Traversal (Important)

### AP-029: Missing Traversal Order

**Severity**: 🟠 Important | **Confidence**: 85

**Description**: Interactive elements in wrong focus order for TalkBack users. By default, Compose traverses elements in source order (top-to-bottom, left-to-right), which may not match the visual or logical flow of the UI.

**Detection Pattern** (Regex):
```regex
(Row|Column|Box)\s*\([^)]*\)\s*\{[^}]*(Button|IconButton|clickable)[^}]*(Button|IconButton|clickable)
```

**Detection Script** (grep):
```bash
# Find layouts with multiple interactive elements
grep -r "Row\|Column\|Box" --include="*.kt" src/ -A20 | grep -B10 "Button\|IconButton\|clickable" | grep -B10 "Button\|IconButton\|clickable"
# Check if semantics { traversalIndex } is present
grep -r "traversalIndex" --include="*.kt" src/
# Manual review needed for complex layouts
```

**Compound Check**:
```bash
# Find files with multiple clickables but no traversal control
for file in $(grep -r "clickable\|Button" --include="*.kt" -l src/); do
  count=$(grep -c "clickable\|Button" "$file")
  if [ "$count" -gt 3 ] && ! grep -q "traversalIndex\|isTraversalGroup" "$file"; then
    echo "CHECK: $file has $count interactive elements without traversal order"
  fi
done
```

**Bad Example**:
```kotlin
@Composable
fun EventCard(event: Event, onJoin: () -> Unit, onShare: () -> Unit) {
    Row(modifier = Modifier.fillMaxWidth()) {
        // Visual order: Image, Title, Share button, Join button
        // TalkBack order: Same as source (may not match visual layout)

        AsyncImage(
            model = event.imageUrl,
            contentDescription = "Event image"
        )

        Column(modifier = Modifier.weight(1f)) {
            Text(event.title)
            Text(event.date)
        }

        // Share button appears BEFORE Join visually, but users expect
        // to focus on the primary action (Join) first
        IconButton(onClick = onShare) {
            Icon(Icons.Default.Share, "Share")
        }

        Button(onClick = onJoin) {
            Text("Join")
        }
    }
}
```

**Good Example**:
```kotlin
@Composable
fun EventCard(event: Event, onJoin: () -> Unit, onShare: () -> Unit) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .semantics { isTraversalGroup = true } // Group for logical traversal
    ) {
        AsyncImage(
            model = event.imageUrl,
            contentDescription = "Event image for ${event.title}",
            modifier = Modifier.semantics { traversalIndex = 1f }
        )

        Column(
            modifier = Modifier
                .weight(1f)
                .semantics { traversalIndex = 2f }
        ) {
            Text(event.title)
            Text(event.date)
        }

        // Primary action gets lower traversalIndex (focused first)
        Button(
            onClick = onJoin,
            modifier = Modifier.semantics { traversalIndex = 3f }
        ) {
            Text("Join")
        }

        // Secondary action gets higher traversalIndex (focused second)
        IconButton(
            onClick = onShare,
            modifier = Modifier.semantics { traversalIndex = 4f }
        ) {
            Icon(Icons.Default.Share, "Share event")
        }
    }
}
```

**Alternative - Merge Semantics for Simple Cases**:
```kotlin
@Composable
fun EventCard(event: Event, onJoin: () -> Unit) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable(onClick = onJoin) // Make whole card clickable
            .semantics(mergeDescendants = true) { // Single TalkBack focus
                contentDescription = "${event.title}, ${event.date}. Double tap to join."
            }
    ) {
        AsyncImage(model = event.imageUrl, contentDescription = null) // Decorative
        Column {
            Text(event.title)
            Text(event.date)
        }
    }
}
```

**Rationale**: TalkBack users navigate sequentially through focusable elements. If the visual layout doesn't match the logical focus order, users may struggle to find primary actions or understand the content hierarchy. Use `isTraversalGroup` to define navigation groups and `traversalIndex` to specify explicit order within groups. For simple cards, `mergeDescendants = true` creates a single focus target.

---

### AP-030: Missing Custom Accessibility Actions

**Severity**: 🟠 Important | **Confidence**: 80

**Description**: Complex gestures (swipe-to-dismiss, long press, drag) without accessible alternatives. TalkBack users cannot perform multi-touch or gesture-based interactions.

**Detection Pattern** (Regex):
```regex
\.(pointerInput|detectDragGestures|detectTapGestures|swipeable|draggable|longPress)
```

**Detection Script** (grep):
```bash
# Find gesture-based interactions
grep -r "pointerInput\|detectDragGestures\|detectTapGestures\|swipeable\|draggable" --include="*.kt" src/
# Check if customActions is defined
grep -r "customActions" --include="*.kt" src/
# Flag files with gestures but no custom actions
for file in $(grep -r "pointerInput\|swipeable\|draggable" --include="*.kt" -l src/); do
  if ! grep -q "customActions" "$file"; then
    echo "WARN: $file has gestures without customActions"
  fi
done
```

**Compound Check**:
```bash
# Find SwipeToDismiss without accessible alternative
grep -r "SwipeToDismiss\|Dismissible\|rememberDismissState" --include="*.kt" src/ | while read line; do
  file=$(echo "$line" | cut -d: -f1)
  if ! grep -q "customActions" "$file"; then
    echo "ACCESSIBILITY: $file uses swipe-to-dismiss without customActions"
  fi
done
```

**Bad Example**:
```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun NotificationItem(
    notification: Notification,
    onDismiss: () -> Unit,
    onMarkRead: () -> Unit
) {
    val dismissState = rememberSwipeToDismissBoxState(
        confirmValueChange = { value ->
            if (value == SwipeToDismissBoxValue.EndToStart) {
                onDismiss()
                true
            } else false
        }
    )

    SwipeToDismissBox(
        state = dismissState,
        backgroundContent = {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .background(Color.Red),
                contentAlignment = Alignment.CenterEnd
            ) {
                Icon(Icons.Default.Delete, "Delete")
            }
        }
    ) {
        // TalkBack users cannot swipe to dismiss!
        NotificationContent(notification)
    }
}

// Long press without accessible alternative
@Composable
fun MessageBubble(message: Message, onReact: () -> Unit) {
    Box(
        modifier = Modifier
            .pointerInput(Unit) {
                detectTapGestures(
                    onLongPress = { onReact() } // TalkBack can't long press!
                )
            }
    ) {
        Text(message.content)
    }
}
```

**Good Example**:
```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun NotificationItem(
    notification: Notification,
    onDismiss: () -> Unit,
    onMarkRead: () -> Unit
) {
    val dismissState = rememberSwipeToDismissBoxState(
        confirmValueChange = { value ->
            if (value == SwipeToDismissBoxValue.EndToStart) {
                onDismiss()
                true
            } else false
        }
    )

    SwipeToDismissBox(
        state = dismissState,
        modifier = Modifier.semantics {
            // Provide accessible alternatives for swipe actions
            customActions = listOf(
                CustomAccessibilityAction("Dismiss notification") {
                    onDismiss()
                    true
                },
                CustomAccessibilityAction("Mark as read") {
                    onMarkRead()
                    true
                }
            )
        },
        backgroundContent = {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .background(MaterialTheme.colorScheme.error),
                contentAlignment = Alignment.CenterEnd
            ) {
                Icon(
                    Icons.Default.Delete,
                    contentDescription = null // Action described in customActions
                )
            }
        }
    ) {
        NotificationContent(notification)
    }
}

// Long press with accessible alternative
@Composable
fun MessageBubble(message: Message, onReact: () -> Unit) {
    Box(
        modifier = Modifier
            .pointerInput(Unit) {
                detectTapGestures(
                    onLongPress = { onReact() }
                )
            }
            .semantics {
                customActions = listOf(
                    CustomAccessibilityAction("Add reaction") {
                        onReact()
                        true
                    }
                )
            }
    ) {
        Text(message.content)
    }
}

// Drag-to-reorder with accessible alternatives
@Composable
fun ReorderableItem(
    item: Item,
    onMoveUp: () -> Unit,
    onMoveDown: () -> Unit
) {
    Row(
        modifier = Modifier
            .draggable(
                state = rememberDraggableState { /* handle drag */ },
                orientation = Orientation.Vertical
            )
            .semantics {
                customActions = listOf(
                    CustomAccessibilityAction("Move up") {
                        onMoveUp()
                        true
                    },
                    CustomAccessibilityAction("Move down") {
                        onMoveDown()
                        true
                    }
                )
            }
    ) {
        Icon(Icons.Default.DragHandle, contentDescription = "Drag to reorder")
        Text(item.title)
    }
}
```

**Rationale**: TalkBack users interact through single taps, swipes (for navigation, not gestures), and the actions menu. Multi-touch gestures like swipe-to-dismiss, long press, and drag-and-drop are inaccessible without alternatives. The `customActions` semantic property adds items to TalkBack's local context menu (accessed by swiping up then right), providing equivalent functionality through accessible means.

---

## Summary Table

| ID | Anti-Pattern | Severity | Category |
|----|--------------|----------|----------|
| AP-001 | Passing MutableState | 🔴 Critical | State |
| AP-002 | Reading Scroll in Composition | 🔴 Critical | State |
| AP-003 | Creating State in Loops | 🔴 Critical | State |
| AP-004 | Missing rememberSaveable | 🟠 Important | State |
| AP-005 | Creating Modifiers in Composition | 🟠 Important | Performance |
| AP-006 | Unstable Collections | 🟠 Important | Performance |
| AP-007 | Heavy Computation Without remember | 🟠 Important | Performance |
| AP-008 | Column Instead of LazyColumn | 🟡 Warning | Performance |
| AP-009 | Missing Keys in LazyColumn | 🟠 Important | Performance |
| AP-010 | Missing contentDescription | 🔴 Critical | Accessibility |
| AP-011 | Small Touch Targets | 🔴 Critical | Accessibility |
| AP-012 | Wrong Semantics Role | 🟠 Important | Accessibility |
| AP-013 | Hardcoded Colors | 🟠 Important | Styling |
| AP-014 | Hardcoded Font Sizes | 🟠 Important | Styling |
| AP-015 | Hardcoded Dimensions | 🟡 Warning | Styling |
| AP-016 | API Calls in Composition | 🔴 Critical | Side Effects |
| AP-017 | Missing DisposableEffect Cleanup | 🟠 Important | Side Effects |
| AP-018 | Hardcoded API Keys | 🔴 Critical | Security |
| AP-019 | Secrets in SharedPreferences | 🔴 Critical | Security |
| AP-020 | Creating ViewModel Manually | 🟠 Important | Architecture |
| AP-021 | Missing derivedStateOf | 🟠 Important | Scroll & Derived State |
| AP-022 | Nested Scrolling Conflicts | 🔴 Critical | Scroll & Derived State |
| AP-023 | Un-remembered FocusRequester | 🟠 Important | Focus Management |
| AP-024 | Missing LaunchedEffect Keys | 🟠 Important | Effect Keys |
| AP-025 | Keystroke Hoisting | 🟠 Important | Text Input Performance |
| AP-026 | Missing Image Sizing | 🟠 Important | Image Loading & Memory |
| AP-027 | No Coil/Glide Cache Policy | 🟠 Important | Image Loading & Memory |
| AP-028 | Bitmap Decode Without Subsampling | 🔴 Critical | Image Loading & Memory |
| AP-029 | Missing Traversal Order | 🟠 Important | Accessibility Traversal |
| AP-030 | Missing Custom Accessibility Actions | 🟠 Important | Accessibility Traversal |

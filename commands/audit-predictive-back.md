---
name: audit-predictive-back
description: Audit Android code for Predictive Back gesture API compliance and gesture animations
args:
  - name: code
    type: string
    description: Kotlin/Compose code to audit for Predictive Back support
    required: true
  - name: focus
    type: string
    enum: [manifest, callbacks, compose, animations, navigation, fragment, all]
    description: Specific area to focus on (default: all)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Android Predictive Back Gesture Audit

Comprehensive audit of Android code for Predictive Back gesture API support and animation compliance. Covers Android 13+ with special attention to Android 15+ system animations and Jetpack Compose integration.

## Android Version Context

- **Android 13+**: OnBackInvokedCallback API introduced
- **Android 14+**: Developer preview of predictive back animations
- **Android 15+**: System animations (back-to-home, cross-activity, cross-task) enabled by default for opted-in apps
- **AndroidX Activity 1.6.0+**: BackHandler and PredictiveBackHandler composables

## Audit Checklist

### Manifest Configuration (Critical Priority)
{{#if (or (eq focus "manifest") (eq focus "all") (not focus))}}
- [ ] **PB-001**: `android:enableOnBackInvokedCallback="true"` set in `<application>` tag
- [ ] **PB-002**: Verify no conflicting `onBackPressed()` legacy methods (deprecated since Android 11)
- [ ] **PB-003**: Target SDK >= 33 (Android 13) to use predictive back
- [ ] **PB-004**: Compile SDK >= 34 recommended (Android 14)
- [ ] **PB-005**: AndroidX Activity library >= 1.6.0
- [ ] **PB-006**: AndroidX Navigation >= 2.6.0 for Navigation Component support
- [ ] **PB-007**: No conflicting `@Deprecated onBackPressed()` overrides
{{/if}}

### Callback Registration (Critical Priority)
{{#if (or (eq focus "callbacks") (eq focus "all") (not focus))}}
- [ ] **PB-008**: Using `OnBackInvokedCallback` (platform) or `BackHandler` (AndroidX)
- [ ] **PB-009**: Registering callbacks with correct priority (PRIORITY_DEFAULT vs PRIORITY_OVERLAY)
- [ ] **PB-010**: Callback only registered when needed (conditional in Fragment/Dialog lifecycle)
- [ ] **PB-011**: Unregistering callbacks in `onDestroy()` / `onDispose()`
- [ ] **PB-012**: Not creating multiple callbacks for same back action
- [ ] **PB-013**: Using `OnBackInvokedDispatcher` correctly for activity-level back handling
- [ ] **PB-014**: Testing callback stack behavior (last-added enabled callback handles gesture)
{{/if}}

### Jetpack Compose Integration (Critical Priority)
{{#if (or (eq focus "compose") (eq focus "all") (not focus))}}
- [ ] **PB-015**: Using `PredictiveBackHandler` for custom animations
- [ ] **PB-016**: Using `BackHandler` for simple back interception
- [ ] **PB-017**: `BackHandler` placed at correct composition level (dialog/sheet only)
- [ ] **PB-018**: `PredictiveBackHandler` collects `BackEventCompat` via Flow correctly
- [ ] **PB-019**: Accessing progress, touchX, touchY from `BackEventCompat`
- [ ] **PB-020**: Not blocking predictive back for system back animations (cross-activity, back-to-home)
- [ ] **PB-021**: Material3 components (SearchBar, ModalBottomSheet) auto-animate with manifest flag
- [ ] **PB-022**: Custom Composable back handling doesn't interfere with system animations
{{/if}}

### Back Gesture Animations (Important Priority)
{{#if (or (eq focus "animations") (eq focus "all") (not focus))}}
- [ ] **PB-023**: Predictive preview animates destination screen (scale 90% or custom)
- [ ] **PB-024**: Current screen animates out during gesture (fade or translate)
- [ ] **PB-025**: Animation progress tied to gesture progress (Flow collection)
- [ ] **PB-026**: Animation cancels and reverses if user releases mid-gesture
- [ ] **PB-027**: Animation completes smoothly if user completes gesture
- [ ] **PB-028**: No frame drops during animation (use `Modifier.animateContentSize()` or custom)
- [ ] **PB-029**: Respects system animation speed preference (Settings > Accessibility > Animation)
- [ ] **PB-030**: Drawer/Sheet animations use smooth Bezier easing
{{/if}}

### Navigation Component (Important Priority)
{{#if (or (eq focus "navigation") (eq focus "all") (not focus))}}
- [ ] **PB-031**: Navigation Component handles back automatically (no manual `popBackStack()`)
- [ ] **PB-032**: Custom `OnBackInvokedCallback` doesn't conflict with NavHostController
- [ ] **PB-033**: Fragment back stack is managed by NavController (not manual FragmentTransaction)
- [ ] **PB-034**: Dialog Fragments use `NavController` for back handling
- [ ] **PB-035**: Bottom sheet navigation properly registered with predictive back
- [ ] **PB-036**: Testing cross-fragment predictive back animations
{{/if}}

### Fragment Back Handling (Important Priority)
{{#if (or (eq focus "fragment") (eq focus "all") (not focus))}}
- [ ] **PB-037**: Fragments using `BackHandler` in `onViewCreated()` not `onCreate()`
- [ ] **PB-038**: Dialog fragments properly unregister back handlers in `onDismiss()`
- [ ] **PB-039**: Nested fragments register callbacks at correct priority
- [ ] **PB-040**: Fragment back handler respects navigation hierarchy
- [ ] **PB-041**: Testing back gesture with multiple fragment levels
{{/if}}

### Anti-Patterns (High Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **NOT using**: Deprecated `Activity.onBackPressed()` method
- [ ] **NOT blocking**: System animations for cross-activity/back-to-home
- [ ] **NOT mixing**: OnBackInvokedCallback with onBackPressed() in same activity
- [ ] **NOT ignoring**: Priority levels when registering multiple callbacks
- [ ] **NOT blocking**: Back gesture entirely (only customize, don't disable)
- [ ] **NOT creating**: Anonymous OnBackInvokedCallback objects (cause memory leaks)
- [ ] **NOT calling**: `unregisterOnBackInvokedCallback()` without corresponding `registerOnBackInvokedCallback()`
{{/if}}

### Testing & Validation (Important Priority)
{{#if (or (eq focus "all") (not focus))}}
- [ ] **PB-042**: Testing predictive back gesture with physical swipe from edge
- [ ] **PB-043**: Testing gesture cancellation (release mid-gesture, preview reverts)
- [ ] **PB-044**: Testing gesture completion (swipe fully, back executes)
- [ ] **PB-045**: Testing cross-activity predictive back animations
- [ ] **PB-046**: Testing nested dialog/sheet back handling
- [ ] **PB-047**: Testing with Developer Options > Predictive Back Gesture enabled
- [ ] **PB-048**: Verifying no jank or stuttering during animations
- [ ] **PB-049**: Testing on Android 13, 14, and 15+
- [ ] **PB-050**: Testing animation cancellation (reverse animation smooth)
{{/if}}

## Code Examples

### Correct Patterns

#### Manifest Configuration
```xml
<!-- AndroidManifest.xml -->
<application
    android:enableOnBackInvokedCallback="true"
    ...>
    <!-- Predictive back enabled for entire app -->
</application>
```

#### Using BackHandler in Compose
```kotlin
// GOOD: Simple back interception
@Composable
fun MyDialog() {
    var isOpen by remember { mutableStateOf(true) }

    if (isOpen) {
        Dialog(
            onDismissRequest = { isOpen = false }
        ) {
            // BackHandler automatically registered inside Dialog
            BackHandler(enabled = true) {
                isOpen = false
            }

            // Dialog content
            Text("Confirm action?")
            Button(onClick = { isOpen = false }) {
                Text("Close")
            }
        }
    }
}
```

#### Using PredictiveBackHandler for Custom Animations
```kotlin
// GOOD: Custom predictive back animation with progress
@Composable
fun DrawerWithPredictiveBack(
    isOpen: Boolean,
    onClose: () -> Unit
) {
    val scope = rememberCoroutineScope()
    var isAnimating by remember { mutableStateOf(false) }
    var animationProgress by remember { mutableFloatStateOf(0f) }

    // Collect back gesture progress
    PredictiveBackHandler(onBack = {
        scope.launch {
            it.collect { backEvent ->
                // Update animation progress based on gesture
                animationProgress = backEvent.progress
                isAnimating = animationProgress > 0f

                // Animate drawer position based on progress
                when (backEvent.touchX) {
                    // Handle gesture direction
                }
            }
        }
    }) {
        // Execute back when gesture completes
        onClose()
    }

    // Drawer content with animation
    Box(
        modifier = Modifier
            .offset(x = (animationProgress * 100).dp)
            .alpha(1f - (animationProgress * 0.2f))
    ) {
        Drawer()
    }
}
```

#### Activity-Level Back Handling
```kotlin
// GOOD: Proper callback management
class MainActivity : AppCompatActivity() {
    private lateinit var backCallback: OnBackInvokedCallback

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Register back callback
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            backCallback = OnBackInvokedCallback {
                // Handle back - called when gesture completes
                handleBackPressed()
            }
            onBackInvokedDispatcher.registerOnBackInvokedCallback(
                OnBackInvokedDispatcher.PRIORITY_DEFAULT,
                backCallback
            )
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        // Unregister callback
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            onBackInvokedDispatcher.unregisterOnBackInvokedCallback(backCallback)
        }
    }

    private fun handleBackPressed() {
        // Your back logic
    }
}
```

#### Material3 Predictive Back Animations
```kotlin
// GOOD: SearchBar with auto predictive back animation
@Composable
fun SearchWithPredictiveBack() {
    var expanded by remember { mutableStateOf(false) }

    SearchBar(
        query = "",
        onQueryChange = {},
        onSearch = { expanded = false },
        active = expanded,
        onActiveChange = { expanded = it },
        // SearchBar automatically animates with predictive back
        // when android:enableOnBackInvokedCallback="true"
        modifier = Modifier.fillMaxWidth()
    ) {
        // Search results
    }
}
```

### Incorrect Patterns

#### Bad: Manifest Missing Flag
```xml
<!-- BAD: Predictive back disabled -->
<application
    <!-- Missing android:enableOnBackInvokedCallback -->
    ...>
</application>
```

#### Bad: Mixed Back APIs
```kotlin
// BAD: Using both deprecated and new APIs
override fun onBackPressed() {
    // Deprecated, conflicts with predictive back
}

// Also registering callback
OnBackInvokedCallback {
    // This creates conflict
}
```

#### Bad: No Callback Cleanup
```kotlin
// BAD: Memory leak - callback never unregistered
class DialogFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // Register but never unregister
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            requireActivity().onBackInvokedDispatcher.registerOnBackInvokedCallback(
                OnBackInvokedDispatcher.PRIORITY_DEFAULT
            ) {
                dismiss()
            }
        }
        // Missing unregister in onDestroyView()
    }
}
```

#### Bad: Blocking System Back Animations
```kotlin
// BAD: Prevents system back-to-home animation
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // This blocks system animations
        onBackInvokedDispatcher.registerOnBackInvokedCallback(
            OnBackInvokedDispatcher.PRIORITY_DEFAULT
        ) {
            // Prevents back-to-home animation
            System.exit(0)  // WRONG!
        }
    }
}
```

#### Bad: PredictiveBackHandler Without Navigation
```kotlin
// BAD: Not collecting or updating animation state
@Composable
fun BadAnimationHandler() {
    PredictiveBackHandler(onBack = {}) {
        // Not collecting BackEventCompat - gesture progress ignored
        // Animation doesn't respond to gesture
    }
}
```

#### Bad: Fragment Callback Timing
```kotlin
// BAD: Registering in onCreate() instead of onViewCreated()
class MyFragment : Fragment() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Wrong! View not created yet
        BackHandler {
            dismiss()
        }
    }
}
```

## Anti-Patterns to Avoid

### 1. Using Deprecated `onBackPressed()`
**Problem**: Conflicts with predictive back, not called on Android 13+
```kotlin
// NEVER DO THIS
override fun onBackPressed() {
    super.onBackPressed()
}
```

### 2. Blocking Back Gesture Entirely
**Problem**: Prevents system animations (back-to-home, cross-task)
```kotlin
// NEVER DO THIS - blocks ALL back gestures
override fun onBackInvokedDispatcher() {
    throw IllegalStateException("Back disabled!")
}
```

### 3. Anonymous Callback Objects
**Problem**: Can't unregister, causes memory leaks
```kotlin
// NEVER DO THIS
onBackInvokedDispatcher.registerOnBackInvokedCallback(PRIORITY_DEFAULT) {
    dismiss()
}
// Can't unregister - object reference lost
```

### 4. Not Respecting Priority Levels
**Problem**: Callback order unpredictable
```kotlin
// NEVER DO THIS - no priority specified
onBackInvokedDispatcher.registerOnBackInvokedCallback {
    dismiss()
}
```

### 5. Callback in Activity + Dialog Both Handling Back
**Problem**: Creates conflicting back behaviors
```kotlin
// BAD: Both activity and dialog handle back
class MyActivity : AppCompatActivity() {
    init {
        // Activity-level callback
        onBackInvokedDispatcher.registerOnBackInvokedCallback(...) {
            finish()
        }
    }
}

class MyDialog : DialogFragment() {
    override fun onViewCreated(...) {
        // Also dialog-level callback - conflicts!
        BackHandler { dismiss() }
    }
}
```

### 6. Not Unregistering in Fragment
**Problem**: Callback persists after fragment destroyed
```kotlin
// BAD: No cleanup
class MyFragment : Fragment() {
    private lateinit var callback: OnBackInvokedCallback

    override fun onViewCreated(...) {
        callback = OnBackInvokedCallback { dismiss() }
        requireActivity().onBackInvokedDispatcher
            .registerOnBackInvokedCallback(PRIORITY_DEFAULT, callback)
        // Missing unregister in onDestroyView()
    }
}
```

### 7. Ignoring Animation Progress
**Problem**: Gesture doesn't animate predictively
```kotlin
// BAD: Not using BackEventCompat progress
PredictiveBackHandler {
    // Ignoring gesture progress - no animation
    navigateBack()
}
```

## Testing Checklist

### Manual Gesture Testing
- [ ] Swipe from left edge to right - triggers back preview
- [ ] Swipe slowly - smooth animation follows gesture
- [ ] Release mid-gesture - animation reverses smoothly
- [ ] Swipe past threshold - back executes, screen transitions
- [ ] Test on multiple screens (activity, dialog, sheet)

### Cross-Activity Testing
- [ ] Activity A → Activity B
- [ ] Predictive back shows Activity A preview
- [ ] Back-to-home gesture shows home preview
- [ ] Cross-task gesture transitions smoothly

### Fragment/Dialog Testing
- [ ] Open dialog → predictive back shows previous screen
- [ ] Nested dialogs → back dismisses top dialog only
- [ ] Multiple fragments → back behavior correct at each level
- [ ] Dynamically added fragments → back handles correctly

### Accessibility Testing
- [ ] TalkBack announces back gesture
- [ ] Back gesture works with accessibility settings
- [ ] Animation respects reduced motion setting

### Animation Performance
- [ ] No jank during gesture (60fps minimum)
- [ ] No memory leaks during repeated back gestures
- [ ] Animation smooth on low-end devices (API 33)
- [ ] Animation smooth on high-end devices (API 35)

## Dependencies

```gradle
// Required
implementation 'androidx.activity:activity:1.6.0' // BackHandler, PredictiveBackHandler

// Recommended
implementation 'androidx.activity:activity-compose:1.7.0'
implementation 'androidx.navigation:navigation-compose:2.6.0'
implementation 'androidx.compose.material3:material3:1.1.0'
```

## Related Documentation

- [Android Developers: Add support for the predictive back gesture](https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture)
- [Jetpack Compose: About Predictive Back](https://developer.android.com/develop/ui/compose/system/predictive-back)
- [Jetpack Compose: Set up Predictive Back](https://developer.android.com/develop/ui/compose/system/predictive-back-setup)
- [Jetpack Compose: Access progress manually](https://developer.android.com/develop/ui/compose/system/predictive-back-progress)
- [Android Developers: Predictive Back Design](https://developer.android.com/design/ui/mobile/guides/patterns/predictive-back)
- [Android Codelab: Handling Gesture Back Navigation](https://codelabs.developers.google.com/handling-gesture-back-navigation)
- [Jetpack Compose: BackHandler documentation](https://developer.android.com/reference/kotlin/androidx/activity/compose/package-summary)

## Code to Audit

```kotlin
{{code}}
```

## Output Format

Generate a structured audit report:

```markdown
## Predictive Back Audit: [File/Component Name]

### Summary
- **Manifest Flag**: ✅ Enabled / ❌ Missing
- **API Level**: Target SDK X (Android X)
- **Callbacks**: Y registered, Z unregistered
- **Animation Support**: ✅ Full / ⚠️ Partial / ❌ None
- **Navigation Component**: ✅ Yes / ❌ No

### Critical Issues (🔴)
List critical issues blocking predictive back support.
Each with: Line number, description, severity, fix suggestion.

### Important Issues (🟠)
List issues breaking animations or gesture handling.

### Warnings (🟡)
List suboptimal patterns that work but should improve.

### Suggestions (🔵)
List polish opportunities for smooth animations.

### Animation Analysis
- Gesture preview: [Description]
- Animation easing: [Type or missing]
- Progress tracking: ✅ / ❌
- Cancellation handling: ✅ / ⚠️ / ❌

### Testing Recommendations
1. [Most urgent manual test]
2. [Second priority test]
3. [Third priority test]

### Summary Statistics
- Total issues: X
- Critical: X | Important: X | Warning: X | Suggestion: X
- Predictive Back compliance: X%
```

## Example Output

```markdown
## Predictive Back Audit: MainActivity.kt

### Summary
- **Manifest Flag**: ✅ Enabled (android:enableOnBackInvokedCallback="true")
- **API Level**: Target SDK 34 (Android 14)
- **Callbacks**: 1 registered, ❌ 0 unregistered
- **Animation Support**: ⚠️ Partial (no custom animations)
- **Navigation Component**: ✅ Using NavController

### Critical Issues (🔴)

**Line 45**: Callback never unregistered (memory leak)
```kotlin
// BAD
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    onBackInvokedDispatcher.registerOnBackInvokedCallback(
        OnBackInvokedDispatcher.PRIORITY_DEFAULT
    ) {
        finish()
    }
    // Missing unregister in onDestroy()
}

// GOOD
private lateinit var backCallback: OnBackInvokedCallback

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    backCallback = OnBackInvokedCallback { finish() }
    onBackInvokedDispatcher.registerOnBackInvokedCallback(
        OnBackInvokedDispatcher.PRIORITY_DEFAULT,
        backCallback
    )
}

override fun onDestroy() {
    super.onDestroy()
    onBackInvokedDispatcher.unregisterOnBackInvokedCallback(backCallback)
}
```
Severity: 🔴 Critical | Confidence: 95 | Rule: PB-011

### Important Issues (🟠)

**Line 78**: SearchBar not animating with predictive back
```kotlin
// BAD
SearchBar(
    query = query,
    onQueryChange = { },
    // No support for predictive back animations
)

// Note: SearchBar auto-animates IF manifest flag enabled
// Verify in AndroidManifest.xml:
// <application android:enableOnBackInvokedCallback="true">
```
Severity: 🟠 Important | Confidence: 85 | Rule: PB-021

### Warnings (🟡)

**Line 120**: Using implicit priority for back callback
```kotlin
// Could be clearer:
onBackInvokedDispatcher.registerOnBackInvokedCallback(
    backCallback
    // Priority missing - assumes PRIORITY_DEFAULT
)

// Better:
onBackInvokedDispatcher.registerOnBackInvokedCallback(
    OnBackInvokedDispatcher.PRIORITY_DEFAULT,
    backCallback
)
```
Severity: 🟡 Warning | Confidence: 70 | Rule: PB-009

### Animation Analysis
- **Gesture preview**: Not implemented (uses default system animation)
- **Animation easing**: System default (no custom easing)
- **Progress tracking**: ❌ Not using PredictiveBackHandler
- **Cancellation handling**: ✅ System handles automatically

**Recommendation**: For custom drawer animations, use PredictiveBackHandler with BackEventCompat.progress collection.

### Testing Recommendations
1. Test predictive back gesture from left edge on Android 15+ device
2. Add custom animation for drawer using PredictiveBackHandler
3. Verify callback unregistration with memory profiler
4. Test cross-activity predictive back (MainActivity → DetailActivity)

### Summary Statistics
- Total issues: 3
- Critical: 1 | Important: 1 | Warning: 1 | Suggestion: 0
- Predictive Back compliance: 70%
```

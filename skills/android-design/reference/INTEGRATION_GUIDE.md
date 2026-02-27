# Android Design Skill - Integration Guide

Complete guide to using the enhanced reference materials and anti-pattern detection capabilities.

---

## Quick Reference Map

```
/reference/
├── INTEGRATION_GUIDE.md                    (This file - how to use everything)
├── material-design-3.md                    (M3 specs, colors, typography)
├── compose-rules-integration.md            (Static analysis setup + rules)
├── compose-antipatterns.md                 (20 anti-patterns + detection patterns)
├── compiler-metrics-guide.md               (Reading Compose stability reports)
├── now-in-android-patterns.md              (Google reference app architecture)
├── typography-android.md
├── color-theming.md
├── spacing-touch-targets.md
├── motion-android.md
└── compose-patterns.md
```

---

## Usage Workflows

### Scenario 1: Audit Compose Code for Anti-Patterns

**Goal**: Review code against all 20 anti-patterns

**Steps**:

1. **Read antipatterns database** → `/reference/compose-antipatterns.md`
   - Browse all 20 anti-patterns
   - Understand severity + confidence levels
   - Identify which apply to your codebase

2. **Run detection scripts** (from antipatterns file):
   ```bash
   # Example: Find hardcoded colors (AP-013)
   grep -r "color\s*=\s*Color\s*(" --include="*.kt" src/ | grep -v "MaterialTheme.colorScheme"

   # Example: Find unstable collections (AP-006)
   grep -r "val\s\+\w\+:\s\*(List\|Set\|Map)<" --include="*.kt" src/ | grep -v "ImmutableList"
   ```

3. **For each violation found**:
   - Look up the anti-pattern in compose-antipatterns.md
   - Review the "Good Example" section
   - Apply the fix
   - Run tests to verify

4. **Optional: Set up static analysis** (compose-rules-integration.md):
   ```bash
   ./gradlew ktlintCheck detekt
   ```

---

### Scenario 2: Optimize Compose Performance

**Goal**: Improve recomposition performance using compiler metrics

**Steps**:

1. **Generate compiler metrics** → `/reference/compiler-metrics-guide.md`
   ```bash
   ./gradlew clean assembleDebug
   ```

2. **Read the metrics files**:
   - `build/compose_metrics/*_composables.csv` - Identify unskippable composables
   - `build/compose_metrics/*_classes.txt` - Check why classes are unstable

3. **Apply fixes from guide**:
   - Option A: Use `ImmutableList` instead of `List<T>`
   - Option B: Mark data classes `@Immutable`
   - Option C: Add `remember()` for expensive operations

4. **Verify improvements**:
   ```bash
   # Rebuild and check metrics again
   grep "skippable=true" build/compose_metrics/*_composables.csv | wc -l
   ```

---

### Scenario 3: Review Against Material Design 3

**Goal**: Ensure UI follows M3 specifications

**Files**:
- `/reference/material-design-3.md` - Component specs, colors, typography
- `/reference/spacing-touch-targets.md` - 48dp minimum, grid standards
- `/reference/color-theming.md` - Dynamic color + theme tokens
- `/reference/typography-android.md` - 15 typography scales

**Quick Checks**:
```kotlin
// ✓ Use semantic tokens, not hardcoded values
Text(style = MaterialTheme.typography.headlineLarge)  // GOOD

// ✓ 48dp minimum touch targets
IconButton(onClick = { })  // GOOD (auto 48dp)

// ✓ Use theme colors
Surface(color = MaterialTheme.colorScheme.surface)  // GOOD

// ✓ Apply shapes from M3 scale
Card(shape = RoundedCornerShape(MaterialTheme.shapes.medium))  // GOOD
```

---

### Scenario 4: Learn from Production Reference App

**Goal**: Understand how Google implements Material 3 in production

**File**: `/reference/now-in-android-patterns.md`

**Key Sections**:
- Material 3 + Dynamic Color implementation
- Modularization strategy (feature modules)
- Baseline Profiles for AOT compilation (60-80% faster launch)
- Data flow with ViewModel + StateFlow
- Reusable composable component pattern
- Macro benchmarking for performance validation

**Copy-Paste Ready Code**:
- Theme setup with dynamic color fallback
- ViewModel architecture pattern
- Component with `Modifier` as first parameter
- Test pattern for screen-level validation

---

### Scenario 5: Set Up Static Analysis Pipeline

**Goal**: Automate anti-pattern detection in CI/CD

**File**: `/reference/compose-rules-integration.md`

**Steps**:

1. **Install Compose Rules** (ktlint or detekt):
   ```gradle
   dependencies {
       ktlintRuleset("io.nlopez.compose:rules-ktlint:0.1.0")
   }
   ```

2. **Run checks locally**:
   ```bash
   ./gradlew ktlintCheck detekt
   ```

3. **Add to CI/CD** (.github/workflows/lint.yml):
   ```yaml
   - name: Run ktlint
     run: ./gradlew ktlintCheck

   - name: Run detekt
     run: ./gradlew detekt
   ```

4. **Configure team policies**:
   - Create `detekt.yml` with rules enforcement levels
   - Document suppression policy (which rules can be suppressed)
   - Set up pre-commit hooks (optional)

---

## Anti-Pattern Categories & Priorities

### Critical (🔴) - Fix Immediately
- AP-001: Passing MutableState to children
- AP-002: Reading scroll state in composition
- AP-003: Creating state in loops
- AP-010: Missing contentDescription
- AP-011: Small touch targets (< 48dp)
- AP-016: API calls in composition
- AP-018: Hardcoded API keys
- AP-019: Secrets in SharedPreferences

### Important (🟠) - Fix Soon
- AP-004: Missing rememberSaveable
- AP-005: Creating modifiers in composition
- AP-006: Unstable collections in state
- AP-007: Heavy computation without remember
- AP-009: Missing keys in LazyColumn
- AP-012: Wrong semantics role
- AP-013: Hardcoded colors
- AP-014: Hardcoded font sizes
- AP-017: Missing DisposableEffect cleanup
- AP-020: Creating ViewModel in composable

### Warning (🟡) - Improve When Possible
- AP-008: Column instead of LazyColumn
- AP-015: Hardcoded dimensions

---

## Detection Patterns Reference

### Regex Patterns (Copy-Paste)

```bash
# AP-001: MutableState parameters
fun\s+\w+\([^)]*:\s*MutableState<[^>]+>[^)]*\)

# AP-006: Unstable collections
(data\s+class|class)\s+\w+.*:\s*(List|Set|Map)<[^>]+>\s*=

# AP-010: Missing contentDescription
(Icon|Image|AsyncImage)\s*\([^)]*contentDescription\s*=\s*null

# AP-013: Hardcoded colors
color\s*=\s*(Color\(0x|Color\.(White|Black|Red|Green|Blue))

# AP-014: Hardcoded font sizes
fontSize\s*=\s*\d+\s*\.sp|fontWeight\s*=\s*FontWeight\.(Bold|SemiBold|Medium)

# AP-018: Hardcoded secrets
(const\s+val|val)\s+(API_KEY|SECRET|PASSWORD|TOKEN)\s*=\s*"(sk-|api_|secret_)[a-zA-Z0-9]+"
```

### Grep Commands (Copy-Paste)

```bash
# Find all unskippable composables (after generating metrics)
grep "skippable=false" build/compose_metrics/*_composables.csv

# Find hardcoded colors
grep -r "color\s*=\s*Color\s*(" src/ | grep -v "MaterialTheme"

# Find missing @Immutable on data classes with collections
grep -B2 "val.*:\s*List<\|Set<" src/ | grep -v "@Immutable"

# Find API calls outside LaunchedEffect
grep -r "api\.\|repo\." src/ | grep -v "LaunchedEffect"
```

---

## Integration with SKILL.md

The main skill file (`SKILL.md`) references all these guides:

```yaml
references:
  - ./reference/material-design-3.md
  - ./reference/compose-rules-integration.md
  - ./reference/compose-antipatterns.md
  - ./reference/compiler-metrics-guide.md
  - ./reference/now-in-android-patterns.md
  - ./reference/typography-android.md
  - ./reference/color-theming.md
  - ./reference/spacing-touch-targets.md
  - ./reference/motion-android.md
  - ./reference/compose-patterns.md
```

When auditing code, pull relevant sections from these files based on violation category.

---

## Best Practices for Code Review

### During Review

1. **Use anti-pattern database** (compose-antipatterns.md)
   - Scroll to relevant anti-pattern
   - Share "Bad Example" and "Good Example" with author
   - Link to rationale

2. **Reference Material 3 specs** (material-design-3.md)
   - Colors: Is it using `MaterialTheme.colorScheme.*`?
   - Typography: Is it using `MaterialTheme.typography.*`?
   - Touch targets: Is interactive element ≥ 48dp?

3. **Check compiler metrics** (compiler-metrics-guide.md)
   - Is the composable skippable? (Check metrics)
   - Can we use `@Immutable` or `ImmutableList`?

4. **Recommend patterns from NiA** (now-in-android-patterns.md)
   - ViewModel + StateFlow setup
   - Modularization structure
   - Baseline profiles for performance

---

## Continuous Improvement

### Monthly Tasks

- [ ] Run `./gradlew detekt` and review violations
- [ ] Generate compiler metrics: `./gradlew assembleDebug` (with metrics enabled)
- [ ] Check for unskippable composables: `grep "skippable=false"`
- [ ] Review code changes against anti-pattern checklist

### Quarterly Review

- [ ] Update team's "Compose Rules" configuration (compose-rules-integration.md)
- [ ] Document any new patterns discovered (add to now-in-android-patterns.md)
- [ ] Benchmark app performance with MacroBenchmark
- [ ] Review Google's Now in Android for new architectural improvements

---

## Tool Integration

### IntelliJ IDEA / Android Studio

1. **Enable Real-Time Linting**:
   - Settings → Languages & Frameworks → Kotlin → Linter
   - ktlint should auto-detect compose-rules plugin

2. **Create Custom Inspections**:
   - Settings → Editor → Inspections → Kotlin
   - Search for "Compose" to see available inspections
   - Enable all critical ones

3. **Quick Fix**:
   - Alt+Enter on any violation to see suggested fixes

### CI/CD Integration

Add to `.github/workflows/lint.yml`:
```yaml
- name: Run Compose Linting
  run: ./gradlew ktlintCheck detekt

- name: Generate Metrics
  run: ./gradlew assembleDebug -Pandroidx.enableComposeCompilerMetrics=true

- name: Check Metrics
  run: grep "skippable=false" build/compose_metrics/*_composables.csv
```

---

## References

- **Compose Rules GitHub**: https://github.com/mrmans0n/compose-rules
- **Now in Android**: https://github.com/android/nowinandroid
- **Compose Compiler Metrics**: https://chrisbanes.me/posts/composable-metrics/
- **Material Design 3**: https://m3.material.io/
- **Jetpack Compose**: https://developer.android.com/develop/ui/compose

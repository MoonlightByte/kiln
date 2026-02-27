# Compose Rules Integration Guide

Integration with **compose-rules** (https://github.com/mrmans0n/compose-rules) for static analysis of Jetpack Compose code.

---

## Overview

Compose Rules provides a collection of linting rules that catch common mistakes and anti-patterns in Jetpack Compose development. The rules are implemented as both ktlint and detekt plugins, enabling seamless integration into your build process.

**Repository**: https://github.com/mrmans0n/compose-rules
**License**: Apache 2.0
**Maintenance**: Actively maintained (originally forked from Twitter's Compose rules)

---

## Installation

### Via Gradle (ktlint Integration)

```gradle
plugins {
    id "com.android.application"
    id "org.jmailen.kotlinter" version "3.14.0"  // ktlint plugin
}

dependencies {
    // Compose rules for ktlint
    ktlintRuleset("io.nlopez.compose:rules-ktlint:0.1.0")
}

ktlint {
    reporters {
        reporter("plain")
        reporter("sarif")
    }
    filter {
        exclude("**/generated/**")
    }
}
```

### Via Gradle (detekt Integration)

```gradle
plugins {
    id "io.gitlab.arturbosch.detekt" version "1.23.1"
}

dependencies {
    detektPlugins("io.nlopez.compose:rules-detekt:0.1.0")
}

detekt {
    source = files("src")
    config = files("detekt.yml")
}
```

---

## Supported Rules

### Compose Function Rules

**Rule: ComposeNaming**
- ✓ All `@Composable` functions must follow PascalCase naming
- ✓ Private functions should start with lowercase (when using internal composition)
- Rationale: Consistent with Kotlin class naming conventions

**Rule: ComposeLambdaParameterNaming**
- ✓ Lambda parameters in composables should be named descriptively (e.g., `onSomething`, `content`)
- Avoid single letters or generic names
- Rationale: Improves code clarity and IDE autocompletion

### State Management Rules

**Rule: UnreachableComposableRule**
- ✓ Detects composables that are defined but never called
- Rationale: Finds dead code

**Rule: RememberMissingRule**
- ✓ Warns when expensive operations aren't wrapped in `remember`
- Rationale: Prevents unnecessary recomposition performance hits

**Rule: ModifierMissingRule**
- ✓ Recommends `Modifier` as first parameter for all composables that use layout
- Rationale: Follows Material 3 and Google best practices for composability

### Parameter Rules

**Rule: MutableParametersRule**
- ✓ Flags `MutableState<T>` parameters (should be passed as values + callbacks)
- Example: `count: MutableState<Int>` → `count: Int, onCountChange: (Int) -> Unit`
- Rationale: Enforces unidirectional data flow

**Rule: PreviewPublicRule**
- ✓ Warns if preview functions are public (should be private)
- Rationale: Preview functions are for development only

---

## Running the Rules

### Command Line (ktlint)

```bash
# Check formatting and rules
./gradlew ktlintCheck

# Auto-fix issues where possible
./gradlew ktlintFormat

# Report specific files
./gradlew ktlint --info
```

### Command Line (detekt)

```bash
# Run detekt analysis
./gradlew detekt

# Generate HTML report
./gradlew detekt --html=build/reports/detekt/detekt.html
```

### CI/CD Integration

```yaml
# .github/workflows/lint.yml
name: Lint

on: [pull_request, push]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with:
          java-version: '17'

      - name: Run ktlint
        run: ./gradlew ktlintCheck

      - name: Run detekt
        run: ./gradlew detekt

      - name: Upload detekt report
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: detekt-report
          path: build/reports/detekt/
```

---

## Configuration

### ktlint Configuration (ktlint.properties)

```properties
# Disable specific rules
disabled_rules=rule-name-1,rule-name-2

# Custom indentation
indent_size=4

# Max line length
max_line_length=120
```

### detekt Configuration (detekt.yml)

```yaml
compose-rules:
  active: true
  ComposeNaming:
    active: true
  MutableParametersRule:
    active: true
  RememberMissingRule:
    active: true
    # Optional: Custom threshold for "expensive" operations
    threshold: 100 # complexity units
```

---

## Common Violations & Fixes

### Violation: MutableState Parameter

**Detected by**: `MutableParametersRule`

```kotlin
// BAD - Flagged by rule
@Composable
fun Counter(count: MutableState<Int>) {
    Button(onClick = { count.value++ }) {
        Text("Count: ${count.value}")
    }
}
```

**Fix**:
```kotlin
// GOOD - Passes rule
@Composable
fun Counter(count: Int, onCountChange: (Int) -> Unit) {
    Button(onClick = { onCountChange(count + 1) }) {
        Text("Count: $count")
    }
}
```

---

### Violation: Missing Modifier Parameter

**Detected by**: `ModifierMissingRule`

```kotlin
// BAD - No modifier parameter
@Composable
fun EventCard(title: String) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(16.dp)
    ) {
        Text(title)
    }
}
```

**Fix**:
```kotlin
// GOOD - Modifier as first parameter
@Composable
fun EventCard(
    title: String,
    modifier: Modifier = Modifier
) {
    Card(modifier = modifier.fillMaxWidth().padding(16.dp)) {
        Text(title)
    }
}
```

---

### Violation: Missing remember() for Expensive Operations

**Detected by**: `RememberMissingRule`

```kotlin
// BAD - Regex compiled every recomposition
@Composable
fun EmailValidator(email: String) {
    val emailRegex = Regex("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}")
    val isValid = emailRegex.matches(email)

    Text(if (isValid) "Valid" else "Invalid")
}
```

**Fix**:
```kotlin
// GOOD - Cached with remember
@Composable
fun EmailValidator(email: String) {
    val emailRegex = remember {
        Regex("[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}")
    }
    val isValid = emailRegex.matches(email)

    Text(if (isValid) "Valid" else "Invalid")
}
```

---

### Violation: Incorrect Naming Convention

**Detected by**: `ComposeLambdaParameterNaming`

```kotlin
// BAD - Generic lambda names
@Composable
fun Dialog(
    title: String,
    cb1: () -> Unit,
    cb2: () -> Unit
) {
    // ...
}
```

**Fix**:
```kotlin
// GOOD - Descriptive names
@Composable
fun Dialog(
    title: String,
    onConfirm: () -> Unit,
    onDismiss: () -> Unit
) {
    // ...
}
```

---

## Integration with IDE

### Android Studio / IntelliJ IDEA

1. **Enable Real-Time Inspection**: Settings → Languages & Frameworks → Kotlin → Linter
2. **Run Quick Fix**: Alt+Enter on any violation
3. **Suppress Warnings**: `@Suppress("ComposeNaming")` above function
4. **Custom Inspections Profile**: Create `.editorconfig` for team standards

```editorconfig
# .editorconfig
root = true

[*.kt]
indent_size = 4
max_line_length = 120
ij_kotlin_allow_trailing_comma = true
```

### VS Code with Kotlin Extension

```json
// .vscode/settings.json
{
    "kotlin.linting.debounce": 500,
    "kotlin.linting.enabled": true
}
```

---

## Best Practices for Adoption

### 1. Gradual Rollout

- Start with rules that don't require code changes (naming conventions)
- Fix critical rules first (MutableState, missing remember)
- Use `@Suppress` temporarily for legacy code

```kotlin
@Suppress("MutableParametersRule")
@Composable
fun LegacyComposable(state: MutableState<String>) {
    // Existing code - plan refactor
}
```

### 2. Team Configuration

```gradle
// gradle/compose-rules.gradle
ext {
    composeRulesVersion = "0.1.0"
}

dependencies {
    ktlintRuleset("io.nlopez.compose:rules-ktlint:${composeRulesVersion}")
    detektPlugins("io.nlopez.compose:rules-detekt:${composeRulesVersion}")
}
```

### 3. Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit
./gradlew ktlintCheck detekt || exit 1
```

### 4. Documentation

Document which rules are enforced in your project:

```md
# Compose Rules Configuration

## Enforced Rules

- ✓ **MutableParametersRule**: Strict enforcement (no suppressions)
- ✓ **ModifierMissingRule**: Strict enforcement
- ✓ **RememberMissingRule**: Exceptions for simple operations < 10 units complexity

## Suppression Policy

- MutableParametersRule: Only in legacy composables being refactored
- PreviewPublicRule: Never suppress, always fix

## Exemptions

- `/src/test/` - Test code exempt from naming conventions
- `/src/androidTest/` - Integration tests exempt
```

---

## References

- **Official Repo**: https://github.com/mrmans0n/compose-rules
- **Compose Guidelines**: https://google.android.com/design/develop-compose
- **ktlint**: https://ktlint.github.io/
- **detekt**: https://detekt.dev/

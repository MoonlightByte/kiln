# Keyboard Handling Patterns - Forms and Input Management

## Overview
LLMs frequently generate forms where the keyboard covers input fields, making them unusable. This document covers proper keyboard-aware layouts for both platforms.

## The Problem
When users tap a text field near the bottom of the screen, the soft keyboard appears and covers:
- The active input field itself
- Submit buttons
- Other form elements

Users cannot see what they're typing or submit the form without dismissing the keyboard first.

---

## Android (Jetpack Compose)

### BANNED: Static Padding

```kotlin
// NEVER do this - keyboard will cover inputs
Column(
    modifier = Modifier
        .fillMaxSize()
        .padding(bottom = 16.dp)  // Fixed padding ignores keyboard!
) {
    TextField(value = email, onValueChange = { email = it })
    TextField(value = password, onValueChange = { password = it })
    Button(onClick = { submit() }) {
        Text("Submit")
    }
}
```

### REQUIRED: WindowInsets.ime + imePadding()

**Android:**
```kotlin
@Composable
fun LoginForm() {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .imePadding()  // Automatically adjusts for keyboard height
            .verticalScroll(rememberScrollState())  // Enable scrolling
            .padding(16.dp)
    ) {
        TextField(
            value = email,
            onValueChange = { email = it },
            label = { Text("Email") },
            keyboardOptions = KeyboardOptions(
                keyboardType = KeyboardType.Email,
                imeAction = ImeAction.Next
            ),
            keyboardActions = KeyboardActions(
                onNext = { focusManager.moveFocus(FocusDirection.Down) }
            )
        )

        Spacer(modifier = Modifier.height(16.dp))

        TextField(
            value = password,
            onValueChange = { password = it },
            label = { Text("Password") },
            visualTransformation = PasswordVisualTransformation(),
            keyboardOptions = KeyboardOptions(
                keyboardType = KeyboardType.Password,
                imeAction = ImeAction.Done
            ),
            keyboardActions = KeyboardActions(
                onDone = {
                    focusManager.clearFocus()
                    submit()
                }
            )
        )

        Spacer(modifier = Modifier.height(24.dp))

        Button(
            onClick = { submit() },
            modifier = Modifier.fillMaxWidth()
        ) {
            Text("Login")
        }
    }
}
```

### BringIntoViewRequester for Precise Scrolling

When you need to ensure a specific field scrolls into view when focused:

```kotlin
@OptIn(ExperimentalFoundationApi::class)
@Composable
fun FormWithBringIntoView() {
    val bringIntoViewRequester = remember { BringIntoViewRequester() }
    val coroutineScope = rememberCoroutineScope()

    Column(
        modifier = Modifier
            .fillMaxSize()
            .imePadding()
            .verticalScroll(rememberScrollState())
    ) {
        // Many fields above...

        TextField(
            value = notes,
            onValueChange = { notes = it },
            label = { Text("Notes") },
            modifier = Modifier
                .bringIntoViewRequester(bringIntoViewRequester)
                .onFocusChanged { focusState ->
                    if (focusState.isFocused) {
                        coroutineScope.launch {
                            bringIntoViewRequester.bringIntoView()
                        }
                    }
                }
        )
    }
}
```

### Keyboard Actions Summary

| ImeAction | Keyboard Button | Use Case |
|-----------|----------------|----------|
| `ImeAction.Next` | "Next" arrow | Move to next field |
| `ImeAction.Done` | "Done" checkmark | Submit form or close keyboard |
| `ImeAction.Search` | Search icon | Trigger search |
| `ImeAction.Send` | Send arrow | Send message |
| `ImeAction.Go` | "Go" button | Navigate/open URL |

### Complete Android Form Pattern

```kotlin
@Composable
fun CompleteFormScreen(
    viewModel: FormViewModel = hiltViewModel()
) {
    val focusManager = LocalFocusManager.current
    val scrollState = rememberScrollState()

    // Dismiss keyboard on background tap
    Box(
        modifier = Modifier
            .fillMaxSize()
            .pointerInput(Unit) {
                detectTapGestures(onTap = {
                    focusManager.clearFocus()
                })
            }
    ) {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .imePadding()
                .verticalScroll(scrollState)
                .padding(16.dp)
        ) {
            OutlinedTextField(
                value = viewModel.name,
                onValueChange = viewModel::onNameChange,
                label = { Text("Name") },
                singleLine = true,
                keyboardOptions = KeyboardOptions(
                    capitalization = KeyboardCapitalization.Words,
                    imeAction = ImeAction.Next
                ),
                keyboardActions = KeyboardActions(
                    onNext = { focusManager.moveFocus(FocusDirection.Down) }
                ),
                modifier = Modifier.fillMaxWidth()
            )

            Spacer(Modifier.height(16.dp))

            OutlinedTextField(
                value = viewModel.email,
                onValueChange = viewModel::onEmailChange,
                label = { Text("Email") },
                singleLine = true,
                keyboardOptions = KeyboardOptions(
                    keyboardType = KeyboardType.Email,
                    imeAction = ImeAction.Next
                ),
                keyboardActions = KeyboardActions(
                    onNext = { focusManager.moveFocus(FocusDirection.Down) }
                ),
                modifier = Modifier.fillMaxWidth()
            )

            Spacer(Modifier.height(16.dp))

            OutlinedTextField(
                value = viewModel.phone,
                onValueChange = viewModel::onPhoneChange,
                label = { Text("Phone") },
                singleLine = true,
                keyboardOptions = KeyboardOptions(
                    keyboardType = KeyboardType.Phone,
                    imeAction = ImeAction.Next
                ),
                keyboardActions = KeyboardActions(
                    onNext = { focusManager.moveFocus(FocusDirection.Down) }
                ),
                modifier = Modifier.fillMaxWidth()
            )

            Spacer(Modifier.height(16.dp))

            OutlinedTextField(
                value = viewModel.message,
                onValueChange = viewModel::onMessageChange,
                label = { Text("Message") },
                minLines = 3,
                maxLines = 5,
                keyboardOptions = KeyboardOptions(
                    capitalization = KeyboardCapitalization.Sentences,
                    imeAction = ImeAction.Done
                ),
                keyboardActions = KeyboardActions(
                    onDone = {
                        focusManager.clearFocus()
                    }
                ),
                modifier = Modifier.fillMaxWidth()
            )

            Spacer(Modifier.height(24.dp))

            Button(
                onClick = {
                    focusManager.clearFocus()
                    viewModel.submit()
                },
                modifier = Modifier.fillMaxWidth()
            ) {
                Text("Submit")
            }

            // Extra padding at bottom for keyboard clearance
            Spacer(Modifier.height(32.dp))
        }
    }
}
```

### AndroidManifest.xml Configuration

Ensure your Activity has the correct soft input mode:

```xml
<activity
    android:name=".MainActivity"
    android:windowSoftInputMode="adjustResize">
</activity>
```

| Mode | Behavior |
|------|----------|
| `adjustResize` | REQUIRED - Resizes layout to fit keyboard |
| `adjustPan` | Pans content (can cut off top) |
| `adjustNothing` | Keyboard covers content (NEVER use for forms) |

---

## iOS (SwiftUI)

### BANNED: Static Layout Without Scroll

```swift
// NEVER do this - keyboard covers inputs
VStack {
    TextField("Email", text: $email)
    SecureField("Password", text: $password)
    Button("Login") { submit() }
}
.padding()
```

### REQUIRED: ScrollView + @FocusState

**iOS:**
```swift
struct LoginForm: View {
    @State private var email = ""
    @State private var password = ""
    @FocusState private var focusedField: Field?

    enum Field: Hashable {
        case email, password
    }

    var body: some View {
        ScrollViewReader { proxy in
            ScrollView {
                VStack(spacing: 16) {
                    TextField("Email", text: $email)
                        .textContentType(.emailAddress)
                        .keyboardType(.emailAddress)
                        .submitLabel(.next)
                        .focused($focusedField, equals: .email)
                        .id(Field.email)

                    SecureField("Password", text: $password)
                        .textContentType(.password)
                        .submitLabel(.done)
                        .focused($focusedField, equals: .password)
                        .id(Field.password)

                    Button("Login") {
                        focusedField = nil  // Dismiss keyboard
                        submit()
                    }
                    .buttonStyle(.borderedProminent)
                }
                .padding()
            }
            .onSubmit {
                switch focusedField {
                case .email:
                    focusedField = .password
                case .password:
                    focusedField = nil
                    submit()
                case .none:
                    break
                }
            }
            .onChange(of: focusedField) { newValue in
                if let field = newValue {
                    withAnimation {
                        proxy.scrollTo(field, anchor: .center)
                    }
                }
            }
        }
    }
}
```

### Dismiss Keyboard on Background Tap

```swift
struct ContentView: View {
    @FocusState private var isFocused: Bool

    var body: some View {
        ScrollView {
            VStack {
                TextField("Input", text: $text)
                    .focused($isFocused)
                // ... more fields
            }
        }
        .onTapGesture {
            isFocused = false  // Dismiss keyboard
        }
    }
}
```

### Keyboard Toolbar Actions

```swift
TextField("Amount", text: $amount)
    .keyboardType(.decimalPad)
    .toolbar {
        ToolbarItemGroup(placement: .keyboard) {
            Spacer()
            Button("Done") {
                focusedField = nil
            }
        }
    }
```

### Complete iOS Form Pattern

```swift
struct CompleteFormView: View {
    @State private var name = ""
    @State private var email = ""
    @State private var phone = ""
    @State private var message = ""
    @FocusState private var focusedField: FormField?

    enum FormField: Hashable {
        case name, email, phone, message
    }

    var body: some View {
        ScrollViewReader { proxy in
            ScrollView {
                VStack(alignment: .leading, spacing: 20) {
                    // Name Field
                    VStack(alignment: .leading, spacing: 4) {
                        Text("Name")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                        TextField("Your name", text: $name)
                            .textContentType(.name)
                            .textInputAutocapitalization(.words)
                            .submitLabel(.next)
                            .focused($focusedField, equals: .name)
                            .id(FormField.name)
                    }

                    // Email Field
                    VStack(alignment: .leading, spacing: 4) {
                        Text("Email")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                        TextField("your@email.com", text: $email)
                            .textContentType(.emailAddress)
                            .keyboardType(.emailAddress)
                            .textInputAutocapitalization(.never)
                            .submitLabel(.next)
                            .focused($focusedField, equals: .email)
                            .id(FormField.email)
                    }

                    // Phone Field
                    VStack(alignment: .leading, spacing: 4) {
                        Text("Phone")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                        TextField("(555) 123-4567", text: $phone)
                            .textContentType(.telephoneNumber)
                            .keyboardType(.phonePad)
                            .submitLabel(.next)
                            .focused($focusedField, equals: .phone)
                            .id(FormField.phone)
                    }

                    // Message Field
                    VStack(alignment: .leading, spacing: 4) {
                        Text("Message")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                        TextField("Your message...", text: $message, axis: .vertical)
                            .lineLimit(3...6)
                            .submitLabel(.done)
                            .focused($focusedField, equals: .message)
                            .id(FormField.message)
                    }

                    // Submit Button
                    Button(action: submit) {
                        Text("Submit")
                            .frame(maxWidth: .infinity)
                    }
                    .buttonStyle(.borderedProminent)
                    .padding(.top, 8)

                    // Extra space for keyboard
                    Spacer()
                        .frame(height: 100)
                }
                .textFieldStyle(.roundedBorder)
                .padding()
            }
            .onSubmit {
                advanceToNextField()
            }
            .onChange(of: focusedField) { _, newValue in
                if let field = newValue {
                    withAnimation(.easeInOut(duration: 0.3)) {
                        proxy.scrollTo(field, anchor: .center)
                    }
                }
            }
            .toolbar {
                ToolbarItemGroup(placement: .keyboard) {
                    Button("Previous") {
                        moveToPreviousField()
                    }
                    .disabled(focusedField == .name)

                    Button("Next") {
                        advanceToNextField()
                    }
                    .disabled(focusedField == .message)

                    Spacer()

                    Button("Done") {
                        focusedField = nil
                    }
                }
            }
            .onTapGesture {
                focusedField = nil
            }
        }
    }

    private func advanceToNextField() {
        switch focusedField {
        case .name: focusedField = .email
        case .email: focusedField = .phone
        case .phone: focusedField = .message
        case .message:
            focusedField = nil
            submit()
        case .none: break
        }
    }

    private func moveToPreviousField() {
        switch focusedField {
        case .email: focusedField = .name
        case .phone: focusedField = .email
        case .message: focusedField = .phone
        case .name, .none: break
        }
    }

    private func submit() {
        // Handle submission
    }
}
```

### When to Use ignoresSafeArea(.keyboard)

Use sparingly for specific cases where you want content to remain visible behind the keyboard:

```swift
// Chat input that should stay at bottom
VStack {
    // Messages list
    ScrollView {
        // ...
    }

    // Input bar - stays at screen bottom, keyboard overlaps messages
    HStack {
        TextField("Message", text: $message)
        Button("Send") { send() }
    }
    .padding()
    .background(.bar)
}
.ignoresSafeArea(.keyboard)  // Messages scroll, input stays put
```

---

## Form Navigation Patterns

### Tab/Next Button Behavior

Both platforms should support sequential field navigation:

| Action | Android | iOS |
|--------|---------|-----|
| Next Field | `ImeAction.Next` + `KeyboardActions(onNext = {...})` | `.submitLabel(.next)` + `.onSubmit { ... }` |
| Submit | `ImeAction.Done` + `KeyboardActions(onDone = {...})` | `.submitLabel(.done)` + `.onSubmit { ... }` |
| Manual Move | `focusManager.moveFocus(FocusDirection.Down)` | `focusedField = .nextField` |

### Focus Traversal Order

**Android:**
```kotlin
// Order is determined by layout position unless using focusRequester
val (first, second, third) = remember { FocusRequester.createRefs() }

TextField(
    modifier = Modifier.focusRequester(first),
    keyboardActions = KeyboardActions(
        onNext = { second.requestFocus() }
    )
)

TextField(
    modifier = Modifier.focusRequester(second),
    keyboardActions = KeyboardActions(
        onNext = { third.requestFocus() }
    )
)

TextField(
    modifier = Modifier.focusRequester(third),
    keyboardActions = KeyboardActions(
        onDone = { focusManager.clearFocus() }
    )
)
```

**iOS:**
```swift
enum Field: Hashable, CaseIterable {
    case first, second, third
}

@FocusState private var focusedField: Field?

// In onSubmit handler
if let current = focusedField,
   let index = Field.allCases.firstIndex(of: current),
   index + 1 < Field.allCases.count {
    focusedField = Field.allCases[index + 1]
} else {
    focusedField = nil
}
```

---

## Common Anti-Patterns

### BAD: Hardcoded Bottom Padding

```kotlin
// Android - WRONG
Modifier.padding(bottom = 300.dp)  // Guessing keyboard height

// iOS - WRONG
.padding(.bottom, 300)  // Guessing keyboard height
```

### BAD: No Scrolling on Long Forms

```kotlin
// Android - WRONG: Form longer than screen with no scroll
Column(modifier = Modifier.fillMaxSize()) {
    TextField(...)  // Field 1
    TextField(...)  // Field 2
    TextField(...)  // Field 3
    TextField(...)  // Field 4 - may be unreachable!
    Button(...)     // Submit - definitely covered
}
```

```swift
// iOS - WRONG: Form longer than screen with no scroll
VStack {
    TextField(...)  // Field 1
    TextField(...)  // Field 2
    TextField(...)  // Field 3
    TextField(...)  // Field 4 - may be unreachable!
    Button(...)     // Submit - definitely covered
}
```

### BAD: No Keyboard Dismissal

```kotlin
// Android - WRONG: User stuck with keyboard open
TextField(
    keyboardOptions = KeyboardOptions(imeAction = ImeAction.None)
)
```

```swift
// iOS - WRONG: No way to dismiss keyboard on numeric pad
TextField("Amount", text: $amount)
    .keyboardType(.decimalPad)
    // Missing: toolbar with Done button
```

### GOOD: Complete Keyboard Handling

```kotlin
// Android - CORRECT
Column(
    modifier = Modifier
        .fillMaxSize()
        .imePadding()                           // 1. React to keyboard
        .verticalScroll(rememberScrollState())  // 2. Enable scrolling
        .pointerInput(Unit) {                   // 3. Dismiss on tap
            detectTapGestures { focusManager.clearFocus() }
        }
) {
    TextField(
        keyboardOptions = KeyboardOptions(
            imeAction = ImeAction.Next           // 4. Enable navigation
        ),
        keyboardActions = KeyboardActions(
            onNext = { focusManager.moveFocus(FocusDirection.Down) }
        )
    )
    // ... more fields with proper imeAction chain
}
```

```swift
// iOS - CORRECT
ScrollViewReader { proxy in
    ScrollView {                                    // 1. Enable scrolling
        VStack {
            TextField(...)
                .focused($focusedField, equals: .field1)
                .id(FormField.field1)
                .submitLabel(.next)                // 2. Enable navigation
            // ... more fields
        }
    }
    .onTapGesture { focusedField = nil }          // 3. Dismiss on tap
    .onChange(of: focusedField) { _, field in     // 4. Scroll to field
        if let field {
            withAnimation { proxy.scrollTo(field) }
        }
    }
    .toolbar {                                     // 5. Keyboard toolbar
        ToolbarItemGroup(placement: .keyboard) {
            Button("Done") { focusedField = nil }
        }
    }
}
```

---

## Checklist

Before submitting any form screen, verify:

- [ ] **Scrolling enabled** - Form can scroll when keyboard appears
- [ ] **IME padding applied** (Android: `imePadding()`, iOS: automatic with ScrollView)
- [ ] **Focus state tracked** for field navigation
- [ ] **Next/Done actions** configured for each field
- [ ] **Background tap dismisses** keyboard
- [ ] **Keyboard toolbar** added for numeric/special keyboards (iOS)
- [ ] **Fields scroll into view** when focused
- [ ] **Submit button visible** when keyboard is open
- [ ] **Tested on device** with actual keyboard (not just preview)
- [ ] **windowSoftInputMode="adjustResize"** in AndroidManifest.xml

## Platform-Specific Notes

### Android
- Always use `imePadding()` modifier
- Set `windowSoftInputMode="adjustResize"` in manifest
- Use `BringIntoViewRequester` for complex scroll scenarios
- `focusManager.clearFocus()` dismisses keyboard

### iOS
- ScrollView handles most keyboard avoidance automatically
- Use `@FocusState` to track and control focus
- Add keyboard toolbar for non-standard keyboards (`.decimalPad`, `.phonePad`)
- `ScrollViewReader` + `scrollTo()` for precise positioning
- `.ignoresSafeArea(.keyboard)` only for special cases like chat UIs

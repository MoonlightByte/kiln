---
name: critique-design
description: Comprehensive mobile design critique covering visual hierarchy, typography, color harmony, spacing, and user experience
args:
  - name: design_file
    type: string
    description: Figma URL, design screenshot, or component code to critique
    required: true
  - name: focus
    type: string
    enum: [visual-hierarchy, typography, color, spacing, consistency, information-architecture, motion, emotional-design, all]
    description: Specific design area to focus on (default: all)
    required: false
  - name: platform
    type: string
    enum: [android, ios, web, auto]
    description: Target platform (default: auto-detect)
    required: false
severity_levels:
  - critical
  - important
  - warning
  - suggestion
---

# Mobile Design Critique Framework

Comprehensive design review based on 2025-2026 best practices, Material Design 3, and modern UX principles.

## Instructions

Analyze the provided design against modern mobile design standards. Focus on accessibility, user experience, visual hierarchy, and design consistency. Reference current design methodologies from leading companies (Google, Apple, Figma) and WCAG 2.2 accessibility standards.

## Design Review Criteria

### 1. Visual Hierarchy

**Principle:** Guide user attention from primary → secondary → tertiary content without effort.

#### Emphasis Levels (Critical)

**Guideline:**
- Limit emphasis to 3-5 levels (primary, secondary, tertiary, supporting, decorative)
- Use size, weight, color, and spacing to create clear distinction
- Apply "squint test": Step back and squint—what draws attention first?

**Audit Points:**
- [ ] Primary action/content is immediately obvious
- [ ] Secondary content supports but doesn't compete
- [ ] Tertiary content doesn't distract from primary
- [ ] No more than 5 emphasis levels (reduces cognitive load)
- [ ] Hierarchy remains clear at small screen sizes

**What to Check:**
```
Hierarchy violations:
- Multiple competing CTA buttons (all same size/weight)
- Headlines and body text have insufficient size contrast
- Decorative elements draw more attention than content
- Color emphasis inconsistent (green for success, but also for warnings)
```

#### Contrast and Attention (Important)

**Guideline:**
- Use vibrant colors strategically for CTAs and important content
- High contrast catches attention immediately
- WCAG 2.2: 4.5:1 for normal text, 3:1 for large text (18sp+/14pt bold+)
- Icons and graphics: 3:1 contrast ratio

**Audit Points:**
- [ ] Primary CTAs use highest contrast color
- [ ] Text meets WCAG 2.2 AA (4.5:1) minimum
- [ ] Interactive elements distinguishable by contrast alone
- [ ] Color not used as only indicator (e.g., red error without icon)

#### Visual Weight Distribution (Important)

**Guideline:**
- Balance visual weight across layout (don't cluster all content to one side)
- Use whitespace to create visual balance
- Grid-based layouts improve predictability

**Audit Points:**
- [ ] Visual weight distributed evenly across screen
- [ ] No dead space or awkward gaps
- [ ] Left-align vs. center-align intentional (avoid random mixing)
- [ ] Grid structure supports content (4-col or 6-col for mobile)

---

### 2. Typography (Material Design 3 Standard)

#### Type Scale (Critical)

**Material Design 3 Hierarchy:**
| Level | Size | Weight | Use Case |
|-------|------|--------|----------|
| Display | 57sp/45pt | Bold/Light | App title, hero headlines |
| Headline | 32sp/28pt | Bold | Section titles, major headings |
| Title | 22sp/20pt | Bold | Card titles, screen titles |
| Body | 16sp/16pt | Regular | Primary content, descriptions |
| Label | 12sp/11pt | Medium/Bold | Buttons, tags, labels |

**Audit Points:**
- [ ] Using M3 type scale (no arbitrary sizes)
- [ ] Type scale adapts to screen size (responsive)
- [ ] Font weights consistent with Material Design 3
- [ ] No text smaller than 12sp (unreadable)
- [ ] Line height: 1.5× base size (e.g., 16sp text → 24sp line height)

#### Typography Pairing (Important)

**Guideline:**
- Choose fonts from same family or with harmonious proportions
- Avoid mixing more than 2-3 typefaces
- Sans-serif for mobile (better legibility)
- Font expressiveness secondary to legibility

**Audit Points:**
- [ ] Maximum 2 typefaces (primary + accent)
- [ ] Both fonts legible at all sizes
- [ ] Font pairing has intentional contrast (serif + sans, or weight contrast)
- [ ] Accent font used sparingly (headlines only)
- [ ] Fallback fonts specified (system fonts if custom fonts fail)

#### Text Readability (Important)

**Guideline:**
- Content scannable before reading (structure via typography)
- Variations in size, weight, style guide reader
- Max line length: 40-50 characters for mobile

**Audit Points:**
- [ ] Text structure clear (headings, subheadings, body)
- [ ] Line length optimal for readability (<50 chars on mobile)
- [ ] Font size increases with hierarchy level
- [ ] Weight (bold) used for emphasis, not size alone
- [ ] Color not only difference between text levels

#### Dynamic Type / Text Scaling (Important)

**Audit Points:**
- [ ] Using `sp` units (Android) or `.font()` (iOS), not hardcoded px
- [ ] Text scales at 200% without layout breaking
- [ ] Truncation handles (ellipsis) for long text
- [ ] Testing at Largest accessibility text size (200%+)

---

### 3. Color Harmony & Accessibility

#### Color Palette Coherence (Critical)

**Guideline:**
- 2025 trend: Consistent, intentional color palettes (not random)
- Use color systems (Material You color tokens, design tokens)
- All colors derive from 1-2 base colors or designed system
- Avoid: Arbitrary color choices that clash with brand

**Audit Points:**
- [ ] Color palette has clear structure (primary, secondary, tertiary)
- [ ] All colors in palette work together harmoniously
- [ ] Sufficient contrast between adjacent colors
- [ ] Color palette finite and repeatable (not infinite random colors)
- [ ] Dark mode palette designed (not just inverted)

#### Contrast Compliance (Critical)

**WCAG 2.2 Levels (Mobile-Specific):**

**Level AA (Standard):**
- Normal text (< 18sp): 4.5:1 ratio
- Large text (18sp+): 3:1 ratio
- UI components & graphics: 3:1 ratio

**Level AAA (Enhanced):**
- Normal text: 7:1 ratio
- Large text: 4.5:1 ratio

**Audit Points:**
- [ ] All text on backgrounds meets 4.5:1 minimum
- [ ] Interactive elements (buttons, links) meet 3:1
- [ ] Icons and graphics have sufficient contrast
- [ ] Disabled state still meets 3:1 (if visible)
- [ ] Focus indicators contrast 3:1 with surroundings

#### Color Blindness Considerations (Important)

**5% of users have color vision deficiency (CVD):**
- Red-green: most common (7-8% of males, 0.4% of females)
- Blue-yellow: less common (~0.1%)
- Monochromatic: rare (~0.001%)

**Best Practices:**
- Don't use red-green alone (use icons, patterns, text labels)
- Avoid color combinations: red + green, red + brown, green + brown
- Test with Color Oracle or similar simulator
- Use shape/pattern in addition to color (e.g., ✓ for success, ✗ for error)

**Audit Points:**
- [ ] Color not sole indicator of meaning (error states use icon + text)
- [ ] Palette avoids red-green transitions
- [ ] High contrast maintained for CVD users
- [ ] Interactive states (hover, focus) not color-only
- [ ] Design remains functional in grayscale

#### Dark Mode Support (Important)

**2025 Standard: All apps support dark mode**

**Audit Points:**
- [ ] Dark mode palette designed (not auto-inverted)
- [ ] Colors readable in dark mode
- [ ] Sufficient contrast in dark mode (same 4.5:1/3:1 rules)
- [ ] Images/photos visible in dark mode (no transparency issues)
- [ ] Text color intentional in dark mode (not just `#FFFFFF`)

---

### 4. Whitespace & Spacing

#### Spacing System (Critical)

**Standard Base Units:**
- **4px or 8px** as foundation (pick one, then scale)
- **Typical scale:** 4, 8, 12, 16, 24, 32, 48, 64, 96, 128px
- **Mobile padding:** 16px (default), 24px (large), 12px (compact)
- **Mobile margins:** Match padding scale (16, 24, 32 between sections)

**Audit Points:**
- [ ] All spacing uses consistent base unit (4px or 8px)
- [ ] Spacing is multiple of base (not random values like 13px, 17px)
- [ ] Padding inside components consistent (10dp padding on all cards)
- [ ] Gap between elements follows scale
- [ ] Responsive spacing (desktop larger than mobile)

#### Visual Breathing Room (Important)

**Guideline:**
- Whitespace is not wasted space; it creates hierarchy and calm
- Dense layouts feel overwhelming; generous spacing feels premium
- 2025 trend: More whitespace, fewer elements per screen

**Audit Points:**
- [ ] Content doesn't feel cramped
- [ ] Whitespace supports visual hierarchy
- [ ] Top and bottom padding adequate (especially near notch/safe area)
- [ ] Between-section spacing ≥ within-group spacing
- [ ] No elements touching edges (minimum 16dp margin on all sides)

#### Grouping Through Spacing (Important)

**Principle:** Related elements close, unrelated elements far apart

**Audit Points:**
- [ ] Spacing within cards/groups: 8-12dp
- [ ] Spacing between cards: 16-24dp
- [ ] Spacing between sections: 32-48dp
- [ ] Implied grouping clear without explicit borders
- [ ] Whitespace rhythm creates visual pattern (not random)

#### Safe Areas & Notch Consideration (Critical for iOS)

**Audit Points:**
- [ ] Content respects safe area (avoids notch/home indicator)
- [ ] Padding around safe area consistent
- [ ] Critical buttons not hidden by safe area edges
- [ ] Android gesture nav. doesn't obscure buttons (24dp minimum from bottom)

---

### 5. Design Consistency & Pattern Reuse

#### Component Consistency (Critical)

**Guideline:**
- Same UI element = same appearance everywhere
- Buttons, cards, modals, forms should be predictable
- If patterns differ, difference must be intentional and justified

**Audit Points:**
- [ ] All buttons same style (e.g., rounded radius, padding)
- [ ] Card styling consistent (no some rounded, some flat)
- [ ] Form fields uniform (input style, error style, focus style)
- [ ] Icons consistent (line weight, style, size)
- [ ] Dividers (if used) have consistent appearance

#### Style Guide Compliance (Important)

**Audit Points:**
- [ ] Following design system tokens (colors, typography, spacing)
- [ ] Material Design 3 guidelines applied consistently
- [ ] iOS using Apple Human Interface Guidelines (if iOS)
- [ ] No hardcoded values (all design tokens)
- [ ] Deviation from system justified and documented

#### Predictable Interaction Patterns (Important)

**Guideline:**
- Users should predict behavior from similar patterns elsewhere
- Navigation patterns consistent (back button in same place)
- Error handling patterns consistent
- Confirmation dialogs look the same

**Audit Points:**
- [ ] Back button placement consistent
- [ ] Primary CTA button placement consistent
- [ ] Error messages styled identically
- [ ] Loading states look the same
- [ ] Disabled state appearance consistent

---

### 6. Information Architecture & Navigation

#### Content Hierarchy (Critical)

**Guideline:**
- Content organized by user mental model (not system structure)
- 5-7 top-level categories (avoid overwhelming users)
- Clear path from any screen to any other
- Minimize taps required for primary tasks

**Audit Points:**
- [ ] Primary content prominent (easy to find)
- [ ] Navigation categories < 7 items
- [ ] Rarely-used content not in primary navigation
- [ ] Deep nesting avoided (max 3 levels)
- [ ] Primary user flow achievable in < 4 taps

#### Navigation Clarity (Important)

**Guideline:**
- Users always know where they are
- Navigation always visible (tab bar or hamburger)
- Back button available or obvious way to return
- Breadcrumbs helpful for deep hierarchies

**Audit Points:**
- [ ] Current screen highlighted in navigation
- [ ] Tab bar/menu always visible (not hidden while scrolling)
- [ ] Back button present and functional
- [ ] No "orphaned" screens (every screen reachable)
- [ ] Navigation labels clear and concise (1-2 words)

#### User Flow Efficiency (Important)

**Guideline:**
- Primary user tasks require minimal taps/swipes
- Form fields in logical order
- Required vs. optional fields clear
- Success state obvious

**Audit Points:**
- [ ] Signup/login flow: < 4 screens (preferably 1-2)
- [ ] Create content flow: < 6 steps
- [ ] Main user task achievable in < 5 taps
- [ ] Form field order logical
- [ ] Progress indicator shown for multi-step flows
- [ ] Success state clear (confirmation message, navigation)

#### Mobile-Specific Navigation (Important)

**Audit Points (Android):**
- [ ] Back navigation follows Android conventions
- [ ] Gesture nav safe (buttons not near edges)
- [ ] Bottom sheet used for secondary actions (not top)
- [ ] Bottom nav bar clear and reachable

**Audit Points (iOS):**
- [ ] Navigation bar clear (shows title/back)
- [ ] Safe area respected (notch/home indicator)
- [ ] Gesture back recognized (swipe from left edge)
- [ ] Tab bar appears at bottom (not top)

---

### 7. Motion & Microinteractions

#### Animation Purpose (Important)

**Guideline:**
- Every animation serves a purpose (don't animate just for delight)
- Animations clarify state changes, not distract
- Loading states feel responsive (not stuck)
- Transitions feel smooth (not jarring)

**Audit Points:**
- [ ] Animations indicate state changes (open/close, load/complete)
- [ ] Animations don't block interaction (>500ms = provide feedback)
- [ ] Animations respect reduced-motion preference
- [ ] Loading spinners appear quickly (< 500ms of load)
- [ ] Dismiss animations are fast (not dragging)

#### Material Design 3 Motion Standards (Important)

**Timing:**
- Short interactions: 50-100ms (ripple, tap feedback)
- Medium transitions: 200-300ms (screen transitions)
- Long animations: 300-500ms (complex state changes)

**Easing:**
- Standard easing for forward motion
- Decelerate easing for objects entering screen
- Accelerate easing for objects leaving screen

**Audit Points:**
- [ ] Animations use M3 timing curves
- [ ] No animation longer than 500ms without user interaction
- [ ] Transitions feel natural (not instant, not slow)
- [ ] Microinteractions feel responsive (tap feedback immediate)
- [ ] Loading states show progress (spinners, progress bars)

#### Delight & Personality (Suggestion)

**2025 Trend: Emotional design through thoughtful microinteractions**

**Audit Points:**
- [ ] Hover states on interactive elements (if web)
- [ ] Tap feedback (ripple, color change) immediate
- [ ] Pull-to-refresh animation feels playful (not mechanical)
- [ ] Empty state has personality (icon + message)
- [ ] Success animations are celebratory (not mundane)

**Examples of Delight:**
- Duolingo's character reaction to wrong answer
- Slack's emoji reactions
- Pull-to-refresh animations that feel responsive
- Success animations with subtle bounce

---

### 8. Emotional Design & User Connection

#### Brand & Tone (Important)

**Guideline:**
- Design expresses brand personality
- Tone consistent across copy, color, typography, imagery
- Emotional design builds trust and engagement

**Audit Points:**
- [ ] Color palette reflects brand personality (playful vs. professional)
- [ ] Typography aligns with brand voice
- [ ] Imagery and icons consistent with brand
- [ ] Empty states, errors communicate with brand voice
- [ ] Overall feel intentional (not generic)

#### Accessibility as Inclusion (Critical)

**Guideline:**
- Accessible design is inclusive design
- Supports users with disabilities, older users, users in bright sunlight
- Improves experience for all users (captions help in noisy environments)

**Audit Points:**
- [ ] Color contrast sufficient (WCAG 2.2 AA minimum)
- [ ] Text resizable without breaking layout
- [ ] Touch targets 48×48dp minimum
- [ ] Focus indicators visible
- [ ] Icons have text labels (not icon-only buttons)
- [ ] Loading states announced (not silent spinners)

#### User Feedback & Validation (Important)

**Principle:** Users should feel heard and understood

**Audit Points:**
- [ ] Form validation provides helpful error messages
- [ ] Success states confirm completion
- [ ] Loading states show progress (not silent wait)
- [ ] Empty states guide next action
- [ ] Disabled state explains why (not just grayed out)

#### Trust & Reassurance (Important)

**What builds trust:**
- Consistent, predictable behavior
- Clear pricing/cost before commitment
- Security indicators (padlock, https)
- User control (can undo, go back, cancel)
- Transparent processes (multi-step confirmation)

**Audit Points:**
- [ ] Destructive actions require confirmation
- [ ] Undo available for important actions
- [ ] Privacy/security controls obvious
- [ ] Pricing transparent before purchase
- [ ] User data handling clear

---

## Platform-Specific Considerations

### Android (Material Design 3)

- [ ] Using Material Design 3 color system (`MaterialTheme.colorScheme.*`)
- [ ] Shape scale applied consistently (4dp to 28dp corner radius)
- [ ] Tonal elevation for depth (not shadows)
- [ ] Bottom app bar for primary actions (not top)
- [ ] Ripple feedback on tappable elements
- [ ] Floating action button (FAB) for primary action

### iOS (Apple Human Interface Guidelines)

- [ ] Safe area respected (notch, home indicator)
- [ ] Consistent with iOS design language
- [ ] Using SF Symbols for icons
- [ ] Native UI components (not over-customized)
- [ ] Gesture navigation (swipe back)
- [ ] Dark mode support

---

## Design to Review

```
{{design_file}}
```

---

## Critique Output Format

Generate structured critique report:

```markdown
## Design Critique Report: [Design Name]

### Executive Summary

**Overall Assessment:** [Rating: Excellent/Good/Fair/Needs Improvement]

**Key Strengths:**
1. [Major positive aspect]
2. [Major positive aspect]

**Key Opportunities:**
1. [Primary improvement area]
2. [Secondary improvement area]

---

## Visual Hierarchy Analysis

### Current State
[Describe how visual hierarchy guides user attention]

### Strengths
- [What works well about the hierarchy]

### Opportunities
- [Contrast issues, competing elements, unclear emphasis]
- **Recommendation:** [How to improve]

### Hierarchy Checklist
- [ ] Primary element immediately obvious
- [ ] Secondary content supports without competing
- [ ] Squint test reveals correct priority
- [ ] 3-5 emphasis levels maximum

---

## Typography Analysis

### Font Selection
- **Primary Font:** [Font name, used for body/headlines]
- **Secondary Font:** [Font name if used]
- **Verdict:** ✅/⚠️/❌ [Compliant/Warning/Issue]

### Type Scale Assessment
| Level | Size | Weight | Assessment |
|-------|------|--------|------------|
| Display | 57sp+ | Bold | ✅ |
| Headline | 28-32sp | Bold | ⚠️ |
| Title | 20-22sp | Bold | ✅ |
| Body | 14-16sp | Regular | ✅ |
| Label | 12sp | Medium | ✅ |

**Issues Found:**
- [Hardcoded sizes not using type scale]
- [Inconsistent font weights]
- [Line height not optimal]

**Recommendations:**
1. [Most critical typography fix]
2. [Secondary fix]

---

## Color & Contrast Analysis

### Color Palette Assessment
| Color | Hex | Usage | Contrast | Notes |
|-------|-----|-------|----------|-------|
| Primary | #6200EE | CTA buttons | ✅ 5.2:1 | Good |
| Secondary | #03DAC6 | Accents | ⚠️ 3.8:1 | Low contrast |
| Background | #FFFFFF | Surface | ✅ | - |

**Contrast Violations:**
- [Color pair with insufficient contrast]
- **Fix:** [Recommend contrast value or color change]

**Color Accessibility:**
- CVD Safe: ✅/❌
- Dark Mode Support: ✅/❌
- Missing patterns/icons: ✅/❌

**Recommendations:**
1. Increase secondary color contrast to 4.5:1
2. Add icons to error states (not color-only)
3. Test dark mode with adjusted palette

---

## Spacing & Layout Analysis

### Spacing Audit
- **Base Unit:** 8px ✅
- **Padding (default):** 16dp ✅
- **Margin (between sections):** 24dp ⚠️ (should be 32dp)
- **Whitespace Quality:** Good (breathing room)

**Issues:**
- Random spacing values (13px, 17px, 21px) - should use scale
- Dense card layout (8dp padding) feels cramped
- No visual separation between sections

**Recommendations:**
1. Standardize all spacing to 8px scale (8, 16, 24, 32, 48, 64)
2. Increase section margins to 32dp
3. Add breathing room around headers

---

## Design Consistency Analysis

### Component Audit
| Component | Consistency | Notes |
|-----------|------------|-------|
| Buttons | ❌ | Mixed border radius (4dp vs 8dp) |
| Cards | ⚠️ | Padding consistent, shadows vary |
| Input fields | ✅ | Uniform style |
| Icons | ✅ | Consistent weight and size |

**Inconsistencies Found:**
- Primary buttons use 4dp radius, secondary use 8dp
- Some cards have shadows, others don't
- Icon sizes range 20-24dp (should standardize)

**Recommendations:**
1. Define button corner radius (recommend 8dp for M3)
2. Apply consistent shadow to all cards (or remove)
3. Standardize icon size (20dp or 24dp)

---

## Information Architecture & Navigation

### Navigation Audit
- **Top-level categories:** 6 (acceptable, < 7)
- **Max depth:** 3 levels ✅
- **Primary task flow:** 5 taps ✅ (< 7 is good)
- **Current screen indication:** ✅ Highlighted in nav

**Issues:**
- [Navigation label ambiguity]
- [Deep nesting on secondary flows]
- [No back button on modal]

**Recommendations:**
1. Rename "Explore" to "Browse" (clearer intent)
2. Move third-level content to separate screens
3. Add back button to all modals

---

## Motion & Microinteractions

### Animation Review
| Animation | Duration | Purpose | Assessment |
|-----------|----------|---------|------------|
| Button tap | 150ms | Feedback | ✅ |
| Screen transition | 300ms | Navigation | ✅ |
| Loading spinner | ∞ | Progress | ❌ (no indication) |
| Pull-to-refresh | 200ms | Responsiveness | ✅ |

**Issues:**
- Loading spinner doesn't indicate progress (silent wait)
- No microinteraction on form submission
- Transitions feel instant (not smooth)

**Recommendations:**
1. Add progress indicator to loading states
2. Add button press animation (color change + ripple)
3. Verify transition animations use proper easing

---

## Emotional Design & Trust

### Brand & Tone Assessment
- **Personality:** [Professional/Playful/Friendly/etc.]
- **Consistency:** ✅/⚠️/❌
- **Trust signals:** [Security, transparency, user control]

**Strengths:**
- Friendly tone in empty states
- Clear error messaging

**Opportunities:**
- Add more delight to success states
- Increase user control indicators
- Make privacy settings more visible

**Recommendations:**
1. Celebrate successful submissions with animation
2. Add "undo" option to destructive actions
3. Show privacy policy link in signup flow

---

## Accessibility Compliance

### WCAG 2.2 AA Assessment

| Criterion | Status | Issues |
|-----------|--------|--------|
| 1.4.3 Contrast | ✅ | None |
| 1.4.4 Text Resize | ⚠️ | Large text breaks layout |
| 2.5.8 Target Size | ✅ | 48×48dp minimum |
| 1.1.1 Text Alternatives | ✅ | Icons have labels |

**Violations:**
- [Specific accessibility issue with fix]

**Recommendations:**
1. Test at 200% text size (Settings > Accessibility)
2. Verify focus indicators are visible
3. Ensure all images have alt text

---

## Summary & Priority Actions

### Severity Breakdown
- 🔴 **Critical:** 2 issues (contrast, spacing scale)
- 🟠 **Important:** 3 issues (consistency, navigation)
- 🟡 **Warning:** 2 issues (motion, delight)
- 🔵 **Suggestion:** 2 ideas (polish, enhancement)

### Top 3 Priority Fixes
1. **Increase secondary color contrast** (affects trust/accessibility)
   - Current: 3.8:1 | Target: 4.5:1
   - Fix: Adjust secondary color from #03DAC6 to #00BCD4

2. **Standardize spacing scale** (affects consistency/polish)
   - Issue: Random values (13, 17, 21px)
   - Fix: Use 8px scale exclusively (8, 16, 24, 32, 48, 64)

3. **Add progress indication to loading states** (affects trust/engagement)
   - Issue: Silent spinner feels stuck
   - Fix: Add progress bar or animated percentage

### Estimated Implementation Time
- Critical fixes: 2-4 hours
- Important improvements: 4-8 hours
- Polish/suggestions: 2-4 hours

### Next Steps
1. [Most urgent action]
2. [Validate with user testing]
3. [Document in design system]

---

## Detailed Recommendations

[Provide specific fixes with code examples if applicable, color hex values, measurements in dp/pt, etc.]

```

## Research & Reference

Based on 2025-2026 design best practices:

- [Visual Hierarchy: Key UX Principles That Drive Results](https://www.sessions.edu/notes-on-design/visual-hierarchy-key-ux-principles-that-drive-results/)
- [Improving Visual Hierarchy: Techniques Guide (2026)](https://www.parallelhq.com/blog/what-can-be-used-to-improve-visual-hierarchy)
- [Typography – Material Design 3](https://m3.material.io/styles/typography/applying-type)
- [Figma Blog: How we do design critiques at Figma](https://www.figma.com/blog/design-critiques-at-figma/)
- [WCAG 2.2 Guide for Accessible Mobile Apps](https://www.adatray.com/blog/wcag-2-2-mobile-app-accessibility)
- [Spacing Rules for Mobile App Layouts](https://thisisglance.com/learning-centre/what-spacing-rules-create-better-mobile-app-layouts)
- [Information Architecture for Mobile Design (2025)](https://www.interaction-design.org/literature/topics/information-architecture)
- [Microinteractions: Boosting User Engagement](https://thisisglance.com/blog/microinteractions-boosting-user-engagement-in-mobile-app-design)
- [Emotional Design in Mobile Apps (2025)](https://www.graphiceagle.com/emotional-design-in-mobile-apps-how-to-connect-with-users-in-2025)
- [Accessibility & Color Blindness](https://www.boia.org/blog/accessibility-tips-using-accessible-colors-in-mobile-apps)
- [Material 3 Design System Overview](https://m3.material.io/styles)
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)

# Heading Hierarchy and Semantic Structure - NVDA Findings

Tested using: NVDA + Chrome + Live Server  
Tool used: NVDA Elements List (`NVDA + F7`) — Headings tab and Landmarks tab

---

# WCAG Mapping

## Heading Hierarchy

Principle: Perceivable  
Guideline: 1.3 Adaptable  
Success Criterion: 1.3.1 Info and Relationships  
Requirement: Heading levels must convey structure and relationships between content sections

Principle: Operable  
Guideline: 2.4 Navigable  
Success Criterion: 2.4.6 Headings and Labels  
Requirement: Headings must describe the topic or purpose of their section

## Semantic Structure

Principle: Perceivable  
Guideline: 1.3 Adaptable  
Success Criterion: 1.3.1 Info and Relationships  
Requirement: Structure conveyed visually must also be available to assistive technology

Principle: Operable  
Guideline: 2.4 Navigable  
Success Criterion: 2.4.1 Bypass Blocks  
Requirement: Landmark regions allow screen reader users to skip directly to main content or navigation

---

# Heading Hierarchy

## How to Test

Do not just press `H` to navigate headings — that only confirms NVDA can find them.  
The real test is `NVDA + F7` → Headings tab. This shows the complete heading tree the way a screen reader user scans a page before reading — exactly like a sighted user visually scans headings to understand page structure at a glance.

---

## Case 1: `div` + `strong` used instead of a heading tag

Failure:
```html
<div><strong>Latest Articles</strong></div>
```

NVDA Elements List (Headings tab): Not listed — completely invisible to screen readers

Why it fails: `strong` only adds visual bold styling. It carries no semantic meaning. NVDA builds the heading tree from actual `h1`–`h6` tags only. A screen reader user scanning headings will never find this content and has no idea a section called "Latest Articles" even exists on the page.

Fix: Use the appropriate heading tag that fits the hierarchy.
```html
<h2>Latest Articles</h2>
```

---

## Case 2: Skipping heading levels — `h1` jumping to `h4`

Failure:
```html
<h1>Welcome to TechBlog</h1>
<div><strong>Latest Articles</strong></div>
<h4>Article One — Getting Started With React</h4>
```

NVDA Elements List (Headings tab):
```
Level 1 — Welcome to TechBlog
Level 4 — Article One — Getting Started With React
```

Why it fails: `h2` and `h3` are missing entirely. Screen reader users rely on heading levels to understand parent-child relationships between sections. A missing level makes the structure feel disoriented — like a table of contents jumping from Chapter 1 directly to Chapter 4 with nothing in between.

Fix: Never skip heading levels. Use CSS for visual sizing, not heading level changes.
```html
<h1>Welcome to TechBlog</h1>
<h2>Latest Articles</h2>
<h3>Article One — Getting Started With React</h3>
```

---

## Case 3: Heading level going backwards — `h2` after `h4`

Failure:
```html
<h4>Article One — Getting Started With React</h4>
<h2>Article Two — CSS Grid vs Flexbox</h2>
```

NVDA Elements List (Headings tab):
```
Level 4 — Article One — Getting Started With React
Level 2 — Article Two — CSS Grid vs Flexbox
```

Why it fails: Heading levels should only go deeper for subsections or return to the same level for sibling sections. Jumping backwards from `h4` to `h2` mid-page creates a completely disoriented structure. Both articles are siblings — they should be at the same level.

Fix: Keep sibling sections at the same heading level consistently.
```html
<h3>Article One — Getting Started With React</h3>
<h3>Article Two — CSS Grid vs Flexbox</h3>
```

---

## Case 4: `h5` used with no parent heading context

Failure:
```html
<div class="featured">
    <h5>Featured This Week</h5>
</div>
```

NVDA Elements List (Headings tab):
```
Level 5 — Featured This Week
```

Why it fails: This heading appears floating with no `h2`, `h3`, or `h4` above it in its section to establish where it sits in the page. A screen reader user jumping to it has no idea what section it belongs to. It is like reading a subchapter with no chapter title above it.

Fix: Since this is a top-level section inside main, `h2` is the correct level.
```html
<section aria-label="Featured this week">
    <h2>Featured This Week</h2>
</section>
```

---

## Case 5: `strong` used instead of semantic labels for article metadata

Failure:
```html
<div class="article-meta">
    <strong>Author</strong>
    <p>Jane Doe</p>
    <strong>Published</strong>
    <p>May 2026</p>
</div>
```

NVDA Elements List (Headings tab): Neither label listed

Why it fails: `strong` is purely visual emphasis. These are not headings and should not be — they are metadata. Using `strong` creates a fake visual structure that has no semantic equivalent for screen readers. NVDA reads them as plain bold text with no structural meaning.

Fix: Use `footer` inside `article` for metadata. Express label and value as plain descriptive text.
```html
<footer>
    <p>Author: Jane Doe</p>
    <p>Published: May 2026</p>
</footer>
```

---

## Case 6: `h6` used in sidebar with no heading hierarchy

Failure:
```html
<div class="sidebar">
    <h6>Related Posts</h6>
    <h6>Newsletter</h6>
</div>
```

NVDA Elements List (Headings tab):
```
Level 6 — Related Posts
Level 6 — Newsletter
```

Why it fails: `h6` implies five levels of parent headings above it. Using it as the first heading in a sidebar means the entire hierarchy above it is missing. Both sidebar sections sharing the same `h6` also gives users no way to understand their relationship to the rest of the page.

Fix: Use `h2` for top-level sidebar sections since they are siblings to main content sections.
```html
<aside aria-label="Sidebar">
    <h2>Related Posts</h2>
    <section aria-label="Newsletter signup">
        <h2>Newsletter</h2>
    </section>
</aside>
```

---

# Semantic Structure

## How to Test

Press `NVDA + F7` → Landmarks tab. This shows all semantic landmarks NVDA detected. Screen reader users use landmarks to skip directly to the part of the page they want — exactly like a sighted user visually jumps to the navigation or footer. Press `D` to jump between landmarks while browsing.

---

## Case 7: All structural divs — no landmarks detected

Failure:
```html
<div class="header">...</div>
<div class="nav">...</div>
<div class="main-content">...</div>
<div class="sidebar">...</div>
<div class="footer">...</div>
```

NVDA Elements List (Landmarks tab): Nothing listed  
NVDA with `D` key: Announces "no next landmark"

Why it fails: Class names are completely invisible to assistive technology. A page built entirely with divs has no navigable landmark structure. Screen reader users must listen to the entire page linearly from top to bottom — they cannot skip to main content, jump to navigation, or go directly to the footer.

Fix: Replace every structural div with its semantic landmark equivalent.

| Broken | Fixed | NVDA landmark name |
|---|---|---|
| `div.header` | `header` | banner |
| `div.nav` | `nav aria-label="Main navigation"` | navigation |
| `div.main-content` | `main` | main |
| `div.sidebar` | `aside aria-label="Sidebar"` | complementary |
| `div.footer` | `footer` | contentinfo |

---

## Case 8: Headings inside landmarks do not appear in the Landmarks tab

Fixed code:
```html
<main>
    <h1>Welcome to TechBlog</h1>
</main>
```

NVDA with `D` key: Announces "main landmark" followed by "Welcome to TechBlog heading level 1"  
NVDA Elements List (Landmarks tab): Shows `main` — but `Welcome to TechBlog` heading is not listed here  
NVDA Elements List (Headings tab): Shows `Level 1 — Welcome to TechBlog` correctly

Why this happens: The Headings tab and Landmarks tab are completely independent views. The Landmarks tab shows only landmark containers — not the content inside them. Headings inside a landmark only appear in the Headings tab. This is not a bug — it is how NVDA Elements List is designed. Both tabs together give the complete picture of page structure.

---

## Case 9: `section` with `aria-label` appears as "region" in landmark tree

Fixed code:
```html
<section aria-label="Featured this week">
    <h2>Featured This Week</h2>
</section>
```

NVDA with `D` key: Announces "Featured this week region"  
NVDA Elements List (Landmarks tab): Listed as region — not "section"

Why this happens: This is expected ARIA behaviour. The `section` HTML element maps to the ARIA `region` role when it has an accessible name via `aria-label` or `aria-labelledby`. NVDA uses the ARIA role name, not the HTML element name. So "region" in the landmark tree means the `section` element is working correctly.

Important: A `section` without `aria-label` does NOT appear in the landmark tree at all. The `aria-label` is what makes it discoverable as a landmark.

---

## Case 10: Discrepancy between Elements List tree label and `D` key announcement

Observed behaviour:  
NVDA `D` key correctly announces "Featured this week region"  
NVDA Elements List tree displays the same element with an incorrect label such as "main" because NVDA organizes landmarks hierarchically based on DOM structure.

Why this happens: This is a visual rendering quirk in NVDA's Elements List tree display — not a real structural error. The `D` key announcement reflects what NVDA actually computes from the accessibility tree at runtime.

The `D` key navigation announcement is always the ground truth — not the tree label.

How to test correctly:
- Use Elements List (`NVDA + F7`) for a quick structural overview
- Use `D` key navigation to verify what NVDA actually announces for each landmark
- If the two conflict, always trust the `D` key announcement
- Never rely on the tree label alone when documenting findings

---

## Case 11: Multiple `nav` elements need `aria-label` to distinguish them

Failure:
```html
<nav>...</nav>
<nav>...</nav>
```

NVDA Elements List (Landmarks tab):
```
navigation
navigation
```

Why it fails: When two `nav` landmarks have the same name, screen reader users cannot tell them apart. Both appear as "navigation" in the landmark tree with no way to distinguish which is the main nav and which is the footer nav.

Fix: Always add `aria-label` when there are multiple instances of the same landmark type on a page.
```html
<nav aria-label="Main navigation">...</nav>
<nav aria-label="Footer navigation">...</nav>
```

NVDA Elements List (Landmarks tab) after fix:
```
navigation  Main navigation
navigation  Footer navigation
```

---

## Case 12: `div` used instead of `ul`/`li` for list items

Failure:
```html
<div class="related-list">
    <div>Post 1 — Intro to TypeScript</div>
    <div>Post 2 — Node.js Best Practices</div>
    <div>Post 3 — Understanding Closures</div>
</div>
```

NVDA announces: Reads each item as plain text with no list context

Why it fails: NVDA announces proper lists as "list with 3 items" before reading — giving users immediate context about what is coming and how many items to expect. Div-based lists give no such announcement. Users have no idea these items are related or how many there are.

Fix: Use `ul` for unordered lists, `ol` for numbered lists.
```html
<ul>
    <li>Post 1 — Intro to TypeScript</li>
    <li>Post 2 — Node.js Best Practices</li>
    <li>Post 3 — Understanding Closures</li>
</ul>
```

NVDA now announces: "List with 3 items — bullet Post 1, bullet Post 2, bullet Post 3"

---

## Key Learnings About NVDA Elements List

- `NVDA + F7` opens the Elements List dialog
- Switch between Headings, Landmarks, Links, Buttons tabs
- Headings tab and Landmarks tab are independent — headings inside a landmark only appear in the Headings tab, not the Landmarks tab
- `section` only appears as a landmark when it has `aria-label` or `aria-labelledby` — without it NVDA skips it entirely as a landmark
- `section` with `aria-label` appears as region in the landmark tree — this is correct ARIA behaviour, not a bug
- Divs never appear in the Landmarks tab regardless of class names or styling
- `D` key jumps between landmarks in browse mode — if NVDA says "no next landmark" the page has no semantic structure at all
- The `D` key announcement is always the ground truth — the Elements List tree can occasionally show visual rendering quirks that do not reflect actual NVDA behaviour
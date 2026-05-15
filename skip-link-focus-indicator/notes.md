# Focus Indicators and Skip Navigation - NVDA Findings

Tested using: NVDA + Chrome + Live Server  
Keyboard testing: Tab, Shift+Tab, Enter

---

# WCAG Mapping

## Skip Navigation

Principle: Operable  
Guideline: 2.4 Navigable  
Success Criterion: 2.4.1 Bypass Blocks  
Requirement: A mechanism must be available to skip blocks of content that are repeated on multiple pages

## Focus Indicators

Principle: Operable  
Guideline: 2.4 Navigable  
Success Criterion: 2.4.7 Focus Visible  
Requirement: Any keyboard operable interface must have a visible keyboard focus indicator

Principle: Operable  
Guideline: 2.4 Navigable  
Success Criterion: 2.4.11 Focus Appearance (WCAG 2.2)  
Requirement: Focus indicator must meet minimum size and contrast requirements

---

# Skip Navigation

## How to Test

Open the page in browser. Without clicking anything, press `Tab` once.  
The skip link should appear visually and NVDA should announce it.  
Press `Enter` to activate it — focus should jump directly to main content,  
bypassing all navigation links.

---

## Case 1: No skip navigation link

Failure:
```html
<nav aria-label="Main navigation">
    <a href="/home">Home</a>
    <a href="/about">About</a>
    <a href="/articles">Articles</a>
    <a href="/contact">Contact</a>
</nav>
<main id="main-content">...</main>
```

NVDA + Tab behaviour: Reads every nav link one by one before reaching main content  
```
Home link
About link
Articles link
Contact link
← only now does focus reach main content
```

Why it fails: Keyboard and screen reader users must Tab through every navigation link on every page load to reach the actual content. On pages with large navigation menus this becomes extremely tedious. WCAG 2.4.1 requires a mechanism to skip repeated blocks.

Fix: Add a skip link as the very first element in body. Hidden off-screen by default, visible on focus.
```html
<a class="skip-link" href="#main-content">Skip to main content</a>
```

```css
.skip-link {
    position: absolute;
    top: -100%;
    left: 0;
    background: #000;
    color: #fff;
    padding: 0.75rem 1.25rem;
    font-size: 1rem;
    text-decoration: none;
    z-index: 9999;
}

.skip-link:focus {
    top: 0;
}
```

NVDA + Tab after fix:
```
Skip to main content link    ← appears on first Tab
← press Enter
Welcome to TechBlog heading  ← focus jumps directly to main
```

---

## Case 2: Why `display:none` and `visibility:hidden` cannot be used to hide the skip link

A common mistake when hiding the skip link is using `display:none` or `visibility:hidden`.

```css
/* Wrong — removes element from tab order entirely */
.skip-link {
    display: none;
}

.skip-link:focus {
    display: block;
}
```

Why it fails: `display:none` removes the element from the accessibility tree and tab order. It can never receive focus, so the `:focus` rule never fires. The skip link becomes completely unreachable by keyboard.

Fix: Use `position: absolute` with a negative `top` value to move it off-screen visually while keeping it in the tab order. Bring it back on focus by setting `top: 0`.

---

## Case 3: `main` needs `tabindex="-1"` to receive programmatic focus

Fixed code:
```html
<main id="main-content" tabindex="-1">
```

Why this is needed: When the skip link is activated, the browser moves focus to the element with the matching `id`. But `main` is not natively focusable. Without `tabindex="-1"`, some browsers move the viewport to the element but do not actually move keyboard focus there — the next Tab press goes back to where focus was before. `tabindex="-1"` makes the element programmatically focusable without adding it to the natural tab order.

Note: `tabindex="-1"` means the element cannot be reached by Tab — only by programmatic focus like a skip link click or JavaScript. This is correct behaviour for `main`.If `tabindex="0"`is used, `main` gets activated everytime in the flow as per the natural tab order which is uncessary.

---

# Focus Indicators

## How to Test

Open the page. Press `Tab` to move through interactive elements.  
Every focused element — links, buttons, inputs — must show a clearly visible focus ring.  
If focus moves but nothing is visible on screen, the focus indicator is broken.

---

## Case 4: `outline: none` with no alternative

Failure:
```css
a:focus,
button:focus,
input:focus {
    outline: none;
}
```

NVDA behaviour: NVDA still announces focused elements correctly — screen readers do not rely on the visual outline  
Keyboard user behaviour: Focus moves invisibly — no visual indication of where keyboard focus is

Why it fails: Sighted keyboard users — including people with motor disabilities who cannot use a mouse — have no way to see where focus is on the page.  WCAG 2.4.7 requires focus to always be visible.

Fix: Never use `outline: none` without providing an explicit alternative focus style.
```css
*:focus {
    outline: 3px solid #005fcc;
    outline-offset: 2px;    
}
```

Why these values:
- `3px solid` — thick enough to be clearly visible against most backgrounds
- `outline-offset: 2px` — small gap between element edge and ring, prevents outline merging into border
- Colour `#005fcc` — high contrast blue that meets WCAG contrast ratio against white backgrounds

---

## Case 5: Focus style on `:hover` only — keyboard users get nothing

Failure:
```css
button:hover {
    background-color: #0057b8;
    color: white;
}
```

Why it fails: `:hover` is triggered by mouse movement only. A keyboard user tabbing to the button sees no visual change at all. Hover styles and focus styles must be defined separately.

Fix: Always define `:focus-visible` alongside `:hover`.
```css
button:hover {
    background-color: #0057b8;
    color: white;
}

button:focus-visible {
    outline: 3px solid #005fcc;
    outline-offset: 2px;
}
```

---

## Case 6: `:focus` vs `:focus-visible`

`:focus` fires for all focus events — keyboard Tab and mouse click.  
`:focus-visible` fires only when the browser determines keyboard navigation is being used.

```css
/* Applies to ALL focus — keyboard and mouse */
*:focus {
    outline: 3px solid #005fcc;
    outline-offset: 2px;
}

/* Restore ring for keyboard focus */
*:focus-visible {
    outline: 3px solid #005fcc;
    outline-offset: 2px;
}
```

Why this matters: Mouse users typically do not need or expect a focus ring when clicking a button. Keyboard users always need it. `:focus-visible` lets you satisfy both without removing the ring for keyboard users.

Important: `:focus-visible` alone without a `:focus` fallback — older browsers that do not support `:focus-visible` will show no focus ring at all for keyboard users.

---

## Key Learnings

- Skip link must be the very first element in `body` — before any navigation
- Skip link must be hidden with `position: absolute` and negative `top` — never `display:none`
- `main` needs `tabindex="-1"` to correctly receive programmatic focus from skip link
- Never use `outline: none` without an explicit replacement focus style
- `:focus-visible` targets keyboard focus only — use it alongside `:focus` for full browser support
- Focus indicators must be visible to sighted keyboard users — NVDA announcing an element is not enough

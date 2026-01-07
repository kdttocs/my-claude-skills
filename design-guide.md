---
name: design-guide
description: Ensures all UI components look modern and professional with clean, minimal design. Use this skill when building ANY user interface - React components, HTML pages, dashboards, forms, buttons, cards, modals, or any web UI element. Triggers on requests to build, create, style, or design UI components, web pages, interfaces, dashboards, or frontend elements.
---

# Design Guide

Apply these principles to every UI component. Modern, professional design that feels intentional.

## Core Principles

### Visual Foundation
- **Clean and minimal**: Generous white space, not cluttered. Let elements breathe.
- **Neutral palette**: Grays and off-whites as base. ONE accent color, used sparingly.
- **No generic gradients**: Absolutely no purple/blue gradients. Solid colors or very subtle gradients only.

### Spacing (8px Grid)
Use consistent spacing based on 8px increments:
- `8px` - tight spacing (icon gaps, inline elements)
- `16px` - standard spacing (between related items)
- `24px` - section spacing (form field gaps)
- `32px` - larger gaps (between card content sections)
- `48px` - generous spacing (section dividers)
- `64px` - major spacing (page sections)

### Typography
- **Hierarchy**: Clear distinction between headings, body, captions
- **Body text**: 16px minimum - never smaller for readable content
- **Max 2 fonts**: One for headings (optional), one for body
- **Line height**: 1.5 for body text, 1.2-1.3 for headings

### Visual Depth
- **Subtle shadows**: `box-shadow: 0 1px 3px rgba(0,0,0,0.1)` for light elevation
- **Medium shadows**: `box-shadow: 0 4px 6px rgba(0,0,0,0.1)` for cards/modals
- **Rounded corners**: Use `4px` to `8px`. Not everything needs rounding.
- **Borders OR shadows**: Pick one per element, not both.

### Interactivity
Every interactive element needs clear states:
- **Default**: Base appearance
- **Hover**: Subtle feedback (darken 5-10%, slight shadow lift)
- **Active/Pressed**: Pressed appearance (darken further, reduce shadow)
- **Disabled**: Reduced opacity (0.5-0.6), `cursor: not-allowed`
- **Focus**: Visible ring for accessibility (`outline` or `ring`)

### Responsive (Mobile-First)
- Design for mobile viewport first
- Scale up for larger screens
- Touch targets minimum 44px
- Stack elements vertically on mobile

## Component Patterns

### Buttons
```css
/* Primary button */
.btn-primary {
  padding: 12px 24px;
  background: var(--accent-color);
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  font-weight: 500;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  cursor: pointer;
  transition: all 0.15s ease;
}
.btn-primary:hover {
  filter: brightness(0.95);
  box-shadow: 0 2px 4px rgba(0,0,0,0.12);
}
.btn-primary:active {
  filter: brightness(0.9);
  box-shadow: 0 1px 2px rgba(0,0,0,0.1);
}
.btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

/* Secondary button */
.btn-secondary {
  padding: 12px 24px;
  background: white;
  color: var(--gray-700);
  border: 1px solid var(--gray-300);
  border-radius: 6px;
  /* ... similar states */
}
```

### Cards
```css
/* Clean card - shadow approach */
.card {
  background: white;
  border-radius: 8px;
  padding: 24px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

/* Alternative - border approach (pick one, not both) */
.card-bordered {
  background: white;
  border-radius: 8px;
  padding: 24px;
  border: 1px solid var(--gray-200);
}
```

### Forms
```css
.form-group {
  margin-bottom: 24px;
}
.form-label {
  display: block;
  margin-bottom: 8px;
  font-size: 14px;
  font-weight: 500;
  color: var(--gray-700);
}
.form-input {
  width: 100%;
  padding: 12px 16px;
  font-size: 16px;
  border: 1px solid var(--gray-300);
  border-radius: 6px;
  transition: border-color 0.15s ease;
}
.form-input:focus {
  outline: none;
  border-color: var(--accent-color);
  box-shadow: 0 0 0 3px rgba(var(--accent-rgb), 0.1);
}
.form-input.error {
  border-color: var(--error-color);
}
.form-error {
  margin-top: 8px;
  font-size: 14px;
  color: var(--error-color);
}
```

## Color Variables Template
```css
:root {
  /* Base neutrals */
  --gray-50: #f9fafb;
  --gray-100: #f3f4f6;
  --gray-200: #e5e7eb;
  --gray-300: #d1d5db;
  --gray-400: #9ca3af;
  --gray-500: #6b7280;
  --gray-600: #4b5563;
  --gray-700: #374151;
  --gray-800: #1f2937;
  --gray-900: #111827;
  
  /* Single accent - customize per project */
  --accent-color: #2563eb;  /* Example: blue */
  --accent-rgb: 37, 99, 235;
  
  /* Semantic */
  --error-color: #dc2626;
  --success-color: #16a34a;
  --warning-color: #d97706;
  
  /* Backgrounds */
  --bg-primary: #ffffff;
  --bg-secondary: var(--gray-50);
}
```

## Anti-Patterns (Never Do)

**Visual Chaos**
- Rainbow gradients or multiple bright colors
- Every element a different color
- Purple/blue AI-slop gradients

**Poor Typography**
- Text smaller than 16px for body content
- Mixing 3+ fonts
- Inconsistent heading sizes

**Bad Spacing**
- Arbitrary pixel values (13px, 17px, 22px)
- Elements crammed together
- Inconsistent gaps between similar items

**Overdesign**
- Both borders AND shadows on same element
- Excessive rounded corners on everything
- Heavy drop shadows everywhere

**Missing States**
- No hover feedback on clickable elements
- No focus indicators for accessibility
- No disabled styling

## Quick Checklist

Before finishing any UI:
- [ ] Spacing follows 8px grid
- [ ] Only ONE accent color used
- [ ] Body text is 16px+
- [ ] All interactive elements have hover/active/disabled states
- [ ] Cards use border OR shadow, not both
- [ ] Shadows are subtle, not heavy
- [ ] Works on mobile viewport

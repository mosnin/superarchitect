# Design Principles — SuperArchitect OS

**Version:** 1.0
**Owner:** Designer Agent (Team 7)
**Last Updated:** 2026-03-29

---

## Purpose

These principles guide every design decision within the SuperArchitect OS. They apply to user interfaces, API design, system interactions, documentation, and any surface where a human interacts with the system. Design is not decoration — it is how the system communicates its structure and intent to its users.

---

## 1. Core Design Principles

### 1.1 Clarity Over Cleverness

Every interface element must communicate its purpose immediately. If a user has to think about what something does, the design has failed. Labels are clear. Actions are obvious. States are visible. Jargon is eliminated unless the audience is domain experts who expect it.

**Application:**
- Button labels describe the action: "Create Order" not "Submit"
- Error messages explain what happened and what to do: "Email address is invalid. Enter a valid email like name@example.com" not "Validation error"
- Navigation labels match the user's mental model, not the system's architecture
- Empty states explain what belongs here and how to get started

### 1.2 Consistency Reduces Cognitive Load

Identical things look identical. Similar things look similar. Different things look different. Users learn patterns once and apply them everywhere. Breaking consistency forces the user to relearn — which is expensive and frustrating.

**Application:**
- Design system enforced across all interfaces
- Same action in the same position on every screen
- Consistent color coding: red=destructive, green=success, yellow=warning
- Consistent interaction patterns: inline editing works the same everywhere
- Consistent terminology: one word for one concept throughout the system

### 1.3 Progressive Disclosure

Show only what is needed at each step. Complexity exists but is revealed gradually. The interface starts simple and deepens as the user needs more. Advanced features do not clutter the basic experience.

**Application:**
- Default settings that work for 80% of users; advanced settings behind "More options"
- Wizard flows for complex multi-step processes
- Tooltips and contextual help for complex fields
- Collapsed sections for infrequently used information
- Smart defaults that reduce the number of decisions a user must make

### 1.4 Feedback is Immediate and Honest

Every user action produces visible feedback. The system never silently succeeds, silently fails, or leaves the user wondering what happened. Loading states, success confirmations, error messages, and progress indicators are mandatory.

**Application:**
- Button state changes on click (loading spinner or disabled state)
- Success messages confirm what happened: "Order #1234 created"
- Error messages appear inline, next to the field that caused them
- Progress bars for operations that take more than 1 second
- Skeleton screens for loading states (not blank pages)
- Optimistic UI updates for low-risk actions (with rollback on failure)

### 1.5 Forgiveness Over Prevention

Users make mistakes. The system should make mistakes easy to undo rather than making actions hard to take. Undo is better than "Are you sure?" dialogs (which users learn to click through without reading).

**Application:**
- Soft delete with undo: "Item deleted. Undo" instead of "Are you sure you want to delete?"
- Draft auto-save so work is never lost
- Undo/redo for all content editing
- Confirmation only for truly irreversible, high-impact actions
- Grace periods: "Email scheduled. Cancel within 30 seconds"

### 1.6 Accessibility is Not Optional

Accessible design is good design. Systems must be usable by everyone, including users with visual, motor, auditory, and cognitive disabilities. Accessibility is a requirement, not a nice-to-have.

**Standards:**
- WCAG 2.1 AA compliance minimum
- Keyboard navigation for all interactive elements
- Screen reader compatibility (semantic HTML, ARIA labels)
- Color contrast ratio >= 4.5:1 for normal text, >= 3:1 for large text
- Focus indicators visible on all interactive elements
- No information conveyed by color alone
- Text alternatives for all non-text content
- Captions for video and audio content
- Reduced motion support for users with vestibular disorders

### 1.7 Performance is a Design Constraint

A beautiful interface that takes 5 seconds to load is a bad interface. Perceived performance matters as much as actual performance. Design for speed: optimize critical rendering path, use skeleton screens, lazy load below-the-fold content.

**Performance Budgets:**
| Metric | Target |
|--------|--------|
| First Contentful Paint | < 1.0s |
| Largest Contentful Paint | < 2.5s |
| Cumulative Layout Shift | < 0.1 |
| First Input Delay | < 100ms |
| Time to Interactive | < 3.0s |
| Total page weight | < 500KB (initial load) |

---

## 2. Information Architecture Principles

### 2.1 Organization
- Group related items together (proximity principle)
- Create clear hierarchies: primary > secondary > tertiary navigation
- Use the user's language for categories, not internal terminology
- Limit navigation items to 7 +/- 2 at each level (Miller's Law)
- Search as a first-class navigation mechanism for large systems

### 2.2 Navigation
- Users should always know where they are (breadcrumbs, active states)
- Users should always know where they can go (visible navigation)
- Users should always be able to get back (back button, home link)
- Deep links: every meaningful state should have a shareable URL
- Navigation structure reflects user tasks, not system architecture

### 2.3 Content Hierarchy
- Most important content is most prominent (visual hierarchy)
- Headings create scannable structure
- White space separates logical groups
- Call-to-action buttons are visually distinct from secondary actions
- Dense information uses tables; sparse information uses cards/lists

---

## 3. Component Design Principles

### 3.1 Design System Requirements
Every system built by the SuperArchitect OS must have a design system that includes:

**Foundation:**
- Color palette with semantic color names (primary, secondary, success, warning, error)
- Typography scale (heading levels, body text, captions, labels)
- Spacing scale (consistent spacing values: 4px, 8px, 12px, 16px, 24px, 32px, 48px, 64px)
- Icon set with consistent style, size, and meaning
- Grid system for layout consistency

**Components (minimum set):**
- Button (primary, secondary, tertiary, destructive, disabled, loading states)
- Input fields (text, number, date, select, multi-select, textarea, file upload)
- Form validation (inline errors, success states, helper text)
- Table (sortable, filterable, paginated, selectable, responsive)
- Modal/Dialog (confirmation, form, informational)
- Toast/Notification (success, error, warning, info)
- Navigation (top bar, sidebar, breadcrumbs, tabs)
- Card (content container with consistent padding and borders)
- Loading states (spinner, skeleton, progress bar)
- Empty states (illustrations + call-to-action)

### 3.2 Component Rules
- Every component documents its props, states, and usage guidelines
- Every component has responsive behavior defined
- Every component has accessibility annotations
- Components are composable: complex UIs are built from simple components
- State management is predictable: components reflect their data consistently

---

## 4. Interaction Design Principles

### 4.1 Direct Manipulation
- Objects can be directly acted upon (click to edit, drag to reorder)
- The result of an action is immediately visible
- Actions are reversible where possible

### 4.2 Responsive Design
- Mobile-first design: start with the smallest screen, enhance for larger
- Breakpoints: 320px (mobile), 768px (tablet), 1024px (laptop), 1440px (desktop)
- Touch targets: minimum 44x44px on mobile
- Content reflows gracefully across breakpoints (no horizontal scrolling)
- Critical actions accessible on all screen sizes

### 4.3 Animation and Motion
- Animation serves a purpose: guides attention, shows relationships, provides feedback
- Duration: micro-interactions 100-300ms, transitions 200-500ms
- Easing: ease-out for entrances, ease-in for exits, ease-in-out for transitions
- Reduce motion: respect `prefers-reduced-motion` media query
- No animation that blocks user interaction

### 4.4 Error Prevention and Recovery
- Inline validation as users fill forms (after field blur, not on every keystroke)
- Disable submit button until form is valid (with clear indication of what is missing)
- Auto-save for long forms (with visible "Saved" indicator)
- Confirmation for destructive actions (delete, overwrite, send)
- Clear error recovery path: what went wrong, how to fix it

---

## 5. Data Visualization Principles

### 5.1 Chart Selection
| Data Relationship | Chart Type |
|------------------|------------|
| Comparison | Bar chart (vertical or horizontal) |
| Trend over time | Line chart |
| Part of whole | Pie chart (max 5 segments) or stacked bar |
| Distribution | Histogram or box plot |
| Correlation | Scatter plot |
| Geographic | Map |
| Hierarchy | Treemap |

### 5.2 Visualization Rules
- Title every chart with what it shows (not how to read it)
- Label axes clearly with units
- Start Y-axis at zero for bar charts (avoid misleading scale)
- Use color meaningfully (not decoratively)
- Provide data table alternative for accessibility
- Interactive: hover for detail, click to drill down
- Responsive: readable on mobile (simplify or reflow, do not just shrink)

---

## 6. API Design as User Experience

APIs are interfaces too. Developers are users. API design follows the same principles as UI design.

### 6.1 API UX Principles
- Consistent naming conventions (camelCase or snake_case, not both)
- Predictable URL patterns (/resources, /resources/:id, /resources/:id/sub-resources)
- Meaningful HTTP status codes (not everything is 200 or 500)
- Helpful error messages with actionable guidance
- Pagination, filtering, and sorting on collection endpoints
- API documentation that is always up-to-date (generated from code)
- SDK/client libraries for common languages

### 6.2 Developer Experience
- Time to first API call < 5 minutes
- Authentication setup documented with copy-paste examples
- Interactive API explorer (Swagger UI, Redoc)
- Sandbox environment for safe experimentation
- Changelog for API versions with migration guides

---

## 7. Design Review Checklist

Before any interface ships:

- [ ] Meets all functional requirements from Product spec
- [ ] Consistent with design system (no one-off styles)
- [ ] Keyboard navigable (Tab, Enter, Escape, Arrow keys)
- [ ] Screen reader tested (VoiceOver, NVDA)
- [ ] Color contrast meets WCAG AA
- [ ] Responsive across all breakpoints
- [ ] Loading states implemented
- [ ] Empty states implemented
- [ ] Error states implemented
- [ ] Works with realistic data (not just placeholder content)
- [ ] Performance within budget (LCP < 2.5s)
- [ ] Animations respect reduced-motion preference
- [ ] Touch targets >= 44px on mobile

---

*Design is not how it looks. Design is how it works. Every pixel, every word, every interaction is a decision that affects the user's ability to accomplish their goal. Make every decision with the user's success as the objective.*

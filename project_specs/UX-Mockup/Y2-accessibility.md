---

## Accessibility Notes

TaskFlow targets **WCAG 2.1 Level AA** compliance. This is specified in the FRD non-functional constraints ("status badges meet WCAG AA color contrast") and PRD ("Form inputs have labels; status badges have sufficient color contrast (WCAG AA minimum)").

---

### Color Contrast

All text must meet WCAG AA minimum contrast ratios:
- **Normal text (< 18pt / < 14pt bold):** minimum 4.5:1
- **Large text (≥ 18pt / ≥ 14pt bold):** minimum 3:1
- **UI components and graphical objects:** minimum 3:1

**Status Badge Requirements (critical per FRD F01):**

| Badge | Background | Text | Minimum Ratio | Must verify |
|-------|-----------|------|---------------|-------------|
| To Do | `#E5E7EB` (light gray) | `#374151` (dark gray) | 4.5:1 | ✅ ~8.6:1 |
| In Progress | `#DBEAFE` (light blue) | `#1D4ED8` (dark blue) | 4.5:1 | ✅ ~5.9:1 |
| Done | `#D1FAE5` (light green) | `#065F46` (dark green) | 4.5:1 | ✅ ~7.7:1 |

> These are reference values. Implementation team must run final contrast checks using a tool such as [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) before shipping.

**Do not rely on color alone** to convey status — the badge text ("To Do", "In Progress", "Done") also communicates the status for colorblind users.

---

### Form Inputs and Labels

Every form input must have an associated `<label>` element. This is a hard requirement from the FRD (F00, F02 §Accessibility).

**Correct pattern:**
```html
<label for="title">Title <span aria-hidden="true">*</span>
  <span class="sr-only">(required)</span>
</label>
<input type="text" id="title" name="title" required aria-required="true">
```

**Optional fields:**
```html
<label for="description">
  Description <span class="visually-hidden">(optional)</span>
</label>
<textarea id="description" name="description"></textarea>
```

**Rules:**
- No `placeholder`-only labeling (placeholder text disappears on focus and has insufficient contrast).
- Required fields: use `required` attribute + `aria-required="true"`.
- Optional fields: "(optional)" in the label text.

---

### Validation Error Accessibility

**When validation errors occur (422 form re-render):**

1. **Error summary** at top of form with `role="alert"` so screen readers announce it immediately:
   ```html
   <div role="alert" class="error-summary">
     Please fix the errors below before saving.
   </div>
   ```

2. **Field-level errors** linked via `aria-describedby`:
   ```html
   <input id="title" aria-invalid="true" aria-describedby="title-error">
   <span id="title-error" class="error-message">Title is required.</span>
   ```

3. **Page focus** on form re-render: move focus to the error summary element so keyboard users immediately know an error occurred.

---

### Keyboard Navigation

All interactive elements must be reachable and operable via keyboard alone:

| Element | Keyboard Behavior |
|---------|-----------------|
| Navigation links (filter tabs, "New Task", "← Back") | Tab to focus; Enter to activate |
| Form inputs | Tab order: Title → Description → Status → Due Date → Submit → Cancel |
| Submit buttons | Enter key activates focused button |
| Delete Confirmation dialog | Escape closes dialog (Cancel path); Tab cycles between Cancel and Delete Task buttons; Enter activates focused button |
| Delete confirmation default focus | **Cancel button** receives focus when dialog opens (prevents accidental Enter-to-delete) |
| Filter tabs | Tab to navigate between tabs; Enter to activate |

**Focus management for dialog:**
- When the delete confirmation dialog opens, focus is moved into the dialog (Cancel button).
- When the dialog closes (cancel), focus returns to the "Delete" trigger element on the task list.
- Focus must not escape the dialog while it is open (focus trap).

---

### Screen Reader Considerations

**Task List:**
- Each task row should use semantic list markup (`<ul>` / `<li>`) or a `<table>` with proper headers — not `<div>` soup.
- If using a `<table>`: `<th scope="col">` for column headers (Title, Status, Due Date, Actions).
- Status badges: plain text inside the badge is sufficient; no additional ARIA needed if text is present.
- "Edit" and "Delete" action links in each row must include the task title in their accessible label to distinguish them:
  ```html
  <a href="/tasks/42/edit" aria-label="Edit Fix API endpoint timeout">Edit</a>
  <button aria-label="Delete Fix API endpoint timeout">Delete</button>
  ```
  Without this, a screen reader user hears "Edit, Edit, Edit, Delete, Delete, Delete" with no context.

**Empty States:**
- Empty state message should be in a `<p>` inside the list container (or `<td colspan>` if using a table) so it's in the correct document flow.

**Filter Tabs:**
- Use `<nav aria-label="Filter tasks by status">` wrapper.
- Active tab: `aria-current="page"` on the active `<a>` element.

**Page Titles:**
- Unique `<title>` for each page:
  - Task List: "Tasks — TaskFlow"
  - New Task: "New Task — TaskFlow"
  - Edit Task: "Edit: [task title] — TaskFlow"
  - Error: "404 Not Found — TaskFlow"

---

### ARIA Landmark Roles

Use semantic HTML or ARIA landmarks for screen reader navigation:

| Region | Element / Role |
|--------|---------------|
| App header + "New Task" button | `<header>` |
| Filter nav | `<nav aria-label="Filter tasks by status">` |
| Task list | `<main>` wrapping the list content |
| Form | `<main>` > `<form>` |
| Footer (if present) | `<footer>` |
| Delete dialog | `<dialog>` element with `aria-labelledby` pointing to dialog heading |

---

### Summary Checklist

- [ ] All status badge text/background combinations meet ≥ 4.5:1 contrast ratio
- [ ] Every form input has an associated `<label>` element
- [ ] Required fields marked with `required` + `aria-required="true"`
- [ ] Validation errors use `role="alert"` for summary and `aria-describedby` for field linkage
- [ ] `aria-invalid="true"` set on invalid fields
- [ ] Delete confirmation focuses Cancel button on open; traps focus in dialog
- [ ] Row-level Edit/Delete links include task title in `aria-label`
- [ ] Filter tabs use `aria-current="page"` for active tab
- [ ] Page `<title>` is unique per screen
- [ ] Keyboard-navigable in full Tab order without mouse
- [ ] No color-only status communication (badge text always present)
- [ ] Touch targets ≥ 44px on mobile

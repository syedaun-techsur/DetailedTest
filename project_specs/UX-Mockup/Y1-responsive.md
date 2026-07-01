---

## Responsive Considerations

TaskFlow is an **internal web tool** used primarily on desktop browsers by team members at their workstations. The PRD explicitly states "web-first; browser is sufficient for internal tooling." Mobile native app is out of scope. However, basic responsive behavior ensures the app is usable on a tablet or laptop at varying window sizes.

---

### Breakpoints

| Breakpoint | Width | Target |
|-----------|-------|--------|
| Desktop | > 1024px | Primary: team members at workstations |
| Tablet | 768px – 1024px | Secondary: laptops, iPad landscape |
| Mobile | < 768px | Tertiary: mobile browser (functional but not optimized) |

---

### Desktop (> 1024px)

**Task List:**
- Full-width layout with generous padding.
- Task rows display all columns side-by-side: title (links), status badge, due date, description preview, action buttons.
- Filter tabs in a single horizontal row above the list.
- "New Task" button in the top-right of the header.

```
┌─────────────────────────────────────────────────────────────────┐
│  TaskFlow                                    [ + New Task ]     │
├─────────────────────────────────────────────────────────────────┤
│  [ All ]  [ To Do ]  [ In Progress ]  [ Done ]                  │
├─────────────────────────────────────────────────────────────────┤
│  Fix API endpoint timeout    [In Progress]  Jul 15  Investigate │
│  Update README               [To Do]        —       (no desc)   │
│  Deploy to staging           [Done]         Jul 01  Deployed #42│
└─────────────────────────────────────────────────────────────────┘
```

**Create / Edit Forms:**
- Centered single-column form, max-width ~600px.
- Label above input pattern (not inline labels) for clarity.
- All form fields visible without scrolling on typical 768px+ viewport height.

---

### Tablet (768px – 1024px)

**Task List:**
- Same layout as desktop; may require minor padding reduction.
- Action buttons (Edit, Delete) may stack vertically per row if horizontal space is tight.
- Filter tabs still in a single row — 4 short labels fit comfortably.

**Create / Edit Forms:**
- No layout change from desktop; single-column form is naturally responsive.
- Touch targets for dropdowns and date pickers should be ≥ 44px height (WCAG 2.1 touch target guideline).

---

### Mobile (< 768px)

**Task List:**
- Stack layout: title on its own line, status badge and due date on the next line, description preview below.
- Action buttons (Edit, Delete) either stack below description or collapse into a "..." overflow menu.
- Filter tabs may wrap to two rows or become a dropdown select if all 4 don't fit.
- "New Task" button may move below the filter row if header space is constrained.

```
┌──────────────────────────────────────────┐
│  TaskFlow            [ + New Task ]      │
├──────────────────────────────────────────┤
│  [All] [To Do] [In Progress] [Done]      │
├──────────────────────────────────────────┤
│  Fix API endpoint timeout                │
│  [In Progress]  •  Jul 15, 2026          │
│  Investigate why /tasks endpoint...      │
│                       [Edit] [Delete]    │
├──────────────────────────────────────────┤
│  Update README                           │
│  [To Do]  •  —                           │
│                       [Edit] [Delete]    │
└──────────────────────────────────────────┘
```

**Create / Edit Forms:**
- Full-width inputs (100% width).
- Adequate spacing between fields for touch interaction.
- Date input uses native mobile date picker (browser-native `type="date"`).

---

### Key Responsive Principles

1. **No horizontal scrolling** at any breakpoint — all content adapts to available width.
2. **Filter tabs** are the most space-constrained element at mobile — consider a `<select>` dropdown fallback at < 480px if labels overflow.
3. **Touch targets** for buttons and links: minimum 44×44px per WCAG 2.1 Success Criterion 2.5.5 (Level AAA) and iOS/Android guidelines.
4. **Font sizes** remain readable at all breakpoints — base font ≥ 16px on mobile to prevent iOS auto-zoom on inputs.
5. **Status badges** scale with text — pill shape with padding remains visible at all sizes.

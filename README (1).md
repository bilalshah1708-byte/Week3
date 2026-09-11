# Task Rail

A lightweight, dependency-free kanban-style task board. Three panels — **Backlog**, **In Progress**, **Done** — with state held in plain JavaScript and updated live from DOM events.

## Features

- Add a task to any panel via its inline form
- Advance a task to the next panel with one click
- Delete a task
- Drag and drop a task card between panels
- Panel counts and an overall "open" tally update automatically
- Responsive layout (stacks to a single column on narrow screens)
- Respects `prefers-reduced-motion`

## Files

- `task-panels.html` — the entire app: markup, styling, and behavior in one self-contained file

## Running it

No build step or server required. Open `task-panels.html` directly in any modern browser.

## How it works

All tasks live in a single in-memory array:

```js
let tasks = [
  { id, text, status } // status is 'backlog' | 'progress' | 'done'
];
```

A single `render()` function clears the board and rebuilds it from that array, so the DOM always reflects current state — there's no manual DOM patching to keep in sync.

State changes are driven entirely by delegated DOM event listeners on the board container:

| Event | Trigger | Effect |
|---|---|---|
| `submit` | Submitting a panel's add form | `addTask()` pushes a new task, then re-renders |
| `click` | Clicking a card's `›` button | `advanceTask()` moves the task to the next status |
| `click` | Clicking a card's `×` button | `deleteTask()` removes the task |
| `dragstart` / `dragend` | Picking up / releasing a card | Toggles the `.dragging` class for visual feedback |
| `dragover` | Dragging over a panel | Highlights the panel as a drop target |
| `drop` | Dropping a card on a panel | `moveTask()` updates the task's status, then re-renders |

Because listeners are attached once to the board container (event delegation) rather than to individual cards, newly added or moved cards work immediately without re-binding handlers.

## Customizing

- **Add a panel**: add an entry to the `panels` array and a corresponding entry in `nextStatus`.
- **Change the advance order**: edit the `nextStatus` map.
- **Persist data**: the app currently keeps state in memory only (it's a static HTML file, so there's no backend and no browser storage wired up) — swap the mutator functions to call your own storage or API if you need persistence.

## Notes

- Input is sanitized via `escapeHtml()` before being inserted into the DOM to avoid HTML injection from task text.
- Card IDs are a simple incrementing counter (`seq`), not stable across page reloads since there's no persistence layer.

# Hosting keycraft on the web

Read this before marking up the jump menu. Then check current practice with the modern-web-guidance skill (popover, listbox, keyboard) and follow what it says now. The notes below are the keycraft constraints that guidance is easy to miss.

The engine stays a pure function. This file is the host.

- A jump menu is a list a person filters and lands from. Use `role="listbox"` and `role="option"` with `aria-activedescendant`. `role="menu"` fits an action menu that keeps one tab stop, which is a different widget.
- Give the menu a top layer (`popover="manual"`) and show it when it opens. Do not set `role="dialog"` or a dialog class the app already treats as a gate. A dialog gate would swallow the menu's own keys.
- Listen for `keydown` on `window` in the capture phase. Call `preventDefault` and `stopPropagation` only when the engine action is not pass.
- The footer is text, the menu hint. The engine holds the query. An empty query shows a hint such as "type a name".
- Activate a clicked row with that row's index. A pointer down outside the menu closes it (back one layer).
- On `focusin`, a text field outside the terminal moves the engine to insert, closes the menu, and keeps the cheat. Leaving that field for a non-field returns to verbs. Terminal focus leaves the mode as it was.
- A key whose target is inside the terminal passes, including Esc.
- Composition is `event.isComposing`. Also treat `keyCode` 229 as composition when the key is Enter. Some keyboards report 229 for ordinary characters, so 229 is not composition on every key.
- Reset the row's height, border, and background when the app's button style would clip it. Color the menu with the app's surface, text, border, and accent tokens so each theme stays readable.
- Fetch a large scope when that scope opens. Drop a response that is older than the newest request for that scope.
- Mount the letter host on the desktop app surface only. A phone shell that does not use this keymap does not get the listener.
- The chord listener sees the same `keydown` after the letter engine passes. Read `code`, `shiftKey`, `altKey`, and `metaKey || ctrlKey`. Call `preventDefault` only for a chord the host runs.
- The card uses `popover="manual"` and no dialog role, or the letter gate treats it as a dialog and swallows verbs. A pointer down outside the card closes it.

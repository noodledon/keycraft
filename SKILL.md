---
name: keycraft
description: >
  Keycraft decides a keymap for any app: which repeated work deserves a
  shortcut, and whether that shortcut is a letter, a chord, or a jump
  menu. Use when the user wants shortcuts, vim-style letters, a Fizzy
  menu, a jump menu, Cmd or Ctrl bindings, or runs /keycraft.
---

# Keycraft

Decide the keymap, then bind it. The method is the same in every app. The keys follow from what that app repeats.

**Weight** is how scarce a key is. A home-row letter is the heaviest: spend one on the action this screen repeats. The **door** is one heavy key for the whole app, and it opens the **jump menu**. A **chord** is lighter because it takes a modifier, and it is the binding that still works while someone is typing. A rare or easy-to-miss action keeps the pointer.

**?** shows the letter line and the **card**. **Esc** closes one **layer**.

## Steps

1. **Name the work.** For each screen, name the noun a person lands on, the thing they select, and the field they type into, in the app's own words. Done when every screen has those three, or is marked view-only.

2. **Find what is slow.** List the actions a person repeats: reaching a named thing, acting on the current selection, changing view, and anything they do with a field focused. Done when each repeated action is one line with its screen and whether a text field is focused when it happens.

3. **Pick one mechanism.** Apply [Decide](#decide) to every line from step 2. Done when each line is a door, a scope, a verb, a two-beat, a chord, or the pointer, and no line has two primary mechanisms.

4. **Assign the key.** A verb takes the free key in the matching [Verb catalog](#verb-catalog) row. A chord takes the first free binding in the matching [Chords](#chords) row. Keep a binding the app already uses. When the platform keeps a binding (a browser New Window, Close Window, or address bar), use the next binding in that row and write the one that works. A chosen mechanism with no catalog row gets the heaviest free key of its class. Done when the letter line and the card name exactly the keys the host will run, in the app's words, and name nothing step 3 left on the pointer.

5. **Write the engines.** Implement [Letters](#letters) and [Chord engine](#chord-engine) as pure functions of state and key. The host is not in these tests. Done when a test fails if a letter rule changes or a bound chord's key, Shift, or Alt changes, and an unbound combination returns pass.

6. **Host both layers.** The host focuses, fetches, and navigates. The engines decide. Letters listen first and pass any key that carries Meta, Ctrl, or Alt. The chord function then runs on that same key, including inside a field, a terminal, and a dialog. On the web, read [references/web.md](references/web.md) before the widget or the listener. When the app is a board of cards, or the user names Fizzy, read [references/fizzy.md](references/fizzy.md) before the board keys. Done when the door opens on every screen outside a gate, Enter lands, a text field receives verb letters as characters, a chord from the card still runs inside that field, unshifted Esc leaves the field, and a terminal and an IME composition keep their letter keys.

7. **Exercise what you bound.** In the running app, perform every verb on the letter line and every row on the card. Repeat two chords from inside a text field, and one from a terminal when the app has one. Filter, land, and confirm a platform-owned binding still does the platform's action. Done when each of those happened and the card matches the keys that worked.

## Decide

Use the first row that matches the slowness.

| The slowness | Mechanism |
| --- | --- |
| A named thing, reached from more than this screen | Door. One jump menu for the app. |
| That same menu, already narrowed to one noun | Scope. A second letter opens files, people, or boards. |
| An action on the current selection, many times a minute, field not focused | Verb. One key, this screen only. |
| A visible list, then an action on the highlighted row | Two-beat. Arrows walk, then one verb. One key per row spends the home row on the list. |
| The action happens while a field is focused, or people already use this combo for it | Chord. |
| Two related reaches (this document or the whole corpus, the primary column or a second one) | The same chord with Shift. |
| Rare, or destructive with no confirm | Pointer. It stays a click. |

One line gets one primary mechanism. A letter is a second path only when this host cannot swallow the chord.

## Terms

- **Door.** One key, free of chords, that opens the jump menu on every screen. Default `J` when `J` is free.
- **Jump menu.** The list behind the door. An empty query shows the app's order, recents first when the app has them, capped at 50. Typing **filters**. **Enter** **lands**. A single match stays highlighted until Enter.
- **Land.** The highlighted row becomes the current place or runs that row's action, then the menu closes.
- **Verb.** One key, one action, for this screen's noun. The same letter may be a different verb on another screen.
- **Walk.** Move the highlight. Arrows always walk. On a vertical list, `j` and `k` walk too. Screen walk stops at the ends. Menu walk wraps.
- **Filter.** Characters narrow the jump menu. Match case-insensitively, stable, in this order: exact label, label prefix, haystack prefix, then substring of the label or the haystack. Cap 50. `/` starts a filter with an empty query so a name can begin with a verb letter. Once filtering, verb letters append. Arrows still walk. Backspace edits; an empty query leaves filtering off.
- **Gate.** A place that already owns the letter keys. Letters pass through. Gates: a text field, a terminal, an IME composition, an open dialog, and any key that carries Meta, Ctrl, or Alt.
- **Chord.** Meta, Ctrl, or Alt, with or without Shift. It runs inside a gate. Meta and Ctrl are the same binding unless the host can tell them apart. Match the physical key (`KeyB`, `Digit1`, `Comma`), never the character Shift or Alt would type.
- **Shifted verb.** Shift plus a letter, with no Meta, Ctrl, or Alt. It follows the letter gates. Use it when the bare letter is taken.
- **Layer.** One thing Esc closes: the open menu, or the card when no menu is open, or a text field. The letter line stays.
- **Card.** The list `?` opens. One row per chord the host runs: the keys, then the action in the app's words. Verbs still run while it shows. It is not a dialog.
- **Insert.** A verb that focuses the composer or this noun's field. Unshifted Esc leaves the field and returns to verbs.

## Verb catalog

A default key for a verb [Decide](#decide) already chose. Skip a row this app has no noun for.

| Verb | Key to try | The noun |
| --- | --- | --- |
| Door | `J` | Places and named things. Every app that has more than one place. |
| Search | `K` | A corpus larger than, or different from, the door's list. |
| Areas | `1` `2` `3` | A handful of top-level places. |
| Visible items | `1`–`9` | An ordered list on this screen. The digit matches visual order. |
| Walk | arrows, `j` `k` | A selection or a vertical list. |
| Neighbor | `[` `]` | A next and previous surface, column, or page. |
| Open | Enter | A selection that opens. |
| Create | `C` | A noun this screen can add. |
| Edit | `E` | A selection with a field. |
| Done / defer | `]` and `[` when neighbor is free, else two free letters | A selection with those states. |
| Assign to me | `M` | A selection with an assignee. |
| Tag | `T` | A selection with tags. |
| Focus filter | `F` | A screen that narrows in place. |
| Insert | `i` | A composer or a primary field. |
| Back | Esc, and `←` when history exists | Esc always. `←` when the app has a back stack. |
| Cheat | `?` | The letter line and the card. |

Global keys (door, Esc, `?`, and search when chosen) mean the same thing on every screen. Every other verb belongs to one screen.

## Chords

A default binding for a chord [Decide](#decide) already chose. The first binding is the one to try. Skip a row this app has no noun for.

| Action | Bindings to try | The noun |
| --- | --- | --- |
| Command palette | ⌘K, ⌘⇧P | A list of commands. |
| Go to a named record | ⌘P | Files, pages, people, or records too many to scan. |
| Find in this document | ⌘F | The open document, thread, or record. |
| Find across the corpus | ⌘⇧F | Search past the open document. |
| Primary column | ⌘B | A sidebar or primary list that can hide. |
| Secondary column | ⌥⌘B, ⌘⇧E | A second column (files, details, inspector). |
| Other face of this noun | ⌘` | Two views of the same noun. |
| New | ⌘N, then the Create letter | A screen that creates its noun. |
| Close | ⌘W | A tab, pane, or document that closes. |
| Save | ⌘S | A noun that saves. |
| Settings | ⌘, | Settings. |
| Often-toggled overlay | ⌘. | An assistant, inspector, or mode. |
| Next / previous | ⌘⇧], ⌘⇧[ | Ordered tabs, pages, or sections. |
| Focus place n | ⌘1–⌘9 | Numbered panes, columns, or accounts. Plain `1`–`9` stay letter verbs. |
| Split | ⌘\\ | A surface that splits. |
| Confirm | ⌘Enter | A primary confirm that is not already the field's Enter. |

Leave undo, redo, cut, copy, paste, and select-all on the platform (⌘Z, ⌘⇧Z, ⌘X, ⌘C, ⌘V, ⌘A). While a rich-text or code field is focused, leave its formatting chords on the field. An app chord on a different key still runs.

The card names each chord the host runs, and the host runs each row on the card.

### Chord engine

Input is the physical key, whether Shift is down, and whether Alt is down. The host has already required Meta or Ctrl. Output is an action, or pass.

Apply the first rule that matches.

1. Alt is down: the row that names this key with Alt and this Shift. Any other Alt combination passes.
2. The platform and formatting bindings above: pass.
3. The row for this key and this Shift. A toggle (show or hide the same thing) whose key is repeating holds, and the action does not run again. A row whose action can open its surface does that when another surface is showing, and the key is swallowed. A row that has no meaning on this surface passes.
4. Anything else: pass.

## Letters

State is the mode (`verbs` or `insert`), the card flag, and either no menu or a menu `{ scope, query, index, filtering }`.

**Pass** lets the app receive the key. **Hold** consumes the key and leaves state unchanged.

Apply the first rule that matches.

1. Composing, terminal, a key with Meta, Ctrl, or Alt, a dialog, or a view that is not the app's main surface: pass.
2. Insert: unshifted Esc leaves insert, closes the menu, and keeps the card. Any other key passes.
3. Menu + Esc: close the menu, keep the card.
4. Menu + Enter: land when the match count is greater than zero, then close the menu. Otherwise hold.
5. Menu + Backspace: edit the query. An empty query turns filtering off and sets the index to 0. Hold when the query is already empty and filtering is off.
6. Menu + arrows: walk, wrapping, even while filtering. Hold when the match count is zero.
7. Menu + `j` or `k`, filtering off: walk the same way. Hold when the match count is zero.
8. Menu + `?`, filtering off: toggle the card. The menu stays.
9. Menu + `/`, filtering off: turn filtering on. The query stays empty.
10. Menu + the door or a scope door, filtering off: retarget that scope, clear the query, index 0.
11. Menu + a digit `1`–`9`, filtering off: move to that row when it exists. Otherwise hold.
12. Menu + any other character key: append it, turn filtering on, index 0.
13. Menu + anything else: hold.
14. Verbs mode + unshifted Esc while the card is open: close the card.
15. Verbs mode: the screen's verbs, the door, search, and `?`. An unbound key passes.

`?` toggles the card and the letter line together and leaves the mode alone. The door opens a menu at scope `all`, query empty, index 0, filtering off. A scope verb opens that scope the same way. Insert sets mode to insert and closes the menu.

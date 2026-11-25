---
name: keycraft
description: >
  Keycraft adds a productive keymap to any app: a door key opens a jump
  menu, each screen has single-letter verbs, and key combinations run the
  app's real actions even inside a text field. Use when the user wants
  vim-style shortcuts, a Fizzy-style keyboard, a jump menu, Cmd or Ctrl
  shortcuts, or a faster keymap, or when they run /keycraft.
---

# Keycraft

Build a keymap a person can use before they memorize it. Two layers:

- **Letters** work outside a gate. One **door** opens a **jump menu**. Each screen has a few **verbs**. **Walk**, then a verb acts on what is highlighted.
- **Chords** are combinations (⌘, Ctrl, Alt, plus Shift when it changes the action). They run the actions the app already has, including while a person is typing.

**Esc** closes one **layer**. **?** shows this screen's letters and its **card** of chords.

Home-row walk (`j` / `k`) is a verb on a vertical list. The door is how a person reaches a named thing. A chord is how they do the thing they would have clicked, without leaving the keyboard.

## Terms

- **Door.** One key, free of chords, that opens the jump menu on every screen. Default `J` when `J` is free.
- **Jump menu.** The list behind the door. An empty query shows the app's order, recents first when the app has them, capped at 50. Typing **filters**. **Enter** **lands**. A single match stays highlighted until Enter.
- **Land.** The highlighted row becomes the current place or runs that row's action, then the menu closes.
- **Verb.** One key, one action, for this screen's noun. The same letter may be a different verb on another screen. The letter line names only this screen's verbs.
- **Walk.** Move the highlight or the selection. Arrows always walk. On a vertical list, `j` and `k` walk too. Screen walk stops at the ends. Menu walk wraps.
- **Two-beat.** Walk to a thing, then press a verb. The verb applies to the selection. Fizzy's board line is the pattern: move with the arrow keys first.
- **Scope.** A verb that opens the same jump menu already narrowed to one noun (files, people, boards).
- **Filter.** Characters narrow the jump menu. Match case-insensitively, stable, in this order: exact label, label prefix, haystack prefix, then substring of the label or the haystack. Cap 50. `/` starts a filter with an empty query so a name can begin with a verb letter. Once filtering, verb letters append. Arrows still walk. Backspace edits; an empty query leaves filtering off.
- **Gate.** A place that already owns the letter keys. Letters pass through. Gates: a text field, a terminal, an IME composition, an open dialog, and any key that carries Meta, Ctrl, or Alt.
- **Chord.** A binding with Meta, Ctrl, or Alt, with or without Shift. It runs inside a gate. Meta and Ctrl are the same binding unless the host can tell them apart. Match the physical key (`KeyB`, `Digit1`, `Comma`), never the character Shift or Alt would type.
- **Shifted verb.** Shift plus a letter, with no Meta, Ctrl, or Alt. It follows the letter gates. Use it when the unshifted letter is taken.
- **Layer.** One thing Esc closes: the open menu, or the card when no menu is open, or a text field. The letter line stays.
- **Card.** The list `?` opens, one row per chord the host implements: the keys, then the action in the app's words. Verbs still run while it shows. It is not a dialog.
- **Insert.** A verb that focuses the composer or this noun's field. Unshifted Esc leaves the field and returns to verbs.

## Steps

1. **Name the nouns.** List the places a person lands, the thing they select on each screen, and the field they type into, in the app's own words. Done when each screen has a noun, or an explicit note that the screen is view-only.

2. **Inventory gates, actions, and the door.** List existing chords, dialogs, terminals, text fields, and the actions a person repeats: create, open, close, save, find, switch place, show or hide a column, settings. Pick the door, the search key, and the screen verbs from keys that are free. Search gets its own key (default `K`) when the corpus is a different list from the door. Done when the door key appears on no chord and in no text-entry path, and each repeated action is either a chord row or a note that it has no shortcut.

3. **Choose verbs from the catalog.** Take only the rows in [Verb catalog](#verb-catalog) whose noun this app has. Global keys (door, Esc, `?`, and search when chosen) mean the same thing on every screen. Every other verb belongs to one screen. Done when the letter line for each screen names exactly the verbs you chose, in the app's words.

4. **Bind a chord for each listed action.** Take a row from [Chords](#chords) when this app has that noun. Keep a binding the app already uses, and put it on the card. When the familiar binding is taken, or the platform keeps it (a browser's New Window, Close Window, or address-bar keys the page cannot swallow), choose the next free binding in that row and write the one that works. One binding, one action. Shift is a different binding. Done when the card lists exactly the chords the host will run, in the app's words.

5. **Write both engines and their tests.** Implement [Letters](#letters) and [Chord engine](#chord-engine) as pure functions. The host is not in these tests. Done when a test fails if any letter rule or any chord row's key, Shift, or Alt changes, and an unbound combination returns pass.

6. **Host both layers.** The host performs focus, fetch, and navigation. The engines decide. Letters listen first and pass chords through. The chord function then runs on that same key, including inside a field, a terminal, and a dialog. On the web, read [references/web.md](references/web.md) before choosing a widget or a listener. When the app is a board of cards, or the user names Fizzy, read [references/fizzy.md](references/fizzy.md) before choosing board verbs. Done when the door opens on every screen outside a gate, Enter lands, a text field receives verb letters as characters, a chord from the card still runs inside that field, unshifted Esc leaves the field, and a terminal and an IME composition keep their letter keys. When digits are verbs, the digit shows on its item while the letter line is open.

7. **Exercise the card and the letter line.** In the running app, perform every verb on each screen's letter line and every row on the card. Repeat two chords from inside a text field, and one from a terminal when the app has one. Filter, land, and confirm a platform-owned binding still does the platform's action. Done when each of those happened and the card matches the keys that worked.

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

## Verb catalog

Take a row when this app has that noun.

| Verb | Key to try | Use it when |
| --- | --- | --- |
| Door | `J` | Every app. Opens the jump menu of places and named things. |
| Search | `K` | The corpus is larger than, or different from, the door's list. |
| Areas | `1` `2` `3` | The app has a handful of top-level places. |
| Visible items | `1`–`9` | This screen shows an ordered list. The digit matches visual order. |
| Walk | arrows, `j` `k` | This screen has a selection or a vertical list. |
| Neighbor | `[` `]` | This screen has a next and previous surface, column, or page. |
| Open | Enter | Something is selected. |
| Create | `C` | This screen can add its noun. |
| Edit | `E` | The selection has a field. |
| Done / defer | `]` and `[` when neighbor is free, else two free letters | The selection has those states. |
| Assign to me | `M` | The selection has an assignee. |
| Tag | `T` | The selection has tags. |
| Focus filter | `F` | This screen can narrow in place. A scope door uses another letter. |
| Scope door | a free letter | A noun is too big for the current screen (files, people, boards). |
| Insert | `i` | This screen has a composer or a primary field. |
| Back | Esc, and `←` when history exists | Always for Esc. `←` when the app has a back stack. |
| Cheat | `?` | Always. Toggles the letter line and the card. |

## Chords

Take a row when this app has that noun. The first binding is the one to try. The rest are fallbacks when that binding is taken or the platform keeps it.

| Action | Bindings to try | Use it when |
| --- | --- | --- |
| Command palette | ⌘K, ⌘⇧P | The app has a list of commands. |
| Go to a named record | ⌘P | Files, pages, people, or records are too many to scan. |
| Find in this document | ⌘F | The open document, thread, or record can search in place. |
| Find across the corpus | ⌘⇧F | Search reaches past the open document. |
| Primary column | ⌘B | A sidebar or primary list can hide. |
| Secondary column | ⌥⌘B, ⌘⇧E | A second column exists (files, details, inspector). |
| Other face of this noun | ⌘` | The same noun has two views. |
| New | ⌘N, then the Create letter | The screen can create its noun. |
| Close | ⌘W | The screen can close a tab, pane, or document. |
| Save | ⌘S | The noun can be saved. |
| Settings | ⌘, | The app has settings. |
| Often-toggled overlay | ⌘. | An assistant, inspector, or mode opens often. |
| Next / previous | ⌘⇧], ⌘⇧[ | Tabs, pages, or sections are ordered. |
| Focus place n | ⌘1–⌘9 | Panes, columns, or accounts are numbered. Plain `1`–`9` stay letter verbs. |
| Split | ⌘\\ | The surface can split. |
| Confirm | ⌘Enter | A primary confirm is not already the field's own Enter. |

These bindings always pass, in every app: undo, redo, cut, copy, paste, and select-all (⌘Z, ⌘⇧Z, ⌘X, ⌘C, ⌘V, ⌘A). While a rich-text or code field is focused, its formatting chords pass too. An app chord that uses another key still runs.

A letter verb may be a second path to an action that already has a chord. The card names the chord. The letter line names the letter.

### Chord engine

Input is the physical key, whether Shift is down, and whether Alt is down. The host has already required Meta or Ctrl. Output is an action, or pass.

Apply the first rule that matches.

1. Alt is down: the row that names this key with Alt and this Shift. Any other Alt combination passes.
2. The editing bindings above, or a formatting chord while a rich-text or code field is focused: pass.
3. The row for this key and this Shift. A toggle (show or hide the same thing) whose key is repeating holds, and the action does not run again. A row whose action can open its surface does that when another surface is showing, and the key is swallowed. A row that has no meaning on this surface passes.
4. Anything else: pass.

The card is the spec. It names each chord the host runs, and the host runs each row on the card.

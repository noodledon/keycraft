---
name: keycraft
description: >
  Keycraft adds a productive keymap to any app: a door key opens a jump
  menu, each screen has single-letter verbs, arrows walk, and gates pass
  keys through to text fields, terminals, and IME. Use when the user wants
  vim-style shortcuts, a Fizzy-style keyboard, a jump menu, or a faster
  keymap, or when they run /keycraft.
---

# Keycraft

Build a keymap a person can use before they memorize it. One **door** opens a **jump menu** from anywhere. Each screen has a few **verbs**. **Walk**, then a verb acts on what is highlighted. **Esc** closes one **layer**. **?** shows the **cheat** for this screen.

Home-row walk (`j` / `k`) is a verb on a vertical list. The door is how a person reaches a named thing.

## Terms

- **Door.** One key, free of chords, that opens the jump menu on every screen. Default `J` when `J` is free.
- **Jump menu.** The list behind the door. An empty query shows the app's order, recents first when the app has them, capped at 50. Typing **filters**. **Enter** **lands**. A single match stays highlighted until Enter.
- **Land.** The highlighted row becomes the current place or runs that row's action, then the menu closes.
- **Verb.** One key, one action, for this screen's noun. The same letter may be a different verb on another screen. The cheat names only this screen's verbs.
- **Walk.** Move the highlight or the selection. Arrows always walk. On a vertical list, `j` and `k` walk too. Screen walk stops at the ends. Menu walk wraps.
- **Two-beat.** Walk to a thing, then press a verb. The verb applies to the selection. Fizzy's board line is the pattern: move with the arrow keys first.
- **Scope.** A verb that opens the same jump menu already narrowed to one noun (files, people, boards).
- **Filter.** Characters narrow the jump menu. Match case-insensitively, stable, in this order: exact label, label prefix, haystack prefix, then substring of the label or the haystack. Cap 50. `/` starts a filter with an empty query so a name can begin with a verb letter. Once filtering, verb letters append. Arrows still walk. Backspace edits; an empty query leaves filtering off.
- **Gate.** A place that already owns the keyboard. The key passes through. Gates: a text field, a terminal, an IME composition, an open dialog, and any **chord**.
- **Chord.** A shortcut that uses Meta, Ctrl, or Alt. Existing chords keep working.
- **Layer.** One thing Esc closes: the open menu, or a text field. The cheat stays as it was, and verbs still run while it shows.
- **Cheat.** One line, toggled by `?`, that names this screen's verbs in the app's words. While the menu is open, its footer is the hint line and names walk, filter, land, and Esc.
- **Insert.** A verb that focuses the composer or this noun's field. Unshifted Esc leaves the field and returns to verbs.

## Steps

1. **Name the nouns.** List the places a person lands, the thing they select on each screen, and the field they type into, in the app's own words. Done when each screen has a noun, or an explicit note that the screen is view-only.

2. **Inventory gates and pick the door.** List existing chords, dialogs, terminals, and text fields. Pick the door, the search key, and the screen verbs from keys that are free. Search gets its own key (default `K`) when the corpus is a different list from the door. Done when the door key appears on no chord and in no text-entry path.

3. **Choose verbs from the catalog.** Take only the rows whose noun this app has. Global keys (door, Esc, `?`, and search when chosen) mean the same thing on every screen. Every other verb belongs to one screen. Done when the cheat line for each screen names exactly the verbs you chose, in the app's words.

4. **Write the engine and its tests.** Implement [Engine](#engine) as a pure function of state, key, gates, and match count. The host is not in these tests. Done when a test fails if any Engine rule changes.

5. **Host it.** The host performs focus, fetch, and navigation. The engine decides. On the web, read [references/web.md](references/web.md) before choosing a widget. When the app is a board of cards, or the user names Fizzy, read [references/fizzy.md](references/fizzy.md) before choosing board verbs. Done when the door opens on every screen outside a gate, Enter lands, a text field receives verb letters as characters, unshifted Esc leaves that field, and a terminal and an IME composition keep their keys. When digits are verbs, the digit shows on its item while the cheat is open.

6. **Exercise the cheat.** In the running app, perform every verb named on each screen's cheat, plus filter, land, a chord that already existed, a text field, and a terminal or IME when the app has one. Done when each of those happened and the cheat line matches the keys that worked.

## Engine

State is the mode (`verbs` or `insert`), the cheat flag, and either no menu or a menu `{ scope, query, index, filtering }`.

**Pass** lets the app receive the key. **Hold** consumes the key and leaves state unchanged.

Apply the first rule that matches.

1. Composing, terminal, chord, dialog, or a view that is not the app's main surface: pass.
2. Insert: unshifted Esc leaves insert, closes the menu, and keeps the cheat. Any other key passes.
3. Menu + Esc: close the menu, keep the cheat.
4. Menu + Enter: land when the match count is greater than zero, then close the menu. Otherwise hold.
5. Menu + Backspace: edit the query. An empty query turns filtering off and sets the index to 0. Hold when the query is already empty and filtering is off.
6. Menu + arrows: walk, wrapping, even while filtering. Hold when the match count is zero.
7. Menu + `j` or `k`, filtering off: walk the same way. Hold when the match count is zero.
8. Menu + `?`, filtering off: toggle the cheat. The menu stays.
9. Menu + `/`, filtering off: turn filtering on. The query stays empty.
10. Menu + the door or a scope door, filtering off: retarget that scope, clear the query, index 0.
11. Menu + a digit `1`–`9`, filtering off: move to that row when it exists. Otherwise hold.
12. Menu + any other character key: append it, turn filtering on, index 0.
13. Menu + anything else: hold.
14. Verbs mode: the screen's verbs, the door, search, and `?`. An unbound key passes.

`?` toggles the cheat in verbs mode and leaves the mode alone. The door opens a menu at scope `all`, query empty, index 0, filtering off. A scope verb opens that scope the same way. Insert sets mode to insert and closes the menu.

## Catalog

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
| Cheat | `?` | Always. |

Creation, edit, and save that already use a chord stay on that chord. The letter verb is the mouse-free path to the same action.

The cheat line is the spec: it names each verb the host implements, and the host implements each name on the line.

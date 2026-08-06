# CV2025 test plan — everything untested/unverified in this pack

This shop runs **CV2025.4** — every "verified against real CV output" result below was confirmed on that
specific version, not CV2025 generically.

**Status (2026-07-31): items 2, 6, and 7 verified on `jonah-shelf-standards-rev17.txt`** — dados are
controllable end-to-end (item 2), both default and through-bore dados nest correctly (item 6), and the
forward/reverse dado pair fires correctly, confirming `S_EXTND` placement is intentional, not a bug
(item 7). **Item 8 (`jonah-notch-construction.txt`) also verified** — the CV2023+ drawer-stretcher
origin issue doesn't reproduce on this install; the unmodified script already keeps a DS within the
interior frame. **Item 5 (Centerline Route) confirmed broken, fixed (Revision 22), and verified fixed**
— the route branch used a non-cutting `line` object with zero width instead of a real `route` object,
so nothing ever nested; now nests correctly. **Item 3 won't be tested** (user doesn't use shelf
standards day-to-day, won't touch assembly-wizard config to check it). **Item 10 also won't be tested**
(both sub-items are niche edge cases with no clear repro path — left as known gaps to deal with if/when
actually hit). Items 4 and 9 remain open, available to pick up later, not currently being pursued.

**Note added 2026-08-02:** cross-checking against the user's separate `cabinet-vision` skill write-up
of these same scripts surfaced one caveat worth reading before trusting the "verified" line above at
face value: `jonah-shelf-standards-rev17.txt` is currently *disabled* in this shop's live UCS Manager
(confirmed by screenshot). The verification itself is sound — it was done by running the script
directly — but the production install isn't currently running the fixed version day-to-day. See
README.md's "What was changed" and "CV2025 status" sections for the full note, and item 11 below for a
new, genuinely open question that same cross-check surfaced.

Written for someone who hasn't built or tested a UCS in CV before. Section 0 covers the mechanics
you'll reuse for every item below; the numbered items are the actual things to check, ordered
cheapest/fastest first. Report results back in order — early items sometimes make later ones moot.

A caveat on Section 0: I don't have Cabinet Vision in front of me, so these steps are reconstructed
from this repo's own documentation plus general knowledge of the product, not click-tested by me.
Menu wording/location can drift between CV versions and shop configurations. If a step doesn't match
what you see, **stop and send me a screenshot of where you are** (like you did for the two errors
already) rather than guessing — that's exactly how we caught both bugs so far.

Legend: 🔧 = attribute/UCS-editor only, no new geometry needed. 🧱 = requires building a small test
cabinet/interior. 🏭 = requires running the part to S2M/CNC output.

---

## 0. Mechanics you'll need for every test below

### 0a. Get a script into CV as a UCS

Each script in `scripts/` needs to become an entry in the target Assembly's **User Created Standards**
list (this is the "UCS Manager" the docs keep referencing) before it can run at all:

1. Open **User Created Standards** (the UCS Manager) — it's on the toolbar above the main workspace,
   not tucked inside any particular object or editing view. It's a global tool, not something you open
   from within the Assembly/Section Editor.
2. From there, get to the UCS list for the Assembly you want to test on (an existing cabinet from the
   catalog, or a new one).
3. Add a new entry and **paste in the script text** — copy/paste is the only way to get a script in
   short of a full `.pkg` import (no "import a `.txt` file" option). The **first line of the script
   becomes its name** in this list (e.g. `;Jonah's Shelf Standards REV 17` shows up as that).
4. Save/close the UCS Manager, then **rebuild** (see 0c) so it actually runs once.

List **order matters** — if you're testing a script that depends on another UCS having already run
(none of the ones in this test plan do, but the casework-wall stack does), it needs to sit below that
one in the list. Not relevant for items 2-7 below (shelf-standards is self-contained); relevant if you
later test the casework-wall stack, which must be imported as all 12 scripts in the documented order —
see `docs/casework-walls-jonah.md`'s stack table.

### 0b. Set a Public/part-level attribute

Attributes a script declares with `Public` (UCS-level defaults) or plain declarations inside the
`OBJECT == 17` part branch (part-level overrides) show up in an attribute panel **lateral to (beside)
the Object Tree, in the Home tab** — not nested inside it, by design, so you shouldn't have to dig into
an individual object's own parameters to find them. Click the value next to a parameter name to edit
it; a `<lst>` parameter shows as a dropdown, a `<meas>` as a measurement field, `<bool>` as true/false.

### 0c. Rebuild and read errors

There's no dedicated "Rebuild" command — the built-in ways to force CV to re-run a UCS are: switch
views (e.g. Plan → Section → back), or click blank space in a layout view and press Enter, or close and
reopen the job. If a script has a hard error, CV shows the popup dialog you've already seen twice
(`CABINET VISION` title, yellow warning icon, `<UCS name> (line N)` and a short error like
`Use of Undefined (x)`) — that popup and its exact wording is the single most useful thing to screenshot
me when something goes wrong.

### 0d. Prerequisite specific to shelf-standards (items 2-7 below): getting an `L?VBORE` operation to test against

This is what item 2's follow-up screenshot ran into: `jonah-shelf-standards-rev17.txt` only converts
line-bore holes CV's **own** hardware/hole system already created (operations literally named
`LFVBORE`/`LRVBORE`, or similar single-character variants matching the `L?VBORE` wildcard) — it doesn't
generate shelf standards from a blank end. If the part you attached the script to has no such operation
in its Object Tree yet (as with the `LU` end in your screenshot, which only had `UBDADO`/`LEFHBORE`),
the part-level attributes will still appear (that half of the script always runs), but nothing else
will happen — no dado, no visual part.

Operations aren't a standalone thing you toggle directly — `LFVBORE`/`LRVBORE` get generated as a
side effect of the interior actually having shelves. The play: enter the Section Editor, go to the
interior, and add shelves. **Confirmed: adjustable shelves only** — fixed shelves don't trigger
`L?VBORE` generation, so the shelf-standards script has nothing to act on with a fixed shelf (this
tracks with the feature's whole purpose: shelf standards are pilaster strips for *adjustable* shelf
clips, so it makes sense CV only line-bores for those). Once adjustable shelves exist, CV's own
line-boring produces the `L?VBORE` operations on the adjacent end/partition, and the shelf-standards
script has something to act on.

---

## 2. 🔧 Is `ucs_ShelfStd` resolving cleanly? — ✅ VERIFIED: dados are controllable end-to-end

**Round 1 result:** CV threw `Use of Undefined (ucs_ShelfStd)` — a `Public` declared before the
`For Each` loop, which classic UCS doesn't allow. Fixed in Revision 20 (moved inside the loop).

**Round 2 result:** no error, but also no dado/visual — because the test end had no `L?VBORE`
operation for the script to act on. Not a script bug; a missing precondition, resolved by adding
shelves to the interior (see 0d — confirmed working, `Front Line Bore (LFVBORE)` / `Rear Line Bore
(LRVBORE)` appeared).

**Round 3 result — the real bug:** with `L?VBORE` operations present and `JCS_ShelfStd_Use = 1`,
`JCS_ShelfStd_LengthType = 3`, `JCS_ShelfStd_Length_Over = -1` all showing correctly on `Unfinished
Left End (LU)`, still **no dado, no visual part, nothing created.** Root cause, found by re-checking
the original source directly: the `;operations only from here on down` block — which computes and
*stores* `JCS_Use_ShelfStd` on the part so operations can read it back via `:JCS_Use_ShelfStd` — had
been placed *after* `if OBJECT == 17 then ... end if` closed in the patched version, instead of inside
it as the original source actually has it. That meant the value was never stored on the part, so every
operation's `:JCS_Use_ShelfStd == 0` check read null/undefined and exited immediately — explaining
exactly what was seen: correct part attributes, zero downstream effect. Fixed in Revision 21 by moving
the block back inside `OBJECT == 17`, matching the original source exactly.

**Round 4 result — verified.** With Revision 21, dados are controllable on an adjustable-shelf end. User
confirmed overall correctness and isn't testing every remaining function individually — this closes out
item 2 as the baseline "does the script work at all" check. One real, confirmed limitation found along
the way: **it doesn't work on fixed shelves** — see the note in 0d above (only adjustable shelves
generate the `L?VBORE` operations this script needs, so a fixed shelf's end/partition has nothing for
it to act on; this is consistent with the feature's purpose, not a bug to fix).

## 3. 🔧 Does entry "6) Wizard Size (Centered)" work without an explicit `=6`? — ❌ WON'T TEST

**User has declined this one permanently** — doesn't use shelf standards day-to-day and isn't willing
to touch assembly-wizard configuration just to verify it. Not a "pause for now," a real "not going to
happen," so treat this as closed rather than pending.

What was going to be checked, for reference if anyone else ever wants to pick it up: with an `L?VBORE`
operation present (0d) and the script attached and working, open the UCS-level `ucs_Dado_LengthType`
attribute (not the part-level `JCS_ShelfStd_LengthType`, which already has `=6`) and pick the
unnumbered **"6. Wizard Size (Centered)"** entry, rebuild, and look at the resulting dado/visual part.
If it behaves as centered-wizard-size, CV assigns positional list entries an implicit value; if it
falls through to something else, the UCS-level list entry needs an explicit `=6` added, matching the
part-level list. Left unresolved in `docs/shelf-standards-jonah.md` as an open question, unchanged.

## 4. 🔧 Does CV normalize `Size1` to `Size01`? (low priority — the fix doesn't depend on the answer)

Skip unless you're curious — this is the lowest-value item in the list. If you do want to check: set
`ucs_Use_Sizes_Behavior_When_Less_Than_Smallest` to "Use minimum size anyway," and in a **scratch copy**
of the script (don't touch the shipped file), change the fallback lines back to the original unpadded
`ucs_Use_Sizes_Size1` / `ucs_Use_Sizes_Size1_MatID`. Build a below-minimum-size shelf and see if it
still resolves a size.

- **If it resolves:** CV normalizes numeric suffixes — the original defect wasn't really a defect.
- **If it silently comes back null/0:** confirms the original bug was real (no action needed either
  way, since the shipped file already uses the padded names).

## 5. 🧱 Route vs. Dado bug (Rev 10) — does `ucs_Operation_Type = Centerline Route` still come out as a dado? — ✅ FIXED AND VERIFIED (Revision 22)

**Result: confirmed broken, root cause found, fixed, and confirmed fixed against real CV2025.4 output.**
With `ucs_Operation_Type` set to "Centerline Route", CV produced a line in the assembly view, but the
operation never appeared on the CAM Reports/nest list — nothing to machine. Not the bug Revision 10's
own comment described ("gets dim'd as a dado even if set to route") — the object was never a dado, it
just was never a real machinable operation.

Root cause (Revision 22): the route branch was `dim ShelfStandardDado as new line` — `line` is a
non-cutting visual/dimension object in CV's object model, not a machinable operation type — with its
width hardcoded to `ShelfStandardDAdo.DX := 0` (also a case typo on the variable name).

**Fix:** switched the route branch to `dim ShelfStandardDado as new route` with `_RCUT := 1` — the same
object type this shop's own casework-wall scripts use for a real machined slot (e.g.
`WireChaseRectangle` / `REMOVABLEPANEL`) — gave it the same real `DX` width as the dado branch instead
of 0, and extended the centerline-to-corner X/Y offset (previously dado-only) to both branches so the
now-wider route stays centered on the shelf-standard line instead of shifting a half-width off it.

**User confirmed fixed** — Centerline Route now nests correctly on CV2025.4. No further action needed here.

## 6. 🧱🏭 "Generate 2 CNC files" bug (Rev 7) — is `TEMP_Fix_For_1_CNC_File` still needed? — ✅ VERIFIED WORKING

**Result: confirmed both ways** — default (non-through) dados nest correctly, and the through-bore case
(dado depth equal to the part's own thickness, where the script's `TEMP_Fix_For_1_CNC_File` workaround
kicks in) also nests correctly. The shipped script produces correct nesting output as-is on CV2025.4.

Left genuinely open, lower priority: whether CV2025.4 would *also* handle the through-bore case correctly
on its own, without the workaround (i.e. whether the workaround is now dead weight vs. still load-bearing)
wasn't specifically isolated — that would mean temporarily disabling `TEMP_Fix_For_1_CNC_File` and
re-checking, which isn't worth doing since the current behavior is already correct either way.

## 7. 🧱🏭 `S_EXTND := true` on the forward op — copy-paste slip or intentional? — ✅ CONFIRMED FIRES CORRECTLY

**Result: confirmed correct as shipped.** Both the forward dado and the reverse-face dado fire and
extend correctly on CV2025.4 — `S_EXTND := true` on the forward op is intentional, not a copy-paste slip.
No change needed; nothing in the script should be touched here.

## 8. 🧱 `JCS_NotchConstruction_CV2023Plus_DSOrigin` — the flag I added, highest-stakes item — ✅ CONFIRMED NOT NEEDED

Different script (`jonah-notch-construction.txt`), different setup — import it per 0a onto a cabinet
with an interior. Build (or use an existing) interior with a drawer stretcher (`DS`) part, and set its
`Notch: Length` option to "Entire Interior" (`JCS_NotchConstruction_LengthType_Use = 1`) so it extends
across the full opening.

**Result: confirmed not reproducing.** With the flag **off** (the shipped default, unmodified) — the DS
part stays correctly within the interior frame. The forum-reported CV2023+ drawer-stretcher origin
issue doesn't show up on this install; the original, unpatched script already behaves correctly here.

The flag itself is left in place (still off by default) rather than removed, in case a different
CV2025 build/config ever hits the forum symptom — but it's not needed on this shop's install, and
nothing further to test here.

## 9. 🧱 Casework-wall CV2025.3 fixes — `_SHPEDGCNT` shape test and `rprec()` rounding guard

This is a bigger lift — it needs the full 12-script casework-wall stack imported in order (see 0a and
`docs/casework-walls-jonah.md`'s stack table), on a wall assembly. Only worth attempting once items
2-8 are comfortable, since it's the most CV-experience-intensive item here. Two things to check on
`jonah-casework-wall-10-studs-and-stud-dadoes.txt` once the stack is running:

1. **Shape test.** Add a `PT` with a `PTDADO` into the top or bottom on an **unshaped** stud (per the
   forum repro this fix comes from). Confirm the part is still correctly treated as unshaped now that
   the test reads `_SHPEDGCNT == 0` instead of `_EDGWP == 0`.
2. **Rounding guard.** Build a face with a genuinely lowered face next to a normal one, and confirm the
   top plate still machines correctly with the `rprec()`-wrapped adjustment line — i.e. that wrapping
   it in `rprec()` didn't also round away a real, non-zero adjustment.

- **If both hold up:** mark this fix verified in README.md.
- **If the shape test misfires either direction:** screenshot it and I'll re-check `_SHPEDGCNT`.
- **If a real lowered-face adjustment gets rounded away:** the guard is too aggressive — tell me the
  magnitude you're seeing and I'll switch to an explicit small-tolerance compare instead of `rprec()`.

## 10. Lower priority / not really actionable without more context — ⛔ WON'T BE TESTED (declined by user)

- **Bill Crouch's .502"-instead-of-1.004"-depth report** — tool-catalog-specific to his shop, not clearly
  reproducible without his exact tool setup. Only chase it if you hit the same symptom.
- **The unresolved "top plate within cleat's space" case** in the casework-wall stud/dado code — the
  author himself says this isn't handled. Not a test, just a known gap to route around if you hit it.

**Declined:** both are super-niche edge cases with no clear repro path on this shop's setup. Left as
known gaps to deal with if/when actually hit, not something to chase preemptively.

## 11. 🔧 What is cabinet attribute `_CB:525` called in CV's UI? — OPEN, added 2026-08-02

`jonah-shelf-standards-rev17.txt` reads `_CB:525` bare, from part context, to detect horizontal-grain
panels (the switch that swaps which dimension bounds a through/stop dado — see Revision 17's own history
entry, "Support for horizontal grain panels"). Surfaced by cross-checking against the user's separate
`cabinet-vision` skill write-up, which flagged this as still-unconfirmed there too. Like every other
`_CB:NNN`/`TOOLID`/`ConstID` value in this corpus, the number is install- or version-specific and
shouldn't be assumed portable. Not urgent — the script works whether or not anyone knows the UI label —
but worth checking next time someone's in the Assembly Manager wizard near a horizontal-grain setting.

---

## After you run these

Send me results (and screenshots of any error dialogs, same as the last two) in order — partial
progress is fine. I'll update README.md/CHANGELOG.md with what's confirmed, flip flags to tested
defaults, apply any corrections the tests reveal, and commit each round separately.

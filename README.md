# Jonah Coleman — Cabinet Vision UCS source pack

This repo collects all of Jonah Coleman's Cabinet Vision classic-UCS
scripting work that this shop has on file, cleaned up and organized for version
control, with a number of documented bugs fixed. Two scripts are now **verified against real CV2025.4
output** (this shop's installed version): `shelf-standards/jonah-shelf-standards-rev17.txt` (dados controllable end-to-end) and
`notch-construction/jonah-notch-construction.txt` (the reported CV2023+ drawer-stretcher origin issue
does not reproduce on this install). The rest of the CV2025 compatibility work is still notes/patches
that haven't been run — see `TESTING.md` for what's confirmed versus still open.

## What's here

```
scripts/
  notch-construction/     jonah-notch-construction.txt   (Rev 12 + this repo's Rev 13 patch)
  shelf-standards/
    jonah-shelf-standards-rev17.txt        patched version, CV2025-PATCH fixes applied (see below)
    jonah-shelf-standards-rev17-raw.txt     unmodified original, kept verbatim for provenance
  casework-walls/         the 12-script "Jonah Rocks Millwork Walls Revision 3" stack, in load-bearing order
  keku-panel-clips/       jonah-keku-panel-clips.txt
  miterfold/              jonah-miterfold.txt
  visible-part-splits/    jonah-visible-part-splits.txt
  overlay-calculations/   jonah-overlay-calculations.txt
                           jonah-overlay-calculations-19ucs.txt   (Order 4 in the casework-wall stack)
  blum-tandembox/         jonah-blum-tandembox-77ucs.txt   (UCS 77, "Blum Tandembox New")
  tools/                  two recovered AutoIt tool sources: aa-create-index-autoit.txt,
                           jonah-multiple-operations-copier-autoit.txt (not UCS -- compiled Win32 tools)
docs/                     curated technique/defect write-ups for the three largest systems, plus
                           external-sync-rules.md (how this repo reconciles with the cabinet-vision skill)
source-threads/           the two Hexagon Nexus discussion threads these scripts came from, plus a
                           third capture (a Nexus "Ideas" post) confirming what `CO` stands for and a
                           real CV limitation on its Z/DZ fields -- all captured verbatim
```

## Authorship

Everything under `scripts/` is Jonah Coleman's own work, identifiable by his parameter-prefix
habit (`JCS_` = his own initials, `FWP_` = Fletcher Wood Products, `AA_` = Architectural Arts — see
`docs/casework-walls-jonah.md` for the fingerprint detail) and, where present, his credit header
(`Jonah@FletcherWoodProducts.com`, `JonahC@ArchitecturalArts.com`).

## What was changed in this pack

1. **`notch-construction/jonah-notch-construction.txt`** — fixed a `delete delete` duplicate-keyword typo;
   added an opt-in, default-off `JCS_NotchConstruction_CV2023Plus_DSOrigin` flag for a forum-reported
   CV2023+ drawer-stretcher origin issue. **Verified against real CV2025.4 — the issue doesn't reproduce on
   this install** (TESTING.md #8).

2. **`shelf-standards/jonah-shelf-standards-rev17.txt`** — patched from the unmodified original
   (`-raw.txt`, kept alongside it for provenance). **Verified against real CV2025.4 as of Revision 22:
   dados, CNC nesting, `S_EXTND` placement, and Centerline Route all work correctly.** Fixes: a missing
   `Public ucs_ShelfStd` declaration, a misplaced code block that was silently breaking every operation, an
   unpadded fallback parameter name, and a broken Centerline Route operation (was never a real machinable
   object). Full round-by-round story in TESTING.md #2 and #5; caveats — currently disabled in this shop's
   live UCS Manager, a revision-numbering discrepancy in the original — in `docs/shelf-standards-jonah.md`.

3. **`casework-walls/jonah-casework-wall-10-studs-and-stud-dadoes.txt`** — a CV 2025.3 regression, confirmed
   by a Verified Answer on the Hexagon Nexus forum, broke this script's "is this part shaped" test. Two
   fixes applied (swapped `_EDGWP == 0` for `_SHPEDGCNT > 0`; wrapped a `DY -=` adjustment in `rprec()` to
   guard a floating-point rounding bug) — sourced from a corroborated forum report, not yet run against a
   real CV2025.4 install by this repo. Full writeup in `docs/casework-walls-jonah.md`.

No other duplicate-keyword or similarly mechanical defects were found across the rest of the corpus.

## CV2025 status — read before running any of this in production

**This shop runs CV2025.4.** Two scripts in this pack — `shelf-standards/jonah-shelf-standards-rev17.txt`
and `notch-construction/jonah-notch-construction.txt` — are now **verified against real CV2025.4 output**
(see "What was changed" above and `TESTING.md` for the full round-by-round story). Everything else below
is either still unverified against a real install, or is general compatibility risk to check before
relying on these scripts on any CV2025 install:

- **`jonah-notch-construction.txt`** — the reported CV2023+ drawer-stretcher origin issue **has been checked
  against real CV2025.4 output and does not reproduce on this shop's install** (TESTING.md #8): with the
  `JCS_NotchConstruction_CV2023Plus_DSOrigin` flag off (the shipped default, unmodified), an "entire interior"
  drawer stretcher stays correctly within the interior frame. The flag stays in the script, still off by
  default, in case a different CV2025 build/config ever hits the forum-reported symptom.
- **`jonah-casework-wall-10-studs-and-stud-dadoes.txt`** has a confirmed CV2025.3 regression (see the patch
  note above) — both known fixes are applied, but only against a forum-corroborated report, not this repo's
  own CV2025.4 testing (TESTING.md #9, still open). If your studs come out unshaped or top plates don't
  machine, this is the first thing to check.
- **`Cab.ConstID`, `TOOLID`, and cabinet-attribute numbers (`_CB:NNN`, `_CV:NNN`) throughout this corpus are
  install-specific.** They are indexes into a local catalog/database and are not guaranteed to mean the same
  thing on a different CV2025 install, even where a value is documented (e.g. `Cab.ConstID == 20` for
  "Casework Wall" — confirmed only for the specific bundle this pack was extracted from). This includes the
  shelf-standards script's own `_CB:525` horizontal-grain flag read (see TESTING.md's open items) — never
  confirmed against the UI, and not portable between installs.
- **The casework-wall stack's ordering is load-bearing and unenforced by CV** — see the stack table in
  `docs/casework-walls-jonah.md`. Import order in a fresh CV2025 catalog should follow that table. This
  stack has not been run against a real install by this repo.
- The `OBJECT` codes used throughout are confirmed by Hexagon's own published `OBJECT` code table for the
  handful this pack cites (`17` = Part, `15` = Subassembly, `10` = Cabinet, `37` = Order) — see the relevant
  `docs/*.md` write-up for each. Any `OBJECT` value not called out there is still just empirically observed
  against this corpus, not independently verified.
- No CNC output from any of these scripts has been checked against a current post-processor.
- **A currently-disabled script isn't the same as a superseded one.** `jonah-shelf-standards-rev17.txt`
  is verified as fixed but is presently switched off in this shop's own UCS Manager — see the caveat under
  "What was changed" above. Check the live Manager state before assuming any script in this pack is
  actually running in production, not just fixed on disk.

If you get the remaining items running on a CV2025 install, the most valuable thing to record back into
this repo is which of the above actually held up and which didn't.

## Source provenance

Full provenance, dating, and technique write-ups are in `docs/`. Raw forum thread captures (setup instructions,
known-issue reports, and the author's own explanations) are in `source-threads/`.

**Why bother version-controlling someone else's forum scripts at all.** A separate Hexagon Nexus thread,
["Jonah I want to see a list of the UCSs you have shared!"](https://nexus.hexagon.com/community/cabinet_vision/f/cv-user-created-standards/38341/jonah-i-want-to-see-a-list-of-the-ucss-you-have-shared),
makes the case better than this repo can: asked for a compiled, linked list of his own UCSs so users could keep them up to date, Jonah Coleman
replied "I won't have time to compile this but someone else could" — and, in the same reply, flagged that
another user was still running an old script that "fixes a bug that was fixed in a later build of 8." No
canonical, versioned source ever existed on his end; whatever's in circulation is whatever got pasted into
whichever forum post, at whatever revision was current that day. That's the gap this repo is filling for
this shop's own copies.

**Confirmed directly, 2026-08-13.** Asked point-blank whether he was fine with this repo publicly documenting
his old CV work, Jonah Coleman replied "Sure, go for it." This isn't inferred from the forum reply above
anymore — it's his direct yes to this specific repo.

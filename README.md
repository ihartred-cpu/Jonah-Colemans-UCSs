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
  shelf-standards/        jonah-shelf-standards-rev17.txt
                            patched version, CV2025-PATCH fixes applied (see below)
                          jonah-shelf-standards-rev17-raw.txt
                            unmodified original, kept verbatim for provenance
  casework-walls/         the 12-script "Jonah Rocks Millwork Walls Revision 3" stack, in load-bearing order
  keku-panel-clips/       jonah-keku-panel-clips.txt
  miterfold/              jonah-miterfold.txt
  visible-part-splits/    jonah-visible-part-splits.txt
  overlay-calculations/   jonah-overlay-calculations.txt
                           jonah-overlay-calculations-19ucs.txt   (Order 4 in the casework-wall stack)
  blum-tandembox/         jonah-blum-tandembox-77ucs.txt   (UCS 77, "Blum Tandembox New")
  tools/                  two recovered AutoIt tool sources: aa-create-index-autoit.txt,
                           jonah-multiple-operations-copier-autoit.txt (not UCS -- compiled Win32 tools)
docs/                     curated technique/defect write-ups for the three largest systems,
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

1. **`notch-construction/jonah-notch-construction.txt`**
   - Fixed a `delete delete JCS_NotchConstruction_Interior_EnableAll` duplicate-keyword typo in the
     disabled-cleanup branch (harmless in practice — the branch is rarely reached — but corrected).
   - Added an **opt-in, default-off** `JCS_NotchConstruction_CV2023Plus_DSOrigin` parameter that attempts to
     compensate for a forum-reported CV2023+ change to the drawer-stretcher part origin (Top Back Left →
     Bottom Front Right), which reportedly makes an interior-extended drawer stretcher stick out the wrong
     side of the cabinet. **Verified against real CV2025.4 output (TESTING.md #8): the issue does not reproduce
     on this shop's install** — with the flag off (shipped default, unmodified), the DS part stays correctly
     within the interior frame. The flag stays in place, still off by default, in case a different CV2025
     build/config hits the forum-reported symptom.

2. **`shelf-standards/jonah-shelf-standards-rev17.txt`** — patched from the unmodified original
   (`jonah-shelf-standards-rev17-raw.txt`, kept alongside it for provenance). **Verified against real
   CV2025.4 output as of Revision 21 — dados are controllable end-to-end.** Getting there took three rounds of real,
   confirmed bugs (see TESTING.md #2 for the full story):
   - **Round 1:** `Public ucs_ShelfStd` declared before the `For Each` loop — CV rejected with
     `Use of Undefined (ucs_ShelfStd)`. Fixed by moving inside the loop (Revision 20).
   - **Round 2:** no error, but no dado/visual either — a missing precondition, not a script bug (the part
     needs an existing `L?VBORE` line-bore operation to convert; see TESTING.md 0d). **Confirmed: only
     adjustable shelves generate `L?VBORE` — this script does not work on fixed shelves**, which is
     consistent with the feature's purpose (shelf standards are pilaster strips for adjustable shelf clips),
     not a defect.
   - **Round 3, the real bug:** with the precondition met, still nothing happened even though the part-level
     attributes looked correct. Root cause: the `;operations only from here on down` block — which computes
     and **stores** `JCS_Use_ShelfStd` on the part so operations can read it back via `:JCS_Use_ShelfStd` —
     had been placed *after* `if OBJECT == 17 then ... end if` closed instead of inside it, contrary to the
     original source. That meant the value was never actually stored, so every operation's
     `:JCS_Use_ShelfStd == 0` check read null and exited immediately. Fixed by moving the block back inside
     `OBJECT == 17`, matching the original source exactly (Revision 21).
   - Also fixed: the "less than smallest size" fallback read `ucs_Use_Sizes_Size1` / `ucs_Use_Sizes_Size1_MatID`,
     never declared anywhere else in the script (the declared, zero-padded names are `Size01`/`Size01_MatID`).
     Changed to read the names that actually exist — not independently exercised by the verification above
     (it's a narrow below-minimum-size fallback path), but low-risk.
   - **Also verified: CNC nesting.** Both default (non-through) dados and the through-bore case (where
     `TEMP_Fix_For_1_CNC_File` kicks in) nest correctly on CV2025.4 (TESTING.md #6).
   - **Also verified: `S_EXTND := true` on the forward op is intentional, not a copy-paste slip** — both
     the forward and reverse-face dados fire and extend correctly (TESTING.md #7).
   - **Round 4, Revision 22: the "Centerline Route" option (`ucs_Operation_Type = 2`) — confirmed broken,
     fixed, and verified fixed on real CV2025.4.** Setting it produced a line in the assembly view but
     nothing on the CAM nest list — not the "gets dim'd as a dado" bug Revision 10's own comment
     described, but two layered bugs: the route branch used `dim ... as new line` (a non-cutting
     visual/dimension object, not a machinable type) with its width hardcoded to 0. Fixed by switching to
     `dim ... as new route` with `_RCUT := 1`, giving it a real width matching the dado branch, and
     extending the centerline-to-corner offset (previously dado-only) to both branches so the route stays
     centered. Centerline Route now nests correctly. See TESTING.md #5.
   - User has confirmed overall correctness and isn't verifying every remaining function individually.
     Item 3 won't be tested (not a shelf-standards user, won't touch assembly-wizard config to check it) —
     Item 10 also won't be tested (niche edge cases with no clear repro path) — see TESTING.md for what
     else is still open (items 4, 9).
   - **Caveat, added 2026-08-02, cross-checked against the user's separate `cabinet-vision` skill
     write-up of the same script: this script is currently *disabled* in this shop's live UCS Manager**
     (confirmed by a screenshot of the "User Created Standards" window — entry "JCS Shelf Standards,"
     `Enabled` checkbox unchecked). This doesn't undermine the verification above — testing was done by
     running the script directly, not by relying on what's switched on day-to-day — but the production
     install isn't currently running it even after Revision 22, and why it's off isn't recorded anywhere
     in either this repo or the skill. Worth asking about before assuming this fix is live in production.
   - **Revision-numbering note, also added 2026-08-02:** the original file's header says `REV 17`, but its
     own shipped revision history runs **Revision 2 through Revision 18** — the Revision 18 work (adding
     the `6) Wizard Size (Centered)` list entry) is already in the body while the title still says 17.
     This repo's Revisions 19-22 continue that numbering on top of 18, not 17 — say "Revision 22 on top of
     the original's Revision 18" rather than "Revision 5" if this ever needs describing to Jonah Coleman or
     anyone else holding the original file.

3. **`casework-walls/jonah-casework-wall-10-studs-and-stud-dadoes.txt`** — a CV 2025.3 regression, confirmed by
   a Verified Answer on the Hexagon Nexus forum, broke this script's "is this part shaped" test. Two fixes
   applied (see `docs/casework-walls-jonah.md` for the full forum writeup):
   - The shape test `if this._EDGWP == 0 then ;not shaped` (line ~30) was swapped for `_SHPEDGCNT > 0`, the
     parameter the forum's Verified Answer names as the correct test — CV 2025.3 can now report `_EDGWP`
     non-zero on a part that was never shaped.
   - `DY -= TEMP_TopAdjust` (line ~366) was wrapped in `rprec()` — a near-zero floating-point residual in
     `TEMP_TopAdjust` was silently shaving ~0.012mm off `DY`, enough to stop downstream top-plate dado
     operations from firing.
   - Sourced from a corroborated forum report, not run against a real CV2025.4 install by this repo. A second,
     unrelated `_EDGWP == 0` line at line ~90 reads as a different idiom and was deliberately left unchanged.

No other duplicate-keyword or similarly mechanical defects were found across the rest of the corpus (checked
by scanning for repeated keywords across every `jonah-*` script).

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
- The `OBJECT` codes used throughout (e.g. `OBJECT == 17` for Part) are empirically observed against older CV
  versions, not published by CV, and have not been independently re-verified beyond what's already confirmed
  above.
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

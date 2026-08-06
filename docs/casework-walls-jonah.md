# Casework Walls — Jonah Coleman's Millwork Walls UCS stack

A 13-script classic-UCS system that turns a single CV assembly into a framed millwork wall — reception walls, die walls, partition walls — with studs, plates, cleats, faces, removable panels and CNC-ready machining, driven entirely from the plan-view shape. It is the largest single UCS system this skill has seen: 5,492 lines across 13 scripts, versioned by its own author as `AA_CaseworkWall_Version` 2.8.

**Provenance and date.** The system arrived as `1106.JONAH Rocks Millwork Walls Revision 3.pkg`, a CV catalog-export bundle (70 files, 83,222,698 bytes uncompressed, internally dated 2018-08-08, md5 `57d229ee1335854480ac6241aacb0b20`), posted by Jonah Coleman to the Hexagon Nexus thread "Jonah's Millwork Walls Revision 3 (reception walls, die walls, etc)". The bundle plus roughly eight years of replies is preserved verbatim at `full-docs/nexus-thread-1106-jonah-rocks-millwork-walls.txt`; all 13 script bodies are filed byte-exact under `examples/jonah-casework-wall-*.txt`. Everything below is verified against those files — where the write-up states a mechanism the source doesn't establish, it says so.

**Read this before reading the scripts.** They are dense enough that a first pass will produce wrong conclusions; two of the traps below (indentation, `dim` rebinding) already caused verifiably wrong readings during this write-up.

## The stack, as data

The `.ucs` records in the bundle carry the whole UCS Manager row, so stack position and flags are exportable facts rather than screenshots of a dialog. All 13 share `CabName='*'`, `CabClass/CabType/CabConst = 0`, `Disabled = 0`, `NewPartSettings/ModifyPartSettings = 0`, `LibraryPart = '#NULL#'`, `CodeModified = 1` — that is, no gating at the manager level at all; every filter is inside the code.

| Order | File in bundle | Filed as | Lines | PreBuild | ObjectID | Name in UCS Manager |
|---|---|---|---|---|---|---|
| 1 | `25.ucs` | `examples/jonah-casework-wall-01-cab-before-build.txt` | 373 | **1** | 141 | CAB Before Build |
| 2 | `115.ucs` | `…-02-faces-interiors-1of2.txt` | 112 | 0 | 248 | Faces and Interiors (1/2) |
| 3 | `125.ucs` | `…-03-faces-interiors-2of2.txt` | 101 | 0 | 248 | Faces and Interiors (2/2) |
| 4 | `19.ucs` | **not filed — see below** | 188 | 0 | 137 | FWP: DOR/DWR Overlays |
| 5 | `20.ucs` | `…-04-fwp-door-drawer-fronts.txt` | 298 | 0 | 0 | FWP: Door/Drawer Fronts |
| 6 | `136.ucs` | `…-05-face-control-1-cleat-delete.txt` | 70 | 0 | 137 | Casework Walls (1/7) - Face Control 1, WALLCLEAT D… |
| 7 | `84.ucs` | `…-06-face-control-2.txt` | 1568 | 0 | 137 | Casework Walls (2/7) - Face Control 2 |
| 8 | `137.ucs` | `…-07-face-control-3-cleat-insert.txt` | 84 | 0 | 137 | Casework Walls (3/7) - Face Control 3 |
| 9 | `120.ucs` | `…-08-apply-before-build.txt` | 57 | **1** | 137 | Casework Walls (4/7) - Apply Before Build |
| 10 | `122.ucs` | `…-09-faces.txt` | 648 | 0 | 137 | Casework Walls (5/7) - Faces |
| 11 | `116.ucs` | `…-10-studs-and-stud-dadoes.txt` | 1328 | 0 | 137 | Casework Walls (6/7) - Studs and Stud Dadoes |
| 12 | `119.ucs` | `…-11-all-other-parts.txt` | 552 | 0 | 137 | Casework Walls (7/7) - All Other Parts |
| 13 | `117.ucs` | `…-12-radius-blank-panel-inset-fix.txt` | 126 | 0 | 137 | FIX FOR RADIUS BLANK PANELS NOT INSETTING |

Two scripts are Apply-Before-Build (`Order` 1 and 9). Ordering is otherwise load-bearing and unenforced: `117.ucs` line 1 is just a comment, `;This should be before any other part level UCS`, and nothing in CV checks it — yet it ships at position 13. That contradiction is in the source as delivered; don't read the comment as describing the shipped stack.

**`19.ucs` is deliberately not filed.** It is an *earlier* revision of the already-filed `examples/jonah-overlay-calculations.txt` — 17 diff lines, and the filed copy (204 lines) is the newer one, carrying a `DOOR_IS_STILE_AND_RAIL` / `DOR.DOORSTYLEID` block and `{JONAH_Interior_Split_Above_Path}.PCTR` lines that the bundle's 187-line version lacks. So the filed copy post-dates 2018-08-08. Refiling the older text would have created a third copy of a script this skill already stores twice (see the note on `overlays-calculation.txt` in `examples/README.md`).

## Setup chain — Jonah Coleman's own installation instructions

Verbatim source: `full-docs/nexus-thread-1106-jonah-rocks-millwork-walls.txt`, first post. Paraphrased in order, with the reasons he gives:

Delete any pre-existing UCS by the names in the table above *before* importing — "Doesn't seem like they properly overwrite." Then import the package choosing to overwrite existing. Edit the new assembly material schedule `CASEWORK WALL - PARTICLE BOARD`: set **all bandings to none**, and repoint the 3/4 particle boards at the shop's own 3/4 material (another thickness works). Under interior parts, set `Casework Wall Cleat` to the same material. Start a new job, place the `Casework Wall` object from the `Jonah Rocks` catalog, and click through the error messages — accept default schedules at that point. Take the object to the assembly editor's section editor, edit assembly properties, and set the **Upper** Standard Material Schedule to `CASEWORK WALL - PARTICLE BOARD`, the Hinge Schedule to `CASEWORK WALL .75 DOORS`, and Assembly Construction to `CASEWORK WALL`. Save the assembly back into the catalog **without changing any preserve checkboxes**.

Two details in that list are easy to skip and are not cosmetic. The schedule must go on the **Upper** slot, and the code says why: `25.ucs:238` reads `_MS11 = _MS1011` with the comment `;ucs dimmed door parts take base door schedules, we want upper` — a UCS-dimmed door part inherits the *base* door schedule, so the system remaps to reach the upper. And the interior part named `Casework Wall Cleat` in the UI is the object the scripts address as `WALLCLEAT` (`Case → Interior`); every cleat operation in `122.ucs` and `136/137.ucs` keys off it.

## The modelling rule

**Edit the shape from plan view only.** Anywhere panels are wanted is a **face**; anywhere the wall joins another wall is a **"None"**; anywhere it touches a wall and a scribe is wanted is **also a "None"**. The one genuinely manual step is adjusting the bottom plates' shape for toe recess.

## Tooling — the bit specs and the TOOLID map

Jonah Coleman's four tooling notes, in his words, are: a **5 mm BRAD point** bit for the plate/cleat screw holes — a v-point "results in a LOT of holes in your table"; a **3/4" bit** for slots when the material is close to but under 3/4 (plywood); a **25/32" bit** for slots when the material *is* 3/4" (particle board); and a **3/16" up-shear bit** for the removable-panel cutouts, "the UCS is designed to force those parts face down to allow for this" — alternatively a toolset with a 1/2" bit rabbeting room for an 1/8" bit. He also notes that as packaged there are probably **no tool assignments at all**: "search the UCS for TOOLID and you'll find commented out lines, along with a description of what kind of tool we use."

That search yields a usable map, assembled from live and commented lines across the stack. Every value here is **install-specific** — a TOOLID is an index into the local tool catalog and means nothing on another machine:

| TOOLID | Tool, per the source's own annotation |
|---|---|
| `3` | 5 mm |
| `72` | 25/32" bit (commented, `116.ucs:107`) |
| `117` | 3/16" |
| `146` | 3/16" up-shear — annotated `;WS TOOLSET 74` at `119.ucs:480` |
| `0` | let CV choose |

Resolution is centralised rather than scattered: `25.ucs:180-186` reads a user-facing hole-size parameter and derives the tool once —

```
AA_CaseworkWall_PilotHoleSize_UseToolID<int> := 0
if AA_CaseworkWall_PilotHoleSize_Use == imp(5) then
    AA_CaseworkWall_PilotHoleSize_UseToolID<int> := 3
end if
if AA_CaseworkWall_PilotHoleSize_Use == 3/16 then
    AA_CaseworkWall_PilotHoleSize_UseToolID<int> := 117
end if
```

Worth copying as a pattern: one place converts a human choice into a machine index, and everything downstream reads the derived parameter. Note `imp(5)` — the metric-to-imperial helper — used so a 5 mm entry compares correctly.

The 3/4-vs-25/32 distinction is implemented as a **negative mortise sized to the bit**, at `116.ucs:101-107`:

```
;for slightly less than 3/4 studs we want a 3/4 bit and for 3/4 studs we want a 20mm bit.  Lets just make that happen.
;make sure this matches the top/bottom dado code
if :DZ >= 0.75 and :DZ <= (0.75 + 1/64) then
    S_MORTR := :DZ - (25/32)
    ;TOOLID<int> := 72 ;25/32 bit
end if
```

`S_MORTR` goes **negative** — the slot is widened by shrinking the mortise, so the geometry matches an oversized bit. The second comment is a maintenance warning: the same test exists in the top/bottom dado code and the two must stay in sync.

Radius cleats are excluded from CNC behind a user flag — `122.ucs:542-543`: `if Cab.AA_CaseworkWall_SendRadiusCleatsToCNC == 0 then` / `WALLCLEAT._NOCNC<int> := 1 ;we don't let these go to CNC`.

## Verified techniques worth absorbing

Each of these was checked line-by-line against the filed source. They are general classic-UCS technique, not casework-wall-specific, and several are the only known example in this skill.

### The probe part — create a part solely to read a value CV won't otherwise expose

To learn what thickness or material ID a schedule will resolve to, declare the part, read the resolved value off it, and never manufacture it. `25.ucs:232-239`:

```
;first place on wall
dim WALLCLEAT AS NEW PART
WALLCLEAT_DZ := this.WALLCLEAT._M:DZ

dim S_DSLAB as new part
S_DSLAB_DZ := this.S_DSLAB._M:DZ
S_DSLAB_MATID<int> := this.S_DSLAB._M:MatID
_MS11 = _MS1011 ;ucs dimmed door parts take base door schedules, we want upper
```

A zero-quantity variant appears at `136.ucs:4-9`, which is the safer form when the part would otherwise reach a cutlist:

```
dim WALLCLEAT as new part
WALLCLEAT.QTY := 0
WALLCLEAT.DZ = _M:DZ
WALLCLEAT.desc = 'Wall Cleat Thickness Test'
WALLCLEAT_DZ := this.WALLCLEAT.DZ
exit
```

### `FaceCopyOperation` — a zero-size operation as a clipboard carrier

This is the mechanism behind the thread's copy/paste-face-information trick, and it appears nowhere else in this skill. A `line` operation of size zero is created purely so the user has something to copy in the part editor's report view. `84.ucs:43-64`:

```
;preserve copy-able face info
if this.FaceCopyOperation.DX != null then
    ;user copied another face to here, can't create one with the same NAME.
    TEMP_CopyFromObject<text> = 'FaceCopyOperation'
    TEMP_CopyToObject<text> = 'CantCopyThis'
    TEMP_Using_CopyFromObject<int> := 1
else
    TEMP_CopyFromObject<text> = 'Nothing'
    TEMP_CopyToObject<text> = 'FaceCopyOperation'
    TEMP_Using_CopyFromObject<int> := 0
end if

dim {TEMP_CopyToObject} as new line
{TEMP_CopyToObject}.desc = '{FaceName} Copy - After Paste You Must Cause a Rebuild'
{TEMP_CopyToObject}.DX := 0
… .DY .DZ .X .Y .Z all := 0 …
{TEMP_CopyToObject}.UCSMOD<int> := 1
{TEMP_CopyToObject}.PID<int> := -1
```

Three things to notice. The name-collision problem is solved by **deliberately poisoning the target name** to `CantCopyThis` when a pasted operation is already present, so the `dim` still succeeds but lands somewhere harmless. `PID := -1` and `UCSMOD := 1` mark it as UCS-owned and unparented. And the `desc` is used as **user documentation** — it literally instructs the user to cause a rebuild, which is why the thread's instructions end that way.

Jonah Coleman's own description of the workflow: take the control point to the part editor's report view, copy the single operation there, paste it onto another control point's part editor report view, return to the assembly and cause a rebuild.

### Forcing a part face-down for machining

The counterpart to the 3/16" up-shear note. An invisible micro-operation exists only to carry an overwhelming face-down vote, `119.ucs:422-429`:

```
DIM USETHISSIDE as new linebore
USETHISSIDE.desc = 'IGNORE THIS ERROR'
USETHISSIDE._FACEWP<int> := 2
USETHISSIDE.DX := 0.001
USETHISSIDE.DY := 0.001
USETHISSIDE.DZ := 0.001
USETHISSIDE._FUDI<int> := 10000
USETHISSIDE._hide<int> := 7
```

`_FUDI := 10000` is the weight that wins the face-selection arbitration; `_hide := 7` keeps it off drawings; the `desc` warns the user that the error it provokes is expected. Three parameters that only make sense together.

### Finding your own dynamically-named object again

`122.ucs:348-352`:

```
INSTANCE<int> := 1
while this.WALLCLEAT@{INSTANCE}.IDENTIFIER != 'WALLCLEAT{TEMP_Loop1}' do
    INSTANCE<INT> += 1
end while
WALLCLEATRABBET{TEMP_Loop1}.OWNER = WALLCLEAT@{INSTANCE}
delete INSTANCE
```

`@{n}` interpolates into the **instance index**, and `.IDENTIFIER` is the searchable name — so a linear scan can locate an object whose instance number isn't known statically. The loop is unbounded and will not terminate if the object is absent. Note `INSTANCE<int>` declared and `INSTANCE<INT>` incremented: type suffixes are case-insensitive.

**Why the search exists at all** is the important part, and the author says it outright at `122.ucs:378`: `;basically after these blocks below WALLCLEAT will no longer refer to the first dim'd actual wall cleat, so be careful`. A later `dim` of the same base name **rebinds the bare identifier**. Once that has happened, the only way back to the earlier object is by instance index — hence the scan.

### Dynamic object creation, three distinct forms

- `dim WALLCLEATRABBET{TEMP_Loop1} as new dadoex` (`122.ucs:326`) — loop-indexed, so N cleats produce N non-colliding operation names.
- `dim {AA_Take_Part_Material} as new part` (`20.ucs:96`) — the *target* is chosen at runtime; the name comes from a parameter.
- `dim {TEMP_CopyToObject} as new line` (`84.ucs:55`) — runtime-chosen and deliberately redirected on conflict, as above.

### Interpolating the `:` tree-walk operator itself

Depth to the face isn't known statically, so the script probes and then stores the operator as text. `119.ucs:240-247`:

```
if :::OBJECT == 15 or ::OBJECT == 15 then
    if ::OBJECT == 15 then
        AA_Colons_To_Face<text> = '::'
        AA_Colons_To_Below_Face<text> = ':'
    else
        AA_Colons_To_Face<text> = ':::'
        AA_Colons_To_Below_Face<text> = '::'
    end if
```

Used later as `{AA_Colons_To_Face}.TR.AA_FaceCallout_Use` (`119.ucs:272`). The pattern generalises: **if `{}` interpolation happens before parsing, anything textual can be computed — including addressing operators.** Incidentally this pins one `OBJECT` code precisely: the level at which `OBJECT == 15` is what the author calls "Face" is officially labeled **Subassembly** in Hexagon's own published `OBJECT` table (`parameter-glossary.md`) — his own informal name for a cabinet-internal Subassembly structure (Case/Interior/Face), distinct from CV's separate top-level Wall Face concept (`OBJECT == 5`).

### Storing a whole object path in a text parameter as a pointer

`115.ucs:29`: `JONAH_Interior_Split_Above_Path<text> = 'Cab.{JONAH_InteriorName}.{TEMP_Part}@{TEMP_Instance}'`, re-expanded later as `{JONAH_Interior_Split_Above_Path}.PROP`. A resolved address, computed once on the face and dereferenced by any downstream script — the closest thing classic UCS has to a pointer.

### Building a `<lst>` dropdown at runtime

`84.ucs:375`/`:383` accumulate options into a text variable and append them to a static prefix, so the dropdown a user sees reflects how many cleats actually exist:

```
'{TEMP_Cleat_Options}|{TEMP_OptionNumText}) Cleat #… ONLY={TEMP_OptionNum}'
```

Labels are zero-padded (`01)`, `02)`) because **CV sorts the list as text**, so `10)` would otherwise sort before `2)`.

### A deliberate foot-gun guard, also via `<lst>`

`25.ucs:152`:

```
AA_CaseworkWall_Default<int> = '<lst>1) Nope=0|2) Still Nope=0|3) Still Nope=0|4) Still Nope=0|5) I really mean it, not an accident=1'
```

Four of five options map to `0`, so a mis-click can't enable the flag. A UI-level safety mechanism built out of nothing but a dropdown definition.

### Hiding inter-pass storage from the user

`84.ucs:1564-1566` writes a value that must survive to the next pass, then hides it and sorts it out of the way:

```
AA_CaseworkWall_LastFaceName<text> = '{FaceName}'
AA_CaseworkWall_LastFaceName<style> = 2
AA_CaseworkWall_LastFaceName<desc> = 'z DO NOT DELETE UCS STORAGE'
```

`<style> = 2` hides the parameter; the leading `z ` in the description sorts it to the bottom of any list that does show it. The same `<style> = 2` trick is used offensively at `120.ucs:29` — `_MS1001<style> = 2 ;this is the trick` — **overriding a material schedule by hiding its parameter.**

### Disposing of unwanted parts by re-parenting

`120.ucs:17`: `OWNER = Cab.TRASH`. Parts aren't deleted, they're re-homed onto a scrap node.

### A diagnostic part as a UI error channel

`84.ucs:84-87`:

```
dim ERROR as new part
ERROR.ErrorText<text> = 'Face {FaceName} CALLOUT NOT SET'
ERROR.visible = false
ERROR.OWNER = CAB
```

An invisible part whose only purpose is to carry a message to the user. Classic UCS has no logging; this is the substitute.

### Face-jump detection

`84.ucs:125-126`:

```
if this.AA_CaseworkWall_LastFaceName != null and AA_CaseworkWall_LastFaceName != '{FaceName}' then
;this TR has jumped faces.  Well now that's fun.  Probably reshaped, maybe user dragged and dropped.
```

Note the two-clause guard — existence first, then inequality. A single inequality test would fire spuriously on the first pass, when the parameter doesn't exist yet.

### `_EDGWP` as an index into the edge properties

`116.ucs:92-94`:

```
if this._EDGWP > 0 then
    if Y > _EDG{_EDGWP}DY/2 then
        Y := _EDG{_EDGWP}DY
```

"Whichever edge is the working edge, read *that* edge's DY." The working-edge parameter doubles as a subscript into the `_EDG1..4*` family.

### CV 2025.3 regression: `_EDGWP` is no longer a reliable "is this part shaped" test, and it broke exactly this stack

**Provenance: a Hexagon Nexus thread the user shared as two screenshots, no accompanying instruction text — mined per the skill's standing convention. The discussion text is transcribed with confidence; the three inline code screenshots were cross-checked line-for-line against `examples/jonah-casework-wall-10-studs-and-stud-dadoes.txt`, already in this skill, and match it exactly** (lines 30, 78-98, and 361-367 of that file, allowing for the poster's own disclaimer that he'd edited his copy so his line numbers wouldn't match). That independent byte-match is why this is published as a confirmed finding rather than an OCR guess — screenshot-derived code is normally the least trustworthy source this skill accepts, but here the code is verifiable against a file already on file.

**The thread: "`_EDGWP` now used instead of `_FACEWP` on non-shaped parts - intentional?"** (Wes Shimwell, CV 2025 forum). Adding a partition with a dado into the top or bottom, then checking the `PTDADO` object in the Object Tree: in prior CV versions the working-plane parameter came back as `_FACEWP`; in **2025.3** it comes back as `_EDGWP` instead — even on a part that was never shaped. **Toby Richards' reply carries a green "Verified Answer" checkmark** (elevated credibility, not just community speculation): this is a side effect of a new CV feature letting a UCS dim a connection *per edge* (`_EDGWP = 1` rather than `_FACEWP = 3`), which makes `_EDGWP` meaningful on parts that were never shaped. **His fix, stated directly: to test whether a part is actually shaped, use `_SHPEDGCNT > 0` instead of `_EDGWP == 0`.**

**This is not a hypothetical for this skill — it breaks the exact idiom this stack relies on.** `examples/jonah-casework-wall-10-studs-and-stud-dadoes.txt:30` reads `if this._EDGWP == 0 then ;not shaped` (see the `_EDGWP`-as-edge-index section above for the working code this guards). On 2025.3 that condition can now be false on an unshaped part, changing which branch runs. Per this skill's own parameter reference, `_SHPEDGCNT` (`standard-ucs.md`, `parameter-glossary.md`) already exists as the shape-count parameter — Toby Richards' fix is exactly the swap this skill's own glossary entry for `_EDGWP`/`_FACEWP` needs, and it has been added there.

**Wes Shimwell's own reported workaround, for this specific shop's casework walls, is narrower than a general fix and shouldn't be copied as one:** "just commented out the lines that check if it's shaped because we never manually shape our studs." That works only because his shop's studs are never shaped — it is not the general `_SHPEDGCNT > 0` fix Toby gave, and porting "just delete the shape check" into a script whose parts *can* be shaped would silently break the shaped case.

**A second, unrelated defect surfaced in the same reply, worth its own entry because it's a genuinely new failure mode for this skill: a floating-point-zero rounding bug that silently stopped an operation from firing.** Wes: "Also commented out line 365 due to the rounding issue. Even though `TEMP_Topadjust` = 0, it was reducing the stud height by 0.012mm which caused all top plate operations to not fire." The code in question, verified byte-exact at `examples/jonah-casework-wall-10-studs-and-stud-dadoes.txt:361-367`:

```
;adjust height down for lowered faces
TEMP_TopAdjust := Cab.{TEMP_FaceName}.TR.AA_Face_TopAdjust
if Cab.{TEMP_OppFaceName}.TR.AA_Face_TopAdjust > TEMP_TopAdjust then
    TEMP_TopAdjust := Cab.{TEMP_OppFaceName}.TR.AA_Face_TopAdjust
end if
DY -= TEMP_TopAdjust
delete TEMP_TopAdjust
```

Read plainly, `TEMP_TopAdjust` should be exactly `0` when no face is lowered, making `DY -= TEMP_TopAdjust` a no-op. In practice a value that *reads* as `0` in the sidebar can carry residual floating-point error from whatever upstream chain computed `AA_Face_TopAdjust`, and subtracting that near-zero-but-not-quite-zero value shaved 0.012mm off the part's `DY` — small enough to look correct in the model, large enough to break the geometric-contact test a downstream top-plate dado operation depends on (see mental model #4 and `_NORM` in `SKILL.md`/`standard-ucs.md`: an operation that isn't in real contact with its neighbor doesn't fire). This is the same underlying hazard `rprec()` exists to guard against elsewhere in this stack (see "There is no `atan2`, and `rprec()` exists because floats don't compare," above) — a value that *should* be zero after arithmetic is not guaranteed to actually equal zero. If a `DY -=`/`+=` adjustment by a computed value that's "supposed to be zero" is ever suspected of silently killing a downstream operation, `rprec()` the adjustment (or compare it with a small tolerance) before applying it, rather than trusting it to be exactly zero when it should be.

**Neither fix from this thread has been applied to the verbatim source filed in this skill** — per the standing rule, `examples/jonah-casework-wall-10-studs-and-stud-dadoes.txt` is never edited to reflect a fix, even a confirmed one. If a user asks to actually fix this stack for a 2025.3+ install, the two changes are: swap the `_EDGWP == 0` shape test for `_SHPEDGCNT > 0` at line 30 (and anywhere else in the 13-script stack the same idiom recurs — not independently re-checked here), and guard the `DY -= TEMP_TopAdjust` line with a tolerance or `rprec()` rather than deleting it outright, since deleting it (as Wes did) means a genuinely lowered face no longer adjusts height at all.

**Status update, 2026-07-31, from a separate CV2025.4 test session (`jonah-coleman-cv-source` repo work) that independently arrived at the same two fixes:** both changes — the `_SHPEDGCNT > 0` swap and wrapping the `TEMP_TopAdjust` delta in `rprec()` before applying it — have now been applied in that repo's own copy of the affected script, but are explicitly flagged there as **still unverified against real CV2025.4 output**, same open status this file already carried. This isn't new confirmation that the fixes work; it's corroboration that a second, independent effort landed on the identical diagnosis and fix shape from the forum thread alone, and that the remaining step (verifying against real output) genuinely hasn't happened yet in either place. **Still the case as of that pack's 2026-08-01 closeout** — this is the larger of the two remaining open items in that repo's own test plan (the full 13-script stack import is the most CV-experience-intensive item there), left open rather than pursued alongside the rest.

### Memoisation on instance 1, with the author's own performance note

`116.ucs:1226-1228`:

```
;check to see if this dado is contained by another, so we don't drill square outs then
if :.{name}@1.OBJID == OBJID or DY != :.{name}@1.DY or AA_DisableCleats + AA_DisableToeNotch + AA_DisableCounterNotch > 0 then
;first one in the section, we'll do the math here - this is extremely bad for performance so we want to keep ourselves from doing it on each stud
```

Two idioms in one line. `:.{name}@1` addresses **the parent's first sibling carrying my own name** — an "am I the first one?" test that needs no counter. And the three `AA_Disable*` flags are **summed rather than OR-ed**, a cheap boolean-or in a language without one.

### There is no `atan2`, and `rprec()` exists because floats don't compare

Jonah Coleman implements `atan2` by hand **three times** — `122.ucs:82` `;guess I'll have to implement atan2 myself`, `122.ucs:483` `;AGAIN!!!! I'll have to implement atan2 myself`, and a third copy in `117.ucs`. Also `122.ucs:116`: `;atan returns radians, we need to convert to degrees`. If a task needs a quadrant-correct angle, expect to write it.

`rprec()` appears 12 times, always to make an angle comparison survive floating point. `122.ucs:139-141`:

```
;fixes for conditions where CV doesn't line up the interior and face properly... WTF
if rprec(JONAH_Face_AY) == 225 and rprec(JONAH_FACE_TO_LEFT_AY) == 180 then
```

### `OBJECT` as a provenance test

Two places use a **parent-level** `OBJECT` code to ask where the object came from, and the author's comments label the branches. `20.ucs:4`:

```
if :OBJECT == 10 then
    ;copied
    EXIT
end if
```

And `25.ucs:39`:

```
if this.CABNO == null and :OBJECT != 10 and :OBJECT != 37 then
    ;just pulled from library
    delete AA_CaseworkWall_Version
    …
```

So a null `CABNO` combined with a parent that is neither `10` nor `37` means "freshly pulled from the library," and the script uses that to wipe its own version stamps and start clean. These checks read `:OBJECT` — the *parent's* type, one level up. **Correction, 2026-08-05**: this section previously called the codes "unpublished" — that was stale even at the time it was written, since `OBJECT` had already been confirmed as a real Hexagon-published table elsewhere in this skill (`parameter-glossary.md`, `nexus-thread-82060-object-type-vs-class.txt`), just never cross-checked back into this file. Per that table, `10`=Cabinet and `37`=Order — so `:OBJECT == 10` literally means "my immediate parent is a Cabinet" and `:OBJECT == 37` means "my immediate parent is an Order," a plausible mechanism behind the author's practical observation (a freshly copied operation may sit transiently under the Cabinet; an object not yet pulled into a real Order may read some other parent type) that hasn't been independently verified against a real cabinet. The author's own "copied"/"library" comments remain the confirmed practical behavior regardless of the exact mechanism — parent object type is usable as a provenance discriminator, letting one script distinguish a copy from a library placement from an edit-in-place.

## Traps — things that read wrong on first pass

**Indentation is not significant.** `122.ucs:8` and `120.ucs:8` each place `exit` at **column 0** while lexically inside an `if name == '…' then` block. The statement belongs to the `if`. Read as an unconditional top-level exit — which is exactly what the layout suggests — it appears to kill 640 lines of the script; in fact it ends only the ~3 lines remaining in that branch. This misreading happened during this write-up and was caught only by re-reading the enclosing block. **When judging control flow in UCS, find the enclosing `if`/`end if` pair; never infer scope from whitespace.**

**A later `dim` of the same base name rebinds the bare identifier.** See the author's own warning above. Any code between the first `dim` and a later one is reading a different object than code after it.

**`for each` is case-insensitive, and a census that ignores that will be wrong.** A case-sensitive `grep -c 'For Each'` over these 13 scripts reports four with zero matches; the real split is 9 `For Each` / 6 `for each`. Separately, a count of 2 in a file does not mean two loops — in this stack both such cases are **comments containing the phrase** (`;calculate overlays for each door/drawer opening` in `19.ucs`, `;Single for each cab BEFORE BUILD` in `25.ucs`). With that corrected, the skill's existing rule holds across all 37 real production scripts now on file: **exactly one real `for each` per script, always the first executable line, never terminated.**

**Parameter arrays are emulated, and adding a field means editing three scripts.** See the next section; the author's own capitalised warning is at `84.ucs:539`.

## Parameter-array emulation with a `-1` sentinel command channel

Classic UCS has no arrays, so the cleat set is stored as flat numbered parameters: `AA_Cleats_{N}_Width`, `_AFF`, `_Inset`, `_PLOW`, `_ADJL`, `_ADJR`, `_ADJT`, `_ADJB`, `_NoPart`, `_Breaks_Count`, `_Strut_*`, `_Rabbet*`, `_Closure*`. `AA_Cleats_Number` holds the length. Existence is tested with `this.X != null`.

Insert and delete are implemented as **command slots**: `AA_Cleats_Modify_Insert` and `AA_Cleats_Modify_Delete` carry a slot number, are acted on, and are then reset to `-1` to mark them consumed. `136.ucs` is the delete half (shift every field down), `137.ucs` the insert half (shift up, then `delete` each parameter of the vacated slot) — which is why those two scripts sit at opposite ends of the Face Control group in the stack order.

The cost of the design is stated by the author at `84.ucs:539`: `;REMEMBER WHEN ADDING ANY NEW ATTRIBUTES IN HERE TO ADD TO INSERTING AND DELETING CODE`. Adding one field means touching the consumer, the inserter and the deleter.

The loop is written to tolerate the count and the population disagreeing (`84.ucs:542`):

```
while this.AA_Cleats_{TEMP_Loop1}_Width != null or TEMP_Loop1 <= Cab.{AA_FollowFace_Studs}.TR.AA_Cleats_Number do
```

An `or` rather than an `and` — it keeps going if either the data or the counter says there's more. Defensive, given that a UI edit can leave the two out of step.

## Cross-UCS parameter namespaces

Two namespaces, with different scopes and different owners.

**`JONAH_*` is published on the face or interior** and read by everything downstream: `JONAH_FACE_Opposite`, `JONAH_Face_AY`, `JONAH_FACE_TO_LEFT_AY`, `JONAH_EndX`/`EndZ`/`MidX`/`MidZ`, `JONAH_InteriorName`, `JONAH_FaceName`, `JONAH_Interior_Split_Above_Path`.

**`AA_*` lives on the cabinet or the TR** — roughly 130 names, including the `AA_Disable*` flag family and the version stamps `AA_CaseworkWall_Version` (2.8) / `_VersionLast` / `_VersionCreated`, which is how the system detects its own upgrades.

**The opposite face is not a CV property.** It is computed geometrically in `125.ucs:77-83` as the nearest face 180° opposed, published as `JONAH_FACE_Opposite`, and then dereferenced with a two-step cast at `116.ucs:344-345`:

```
TEMP<int> = Cab.{TEMP_FaceName}.JONAH_FACE_Opposite
TEMP_OppFaceName<text> = '{TEMP}'
```

Read as a number, re-stringified, then interpolated as an address. Jonah Coleman's radius pitfall #4 (below) is the user-facing symptom of this computation failing on odd shapes — and the "override each face and specify which is the opposite face" workaround is possible precisely *because* the value is a writable published parameter rather than something CV owns.

`125.ucs:27` also notes the memoisation: `;we already calculated this on that face, use the stored value`.

## `Cab.Is_Casework_Wall` — the integration point, and a closed open question

`25.ucs:28-32` sets the flag that gates the entire rest of the stack:

```
if Cab.ConstID == 20 or Cab.ConstID == 28 or Cab.ConstID == 36 or this.Is_Casework_Wall == 1 then
    Is_Casework_Wall<int> := 1
```

It is consumed in `116`, `117`, `119`, `120`, `122`, `136` and `137` — and, importantly, in already-filed material this bundle had nothing to do with: `examples/jonah-keku-panel-clips.txt:190` reads `if Cab.Is_Casework_Wall == 1 then FWP_PanelClip_Placement<int> := 3`. That independently corroborates the thread's claim that "this UCS integrates tightly with my KEKU clips UCS and my miterfolding UCS."

**This line also closes a question the skill previously flagged as unverified: `Cab.ConstID` does return an ID from the `Construction` / Construction-Method table.** The bundle ships `20.ccm` = Construction Method **"Casework Wall"**, and `Package.lst` independently records that entry as extension `ccm`, display name `Construction Method: Casework Wall`, plain name `Casework Wall`, object ID `20`. Three-way agreement inside one self-consistent source: the filename numeral, the manifest's object ID, and the author's own runtime test. See `catalog-export-format.md`.

**The caveat survives the confirmation.** IDs `28` and `36` are tested by the same line and those construction methods are *not* in the bundle, so `ConstID` values remain **install-specific** — exactly as already recorded for `ToolID`. Never port a `ConstID` literal between installs.

## Jonah Coleman's five radius pitfalls

From his own verified reply in the thread, prefaced with "This file was sent to me by someone running into some common pitfalls in CV with radius items." These are CV behaviours, not bugs in his UCS, and four of the five apply to any radius work regardless of this system:

1. **If the inside and outside faces are not exactly concentric, studs come out non-tangent to the arc.** His fix: never trust CAD snapping — draw one face and use the **offset tool** to derive the other, so concentricity is guaranteed. The same applies to straight edges: faces that are not *precisely* the same angle produce "funky behavior (infinitely long interiors, etc)."
2. **Placing studs inside radius faces is a one-shot.** After the first sectioning, newly added studs don't appear. Place every stud needed on the first sectioning, then move them.
3. **Sometimes one side of a radius face is simply unworkable.** Use the other — it doesn't matter which face the studs are placed from.
4. **On odd shapes CV sometimes fails to identify the opposite face**, so studs don't get machining from both sides. Override it per face and specify the opposite face explicitly (see the `JONAH_FACE_Opposite` note above for why this is possible).
5. **On odd shapes, studs can run through multiple sections.** Set the **face priority** of the in-between section to 1 (or higher than both ends); that stops the run-through once studs are added to it.

**A separate radius limitation, from the same post:** radius-shaped struts **do not reach S2M with a shape**. His workaround is to double-click the part (UCS-added parts became selectable in v10), take it to the part editor, and copy its shape to the clipboard for use in S2M — or to use that shape to build manual parts. He notes he uses another program to auto-shape them.

## Other author comments worth reading before editing anything

- `117.ucs:1` — `;This should be before any other part level UCS`. An ordering contract expressed only as a comment; nothing enforces it, and the shipped stack contradicts it.
- `20.ucs:114` — `;these parameters do not come through to S2M and we need them for unrolling.` A named, accepted export gap.
- `115.ucs:85` — `;I don't know why it does this some times`. The author's own admission; any mechanism explanation here would be reconstruction.
- `116.ucs:576-577` — `;except if the there is a top plate within this cleat's space - can't do that!  Think like a desk with a raised and a lower section.` / `;problem here is there could be a cleat on the outside... hmmm....` An **unresolved case, admitted in the source.** Don't assume the raised/lowered-section geometry is handled. **Permanently declined for testing, per the `jonah-coleman-cv-source` git pack's 2026-08-01 closeout** — a niche edge case with no clear repro path on this shop's setup, left as a known gap to route around if actually hit rather than something chased preemptively. Closed as won't-test, not left pending.
- `119.ucs:466` — `REMOVABLEPANEL.GROUP<int> := 1000 ;AFTER THE OUTLINE.` `GROUP` as explicit operation ordering.

## Videos

Six screencast links are recorded in `full-docs/nexus-thread-1106-jonah-rocks-millwork-walls.txt` — five in the first post and one attached to the radius-pitfalls reply. They are 2018-era Screencast.com URLs and have **not** been fetched or verified as live. Bill Northrup's reply in the thread is worth knowing about before recommending them: "The videos go a bit too fast right off the start, with no audio."

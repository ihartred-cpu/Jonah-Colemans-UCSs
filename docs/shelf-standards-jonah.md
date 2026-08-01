# Jonah Coleman's Shelf Standards UCS — techniques, defects, and what it settles

Source: `full-docs/jonah-shelf-standards-rev17-raw.txt`, supplied by the user 2026-07-31. Author: Jonah Coleman — the header names him, and the commented library path `FWP\Hardware Visuals\Shelf Standard` plus the `JCS_` parameter prefix both match his known fingerprint (see `examples/README.md` on prefixes).

**Read the capture note before quoting line numbers: there are none.** The paste arrived with its line breaks collapsed into runs of spaces — the forum copy/paste mangling Jonah Coleman himself warns about in `full-docs/nexus-thread-notch-construction-rev12.txt` ("Yep use internet explorer"). Because a UCS `;comment` runs to end of line, reconstructing the breaks would mean guessing where a trailing comment ends and the next statement begins, so the script is filed verbatim in `full-docs/` rather than byte-exact in `examples/`. **Every citation in this file is therefore by construct, not by line.** If a clean copy ever turns up — exported from UCS Manager, or re-copied in a browser that preserves breaks — it belongs in the examples/ folder as jonah-shelf-standards.txt and this file's citations can be tightened.

**Revision numbering:** header says REV 17, the revision history at the foot runs through Revision 18 ("Add wizard sizes (centered)"), and the length-type choice list does carry a `6) Wizard Size (Centered)` entry — so the Revision 18 work is in the body while the title still says 17. Recorded, not resolved.

## What the script does

Shelf standards are the metal pilaster strips let into a cabinet's ends and partitions to carry adjustable shelf clips. CV's assembly wizard produces the line-bore holes; this UCS replaces or supplements them with a dado (or a centerline route) sized to accept the standard, optionally creates a visual/cutlist part for the standard itself, and neutralizes the system line-bore that would otherwise still machine.

It runs as one `For Each` over a name list that mixes part names with an operation-name wildcard:

```
For Each LU|LF|S_LSE|RU|RF|S_RSE|PT|S_DIV|L?VBORE Part
```

Left/right uppers, left/right finished ends, left/right section ends, partitions, section dividers, and `L?VBORE` — the wildcard form that the user's own `examples/linebore-attributes.txt` also uses. This script additionally shows what `?` resolves to, because it branches on the concrete names to label the operation:

```
if name == 'LFVBORE' then
    ShelfStandardDado.desc = 'Shelf Standard - Front'
else
    if name == 'LRVBORE' then
        ShelfStandardDado.desc = 'Shelf Standard - Back'
    else
        ShelfStandardDado.desc = 'Shelf Standard - Mid'
    end if
end if
```

So `LFVBORE`/`LRVBORE` are front and rear vertical line-bores, with anything else matching the wildcard treated as a mid bore. That last `else` is a catch-all, not a name — don't read a third literal name out of it.

## The technique that makes it interesting: one script, two object levels, dispatched on `OBJECT`

The single most reusable thing here is the opening dispatch. The same `For Each` body handles both the *part* and the *operations on that part*, and `OBJECT` is what tells them apart:

```
if OBJECT == 17 then
    ;parts
    ...publish the three part-level override attributes...
    exit
end if

if :JCS_Use_ShelfStd == 0 then
    exit
end if
;operations only from here on down
```

`;parts` is Jonah Coleman's own comment, and `;operations only from here on down` is his too. That makes this a **third and fourth independent attestation of `OBJECT == 17` meaning a Part** — it already appears in `examples/jonah-visible-part-splits.txt` and `examples/jonah-casework-wall-12-radius-blank-panel-inset-fix.txt` — and, unusually, one where the author labels the branch himself rather than leaving the code to imply it. The caution in `standard-ucs.md` about `OBJECT` codes being empirically observed rather than published still stands; what strengthens is specifically 17-as-Part, and what's genuinely new is the **use** of it: a part-name filter that includes operation names yields both levels, and `OBJECT` is the discriminator. That is a cheaper structure than the two-script split the corpus otherwise favours.

Note the addressing shift across the boundary. Inside the `OBJECT == 17` branch the attributes are written bare (`JCS_ShelfStd_Use<int> = …`) and probed with `this.` (`if this.JCS_ShelfStd_Use == null then`). Below it, the same value is read with a single leading colon (`:JCS_Use_ShelfStd`, `:JCS_ShelfStd_Length_Over`) because the operation's parent *is* the part.

## The Automatic / Enabled / Disabled override triad

Each of the three part-level attributes is published only if absent, styled visible, and given a sortable numbered `desc`:

```
if this.JCS_ShelfStd_Use == null then
    JCS_ShelfStd_Use<int> = '<lst>1) Automatic=-1|2) Enabled=1|Disabled=0'
    JCS_ShelfStd_Use<style> = 1
    JCS_ShelfStd_Use<desc> = 'Shelf Standards: 01) Enable'
end if
```

`Shelf Standards: 01)` / `02)` / `03)` is the same desc-prefix-plus-number convention as the `Notch: ` prefix in `notch-construction-jonah.md` — a namespace and a sort key in one, because the attribute list sorts on `desc`.

The `-1` sentinel is the point. `Automatic` doesn't mean "decide cleverly"; it means "fall through to the UCS-level default":

```
JCS_Use_ShelfStd<int> := JCS_ShelfStd_Use
if JCS_Use_ShelfStd == -1 then
    JCS_Use_ShelfStd<int> := ucs_ShelfStd
end if
```

This is the four-line default-to-per-part-override idiom already recorded from the notch script, here generalized to three states instead of two. The same shape governs length type, whose `<lst>` carries `0) Automatic=-1` alongside six real modes.

**Defect: `ucs_ShelfStd` is never declared.** It appears exactly once in the whole script — in the line above — with no `Public ucs_ShelfStd` anywhere. So on a stock copy, choosing `Automatic` resolves the enable flag against a parameter that doesn't exist. The parallel default *is* declared for length type (`ucs_Dado_LengthType`), so this reads as an omission from the Revision 17 work that added the part-level overrides, not a deliberate design. Anyone installing this script should add the missing `Public` before using `Automatic`.

**Cleanup on disable.** When the part is switched off, the two dependent attributes are removed rather than left stale:

```
if JCS_Use_ShelfStd == 0 then
    ;clean up
    delete JCS_ShelfStd_LengthType
    delete JCS_ShelfStd_Length_Over
    exit
end if
```

That's progressive disclosure applied to *part attributes*: a disabled part shows one control instead of three. It matters because published parameters are sticky — CV keeps a parameter once a UCS creates it, so the `delete` is the only thing that makes the UI shrink again.

## `Public` is a runtime statement, not a header

This script demonstrates that more thoroughly than anything else in the corpus. `Public` declarations appear in three distinct places:

- A block near the top for the always-relevant settings (dado width, depth, visual toggles, oversize, part type).
- **Conditionally**, inside `if TEMP_UCS_Dado_LengthType == 2 or TEMP_UCS_Dado_LengthType == 3 then` — the "less than smallest" behaviour list and all thirty `ucs_Use_Sizes_SizeNN` / `_MatID` parameters. They only exist when a set-sizes mode is selected.
- **Far down the body, mid-flow** — `Public ucs_Operation_Type` immediately before the `dim`, and `Public ucs_ToolID_For_Dado` immediately before the TOOLID assignment, each declared at its point of use.

The conditional block is the one worth stealing: it keeps a thirty-parameter table out of the UCS parameter list entirely unless the user has chosen a mode that needs it. The cost is that the parameters vanish when the mode changes, so a value typed under one mode is not guaranteed to survive a round trip through another.

Both `<lst>` spellings appear, in one script: quoted (`= '<lst>1) Automatic=-1|…'`) on the part attributes, and bare with a trailing comment (`Public ucs_Dado_Display_Visual<int> = <lst>False=0|True=1 ;Display a part visual`) on the Publics. The trailing comment after a bare `<lst>` is the comment-as-prompt convention — the text after `;` becomes the parameter's UI label.

One inconsistency inside the lists: in `ucs_Dado_LengthType` the final entry is written `6. Wizard Size (Centered)` with **no `=6`**, unlike its five siblings, and `;Dado Type` follows immediately. Its part-level twin `JCS_ShelfStd_LengthType` does assign `6) Wizard Size (Centered)=6`. Recorded as observed; whether CV assigns entry 6 an implicit value or the entry is inert was not tested.

## Parameter-array emulation with a zero-padded name walk

The corpus already has runtime-built parameter names (`{TEMP_PartType}@{TEMP_Loop2}` in the notch script, `{AA_ParmPrefix}` in miterfold). This is the first instance of a **zero-padded** array, and the padding is handled by flipping a text variable mid-loop:

```
TEMP_Loop1<int> := 15
TEMP_ZERO_PAD<text> = ''
while TEMP_Loop1 >= 1 and TEMP_Use_Size == 0 do
    if TEMP_Loop1 == 9 then
        TEMP_ZERO_PAD<text> = '0'
    end if
    if this.ucs_Use_Sizes_Size{TEMP_ZERO_PAD}{TEMP_Loop1} > 0 and DY >= this.ucs_Use_Sizes_Size{TEMP_ZERO_PAD}{TEMP_Loop1} then
        TEMP_Use_Size := ucs_Use_Sizes_Size{TEMP_ZERO_PAD}{TEMP_Loop1}
        TEMP_Use_MatID<int> := ucs_Use_Sizes_Size{TEMP_ZERO_PAD}{TEMP_Loop1}_MatID
    end if
    TEMP_Loop1<int> -= 1
end while
delete TEMP_Loop1
```

Four things to take from it. The loop **descends** from the largest index and the `while` condition carries `and TEMP_Use_Size == 0`, so it stops at the first (largest) size that fits — a "biggest that fits" search with no separate break. The pad flips at 9 and stays flipped, which is correct for a descending walk and would be wrong for an ascending one. Two interpolations can be concatenated (`{TEMP_ZERO_PAD}{TEMP_Loop1}`) and a suffix can follow one (`…{TEMP_Loop1}_MatID`). And the probe uses `this.` while the read does not — `this.` to test existence and value, bare to fetch.

**Defect: the fallback reads unpadded names.** When nothing fits, behaviour 1 ("Use minimum size anyway") does:

```
TEMP_Use_Size := ucs_Use_Sizes_Size1
TEMP_Use_MatID<int> := ucs_Use_Sizes_Size1_MatID
```

`ucs_Use_Sizes_Size1` and `ucs_Use_Sizes_Size1_MatID` each appear exactly once in the script — here. The declared parameters are `ucs_Use_Sizes_Size01` and `ucs_Use_Sizes_Size01_MatID`. Unless CV normalizes numeric suffixes (nothing in the corpus suggests it does), behaviour 1 reads null and the other three behaviours (change to wizard/stop/through dado) are the only ones that work. This is the zero-padding scheme biting its own author, and it's a good argument for padding *or* not padding consistently rather than mixing.

The three non-default behaviours work by **rewriting the mode variable** rather than branching — `TEMP_UCS_Dado_LengthType<int> := 1` / `:= 4` / `:= 5` — so control falls into the already-written wizard/stop/through code below. Revision 16's history entry records that this didn't originally work for the through-dado case.

## The created part's *type* lives in a text parameter

```
public ucs_Dim_Part_Type<text> = CHROME ;Part Name to create for cutlists- MUST BE ALL UPPER CASE
...
dim {ucs_Dim_Part_Type} as new part
{ucs_Dim_Part_Type}.name = 'Shelf Standard {TEMP_Use_Size}'
;{ucs_Dim_Part_Type}.LibPart = 'FWP\Hardware Visuals\Shelf Standard'
```

Every subsequent property reference goes through the same interpolation. This is one level beyond the `{AA_ParmPrefix}` trick: not a parameter name assembled at runtime but the **`dim` identifier itself**, which is what determines the part type CV files the created part under for cutlist and material-schedule purposes. Making it a Public means one script can feed different shops' part-type vocabularies without editing code — Revision 15's history entry calls this out as its purpose ("Add part type parameter to control what part gets created").

The all-caps constraint is the author's, stated as a requirement and not explained. Take it at face value; it is consistent with part-type names being uppercase throughout the corpus, and note it does **not** conflict with identifiers being case-insensitive (below) — the constraint is presumably on the *type lookup*, not the identifier.

`.LibPart` is present but commented out, with a header note telling the user to uncomment two lines and repoint the path. When it stays commented, the part is a plain box, and `ucs_Dado_Display_Visual == 0` sets `.visible = false` so it exists for the cutlist without appearing in 3D — the same "part for data, not for looks" pattern as the probe-part idiom in `casework-walls-jonah.md`, but persisted rather than measured and discarded.

## Identifier case-insensitivity, multiply attested

The skill already records that `for each` is case-insensitive. This script shows the same is true of identifiers and property names, because it mixes casings for the same thing and works anyway:

- `TEMP_Use_Size` and `TEMP_USE_Size` (also `TEMP_USE_DY`, `TEMP_USE_SIZE`, `TEMP_USE_Y`)
- `TEMP_UCS_Dado_LengthType` and `TEMP_UCS_Dado_Lengthtype`
- `ShelfStandardDado` and `ShelfStandardDAdo`
- `.MatID` and `.MATID`
- `Public` and `public`

Each variant appears in a position where a distinct new variable would break the logic — `ShelfStandardDAdo.DX := 0` is the route branch's only assignment of the operation's width, and `TEMP_USE_SIZE` is the only place the visual part's `DY` gets set. So these are typos that are harmless *because* the language folds case, which is a stronger demonstration than a deliberate example would be. Practical consequence: don't chase a "missing" variable in his code on a case mismatch, and don't rely on case to distinguish two parameters.

## The "1 CNC file" through-bore workaround

The detector is a dimension comparison, not a setting query:

```
TEMP_Fix_For_1_CNC_File<bool> := false
if DZ = :DZ then
    TEMP_Fix_For_1_CNC_File<bool> := true
    if TEMP_Use_FACEWP == 1 then
        TEMP_Use_Reverse_FACEWP := 2
    else
        TEMP_Use_Reverse_FACEWP := 1
    end if
    ucs_Dado_Depth := ucs_Dado_Depth_BothSides
end if
```

An operation whose depth equals the part's own thickness has become a through operation, which is what "generate 1 CNC file" does to a two-sided bore. The response has three parts: swap to a separate both-sides depth parameter, flip `_FACEWP` between 1 and 2 to get the mirrored work plane, and later emit a second `Dadoex` on that reverse face with the X coordinate mirrored about the part width:

```
ShelfStandardDadoReverse.X := :DX - (X - ucs_Dado_Width/2)
```

The visual part also gets `QTY := 2` in this case, since a through condition means a standard on each face. Revision 7's history entry dates the workaround and calls it "a work-around for CV bug where 'generate 2 cnc files' doesn't work" — so this is a CV defect being papered over in user code, and worth checking whether it still reproduces before copying the complexity.

The header records a limitation the workaround does not cover: *"Part visual does not work with '1 cnc file' through boring on partitions/dividers."*

**Two ordering observations, flagged as read from a collapsed capture rather than confirmed:**

- `TEMP_Use_FACEWP` is *read* in the block above but only *assigned* much later (`if _FACEWP > 0 then TEMP_Use_FACEWP := _FACEWP`), and it is `delete`d at the end of the script. So the reverse-face decision appears to test a parameter that has no value yet on this pass. It resolves to the same answer either way when `_FACEWP` is 2, which may be why it has never surfaced.
- Inside the block that otherwise configures `ShelfStandardDadoReverse`, the through-dado case sets `ShelfStandardDado.S_EXTND := true` — the *forward* operation. Given every neighbouring line targets the reverse op, this looks like a copy-paste slip, but extending the forward op could also be intentional. Not tested.

Both are the kind of claim that needs the real line breaks to state firmly. Recorded here so a future clean copy can settle them.

## Neutralizing the host instead of deleting it

The script ends by shrinking the line-bore it rode in on rather than removing it:

```
DX := 0
_NOCNC := 1
Z := -:DZ + 0.01
DZ := 0.001
exit
```

Revision 13's history entry explains why: *"System operations are left in place to allow for editing in the CAM editor."* Deleting the system bore would remove it from the operator's reach; zeroing its width, marking it `_NOCNC`, and pushing it just outside the part leaves a handle in the CAM editor while guaranteeing it machines nothing. Compare the `EDX = -DX` cutlist-exclusion trick in `standard-ucs.md` — same instinct (make it dimensionally inert rather than absent), different target.

## Operation type: a documented CV bug in the history

```
Public ucs_Operation_Type<int> = '<lst>Dado=1|Centerline Route=2' ;Operation Type
if ucs_Operation_Type == 1 then
    dim ShelfStandardDado as new Dadoex
else
    dim ShelfStandardDado as new line
end if
```

Two operation classes behind one identifier, with the route branch distinguished downstream by `DX := 0` (a route has no width; a dado does). Revision 10's history entry records that this **does not actually work**: *"currently does not work due to bug in CV, somehow it gets dim'd as a dado even if it is set to route."* Undated relative to any CV version, and unresolved in the history through Revision 18 — so if a `dim … as new line` in a conditional comes out as a dado, that's a known CV behaviour with a decade of provenance, not new breakage. Worth probing on a current install.

For the dado case the centerline is shifted by half the width using the part's own rotation, the same `sin`/`cos` projection style the rest of Jonah Coleman's code uses in the absence of an `atan2`:

```
TEMP_Use_X := TEMP_Use_X - sin(AZ+90)*(ucs_Dado_Width/2)
TEMP_Use_Y := TEMP_Use_Y + cos(AZ+90)*(ucs_Dado_Width/2)
```

## Length derived from the line-bore, and the horizontal-grain switch

The default dado length is read off the bore pattern it replaces:

```
TEMP_Use_DY := (REPT-1)*SPCNG
```

`REPT` and `SPCNG` are the line-bore repeat count and spacing (see `parameter-glossary.md`), so the dado spans exactly the bored range. A part-level `JCS_ShelfStd_Length_Over > 0` overrides it, and the centered modes recover the difference against the original span before re-centering.

Grain direction switches which of the part's dimensions bounds a through or stop dado:

```
TEMP_Horizontal_Grain<int> := 0
if _CB:525 == 1 then
    TEMP_Horizontal_Grain<int> := 1
end if
```

This is the **second attestation of the `_CB:NNN` numeric cabinet-attribute reference** in the corpus, and the first in bare form — `examples/jonah-casework-wall-01-cab-before-build.txt` uses the qualified `Cab._CB:622`. Here it is unqualified from part context, which means the bare form resolves up the tree the same way other bare parameter reads do. `525` is presumably a horizontal-grain flag on the cabinet; the number is install- or version-specific and should not be reused without checking. The `_CV:NNN`/`_CB:NNN` resolution route via the Assembly Manager wizard is covered in `full-docs/ucs-breakdown.txt`.

Downstream, horizontal grain swaps `:DX` for `:DY` as the span and `TEMP_Use_X` for `TEMP_Use_Y` as the origin. Revision 17's history entry — "Support for horizontal grain panels" — dates this as the newest structural work in the script.

## What the revision history is worth reading for

The 18-entry history is unusually informative about CV itself, not just the script:

- **Revision 8** removed the entire mid-shelf-standard feature "since assembly wizard now creates mid line-bores" — a CV capability arriving and letting user code shrink. The inverse of the usual direction.
- **Revision 9** — "Added exit function to beginning of UCS for performance." Early exit as a deliberate optimization, which is what the `OBJECT == 17` and `:JCS_Use_ShelfStd == 0` guards are doing. Consistent with the same author's practice in the casework-wall stack.
- **Revision 15** — "Change to use in-line parameter evaluation," the same CV/Solid 8.0 feature that collapsed the notch script from 1350 lines to 420 (see `notch-construction-jonah.md`). This script's zero-padded name walk is only possible because of it.
- **Revisions 11 and 14** are all part-visual orientation fixes (wrong position with unequal front/back spacing; facing into the bottom of the dado; rotating the wrong way when the bore is rotated). Placing a visual part into an operation's coordinate frame is evidently the hardest part of this script to get right, which matches the volume of `AX`/`AY`/`AZ`/`_FACEWP` juggling in the body.
- **Revisions 3 and 8** are both about making the parameter UI legible (adding `(0=false, 1=true)` notations, then replacing them with real choice lists, then collapsing many booleans into one mode variable). The design pressure runs toward fewer, richer controls.

## Open questions this raises

- Does CV normalize `Size1` to `Size01`? If it does, the fallback defect above evaporates. Cheap to probe.
- Does entry `6. Wizard Size (Centered)` without an `=6` receive an implicit value in a bare `<lst>`?
- Does the `dim … as new line` / comes-out-as-dado bug from Revision 10 still reproduce on a current install?
- Does the "generate 2 CNC files" bug from Revision 7 still reproduce, or is the whole `TEMP_Fix_For_1_CNC_File` apparatus now dead weight?
- What is cabinet attribute `525` called in the UI, and is it stable across versions?
- Is `ucs_ShelfStd` declared in the user's installed copy (i.e. is the missing `Public` a paste casualty or a real omission)?

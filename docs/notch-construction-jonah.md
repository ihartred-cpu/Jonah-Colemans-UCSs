# Notch Construction — Jonah Coleman's Revision 12, and what its thread establishes

`scripts/notch-construction/jonah-notch-construction.txt` is one of the most widely used community UCS scripts CV has: 446 lines that make intersecting interior parts (partitions, drawer stretchers, fixed and adjustable shelves) notch through each other automatically, by detecting the intersection geometrically and adding a matched dado / master-dado pair to both parts so they slide together. Harley's one-line explanation on the thread is the clearest anyone managed: "there is a notch in the front of the partition and a notch in the back of the shelf allowing them to slide together."

The script itself was already on file. What this page adds is everything the **thread around it** establishes — the version history, the known breakages, the language facts the code demonstrates but nobody had written up, and one long-standing open question that it closes. Thread archive: `source-threads/nexus-thread-notch-construction-rev12.txt` (partial capture — the thread has 127 replies and the paste holds roughly 30).

## Version, vintage, and the thing to check first

The file header says **Revision 12** and `;This UCS can only be used with versions of CV >= 8`. Jonah Coleman's own opening post says "This is revision 11" — the post was edited in place as he shipped revisions and the prose fell out of sync with the code. The revision history at the end of the file is the authority; it runs to 12 and the last entry is `;Add parameter to change notch size per Part.`

**Revision 9 is the interesting one, and it dates the whole design.** Its history entry: `;Update to use new version 8 inline parameter evaluation - now supports unlimited number of sectionings`. His post puts numbers on it — the code went **from 1350 lines to 420** and gained unlimited intersection support, because CV/Solid 8.0 added runtime `{}` parameter interpolation. He credits it to Chip ("Somebody buy Chip a case of beer or something") and marks the spot in the code at line 168: `;new awesome inline parameter evaluation, thanks Solid!`. That is worth knowing when reading *any* pre-CV8 UCS: the unrolled if-ladders you see in old scripts are not bad style, they are what the language required before `{}` existed.

**This script has a forum-reported, previously unfixed CV2023 breakage — now with one real data point against it.** Eric@bloomsburydesign.com, roughly two years before capture: the **drawer stretcher origin moved**, from Top Back Left in CV11 to Bottom Front Right in CV2023, so the extended drawer stretcher now sticks out the left side of the cabinet. No reply to that post is present in the capture, and no newer revision is referenced anywhere in it. The general lesson generalises past this script: a part-origin change between CV majors silently reverses the sign of every position expression written against it, and nothing errors.

**Direct corroboration this is a real, live concern in this shop's own install, from a screenshot of the UCS Manager's Public Variables pane** (see the cabinet-vision skill's `standard-ucs.md` for the full context of that screenshot): with this exact script selected, its public parameters include **`CV2023+ DS origin compensation (untested) = False`** — a toggle whose name reads as a direct attempt to compensate for the drawer-stretcher-origin regression above (`DS` most plausibly "Drawer Stretcher," matching the failure mode Eric reported). This confirms someone (the user or Jonah Coleman) took the CV2023 regression seriously enough to add a compensation switch to the script rather than leaving it purely as a forum report, and confirms that switch defaults to **off** (`False`) in this install.

**Updated 2026-07-31, from real click-testing on this shop's own CV2025.4 install: the regression does not reproduce with the switch left at its shipped default (off).** With no compensation applied, an "entire interior"-sized drawer stretcher stayed correctly within the interior frame — it did not stick out the wrong side. This is a genuinely useful data point, but read it precisely: it shows the forum-reported regression is **not universal across CV2025 installs/builds**, not that Eric's report was wrong or that the compensation switch is unnecessary everywhere. **Confirmed not needed on this install, per the `jonah-coleman-cv-source` git pack's closeout:** the flag stays in the script, still off by default, in case a different CV2025.4 build/config ever hits the forum symptom — but nothing further to test here. Whether the compensation switch itself does anything when turned on remains untested (its own name still says `(untested)`); the switch's logic wasn't exercised, only the default (off) path. If a user on modern CV *does* hit the drawer-stretcher symptom, don't assume a workaround is needed by default — check the real cabinet first — but if it does show up, this parameter is still the first thing to point them at, and flipping it is still untested.

**Two other unresolved reports in the thread**, both left hanging, both worth recognising if they recur:

- **Bill Crouch**: notch and outline cuts both come out of the NBR at .502" depth in a single pass on 1" material, where 1.004" was wanted. Tool data verified between CV9 and the NBR, max depth of cut set to 25/32", same tool assigned to both operations. Nobody answered. Note the coincidence worth checking first: .502 ≈ half of 1.004. **Explicitly declined as not worth chasing further, 2026-07-31:** flagged during the same CV2025 testing session as tool-catalog-specific to Bill Crouch's own shop setup, not reproducible without that exact tool configuration — left as a known gap to route around if a similar report ever recurs, not something to fix speculatively.
- **Palmer**: after changing FS and PT material to 1/4" by room, the dados into sides/top/bottom stayed at 5/8" — the width they would be for 3/4" material with a qualified dado. Switching the room construction method away from qualified dado fixed it. Jonah Coleman's answer is a useful diagnostic instinct rather than a fix: *are you asking about the dadoes my UCS creates or the system ones?* Those are two different subsystems and conflating them makes the problem unanswerable.

## Scope limit, stated by the author

**It does not work on angled parts.** Adam Mauss asked directly about shelves set at an angle; Jonah Coleman: "No. It is premised on standard orientations." Mark Sams then hit the same wall at 20° — he could angle the partitions but the notch stayed put. Jonah Coleman's reply is the fullest statement of the limit:

> I did a lot of work at one point on a version that could handle arbitrary angles, and lost that work in an unfortunate accident involving my wiz_Data.wdb file. It is *not* easy.

Two things to take from that. Arbitrary-angle notching **exists nowhere** — don't go looking for a version that handles it, and don't assume it's a small extension. And `wiz_Data.wdb` is named here as **the file that holds UCS source** — losing it lost the work. It is the only mention of that filename anywhere in this skill, and it is the reason UCS scripts should be exported and version-controlled outside CV rather than lived in.

Mark Sams' partial workaround, for the record: he got the dado in the *partition* to follow the angle, but not the one in the fixed shelf, and finished the job by editing each part by hand.

## Closed: what `CO` is

The cabinet-vision skill's `standard-ucs.md` has carried `Cab.Interior.CO` as an **open question** — attested only in one of the user's own scripts, absent from that skill's `internal-part-names.md` and `object-intelligence.md`, guessed at as "most likely the interior core/opening region node." This script settles it. Lines 93-107:

```
TEMP_Loop1<int> := 1
TEMP_Parent_X<meas> := 0
…
while CO@{TEMP_Loop1}.PID > 0 and TEMP_Parent_DX == 0 do
    if CO@{TEMP_Loop1}.ID == PARID then
        TEMP_Parent_X<meas> := CO@{TEMP_Loop1}.X
        TEMP_Parent_Y<meas> := CO@{TEMP_Loop1}.Y
        TEMP_Parent_DX<meas> := CO@{TEMP_Loop1}.DX
        TEMP_Parent_DY<meas> := CO@{TEMP_Loop1}.DY
    end if
    TEMP_Loop1<int> += 1
end while
```

**`CO` is the sectioning / cut-out opening object, and it is an indexed collection** — `CO@1`, `CO@2`, … enumerable in exactly the same way as any repeated part, terminated by `.PID > 0` failing. It carries `ID`, `PID`, `X`, `Y`, `DX`, `DY`. The author's own comment on the block is `;identify parent CO`, and the match key is `CO@{n}.ID == PARID` — so **`PARID` on a part is the `ID` of the `CO` that contains it**, and walking the collection to find the matching `ID` is how you get from a part to its own opening's geometry. `CO@1` is used separately as the *outermost* opening: lines 110-118 compute how far to extend a part by comparing the part's own parent CO against `CO@1`.

Both halves of the old guess were right — it is the opening region node — but the addressable-collection behaviour and the `PARID` link are new, and they are what make it usable. The caution in the cabinet-vision skill's `standard-ucs.md` about confirming on a real cabinet before writing new code still applies to the *specific fields* the user's own script reads (`CO.DZ` in particular is not exercised here); it no longer applies to the question of what `CO` is.

**Update, 2026-08-04 — the abbreviation itself, and the `CO.DZ` caution above, are now both settled.** Per the user directly: `CO` literally stands for "Case Open," or "Case Opening." Independently corroborated, from an entirely separate and unprompted source, by a Hexagon Nexus "Ideas" post from Matt Bauman (the same named source as the thread-134912 capture in `source-threads/`) — its own title reads **"Interior Case Openings (CO) > Provide Meaningful Values to the Z & DZ Parameters (for referencing in UCS)."** That post also resolves the `CO.DZ` caution above: `Z` and `DZ` on a `CO` are **always `0`, unconditionally** — a standing Hexagon limitation Bauman is requesting be fixed ("Currently the Z & DZ parameters for interior case openings are both 0"), not a field this script simply didn't happen to exercise. `X`/`DX` (and by extension `Y`/`DY`) are the fields that actually carry real geometry — Bauman's own workaround confirms this ("I often reference `INTERIOR.CO.X` or `INTERIOR.CO.DX`"). Practical takeaway: don't write UCS logic that expects `CO.Z`/`CO.DZ` to hold a real depth or setback value. Captured verbatim at `source-threads/nexus-idea-co-z-dz-values.txt`.

## Public parameters, and the comment-as-prompt convention

Every user-facing knob in this script is declared with `Public` and a **trailing comment that becomes the prompt**:

```
Public JCS_NotchConstruction_NotchLength<meas> = 0 ;Notch length (0=half)
Public JCS_NotchConstruction_OversizeDado<meas> = 1/32 ;Notch dado width oversize
Public JCS_NotchConstruction_DebugLevel<int> = '<lst>False=0|True=1' ;Debug
Public JCS_NotchConstruction_NotchType<int> = '<lst>Horz in front=1|Horz behind=2' ;Notch Type
```

The per-part copies are then declared *without* `Public` and given their label explicitly, via the `<desc>`/`<style>` pair:

```
JCS_NotchConstruction_NotchType_Use<int> = '<lst>Horz in front=1|Horz behind=2'
JCS_NotchConstruction_NotchType_Use<desc> = 'Notch: Type'
JCS_NotchConstruction_NotchType_Use<style> = 1
JCS_NotchConstruction_NotchType_Use<int> := JCS_NotchConstruction_NotchType
```

Read that four-line block carefully, because it is the canonical **UCS-default-to-per-part-override** idiom: declare the dropdown, label it, make it visible (`<style> = 1`), then seed it with `:=` from the Public default. The `=` on the declaration and the `:=` on the seed are doing different jobs — the declaration must not re-evaluate on every rebuild or the user's override would be wiped each time. All of it sits behind `if this.X == null then`, so it happens once per part.

The `<desc>` strings are also a naming convention worth copying: every one is prefixed `'Notch: '` — `Notch: Enable`, `Notch: Type`, `Notch: Length`, `Notch: Dado Length (0=half)`, `Notch: Enable Entire Interior`. In a sidebar full of parameters from a dozen scripts, that prefix is what makes one system's controls findable.

`<bool>` is used for the on/off flag (`JCS_NotchConstruction_Run_Me_Through<bool> = false`) and tested bare: `if JCS_NotchConstruction_Run_Me_Through then`, no comparison operator.

## The group-control pattern: "first of my type owns the collective switch"

A neat solution to a real UI problem — where do you put a control that applies to a whole interior, when the script only ever runs per-part? Lines 17-29:

```
if :.{name}@1.OBJID == this.OBJID then
    ;this is the first of the type of seperator in the interior, add parameter allowing all to enabled
    if this.JCS_NotchConstruction_Interior_EnableAll == null then
        JCS_NotchConstruction_Interior_EnableAll<int> = '<lst>Nothing=0|Enable All=1|Disable All=2'
        …
    end if
else
    delete JCS_NotchConstruction_Interior_EnableAll
end if

TEMP_ForceEnable<int> := :.{name}@1.JCS_NotchConstruction_Interior_EnableAll
```

`:.{name}@1` — the parent's first sibling carrying my own name — is the "am I the first one?" test, and comparing `OBJID` is how identity is established. The first `PT` in the interior gets the Enable-All dropdown; every other `PT` explicitly **deletes** its own copy so the control appears exactly once; and then every part, first or not, reads the value back off `:.{name}@1`. Note the third option: `Nothing=0` is the resting state, so the dropdown is a momentary command rather than a persistent value.

The same `:.{name}@1` idiom appears in the Casework Wall stack as a memoisation guard (see `casework-walls-jonah.md`) — same test, different purpose.

## The debug-parameter pattern

`Public JCS_NotchConstruction_DebugLevel` gates a full diagnostic mode, and the implementation is worth copying wholesale because classic UCS has no debugger. When debug is on, the intermediate values get **written onto the operation object** where the part editor will show them:

```
if JCS_NotchConstruction_DebugLevel == 1 then
    NotchMaster.CHECK_PART_X := CHECK_PART_X
    … all six position/size values …
    NotchMaster.TEMP_Diff_Z := TEMP_DIFF_Z
end if
```

and when it is off, the same values are explicitly deleted at the end of each pass (`delete CHECK_PART_X`, … through `delete CHECK_PART_AZ`) so they don't accumulate on the part. There is also a **progress marker** — `JCS_NotchConstruction_Debug_Value := 1`, then `:= 2` further in — which is how you find out where a script died when it produces no error, only a wrong result. And a per-type tally, `JCS_NotchConstruction_Debug_INTERSECTIONS_{TEMP_PartType} := TEMP_INTERSECTION_COUNT_THIS_SEPARATOR_TYPE`, interpolating the part type into the parameter name so five counters come out of one line.

Note `TEMP_DIFF_Z` vs `TEMP_Diff_Z` used interchangeably in the same statement — more confirmation that identifiers are case-insensitive.

## Bounded search as a deliberate optimisation

Lines 121-129:

```
;this code allows us to limit our seach to the items that are likely to intercept.
;setting TEMP_Loop1 to 1 and TEMP_End_Loop1 to 5 would cause a complete search, if we ever think that might happen.
if name == 'FS' or name == 'DS' or name == 'SH' then
    TEMP_Loop1<int> := 4
    TEMP_End_Loop1<int> := 5
else
    TEMP_Loop1<int> := 1
    TEMP_End_Loop1<int> := 3
end if
```

The five separator types are indexed 1=FS, 2=DS, 3=SH, 4=PT, 5=S_DIV, and the loop bounds encode a fact about the geometry: horizontals can only ever intersect verticals and vice versa, so each part searches half the space. The comment documents how to widen it back to a full search — which is the right way to leave an optimisation like this, since the reasoning is invisible from the code alone.

The inner loop over instances is the standard unbounded-until-`PID`-fails walk, with the part type itself interpolated:

```
if {TEMP_PartType}@{TEMP_Loop2}.PID > 0 then
    CHECK_PART_X := {TEMP_PartType}@{TEMP_Loop2}.X
    …
else
    TEMP_Out_Of_Part_Instances := 1
end if
```

## The dado / master-dado pair, and negative `S_MORT*` for oversize

Each detected intersection produces two operations — a `dado` on the part being iterated and an `mdado` (master dado) on the part it runs into:

```
dim NotchDado as new dado
NotchDado.desc = 'Notch Construction Dado'
NotchDado._FACEWP := 1
…
dim NotchMaster as new mdado
NotchMaster.desc = 'Notch Construction Master Dado'
NotchMaster._EDGWP := 1
…
NotchMaster.S_MORTL := -1*(JCS_NotchConstruction_OversizeDado/2)
NotchMaster.S_MORTR := -1*(JCS_NotchConstruction_OversizeDado/2)
```

`_FACEWP` on the face-worked dado, `_EDGWP` on the edge-worked master — the two working-plane parameters used as a matched pair, one per operation type. And **`S_MORTL`/`S_MORTR` set negative widens the slot**, half the oversize off each side; the Casework Wall stack uses the identical sign convention to hit a specific bit diameter (`casework-walls-jonah.md`). Both operations get a `desc`, which is what makes them identifiable in the part editor's report view later.

`ABS()` is attested here (`TEMP_Diff_Z := ABS(Z - CHECK_PART_Z)`) — one of the few built-in functions this skill has a real usage for.

## Two bugs in the shipped source, and one language gotcha

**Line 403 reads `delete delete JCS_NotchConstruction_Interior_EnableAll`.** A duplicated keyword in the cleanup branch of the widely-distributed release. Whether CV errors on it, ignores the second token, or silently skips the statement is **not known** — do not assert a behaviour. What it does establish is that this branch is rarely reached, since a hard parse error there would have been reported in fourteen years of a 4,699-view thread. Worth checking before copying that cleanup block.

**The two-nested-`if` comment, twice** (lines 185 and 272):

```
;don't know why it doesn't work to combine these into a single if-then, but it doesn't.  This is where most of my debug time was spent.
```

The code splits an X-range test and a Y-range test into nested `if`s rather than joining them with `and`. Taken at face value this says **compound `and` conditions can behave differently from nested `if`s in classic UCS** — and it is the author's most experienced-hands finding in the file. But he says outright he doesn't know why, so any mechanism explanation would be reconstruction. Treat it as: if a multi-clause `and` is behaving inexplicably, try nesting before assuming your geometry is wrong. Note that elsewhere in the same script multi-clause `and` conditions *are* used and evidently work (`while CO@{TEMP_Loop1}.PID > 0 and TEMP_Parent_DX == 0 do`), so this is not a blanket rule.

**Revision 11's history entry names a real trap:** `;Fix for loop variables becoming a measurement.` An integer loop counter can acquire a `<meas>` type from arithmetic, after which comparisons and increments behave as measurements. This is why every counter in the file is re-typed on every touch — `TEMP_Loop1<int> += 1`, not `TEMP_Loop1 += 1`. **Carry the type suffix on every assignment to a counter.**

## `JCS_` — a third Jonah Coleman prefix, and what it does to the fingerprint rule

Every parameter in this script is prefixed `JCS_`, on a file whose credit line is `Jonah@FletcherWoodProducts.com`. That matters for the attribution fingerprint recorded in this repo's own `README.md` (Authorship section), which says he namespaces by **employer** initials — `FWP_` for Fletcher Wood Products, `AA_` for Architectural Arts. Here the employer is FWP and the prefix is `JCS_`, which reads as his own initials rather than the shop's.

So the rule needs widening rather than correcting: **he consistently namespaces, but not always by employer.** `JCS_` is the earliest form and personal; `FWP_` and `AA_` are the later, employer-keyed forms. As an authorship tell that is *stronger*, not weaker — a dense, consistently-prefixed parameter namespace with `<desc>`/`<style>` labels and a revision history at the end of the file is the signature, whichever prefix it uses. As a *dating* tell it is useful too: `JCS_` on a file suggests the FWP era or earlier.

## Practical notes from the thread

**Where the controls appear.** Jared spent a while stuck because he assumed Section Level; Adam Mauss's answer is the useful one — from an ortho view, click the partition or fixed shelf and the options are in the list at the **bottom left of the screen**. Enabling `Notch: Enable` there causes the remaining parameters to appear, because they are declared inside `if JCS_NotchConstruction_Run_Me_Through then`.

**Copying UCS code off the Nexus forum mangles it.** linfordz got "one really long line"; Other Bruce got a parse failure that moved to a different blank line each time he deleted line three. Jonah Coleman's advice: double-click the text first, then copy — and in Chrome, double-click, then click at the start and drag to the bottom before copying. Bluntly, later: "Yep use internet explorer." Anything pasted out of that forum should be checked for collapsed line breaks before it is blamed on the code.

**Jonah Coleman's own use of it** goes well beyond the obvious: "I even do this now where I have a 2 door 2 drawer base with a partition- send the partition all the way to the top and use notch construction to put the drawer stretcher all the way across. And for cubbies... oh my god."

**Training, per Hexagon.** Chip Martin's answer to "is there a UCS for Dummies?" was the eLearning courses **Object Intelligence Complete** and **User Created Standards Complete** — the only named CV training resources in this skill.

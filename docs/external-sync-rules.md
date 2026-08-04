# Rules for syncing with the `cabinet-vision` skill (and other external sources)

This repo and the user's separate `cabinet-vision` Claude skill cover overlapping ground — the skill has its
own, much deeper technique write-up of these same Jonah Coleman scripts, plus a great deal of CV material
this repo doesn't need. The two are maintained independently and periodically get cross-checked against each
other. This page is the accumulated rule set from doing that a half-dozen times so far, so the next pass
doesn't have to rediscover it. Not CV technique content — process documentation for this repo's own upkeep.

## House style always wins over the source

This repo's own verbiage rules are enforced regardless of what a synced source says, every time:

- **Attribution: "Jonah Coleman," never "St. Jonah."** The skill has used both names at different points in
  its own history; whichever it currently uses, this repo normalizes to his real name on import.
- **No "mangled"/"mangled paste"/"mangled file" describing this repo's own materials.** Reword to something
  like "collapsed-line-break paste." Exception: Jonah Coleman's own general warning about Nexus-forum
  copy/paste "mangling" is fine to keep verbatim as a quote — that's *his* word for *his* observation, not
  this repo's narration.
- **"worth stealing" → "worth learning from" / "worth absorbing."**
- **No "Videos" section or screencast.com link commentary in curated `docs/*.md`.** The verbatim forum-thread
  capture in `source-threads/` is a primary-source capture, not commentary, and is exempt — dead links in a
  raw capture are fine; a curated write-up recommending them is not.
- **Confirmed-against-real-CV claims say "CV2025.4"** (this shop's actual installed version), not generic
  "CV2025" — except genuine forward-looking mentions about other installs, or the separately-named "CV2025.3"
  forum regression, which is a different, specific version reference that must not get flattened into 2025.4.

A sync that reintroduces any of these is a regression to catch and fix, not new content to accept as-is —
this has actually happened (an external doc-sync pass reintroduced "mangled paste," "worth stealing," and
the Videos section all at once; see the 2026-08-02 CHANGELOG entries).

## Path references get rewritten, not copied verbatim

The skill's internal layout (`examples/`, `full-docs/`) doesn't match this repo's (`scripts/<subdir>/`,
`source-threads/`). Any path reference copied in from the skill gets rewritten to the real local path:

- `examples/<script>.txt` → `scripts/<subdir>/<script>.txt`, using this repo's actual folder for that script.
- `full-docs/nexus-thread-*.txt` → `source-threads/nexus-thread-*.txt`.
- References to **skill-only docs with no local equivalent** (`standard-ucs.md`, `parameter-glossary.md`,
  `SKILL.md`, `object-intelligence.md`, `internal-part-names.md`, `catalog-export-format.md`,
  `ucs-breakdown.txt`, and skill-only example scripts like `linebore-attributes.txt` that were never
  packaged here) get reworded to explicitly name the source — "the cabinet-vision skill's `standard-ucs.md`"
  — rather than presented as a bare/broken repo-relative link.

## Two different kinds of "new content" get handled differently

**Script-specific technique/defect findings** (something new about `shelf-standards-jonah.md`,
`casework-walls-jonah.md`, or `notch-construction-jonah.md`'s own subject matter) stay **cross-referenced
only**, not duplicated wholesale — unless the new finding is a factual correction to a claim this repo
already makes, in which case it gets folded into the existing paragraph with an explicit "Update, <date>"
marker rather than silently overwritten.

**General-purpose CV reference material** — not tied to any one script, useful to anyone reading classic UCS
in this pack (official published keyword lists, numeric code tables, cross-system distinctions) — gets
**folded in outright** as new content, in its own file (`docs/cv-object-reference.md` is the current
example). It doesn't need re-verification against Jonah Coleman's scripts specifically to be worth having;
it just needs to be clearly sourced and clearly distinguished from anything this pack's own docs claim
empirically.

## Reconcile discrepancies explicitly — never silently pick a winner

When a newly authoritative source (a published Hexagon doc, a staff-verified forum reply) partly disagrees
with something this pack's docs already say from direct script observation, both sides get stated, with the
newer source's confirmed parts and the older claim's residual (either confirmed, contradicted, or left
plausible-but-unverified) both spelled out. This has come up concretely: Hexagon's published `OBJECT` table
confirms `OBJECT == 17` = Part exactly, but labels `15` as Subassembly where this pack's own docs (from
Jonah Coleman's code comments) call it "Face," and labels `10`/`37` as Cabinet/Order where this pack's docs
call them "copied"/"pulled from library." The fix in each case was a reconciliation paragraph — official
label stated plainly, the pack's own empirical reading kept and explained as plausible-but-not-independently-
verified — not a rewrite that erases one side.

## Verification workflow for a sync pass

1. **Diff the actually-overlapping files directly**, don't trust a summary of what changed. Read both
   versions of each shared doc (`shelf-standards-jonah.md`, `casework-walls-jonah.md`,
   `notch-construction-jonah.md` currently overlap the skill's `references/` folder) and separate genuine
   content deltas from the intentional stylistic differences listed above.
2. **Only fold in what traces to an authoritative source** — Hexagon's own published documentation, a
   staff-verified forum reply, or direct click-testing against a real CV install. Community speculation
   (even when interesting) gets flagged as speculation, not stated as settled fact.
3. **Every sync pass gets a `CHANGELOG.md` entry** describing what was checked and what was (or wasn't)
   found — including "checked, no factual deltas" passes. The sync history is itself worth keeping, since
   the two sources can drift again and the next check benefits from knowing what was already ruled out.
4. **When nothing needs to change, say so plainly** rather than making a cosmetic edit to look thorough — a
   confirmed "checked and clean" is a real result.

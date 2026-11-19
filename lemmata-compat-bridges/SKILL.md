---
name: lemmata-compat-bridges
description: Use when `lem publish` reports compatibility obligations "NOT proved" (FAILED or INCONCLUSIVE) and you need to write `<Obligation>_Manual` bridge lemmas in CompatProofs.dfy — reading .lem-publish/obligations.dfy, the obligation kinds (BodyEquivalent, Equivalent, PreWeakened, PostStrengthened, CoerceN, TypeEquivalent, ValueEquivalent), the direct route (Old.M.F vs M.F) vs the shadow route (DpackOldVocab_S_* / DpackNewVocab_S_* and F_Body), and what the bridge screen refuses ({:axiom}, assume, {:only}, bodyless lemmas). Also use to decide whether a change is really breaking and should go major instead.
tags:
  - lemmata
  - proof
  - release
related_skills:
  - lemmata-publish
  - dafny-lemma
  - dafny-calc
  - dafny-decreases
  - dafny-opaque-reveal
---

# Lemmata: Compatibility bridges (`CompatProofs.dfy`)

When you publish a new release, lem checks it against its predecessor. The
old release is materialized as `module Old`, each changed contract or body
becomes a lemma with an empty body, and the verifier runs. An obligation that
verifies empty is **patch**. One that needs your proof, a **bridge**, is
**minor**. One nobody proves is **major**.

A bridge proves *that the new release still satisfies what dependents relied
on*. It is not a way to get a version number. If the change genuinely alters
behavior a caller could observe, the honest answer is a major bump; ask the
user before taking it (`lem publish --allow-major`).

## The loop

```bash
lem publish                      # FAILED/INCONCLUSIVE obligations listed with hints
# goals:  .lem-publish/obligations.dfy        (the emitted lemmas, read-only)
# stubs:  CompatProofs.dfy at the package root (or .lem-publish/CompatProofs.stubs.dfy
#         when CompatProofs.dfy already exists: copy the new stubs in)
# ... write the proofs ...
lem publish                      # re-runs (predecessor cached); repeat until 0 unproved
```

- **FAILED** means the verifier errored on the obligation as emitted, whose
  body is empty. That is *not* a refutation. An equivalence needing induction
  fails exactly like a real value change, so check which one you have before
  proving.
- **INCONCLUSIVE** means it ran out of budget. A bridge with a few
  intermediate asserts usually fixes it.
- A symbol can stay breaking even when all its own obligations prove, because
  something it calls broke. The report's `propagated` list shows the path.
  Bridge the callee, not the symbol.

## Anatomy of a stub

The stub is the emitted lemma renamed `<Obligation>_Manual`, with parameters,
`requires`, and `ensures` verbatim. Only the body is yours. The obligation
name is `<Module_Path>_<symbol>_<Kind>`, with dots in the module path turned
into underscores, e.g. `Ramdaf_List_commuteMap_BodyEquivalent`. Copy names
from the stub or the publish output; a misspelled `_Manual` lemma is ignored
with a warning.

| suffix | what it asks |
| --- | --- |
| `BodyEquivalent` | the rewritten transparent function equals the old one on the old domain |
| `Equivalent` | a redefined predicate: old `<==>` new |
| `PreWeakened` | old `requires` implies new `requires` |
| `PostStrengthened` | old `requires` and new `ensures` imply old `ensures` |
| `Coerce0`, `Coerce1`, ... | a changed parameter or result type still coerces: parameter old→new, result new→old |
| `TypeEquivalent` | a subset type or newtype constraint is unchanged in meaning |
| `ValueEquivalent` | a changed `const` keeps its value |

Parameters are at the **old** types. When `n: nat` widened to `n: int`, the
bridge takes `n: nat`, because the obligation only speaks for calls the old
signature allowed.

## Two routes — prove the statement the stub gives you

**Direct route** (the hint says "it compares Old.F with F directly"):

```dafny
module CompatProofs {
  import Old
  import Ramdaf.List
  import OL = Old.Ramdaf.List

  lemma ReduceFoldl<A, B>(f: (A, B) -> A, g: (A, B) -> A, acc: A, s: seq<B>)
    requires forall a: A, x: B :: f(a, x) == g(a, x)
    ensures OL.reduce(f, acc, s) == List.foldl(g, acc, s)
    decreases |s|
  { if |s| > 0 { ReduceFoldl(f, g, f(acc, s[0]), s[1..]); } }

  lemma Ramdaf_List_commuteMap_BodyEquivalent_Manual<A, B>(
      fn: seq<A> -> seq<B>, of: seq<B> -> seq<seq<B>>, list: seq<seq<A>>)
    ensures OL.commuteMap(fn, of, list) == List.commuteMap(fn, of, list)
  { /* unfold both, then ReduceFoldl(...) */ }
}
```

**Shadow route** (the hint mentions `Dpack{Old,New}Vocab_S_<module>`). The
statement is over generated vocabulary modules, and the shadow of `F` is
`F_Body` (`F_Pre` / `F_Post` for contracts):

```dafny
module CompatProofs {
  import Lib
  import DpackOldVocab_S_Lib
  import DpackNewVocab_S_Lib

  lemma Lib_Sum_BodyEquivalent_Manual(xs: seq<int>)
    ensures DpackOldVocab_S_Lib.Sum_Body(xs) == DpackNewVocab_S_Lib.Sum_Body(xs)
  { /* induction on xs */ }
}
```

Do **not** restate a shadow obligation as `Old.Lib.Sum(xs) == Lib.Sum(xs)`.
Relating `Old`'s body to `Old.Lib.Sum` is the claim under proof, so assuming
it would be circular, and that lemma will not discharge the obligation. What
the shadow asks is that the new function satisfies the old defining
equation, usually the same induction stated one step earlier. The vocabulary
modules exist only inside the generated artifact. Write against them anyway;
`lem publish` resolves them.

## What the bridge screen refuses (tool error, not a proof failure)

Everything a bridge assumes, the verdict would assume. So `CompatProofs.dfy`
must be pure proof:

- refused: a lemma or function without a body, `assume`, a body-less
  `forall` statement, `{:axiom}`, `{:extern}`, `{:verify false}`, `{:only}`
  (anywhere in the artifact)
- allowed attributes: proof-search, budget, and splitting only, such as
  `{:induction}`, `{:trigger}`, `{:fuel}`, `{:timeLimit}`, `{:split_here}`,
  `{:focus}`, `{:isolate_assertions}`
- A verifier error in a helper is charged to **every** `_Manual` lemma that
  names that helper, even transitively. One broken helper can fail every
  bridge, so keep helpers small and verify them first.
- `expected-errors` never excuses a bridge error.

## Practical notes

- `CompatProofs.dfy` stays in the package and ships with it, but it is not
  part of the package build or the trust surface. Commit it: the next release
  is checked against it, and stale lemmas that match no obligation are
  skipped with a warning.
- Errors are reported at `CompatProofs.dfy line N`. Fix them there, not in
  `.lem-publish/`, which is regenerated.
- Induction over a recursive function usually needs a `decreases`, a snoc or
  cons unfolding lemma, and `calc` (see `dafny-lemma`, `dafny-calc`,
  `dafny-decreases`).
- Some compatible changes cannot be proved today; function extensionality is
  the standing example, because lambda equality is intensional. Those are
  conservatively major. Don't fight it with `assume`; the screen rejects it.
- Making a function `opaque` with its body unchanged is minor: dependents
  re-verify after adding `reveal F();` (see `dafny-opaque-reveal`).
- Standalone checker: `lem-compat` ships beside `lem` under
  `compat/<dafny-minor>/` (needs `LEMMATA_DAFNY_DIR` set to the toolchain's
  dafny directory). `lem-compat bridge <old> <new>` re-checks only the bridge
  and its obligations, which is faster, but the tier always comes from a full
  `lem publish` run.

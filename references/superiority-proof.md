# LAFS over K&R: a conditional proof of dominance

Companion to `specification.md` and `hybrid-form.md`. It states what can be proved about LAFS versus K&R brace placement, from geometry and arithmetic first and perception second, and shows why "Worse is Better" does not rescue K&R.

**Simpler version:** `superiority-proof-eli5.md` explains every section of this document in plain language, in the same order (ELI5 version).

## Contents

0. How to read this document
1. Definitions and the Steelman Protocol
2. Theorems (geometry and arithmetic)
3. Measured examples
4. Premises, corollaries and the priority axiom
5. "Worse is Better"
6. Boundary conditions: where the proof stops
7. Falsification protocol
8. Ledger: what is proved and what is premised

---

## 0. How to read this document

A proof is only as strong as its premises, so every statement here is labeled. **D** is a definition. **T** is a theorem that follows from definitions and arithmetic alone. **A** is an axiom and **P** is a perceptual premise; a reader may inspect and reject these. **C** is a corollary that combines a theorem with a premise. Nothing here appeals to popularity, taste or authority; `hybrid-form.md` §1 already rules those out.

The claims, stated precisely:

- **C0, structural impossibility (T1).** K&R has a nonzero parity defect by definition. It cannot be removed without ceasing to be K&R, so no formatter or future tooling can fix it.
- **C1, bounded and recoverable cost (T3, T5, T6).** LAFS pays for parity in a currency whose price does not grow with the head, caps that price, and takes rows back by flattening.
- **C2, dominance under the parity axiom (A1, §4).** If parity is a hard constraint, which is the specification's own axiom (§1.1, "Geometry over compactness"), then K&R is never optimal wherever a detached opener is legal. If parity is only weighed against rows at a finite rate, K&R can win solely on short heads, below a threshold set by the reader and not by the style.
- **C3, survival is not quality (§5).** "Worse is Better" explains why K&R survives. It does not show that K&R is better.

"K&R can never be better" is therefore exact in one sense and only in that sense: K&R is never *optimal* under A1 where detaching is legal. §6 lists the places where the argument narrows, including the one place the specification itself concedes K&R-shaped geometry.

## 1. Definitions and the Steelman Protocol

**D1 Placement.** Tokens sit on a grid of (row, column) cells, with columns measured in display cells (specification §7.5).

**D2 Pair.** A container is a matched opener `o` and closer `c`. Its vertical span is V = row(c) − row(o). It is *flat* when V = 0 and *tall* when V ≥ 1. A Monolith is flat, and flat pairs have no vertical geometry, so parity is defined for tall pairs only.

**D3 Head metrics.** `b` is the column of the head's first character, `h` is the head's width in cells, `g ∈ {0, 1}` is the gap between head and opener when they share a row, `s = 4` is the indent step, and `n ≥ 1` is the number of item rows.

**D4 Boundary run.** A run is a maximal string of adjacent opening glyphs, or of adjacent closing glyphs. A lone container is a run of length 1. The **parity defect** of a tall run-pair is Δ = |col(first glyph of the opening run) − col(first glyph of the closing run)|. Inside a fused run of width k, individual pairs differ by at most k − 1 cells; that residual depends on chain length only and never on head text.

**D5 Layouts.** Row counts include the head row, and all three layouts have V = n + 1.

| Layout | Opener at | Closer at | Content column | Rows | Parity defect Δ |
|---|---|---|---|---|---|
| K&R | (r, b+h+g) | (r+n+1, b) | b + s | n + 2 | h + g |
| Allman (LAFS Expanded) | (r+1, b) | (r+n+2, b) | b + s | n + 3 | 0 |
| Column-anchored Hybrid | (r, b+h+g) | (r+n+1, b+h+g) | b + h + g + s | n + 2 | 0 |

**D6 Unit costs.** ρ is the cost to a reader of one additional row. δ is the cost of one column of horizontal displacement between a matched run-pair. Both are unknown positive constants; every theorem below holds for any positive values.

**The Steelman Protocol.** Define **K&R\*** as K&R brace placement plus everything a formatter can add: canonical output (layout depends on tokens only), flattening of any container that fits the Canvas, one item per line in expanded containers, and hugging of a sole item onto the head row (the habit that produces `({` and `}))` runs). By L7 the token stream is identical in both styles, so whitespace is the only variable. After this grant, K&R\* and LAFS differ in exactly one thing: where the opener and closer of a tall container sit. Every theorem below is about that difference.

## 2. Theorems

### T1. Parity defect

**Statement.** For a tall container with a nonempty head, K&R has Δ = h + g ≥ 1, while Allman and column-anchored Hybrid have Δ = 0.

**Proof.** In K&R the opener ends the head row, so col(o) = b + h + g, and the closer sits at the head's first column, col(c) = b. Hence Δ = h + g. A head is at least one cell wide, so Δ ≥ 1. In Allman the opener sits alone at b below the head and the closer is at b, so Δ = 0. In Hybrid both sit at b + h + g, so Δ = 0. ∎

**Corollary 1.1 (unfixability).** K&R is *defined* by "opener last on the head row, closer at the head's first column". Δ = 0 needs col(o) = col(c). Holding the closer at b forces the opener to b, which is off the head row (Allman). Holding the opener on the head row forces the closer to b + h + g (Hybrid). Either move leaves K&R. No option, formatter or future tool can give K&R a zero defect, because the defect belongs to the layout and not to its implementation. ∎

This result is definitional on purpose. The *value* of Δ = 0 is the axiom A1 in §4; the *impossibility* of reaching it inside K&R is arithmetic.

### T2. Collinearity and rails

**Statement.** In Allman, the head's first cell, the opener and the closer lie on one column, b. In K&R only the head's first cell and the closer do, and the opener is displaced by h + g. Consequently every opener of a non-flat LAFS container in Expanded or Grouped form (Hybrid excepted, since the parser forces its opener inline) sits on the indent grid {b₀ + s·j : 0 ≤ j < D}, where D is the maximum nesting depth, so openers occupy at most D distinct columns. In K&R\* openers sit at b + h + g, so they occupy as many distinct columns as there are distinct head widths in the file.

**Proof.** Immediate from D5. ∎

### T3. The currency theorem

**Statement.** Every layout pays for pairing an opener with its closer in some currency. K&R pays h + g columns of displacement, at cost δ(h + g). Hybrid pays h + g columns of content shift. Allman pays rows: its own opener row, plus its own closer row when K&R\* merged that closer into a clump, plus its own head row when K&R\* hugged a sole item onto the enclosing head row. Call that count r. Then 1 ≤ r ≤ 3 per tall run-pair, and r does not depend on h.

**Proof of the bound.** Per tall run-pair, LAFS owns at most three rows: an opening row, a closing row and its own head row. K&R\* owns none of the opening row (the opener ends a head row) and at most one closing row, which it may share. So the extra is at least 1 (the opener row, always) and at most 3. ∎

**Break-even.** LAFS is cheaper than K&R\* on a run-pair if and only if rρ < δ·Δ, that is, Δ > Π, where Π = rρ/δ is the *detachment price* expressed in columns. Because Δ = h + g grows without bound in h while r ≤ 3 is fixed, for every reader (every finite Π) there is a head width beyond which Allman is strictly cheaper, and the saving δΔ − rρ grows without limit. Conversely, for any finite Π, K&R\* can be cheaper only on containers with Δ ≤ Π.

The table counts how many of the eight tall run-pairs measured in §3 lie above each price:

| Detachment price Π (columns) | LAFS cheaper when K&R\* Δ exceeds | Run-pairs in §3 above the line |
|---|---|---|
| 4 | 4 | 8 of 8 |
| 8 | 8 | 6 of 8 |
| 16 | 16 | 5 of 8 |
| 32 | 32 | 3 of 8 |

Hybrid has the same linear scaling as K&R, now in content indent. That is why `hybrid-form.md` caps the head (content indent ≤ 40, never > 64): LAFS pays in the constant currency (rows) wherever the grammar allows and, where it does not, keeps the linear currency small by shortening heads. ∎

### T4. Run mirroring versus run merging

**Statement.** In LAFS every closing run is the exact reverse of exactly one opening run, starting at the same column (L6, L2). Pairing is therefore read from geometry alone. In K&R\* a closing run such as `}))` may merge closers that belong to m ≥ 2 distinct opening runs, all sitting on one head row at columns c₁ < c₂ < c₃ that depend on the head's text. The closers themselves sit at constant columns (0, 1, 2 relative to the line), which carry no information about c₁, c₂, c₃. For m ≥ 2, same-type adjacent closers (`))`, `}}`) can be paired only by counting nesting depth.

**Proof.** The opening columns are functions of head text (D3); the closing columns are functions of the indent only. Two functions of independent inputs do not determine each other. In LAFS (L2, L3, L6) each closer or fused closing run starts a line at its own opener's column. ∎

Case D in §3 shows it: the closing run `}));` pairs with openers at columns 45 and 63–64.

### T5. Grouping makes the row overhead constant

**Statement.** Take a sole-child chain of k nested containers around n item rows. K&R\* uses n + 2 rows. Naive Allman (every opener and closer alone) uses n + 2k + 1 rows, an overhead of 2k − 1. Grouped LAFS uses n + 3 rows, an overhead of exactly 1 for every k ≥ 2.

**Proof.** K&R\*: one head row carrying all k openers, n item rows, one closing row. Naive Allman: head row, k opener rows, n item rows, k closer rows. Grouped: head row, one fused opener row, n item rows, one fused closer row. ∎

Grouping removes the dependence on nesting depth that makes naive Allman expensive, which is the valid objection to dogmatic Allman (`SKILL.md`, "Why this style exists").

### T6. Row balance

**Statement.** Compare against K&R as it is usually practiced, with braces on every block and no flattening of blocks. Let M be the number of maximal flat statements (a Monolith that K&R expands) and X the number of tall run-pairs LAFS must detach. Each flat statement saves at least 2 rows (K&R spends n + 2 ≥ 3 rows for a block, LAFS spends 1), and each detached run-pair costs at most 3 rows (T3). Hence LAFS uses fewer rows overall whenever 2M > 3X. When K&R does not hug sole items and does not clump, the condition relaxes to 2M > X.

**Proof.** Net saving ≥ 2M − 3X by the two bounds above. ∎

This is a sufficient condition, and it is checkable on any codebase by counting. Against K&R\* (which also flattens), the flat savings cancel and K&R\* keeps its row lead on tall containers, at most r rows each; T3 prices that lead against Δ.

### O1. K&R's own text uses two rules

This is an observation about a document, not a theorem. In *The C Programming Language* the opening brace of a function definition sits on its own line, while the braces of `if`, `while` and `for` hug the head:

```
int Max(A, B)
int A, B;
{
    return A > B ? A : B;
}

while (Count > 0) {
    Count = Next(Count);
}
```

The usual explanation for the function case is that old-style parameter declarations sat between `)` and `{`, so the brace could not hug the head. That reason no longer exists in ANSI C, and the exception remained. K&R therefore already contains Allman geometry for its largest containers and fails the single-rule test of the specification (§1.2, "Binary honesty") on its own terms.

## 3. Measured examples

All numbers below were computed by a bracket-matching script over the exact snippets shown (tall run-pairs only; Δ at run level, D4). The cases are illustrations of the quantities defined above, not a sample of real code, and they are not selected to flatter LAFS: cases B, C and D cost LAFS rows.

**Case A.** A line comment forces the function body out of Monolith, and the `if/else` fits on one line.

```
// K&R*
int ProcessAccountRecord(Account_Record *Record, Process_Options Options) {
    // Archived records are read-only: never update them
    if (Record->IsActive) {
        Update(Record);
    } else {
        Archive(Record);
    }
    return 0;
}

// LAFS
int ProcessAccountRecord(Account_Record *Record, Process_Options Options)
{
    // Archived records are read-only: never update them
    if (Record->IsActive) { Update(Record); } else { Archive(Record); }
    return 0;
}
```

**Case B.** A sole-child chain (Container Grouping).

```
// K&R*
InitializeCoreEngine([
    // Order matters: security first
    "SecurityModule",
    "DatabaseDriver",
    "RoutingInterface"
]);

// LAFS
InitializeCoreEngine
([
    // Order matters: security first
    "SecurityModule",
    "DatabaseDriver",
    "RoutingInterface"
]);
```

**Case C.** A callback with several arguments.

```
// K&R*
LoadData(
    Config.Get('Url'),
    (Data) => {
        // Persist only non-empty payloads
        if (Data) { Parse(Data); Save(Data); }
    },
    True
);

// LAFS
LoadData
(
    Config.Get('Url'),
    (Data) =>
    {
        // Persist only non-empty payloads
        if (Data) { Parse(Data); Save(Data); }
    },
    True
);
```

**Case D.** A closing run that merges closers of different heads.

```
// K&R*
Items.Filter((Item) => Item.IsActive).ForEach((Item) => Process({
    // Ids only: payload is fetched lazily
    Id: Item.Id
}));

// LAFS
Items.Filter((Item) => Item.IsActive).ForEach
(
    (Item) => Process
    ({
        // Ids only: payload is fetched lazily
        Id: Item.Id
    })
);
```

| Case | Rows K&R\* | Rows LAFS | Tall run-pairs K&R\* (Δ per pair) | Σ Δ K&R\* | Tall run-pairs LAFS | Σ Δ LAFS |
|---|---|---|---|---|---|---|
| A | 9 | 6 | 3 (74, 22, 7) | 103 | 1 | 0 |
| B | 6 | 7 | 1 (20) | 20 | 1 | 0 |
| C | 8 | 10 | 2 (8, 10) | 18 | 2 | 0 |
| D | 4 | 8 | 2 (45, 63) | 108 | 2 | 0 |
| Total | 27 | 31 | 8 | 249 (mean 31.1) | 6 | 0 |

Reading the table:

- LAFS spends 8 rows on detaching 6 tall run-pairs, which is r ≈ 1.3 (case D is the r = 2 case: the extra opener row and the unmerged closer or sole-item row). It recovers 4 rows by flattening the `if/else` in case A, for a net of +4 rows (31 against 27).
- In exchange the parity defect falls from 249 columns to 0. Two of the eight K&R\* defects (22 and 7 in case A) vanish through flattening and the rest through detaching.
- Remove the comment in case A and the whole function becomes a 155-column Monolith. That is 1 row against 8 for K&R, and it is within the default 160-column Canvas and the complexity budget.

## 4. Premises, corollaries and the priority axiom

**A1 (priority of parity).** Parity (Δ = 0 for every tall run) is a hard constraint, and rows are minimized second. This is the specification's own axiom (§1.1) and it is lexicographic: no number of saved rows compensates a nonzero Δ.

**P1 (predictable fixation).** Locating a glyph at a column the reader already knows, such as the indent grid, costs less than locating a glyph whose column depends on the text of its line.

**P2 (closure and symmetry).** Two boundary glyphs that share a column at the two ends of a vertical span are perceived as one object. This is the Gestalt principles of closure and symmetry.

**P3 (read-dominance).** A line is written once and read many times, so a saving paid on every reading outweighs a cost paid once at authoring. This is the commonly quoted rule of thumb that reading outweighs writing by a large factor; treat it as a premise, not a measurement.

**C-1 (from T1, P1, P2).** Matching a closer to its opener in K&R takes a vertical scan to the head row followed by a horizontal search of h + g cells for a glyph at a data-dependent column. In LAFS the scan ends on the glyph. The matching task has one step instead of two, and the second step in K&R grows with the head.

**C-2 (from T4, P1).** In K&R\* a closing line such as `}));` forces nesting-depth counting, the one pairing task LAFS removes from non-flat containers. Counting errors in clumps are the characteristic mismatch error.

**C-3 (from T3, P3).** The per-reading saving of Δ columns is multiplied by every reading, and the detachment price (at most three rows) is also paid on every reading but is bounded. For heads wider than Π the net per-reading saving is positive and grows with the head.

**Why A1 is the reasonable axiom**, even for a reader who would trade rows for parity at some finite rate:

1. *Price asymmetry (T3).* The detachment price is bounded by three rows. The displacement it removes is unbounded in the head width. A1 sets the reader's threshold Π to zero; for wide heads that costs nothing, and for narrow heads it costs at most r rows.
2. *Recoverability.* Rows can be clawed back by Monolith (T6) and Grouping (T5). The parity defect cannot be reclaimed by any tooling (Corollary 1.1). A1 therefore spends the recoverable resource to buy the unrecoverable one.

## 5. "Worse is Better"

### 5.1 What the thesis claims

Gabriel's essay contrasts two design philosophies. The "worse" one puts simplicity of *implementation* above simplicity of interface, correctness, consistency and completeness, and argues that such systems spread faster. C and Unix are its standard examples, which is why the phrase attaches itself to the language K&R wrote about. The claim is about **survival**: a simpler-to-implement system can outlive a more correct one. Gabriel himself later published replies disputing the thesis.

### 5.2 Formatting has no correctness axis

The thesis trades implementation simplicity against correctness and completeness. Layout cannot make that trade. By L7 every layout of a token stream means exactly what every other layout means, and by L8 formatting is idempotent. "Worse" can therefore only mean "costs the reader more", and "simpler" can only mean "cheaper for the tool author". The whole question reduces to one inequality.

### 5.3 The break-even inequality

Let I be the one-time extra cost of implementing a LAFS-class formatter over a K&R-class one, U the number of users, R the number of readings per user, and Δc the per-reading saving from T1 to T4 (positive by C-1 and C-3). LAFS is cheaper in total if and only if

I < U · R · Δc, that is, U · R > I / Δc.

The left side is a fixed cost and the right side grows linearly with the readership. For every positive Δc there is a finite readership beyond which the fixed cost is repaid. The thesis's mechanism, implementation simplicity, is a fixed cost, and a per-reading cost eventually beats any fixed cost.

### 5.4 Lock-in is low

The thesis assumes the worse system becomes entrenched because switching is expensive. A layout change is mechanical: it preserves tokens and comments (L7), it is idempotent (L8), and the whole repository can be converted in one pass. The usual cost, noisy blame history, is mitigated by Git's ignore-revisions mechanism (`blame.ignoreRevsFile`). Switching cost is not zero (§6, B4), but it is a one-time, automatable cost, and §5.3 says what readership repays it.

### 5.5 Survival is not quality

Suppose §5.2 to §5.4 are all rejected. The thesis still only predicts that the worse design can win on adoption. K&R's prevalence is consistent with that prediction and is therefore *evidence about survival*, not about layout quality. A frequently offered explanation for the original choice is scarcity of screen rows and paper; whether or not that was the cause, the constraint no longer binds, and popularity does not appear anywhere in T1 to T6.

**Conclusion.** The geometry result of §2 and the survival thesis do not conflict. They answer different questions. Used as an argument that K&R is *better*, "Worse is Better" is a category error: it confuses the probability that a layout survives with the cost a reader pays for it. Used as an explanation of why K&R is widespread, it is fine, and it is not a justification.

## 6. Boundary conditions: where the proof stops

**B1. Parser-forced languages.** In Go, in Python at statement depth 0, and in similar grammars, a detached opener is illegal or changes meaning. There Hybrid keeps Δ = 0 but pays the other linear currency: content shifts right by h + g, which is the Void of `hybrid-form.md`. When the Void passes the hard limit (content indent > 64) with tokens fixed, the specification itself prescribes the concession layout (H6): body at base indent + 4 and closer at the base indent, which has K&R geometry. So in that domain the exact claim is that K&R geometry is never *preferred* where a detach is legal, and LAFS minimizes the cost where it is not by shortening heads (Head Reduction Ladder). The unconditional "never" does not hold there, and the specification says so.

**B2. Rows on short heads.** Against K&R\*, LAFS spends between one and three more rows per tall run-pair. If a reader's detachment price Π is large, K&R\* is cheaper on containers with Δ ≤ Π (T3). A1 rules this out by axiom; a reader who rejects A1 can keep K&R\* for short heads such as `else` and `try`.

**B3. Flat-to-expanded transitions.** Because LAFS flattens blocks, a block that crosses the Canvas or the complexity budget changes from one row to many, producing a larger diff at the crossing. K&R as usually practiced never flattens blocks and so never has that transition. The budget (≤ 4 statements) limits the effect and does not remove it.

**B4. Ecosystem and migration.** Editors, linters, CI, existing codebases and new contributors default to K&R-shaped output. Migration is a nonzero, one-time cost (§5.4). Under the foreign-contract rule, an enforced formatter wins until the user decides otherwise.

**B5. Evidence.** T1 to T6 are arithmetic. P1 to P3 are premises, and this document cites no controlled experiment for them. If they fail, T1 to T6 still hold as geometry, but they lose their weight as arguments about reading cost. §7 describes the test.

## 7. Falsification protocol

A claim that cannot be refuted is not a proof, so here is the experiment that would refute P1 and P2.

- **Materials.** Programs rendered in both K&R\* and LAFS from the same token streams (L7 guarantees this). Stratify by nesting depth (1 to 6) and by head width (≤ 8, 9 to 24, ≥ 25 cells).
- **Tasks.** (a) Find the partner of a marked closer. (b) Decide whether a block contains a given statement. (c) Detect a planted nesting error such as a misplaced closer in a clump.
- **Metrics.** Time to answer, error rate, and, if available, fixation counts.
- **Design.** Within-subject, counterbalanced for order, with a practice block to remove familiarity bias in either direction.
- **Predictions.** (1) On task (a) for tall pairs, LAFS is faster and the gap grows roughly linearly with head width (slope ≈ δ). (2) No difference on flat containers. (3) On task (b) at shallow depth, K&R\* may be faster because it uses fewer rows. (4) On task (c), K&R\* error rates rise with the number of merged closers m.
- **Refutation.** No head-width slope on (a), or K&R\* faster across all strata, would falsify P1 and P2. C-1 to C-3 then lapse, T1 to T6 remain as geometry, and A1 loses its justification.

## 8. Ledger: what is proved and what is premised

| Statement | Status | Rests on |
|---|---|---|
| Δ_K&R = h + g ≥ 1 and Δ_LAFS = 0 (T1) | Proved | Definitions D1 to D5 |
| K&R cannot reach Δ = 0 without ceasing to be K&R (Cor. 1.1) | Proved | The definition of K&R |
| Head, opener and closer are collinear in Allman only (T2) | Proved | D5 |
| Allman's price is constant in the head, K&R's and Hybrid's are linear; break-even Δ > Π (T3) | Proved | δ, ρ > 0 |
| LAFS closing runs mirror opening runs, K&R's can merge several (T4) | Proved | D3, L6 |
| Grouping makes row overhead 1 instead of 2k − 1 (T5) | Proved | Counting |
| LAFS uses fewer rows when 2M > 3X (T6) | Proved as a sufficient condition | Per-codebase counts |
| K&R's text splits function braces from control braces (O1) | Observed | The K&R text |
| Matching, clump and per-reading advantages (C-1 to C-3) | Conditional | P1, P2, P3 |
| K&R is never optimal where a detach is legal | Follows | A1 + T1 |
| "Worse is Better" does not make K&R better (§5) | Argued | §5.2 to §5.5 |
| Where the argument narrows (B1 to B5) | Stated | See §6 |

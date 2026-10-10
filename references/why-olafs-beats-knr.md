# LAFS versus K&R: The Formal Case

This document states, as precisely as the subject allows, why LAFS (Language-Agnostic Formatting Style) is the structurally superior layout and where K&R (end-of-line openers) can and cannot compete. Every claim is tagged **Proven** (follows from the definitions, true for any input), **Hypothesis** (plausible from perceptual science, falsifiable, untested for code) or **External** (adoption, tooling, history). Read it when you must justify the style, argue for it, or answer "isn't this just preference?".

## Contents

- 0. Status of the claims
- 1. Model and definitions
- 2. Metrics
- 3. Theorems T1 to T7
- 4. The scoreboard
- 5. The geometry of the rectangle
- 6. Cognitive hypotheses and how to test them
- 7. Cost model and "Worse is Better"
- 8. Standard defenses of K&R and the answers
- 9. Verdict

## 0. Status of the claims

The strongest honest statement is this: LAFS is strictly better than K&R on four structural-legibility metrics, is line-optimal within its own class, and pays a bounded, exactly computable price in vertical lines. K&R is better on exactly two measurable axes (line count, and horizontal reach where a parser forces an inline opener). The claim "K&R can never be better on anything" is **false as stated** (T5 and T7 are counterexamples) and no argument can make it true. The claim "K&R is never better on structural legibility, and its two advantages are small and recoverable" is **provable** and is what this document establishes.

Evidence from experiments is thin and must not be oversold. The studies found while preparing this document test **indentation**, not brace placement. Miara et al. (1983) found that moderate indentation (2 to 4 spaces) gave the best comprehension and that "blocked" versus "non-blocked" indentation made no difference. Bauer et al. (ICPC 2019, 22 participants, eye tracking) could not detect an effect of indentation depth on correctness or visual effort, with at most a small effect on fixation duration. A 2023 randomized controlled trial (Morzeck et al., with a follow-up by Hanenberg et al.) reported large effects of removing indentation from Java control flow, and a JSON experiment summarized in the Wikipedia article on indentation style reported non-indented code taking far longer to read. Taken together: *making structure visible through layout helps when it is absent; nobody has yet isolated brace placement (K&R versus Allman) in a controlled study.* Section 6 gives the protocol that would settle it. Until then the cognitive superiority of LAFS is a hypothesis supported by geometry, not a measured fact.

## 1. Model and definitions

Source text is a grid of cells with integer rows and display columns. A **container** is a pair (o, c) of opener and closer cells, nested in a forest. For a container spanning more than one row:

- **Head** h is the text on the logical line before the opener; w(h) is its display width in columns (w ≥ 0), g ∈ {0, 1} is the gap before the opener, and base indent b is the column where the head begins.
- **Content indent** is one level (4 columns) deeper than the anchor.
- **K&R layout (disciplined variant).** The opener sits on the head's line, so col(o) = b + w(h) + g. Items are one per line at b + 4. The closer is on its own line at col(c) = b, optionally cuddled with continuation text (`} else {`) or trailing glue (`);`).
- **Allman-pure layout 𝒜.** Opener alone on its own line at col b; closer alone at col b; items one per line at b + 4; the head occupies its own line above the opener.
- **LAFS.** 𝒜 plus two mechanisms: **Monolith First** (a container that fits and has no forced break is one line) and **Container Grouping** (a chain of sole-child containers shares one compound opener line and one mirrored compound closer line at a single column), plus the parser-forced Hybrid exception.

**Fairness rule.** K&R is granted Monolith too: any container that fits on one line may be written flat in either style. The comparison is only meaningful for containers that cannot be flat. "Disciplined" K&R excludes stacked callbacks (`App.Run(function() {`) and clumped closers (`});`); those variants compress further and strengthen K&R's line advantage but worsen T4. A **run** is a maximal sole-child chain; a **headed run** has a non-empty head.

## 2. Metrics

All are computable from the text alone, so a linter can report them.

| Metric | Definition |
|---|---|
| **M1 Parity gap** | Σ over multi-line containers of \|col(o) − col(c)\| |
| **M2 Anchor sensitivity** | ∂ col(o) / ∂ w(h): how far the opener moves when the head text changes |
| **M3 Mixed lines** | Number of lines that contain a multi-line boundary and also non-boundary content (head text; glue and comments excluded) |
| **M4 Closer fan-in** | For each closing line, the number of distinct source rows that hold the openers it closes; report the maximum |
| **M5 Line count** | Total rows |
| **M6 Horizontal reach** | Absolute column where the body of a container starts |

## 3. Theorems

### T1. Parity gap (Proven)

*In K&R every headed multi-line container has parity gap w(h) + g ≥ 1. In LAFS (non-Hybrid) every multi-line container has gap 0.*

Proof. K&R: col(o) = b + w + g and col(c) = b, so the gap is w + g, and a non-empty head has w ≥ 1. LAFS: by law L2 both boundaries sit at col b (compound boundaries at their shared anchor). ∎

Example (computed, not estimated):

```
LoadData(                         // opener at column 8,  closer at column 0  -> gap 8
    Config.Get('Url'),
    function(Data) {              // opener at column 19, closer at column 4  -> gap 15
        if (Data) {               // opener at column 18, closer at column 8  -> gap 10
            Parse(Data);
            Save(Data);
        }
    },
    True
);
```

K&R total parity gap M1 = 8 + 15 + 10 = **33 columns**; the LAFS rendering of the same code has M1 = **0**.

### T2. Anchor invariance (Proven)

*In K&R, ∂ col(o)/∂ w(h) = 1: renaming the head moves the opener one column per character. In LAFS the derivative is 0: the opener's column depends only on the base indent.*

Proof. col(o) = b + w + g (K&R) and col(o) = b (LAFS). ∎ Consequence: the same container has the same skeleton whatever its head is called. Concretely, `LoadData(` puts the opener at column 8 and `LoadDataFromRemoteConfiguration(` puts it at column 31, while in LAFS both put it at column 0. A reader cannot learn the position of K&R openers; they must be re-located for every container.

### T3. Purity of lines (Proven)

*In K&R every headed multi-line opener produces a mixed line. In LAFS (outside Monolith and Hybrid) M3 = 0: every line is either head text, boundary tokens (plus glue and comments), or item content.*

Proof. K&R places the opener on the head's line, so that line holds head text and a boundary. LAFS places head, opener and closer on separate lines by law L1 and L3. ∎ This is the formal content of "separate the container from what it contains". Cuddled forms (`} else {`) are mixed lines of a worse kind: they combine a closer, a keyword and an opener.

### T4. Closer fan-in (Proven)

*In LAFS every closing line closes openers that sit on a single source row (fan-in 1). In K&R the fan-in is unbounded.*

Proof. L3 puts each closer or fused closer run on its own line; L6 requires a run's closers to mirror one opener run, which sits on one line at one anchor. So a closing line pairs with exactly one row. In K&R, a clumped closing line such as `});` can close a `(` opened on one row and a `{` opened on another, and a deeper clump can close openers from as many rows as there are closers. ∎

### T5. The line tax (Proven; K&R wins)

*For disciplined K&R, Lines(LAFS) − Lines(K&R) = H + J, where H is the number of non-flat headed runs (K&R hugs each opener onto the head line) and J is the number of continuation joins (K&R puts the closer on the same line as `else`, `.Method(` and similar).*

Proof. Take any non-flat headed run. LAFS emits head line, opener line, items, closer line; K&R emits head+opener line, items, closer line: one fewer. A continuation after a non-flat closer takes its own line in LAFS and shares the closer's line in K&R: one fewer. All other lines are common. Runs with an empty head (items of a list) cost nothing: K&R's opener also begins its own line. ∎

Verified on two cases. The `LoadData` code above has H = 3 (`LoadData`, `function(Data)`, `if (Data)`) and J = 0: 10 lines in K&R, 13 in LAFS, tax 3. A plain `if/else` has H = 2 and J = 1: 5 lines in K&R, 8 in LAFS, tax 3.

Bound: the tax is linear in the *number of non-flat headed runs*, not in code size, and Monolith First removes it for every container that fits. Container Grouping removes the extra boundary lines a naive Allman layout would add (2 per fused link). What remains is a constant of one line per headed run.

### T6. Optimality inside the parity-pure class (Proven, proof sketch)

*Let 𝒜* be the set of layouts of a container forest in which (P1) every non-flat container's boundary lines carry only boundary tokens, glue or comments; (P2) opener and closer share a column; (P3) each item starts its own line; (P4) head text precedes the opener on its own line. Then LAFS with an unbounded complexity budget minimizes the line count over 𝒜*.*

Sketch. Induct on the forest. (a) If a container can be flat (it fits and has no forced break), flat uses 1 line, and every non-flat form uses at least 3 (head, opener, closer), so flat is minimal; fitting is monotone, so flattening the outermost eligible container is also optimal for its children. (b) If it cannot be flat, it needs an opener line and a closer line, plus the lines of its items. (c) **Lemma:** two nested non-flat containers C ⊃ D can share an opener line and a closer line while satisfying P1 to P3 iff D is the sole content of C. Sharing an opener line forces D to be C's first content; sharing a closer line forces it to be the last; both together make it the only content. So fusing exactly the sole-child chains saves 2 lines per fused link, and nothing else can fuse. ∎ Corollary: every dogmatic-Allman layout that is not LAFS is *dominated*: strictly more lines for identical structural quality. (The complexity budget is a deliberate extra constraint that can forbid a fitting Monolith; the theorem holds for budget = ∞.)

### T7. Horizontal reach in parser-forced contexts (Proven; K&R wins)

*When the grammar forces the opener onto the head line (Go, JavaScript after `return`), column-anchored Hybrid has content indent b + w + g + 4, while K&R has b + 4. K&R's reach is smaller by w + g.*

Proof. Direct from the definitions. ∎ This is the **Void**: its size is exactly the head width. It is why the Hybrid rule limits itself to short heads (content indent ≤ 40 columns), prefers reducing the head, and only as a last resort adopts K&R-shaped geometry. In that corner K&R is the better layout and LAFS says so.

## 4. The scoreboard

| Axis | Winner | Margin | Status |
|---|---|---|---|
| M1 Parity gap | **LAFS** | 0 versus w + g per headed container (33 columns in the example) | Proven (T1) |
| M2 Anchor sensitivity | **LAFS** | 0 versus 1 column per head character | Proven (T2) |
| M3 Mixed lines | **LAFS** | 0 versus one per headed opener | Proven (T3) |
| M4 Closer fan-in | **LAFS** | 1 versus unbounded | Proven (T4) |
| M5 Line count | **K&R** | H + J lines (3 in both examples) | Proven (T5) |
| M6 Reach in forced-inline contexts | **K&R** | w + g columns | Proven (T7) |
| Line-optimality among parity-pure layouts | **LAFS** | dominates every other Allman variant | Proven (T6) |
| Familiarity, tooling, team convention | **K&R** | not geometric | External (§7) |
| Reading speed, error detection | **undetermined** | hypotheses H1 to H3 | Hypothesis (§6) |

No layout dominates on every axis, and none ever will: M5 and M1 trade off by construction (a closer cannot share the head's line and share its column with an opener that sits at the end of that line). The question is therefore not "which is better on all axes" but "which axes measure the reader's cost". Structural legibility (M1 to M4) is what the style exists for; M5 is a cost bounded by T5 and largely recovered by Monolith First and Container Grouping.

## 5. The geometry of the rectangle

A multi-line container has a bounding rectangle. In LAFS its left edge is a straight vertical line passing through both delimiter glyphs: opener at the top-left corner, closer at the bottom-left corner, content strictly to the right. The pair is **collinear**, so the rectangle is closed by construction. In K&R the pair spans the vector (Δx, Δy) = (w + g, N) where N is the number of rows between them, so the delimiters are at best the ends of a diagonal.

Let r be the cell aspect ratio (cell height / cell width, about 2 in typical monospace fonts). The skew of the pair from the vertical is θ = atan( (w + g) / (N · r) ). LAFS: θ = 0 always. K&R:

| Head width w + g | Rows N | θ (K&R) |
|---|---|---|
| 8 | 10 | 21.8° |
| 24 | 6 | 63.4° |
| 24 | 30 | 21.8° |
| 40 | 10 | 63.4° |

Two observations. The skew is *largest for short multi-line blocks with long heads*, exactly the case where a reader cannot lean on long vertical rails. And the skew changes whenever the head is edited (T2), so it carries no stable information.

The law of **good continuation** in perceptual psychology states that elements arranged along a straight line are seen as one group. LAFS places the two delimiters, and in a run all fused delimiters, on one line; K&R does not. The law of **closure** says a bounded shape is perceived whole. LAFS draws the left edge, the top and the bottom explicitly; K&R leaves the top-left corner at the far right end of a text line. These are hypotheses about code reading (H1 to H3), supported by well-established principles but untested for this stimulus.

Nesting adds a second geometric fact. In LAFS Expanded, a child's anchor column equals the parent's plus 4: depths form a **monotone staircase** read directly from columns, and the staircase is the same for openers as for closers. With Container Grouping a run is one rung; separation is local, so the staircase never needs more rungs than there are real nesting levels.

## 6. Cognitive hypotheses and how to test them

These are the places where the argument leaves mathematics and enters measurement. They are stated so that they can fail.

- **H1 Pairing time.** Finding the matching closer of a multi-line container is faster in LAFS than in K&R, increasingly so as head width grows (T1, T2 predict a head-width interaction).
- **H2 Mismatch detection.** A single deleted or misplaced delimiter is detected faster and more accurately in LAFS, because its column pairing breaks visibly (T1) and clumps hide the break (T4).
- **H3 Chunking.** Comprehension questions that require separating the condition from the block (what does the `if` test; what runs when it is true) are answered faster in LAFS (T3).
- **H4 Cost of the tax.** The extra scrolling and line-hunting from T5 produces a measurable penalty on long files. This hypothesis is *against* LAFS and must be measured for the comparison to be honest.

**Protocol.** Within-subject crossover, 40 or more professional developers plus a novice stratum, randomized style order, matched Java/C#/JavaScript snippets of equal semantic content (Monolith granted to both styles). Tasks: delimiter pairing, seeded mismatch detection, program-output questions, and a long-file navigation task for H4. Measures: correctness, time, fixation count and mean saccade length from eye tracking. Analysis: mixed-effects models with participant and snippet as random effects, head width and nesting depth as moderators, pre-registered hypotheses and effect-size thresholds. **Falsifiers:** no head-width interaction for H1 and no difference for H2 and H3 would remove the cognitive claim (T1 to T4 would remain true as geometry); a significant H4 penalty larger than the H1 to H3 gains would show that the line tax outweighs the benefits for that population.

An expert-fluency caveat applies in advance: experienced programmers read their habitual style fluently, and the existing indentation studies found that expertise masks layout effects. The most informative strata are novices, unfamiliar code, error detection, and code under time pressure.

## 7. Cost model and "Worse is Better"

### What the essay says

Richard Gabriel's "The Rise of Worse is Better" (1989; part of the essay "Lisp: Good News, Bad News, How to Win Big") contrasts the MIT/"Right Thing" approach with the New Jersey approach, which ranks **implementation simplicity** highest, ahead of correctness, consistency and completeness. His thesis is that this approach has **better survival characteristics**, because a small, easy-to-port implementation spreads first and then evolves. Gabriel called his description of the New Jersey approach a caricature, and in December 2000 published a reply titled "Worse Is Better Is Worse".

### Why it does not rescue K&R

1. **It is about adoption, not quality.** The essay explains why "worse" systems spread. It does not claim they read better, run better or reason better; the label itself concedes "worse". Quoting it in defense of K&R concedes the comparison on quality and appeals to popularity.
2. **Its mechanism needs an implementation-cost asymmetry that no longer exists.** The advantage of New Jersey designs is cheap implementation. A layout is a *pure whitespace function over a token stream*: either layout is produced by a formatter in time linear in the file, and conversion between them is mechanical and lossless (tokens are preserved, so conversion is a bijection on token streams). Whatever K&R saved a 1970s toolchain (24-row terminals, line-oriented editors) does not bind a 2026 editor with a tree-sitter parser.
3. **The cost structure favors the reader.** Let I be the one-time implementation cost of a layout's tooling, N the number of times code is read, c the per-read cost, and S the one-time cost of switching a team. Then C_total = I + N·c + S. LAFS beats K&R when ΔI + S < N · (c_K&R − c_LAFS), i.e. when N > (ΔI + S) / Δc. Reads outnumber writes by orders of magnitude, so any positive Δc is repaid after finitely many reads, and ΔI and S are one-time tooling and conversion costs that shrink toward zero once a formatter exists. The sign of Δc is exactly what §4 and §6 are about: geometry (T1 to T4) pushes it positive for LAFS; the line tax (T5) pushes it the other way. Neither side may assume the answer.
4. **Survival and merit are different variables.** QWERTY and the 80-column card survived for contingent reasons. A history of dominance explains who is on K&R today; it does not bound what a reader could achieve.

### What is conceded

Popularity has real value: shared tooling, shared habits, easier onboarding, smaller diffs against existing code. Those are **switching costs and network effects (External)**, borne once per team and reduced by automation. They are not properties of the layout.

## 8. Standard defenses of K&R and the answers

| Defense | Answer |
|---|---|
| "It's just preference." | A preference is a weighting of axes. T1 to T4 are measurements, not tastes: parity gap, anchor sensitivity, mixed lines and fan-in are the same for every reader. The weights are open; the numbers are not. |
| "Millions of developers use it." | An argument from popularity, and a statement about history (24-row terminals, the C book) and network effects, not about reading cost. Adoption is conceded in §7 and priced as switching cost. |
| "It saves vertical space." | True, by exactly H + J lines (T5). Monolith First and Container Grouping recover everything except one line per headed run, and modern displays make that line cheap. Where horizontal reach matters (T7) LAFS itself falls back to K&R-shaped geometry. |
| "The kernel style is K&R." | The goal (do not waste vertical space) is shared and conceded. The mechanism differs: LAFS achieves density with Monolith and Grouping instead of end-of-line openers and clumps. |
| "Formatters and linters default to it." | Tooling follows the standard that exists. That is a switching cost (§7), not evidence about legibility. |
| "Experts read it fine." | Probably true, and also true of non-indented code for a patient expert. The studies cited in §0 show expertise masking layout effects; the relevant strata are novices, unfamiliar code and error detection (§6). |
| "A cuddled `} else {` is more compact." | It is one line shorter (J in T5) and a mixed line (T3). Same trade, same answer. |
| "Allman wastes space." | Dogmatic Allman does, and T6 shows LAFS strictly dominates it: same structural quality with fewer lines. |

## 9. Verdict

What is established, in order of strength:

1. **Geometry (Proven).** LAFS gives every multi-line container a collinear delimiter pair (M1 = 0), an opener whose position is independent of its head (M2 = 0), lines that are either structure or content (M3 = 0), and closing lines that resolve one row of openers (M4 = 1). K&R fails each of these for every headed container by construction (T1 to T4).
2. **Optimality (Proven).** Among layouts that satisfy those properties, LAFS uses the fewest possible lines, so it dominates every other Allman-family layout (T6).
3. **Price (Proven).** Relative to disciplined K&R, LAFS costs exactly one line per non-flat headed run and one per continuation join (T5), and it concedes horizontal reach to K&R in parser-forced contexts by exactly the head width (T7), which is why Hybrid is restricted to short heads.
4. **Cognition (Hypothesis).** The geometric advantages should translate into faster pairing and better mismatch detection (H1 to H3). This is a prediction with a protocol and a falsifier, not a result.
5. **"Worse is Better" (External).** The maxim concerns adoption under implementation-cost constraints that formatting no longer has. It cannot be cited as evidence of K&R's merit, because it concedes "worse".

Stated as one sentence: **LAFS is the unique minimum-height layout among those that keep every delimiter pair collinear, and K&R is better only where a line or a column is the objective, by margins this document computes.** That is a narrower claim than "never better", and it is the claim that survives scrutiny.

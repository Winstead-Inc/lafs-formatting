# The Hybrid Form: Scope, Limits and Why It Exists

Load this file whenever you are about to emit a Hybrid layout (opener left on the head line), or when a language forces an inline opener and the head is long. It **supersedes** the earlier "accept the drift" and `warn_column = 120` wording in `specification.md` §6.7 and §12.

## Contents

1. Position
2. Where Hybrid comes from (and what counts as a reason)
3. The Void problem
4. The rules (H1 to H9)
5. Head Reduction Ladder
6. Authoring versus reformatting
7. Applying the short-head rule per language
8. Worked examples
9. Decision table
10. Voice: how to talk about this

---

## 1. Position

Allman-shaped LAFS is the layout that fits how a human reader's visual system works: the opener and closer share a column, so the eye drops straight down a rail and sees a closed rectangle (translational symmetry, Gestalt closure, no horizontal scanning to find where a block starts). K&R's end-of-line opener is a compression-era artifact: it saved lines on 24-row terminals. Its prevalence is history, inertia and tooling, not evidence that it reads better.

**Popularity is not an argument. It will never be an argument in the first place.**

The Hybrid Form is therefore a **concession**, not a style. It exists only where a language's grammar would reject or reinterpret a detached opener. It must stay small and must never become the default shape of the user's code.

## 2. Where Hybrid comes from (and what counts as a reason)

Hybrid is needed only because some languages were **designed** so that a newline before an opener is meaningful: Go injects a semicolon after a line ending in an identifier, literal, `)`, `]` or `}`; JavaScript restricts line terminators after `return`/`throw`/`yield`/`break`/`continue`, before `=>` and around postfix `++`/`--`; Python, Ruby, Kotlin, Swift, Scala and Lua end a statement at a newline; offside-rule languages read indentation as structure.

None of that is a law of programming. C, C#, Java, Rust, Dart, PHP, SQL, CSS, JSON, OCaml and the Lisps all accept a detached opener, which shows a grammar can support the readable layout at no cost to the machine. Where a language withholds it, the reasons given (fewer semicolons, one mandated style, a simpler lexer) are conveniences for authors or tool builders. None of them concerns the reader, and the reader pays for it on every line.

**What counts as a reason to use Hybrid:** a parse-level fact only: "a detached opener here would be a syntax error or would change meaning in this language."

**What does not count:** "it's idiomatic", "the community does it", "the formatter for this language does it", "the surrounding code does it", "it saves a line", "it's familiar", "a linter prefers it." Preference is not a reason. I is can never be a valid reason. (If CI enforces another formatter, that is a foreign-contract question: tell the user once and ask; see the precedence list in SKILL.md.)

## 3. The Void problem

Column-anchored Hybrid puts the body at `column(opener) + 4` and the closer at `column(opener)`. The longer the head, the farther right the opener sits, and every body line inherits that indent: a large, empty region at the left of the code, a **Void** (named after the [Boötes Void](https://en.wikipedia.org/wiki/Bo%C3%B6tes_void), the vast, nearly empty region of space). Voids waste the Canvas, push content toward the right edge, force earlier Expansion of children, and compound when Hybrids nest.

**Measure.** The *content indent* of a Hybrid is the absolute column where its body lines start: `base indent + head width up to and including the opener + 4`.


| Content indent                            | Verdict                                                                                            |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| ≤ 40 columns (25% of the default Canvas) | **Short-head Hybrid: allowed.**                                                                    |
| 41 to 64                                  | **Discouraged.** Reduce the head first (§5); allowed only if you can't, with a one-line note.     |
| > 64                                      | **Void: column-anchored Hybrid is forbidden.** Reduce the head, or use the concession layout (H6). |

The limits scale with the Canvas (25% and 40%). Nesting spends the budget: every enclosing indent level and every enclosing Hybrid subtracts from what is left, so a second nested Hybrid often crosses the line. For a soft limit of 40, the opener must sit at column 36 or less, so with base indent 0 a head may be 37 columns wide, with base indent 8 only 29.

Heads that fit: `return (`, `throw (`, `yield (`, `Key = [`, `Key: [`, `func Run() {`, `if Fault != nil {`, `} else {`, `Servers: [`. Heads that don't: long qualified calls, long Go signatures, builder chains, Terraform resource headers, deep nesting.

## 4. The rules

- **H1 Purpose test.** Hybrid only when the parser would reject or reinterpret a detached opener (§2). Uncertain? Use Hybrid **only if the head is short**. If the head is long, reduce it (§5) or ask: don't spend a Void on a guess.
- **H2 Monolith first.** Always. Hybrid answers "it doesn't fit and the opener may not detach", never a shortcut around Monolith.
- **H3 Short heads only.** Column-anchored Hybrid requires content indent ≤ 40 (soft) and never > 64 (hard).
- **H4 Reduce before you concede.** When the head is too long, run the Head Reduction Ladder (§5) before any other layout.
- **H5 No nesting spiral.** If a Hybrid inside a Hybrid would push content past the soft limit, restructure (guard clauses, early returns, helper functions) instead of nesting.
- **H6 Concession layout (last resort).** When the opener is parser-forced inline, the head cannot be reduced (existing tokens must not change), and the content indent would exceed 64: opener inline, body at **base indent + 4**, closer at **base indent**. This is the *only* place K&R-shaped geometry is tolerated, because the parser leaves nothing else. State it in one sentence ("long Go signature; closer anchored to the line to avoid a Void"). Never use it voluntarily or because it is familiar.
- **H7 Whitespace-sensitive carve-outs.** Where line-anchoring breaks the grammar (YAML flow collections need continuation lines deeper than the parent key; offside-rule languages need lines deeper than the layout column), the concession is unavailable: use Monolith, or leave the construct Verbatim and tell the user.
- **H8 Never retrofit.** Don't convert an Allman container to Hybrid in a language that allows Allman, and never mimic nearby K&R code in new code.
- **H9 Don't hide a Void.** If a large Hybrid is unavoidable, flag it. The user would rather know than discover it.

## 5. Head Reduction Ladder

Applies when you are **authoring** code (you control the tokens). Stop at the first rung that makes the head short.

1. **Monolith.** If the container fits the Canvas, it is one line and there is no head problem.
2. **Name an intermediate.** Move a long argument, condition or expression into a named local (`Const Total = Subtotal + Tax - Discount;`), then `return Total;`.
3. **Alias.** Short alias for a long import path, type or qualified name (`import X as Y`, a type alias, `use … as …`).
4. **Shorten by structure.** Guard clauses and early returns cut the base indent; split a long chain into steps.
5. **Extract a function.** A small helper with a short name replaces a long call head.
6. **Bundle parameters.** Replace a long parameter list with one options/record type (a `Pascal_Snake_Case` struct or object). It also shortens every call site.
7. **Rename.** Shorter, still meaningful names for your own symbols (never for foreign contracts).

## 6. Authoring versus reformatting


| Situation                                                                | May you change tokens? | Do this                                                                                                                                |
| -------------------------------------------------------------------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Writing new code                                                         | Yes                    | Climb the ladder; a Void should never be produced                                                                                      |
| Reformatting existing code (whitespace only)                             | No                     | Column-anchored if content indent ≤ 64; else concession layout (H6) + a one-line note; suggest the restructure so the user can choose |
| Foreign names that make the head long (YAML keys, DB columns, API paths) | No                     | Monolith if it fits; else H6 where the grammar allows; else Verbatim                                                                   |

## 7. Applying the short-head rule per language


| Language                        | Where Hybrid is forced                                                            | Short-head handling                                                                                                                                                             |
| --------------------------------- | ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| JS/TS                           | only after`return throw yield break continue async`, before `=>`, postfix `++/--` | The head is always short (`return (`). If the expression after `return` is long, assign it to a named const first.                                                              |
| Go                              | every container                                                                   | Short function names and signatures; an options struct for many parameters; named booleans for long conditions; small functions to limit nesting. Existing long signatures: H6. |
| Python                          | statement depth 0                                                                 | Alias or shorten long call heads (`Reports.Quarterly(` instead of a 50-character name); intermediate locals. Inside brackets nested containers detach freely: no Void.          |
| YAML                            | flow collections only                                                             | Prefer Monolith; long keys: Monolith or Verbatim (H7).                                                                                                                          |
| TOML                            | arrays                                                                            | Short keys; if long, H6 is fine (whitespace-insensitive).                                                                                                                       |
| HCL                             | block headers                                                                     | `resource "t" "n" {` headers are long by nature: H6 (whitespace-insensitive).                                                                                                   |
| PowerShell                      | script blocks after pipelines                                                     | Break the pipeline into named variables; short heads.                                                                                                                           |
| Haskell family                  | inside layout blocks                                                              | Short bindings; H7 applies (the closer must stay deeper than the layout column).                                                                                                |
| Ruby, Kotlin, Swift, Scala, Lua | newline-terminated statements                                                     | Same as Python: shorten heads, intermediate names.                                                                                                                              |

## 8. Worked examples

**Void (do not emit).** Content indent 72, far past the hard limit of 64:

```
Reconciliation_Report = BuildQuarterlyReconciliationReportForAccount(
                                                                        AccountId,
                                                                        QuarterStart,
                                                                        # Include reversed entries; auditors ask for them
                                                                        IncludeReversals=True
                                                                    )
```

**Reduced head (authoring).** The call moved behind a short qualified name; content indent 30:

```
Report = Reports.Quarterly(
                              AccountId,
                              QuarterStart,
                              # Include reversed entries; auditors ask for them
                              IncludeReversals=True
                          )
```

**Short head, always fine.** `return (` in JS: `(` at column 11, body at 15, closer at 11:

```
    return (
               <UserDashboardLayout>
                   <NavigationHeader />
               </UserDashboardLayout>
           );
```

**Concession (H6): reformatting an existing Go function whose signature fits the line.** The signature is a Monolith in the head, so `{` sits at column 110 and column-anchoring would put the body at 114 (a Void). Use the concession layout and say why in one sentence:

```
func ProcessAccountData(AccountId string, Options Process_Options, Logger Logger_Interface) (*Account, error) {
    Account, Fault := Query(AccountId)
    if Fault != nil { return nil, Fault }
    return Account, nil
}
```

When *authoring*, climb the ladder instead (bundle parameters, shorten the name):

```
func Process(Job Job_Spec) {
                               Account, Fault := Query(Job.AccountId)
                               if Fault != nil { return nil, Fault }
                               return Account, nil
                           }
```

## 9. Decision table


| Question                                               | Answer | Action                                                                    |
| -------------------------------------------------------- | -------- | --------------------------------------------------------------------------- |
| Does it fit one line?                                  | yes    | Monolith                                                                  |
| May the opener detach here (parse-level fact)?         | yes    | Allman (Expanded/Grouped)                                                 |
| Is the head short (content indent ≤ 40)?              | yes    | Column-anchored Hybrid                                                    |
| Can I change tokens (authoring)?                       | yes    | Ladder (§5), then Hybrid                                                 |
| Content indent 41 to 64 and no way to reduce           |        | Hybrid + note                                                             |
| Content indent > 64, tokens fixed                      |        | Concession (H6) + note, or Verbatim in whitespace-sensitive grammars (H7) |
| "It's idiomatic / everyone does it / It's my prefence" |        | Not a reason: Ignore                                                      |

## 10. Voice: how to talk about this

State the argument from geometry and cognition: shared columns, closure, scanning cost, Voids. Show side-by-side before/after. Don't attribute motives to, or insult, language designers or developers who use K&R, and don't lecture unprompted; the reasons stand on their own and persuade better without contempt. When the user asks why Hybrid is needed, answer in two sentences: *this grammar treats a newline before the opener as meaningful, which is a design choice; we comply to keep the program correct, and we keep the compliance as small as possible.*

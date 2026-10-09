# OLAFS Normative Specification (resolved edition)

Status: consolidated and conflict-free. Where the older source text contradicted itself, §11 records which statement won and why. **Monolith First** is the tie-breaker everywhere.

## Contents
1. Philosophy and goals
2. Scope, authority, application
3. Terminology
4. Containers
5. States and laws
6. Layout decision (normative procedure)
7. Geometry and whitespace
8. Comments
9. Lexical conventions (naming)
10. Color policy
11. Resolved-conflict register
12. Configuration reference
13. Conformance checklist

---

## 1. Philosophy and goals

1. **Geometry over compactness.** Every container is a room with walls. Its opener and closer are load-bearing columns.
2. **Binary honesty.** A container is flat and sealed (Monolith) or fully structured (Symmetry). No hybrids of hugging and detaching, except where a parser forces it, and then with the same column discipline.
3. **Vertical economy without hugging.** The valid objection to strict Allman is wasted lines. OLAFS answers it with Monolith First and Container Grouping, never with end-of-line openers.
4. **Whitespace is cognitive fuel.** Breaks mark concept boundaries; indentation marks nesting; matched closers give closure.
5. **Language-agnostic.** The unit of the style is the *container*, not the brace. Any matched pair of tokens that encloses code or data is subject to the same laws.
6. **Do no harm.** Formatting must never change meaning. When the style and the parser disagree, the parser wins.

## 2. Scope, authority, application

**Authority order:** (1) parser sovereignty, (2) foreign contracts, (3) explicit user instruction, (4) this specification, (5) community convention for gaps.

**In scope:** line breaks, indentation and gaps adjacent to container boundaries; naming conventions (§9); colors (§10). **Out of scope:** semantics, expression-level spacing (`a+b` vs `a + b` stays as authored), import ordering, license headers.

**Token preservation.** Output tokens equal input tokens. Permitted exceptions are *declared repairs* only: (a) Go trailing comma when a multi-line literal/argument/parameter list is expanded; (b) none else by default. Never add or remove semicolons, commas, parentheses, quotes.

**Application scope.** Apply to everything you author. In an existing file, apply to new and changed regions; leave untouched lines alone unless asked to reformat. Generated code, lockfiles, minified files, vendored code and tool-owned files (e.g. files a package manager rewrites) are Verbatim.

**Idempotence.** Layout is a pure function of (tokens, comments, config). Formatting twice equals formatting once.

## 3. Terminology

| Term | Meaning |
|---|---|
| **Container** | A matched pair of boundary tokens (the *glyph pair*) enclosing content: `( )`, `{ }`, `[ ]`, `< >` in generic/tag context, `<tag> </tag>`, `${ }`, `#[ ]`, `$( )`, backticks, `\|…\|`, `$$…$$`, `"""…"""`, and keyword pairs (`do…end`, `begin…end`, `if…fi`). Openers and closers may be multi-character. |
| **Item** | Content between separators at one container level; a sequence of text, child containers and comments. A block's items are its statements. |
| **Head** | The text and Monolith containers on the logical line before an opener (`if (Condition)`, `"Courses":`, `LoadData`). |
| **Opener / closer** | The boundary tokens. **Compound** boundary: several fused on one line (`[{(`, `)}]`). |
| **Glue** | Punctuation-only tokens that ride with a boundary and do not count as content (§6.5). |
| **Anchor** | The (line, display-column) of an opener's first character. |
| **Base indent** | The indent of the line on which the head begins. A detached opener and its closer sit at the base indent. |
| **Canvas** | The maximum display width of a line, in columns. |
| **Forced break** | A reason a container cannot be one line regardless of width (§6.2). |
| **Sole-Child Chain** | Containers C₁ ⊃ C₂ ⊃ … ⊃ Cₖ (k ≥ 2) where each Cᵢ's entire content is exactly Cᵢ₊₁. |
| **Run** | The fused boundary of a chain: an opening run (`[{(`) and a mirrored closing run (`)}]`). |
| **Continuation** | Non-glue text that follows a closer on the same logical statement (`else`, `.Method()`). |
| **Seam** | A closer immediately followed by an opener (`)(`, `)[`, `}{`). |
| **Display column** | Column measured in terminal cells: wide (CJK, many emoji) = 2, combining marks = 0, tab = to the next tab stop. |

## 4. Containers

### 4.1 Definition (character-agnostic)

Any pair of tokens, whether mirrored glyphs (`( )`) or identical glyphs (`` ` ` ``, `| |`, `$$ $$`), whether single characters or multi-character (`${ }`, `<?php ?>`), that bounds content is a container. Identical glyphs lack visual direction, so *column alignment* carries the structure for them.

### 4.2 Catalog

| Kind | Glyphs | Notes |
|---|---|---|
| Paren | `( )` | calls, parameters, grouping, tuples, conditions |
| Brace | `{ }` | blocks, objects, maps, sets, initializers, types |
| Bracket | `[ ]` | arrays, indexing, attributes `#[ ]`, `[[ ]]` |
| Angle | `< >` | only in generic/type-argument/tag contexts (edge-cases §A) |
| Tag | `<x>…</x>`, `<x />` | markup elements; attribute list is the tag's content |
| Interpolation | `${ }`, `#{ }`, `{{ }}`, `{% %}` | code inside strings/templates |
| Compound token | `#[`, `#![`, `$(`, `<(`, `%w(`, `]]`, `*/` | lexed as ONE boundary, never split across lines |
| Quote | `` ` ``, `"""`, `'''` | only as containers when they hold code; text inside is Verbatim |
| Pipe/other | `\|a, b\|` (Rust closure), `$$…$$` (SQL) | identical-glyph containers |
| Keyword pair | `begin…end`, `do…end`, `then…fi`, `case…esac`, `do…done` | see language-guide §K |

### 4.3 Not containers

Operators and punctuation that merely look like boundaries: comparison `<` `>` `<=` `>=`, shifts `<<` `>>` `>>>`, `->` `=>` `<-` `<>` `<=>` `|>`, string/char/regex delimiters when they enclose *text*, Rust lifetimes (`'a`), SQL identifier quoting, comment delimiters (comments are opaque, see §8), unpaired stray delimiters in broken input.

## 5. States and laws

### 5.1 States

1. **Monolith.** Complete on one line. Openers/closers hug their content (permitted: there is no vertical geometry).
2. **Grouped (Simplified Symmetry / Container Grouping).** A fused opening run on its own line below the head; innermost items at +indent; mirrored closing run at the same column; suffix glue after it.
3. **Expanded (Expanded Symmetry / Hierarchical).** Each container: opener alone on a line at the base indent, items one per line at +indent, closer alone at the base indent.
4. **Hybrid (Column-Anchored).** Opener stays on the head line; content at opener column + indent; closer alone at the opener's column.
5. **Verbatim.** Untouched bytes.

### 5.2 Laws

- **L1 Anti-Hug.** Outside a Monolith, an opener never ends a line that carries head text. Only Hybrid may.
- **L2 Parity.** Non-Monolith containers and runs: `col(opener anchor) == col(closer anchor)`. Sole exception: the last-resort concession layout (`hybrid-form.md` H6), used only to avoid a Void when the parser forces an inline opener.
- **L3 Return.** Each closer (run) occupies its own line start. Suffix glue and trailing comment may follow.
- **L4 Indent.** Content = anchor + indent_width. (Hybrid: anchor is the opener column.)
- **L5 No-Partial.** Monolith has no newline; non-Monolith has opener and closer on different lines. A container never has "some" of its structure expanded and the rest hugging.
- **L6 Mirror.** Closing run = reverse of opening run, adjacent characters, one column.
- **L7 Preservation.** Tokens, comment text and order, verbatim bytes unchanged.
- **L8 Idempotence.**
- **L9 No Orphan.** A line never contains only part of a compound boundary (`[{(` is always whole).
- **L10 Verbatim exemption.** Containers whose content is verbatim multi-line text (a template literal, heredoc) are exempt from L2/L3 for the part that would alter the text: only the *opener may detach from the head*; the closer stays where the text ends. (When you author a *new* template in a whitespace-insensitive language, you choose the content and may place delimiters on their own lines.)

## 6. Layout decision (normative procedure)

### 6.1 Canvas and measurement

Canvas default 160 display columns. For container C the **line width** is: base indent + head text (and any Monolith containers in the head) + C flat + C's suffix glue + the remainder of the line that cannot legally break (text up to the next separator or statement end). Trailing comments are excluded. Width uses display columns (§7.5).

### 6.2 Forced breaks (the only reasons, besides width, to leave Monolith)

1. A **line comment** (`//`, `#`, `--`, `;` in Lisp/asm, `%`, `REM`) inside the container (it would swallow code).
2. A **token containing a newline** (multi-line string/template text, heredoc, raw string, block comment spanning lines where the language needs it).
3. A **line-bound directive** inside it (preprocessor line, pragma, `//go:build`, shebang).
4. A statement-terminator constraint: the language ends statements at newlines and `;` isn't idiomatic, and the block would hold more than one statement (newline-terminated languages: max 1).
5. Quarantine: the region is unbalanced or in conditional compilation (edge-cases §E).

A **manual line break** or **blank line** in the source is NOT a forced break (R1).

### 6.3 Complexity budget

Logic never forbids a Monolith by itself. A candidate Monolith must satisfy: ≤ `max_statements` (4) statements in any single block; blocks nested ≤ `max_block_depth` (2); total containers ≤ `max_containers` (12). Counting: a statement is a `;`-terminated or newline-terminated item inside a block; a class/struct member counts as one statement; an `if/else` chain counts each clause body separately.

### 6.4 Monolith eligibility

All of: fits the Canvas (6.1); no forced break (6.2); within budget (6.3); detach policy does not prohibit flat (`MonolithOnly` contexts are always flat or Verbatim). Empty containers are always Monolith: `Constructor(A, B) {}`.

### 6.5 Sole-Child Chain and Grouping

**Fusion rule.** C fuses with D iff C's content, ignoring whitespace and *whitelisted infix glue*, is exactly one container D, with no separator, comment, label or text beside it. Applied transitively. All members share one detach policy. Angle containers do not fuse (default). Typed-literal prefixes (Dart `<String, int>{`) break a run by default (language-guide §Dart).

**Glue taxonomy** (punctuation-only, a closed per-language whitelist):

| Class | Examples | Behavior |
|---|---|---|
| G1 Compound boundary token | `#[`, `${`, `$(`, `<(`, `{{`, `]]` | part of the delimiter; never split |
| G2 Suffix glue | `,` `;` `?` `!` `...` | rides on the closer line (`),` `);` `)?;` `)!`); a fused closing run carries it once, after the whole compound |
| G3 Infix glue inside a run | spread `...`/`...?` before a bracket, address-of `&`, in languages that whitelist it | stays inside the compound line (`(&[`, `...[{`) |
| G4 Prefix operator on the head | `!` of `println!`, `?.`, `::`, `@` | belongs to the head; the opener detaches from it |
| G5 Seam | `)(` `)[` `}{` | continuation: the second opener begins a new line at the base indent with empty head |

Anything else (a label, `Name:`, an identifier, a keyword) is significant content: it ends the run.

**Rules.**
- Glue between closers (e.g. `},]`) joins a closing run only when the child is the *sole* content of its parent. With ≥ 2 items the comma is an item separator on each item's closer line.
- Grouping is **local** to a boundary run. Separating one inner container never forces its ancestors apart; after a separated region the outer closers fuse again if they are adjacent and share a column.
- **Column parity forces SCF.** Closers of a run can only share a column if the parent's closer is adjacent to the child's closer, i.e. the child is the parent's last *and only* content. Leading-child fusion (`[{( x )}, y]`) is therefore not permitted.
- A fused run is not clumping. Clumping is closers of *different containers at different columns* on one line.

**Worked example (A→B→C→D from the Boundary-Run rule).** Container numbering = containment depth.

```
final Body = jsonEncode               // head before container 1
({                                    // run: container 1 + 1.1 (1.1 is 1's sole content)
    'size': PageSize,                 // code, so 1.1.1 is NOT a sole child
    'query':
    {                                 // A: container 1.1.1 separate
        // Published articles only
        'bool':
        {                             // B: container 1.1.1.1 separate
            'filter': [{ 'term': { 'status': 'published' } }],
            // Relevance: the title outranks the body
            'must':
            [{                        // run: 1.1.1.1.1 + 1.1.1.1.1.1
                // One multi-field match
                'multi_match': { 'query': Term, 'fields': ['title^3', 'body'], 'fuzziness': 'AUTO' }
            }]                        // innermost closing run
        }                             // C: closer of 1.1.1.1 separate
    }                                 // D: closer of 1.1.1 separate
});                                   // outer closing run  })  + suffix glue ;
```

Sequence: `GROUPED → SEPARATED → SEPARATED → GROUPED(inner) … GROUPED(outer)`. Separation did not propagate.

**Multiple items** (list closes as `]);`, items stay separate):

```
final Fixture = jsonEncode
([
    { 'id': 7, 'tags': ['draft', 'archived'], }, // Soft-deleted rows must still round-trip
    { 'id': 8, 'tags': ['draft', 'archived'], }, // Soft-deleted rows must still round-trip
]);
```

The comment sits *outside* each `{ }` so each item can be a Monolith. A comment inside a container forbids Monolith for that container. Formatting never moves a comment.

### 6.6 Expansion

Opener alone at the base indent; items one per line at +indent; separators stay on their item; closer alone at the base indent, followed by suffix glue and trailing comment. Children are planned independently.

### 6.7 Hybrid

Applies when the detach-safety table (language-guide §A) says the opener must stay on the head line, or when you cannot establish otherwise **and the head is short** (item 6 below; with a long head, reduce the head or ask).

1. Opener after exactly one space (or per the keyword gap rule).
2. Content at `col(opener) + indent_width`.
3. Closer on its own line at `col(opener)`.
4. Compound openers work the same (`({` inline; compound closer `})` at the first opener's column).
5. Use spaces for the part of the offset beyond the base indent.
6. **Short heads only.** Content indent (absolute column of the body) must be ≤ 40 (25% of the Canvas); 41 to 64 is discouraged; beyond 64 column-anchored Hybrid is forbidden. Reduce the head first (Head Reduction Ladder); as a last resort with tokens fixed use the concession layout (body at base indent + 4, closer at base indent) and say why. See `hybrid-form.md`.
7. Hybrid also governs **continuation joins** where a closer must be followed by text on its line (`} else {` in Go).

### 6.8 Verbatim

String and template *text*, heredoc bodies, regex literals, raw strings, comments' text, generated/minified/vendored files, quarantined regions, and `olaf:off`…`olaf:on` regions. Bytes unchanged (including line endings).

### 6.9 Clause statements and continuation

- After a **non-Monolith closer**, continuation text starts a new line at the base indent (`.Method()`, `else`, `catch`, `finally`, `where`).
- After a **Monolith closer**, continuation stays inline.
- **Clause statements** (if/else-if/else, try/catch/finally, do/while, switch-case chains, `match` arms with blocks): either the entire statement is one Monolith line, or every clause begins its own line at the base indent and its body is decided independently.
- **Join contexts** (Go `} else {`, trailing-dot chains; any language where a newline after `}`/`)` ends the statement): continuation text stays on the closer line and the next opener is Hybrid.

### 6.10 Idempotence argument

Decisions depend only on tokens, comments, config and display columns; the output re-reads to the same tokens; blank lines between items are normalized; spacing at container gaps is stable (§7.3). Therefore `F(F(x)) = F(x)`.

## 7. Geometry and whitespace

### 7.1 Indentation
Default 4 spaces per level. With tabs: leading tabs for base indent, spaces for anything beyond ("smart tabs"), so Hybrid columns hold at any tab width. Never mix for the same line's leading whitespace except smart tabs.

### 7.2 Items and separators
One item per line in non-Monolith containers. Separator stays attached to the end of its item. Trailing separator: preserve as authored; required in Go on expansion; forbidden in strict JSON. Leading-comma style is not supported (it would put a non-glue character at line start).

### 7.3 Gaps and padding
- Monolith inner padding: braces `{ }` one space; parens, brackets, angles none. Empty: `{}` `()` `[]`.
- Gap between head and opener: if the source had no newline there, keep the author's 0 or 1 space; if collapsing from a newline, use 1 space after `if for while switch catch foreach using lock fixed with`-type keywords and before `{`, otherwise 0.
- Gap between two adjacent openers or closers (a run, a seam, `([`): none.
- Everything between tokens inside text runs is preserved as authored (collapse leading/trailing whitespace only).

### 7.4 Blank lines
Between items of a non-Monolith container: keep at most 1 (`max_blank_lines`). Trailing blank lines before a closer and leading blank lines after an opener are removed. Becoming a Monolith removes blank lines.

### 7.5 Width
Display width, not bytes or code points: East Asian wide and emoji = 2, combining = 0, tab = to the tab stop (default 4).

### 7.6 Line endings, BOM, final newline
Preserve the file's dominant line ending and BOM. End files with one newline unless the language/format forbids it.

## 8. Comments

| Kind | Rule |
|---|---|
| Leading (own line before an item) | stays directly above its item at item indent |
| Trailing (same line after item/separator/glue) | stays on that line after one space; after a fused closer it follows the whole compound |
| Dangling (inside an empty container) | block comment: keep inline (`{ /* x */ }`); line comment: container expands, comment alone on the line |
| Head-interior (between head and opener) | stays on the head line; opener drops below |
| Seam (between closer and continuation) | stays after the closer |
| Block comment inside Monolith | allowed if single-line and not a directive |

A line comment makes its container and every ancestor non-Monolith. Doc comments (`///`, `/** */`) attach to the item below. Commented-out code is a comment. Never reflow comment prose, never convert comment styles.

## 9. Lexical conventions (naming)

### 9.1 Mapping

| Entity | Convention |
|---|---|
| Function, method, variable, parameter, field, property, module, namespace, local, loop variable, label | `PascalCase` |
| Struct, enum, class, interface, trait, protocol, record, union, type alias, generic type parameter (multi-word) | `Pascal_Snake_Case` |
| Const, static, compile-time invariant, enum member *when it behaves as a constant* | `SCREAMING_SNAKE_CASE` |
| Enum variant | `PascalCase` |
| Acronym inside a name | all caps: `GetHTTPResponse`, `TargetURL`, `URLGrabber`, `XMLHTTPRequest`; types `HTTP_Request_Payload`; constants `FETCH_TIMEOUT_MS` |
| Single-letter generics `T`, `K`, `V` | unchanged |

Rationale: every user-defined identifier begins with a capital, which separates your domain logic from lowercase language keywords; shape signals role (logic, blueprint, constant) without reading the declaration.

### 9.2 Exception protocol

Keep the original name when renaming would change behavior or break an external consumer. Test: *does anything outside this file find, bind, serialize, reflect or link by this name?* If yes, keep it.

| Category | Examples |
|---|---|
| External/standard/framework API | `console.log`, `HashMap::from`, `jsonEncode`, Flutter widgets |
| Wire/data contracts | JSON keys, protobuf fields, HTTP headers, query params, DB columns/tables, CSV headers |
| FFI/export/ABI | `extern "C"`, `#[no_mangle]`, `//export`, JNI names, WASM exports |
| Entry points and special names | `main`, `__init__`, `__dunder__`, `self`, `setUp`, `tearDown`, `init` |
| Framework callbacks/overrides | `build`, `onCreate`, `componentDidMount`, `toString`, `equals`, `hashCode` |
| Test discovery | `test_*` (pytest), `TestXxx` (Go: already capitalized), `#[test]` fns are fine |
| Reflection/convention bound | JavaBean `getX/setX/isX`, ORM field↔column mapping, DI by name, React hooks `useX` |
| Visibility-by-case | Go package-level unexported names (capitalizing exports them); Ruby locals (a capital makes a constant); Haskell/Elm/Idris values (capital = constructor/type); Erlang/Prolog (capital = variable); Elixir module/atom rules |
| Language-mandated lowercase | CSS properties, HTML tag/attribute names, SQL keywords, shell builtins |

Compiler/linters that merely *warn* about casing (Rust `non_snake_case`, Dart `non_constant_identifier_names`, Python pep8-naming) are silenced with the language's allow/ignore mechanism (`#![allow(non_snake_case)]`, an `// ignore_for_file:` comment). Say so once when you introduce it.

### 9.3 Scope
Applies to identifiers you define. It does not apply to string contents, file names, CSS classes, URL paths, env vars (`UPPER_SNAKE` stays), or documentation prose.

## 10. Color policy

- **Banned absolutely:** Yellow, Gold, Amber, Chartreuse, Warm/Light Yellow, Mustard, Olive Yellow; HSL hue 45° to 65°; `#FFFF00`, `#FFD700`, `#BB7B00`, `#FEF8E7`, `#FFF000`; CSS names `yellow`, `gold`, `khaki`, `lightyellow`, `lemonchiffon`, `palegoldenrod`, `goldenrod`, `darkgoldenrod`.
- **Applies to:** code, syntax themes, UI, CSS variables, icons, badges, ratings, charts, diagrams, alerts, AI responses.
- **Approved:** Warning Orange `#FC6A03`; Error Bright Red `#FF1744`; Pink/Magenta `#FF4081`; Highlight Cyan `#00E5FF`; Azure `#00BFFF`; Emerald `#00E676`; Violet `#C77DFF`; Indigo `#3D5AFE`.
- **Case:** hex digits are uppercase, always.
- **Contrast:** why yellow is banned: it lacks contrast on light themes and bleeds on dark ones.
- **Conflicts:** a user request for yellow gets the "this contradicts your style, nearest approved color is X, confirm?" reply. Third-party or brand-mandated yellow in code you didn't author is left alone and mentioned, not silently changed.
- Colors inside Verbatim text (strings you don't own) are not altered; colors you author are.

## 11. Resolved-conflict register (superseded text → ruling)

Each ruling is normative. Rationale: Monolith First.

| # | Source conflict | Ruling |
|---|---|---|
| R1 | "Vertical Break Trigger / Vertical Leak / for any reason (length, clarity, or preference)" force expansion on any manual break **vs** "Monolith First", "Keep it flat until the canvas forces a break", "If you can write it on one line, do it". | **Monolith First.** Layout is canonical: independent of how the source was broken. Forced breaks are only line comments, newline-bearing tokens, line-bound directives, newline-statement constraints and quarantine. |
| R2 | "Complexity Trigger: control flow ⇒ Expanded mandatory" and "contains no internal branching" **vs** the Monolith examples containing `if` and 2–4 statements. | Complexity Budget (§6.3). The budget, not the presence of logic, decides. |
| R3 | Grouping "only for purely declarative data" **vs** "whenever nothing is between"; Corollary "contains multiple containers, logic or code ⇒ must expand" (collides with Monolith First); "all must expand" **vs** Boundary-Run locality. | Grouping applies to any Sole-Child Chain (§6.5). The Corollary means *leaves Grouped form* (it may still be Monolith). Separation is local. Old text "any internal delimiter expanded ⇒ all must expand" and "NOT Allowed (ALLOWED in only some Exceptions)" are void. |
| R4 | Empty `{}` on its own lines (constructor example) **vs** `Constructor(...) {}` inline. | Empty containers are always Monolith and inline. |
| R5 | `[ A, B, C, D ]`, `[{( Data )}]` padded **vs** `[Alpha, Beta, Gamma]`, `Array[asd,…]` unpadded. | Braces one space; paren/bracket/angle none (§7.3). |
| R6 | "If this formatting breaks functionality … do not use it" **vs** re-indenting template-literal text, and the "Expanded Template" example that inserts braces/newlines into string text. | Text inside strings/templates is never re-indented or edited. Only `${ }`-style interpolation code is a container. Opt-in reindent per whitespace-insensitive embedded language. |
| R7 | "Canvas Breach does NOT fire at 120" **vs** examples with "(Limit is 120)", "60, 80, or 120", "140 or 160". | One user-defined Canvas, default 160. Other figures are illustrative. |
| R8 | "Opening Hugging (Forbidden)" **vs** Monolith "Containers may hug content". | Hugging is forbidden only outside a Monolith. |
| R9 | "Hybrid states are strictly forbidden" **vs** "Compulsory Inline Open". | Hybrid forbidden except where the grammar forces it; then Column-Anchored Hybrid is the legal hybrid, for short heads only (R15). The concession layout is tolerated only under `hybrid-form.md` H6. |
| R10 | "Closing Clumping (Forbidden): `});` `}}]`" **vs** "Mandatory Symmetrical Return … `)}]`". | A fused run at one column (+ suffix glue) is allowed; clumping is mixed-column closers. `})` + `;` is a fused run with glue. |
| R11 | Two-per-line `'Change', function(...)` + detached `{` **vs** one item per line. | One item per line in every non-Monolith container. |
| R12 | Fully expanded JSON sample (small objects/arrays expanded though they fit) **vs** Monolith First. | Small children are Monolith inside an expanded parent (see SKILL.md gallery). |
| R13 | `else` after `}` cuddled vs on a new line; Go requires join. | §6.9: all clauses on own lines unless the whole statement is Monolith; join where the grammar demands. |
| R14 | PowerShell example using a trailing backtick continuation. | Continuation characters are token insertions: not used. Hybrid instead. |
| R15 | Column-anchored Hybrid accepted at any depth ("accept the drift", warn at column 120) **vs** the Void it creates on long heads. | Hybrid is for short heads only: soft limit 40, hard limit 64, Head Reduction Ladder, concession layout as a last resort (`hybrid-form.md`). |

## 12. Configuration reference

```toml
canvas_width = 160
indent_width = 4
indent_style = "spaces"            # spaces | tabs (smart tabs)
tab_stop     = 4
line_ending  = "auto"
max_blank_lines = 1

[monolith]
max_statements   = 4               # 1 for newline-terminated languages
max_block_depth  = 2
max_containers   = 12
max_items        = 0               # 0 = off; optional cap on items in one Monolith

[grouping]
enabled = true
scope   = "any"                    # any | declarative_only
fusion  = "sole_child"             # leading_child is not supported
allow_partial = false

[padding]
paren = 0
bracket = 0
brace = 1
angle = 0

[hybrid]
anchor = "opener_column"           # column-anchored while the head is short
soft_indent_limit = 40             # 25% of canvas; discouraged beyond
hard_indent_limit = 64             # 40% of canvas; column-anchoring forbidden beyond
concession = "line_indent"         # last resort only (hybrid-form.md H6)

[angle]
enabled = true
allow_grouping = false
span_cap = 400                     # chars searched for the matching '>'

[embedded]
reindent = false                   # per language: embedded.css.reindent etc.

[continuation]
default = "newline"                # join where the grammar demands

[on_error]
mode = "skip"                      # skip | fail
max_error_ratio = 0.05
```

## 13. Conformance checklist

A document conforms iff: L1–L10 hold; every Hybrid is justified by a parser reason; tokens/comments/verbatim are preserved (apart from declared repairs); every container is in exactly one state; naming follows §9 or a documented exception; no banned color; hex digits uppercase; and formatting it again is a no-op.

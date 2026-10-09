# LAFS Edge Cases and Their Resolutions

Principle behind every entry: **the grammar decides when it can; heuristics decide only when it can't; uncertainty fails open.** An unrecognized container costs a little formatting. An invented container can corrupt layout or meaning.

## Contents
- A. `<` and `>`: angle brackets versus operators
- B. Unpaired characters
- C. Comments
- D. Strings, templates, heredocs, regex, interpolation
- E. Preprocessors, conditional compilation, macros
- F. Newline-sensitive and layout-sensitive parsers (detach safety)
- G. Chains, continuations, clause keywords, seams
- H. Identical-glyph and keyword-pair containers
- I. Degenerate shapes (empty, single item, same-glyph nesting, deep nesting)
- J. Markup and whitespace-sensitive content
- K. Embedded languages and injections
- L. Broken, partial and snippet code
- M. Unicode, tabs, line endings
- N. Large data, tables, long literals
- O. Token repairs
- P. Generated, minified, tool-owned files
- Q. Existing code and diff hygiene
- R. Cross-language quick reference of hazards

---

## A. `<` and `>`: angle brackets versus operators

### A.1 Lex operators first (longest match)
Never treat these as a lone angle boundary: `->` `=>` `<=` `>=` `<<` `>>` `>>>` `<<=` `>>=` `<=>` `<>` `<-` `|>` `<|` `::<` (this is `::` followed by an opener, see turbofish) and shell redirections `<<EOF` `2>&1` `>>file` `<(`.

**Closer splitting.** While an angle candidate is open, a `>>`, `>>>`, `>=` or `>>=` closes one level at a time: `Vec<Vec<u8>>`, `Map<K, List<V>>`, Dart `Map<String, List<Map<String, Object>>>` (Dart has a real `>>>` operator, so the three `>` must be split), C++11 `vector<vector<int>>`. In formatting, the characters stay adjacent and unmodified; splitting is only an interpretation.

### A.2 Resolution tiers
**Tier 1 (authoritative): the grammar.** If you know the language, you know which positions are generics/tags: type-argument and type-parameter lists, template argument lists, markup tags, JSX elements, `new Foo<T>()`, `::<T>`. Every other `<`/`>` is an operator. No heuristics.

**Tier 2 (no grammar knowledge, or the span is damaged): contextual scoring.** A candidate `<` is an angle opener only when ALL hold:
1. **Left context.** The previous significant token is an identifier, `::`, `.ident`, or a generic-introducing keyword (`template`, `typename`, `fn`, `impl`, `struct`, `enum`, `trait`, `class`, `interface`, `new`, `extends`, `implements`, `where`). Not after a literal, `)`, `]` (so `a[i] < b` and `f(x) < y` stay operators), or an operator.
2. **Clean scan.** Scanning right, skipping balanced `() [] {}` groups wholesale, you find a matching `>` (nesting `<>`) within the span cap (400 characters, no statement terminator). Abort on an unmatched closer, `;`, `&&`, `||`, `==`, `!=`, `===`, `!==`.
3. **Follow-token test.** The token after the matching `>` is one of: `( ) ] } : ; , . ? == != | ^ && || & [ { ::`, end of line, or end of file (the C# specification technique; Dart extends it for constructor tear-offs). A following identifier, number or `<`/`>`/`=` usually means comparison.

A whitespace shape (`a<b` versus `a < b`) is a tie-breaker only, never decisive.

**Tier 3 (fail open).** Anything unresolved is an operator: plain text, never a container.

### A.3 Layout of angle containers
Angle containers are Monolith by default, expand only when the Canvas is breached, break only at top-level commas, and **never fuse** into runs (`angle.allow_grouping = false`). A Monolith angle container in a head belongs to the head, so a following `{` detaches below it:

```
final Inventory = <String, List<Map<String, Object>>>
{
    'fruit': [{ 'name': 'apple', 'weight': 182 }],
};
```

### A.4 Conformance corpus

| Language | Input | Verdict |
|---|---|---|
| Rust | `let V: Vec<Vec<u8>> = X;` | two angle containers (closers split) |
| Rust | `X.parse::<i32>()` | angle (turbofish) |
| Rust | `if A < B && C > D {` | operators |
| Rust | `A as usize < B` | the grammar reads `<` as generics here: follow the grammar |
| C++ | `vector<vector<int>> V;` / `X >> 2` / `1 << 3` | angle ×2 / operator / operator |
| C++ | `template<typename T> void F();` | angle |
| C++ | `A<B>(C)` | angle by follow-token `(`; if the name is not a template it reads as comparisons, but the layout consequence is benign (short span) |
| C++ | `operator<`, `operator<<` | operator tokens |
| C++ | `[]<typename T>(T X){}` | angle after a lambda introducer |
| TS | `Array<{ a: number }>` | angle; nested braces skipped as a balanced group |
| TS/TSX | `const F = <T,>(X: T) => X;` | angle; the comma disambiguates from a JSX tag |
| TSX | `<div className="a">…</div>` | tag container |
| TS | `A < B || C > D` | operators (`||` aborts the scan) |
| Java | `Collections.<String>emptyList()`; `new ArrayList<>()` | angle; diamond `<>` is an empty angle container |
| Java | `for (int I = 0; I < N; I++)` | operator (`;` aborts) |
| C# | `F(G<A, B>(7))` | angle (follow `(`): the language spec's own example |
| C# | `if (A < B && C > D)` | operators |
| Dart | `F(A<B, C>(D))` | angle, generic call (follow `(`) |
| Go | `Ch <- V`, `A < B` | operators (generics use `[ ]`) |
| SQL | `A <> B`, `X <= 3` | operators |
| Shell | `cat <<EOF`, `2>&1`, `<(Cmd)` | redirections; `<(` is a compound opener |
| HTML | `<br>`, `<!-- … -->`, `<![CDATA[ … ]]>` | void tag (leaf), comment (verbatim), CDATA (verbatim) |
| Templates | `{% if A < B %}` | `{%` container; inner `<` operator |
| Prose/Markdown | `X < Y`, `<https://…>` | text; autolink is a leaf |

### A.5 Tags are not heuristics
HTML/XML/JSX tag boundaries are grammar facts. A tag is a container: opener `<name attrs>`, closer `</name>`; self-closing `<name />` is a Monolith leaf; void elements (`br img input meta link hr`) are leaves; elements with optional end tags (`li p td`) are leaves if unclosed. Sloppy `a < b` inside text content is text.

---

## B. Unpaired characters

"Unpaired" covers three different things.

### B.1 Glue: legitimately unpaired, rides with a boundary
Whitelisted punctuation (spec §6.5 classes G1–G5): compound boundary tokens (`#[`, `${`, `$(`), suffix glue (`, ; ? ! ...` after a closer), infix glue allowed inside a run (spread `...`, address-of `&` in some languages), prefix operators that belong to the head (`!` in `println!`, `?.`, `::`), and seams `)(`.

**Placement in the template** (opening slots 1..n before, between and after openers; closing slots mirror):
- slot before container 1: travels with the compound line if it is G1/G3 (`...[{`), stays on the head if G4 (`println!` then `(`);
- slots between openers: only whitelisted G3, else the run is broken;
- slots after the last opener: nothing but whitespace/comments (a label there is content and the opener is not a run member);
- closing slots: suffix glue after the whole fused closing run, once.

### B.2 Delimiter lookalikes in non-code context
Never structural: character literals `'('`, `"}"`, `r#"{"#`, `b'['`, Lisp `#\(`, Rust lifetimes `'a`, regex bodies `/[(]/`, SQL `''` escapes, shell `case X)` patterns, Markdown list `1)`, emoticons in comments, escaped brackets in LaTeX `\{`, brackets inside comments and strings. Rule: *classify by lexical mode before counting brackets.*

### B.3 Genuinely unbalanced
Syntax errors, half-typed code, preprocessor arms (§E), macro fragments. Evaluate balance per statement-sized window. An opener without a closer in the window (or the reverse) is marked **Unpaired**: treat it as ordinary text, never as a run member. The enclosing item is quarantined (Verbatim); well-formed neighbors still format.

### B.4 Stray characters at a boundary
A character glued to a closer that is not on the whitelist (`)#`, `}%`) is content: it ends the run and goes with the following continuation text.

---

## C. Comments

- Classify before formatting: leading, trailing, dangling, head-interior, seam (spec §8).
- A line comment forbids Monolith for its container and all ancestors. A block comment is fine in a Monolith if it is single-line and not a directive.
- **Never move a comment across a token.** The only legal movement is the container's own re-layout: `if (X) { // note` becomes `if (X)` newline `{ // note`, the comment staying after the opener it followed.
- Comments in a fused run attach after the compound: `)}] // end of config`.
- Comments between `}` and `else`: `} // old branch` newline `else`: stays after the closer.
- Region directives (`olaf:off` / `olaf:on` / `olaf:skip-next`) switch to Verbatim.
- Comment prose, markers (`TODO`, `FIXME`), banner lines and ASCII art: untouched.
- Commas or glue *after* a trailing comment cannot exist (the comment ends the line): if the source has an item, comment, then comma on the next line, keep the author's order; do not "fix" it.
- Nested block comments (`/* /* */ */` in Rust/Swift/Haskell) obey the language's nesting rule.

## D. Strings, templates, heredocs, regex, interpolation

- **Text is Verbatim.** Whitespace inside a string, template, heredoc, raw string or regex is part of its value. Never re-indent, trim, reflow, add braces, or move delimiters inside the text (R6).
- A token containing a newline forces its container out of Monolith and pins its continuation lines at their original columns (they are not re-indented with the surrounding code).
- **Interpolations are code containers.** `${ Expr }`, `#{ Expr }`, `{Expr}` in f-strings/JSX: whitespace inside is code, so they may be broken and indented. Nested templates inside `${}` follow the same rule recursively.
- **Opening-delimiter detach is allowed** for a multi-line template (`const X =` newline `` `text… ``): it only alters whitespace *before* the string. The closer stays where the text ends (L10).
- **Authoring new templates** in whitespace-insensitive content (SQL, CSS, HTML, GLSL), you may write the delimiters on their own lines and indent the body, because you choose the content.
- **Heredocs** (`<<EOF`, `<<-EOF`, `<<~EOF`, PHP/Ruby/Perl forms): the body begins on the line *after* the statement. The line holding the start marker is **line-pinned**: do not break or join lines around it; containers on it keep their line structure. `<<~` squiggly heredocs strip indentation: re-indenting the body changes nothing semantically in Ruby but do not touch it anyway.
- **Raw strings and triple quotes** (`r#" "#`, `"""`, `'''`): Verbatim; an inner `"` or brace is not structural.
- **Tagged templates** (`Sql\`…\``, `css\`…\``): same as templates. Embedded-language reindent only if enabled.
- **Regex literals** (`/…/`, `m{…}`, `%r{…}`): Verbatim; bracket characters inside are not containers. Division vs. regex is decided by the grammar (a `/` after an operand is division).
- **Python f-strings, JS template strings, C# `$"…"`, Kotlin `"…${}"`, Swift `"\( )"`:** text Verbatim, expression parts code. For single-line interpolations, leave them flat; never introduce newlines in a string that wasn't multi-line.

## E. Preprocessors, conditional compilation, macros

- **Preprocessor lines** (`#if #ifdef #else #elif #endif #define #include #pragma #region`) are line-bound: they start and end at line boundaries and they force expansion of any container they sit in.
- **Branch-balance rule.** Count the net bracket delta of each arm of an `#if` group. If all arms agree, treat the group as ordinary text. If they disagree (e.g. one arm opens `{`, the other doesn't), **quarantine** the whole group and every container whose span overlaps it. Containers fully outside are formatted normally.
- **Multi-line `#define`** (backslash continuations): the body is Verbatim unless you wrote the macro yourself and it is bracket-balanced code. Never move a `\`.
- **Rust:** attributes `#[ ]`/`#![ ]` are compound-token containers; `cfg` is balanced and needs no quarantine; macro bodies (`macro_rules!`, procedural macro input) are token trees: format only the outer container, treat the inside as Verbatim unless it is a known macro with ordinary syntax (`vec![ ]`, `println!( )`, `format!( )`).
- **C#:** `#if/#region` as above. **PHP:** `<?php … ?>` tags delimit code from Verbatim HTML. **Templating** (`{{ }}`, `{% %}`, ERB `<% %>`, JSP `<% %>`): the tag is a container with Verbatim host text around it.

## F. Newline-sensitive and layout-sensitive parsers

Detaching an opener inserts a newline between head and opener. Three hazard classes:

1. **Statement-terminating newlines** (Go, Python at depth 0, Ruby, Kotlin, Swift, Scala, Lua, R, shell commands, PowerShell, JS after restricted keywords). A detached opener may end the statement early or start a new one.
2. **Continuation hazards after closers** (Go `} else`, trailing-dot chains; Python implicit joins only inside brackets).
3. **Offside/layout rules** (Haskell, F#, Elm, Idris, Nim, YAML, Python's own indentation, Makefile recipes): a line at or left of the layout column starts a new construct. A closer at the base indent can end the block it is inside.

**Resolution.** Consult the detach-safety table in `language-guide.md` §A. If the language/context is not listed or you are not certain, **use Hybrid if the head is short** (opener on the head line, content at opener column + 4, closer at the opener's column; see `hybrid-form.md` for the limits). With a long head, reduce the head or ask instead. Hybrid is safe in all three classes because it matches the layout most code already uses and keeps every continuation line deeper than the layout column.

**Inside enclosing brackets, newlines are usually free** (Python, most newline-terminated languages ignore newlines inside `( )`/`[ ]`/`{ }`): a nested container whose parent is an expanded bracket container may often detach even in Python. Confirm per language in the guide.

## G. Chains, continuations, clause keywords, seams

- After a **non-Monolith closer**, non-glue text starts a new line at the **base indent** (not +4): method chains `.Baz(...)`, `else`, `catch`, `finally`, `where`, `while` of do/while, Dart cascades `..Add(...)`.
- After a **Monolith closer**, text stays inline.
- A chain with several expanded calls produces repeated `)` newline `.Next` newline `(` blocks at one indent. That is the intended rectangle-per-call look.
- **Clause statements** (§6.9): all-Monolith statement or every clause on its own line.
- `else if (B)` is one clause head. `case X:` / `default:` labels are heads of their item, not containers.
- **Seam** `)(`, `)[`: second opener starts a new line with empty head and base indent.
- **Join contexts** (Go, any parser that ends the statement on `}`/`)` + newline): continuation stays on the closer line, so the next opener is Hybrid: `} else {`.
- **Ternaries / `&&`/`||`/`?:` expressions spanning lines:** operators are text; they stay at the start/end of a head as authored; LAFS does not reflow expressions.
- **Arrow functions:** `(Args) =>` is the head; a following `{` detaches below it. A newline *before* `=>` is illegal in JS/TS; never produce it.

## H. Identical-glyph and keyword-pair containers

- **Identical glyphs** (`|a, b|` Rust/Ruby closures, `$$…$$`, backticks, `"""`): structure is carried by column. Monolith if short; otherwise opener on its own line (or Hybrid if the grammar forces), content +4, closer at the opener's column. Distinguish bitwise `|` and division `/` from boundaries by grammar only.
- **Keyword pairs** (`begin…end`, `do…end`, `if…fi`, `case…esac`, `do…done`, Ruby `def…end`, Lua `then…end`, Pascal, Ada, Fortran, VB `If … End If`, SQL `CASE … END`, `BEGIN … END`): same states. Monolith when the language allows it on one line. Expanded (Allman) when the opener keyword may legally sit alone on a line (shell `then`/`do`, Lua, Pascal, Ada); **Hybrid** when it may not (Ruby `do`, Elixir `do`): the `end` goes at the keyword's column. Lower confidence: verify against the language.

## I. Degenerate shapes

- **Empty** `()` `[]` `{}` `<>`: Monolith, inline. With a block comment: `{ /* x */ }`; with a line comment: expand.
- **Single item:** budget applies; if it's a sole child, it's a chain link.
- **Same-glyph nesting** `((A))`, `[[A]]`: chain, fusion allowed (`(( … ))`) unless `[[ ]]` is a compound token (then it's G1).
- **Deep nesting** (> 6 levels): no special rule; Hybrid drift is the only risk. If the Canvas is breached by indent alone, introduce named intermediates or shorten heads; do not violate L2.
- **A container too wide even when Expanded** (a single long string argument): leave the long token alone; the line may exceed the Canvas. Never break a token.
- **Item alone exceeding the Canvas** in a Monolith-eligible container: expand; the item stays on its line.

## J. Markup and whitespace-sensitive content

- Element = container (start tag, children, end tag). Attribute list = items of the start tag.
- Expanded start tag:

```
<Button
    Class="primary"
    OnClick="Submit"
>
```

  (`>` alone at the `<` column, L2 parity.) Self-closing: `/>` on its own line at the `<` column.
- **Inline/text content is whitespace-sensitive.** Do not add or remove whitespace between text and inline elements, and never split a text run. Only break between *element/expression children*. `<pre>`, `<textarea>`, `<script>`, `<style>`: Verbatim (or embedded language).
- **JSX:** text children are trimmed/joined by JSX rules; never reflow them. `{" "}` is meaningful. `return (` is Hybrid. Conditional children `{A && (<X />)}` are expression containers; their parens follow the normal states.
- **Vue/Svelte/Angular templates:** directives `{#if}`, `*ngIf="…"`: treat the quoted expression as Verbatim; block directives as keyword/container pairs.
- **XML:** namespaces and entities untouched; mixed content like HTML text. **CDATA/PI/doctype:** Verbatim.

## K. Embedded languages and injections

Host-language formatting treats the guest as an opaque leaf by default. Guest formatting is opt-in per language (`embedded.<lang>.format`, `.reindent`). Examples: HTML `<script>`/`<style>`, Markdown fences, JS tagged templates (SQL, CSS, GraphQL, HTML), PHP in HTML, JSX in JS, SQL in strings, regex, JSON in YAML block scalars. When you do format a guest, format it *as that language* with its own detach-safety table, and keep its byte-significant whitespace.

## L. Broken, partial and snippet code

- **Snippets without context** (a lone expression, a fragment of a class): format by tokens and containers; do not invent surrounding syntax.
- **Pseudocode and prose-embedded code:** apply the same laws; do not "fix" the pseudocode.
- **Syntax errors:** format well-formed parts, leave the erroneous item Verbatim, do not repair, mention it once.
- **Mid-edit code with unbalanced openers:** quarantine the unbalanced window.
- **Mixed-language files** (shell with embedded awk, Makefile with shell): format by region; unknown regions Verbatim.

## M. Unicode, tabs, line endings

- Display width per §7.5: CJK and emoji = 2 columns; combining marks 0; zero-width joiners 0. Right-to-left text: widths as stored; do not reorder.
- Identifiers with non-Latin scripts: capitalization rules (§9) apply only to cased scripts; scripts without case are untouched.
- Tabs inside head text count to the tab stop; indentation uses smart tabs if tabs are required (Makefiles, Go by habit): leading tabs for base indent, spaces beyond, so L2 holds at any tab width.
- Mixed line endings: normalize to the dominant one; Verbatim spans keep theirs. Preserve BOM and final-newline policy.
- Form feeds, vertical tabs, NUL: leave in Verbatim.

## N. Large data, tables, long literals

- An array/map that exceeds the Canvas expands to one item per line (R11): a 1,000-number array is 1,000 lines. That is correct under the spec.
- **Optional extension, off by default: Table Mode.** For homogeneous scalar literals (numbers, short strings), rows of items filling the Canvas, each row a line at +4, no row split inside an item. Enable only if the user asks; the container is still Expanded (L2/L3 hold).
- **Long string literals:** never break a string. If an item exceeds the Canvas, leave it.
- **Lines of base64/hex/SVG paths:** Verbatim.
- **Code generated from data** (enums, lookup tables): format only if you authored it; otherwise Verbatim.

## O. Token repairs (the only permitted edits)

| Language | Repair | When |
|---|---|---|
| Go | add a trailing `,` after the last element | expanding a composite literal, argument list or parameter list so the closer begins a line |
| (none by default) | | |

Not repairs: adding/removing semicolons, quotes, parentheses, line-continuation characters (backslash, backtick, `_`). Rewrite the layout to avoid the need (usually Hybrid).

## P. Generated, minified, tool-owned files

Verbatim: lockfiles (`package-lock.json`, `Cargo.lock`, `yarn.lock`), minified bundles, source maps, protobuf/gRPC generated code, ORM migration snapshots, `.git*` internals, files with a "DO NOT EDIT" header, vendored dependencies, and files a tool rewrites on every run (a package manager's `package.json` reformat, `go.mod`, `go.sum`). For tool-owned JSON/TOML, follow the tool's canonical shape so you don't create churn; format your own additions to match.

## Q. Existing code and diff hygiene

- Reformat only what you write or change unless asked for a whole-file pass.
- When a change alters a container's state (a comment added, a line grew past the Canvas), re-lay out that container and its direct parent, nothing else.
- When asked to **port or rewrite** into LAFS, preserve tokens and comments; run the checklist; report states you chose where it wasn't obvious (Hybrid forced, a run broken by a label).
- If CI enforces another formatter, say so once and ask before diverging.

## R. Cross-language hazard quick reference

| Hazard | Where | Rule |
|---|---|---|
| ASI after `return throw yield break continue async`, before `=>`, around postfix `++/--` | JS/TS | Hybrid for the opener |
| Semicolon injected after identifier/literal/`)`/`]`/`}` at line end | Go | Hybrid everywhere; join continuations; trailing comma repair |
| Statement ends at newline at depth 0 | Python, Ruby, Kotlin, Swift, Scala, Lua, R | Hybrid at depth 0; free inside brackets |
| Offside rule | Haskell, F#, Elm, Idris, Nim, YAML | Hybrid (closer deeper than the layout column) |
| Inline tables single-line | TOML 1.0 | Monolith only |
| Script block must follow command on same line | PowerShell | Hybrid |
| Multi-line `#define` | C/C++ | Verbatim |
| `>>` / `>>>` closing generics | C++, Java, Rust, Dart | split closers |
| Heredoc body after the line | shell, Ruby, PHP, Perl | line-pin |
| Whitespace-significant text | HTML, JSX, Markdown, template text | never reflow |
| Case changes meaning | Go exports, Ruby constants, Haskell/Elm, Erlang/Prolog | exception protocol (spec §9.2) |

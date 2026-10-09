# OLAFS Language Guide

How the laws apply to specific languages: whether an opener may detach, what glue exists, hazards, and examples. Applies the rules in `specification.md`; resolves the edge cases in `edge-cases.md`.

## Contents
- A. Detach-safety matrix
- B. Procedure for an unfamiliar language
- C. JavaScript, TypeScript, JSX/TSX
- D. Go
- E. Python
- F. Rust
- G. C and C++
- H. C# and Java
- I. Dart (§Dart)
- J. Data and markup formats: JSON, YAML, TOML, XML/HTML, CSS, SQL
- K. Keyword-pair and newline-terminated languages: Ruby, Lua, shell, PowerShell, Kotlin, Swift, Scala, HCL, Haskell family

---

## A. Detach-safety matrix

*Detach* = may the opener move to its own line (Allman)? If not, the container is **Hybrid** (opener stays on the head line; closer at the opener's column). Confidence: ✔ checked against the language specification; ◐ well-established rule, not re-verified here; ⚠ expected, confirm when you can (and until then use Hybrid).

| Language | Detach opener | Continuation after closer on a new line | Monolith statements | Conf. |
|---|---|---|---|---|
| C, C++, C#, Java, Rust, Dart, PHP, Objective-C, Zig, Solidity | Free | Free | 4 | ◐ |
| JSON, JSON5, JSONC, CSS/SCSS/Less, GraphQL, Protobuf, SQL, Lisp family, OCaml | Free | Free | 4 (n/a for data) | ◐ |
| JavaScript / TypeScript | Free, **except** after `return throw yield break continue async`, before `=>`, around postfix `++`/`--` (then Hybrid) | Free | 4 (1 in no-semicolon style) | ✔ |
| Go | **Hybrid always** (semicolon injected after identifier, literal, `)`, `]`, `}`, `return`, `break`, `continue`, `fallthrough`, `++`, `--`) | **Join** (`} else {`; trailing dot for chains) | 1 | ✔ |
| Python | Hybrid at statement depth 0; Free inside any enclosing `( [ {` | n/a (indentation blocks aren't containers) | 1 | ◐ |
| Ruby, Kotlin, Swift, Scala, Lua, R, Julia, Groovy, Elixir | Hybrid at depth 0; usually Free inside enclosing parens | Join for `end`/`else` forms | 1 | ⚠ |
| Haskell, F#, Elm, Idris, Nim (offside rule) | Hybrid (closer deeper than the layout column) | Join | 1 | ◐ |
| Shell (sh/bash/zsh) | Free for function bodies and compound-command keywords; command arguments can't break without `\` (not allowed) → Monolith/Verbatim | Free | 4 (`;` is idiomatic) | ◐ |
| PowerShell | Hybrid (script block must follow the command on its line) | Free | 4 | ◐ |
| HCL/Terraform | Hybrid (block `{` stays on the header line) | Free | 1 | ⚠ |
| YAML | block style: don't touch; flow `[ ]`/`{ }`: Monolith, else Hybrid | n/a | n/a | ◐ |
| TOML | arrays: Hybrid; inline tables: Monolith only (1.0) | n/a | n/a | ◐ |
| HTML/XML/Vue/Svelte | Free for tags and attributes; text content whitespace-sensitive | Free | n/a | ◐ |
| JSX/TSX | tags Free; `return (` Hybrid | Free | n/a | ✔ |
| Makefile, Dockerfile, shell continuation lines | Verbatim (tabs/backslashes are syntax) | n/a | n/a | ◐ |

Ground rules for the table: (1) **uncertainty ⇒ Hybrid, but only for short heads** (see `hybrid-form.md`: Void limits, Head Reduction Ladder, concession layout); (2) formatting never adds `\`, backtick, `_` continuation characters; (3) multi-statement Monoliths need an idiomatic separator, otherwise 1 statement.

## B. Procedure for an unfamiliar language

1. Identify the container glyph pairs (and keyword pairs) and what the language treats as comment, string, char, regex.
2. Is a newline significant at statement level? Does the language have semicolons, offside indentation, or restricted productions? If you cannot say no, treat opener detaching as unsafe ⇒ Hybrid, provided the head is short (`hybrid-form.md`).
3. Decide the glue whitelist: `,` `;` and any suffix operators (`?`, `!`, `...`) that appear after closers.
4. Check `<`/`>`: does the language have angle-bracket generics or tags? If yes, apply edge-cases §A; if no, they are operators.
5. Apply Monolith First, budget, Sole-Child Chain, Expanded/Hybrid.
6. Keep tokens and comments unchanged; keep text inside strings/templates Verbatim.
7. State the assumption in one sentence if you guessed (for example, "treated newlines before `(` as significant, so Hybrid").

---

## C. JavaScript, TypeScript, JSX/TSX

- **Restricted productions** (no newline allowed): `return`, `throw`, `yield`, `break`, `continue`, postfix `++`/`--`, `async` before `function`/arrow, and before `=>`. For `return ( … )`, `throw ( … )`, `yield ( … )` use Hybrid.
- **A line starting with `(`, `[` or a backtick continues the previous expression**: a detached `(` is still a call.
- **Arrow functions:** `(Args) =>` is the head; the body `{` detaches. Never break before `=>`.
- **Glue:** `,` `;` `?` (`?.` is head), `!` (TS non-null `)!`), spread `...`.
- **Angles:** generics `<T,>` in TSX arrows, type assertions in `.ts` only, JSX tags.
- **Semicolon style:** preserve the author's. No-semicolon style ⇒ Monolith holds one statement.
- **Template literals / tagged templates:** Verbatim text; `${ }` interpolation is a container.
- **Objects vs blocks:** at statement start `{` is a block; object literals appear in expression position.
- **Destructuring and import/export lists** are containers; `import { A, B } from 'x'` is a Monolith if it fits.
- **Naming exceptions:** hooks `useX`, React components already PascalCase, `constructor`, `render`, lifecycle methods, DOM/Node/browser API names.

```
async function Load(Id)
{
    const Response = await Fetch
    (
        `/api/items/${Id}`,
        // Never cache: stock levels change by the minute
        { method: 'GET', cache: 'no-store' }
    );
    return (
               Response.ok
               ? await Response.json()
               : null
           );
}
```

(`Fetch`/`fetch` is an external API: in real code keep `fetch`. The example shows structure.)

## D. Go

- Every container is **Hybrid** (the lexer injects `;` after a line ending in an identifier, literal, `)`, `]`, `}` and a few keywords). Brace on the head line; closer at the opener's column.
- **Join continuations:** `} else {`, and method chains put the dot at the **end** of the line.
- **Declared repair:** trailing comma after the last element when a literal, call or parameter list expands.
- **Monolith:** `if X { return }` is fine when it fits (statements: 1; `;` is not idiomatic).
- **Naming exceptions:** exported = capitalized, unexported package-level names must stay lowercase (capitalizing exports them); `main`, `init`, `TestXxx`, struct tags and JSON field tags are contracts.
- Drift warning: nested Hybrid walks right; keep functions small.

```
func Run() {
               Account, Fault := Query(Id)
               // Surface the fault to the caller unchanged
               if Fault != nil {
                                   return nil, Fault
                               }
               return Account, nil
           }
```

`{` at column 11 ⇒ body at 15, `}` at 11. The inner `if {` is at column 31 ⇒ body at 35, `}` at 31.

## E. Python

- Indentation blocks (`if X:`, `def`, `class`, `with`) are **not containers**: never alter them.
- Containers are `( ) [ ] { }` only. At statement depth 0 a detached opener ends the statement ⇒ **Hybrid**. Inside an enclosing bracket newlines are ignored ⇒ nested containers may detach freely.
- **Monolith:** 1 statement (compound `;` lines aren't idiomatic).
- **Glue:** `,` `:` is not glue (it's content); `*`/`**` unpacking prefix is a G4 head prefix.
- **Strings:** f-strings, triple quotes, byte/raw prefixes: Verbatim text. Implicit concatenation stays as authored.
- **Decorators, `lambda`, comprehensions** are text/containers by normal rules; a comprehension's brackets follow the states.
- **Naming exceptions:** dunders, `self`/`cls`, test discovery (`test_*`), `__all__`, framework hooks, stdlib; PEP 8 linters are silenced (`# noqa: N802, N806`).

```
Result = LoadData(
                     Config.Get("Url"),
                     # Retry twice before giving up
                     RetryPolicy(Attempts=2)
                 )
```

`(` at column 17 ⇒ arguments at 21, `)` at 17.

## F. Rust

- Semicolons are explicit ⇒ openers detach freely. Every container may be Allman.
- **Compound tokens:** `#[ ]` and `#![ ]` are single boundary tokens. Macro invocations `Name!( )`, `Name![ ]`, `Name!{ }` have `!` as G4 head glue.
- **Angles:** generics, turbofish `::<T>`, trait objects `dyn`; lifetimes `'a` are not delimiters; `>>`/`>=` closers split; `A as usize < B` is read by rustc as generics, so follow the grammar.
- **Closures:** `|A, B|` is an identical-glyph container in the head; body `{` detaches below it.
- **match arms:** `Pattern =>` is the head of an arm; its `{ … }` follows normal states. Arms are items.
- **Struct literals and tuples** are containers; `where` clauses are continuation text.
- **Naming:** functions/variables PascalCase and types `Pascal_Snake_Case` need `#![allow(non_snake_case, non_camel_case_types, non_upper_case_globals)]`. Keep trait methods you implement from std/crates (`fmt`, `from`, `next`), derive-generated names, `#[serde(rename = "…")]` wire names, and `extern "C"` symbols.

```
let Handler = move |Request: Request_Context|
{
    // Reject early; the pool is already saturated
    if Pool.IsSaturated() { return Err(Pool_Error::Busy); }
    Process(Request)
};
```

## G. C and C++

- Free detach. `#` directives are line-bound (force expansion, quarantine unbalanced `#if` arms).
- **Templates:** `template<…>` and `Name<…>` angle containers; `>>` closer splitting (C++11); `operator<`/`operator<<` are tokens; digraphs `<: :> <% %>` are brackets (treat as such if they appear).
- **Initializer lists, lambdas** `[Captures](Params){ Body }` (three containers in sequence: capture list, params, body), attributes `[[ ]]` (compound token).
- **Macros:** multi-line `#define` Verbatim; macro arguments are token trees: format the outer parens only.
- **`extern "C" { }`, `namespace X { }`:** ordinary braces; body items are declarations.
- **K&R function definitions** with parameter declarations after the `)`: leave as authored.
- **Naming exceptions:** libc/STL names, `main`, ABI symbols, `operator` overloads.

## H. C# and Java

- Free detach. C#: attributes `[Name(Args)]` (bracket + paren), object/collection initializers `new T { A = 1 }`, switch expressions, records `record R(int A);`, string interpolation `$"…{X}…"`, raw strings `"""…"""`, verbatim `@"…"`, `#region`/`#if`. Java: annotations `@A(…)`, generics with wildcards `<? extends T>`, diamond `<>`, lambdas `->`, text blocks `"""` (Verbatim), `switch` arrow forms.
- **Angles** use the follow-token test (`F(G<A, B>(7))`). **Properties** `{ get; set; }` are Monolith if fit.
- **Naming:** C# already PascalCase for methods/properties; locals/params PascalCase per OLAFS. Java: keep bean accessors (`getX`/`setX`/`isX`), `toString`, `equals`, `hashCode`, annotation members, JUnit/Spring names; types Pascal_Snake_Case for your own.

## I. Dart (§Dart)

- Free detach (explicit semicolons).
- **Typed collection literals** `<String, int>{ … }`, `<Widget>[ … ]`: the `<…>` is a Monolith angle container in the **head**, so `{`/`[` detach below it. It breaks a Sole-Child Chain by default: `jsonEncode(<String, dynamic>{` expands `(` with the typed literal as its single item. An optional profile switch, `typed_literal_prefix = glue`, treats the type arguments as part of the literal's compound opener and restores `(<String, dynamic>{ … })`.
- **`>>>` is an operator:** closing `List<Map<String, Object>>>` splits into three closers.
- **Glue:** spread `...` / `...?` before a collection literal is slot-1 glue and travels with the compound (`...[{ … }]`); null-aware `?.` and `!` are head; cascades `..Add(X)` are continuation text starting at the base indent after a non-Monolith closer.
- **Named arguments** (`body:`, `children:`) are labels: significant content that breaks runs. The label stays on its line and the opener drops below at the same indent:

```
return Scaffold
(
    appBar: AppBar(title: Text('Articles')),
    body: ListView
    (
        padding: EdgeInsets.all(16),
        children:
        [
            ...Pinned.map((Item) => Article_Tile(Item)),
            // Everything else, newest first
            for (final Item in Recent) Article_Tile(Item),
        ],
    ),
);
```

- **Naming:** keep framework overrides (`build`, `createState`, `initState`), widget/package APIs, JSON keys; silence `non_constant_identifier_names` for your own PascalCase locals (`// ignore_for_file: non_constant_identifier_names`).

## J. Data and markup formats

**JSON / JSONC / JSON5.** Whitespace-insensitive ⇒ Free detach. Label then container: key line, opener below at the same indent. Trailing commas never in strict JSON. Small children are Monolith inside an expanded parent. Keys are a wire contract: never recased.

```
{
    "ConfigurationName": "StandardProfile",
    "ModuleRegistry": ["FormattingEngine", "ParserIntegration"],
    "AllocationProfile":
    {
        // Quotas are per tenant, not global
        "PrimaryMemory": "Dynamic",
        "StorageQuota": "StandardAllocation"
    }
}
```

**YAML.** Block style (indentation) is untouched. Flow collections: Monolith if fit, else Hybrid:

```
Servers: [
             "alpha.example.com",
             "beta.example.com"
         ]
```

(`[` at column 9 ⇒ items 13, `]` 9; every continuation line is deeper than the parent key at column 0.) Anchors, tags, block scalars `|` `>`: Verbatim.

**TOML.** Arrays: Monolith, else Hybrid (`Key = [` … `]` at the `[` column). Inline tables `{ a = 1 }`: Monolith only (TOML 1.0); if they don't fit, convert is *not* allowed (token change), so leave Verbatim and say so.

**XML / HTML / Vue / Svelte.** Elements are containers; attributes are items; text content is whitespace-sensitive (never reflow).

```
<Button
    Class="primary"
    OnClick="Submit"
>
    Save
</Button>
```

`<pre> <textarea> <script> <style>`: Verbatim or embedded language. Void and unclosed optional-end-tag elements are leaves.

**CSS / SCSS / Less.** `selector` head, `{` detaches, declarations are items separated by `;`. Monolith for small rules:

```
.card { margin: 0; padding: 4px; color: #00E5FF; }

.card:hover
{
    /* Lift the card slightly */
    transform: translateY(-2px);
    border-color: #00BFFF;
}
```

`@media (…)`, `calc( )`, `var( )`, `url( )` (Verbatim if unquoted data), SCSS maps `( key: value )`, `@include X( )` follow the normal states. Hex colors uppercase; no yellow. Class names, custom properties and selectors are contracts with the HTML.

**SQL.** Parentheses (subqueries, CTE bodies, `IN ( )`, `OVER ( )`), `CASE … END`, `BEGIN … END` (keyword pairs), `$$…$$` bodies (identical-glyph, Verbatim text if it is another language). Keywords uppercase as authored. Identifier quoting (`"x"`, backticks, `[x]` in T-SQL) is not a container. `<`/`>`/`<>`/`<=` are operators. Schema names are contracts: match the database.

```
SELECT UserAccountId, COUNT(OrderId) AS TotalOrders
FROM UserOrdersTable
WHERE
(
    OrderStatus = 'Completed' AND
    -- Orders before 2026 are archived elsewhere
    CreatedDate >= '2026-01-01'
)
GROUP BY UserAccountId;
```

## K. Keyword-pair and newline-terminated languages

*(All ⚠/◐: confirm against the language when you can; until then Hybrid.)*

- **Ruby.** `do |x| … end`, `def … end`, `{ |x| … }`. A newline before `do` or `(` ends the call: Hybrid, `end` at the keyword's column (the column-anchored rule), Monolith `{ |X| Use(X) }` when it fits. Constants start uppercase and locals can't (a capital makes a constant): the naming exception applies to locals.
- **Lua.** `then … end`, `do … end`, `function … end`. Newlines are whitespace but `F\n(X)` is a call-or-statement hazard ⇒ Hybrid for call parens; keyword openers `then`/`do` may sit alone (Free).
- **Shell.** Functions `Name() { …; }` (note `;` before `}` when Monolith, a space after `{`), compound commands `if … then … fi`, `while … do … done`, `case … esac`, subshells `( )`, groups `{ ; }`, `$( )`, `${ }`, `$(( ))`, `[[ ]]`. `then`/`do` may be placed on their own lines. Arguments can't be broken without `\` (an inserted token) ⇒ Monolith or Verbatim. Heredocs are line-pinned. Env var names stay `UPPER_SNAKE` (contract).
- **PowerShell.** Script blocks `{ }`, `@( )`, `@{ }`, `$( )`. Detached `{` after a pipeline element is a new statement ⇒ **Hybrid**. No backtick continuations.
- **Kotlin, Swift, Scala.** A newline before `(` typically ends the statement: Hybrid at depth 0; trailing lambdas and `{` after `)`: Scala treats a single newline before `{` as a continuation, Kotlin/Swift generally want it on the head line ⇒ Hybrid unless certain. Generics `<T>` via the grammar. Monolith 1 statement.
- **HCL/Terraform.** `resource "t" "n" {` keeps `{` on the header line (Hybrid); attributes are newline-terminated.
- **Haskell, F#, Elm, Idris, Nim.** Offside rule: every continuation line must sit deeper than the construct's layout column. A closer at the base indent (Allman) can close the block ⇒ Hybrid with the closer at the opener's (deeper) column. `let … in`, `where`, `do` blocks are layout blocks, not containers. In Haskell/Elm, initial capitals mean constructors/types: **do not capitalize values or functions**: use the exception protocol (spec §9.2).
- **Lisp family.** Every `( )` is a container; Free detach; Monolith First; Expanded puts a closer alone, which goes against community habit but follows the style. If the user's codebase is Lisp, mention the trade-off once.
- **OCaml / ReasonML / Elixir / Erlang / Julia / R / MATLAB / Fortran / Pascal / Ada / VB / COBOL:** apply procedure B; keyword pairs per edge-cases §H; naming exceptions for case-significant languages.

---
name: olafs-formatting
description: "OLAFS (Omniversal Language-Agnostic Formatting Style, also called Improvised Allman Style / IAS): the user's mandatory formatting philosophy for ALL code, markup, queries, configuration and structured data in ANY language. Use this skill whenever you write, edit, review, reformat, port or explain code or data, even for a three-line snippet, and whenever the user mentions OLAFS, IAS, Allman, K&R, Monolith, one-liner, Container Grouping, Simplified or Expanded Symmetry, Hybrid exception, container formatting, brace/bracket/paren placement, PascalCase or Pascal_Snake_Case naming, hex color codes, the yellow ban, or building/reviewing a formatter for this style. It decides, for every container ( ) { } [ ] < > tags template literals and keyword pairs, whether it is a one-line Monolith, a Grouped compound like [{( )}], a fully Expanded hierarchy, a parser-forced Hybrid, or Verbatim. Read it before producing any code, not after."
---
# OLAFS: Omniversal Language-Agnostic Formatting Style

## Why this style exists

Code is read as geometry. The eye follows vertical columns and bounded rectangles, so every container (anything with an opener and a closer) should be either **one sealed line** or **a rectangle whose top and bottom edges sit in the same column**. The styles this replaces fail in two ways. K&R hangs the opener on the end of a line (a "floating anchor" whose column depends on the text before it) and piles closers into clumps like `}});`. Dogmatic Allman wastes vertical space. OLAFS keeps Allman's symmetry and wins the space back by flattening anything that fits (Monolith) and fusing nested wrappers into one compound (Container Grouping).

Whitespace is cognitive fuel: line breaks mark conceptual boundaries, indentation shows nesting, and a matched closer gives the brain its "closure". Never cram; never waste.

## How to use this skill

1. Read this file fully. It is enough for ordinary code in a mainstream language.
2. Read **`references/specification.md`** when you need the exact rule, a precise definition, the config defaults, or the resolved-conflict register (§11) to settle an argument about what the style "really says".
3. Read **`references/edge-cases.md`** for `<`/`>` ambiguity, unpaired characters, comments, strings and templates, preprocessors, newline-sensitive parsers, markup, embedded languages, broken input, Unicode.
4. Read **`references/language-guide.md`** before formatting a specific language, especially Go, Python, JS/TS, YAML, TOML, Haskell-like, shell, HTML/JSX, SQL, or anything newline-terminated.
5. Read **`references/hybrid-form.md`** before emitting any Hybrid layout, and whenever a language forces an inline opener and the head is long. It holds the short-head rule, the Void limits and the Head Reduction Ladder.

## Precedence (when rules collide, the higher one wins)

1. **Parser sovereignty.** Output must parse, and mean exactly what the input meant. Whitespace only; tokens are never added or removed (the single exception is a profile-declared repair, e.g. Go's trailing comma).
2. **Foreign contracts.** Wire formats, external API names, DB column names, FFI symbols, framework-discovered names, generated files, lockfiles and tool-owned files keep their existing form. Why: changing them breaks something outside the file you can't see.
3. **The user's explicit instruction in this conversation.**
4. **This skill.**
5. **Community convention**, only to fill gaps this skill is silent on (import order, license headers, Python's indentation blocks).

When editing someone else's file, apply OLAFS to the code you write or change and leave untouched lines alone, unless asked to reformat the whole file. If CI enforces a different formatter, say so once and ask; default to OLAFS.

## The five states


| State           | Other names                             | What it looks like                                                                 | When                                                                  |
| ----------------- | ----------------------------------------- | ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| **1. Monolith** | One-Liner                               | whole container on one line                                                        | **Always tried first**                                                |
| **2. Grouped**  | Simplified Symmetry, Container Grouping | fused compound opener`[{(` / mirrored closer `)}]` on dedicated lines, same column | Monolith failed and the container is a link in a Sole-Child Chain     |
| **3. Expanded** | Expanded Symmetry, Hierarchical         | opener alone, items one per line, closer alone, same column                        | Monolith failed, no chain                                             |
| **4. Hybrid**   | Column-Anchored Hybrid (Allman + K&R)   | opener stays on the head line; content and closer anchor to the**opener's column** | Only where the parser/compiler forbids a detached opener              |
| **5. Verbatim** |                                         | bytes untouched                                                                    | strings, template text, heredocs, generated code, quarantined regions |

Priority: **Monolith ≻ Grouped ≻ Expanded**. Hybrid substitutes for Grouped/Expanded when the language forces it. No other layout exists: **no half-states**. A container is sealed on one line, or detached with symmetric columns, or Hybrid with symmetric columns. "Commit or quit."

## Decision procedure (run it for every container, outermost first)

1. **Write it flat** and measure the *whole line it would occupy*: indentation + head text + flat container + trailing glue (`;` `,` `?`) + whatever must stay on that line up to the next legal break. Trailing comments don't count.
2. **Monolith** if ALL hold: it fits the Canvas (default **160** display columns); it has no *forced break* (a line comment inside, a token containing a newline, a line-bound directive); it is inside the *complexity budget* (≤ 4 statements per block, ≤ 2 nested blocks, ≤ 12 containers; newline-terminated languages: 1 statement). Empty containers are always Monolith. **A manual line break or blank line in the source is NOT a reason to expand**: layout depends on tokens and Canvas only.
3. Otherwise, if this container's whole content is exactly one child container (a **Sole-Child Chain** of length ≥ 2), emit **Grouped**: compound opener on its own line(s) below the head, innermost items at +4, mirrored compound closer at the same column, suffix glue after it.
4. Otherwise, if the language or context forbids a detached opener (see the table in `language-guide.md`; when you can't tell, assume it does) **and the head is short** (content indent ≤ 40), emit **Hybrid**. With a long head, reduce the head first (`hybrid-form.md`).
5. Otherwise emit **Expanded**.
6. Recurse into each child with its own line context. An Expanded parent routinely holds Monolith children.

## The laws (checkable; verify them on your output)


| Law Name        | Law                                                                                                                                                     |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| L1 Anti-Hug     | Outside a Monolith, no opener ends a line that has head text before it. Only Hybrid may.                                                                |
| L2 Parity       | For every non-Monolith container or run:`column(opener) == column(closer)`. Sole exception: the last-resort concession layout in `hybrid-form.md` (H6). |
| L3 Return       | Every closer (or fused closer run) starts its own line. Suffix glue and a trailing comment may follow it.                                               |
| L4 Indent       | Content sits one level (4 spaces) deeper than the anchor (the opener's column for Hybrid).                                                              |
| L5 No-Partial   | Monolith ⇒ no newline inside. Non-Monolith ⇒ opener and closer on different lines.                                                                    |
| L6 Mirror       | A fused run's closer is the exact reverse of its opener, adjacent characters, one column.                                                               |
| L7 Preservation | Tokens, comments (text and order) and verbatim bytes are unchanged.                                                                                     |
| L8 Idempotence  | Formatting the output again changes nothing.                                                                                                            |

## Sole-Child Chain (Container Grouping) in thirty seconds

A container may fuse with its child **only if its entire content, ignoring whitespace and whitelisted glue, is exactly that one child**. Texts, labels (`"Courses":`, `children:`), comments or a second item between them break the run.

```
InitializeCoreEngine
[{(
    // Order matters: security first
    "SecurityModule",
    "DatabaseDriver",
    "RoutingInterface"
)}]
```

- A parent with two or more items is **not** a chain link. It expands, each item on its own line, and each item decides for itself (Monolith, Grouped, Expanded). Commas there are item separators on each item's closer line (`},`) and never join a closing run.
- Glue between closers (a trailing comma on a **sole** child: `},])`) is allowed; with ≥ 2 items it never is.
- Grouping is **local**: a separated inner container does not force its ancestors apart, and the outer closers fuse again afterwards. Grouping is a property of a boundary run, not of a container.
- Never fuse across different detach policies, and by default never fuse angle brackets `< >`.
- Monolith is checked first: if the whole chain fits and has no forced break, write `Name[{("A", "B")}]` on one line.

## Hybrid in thirty seconds

Use when a detached opener would change or break the program (Go: every opener; JS: after `return` `throw` `yield` `break` `continue` `async`, before `=>`, around postfix `++`/`--`; Python at statement depth 0; layout-sensitive languages; TOML arrays; PowerShell script blocks). The opener stays on the head line after one space; content goes at **opener column + 4**; the closer drops to its own line at **the opener's exact column**.

```
function RenderView()
{
    return (
               <UserDashboardLayout>
                   <NavigationHeader ThemeTone="#00E5FF" />
               </UserDashboardLayout>
           );
}
```

`(` sits at column 11 (4 + `return `); content is at 15; `);` is at 11. Use spaces for the offset beyond the base indent even in tab-indented files, otherwise parity is impossible.

**Hybrid is a concession, not a style, and it is for short heads only** (`return (`, `Key = [`, `func Run() {`). It is legal only when a detached opener would be a syntax error or change meaning; "idiomatic" and "the formatter does it" are not reasons. A long head pushes the body far right and leaves a **Void** of empty space: keep the content indent at 40 columns or less (25% of the Canvas), never past 64. With a long head, reduce it first (name an intermediate, alias a long name, bundle parameters); only as a last resort, with existing tokens fixed, use the concession layout (body at base indent + 4, closer at base indent) and say why. Unsure whether a detached opener is legal? Use Hybrid if the head is short; otherwise reduce the head or ask. Details in `references/hybrid-form.md`.

## Items, gaps, comments, continuations (cheat sheet)

- **Items.** Expanded: one item per line, separator stays on the item's last token. The old `'Change', function()` two-in-one-line form is superseded. A block's items are its statements.
- **Gaps and padding (Monolith).** `{ }` gets one space inside; `( )` `[ ]` `< >` get none. Between head and opener: keep the author's 0 or 1 space; when collapsing from a newline use 1 space after `if/for/while/switch/catch`-type keywords and before `{`, else 0. Opener directly after another opener: no gap.
- **Trailing separators.** Preserve what the author wrote; Go requires one when a literal/call expands; JSON forbids them.
- **Blank lines.** Between items of an expanded container keep at most one. Blank lines vanish when a container becomes a Monolith.
- **Comments.** A line comment inside a container forbids Monolith for it (and its ancestors). Trailing comments stay attached to the token they followed, after the closer's glue. A comment between head and opener stays on the head line and the opener drops below it. Never move a comment across an item.
- **Continuation after a closer.** After a *non-Monolith* closer, any non-glue text (`.Method(...)`, `else`, `catch`, `where`) starts a new line at the base indent. After a Monolith closer it stays inline.
- **Clause statements.** For `if/else`, `try/catch/finally`, `do/while`: either the whole statement is one Monolith line, or **every clause starts its own line** at the base indent (each clause body may still be Monolith). Never `if (A) { X(); } else` followed by a detached `{`.
- **Seam** `)(` / `)[`: the second opener starts a new line, empty head.

```
if (IsReady) { Start(); } else { Stop(); }          // whole statement fits: one line

if (IsReady) { Start(); }
else
{
    // Fall back to the safe mode first
    Stop();
    Log(Reason);
}
```

## Naming (lexical layer)


| Entity                                                                             | Convention                                                     | Examples                                                                                                   |
| ------------------------------------------------------------------------------------ | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Functions, methods, variables, parameters, modules, namespaces, fields, properties | `PascalCase`                                                   | `ExecuteTransaction`, `UserAccountConfiguration`                                                           |
| Structs, enums, classes, interfaces/traits/protocols, type aliases, records        | `Pascal_Snake_Case`                                            | `Database_Connection_Pool`, `Network_Status_Code`                                                          |
| Constants and statics (compile-time/immutable)                                     | `SCREAMING_SNAKE_CASE`                                         | `MAX_RETRY_THRESHOLD`                                                                                      |
| Acronyms                                                                           | stay fully capitalized inside the convention                   | `GetHTTPResponse`, `TargetURL`, `URLGrabber`, `XMLHTTPRequest`, `HTTP_Request_Payload`, `FETCH_TIMEOUT_MS` |
| Language keywords/literals                                                         | untouched lowercase (or the language's own case: Python`True`) | `if`, `return`                                                                                             |

Why: Every user-defined token starts with a capital, so keywords (lowercase) and your own names are visually separated; the three shapes tell you at a glance whether a name is logic, a blueprint, or a fixed constant.

**Exception protocol.** If capitalizing would change behavior or break something outside the file, keep the original form. Rule of thumb: *if something other than your own code finds, binds, serializes or reflects the name, don't rename it.* Typical cases: external/stdlib/framework APIs; wire-format keys (JSON, protobuf, headers); DB columns; FFI/exports; entry points (`main`, `__init__`, `setUp`); framework lifecycle overrides (`build`, `onCreate`, `toString`); test discovery (`test_*`, `TestXxx`); JavaBean accessors; React hooks (`useX`); Go unexported package-level names (capitalizing exports them); Ruby/Elixir/Haskell/Prolog cases where initial capitals change meaning (constant, module, type, variable). Linters that only warn (Rust `non_snake_case`, Dart `non_constant_identifier_names`, Python N-series) are silenced with the language's allow/ignore directive, never obeyed at the cost of this convention. Details: `specification.md` §9.

## Color policy

- **Yellow is banned** in code, themes, UI, charts, icons, badges, alerts, diagrams and your own prose examples: Yellow, Gold, Amber, Chartreuse, Mustard, Olive Yellow, Light Yellow, any hue 45° to 65° in HSL, and these hex values: `#FFFF00`, `#FFD700`, `#BB7B00`, `#FEF8E7`, `#FFF000`. Avoid CSS names `yellow`, `gold`, `khaki`, `lightyellow`, `lemonchiffon`, `palegoldenrod`, `goldenrod`.
- **Use instead.** Warnings: Orange `#FC6A03`. Errors: Bright Red `#FF1744`. Highlights: Cyan `#00E5FF`. Accents and ratings: Azure `#00BFFF`, Emerald `#00E676`, Violet `#C77DFF`, Indigo `#3D5AFE`.
- **Hex codes are always uppercase**: `#00E5FF`, never `#00e5ff`.
- If the user explicitly asks for yellow in a request, say it contradicts their style, offer the nearest approved color, and use yellow only if they confirm. Don't alter yellow in code you weren't asked to touch; mention it.

## Quick gallery (✔ right, ❌ wrong)

```
// ✔ Monolith: fits, no forced break
LoadData(Config.Get('Url'), (Data) => { if (Data) { Parse(Data); Save(Data); } }, True);

// ✔ Expanded: a line comment forces it; children stay Monolith where they can
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

// ❌ K&R: floating anchors, tail clump
LoadData(Config.Get('Url'), (Data) => {
    // Persist only non-empty payloads
    if (Data) { Parse(Data); Save(Data); }
}, True);

// ❌ half-expanded (hybrid by accident)
LoadData(
    Config.Get('Url'),
    True
);
// ✔ it fits, so it is simply:
LoadData(Config.Get('Url'), True);
```

```
// ✔ chain: only the last call expands; text after an expanded closer starts a new line
Items.Where((Item) => Item.IsActive).Select
(
    (Item) =>
    {
        // Normalize before projecting
        return Item.Id;
    }
)
.ToList();

// ✔ JSON: Monolith children inside an expanded root (root is wider than the Canvas)
{
    "ConfigurationName": "StandardProfile",
    "EvaluationStatus": "Active",
    "IsCompliant": true,
    "ModuleRegistry": ["FormattingEngine", "ParserIntegration"],
    "AllocationProfile": { "PrimaryMemory": "Dynamic", "StorageQuota": "StandardAllocation" }
}

// ✔ label + container: label stays on its line, opener drops below at the same indent
"Courses":
[
    // Core curriculum only
    "Formatting",
    "Programming"
],
```

## Pre-flight checklist (run before you send code)

1. Did I try Monolith first for every container, ignoring how I happened to line-break while thinking?
2. Does any non-Monolith line end with an opener after head text? (Only Hybrid may.)
3. Does every opener share a column with its closer (compound runs: the first character)?
4. Any `});` `}}]` `]);` clumps of different-column closers? (A fused run `)}]` + suffix glue `;` is fine.)
5. Did I fuse only true Sole-Child Chains (no labels, texts, comments, siblings between)?
6. Is every Hybrid forced by a real parser reason, is its head short (content indent ≤ 40), and is its closer at the opener's column?
7. Did I leave string, template, heredoc, regex and comment text untouched?
8. Tokens unchanged (apart from declared repairs)? Comments preserved in order?
9. Names: PascalCase / Pascal_Snake_Case / SCREAMING_SNAKE, acronyms capitalized, foreign names left alone?
10. Any yellow? Any lowercase hex digits?
11. Would formatting this output again change it?

## Configuration defaults (all overridable by the user)


| Key                                                              | Default                                                                                |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| `canvas_width`                                                   | 160 display columns                                                                    |
| `indent_width` / `indent_style`                                  | 4 / spaces (smart tabs when tabs are required)                                         |
| `monolith.max_statements` / `max_block_depth` / `max_containers` | 4 / 2 / 12 (newline-terminated languages: 1 statement)                                 |
| `padding`                                                        | brace = 1 space; paren, bracket, angle = 0                                             |
| `max_blank_lines`                                                | 1                                                                                      |
| `grouping.scope` / `fusion`                                      | any chain / sole-child                                                                 |
| `angle.enabled` / `angle.allow_grouping`                         | true / false                                                                           |
| `hybrid.soft_indent_limit` / `hard_indent_limit`                 | 40 / 64 columns (25% / 40% of Canvas); beyond the hard limit use the concession layout |
| `continuation`                                                   | newline (join where the language demands)                                              |
| `embedded.<lang>.reindent`                                       | false                                                                                  |

## Reference map

- `references/specification.md`: normative rules, definitions, geometry, config, **resolved-conflict register**.
- `references/edge-cases.md`: `<`/`>`, unpaired characters, comments, strings/templates/heredocs, preprocessors, markup, embedded languages, broken or partial code, Unicode/tabs, huge data, token repairs, generated files.
- `references/language-guide.md`: detach-safety matrix and per-language profiles with examples; procedure for unknown languages.
- `references/hybrid-form.md`: why Hybrid exists, the short-head rule, the Void limits, the Head Reduction Ladder, the concession layout, per-language handling, and how to talk about it.

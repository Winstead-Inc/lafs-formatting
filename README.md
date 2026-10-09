# LAFS: Language-Agnostic Formatting Style

> **A cognitive-first, geometry-based code formatting standard and universal AI agent skill.**

[![Standard: LAFS](https://img.shields.io/badge/Standard-LAFS-00E5FF.svg?style=flat-square)](#)
[![Format: Language--Agnostic](https://img.shields.io/badge/Language-Agnostic-3D5AFE.svg?style=flat-square)](#)
[![Agent Skill: Ready](https://img.shields.io/badge/Agent_Skill-Antigravity%20%7C%20Claude%20%7C%20Copilot-00E676.svg?style=flat-square)](#)
[![Geometry: Symmetrical](https://img.shields.io/badge/Geometry-Translational_Symmetry-C77DFF.svg?style=flat-square)](#)

---

## Table of Contents

- [Overview](#overview)
- [The Cognitive & Geometric Philosophy](#the-cognitive--geometric-philosophy)
- [Resolving the Linus Paradox](#resolving-the-linus-paradox)
- [The Five Structural States](#the-five-structural-states)
- [Visual Comparison](#visual-comparison)
- [The Eight Invariant Laws](#the-eight-invariant-laws)
- [Lexical Conventions (Casing Hierarchy)](#lexical-conventions-casing-hierarchy)
- [Color Policy & Design Aesthetics](#color-policy--design-aesthetics)
- [Pre-Flight Conformance Checklist](#pre-flight-conformance-checklist)
- [Installation & Agent Integration](#installation--agent-integration)
- [Repository Structure & Deep References](#repository-structure--deep-references)
- [Configuration Defaults](#configuration-defaults)
- [Contributing & License](#contributing--license)

---

## Overview

The **Language-Agnostic Formatting Style (LAFS)** (also known as *Language-Agnostic Formatter* or *Improvised Allman Style / IAS*) is a structural formatting standard designed for modern wide viewports and human-AI pair programming.

Unlike conventional formatting conventions that treat line breaks and brace placement as arbitrary matters of personal taste, LAFS treats source code as a **discrete two-dimensional coordinate geometry**. Every delimiter—parentheses `()`, brackets `[]`, braces `{}`, angle brackets `<>`, and keyword pairs—acts as a load-bearing column bounding a logical enclosure.

LAFS enforces an uncompromising **binary state model**: code containers are either **fully horizontal (Monolith)** or **fully vertical (Translational Symmetry)**. Partial, clinging, or accidental hybrid layouts are strictly prohibited.

---

## The Cognitive & Geometric Philosophy

### 1. Vector Alignment vs. Diagonal Drift

In any text editor, characters occupy a discrete 2D grid defined by column ($X$) and line ($Y$) coordinates. Reading code relies on the ocular tracking of structural boundaries.

* **K&R Asymmetry (Floating Anchors):** When an opening brace clings to the tail of a statement (`if (Condition) {`), its horizontal coordinate is volatile—it depends entirely on statement length. The inner block indents to $X + 4$, while the closing delimiter drops to $X = 0$. The eye is forced to trace an irregular, jagged diagonal vector. Scope exits collapse into unreadable tail clumps like `}});`.
* **LAFS Translational Symmetry:** Delimiters share the identical horizontal coordinate ($X$), anchoring the scope in a predictable, load-bearing column:

  $$
  \text{Open Container:} \quad (X, Y)
  $$

  $$
  \text{Enclosed Block:} \quad (X + 4, Y + 1 \dots Y + N)
  $$

  $$
  \text{Close Container:} \quad (X, Y + N + 1)
  $$

### 2. Gestalt Law of Closure

Human perception instinctively groups elements that form clear, closed boundaries. Decoupling structural anchors from statement logic gives every block a dedicated visual threshold and baseline, allowing the engineer or AI model to instantly comprehend nesting depth without prematurely parsing internal tokens.

---

## Resolving the Linus Paradox

Linus Torvalds famously criticized traditional Allman formatting for wasting valuable vertical screen real estate, forcing engineers to scroll through excessive whitespace voids. The traditional C/K&R response—gluing braces to statement tails—recovers vertical density at the expense of geometric structural clarity.

LAFS resolves this paradox completely:

1. **Monolith First:** Complete expressions, simple control blocks, flat parameter lists, and atomic data structures stay on a single line if they fit the canvas.
2. **Container Grouping (Sole-Child Chains):** Consecutive nested delimiters are collapsed into compound opening and closing nodes (`[{( ... )}]`), eliminating vertical cascading voids while maintaining a strict load-bearing column.
3. **Zero Floating Anchors:** Openers never cling to statement tails across line boundaries.

---

## The Five Structural States

Every container in LAFS exists in exactly one of five states. Hybrid states are forbidden (unless enforced by compiler grammar):

```
Priority: Monolith ≻ Grouped ≻ Expanded (Hybrid only where compiler mandates)
```

| State | Name | Visual Signature | Trigger Condition |
| :--- | :--- | :--- | :--- |
| **State 1** | **Monolith** | `LoadData(URL, True);` | Container fits within canvas (≤ 160 cols), budget ≤ 4 statements, ≤ 2 nested blocks, no internal line comments. |
| **State 2** | **Grouped** *(Simplified Symmetry)* | `[{( ... )}]` | Container exceeds canvas/budget and forms a Sole-Child Chain (content is exclusively one child container). |
| **State 3** | **Expanded** *(Hierarchical)* | Opener alone, items line-by-line, closer alone | Multi-item lists, branching control flow, or mixed statements that exceed single-line bounds. |
| **State 4** | **Hybrid** *(Column-Anchored)* | `return (`<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`<Content />`<br>&nbsp;&nbsp;&nbsp;&nbsp;`);` | **Concession only**: parser forbids detached opener (e.g., Go compiler, JS ASI after `return`, Python depth 0). Content and closer anchor to the opener's column. |
| **State 5** | **Verbatim** | Exact source bytes preserved | Multiline string literals, template text, heredocs, regex, and quarantined generated files. |

---

## Visual Comparison

### 1. Complex Method Invocations

#### ❌ Kernighan & Ritchie (K&R)

```typescript
// Floating opener anchor, ragged diagonal vector, tail clump
LoadData(Config.Get("Url"), (Data) => {
    if (Data) {
        Parse(Data);
        Save(Data);
    }
}, true);
```

#### ❌ Accidental Half-State

```typescript
// Clinging opener on line 1, closer detached on line 4 (forbidden)
LoadData(
    Config.Get("Url"),
    true
);
```

#### ✔️ LAFS: Monolith (Fits Viewport)

```typescript
// Symmetrical, single-line density
LoadData(Config.Get("Url"), (Data) => { if (Data) { Parse(Data); Save(Data); } }, true);
```

#### ✔️ LAFS: Symmetrical Expansion (Multiline) and Monolith Alternative

```typescript
// Symmetrical load-bearing columns; internal logic remains clean
LoadData
(
    Config.Get("Url"),
    (Data) =>
    {
        // Persist only non-empty payloads
        if (Data) { Parse(Data); Save(Data); }
    },
    true
);

// Inner Monolith for saving vertical space
LoadData
(
    Config.Get("Url"),
    (Data) => { if (Data) { Parse(Data); Save(Data); } },
    true
);
```

---

### 2. Deeply Nested Containers (Container Grouping)

#### ❌ Dogmatic Allman (Vertical Dispersion Void)

```typescript
InitializeCoreEngine
(
    [
        "SecurityModule",
        "DatabaseDriver",
        "RoutingInterface"
    ]
);
```

#### ✔️ LAFS: Container Grouping (Compound Node)

```typescript
InitializeCoreEngine
([
    "SecurityModule",
    "DatabaseDriver",
    "RoutingInterface"
]);
```

---

## The Eight Invariant Laws

Every block of formatted code can be verified against eight invariant laws:

* **L1 (Anti-Hug):** Outside a Monolith, no opener ends a line that has head text before it. Only Hybrid may.
* **L2 (Parity):** For every non-Monolith container: `column(opener) == column(closer)`.
* **L3 (Return):** Every closer (or fused closer run) starts its own line. Suffix punctuation (`;`, `,`) may follow.
* **L4 (Indent):** Content sits exactly one indentation level (+4 spaces) deeper than the opener anchor.
* **L5 (No-Partial):** Monolith implies no newline inside. Non-Monolith implies opener and closer reside on separate lines.
* **L6 (Mirror):** A fused closing run is the exact reverse sequence of its opening run (`[{( ... )}]`), adjacent, at one column.
* **L7 (Preservation):** Tokens, comments, and verbatim text are never altered, discarded, or reordered.
* **L8 (Idempotence):** Running the formatter a second time produces an identical token stream and coordinate layout.

---

## Lexical Conventions (Casing Hierarchy)

LAFS establishes a strict three-tier typographic pacing standard:

| Scope | Casing Standard | Examples |
| :--- | :--- | :--- |
| **Logic & Execution** | `PascalCase` | `ExecuteTransaction`, `UserAccountConfiguration`, `FetchData` |
| **Structural Blueprints** | `Pascal_Snake_Case` | `Database_Connection_Pool`, `Network_Status_Code`, `User_Model` |
| **Immutable Invariants** | `SCREAMING_SNAKE_CASE` | `MAX_RETRY_THRESHOLD`, `GLOBAL_CIPHER_KEY`, `DEFAULT_TIMEOUT` |
| **Established Acronyms** | Preserved Uppercase | `GetHTTPResponse`, `TargetURL`, `URLGrabber`, `HTTP_Request_Payload` |
| **Language Keywords** | Native Lowercase | `if`, `while`, `return`, `struct`, `impl`, `class` |

> **Exception Protocol:** Never rename foreign interfaces, external JSON wire keys, database schema column names, FFI symbols, or language-mandated entry points (`main`, `__init__`, `useX`).

---

## Color Policy & Design Aesthetics

LAFS enforces strict chromatic and accessibility guidelines across code, documentation, UI components, and charts:

* 🚫 **The Yellow Ban:** Colors in the 45°–65° HSL hue range are strictly prohibited (e.g., `#FFFF00`, `#FFD700`, `#BB7B00`, `#FFF000`).
* 🎨 **Approved Palette:**
  * **Warnings:** Vibrant Orange (`#FC6A03`)
  * **Errors / Destructive:** Bright Red (`#FF1744`)
  * **Highlights / Brand:** Electric Cyan (`#00E5FF`)
  * **Information / Logic:** Azure Blue (`#00BFFF`)
  * **Success / Safe:** Emerald Green (`#00E676`)
  * **Special / Accents:** Violet (`#C77DFF`) or Indigo (`#3D5AFE`)
* 🔤 **Uppercase Hex Codes:** Always write hex values in uppercase (`#00E5FF`, never `#00e5ff`).

---

## Pre-Flight Conformance Checklist

Before committing or outputting code, verify conformance:

1. [ ] **Monolith First:** Was Monolith evaluated first for every container?
2. [ ] **Anti-Hug:** Does any non-Monolith line end with an opener attached to text?
3. [ ] **Coordinate Parity:** Does every opener match the exact column coordinate of its closer?
4. [ ] **No Clumping:** Are closers free of stepped clumps like `}});`?
5. [ ] **True Chains:** Did Container Grouping fuse only true Sole-Child Chains?
6. [ ] **Legitimate Hybrid:** Is every Hybrid forced by an actual parser requirement, with a short head (indent ≤ 40)?
7. [ ] **Verbatim Text:** Are strings, comments, and regex literals strictly preserved?
8. [ ] **Casing:** Are logic entities `PascalCase`, types `Pascal_Snake_Case`, and constants `SCREAMING_SNAKE_CASE`?
9. [ ] **Zero Yellow:** Is all yellow excluded in favor of approved chromatic alternatives?
10. [ ] **Idempotence:** Would running the formatter again produce zero diff?

---

## Installation & Agent Integration

### Using as an AI Agent Skill

This repository is organized as an **Agent Skill** compatible with Google Antigravity, Claude Code, GitHub Copilot, and Cursor agent environments.

#### Project-Level Installation

Clone or copy this repository into your project's `.agents/skills/` directory:

```bash
mkdir -p .agents/skills
git clone https://github.com/Winstead-Inc/olafs-formatting.git .agents/skills/lafs-formatting
```

#### Global Installation (User-Wide)

To make LAFS available across all workspaces on your machine:

```bash
# For Antigravity / Gemini agents:
git clone https://github.com/Winstead-Inc/olafs-formatting.git ~/.gemini/config/skills/lafs-formatting

# For Claude Code agents:
git clone https://github.com/Winstead-Inc/olafs-formatting.git ~/.claude/skills/lafs-formatting
```

When installed, AI pair programmers automatically inspect [`SKILL.md`](./SKILL.md) and apply LAFS principles to all generated and reformatted code.

---

## Repository Structure & Deep References

```text
lafs-formatting/
├── README.md                 # Public documentation, quickstart & visual guide
├── SKILL.md                  # Main AI agent skill definition with decision procedure
└── references/               # Normative specifications & deep dive manuals
    ├── specification.md      # Comprehensive normative specification & conflict register
    ├── language-guide.md     # Matrix of language-specific detach policies (Go, JS, Python, Rust...)
    ├── hybrid-form.md        # Technical specification for Column-Anchored Hybrid layout
    └── edge-cases.md         # Handling generics, preprocessors, heredocs, and broken input
```

* **[`SKILL.md`](./SKILL.md)**: The core entry point for autonomous coding agents.
* **[`references/specification.md`](./references/specification.md)**: Formal mathematical definitions, coordinate geometry, and the resolved-conflict register.
* **[`references/language-guide.md`](./references/language-guide.md)**: Language-specific detach safety matrix and parser profiles.
* **[`references/hybrid-form.md`](./references/hybrid-form.md)**: The Head Reduction Ladder, Void limits, and short-head rules.
* **[`references/edge-cases.md`](./references/edge-cases.md)**: Angle bracket ambiguity, macro expansions, and template strings.

---

## Configuration Defaults

All LAFS parameters are configurable via project-level configuration:

| Configuration Key | Default Value | Description |
| :--- | :--- | :--- |
| `canvas_width` | `160` | Maximum column width before mandatory container expansion. |
| `indent_width` | `4` | Indentation width in spaces (smart tabs supported when enforced). |
| `monolith.max_statements` | `4` | Maximum statements allowed inside a single Monolith container. |
| `monolith.max_block_depth` | `2` | Maximum nesting depth permitted within a Monolith container. |
| `monolith.max_containers` | `12` | Maximum atomic containers permitted on a single Monolith line. |
| `padding.brace` | `1 space` | Padding inside inline `{ ... }` blocks. |
| `padding.paren_bracket` | `0 spaces` | Padding inside inline `(...)` and `[...]` containers. |
| `hybrid.soft_indent_limit` | `40 cols` | Threshold triggering head reduction before hybrid expansion. |
| `hybrid.hard_indent_limit` | `64 cols` | Hard ceiling before invoking fallback concession layout. |

---

## Contributing & License

Contributions, edge-case evaluations, and language profile extensions are welcome! Please open an issue or pull request in the [GitHub Repository](https://github.com/Winstead-Inc/olafs-formatting).

Distributed under the [MIT License](LICENSE).

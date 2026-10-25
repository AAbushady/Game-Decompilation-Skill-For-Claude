# Game Decompilation Skill For Claude

A set of [Claude Code](https://claude.com/claude-code) skills for building a **byte-matching decompilation** of a
game: C/C++ source that, built with the original compiler and flags, produces the retail binary byte for byte, and
that still reads like code the original team wrote.

The skills are project-independent. They describe the method (how to reconstruct, diagnose, permute, review and
merge), and each game repository supplies its own commands and conventions through an `AGENTS.md` file.

## Principles

- **The byte gate decides.** A function is done only when the whole binary, linked with that function built from
  source, hashes to the retail binary. Fuzzy scores, local diffs and an agent's opinion are only filters in front
  of that gate, so a wrong answer can't land.
- **Identical bytes are not enough.** A match also has to pass the reader test: *would a programmer on the
  original team have typed this?* Code that matches through tricks (dummy temporaries, `volatile` carriers, empty
  blocks, offset casts, reused variables) is a **fakematch**. It is tracked as `MATCH-PENDING` and treated as a
  clue to the real source, not as progress.
- **Two numbers, always.** Campaigns report *matched under the rules* first and *bytes-matched* second.
- **Diagnose before fighting.** A plateau is classified by which compiler decision diverged before any effort is
  spent on it, and the permuter is reserved for proven register/schedule residue.

## The skills

| skill | use it to |
|---|---|
| [`matcher`](.claude/skills/matcher/SKILL.md) | reconstruct and byte-match functions, units, classes, globals and referents; recover lost matches; finish a match to the code rules |
| [`wall-identifier`](.claude/skills/wall-identifier/SKILL.md) | classify a stuck function as an inline/call-set, control-flow, register/schedule or masked/referent wall, and route it to the right lever |
| [`helper-recovery`](.claude/skills/helper-recovery/SKILL.md) | find inline helpers, accessors and macros the compiler expanded, restore them, and apply them at every site |
| [`permute`](.claude/skills/permute/SKILL.md) | run bounded multi-island permuter searches, keep several distinct states, and turn what they reveal into honest source changes |
| [`decomp-code-rules`](.claude/skills/decomp-code-rules/SKILL.md) | write, review or lint reconstructed source; judge fakematches against genuine period idioms; choose a naming and layout style |
| [`orchestrator`](.claude/skills/orchestrator/SKILL.md) | run a campaign of parallel matcher workers in separate git worktrees, review their results, and merge them through a fixed procedure |

Supporting references:

- [`matcher/references/AGENTS-template.md`](.claude/skills/matcher/references/AGENTS-template.md): the
  project-guide template, including the project adapter, scores, workflow and byte gate.
- [`matcher/references/levers.md`](.claude/skills/matcher/references/levers.md): source-shape levers for
  steering codegen honestly.
- [`decomp-code-rules/references/rules.md`](.claude/skills/decomp-code-rules/references/rules.md): the full rule
  catalogue (R1 and up) with examples, tiers and the fakematch table.
- [`decomp-code-rules/references/style-defaults.md`](.claude/skills/decomp-code-rules/references/style-defaults.md):
  default C and C++ naming and layout tables, taken from Quake III Arena and DOOM 3 BFG, for when the original's
  style is unknown.

## How they fit together

```
orchestrator ── plans lanes, launches workers, merges
   │
   └─ matcher ── the working loop for each function
        ├─ wall-identifier ── "why won't this match?"
        ├─ helper-recovery ── open-coded helpers and macros
        ├─ permute ────────── proven register/schedule residue only
        └─ decomp-code-rules ─ what every line must satisfy, and what counts as a match
```

## Installation

Copy the `.claude/skills/` directory into your decompilation repository (project skills), or into
`~/.claude/skills/` to make them available in every project:

```bash
cp -r .claude/skills/* /path/to/your-decomp/.claude/skills/
```

Claude Code finds the skills automatically. Claude picks them up from the task ("why won't this function
match?"), or you can invoke one directly, for example `/matcher` or `/wall-identifier`.

## Setting up a project

1. **Create `AGENTS.md`** at the root of the game's repository from
   [`AGENTS-template.md`](.claude/skills/matcher/references/AGENTS-template.md). If you don't, the skills will use
   the template and ask you to fill it in.
2. **Fill in the project adapter.** Map the generic verbs the skills use to your project's commands:

   | verb | meaning |
   |---|---|
   | `build` | compile every unit and compare every annotated function |
   | `match <unit>` | compile and compare one unit (the inner loop) |
   | `diff <function>` | side-by-side compiled vs retail disassembly |
   | `report` | refresh the progress report (objdiff or similar) |
   | `gate` | the checks that must stay green before a commit or merge |
   | `disasm` / `xref` / `class` | retail assembly, callers and callees, vtable and hierarchy evidence |
   | `permute <function>` | the permuter, if you have one |
   | `rules [<unit>]` | the rule checker |

3. **Record the rest of the profile:** toolchain and flags, annotation markers, reference sources and how far to
   trust them, and policies (decompiler use, running the game, commit style).
4. **Pin the code-rules profile:** the compiler dialect, the naming and layout style per module (from evidence
   where possible, otherwise from `style-defaults.md`), placeholder patterns, and whether the binary records line
   information.

The project's `AGENTS.md` always overrides the skills' defaults.

## Requirements

The skills assume you already have a matching-decomp toolchain. They don't provide one. Typically that means:

- the original compiler and linker (or an exact equivalent) and the retail binary;
- a split/build setup that can compile individual units and compare them with retail;
- a disassembler or diff tool, and optionally a decompiler, a permuter and objdiff;
- git, since the orchestrator works in worktrees and merges through branches.

## Legal

This repository contains only methodology. It contains no game code, assets or binaries. Use it with games you
own, and follow the laws and the original developer's terms that apply to you.

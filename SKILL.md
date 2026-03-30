---
name: arkts-skill
description: ArkTS language specification guidance for syntax, types, semantics, and diagnostics based on ArkTS Specification 1.2.0. Use when handling ArkTS grammar questions, compiler/type errors, feature compliance checks, TypeScript-to-ArkTS adaptation, or HarmonyOS ArkTS code modernization.
---

# ArkTS Skill

Use this skill to answer ArkTS language and type-system questions with spec-backed guidance.

## Workflow

1. Read the user problem and classify it as one of: syntax, type system, declarations, expressions/statements, classes/interfaces, or migration/compatibility.
2. If the question is TypeScript-to-ArkTS migration related (especially "why not supported", including destructuring-assignment concerns), read `references/huawei-migration-priority.md` first.
3. For migration questions, use local files in `references/huawei-migration/*.md` first (local-first).
4. If local migration files are stale or insufficient, refresh them via `bash scripts/fetch_huawei_migration_refs.sh`.
5. Search chapter files, then open only the matched files.
6. Cite the exact section/topic names in your answer.
7. Prefer minimal, directly compilable ArkTS examples.
8. Keep recommendations simple and deterministic. If multiple options exist, present the simplest compliant one first.

## Fast Search Patterns

Use targeted search on the chapter directory:

- `bash scripts/search_chapters.sh "union|narrow|keyof"`
- `rg -n "type|union|interface|class|function|async|decorator|generic" references/chapters`
- `rg -n "assignment|narrow|overload|extends|implements|readonly|tuple|array|literal" references/chapters`
- `rg -n "error|diagnostic|restriction|forbidden|not allowed|shall|must" references/chapters`

## Maintenance

- Rebuild chapter split from full OCR markdown:
  - `python3 scripts/rebuild_chapters.py`
- Refresh migration local references:
  - `bash scripts/fetch_huawei_migration_refs.sh`
- Keep `references/CHAPTER_INDEX.md` in sync with chapter files after rebuild.

## Output Rules

- Prefer short explanations with one concrete fix.
- For compiler/type errors, include:
  - Root cause in one sentence.
  - Smallest valid code change.
  - Why this matches spec behavior.
- For migration questions, produce a before/after snippet and list breaking points.

## Compile Error Fix Playbook (Field-tested)

Use this checklist first when ArkTS build fails with strict diagnostics:

1. Reproduce with CLI and capture exact diagnostics:
   - `hvigorw assembleHap`
2. Map each error code to one direct rewrite, then recompile immediately.

### Frequent Errors and Minimal Fixes

- `arkts-no-untyped-obj-literals`:
  - Root cause: inline object literals in places where ArkTS requires explicit class/interface instances.
  - Fix: replace `{ ... }` values with class instances (`new Xxx(...)`) or strongly typed containers.
- `arkts-no-obj-literals-as-types`:
  - Root cause: using object literal shape directly in type positions (for example inline `{ v: string, w: number }` arrays).
  - Fix: extract a named `class`/`interface` (for example `WeightedItem`) and use that type.
- `arkts-no-props-by-index`:
  - Root cause: dynamic property indexing on fields like `obj[key]`.
  - Fix: replace with `Map<K,V>` (`get/set`) or explicit `switch`/branch lookup.

### Practical Rewrite Patterns

- `Record<string, T>` dictionaries with dynamic keys:
  - Prefer `Map<string, T>` for mutable runtime lookup.
- i18n maps accessed by `I18N[lang][key]`:
  - Replace with deterministic `switch` function (`t(key)`), especially when key is runtime string.
- Bigram/follower tables as plain object literals:
  - Replace with explicit lookup function `followersOf(ch)` using `switch`.
- Weak-key and weighted pools:
  - Replace anonymous object items with named classes (`WeakKey`, `WeightedItem`) to satisfy strict typing.

### Validation Loop

1. Change the smallest failing region.
2. Re-run `hvigorw assembleHap`.
3. Continue until only warnings remain.
4. Report separately:
   - Compile blockers fixed.
   - Remaining warnings (deprecated APIs, throw-handling hints).


## Reference

- Full OCR markdown: `arktsspecification_ocr_full.md`
- Chapter index: `references/CHAPTER_INDEX.md`
- OCR chapter files: `references/chapters/*.md`
- Migration-first references: `references/huawei-migration-priority.md`
- Migration local files: `references/huawei-migration/*.md`

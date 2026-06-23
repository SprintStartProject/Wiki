# ADR-00X: Use Tree-sitter for AST-Based Code Chunking

| Field | Value |
|---|---|
| **Status** | Accepted |
| **Date** | 2026-06-23 |
| **Deciders** | Ai Team |
| **Supersedes** | – |
| **Superseded by** | – |

---

## Context

The ingestion pipeline currently parses source code files using language-specific regular expressions to identify top-level definitions such as functions, classes, constants, and variables.

This approach has several limitations:

- Only Python, JavaScript, TypeScript, and Go are supported.
- Parsing relies on heuristic pattern matching rather than actual language syntax.
- Decorators, annotations, comments, multiline signatures, and other language constructs may be assigned to incorrect chunks.
- Nested definitions and complex syntax are difficult to handle reliably.
- Supporting additional programming languages requires implementing and maintaining custom regex patterns.

As the project grows and additional languages are expected to be supported, maintaining regex-based parsing becomes increasingly difficult and error-prone.

Tree-sitter is a parser framework that generates concrete syntax trees (CSTs) for source code and supports more than 100 programming languages through reusable grammars. It provides a language-aware mechanism for identifying top-level code structures while using a consistent API across languages.

---

## Decision Drivers

- Improve correctness of code chunk boundaries.
- Correctly associate decorators, annotations, comments, and multiline signatures with their definitions.
- Support multiple programming languages through a unified parsing approach.
- Reduce maintenance overhead caused by language-specific regex patterns.
- Preserve existing ingestion behavior for non-code files.
- Maintain acceptable ingestion performance.
- Enable future language support without significant implementation effort.
- Enrich chunk metadata with symbol information (e.g. function or class names).

---

## Considered Options

- **Option A** – Continue using regex-based parsing.
- **Option B** – Use language-specific AST parsers for each supported language.
- **Option C** – Use Tree-sitter as the primary code parser with regex fallback.

---

## Decision

**Chosen option: Option C – Use Tree-sitter as the primary code parser with regex fallback.**

Tree-sitter provides accurate syntax-aware code boundaries across multiple languages while preserving a consistent implementation model. Existing regex parsing will remain available as a fallback for unsupported languages.

---

## Rationale

The current regex-based implementation is simple but fundamentally limited because it cannot understand source code structure.

Language-specific parsers would provide accurate results but require maintaining different parsing implementations and APIs for each language. This would increase complexity and make adding new languages more expensive.

Tree-sitter provides a balance between correctness, maintainability, and extensibility:

- It accurately identifies top-level definitions using the language grammar.
- It correctly handles decorators, annotations, multiline signatures, and nested constructs.
- It provides a common API across languages.
- Additional languages can be supported by adding grammars rather than implementing new parsers.
- Existing chunking logic, metadata generation, and preamble handling can largely be retained.

Keeping the existing regex parser as a fallback minimizes migration risk and ensures unsupported languages continue to be processed.

---

## Pros and Cons of the Options

### Option A – Regex-Based Parsing

#### Pros

- Simple implementation.
- No additional dependencies.
- Fast execution.

#### Cons

- Incorrect handling of decorators, annotations, and multiline declarations.
- Difficult to extend reliably.
- Requires custom maintenance for every language.
- Not syntax-aware.

### Option B – Language-Specific AST Parsers

#### Pros

- Accurate parsing for each language.
- Full access to language-specific syntax information.

#### Cons

- Requires different implementations per language.
- Higher maintenance burden.
- Inconsistent APIs and parsing behavior.
- More difficult to add new languages.

### Option C – Tree-sitter with Regex Fallback

#### Pros

- Accurate syntax-aware parsing.
- Unified API across languages.
- Supports current languages and many future languages.
- Correct handling of decorators, annotations, comments, and multiline declarations.
- Easier long-term maintenance.
- Allows enrichment of metadata with symbol names and symbol types.
- Existing regex parser can remain as fallback.

#### Cons

- Introduces additional dependencies and grammar management.
- Slightly higher parsing overhead.
- Requires initial implementation effort and test coverage.

---

## Consequences

### Positive

- More accurate code chunk boundaries.
- Improved retrieval quality due to better chunk semantics.
- Decorators, comments, and annotations remain attached to their corresponding definitions.
- Consistent support for Python, JavaScript, TypeScript, and Go.
- Easier addition of future languages.
- Ability to store symbol metadata such as:
  - `symbol_name`
  - `symbol_kind`

### Negative / Trade-offs

- Additional dependency on Tree-sitter and language grammars.
- Increased implementation complexity compared to regex parsing.
- Slightly increased parsing time during ingestion.
- Grammar versions must be maintained and updated.

### Follow-up Actions

- Integrate Tree-sitter Python bindings.
- Add grammars for Python, JavaScript, TypeScript, and Go.
- Implement AST-based top-level definition extraction.
- Preserve existing preamble handling.
- Extend chunk metadata with symbol information.
- Keep the existing regex parser as fallback.
- Add tests for:
  - Decorators and annotations
  - Multiline signatures
  - Nested definitions
  - Empty files
  - Single-definition files
  - Large files with multiple definitions
adr-014-treestitter-ast-based-code-chunking- Benchmark ingestion performance before and after migration.

---

## Links


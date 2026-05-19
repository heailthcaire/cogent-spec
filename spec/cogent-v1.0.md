# Cogent v1.0

A proposed standard for composable LLM skills.

Status: Proposed v1.0 for review  
License: MIT  
Author: Alain Trottier

## Purpose

Cogent is a thin convention layer on top of LLM skill systems. It adds to plain prose what prose cannot express reliably on its own: imports, methods, control flow, contracts between skills, and runtime postconditions.

The goal is not to replace natural language. The goal is to make AI workflows composable, inspectable, testable, reusable, and easier to reason about.

## 1. Design Principles

1. Small surface area. The v1.0 surface should be small enough to understand quickly.
2. Executable, not interpretive. Every Cogent keyword should have one meaning.
3. Backwards-compatible inside the major version. Skills written for v1.0 should run on v1.x.

## 2. Lexical Conventions

- Keywords are ALL CAPS: `IMPORT`, `IF`, `CALL METHOD`.
- Skill and method names are kebab-case: `market-validation`, `score-competitor`.
- Variables are snake_case: `refined_concept`, `readiness`.
- Parameters are wrapped in double brackets: `[[concept]]`.
- Namespaces use `::`: `market-validation::main`.
- Comments use `#`. Anything after `#` on a line is ignored at execution time.

## 3. Skill Frontmatter

Every Cogent-compliant skill declares YAML frontmatter:

```yaml
---
name: market-validation
version: 1.2.0
description: Evaluate market size, competitors, and
  go-to-market viability for an app, product, or tool idea.
exports:
  - main([[concept]]) -> { market_size, competitors, gtm_recommendation }
  - score-competitor([[competitor_name]]) -> { score: number, reasoning: string }
requires:
  - data-research >= 1.0.0
---
```

Required fields:

- `name`: kebab-case identifier, unique across installed skills.
- `version`: semantic version number in `MAJOR.MINOR.PATCH` format.
- `description`: trigger surface the model matches against context.
- `exports`: publicly callable methods with parameter and return signatures.
- `requires`: declared dependencies on other skills, with version constraints. Optional but recommended.

Skills that fail to declare `name`, `version`, `description`, or `exports` are not Cogent-compliant.

## 4. Imports

```text
IMPORT {{skill_name}}
IMPORT {{skill_name}}@{{version_constraint}}
```

`IMPORT` loads the referenced skill into active context and makes its exported methods callable under that skill's namespace.

Examples:

```text
IMPORT market-validation              # any installed version
IMPORT market-validation@1.2.0        # exact version
IMPORT market-validation@^1.0.0       # any 1.x version
IMPORT market-validation@>=1.0.0      # 1.0.0 or higher
```

If the requested version is not installed, the runtime reports the missing dependency and halts the current workflow rather than silently substituting another version.

## 5. Methods

Methods are the smallest composable unit in Cogent. A method has a name, declared parameters, an instruction body, optional postconditions, and an optional return value.

### Defining a Method

```text
DEFINE METHOD {{method_name}}([[{{parameter}}]]):
  {{instructions}}
  RETURN {{output}}
```

Method names live in the namespace of their containing skill. A method may not share its name with its containing skill. Methods listed in the skill's `exports` block are publicly callable; methods not listed are private.

### Calling a Method

```text
{{variable}} = CALL METHOD {{skill}}::{{method_name}}[[{{parameter}}]]
```

Within the same skill, the `skill::` prefix is optional. Across skills, it is required. Return values must be bound to a variable with `=` to be used downstream. A bare `CALL METHOD` executes the method for side effects and discards the return value.

### Field Access

Object-valued variables are read with dot notation:

```text
readiness = CALL METHOD assess-readiness[[refined]]

IF readiness.ready == true:
  ...
```

### Postconditions

A method may declare runtime requirements on its output using `REQUIRE`:

```text
DEFINE METHOD score-competitor([[competitor_name]]):
  Analyze pricing, distribution, and positioning of competitor_name.
  result = { score: number, reasoning: string }
  REQUIRE result.reasoning references at least one specific
    pricing, distribution, or positioning detail.
  RETURN result
```

A failed `REQUIRE` raises an error that propagates to the nearest `TRY/CATCH` block, just like a shape mismatch. This separates structural validity from semantic adequacy.

`REQUIRE` is the AI-native primitive in Cogent: the predicate is evaluated by the language model itself, against the method's own output, at the moment the method completes.

## 6. Control Flow

### Conditionals

```text
IF {{condition}}:
  {{action}}
ELSE IF {{condition}}:
  {{action}}
ELSE:
  {{action}}
```

The condition is a predicate the model evaluates against current context. Predicates may reference variables, compare values, or describe a state in natural language.

### Switch

```text
SWITCH {{variable}}:
  CASE {{value}}: {{action}}
  CASE {{value}}: {{action}}
  DEFAULT: {{action}}
```

Use `SWITCH` when branching on a discrete set of values.

### Loops

```text
FOR EACH {{item}} IN {{collection}}:
  {{action}}

REPEAT {{n}} TIMES:
  {{action}}
```

`FOR EACH` iterates over a known collection. `REPEAT` runs a block a fixed number of times.

### Parallel Execution

```text
PARALLEL FOR EACH {{item}} IN {{collection}}:
  {{action}}
```

`PARALLEL FOR EACH` signals that iterations have no interdependencies and may execute concurrently. As of v1.0, no LLM runtime executes Cogent iterations in parallel; the keyword currently behaves identically to `FOR EACH`.

The semantic distinction matters: marking a loop `PARALLEL` is a contract that iterations are independent. Iterations that mutate shared state inside a `PARALLEL` block produce undefined behavior.

## 7. Error Handling

```text
TRY:
  {{action}}
CATCH {{error_variable}}:
  {{recovery_action}}
FINALLY:
  {{cleanup_action}}
```

A method fails when it cannot produce its declared return shape, when a `REQUIRE` postcondition is not satisfied, when a required field is missing from its input, or when an `IMPORT` cannot be resolved.

Example:

```text
TRY:
  validation = CALL METHOD market-validation::main[[refined]]
CATCH err:
  Log err.message and fall back to a manual recommendation.
  validation = { gtm_recommendation: "manual review required" }
```

`FINALLY` is optional and runs whether or not the `TRY` block succeeded.

## 8. Reserved Keywords and Operators

Keywords:

```text
IMPORT
DEFINE METHOD
CALL METHOD
RETURN
REQUIRE
IF
ELSE IF
ELSE
SWITCH
CASE
DEFAULT
FOR EACH
IN
REPEAT
TIMES
PARALLEL
TRY
CATCH
FINALLY
```

Operators:

```text
=       binding
::      namespace
->      return type
.       field access
@       version pin
#       comment
== != < > <= >= comparisons
```

Total surface: 19 keywords and 11 operators.

## 9. Minimum Compliant Skill

See:

- [`examples/product-ideation.cogent.md`](../examples/product-ideation.cogent.md)
- [`examples/market-validation.cogent.md`](../examples/market-validation.cogent.md)

## 10. Conformance Checklist

A skill is Cogent v1.0 compliant if and only if:

- Frontmatter includes `name`, `version`, `description`, and `exports`.
- `version` follows semantic versioning.
- Every `exports` entry includes a parameter signature and a return shape.
- Every cross-skill method call uses the `skill::method` namespace.
- No method shares a name with its containing skill.
- Every method return value used downstream is bound with `=`.
- Every `IMPORT` of a versioned dependency declares its version constraint.
- Reserved keywords appear in ALL CAPS exactly as listed in Section 8.
- `REQUIRE` predicates, where present, are evaluable against the method's declared return shape.

A v1.0-compliant skill is statically verifiable. A tool can read the frontmatter and body and report violations before runtime.

## 11. Versioning Policy

Cogent itself follows semantic versioning.

- Patch releases, such as `v1.0.1`, fix wording and edge cases.
- Minor releases, such as `v1.1`, add features without breaking existing skills.
- Major releases, such as `v2.0`, may break compatibility, with a documented migration path.

Skills may declare their target Cogent version in optional frontmatter:

```yaml
cogent: 1.0
```

## 12. Roadmap

See [`roadmap/v1.1-proposals.md`](../roadmap/v1.1-proposals.md).

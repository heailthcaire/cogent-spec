# Cogent v1.0 Conformance Tests

This file defines a small test suite for evaluating whether a Cogent runtime, model, or validator can execute and reason about the v1.0 spec.

The tests assume the following example skills are available:

- `examples/product-ideation.cogent.md`
- `examples/market-validation.cogent.md`

## Test 1: Successful End-to-End Execution

### Prompt

```text
I have an idea for a meal-planning app for new parents who are too tired to cook.
```

### Expected Behavior

1. Description match loads `product-ideation`.
2. `IMPORT market-validation@^1.0.0` resolves.
3. `refine-concept` extracts:
   - core problem
   - target user
   - MVP features
4. `REQUIRE result.target_user names a specific demographic` passes.
5. `assess-readiness` returns `ready: true`.
6. `market-validation::main` executes.
7. `score-competitor` runs inside `PARALLEL FOR EACH`.
8. Each `score-competitor` output satisfies its `REQUIRE`.
9. Final report returns `{ refined, validation }`.

### Pass Criteria

The workflow returns a structured `final_report` containing a refined concept and validation output.

## Test 2: Semantic Postcondition Failure

### Prompt

```text
I want to build something for small businesses.
```

### Expected Behavior

1. `refine-concept` produces a target user that is too generic, such as `small businesses`.
2. The `REQUIRE` postcondition fails.
3. The error propagates to the outer `TRY/CATCH`.
4. `main` returns a `final_report` with a blocker explaining that the target user is too generic.

### Pass Criteria

The runtime catches a semantic-quality failure at the method boundary instead of returning a weak concept as if it succeeded.

## Test 3: Failure Inside Iteration

### Prompt

```text
Idea: an app for dog walkers, but I am not sure if there is a market.
```

### Expected Behavior

1. `refine-concept` succeeds.
2. `assess-readiness` returns `ready: true`.
3. `market-validation::main` runs.
4. At least one `score-competitor` call produces insufficient reasoning.
5. The failed `REQUIRE` raises an error.
6. Because the iteration is not locally wrapped in `TRY/CATCH`, the error propagates to `main`.
7. `main` returns a blocker.

### Pass Criteria

The runtime correctly propagates an iteration-level failure to the nearest enclosing `TRY/CATCH`.

## Static Validation Checklist

A validator should be able to flag:

- missing `name`, `version`, `description`, or `exports`
- invalid semantic version format
- exported methods without parameter signatures
- exported methods without return shapes
- cross-skill calls that omit the `skill::method` namespace
- method names that duplicate the containing skill name
- downstream use of unbound return values
- versioned imports without version constraints
- reserved keywords not written in ALL CAPS
- `REQUIRE` predicates that refer to fields not present in the declared return shape

## Known v1.0 Gap

When a `REQUIRE` fails inside a `FOR EACH` or `PARALLEL FOR EACH`, the whole loop aborts unless the iteration is wrapped in `TRY/CATCH`.

This is correct under v1.0, but it motivates a v1.1 proposal for per-iteration error handling.

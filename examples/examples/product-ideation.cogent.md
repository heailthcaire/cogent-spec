---
name: product-ideation
version: 1.0.0
cogent: 1.0
description: Use when the user describes an idea for an app, product, or tool and wants help refining the concept.
exports:
  - main([[user_message]]) -> { final_report }
requires:
  - market-validation >= 1.0.0
---

# Product Ideation Example

## Where This Text Goes

This file is the reusable Cogent skill.

Suggested location:

```text
skills/product-ideation.cogent.md
```

or, for this repository:

```text
examples/product-ideation.cogent.md
```

The YAML frontmatter and Cogent methods below define the workflow. They are not the user's prompt. The user's prompt becomes the value passed into `[[user_message]]`.

## What The User Provides

The user provides a rough idea, such as:

```text
I have an idea for a meal-planning app for new parents who are too tired to cook.
```

In a native Cogent runtime, that text would be passed into:

```text
CALL METHOD product-ideation::main[[user_message]]
```

In today's LLM tools, paste or attach this skill first, then provide the user's rough idea and ask the model to execute `main`.

Example current-use prompt:

```text
Use the product-ideation Cogent skill below.

Execute main with this user_message:
I have an idea for a meal-planning app for new parents who are too tired to cook.
```

## Skill Definition

This example demonstrates:

- `IMPORT`
- private methods
- public `main`
- `REQUIRE`
- `IF / ELSE`
- cross-skill method calls
- `TRY / CATCH`

```text
IMPORT market-validation@^1.0.0

# Private helper; not in exports, not callable from outside.
DEFINE METHOD refine-concept([[raw_idea]]):
  Extract the core problem, target user, and 3 MVP features
  from raw_idea. Be concrete and specific.
  result = { concept, target_user, mvp_features }
  REQUIRE result.target_user names a specific demographic, not
    a generic group like "people" or "businesses".
  RETURN result

DEFINE METHOD assess-readiness([[refined]]):
  Determine if refined.target_user is specific enough to
  validate market fit.
  RETURN { ready: boolean, reason: string }

# Public entry point; listed in exports.
DEFINE METHOD main([[user_message]]):
  TRY:
    refined = CALL METHOD refine-concept[[user_message]]
    readiness = CALL METHOD assess-readiness[[refined]]

    IF readiness.ready == true:
      validation = CALL METHOD market-validation::main[[refined]]
      RETURN { final_report: { refined, validation } }
    ELSE:
      RETURN { final_report: { refined, blocker: readiness.reason } }
  CATCH err:
    RETURN { final_report: { blocker: err.message } }
```

---
name: market-validation
version: 1.2.0
cogent: 1.0
description: Evaluate market size, competitors, and go-to-market viability for an app, product, or tool idea.
exports:
  - main([[concept]]) -> { market_size, competitors, gtm_recommendation }
  - score-competitor([[competitor_name]]) -> { score: number, reasoning: string }
---

# Market Validation Example

## Where This Text Goes

This file is the reusable Cogent skill.

Suggested location:

```text
skills/market-validation.cogent.md
```

or, for this repository:

```text
examples/market-validation.cogent.md
```

The YAML frontmatter and Cogent methods below define the workflow. They are not the user's prompt. The user's product concept becomes the value passed into `[[concept]]`.

## What The User Provides

The user or another skill provides a structured concept, such as:

```text
concept =
{
  product_concept: "A registration-readiness planner for college students",
  target_user: "working, commuter, transfer, and first-generation college students",
  target_buyer: "academic advising offices and student success departments",
  painful_workflow: "Students arrive at advising appointments unprepared for registration planning"
}
```

In a native Cogent runtime, that concept would be passed into:

```text
CALL METHOD market-validation::main[[concept]]
```

In today's LLM tools, paste or attach this skill first, then provide the concept and ask the model to execute `main`.

Example current-use prompt:

```text
Use the market-validation Cogent skill below.

Execute main with this concept:
{
  product_concept: "A registration-readiness planner for college students",
  target_user: "working, commuter, transfer, and first-generation college students",
  target_buyer: "academic advising offices and student success departments",
  painful_workflow: "Students arrive at advising appointments unprepared for registration planning"
}
```

## Skill Definition

This example demonstrates:

- exported methods
- semantic postconditions with `REQUIRE`
- `PARALLEL FOR EACH`
- structured return values

```text
DEFINE METHOD score-competitor([[competitor_name]]):
  Analyze pricing, distribution, and positioning of competitor_name.
  result = { score: number, reasoning: string }
  REQUIRE result.reasoning references at least one specific
    pricing, distribution, or positioning detail.
  RETURN result

DEFINE METHOD main([[concept]]):
  competitors_found = Identify 3-5 direct competitors for concept.

  scored = []
  PARALLEL FOR EACH competitor IN competitors_found:
    score = CALL METHOD score-competitor[[competitor]]
    Append score to scored.

  market_size = Estimate TAM/SAM/SOM for concept.
  gtm_recommendation = Synthesize a go/no-go from scored and market_size.

  RETURN { market_size, competitors: scored, gtm_recommendation }
```

## Notes

`PARALLEL FOR EACH` is used because each competitor can be scored independently. As of v1.0, this is a semantic contract rather than a guaranteed runtime optimization.

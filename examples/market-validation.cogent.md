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

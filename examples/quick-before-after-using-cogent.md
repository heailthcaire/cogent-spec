# Quick Before and After: Why Use Cogent?

This short example shows the basic value of Cogent: turning a one-off prompt into a reusable skill with a clear input, output, and quality check.

## Before: Plain Prompt

```text
Help me improve this product idea. Tell me who it is for, what problem it solves, and whether it is worth building.

Idea: an app that helps students plan college classes around work, sports, commuting, and other real-life constraints.
```

## Typical Result

```text
This app is for college students who need help planning classes. It solves the problem of confusing registration systems and scheduling conflicts. It may be worth building if students and colleges find it useful.
```

Useful, but vague.

The prompt does not require a specific user, a clear buyer, or a testable output.

## After: Cogent Skill

The Cogent text goes in a reusable skill file:

```text
skills/refine-product-idea.cogent.md
```

```text
---
name: refine-product-idea
version: 1.0.0
cogent: 1.0
description: Use when the user provides a rough product idea and wants it clarified.
exports:
  - main([[rough_idea]]) -> { refined_idea }
---

DEFINE METHOD main([[rough_idea]]):
  Extract the target user, buyer, painful workflow, and MVP from rough_idea.
  result = { target_user, buyer, painful_workflow, mvp }
  REQUIRE result.target_user names a specific user segment.
  REQUIRE result.buyer identifies who would pay or approve purchase.
  REQUIRE result.mvp contains only the smallest useful feature set.
  RETURN { refined_idea: result }
```

## User Input

The user's prompt becomes the input:

```text
rough_idea =
an app that helps students plan college classes around work, sports, commuting, and other real-life constraints
```

In a native Cogent runtime:

```text
CALL METHOD refine-product-idea::main[[rough_idea]]
```

In today's LLM tools:

```text
Use the refine-product-idea Cogent skill.

Execute main with this rough_idea:
an app that helps students plan college classes around work, sports, commuting, and other real-life constraints
```

## Better Result

```json
{
  "refined_idea": {
    "target_user": "working, commuter, transfer, and student-athlete college students",
    "buyer": "academic advising offices and student success departments",
    "painful_workflow": "students arrive at registration or advising meetings without a realistic schedule plan that accounts for real-life constraints",
    "mvp": [
      "guided intake",
      "constraint calendar",
      "preferred course list",
      "backup course list",
      "advisor-ready export"
    ]
  }
}
```

## Benefit

The plain prompt asks for help.

The Cogent version defines a reusable workflow with:

- a named skill
- a public method
- a clear input
- a predictable output
- quality checks using `REQUIRE`

That makes the result more specific, repeatable, and easier to test.

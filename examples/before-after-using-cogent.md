# Before and After: Turning a Rambling Prompt Into a Cogent Skill

This example demonstrates how Cogent can turn a long, informal prompt into a more functional, reusable AI workflow.

The point is not that natural language is bad. The point is that long prompts often mix goals, instructions, criteria, outputs, and recovery behavior into one unstructured block. Cogent separates those concerns.

## Scenario

A user wants help evaluating whether a new software product idea is commercially viable.

## Before: Rambling Prompt

```text
I have an idea for a software product and I want you to tell me if it is any good.
The idea is a tool for college students who hate planning classes. The college
registration systems are confusing and mostly built for administrators, not
students. Students have jobs, sports, clubs, commuting issues, and preferred
times, but the systems do not really help them build a schedule around real
life. I want the tool to help students prepare before they meet an advisor,
maybe create a schedule, check conflicts, list backup classes, and produce
something they can show the counselor.

Please analyze whether this is commercially viable. Think about who would use
it, who would pay, whether universities would buy it, whether students would
actually use it, competitors, risks, MVP, pricing, integrations, and whether
this should be sold to students or colleges. Be honest. Also suggest a stronger
version of the idea if my first version is not the best one.
```

## Typical Result From the Rambling Prompt

```text
This is a promising idea because students often struggle with course planning.
The target users are students, especially first-generation, commuter, transfer,
and working students. The buyers may be colleges, advising offices, or student
success departments.

The product could help students enter constraints, choose classes, identify
conflicts, and generate an advisor-ready plan. Existing competitors include
student information systems, degree audit tools, registration portals, and
calendar tools. These tools may not solve the student-preparation problem well.

Risks include SIS integration difficulty, student adoption, institutional sales
cycles, and the possibility that existing tools already solve enough of the
problem. The MVP could include guided intake, constraint calendar, preferred
course list, backup courses, readiness checklist, advisor export, and dashboard.

Overall, this may be commercially viable, especially if positioned as an
advisor-prep and registration-readiness platform for community colleges and
large universities.
```

## What Is Missing

The result is useful, but the prompt is hard to reuse or test.

Problems:

- The task has no explicit method boundaries.
- The model is doing refinement, market analysis, buyer analysis, and MVP design all at once.
- There is no declared output contract.
- There are no quality checks.
- There is no required specificity for target user, buyer, or beachhead.
- There is no error behavior if the idea is too vague.
- Another workflow cannot safely call this prompt as a reusable skill.

## After: Cogent Version

```text
---
name: product-opportunity-review
version: 1.0.0
cogent: 1.0
description: Use when a user provides a rough software product idea and wants a structured commercial viability review.
exports:
  - main([[rough_idea]]) -> { opportunity_report }
requires:
  - market-validation >= 1.0.0
---

IMPORT market-validation@^1.0.0

DEFINE METHOD refine-idea([[rough_idea]]):
  Extract the product concept, target user, target buyer, painful workflow,
  current workaround, and likely product category from rough_idea.
  result = {
    product_concept,
    target_user,
    target_buyer,
    painful_workflow,
    current_workaround,
    product_category
  }
  REQUIRE result.target_user names a specific user segment, not a generic group.
  REQUIRE result.target_buyer identifies who controls budget or purchasing.
  REQUIRE result.painful_workflow describes a recurring workflow, not a one-time inconvenience.
  RETURN result

DEFINE METHOD identify-beachhead([[refined]]):
  Choose the narrowest first market where the pain is frequent, valuable,
  and reachable by a small team.
  result = { beachhead_market, reason, adoption_risk }
  REQUIRE result.beachhead_market is narrower than the full target market.
  REQUIRE result.reason explains why this segment feels the pain more strongly.
  RETURN result

DEFINE METHOD design-mvp([[refined]], [[beachhead]]):
  Define the smallest product that proves value for the beachhead market.
  Include only features needed to complete the core workflow.
  result = { core_workflow, mvp_features, source_of_truth_records }
  REQUIRE result.core_workflow contains 4 to 7 steps.
  REQUIRE result.mvp_features excludes nonessential advanced features.
  RETURN result

DEFINE METHOD assess-commercial-risk([[refined]], [[beachhead]], [[mvp]]):
  Evaluate buyer willingness to pay, user adoption, competitive pressure,
  integration burden, and sales difficulty.
  result = {
    viability_score,
    key_risks,
    strongest_positioning,
    validation_experiment
  }
  REQUIRE result.viability_score is a number from 1 to 10.
  REQUIRE result.validation_experiment can be run cheaply before building the full product.
  RETURN result

DEFINE METHOD main([[rough_idea]]):
  TRY:
    refined = CALL METHOD refine-idea[[rough_idea]]
    beachhead = CALL METHOD identify-beachhead[[refined]]
    mvp = CALL METHOD design-mvp[[refined]][[beachhead]]
    market = CALL METHOD market-validation::main[[refined]]
    risk = CALL METHOD assess-commercial-risk[[refined]][[beachhead]][[mvp]]

    RETURN {
      opportunity_report: {
        refined,
        beachhead,
        mvp,
        market,
        risk
      }
    }
  CATCH err:
    RETURN {
      opportunity_report: {
        blocker: err.message,
        next_step: "Clarify the rough idea and rerun the review."
      }
    }
```

## Same User Input Passed Into the Cogent Skill

```text
rough_idea =
I have an idea for a software product and I want you to tell me if it is any good.
The idea is a tool for college students who hate planning classes. The college
registration systems are confusing and mostly built for administrators, not
students. Students have jobs, sports, clubs, commuting issues, and preferred
times, but the systems do not really help them build a schedule around real
life. I want the tool to help students prepare before they meet an advisor,
maybe create a schedule, check conflicts, list backup classes, and produce
something they can show the counselor.
```

## After: Structured Result

```json
{
  "opportunity_report": {
    "refined": {
      "product_concept": "A registration-readiness and advisor-prep platform that helps students prepare realistic course plans before advising appointments.",
      "target_user": "working, commuter, transfer, first-generation, first-year, and student-athlete college students who need schedules that fit real-life constraints",
      "target_buyer": "academic advising offices, student success departments, registrar teams, community colleges, and universities with overloaded advising capacity",
      "painful_workflow": "Students arrive at registration or advising appointments unprepared, forcing staff to extract constraints, explain options, resolve avoidable conflicts, and build schedules under time pressure.",
      "current_workaround": "Students use degree audits, registration portals, screenshots, spreadsheets, notes, emails, calendar apps, and last-minute advisor conversations.",
      "product_category": "student success workflow software / advisor-prep platform"
    },
    "beachhead": {
      "beachhead_market": "community colleges and commuter-heavy universities with high advising volume and many working or transfer students",
      "reason": "These institutions feel the registration-preparation problem more often because students have complex constraints and advising teams are capacity-limited.",
      "adoption_risk": "Students may not complete intake unless advisors require or strongly encourage it before appointments."
    },
    "mvp": {
      "core_workflow": [
        "Student completes guided intake.",
        "Student enters work, commute, athletics, club, family, and time constraints.",
        "Student lists preferred courses and backup courses.",
        "System checks conflicts and readiness gaps.",
        "Advisor reviews a concise student-prep brief.",
        "Student leaves with an advisor-approved registration-day plan."
      ],
      "mvp_features": [
        "guided intake wizard",
        "constraint calendar",
        "preferred and backup course list",
        "readiness checklist",
        "advisor review page",
        "PDF or shareable export",
        "simple readiness dashboard"
      ],
      "source_of_truth_records": [
        "student goals",
        "life constraints",
        "course intent",
        "backup courses",
        "registration blockers",
        "advisor notes",
        "readiness status",
        "final registration plan"
      ]
    },
    "market": {
      "market_size": "Promising but institution-sales constrained; strongest in advising-capacity and student-success budgets.",
      "competitors": [
        "student information systems",
        "degree audit tools",
        "registration portals",
        "calendar tools",
        "student success platforms"
      ],
      "gtm_recommendation": "Test first with advising offices at community colleges before building deep SIS integrations."
    },
    "risk": {
      "viability_score": 7,
      "key_risks": [
        "long institutional sales cycles",
        "SIS integration expectations",
        "student completion behavior",
        "overlap with existing student success platforms",
        "advisor workflow change resistance"
      ],
      "strongest_positioning": "The advisor-prep system that helps students show up ready to register.",
      "validation_experiment": "Interview 10 academic advisors and run a clickable prototype with 20 students before building integrations."
    }
  }
}
```

## What Improved

The Cogent version is better because it makes the workflow inspectable.

Improvements:

- The rough idea is refined before market assessment.
- Each step has a clear job.
- Each step returns a declared shape.
- `REQUIRE` statements define quality gates.
- The final output is predictable.
- Another skill can call this workflow safely.
- A validator can inspect the skill before runtime.
- The workflow can fail usefully when the input is too vague.

## Takeaway

The original prompt can produce a useful answer.

The Cogent version produces a reusable workflow.

That is the difference between prompting for one answer and designing a skill another AI workflow can call, test, and improve.

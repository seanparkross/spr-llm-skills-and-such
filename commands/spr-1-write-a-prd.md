# Write a PRD

Turn a free-form description of a problem into a structured PRD file through codebase exploration and relentless user interview.

## Process

### 1. Collect the plan

Ask the user for a long, detailed description of the problem they want to solve and any initial ideas for solutions. There is no required format — a brain dump is fine. The goal is to understand their thinking before touching the codebase.

Also ask for the file format and location where the PRD should be saved.

### 2. Explore the codebase

Before interviewing the user, explore the repo to verify their assertions and understand the current state of the implementation. Look for:

- Modules and files that will be affected by this change
- Existing patterns, conventions, and abstractions to follow or build on
- Anything that contradicts or complicates the user's description of the current state

Share findings with the user before starting the interview, especially anything that contradicts their description. The interview should be grounded in the codebase as it actually is, not as either party assumed.

### 3. Interview the user

Interview the user relentlessly about every aspect of the plan until you reach a shared, unambiguous understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one.

Do not move to the next branch until the current one is resolved. Do not accept vague answers — if the user says "it depends", ask what it depends on and resolve each case.

Cover at minimum:

- Every actor who interacts with the feature and what they need
- Every failure mode and what the correct behaviour is
- Every edge case that the user stories imply
- Every integration with existing modules or external systems
- Any decisions that would be difficult or expensive to reverse

When a branch feels resolved, summarise the decisions back to the user and ask whether anything is missing before moving on. Draft the corresponding PRD section now rather than waiting until the end — it is cheaper to correct misunderstandings one section at a time than after a full document lands.

### 4. Design the modules

Sketch out the major modules that will be built or modified. Actively look for opportunities to extract deep modules — modules that encapsulate significant functionality behind a simple, stable, testable interface.

A deep module is the opposite of a shallow one: it hides complexity rather than exposing it. Prefer fewer, deeper modules over many thin ones.

For each module, confirm with the user:

- Does this match their expectations?
- Should tests be written for this module?
- Which parts of its interface are likely to change, and which are stable?

### 5. Assemble the PRD

By this point, most sections should already be drafted from the running summaries in steps 3 and 4. Stitch them together into the template below, fill in any sections that were not yet drafted (Problem Statement, Solution, Out of Scope, Open Questions, Further Notes), and save to the file the user specified.

<prd-template>
# PRD: <feature name>

## Problem Statement

The problem the user is facing, from the user's perspective. Not a technical description — describe the gap between what exists and what is needed.

## Solution

The solution to the problem, from the user's perspective. Not an implementation plan — describe what will be true when the feature is complete.

## User Stories

A long, numbered list of user stories covering all aspects of the feature. Each story follows the format:

> As a `<actor>`, I want `<feature>`, so that `<benefit>`.

Be exhaustive. Include stories for error states, edge cases, and secondary actors. A story that implies a behaviour should have a corresponding story for the failure case.

## Implementation Decisions

Decisions made during the interview that constrain or shape the implementation. Include:

- Architectural decisions and their rationale
- Schema changes
- API contracts
- Specific interaction patterns

Module-level detail belongs in Module Design, not here. Do NOT include file paths or code snippets — these become outdated quickly.

## Module Design

For each module identified in step 4:

- **Name**: what to call it
- **Responsibility**: the single thing it owns
- **Interface**: what callers need to know (inputs, outputs, failure modes)
- **Tested**: yes / no

## Testing Decisions

- What makes a good test for this feature (test external behaviour, not implementation details)
- Which modules will have tests written
- Prior art in the codebase — similar tests to use as reference

## Out of Scope

Explicit list of things that will not be addressed in this PRD. Be specific — vague out-of-scope items create ambiguity later.

## Open Questions

Any unresolved questions that could not be answered during the interview, with a suggested resolution path for each.

## Further Notes

Any context, constraints, or decisions that do not fit the above sections.
</prd-template>

---
name: brainstorming
description: "You MUST use this before any creative work - creating features, building components, adding functionality, or modifying behavior. Explores user intent, requirements and design before implementation."
---

# Brainstorming Ideas Into Designs

## Overview

Help turn ideas into fully formed designs and specs through natural collaborative dialogue.

Start by understanding the current project context, then ask questions in focused batches to refine the idea. Once you understand what you're building, present the design and get user approval.

<HARD-GATE>
Do NOT invoke any implementation skill, write any code, scaffold any project, or take any implementation action until you have presented a design and the user has approved it. This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>

## Anti-Pattern: "This Is Too Simple To Need A Design"

Every project goes through this process. A todo list, a single-function utility, a config change — all of them. "Simple" projects are where unexamined assumptions cause the most wasted work. The design can be short (a few sentences for truly simple projects), but you MUST present it and get approval.

## Checklist

You MUST create a task for each of these items and complete them in order:

1. **Explore project context** — check files, docs, recent commits
2. **Ask clarifying questions** — in focused batches, understand purpose/constraints/success criteria
3. **Propose 2-3 approaches** — with trade-offs and your recommendation
4. **Present design** — in two phases: Phase 1 (architecture + components), then Phase 2 (data flow, error handling, testing) after Phase 1 is approved
5. **Write design doc** — save to `docs/plans/YYYY-MM-DD-<topic>-design.md` and commit
6. **Transition to implementation** — invoke writing-plans skill to create implementation plan

## Process Flow

```dot
digraph brainstorming {
    "Explore project context" [shape=box];
    "Ask clarifying questions" [shape=box];
    "Propose 2-3 approaches" [shape=box];
    "Phase 1: Architecture + Components" [shape=box];
    "Phase 1 approved?" [shape=diamond];
    "Phase 2: Data flow, error handling, testing" [shape=box];
    "Phase 2 approved?" [shape=diamond];
    "Write design doc" [shape=box];
    "Invoke writing-plans skill" [shape=doublecircle];

    "Explore project context" -> "Ask clarifying questions";
    "Ask clarifying questions" -> "Propose 2-3 approaches";
    "Propose 2-3 approaches" -> "Phase 1: Architecture + Components";
    "Phase 1: Architecture + Components" -> "Phase 1 approved?";
    "Phase 1 approved?" -> "Phase 1: Architecture + Components" [label="no, revise"];
    "Phase 1 approved?" -> "Phase 2: Data flow, error handling, testing" [label="yes"];
    "Phase 2: Data flow, error handling, testing" -> "Phase 2 approved?";
    "Phase 2 approved?" -> "Phase 2: Data flow, error handling, testing" [label="no, revise"];
    "Phase 2 approved?" -> "Write design doc" [label="yes"];
    "Write design doc" -> "Invoke writing-plans skill";
}
```

**The terminal state is invoking writing-plans.** Do NOT invoke frontend-design, mcp-builder, or any other implementation skill. The ONLY skill you invoke after brainstorming is writing-plans.

## The Process

**Understanding the idea:**
- Check out the current project state first (files, docs, recent commits)
- Ask questions in focused batches — group related questions together rather than one at a time
- Ask questions as plain text in the conversation, not using the AskUserQuestion tool
- Focus on understanding: purpose, constraints, success criteria

**Exploring approaches:**
- Propose 2-3 different approaches with trade-offs
- Present options conversationally with your recommendation and reasoning
- Lead with your recommended option and explain why

**Presenting the design:**
- Once you believe you understand what you're building, present the design in two phases
- **Phase 1: Architecture + Components** — present the high-level structure and the pieces that make it up together, then get user approval before continuing
- **Phase 2: Data flow, error handling, testing** — once Phase 1 is approved, present these together as one cohesive section
- Scale each phase to its complexity: a few sentences if straightforward, up to 200-300 words if nuanced
- Be ready to go back and clarify if something doesn't make sense

## After the Design

**Documentation:**
- Write the validated design to `docs/plans/YYYY-MM-DD-<topic>-design.md`
- Use elements-of-style:writing-clearly-and-concisely skill if available
- Commit the design document to git

**Implementation:**
- Invoke the writing-plans skill to create a detailed implementation plan
- Do NOT invoke any other skill. writing-plans is the next step.

## Key Principles

- **Batch related questions** - Group related questions together to reduce round-trips
- **Plain text questions** - Ask in conversation, not via the AskUserQuestion tool
- **YAGNI ruthlessly** - Remove unnecessary features from all designs
- **Explore alternatives** - Always propose 2-3 approaches before settling
- **Two-phase validation** - Phase 1 (architecture + components), then Phase 2 (data flow, error handling, testing)
- **Be flexible** - Go back and clarify when something doesn't make sense

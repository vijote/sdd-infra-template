# SpecKit Technical Optimization Guide for LLM Agents

## Overview & Goal
This specification modifies the standard **SpecKit** (Spec-Driven Development / SDD) workflow to optimize for highly technical developer workflows, reduce context token usage, and prevent reasoning degradation in non-frontier LLMs (such as GLM-4.6, local LLMs, or mid-tier code assistants).

By stripping non-technical narratives (user personas, user stories, marketing explanations) and replacing them with **strict technical contracts, interface types, and micro-DAG tasks**, context efficiency is maximized across isolated agent sessions.

---

## 1. SpecKit Constitution Directive (`.specify/constitution.md`)

Replace or amend the `.specify/constitution.md` file in your repository with these strict directives:

```markdown
# System & LLM Execution Directives

## Core Principles
1. Zero Narrative Policy: Skip all introductory text, user personas, marketing justifications, and high-level conversational explanations. Go directly to technical contracts.
2. Architecture First: Use explicit engineering jargon, precise file paths, class/interface names, and architectural layer labels (e.g., NestJS Controller, TypeORM Entity, SvelteKit Server Load).
3. Payload & Token Efficiency: Keep spec and plan artifacts strictly below 200 lines. Use compact markdown tables, bullet points, and code blocks.
4. Granular Dependency Tree (DAG): Write `tasks.md` as an acyclic dependency tree where every task corresponds to a 1:1 file edit or unit test creation.

## Output Constraints
- Never explain framework fundamentals or standard language syntax.
- All acceptance criteria MUST be machine-verifiable (e.g., HTTP status code, unit test pass, type check completion).
- All API changes MUST define exact TypeScript/Zod/DTO input and output interfaces.
```

---

## 2. Optimized Templates (`.specify/templates/`)

### 2.1 Specification Template (`.specify/templates/spec.md`)
```markdown
# Spec: {FEATURE_NAME}

## 1. Technical Scope & Data Contracts
- Domain / Entity Scope: {TARGET_ENTITIES}
- API / Interface Contract:
  ```ts
  // Input DTO / Request Type
  export interface {FEATURE}Request {
    // fields
  }

  // Output DTO / Response Type
  export interface {FEATURE}Response {
    // fields
  }
  ```
- Database / Schema Migrations: {SCHEMA_CHANGES_SUMMARY}

## 2. Technical Acceptance Criteria
- [ ] Criteria 1 (e.g., Schema validation failure returns HTTP 400 with strict DTO format)
- [ ] Criteria 2 (e.g., Unit test coverage for edge cases: invalid input, missing auth)
- [ ] Criteria 3 (e.g., Query execution time within target bounds)
```

### 2.2 Plan Template (`.specify/templates/plan.md`)
```markdown
# Architecture Delta: {FEATURE_NAME}

## 1. Touch Points & File Impact Matrix
| File Path | Operation (Create/Modify/Delete) | Purpose / Exports |
| :--- | :--- | :--- |
| `src/modules/...` | Create | DTOs & Interfaces |
| `src/services/...` | Modify | Service logic implementation |

## 2. Architectural Boundary
- Affected Modules: {MODULE_NAMES}
- Shared Dependencies / Imports: {EXTERNAL_LIBS_OR_INTERNAL_UTILS}
```

### 2.3 Task Template (`.specify/templates/tasks.md`)
```markdown
# Execution Graph (DAG): {FEATURE_NAME}

## Stage 1: Contracts & Types
- [ ] Task 1.1: Create interface DTO in `src/types/{feature}.ts`

## Stage 2: Core Logic & Services
- [ ] Task 2.1: Implement logic in `src/services/{feature}.service.ts` (Depends on 1.1)

## Stage 3: Verification & Tests
- [ ] Task 3.1: Add unit tests in `src/services/{feature}.service.spec.ts` (Depends on 2.1)
```

---

## 3. Session Isolation Protocol for LLM Execution

When starting a new session with an AI agent (such as GLM-4.6 or local models), **do not feed the entire repo history or the full `tasks.md` file**. Use this minimal context payload:

### Prompt Payload Structure per Session:
```text
[CONTEXT]
1. Directive: .specify/constitution.md
2. Active Single Task:
   - File Target: `src/path/to/target.file.ts`
   - Goal: [Task description from tasks.md]
   - Data Contract / Interface: [Type snippet or DTO]
3. Instruction:
   - Output ONLY the implementation code and associated unit test.
   - Do not generate explanations or summary text outside code blocks.
```

---

## 4. Summary of Key Shift

| Standard SpecKit (Product-Centric) | Technical Dev-Direct (Agent-Optimized) |
| :--- | :--- |
| **Specify (`spec.md`)**: User stories, personas, feature descriptions. | **Tech Spec**: DTOs, Zod/TypeScript schemas, API signatures, status codes. |
| **Plan (`plan.md`)**: High-level system descriptions & explanations. | **Architecture Delta**: File touch matrix & modified boundaries only. |
| **Tasks (`tasks.md`)**: Feature-level task lists. | **Micro-DAG**: Atomic tasks mapped 1:1 to single files/functions. |
| **Session Context**: Whole history + full spec. | **Session Context**: Single active task + target file + interface contract. |

# Implementation Blueprint Template

Use this template when generating final implementation plans. Adapt sections as needed based on the feature complexity.

## Full Blueprint Structure

```markdown
# Implementation Blueprint: [Feature Name]

## Executive Summary
[2-3 sentences: What we're building, why it's SOTA, expected impact]

## SOTA Differentiators
What makes this implementation special:
1. [Differentiator 1]: [How it improves UX/performance/delight]
2. [Differentiator 2]: [How it improves UX/performance/delight]
3. [Differentiator 3]: [How it improves UX/performance/delight]

---

## Architecture Decision Records

### ADR-1: [Key Decision Title]
**Status**: Proposed | Accepted | Deprecated | Superseded

**Context**: 
[What problem or situation prompted this decision?]

**Decision**: 
[What we decided to do]

**Consequences**:
- ✅ [Positive outcome]
- ✅ [Positive outcome]
- ⚠️ [Tradeoff accepted]

### ADR-2: [Next Key Decision]
[Repeat structure...]

---

## Technical Specification

### Data Models

```typescript
// Core entities with relationships
interface [EntityName] {
  id: string;
  // fields with types and descriptions
  createdAt: Date;
  updatedAt: Date;
}
```

**Relationships:**
- [Entity A] → [Entity B]: [relationship type and cardinality]

### API Contracts

**Endpoints:**

```
POST /api/[resource]
Request:
{
  "field": "type"
}

Response (201):
{
  "id": "string",
  "field": "type"
}

Error responses:
- 400: Validation error
- 401: Unauthorized
- 409: Conflict
```

### Component Architecture

```
[ComponentTree]
├── [ParentComponent]
│   ├── [ChildA] - [responsibility]
│   ├── [ChildB] - [responsibility]
│   └── [ChildC]
│       └── [GrandChild] - [responsibility]
```

**Component Responsibilities:**
- `[ComponentName]`: [What it owns, what state it manages]

### State Management

**Client State:**
- [State slice]: [Purpose, update triggers]

**Server State:**
- [Data type]: [Caching strategy, invalidation rules]

**Sync Strategy:**
- [How client and server state synchronize]

---

## Implementation Phases

### Phase 1: Foundation (Week 1)
**Goal**: [What this phase establishes]

- [ ] **Task 1.1**: [Specific task]
  - Acceptance: [How we know it's done]
  - Files: `path/to/file.ts`
  
- [ ] **Task 1.2**: [Specific task]
  - Acceptance: [How we know it's done]
  - Dependencies: Task 1.1

### Phase 2: Core Features (Week 2-3)
**Goal**: [What this phase delivers]

- [ ] **Task 2.1**: [Specific task]
  - Acceptance: [Criteria]
  - SOTA element: [What makes this special]

### Phase 3: Polish & Delight (Week 4)
**Goal**: [User experience enhancements]

- [ ] **Task 3.1**: [UX enhancement]
  - User benefit: [What users will notice]

- [ ] **Task 3.2**: [Performance optimization]
  - Target metric: [Specific improvement goal]

---

## Testing Strategy

**Unit Tests:**
- [Component/Function]: [Key scenarios to test]

**Integration Tests:**
- [Flow/Feature]: [End-to-end scenarios]

**Performance Tests:**
- [Metric]: [Baseline and target]

---

## Risk Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk 1] | Medium | High | [How to address] |
| [Risk 2] | Low | Medium | [Escape hatch] |

---

## Future Extensibility

This design enables:
- [ ] [Future feature 1]: [How current design supports it]
- [ ] [Future feature 2]: [What foundation we're laying]

---

## Handoff Notes for AI Coding Tools

**Recommended implementation order:**
1. [File/component to build first]
2. [Next file/component]
3. [Continue...]

**Context the AI coder needs:**
- [Key architectural decisions to maintain]
- [Patterns to follow consistently]
- [Common pitfalls to avoid]

**Quality gates before shipping:**
- [ ] [Check 1]
- [ ] [Check 2]
```

## Lightweight Blueprint (for smaller features)

For features that don't warrant full ADRs:

```markdown
# [Feature Name] Implementation Plan

## What We're Building
[1-2 sentences]

## SOTA Element
[What makes this better than a standard implementation]

## Technical Approach
[Key technical decisions in 3-5 bullets]

## Tasks
- [ ] [Task 1] - [Acceptance criteria]
- [ ] [Task 2] - [Acceptance criteria]
- [ ] [Task 3] - [Acceptance criteria]

## AI Coder Notes
- Start with: [file/component]
- Watch out for: [common pitfall]
```

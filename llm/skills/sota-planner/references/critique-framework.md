# Critique & Elevation Framework

Questions and frameworks for analyzing user ideas and identifying opportunities for SOTA improvements.

## The 10x Question Framework

For each aspect of the feature, ask: "How could this be 10x better?"

### Speed
- Can we make it feel instant? (<100ms perceived)
- Can we eliminate a round-trip?
- Can we do work before the user asks?

### Simplicity
- Can we reduce clicks/steps by half?
- Can we auto-detect what the user wants?
- Can we eliminate a form field?

### Delight
- What would make the user smile?
- What would they screenshot to share?
- What would make them say "how did they know?"

### Power
- What would expert users wish for?
- What manual work could we automate?
- What data could we surface proactively?

---

## Bottleneck Analysis

### Technical Bottlenecks
Questions to ask:
1. Where does this make a network request? Can it be eliminated/prefetched?
2. What's the slowest operation? Can it be async/background?
3. What happens at 10x current scale? 100x?
4. What's the cold start experience vs warm?

Red flags:
- Sequential API calls that could be parallel
- Large payloads for small UI updates
- No caching strategy mentioned
- Single point of failure

### UX Bottlenecks
Questions to ask:
1. How many clicks to complete the core task?
2. What decisions does the user have to make?
3. Where does the user have to wait?
4. What errors could they encounter?

Red flags:
- Form with >5 fields
- Modal inside modal
- "Are you sure?" confirmations
- No undo, only confirm dialogs

### Business Bottlenecks
Questions to ask:
1. What's the conversion funnel? Where do users drop?
2. What keeps users from starting?
3. What keeps power users from upgrading?
4. What generates support tickets?

---

## Gap Identification Matrix

| Aspect | Current State | Industry SOTA | Gap | Opportunity |
|--------|---------------|---------------|-----|-------------|
| Speed | | | | |
| UX Flow | | | | |
| Data Handling | | | | |
| Error States | | | | |
| Edge Cases | | | | |
| Scalability | | | | |
| Accessibility | | | | |
| Mobile | | | | |

Fill this matrix during critique phase to systematically identify improvements.

---

## Elevation Prompts

### For "Standard CRUD" Features
- What if we added real-time collaboration?
- What if it worked offline-first?
- What if we predicted what they'd create next?
- What if we auto-populated from their history?

### For "Dashboard/Analytics" Features
- What if insights surfaced proactively?
- What if we explained "why" not just "what"?
- What if they could ask questions in natural language?
- What if anomalies triggered alerts automatically?

### For "Form/Input" Features
- What if we auto-saved every keystroke?
- What if we validated inline, not on submit?
- What if we suggested content based on context?
- What if they could complete it via voice/mobile?

### For "List/Table" Features
- What if sorting/filtering was instant client-side?
- What if they could customize columns/views?
- What if bulk actions were keyboard-driven?
- What if we remembered their preferences?

### For "Search" Features
- What if typos auto-corrected?
- What if we understood natural language?
- What if recent/frequent items surfaced first?
- What if we showed results as they typed?

### For "Notification" Features
- What if we batched intelligently?
- What if users could snooze contextually?
- What if we learned their preferences over time?
- What if urgent items broke through?

---

## Constructive Critique Phrases

Instead of: "This won't scale"
Say: "What if we needed to handle 100x the load—how might we architect this differently?"

Instead of: "This UX is confusing"
Say: "I'm curious about the flow here—what if a user wanted to [edge case]?"

Instead of: "This is basic"
Say: "This is solid foundation. What if we added [SOTA element] to make it exceptional?"

Instead of: "You're missing [thing]"
Say: "Have you considered [thing]? I've seen [company] do something interesting with this..."

Instead of: "That's too complex"
Say: "I love the ambition. What's the simplest version that still delivers the 'wow'?"

---

## Priority Matrix for Improvements

When multiple improvements are possible, prioritize:

```
                    HIGH IMPACT
                         │
      QUICK WINS    │    STRATEGIC
    (Do immediately)│   (Plan for these)
                    │
LOW ────────────────┼──────────────── HIGH
EFFORT              │                 EFFORT
                    │
       FILLERS      │    AVOID
    (Only if easy)  │   (Complexity trap)
                    │
                    LOW IMPACT
```

Always identify at least one Quick Win for momentum.

---

## Socratic Question Bank

### Understanding the Problem
- "What problem does this solve for the user?"
- "How do users solve this today?"
- "What's the cost of not solving this?"

### Challenging Assumptions
- "What if users wanted [opposite of assumption]?"
- "How do you know users will [expected behavior]?"
- "What's the riskiest assumption here?"

### Exploring Alternatives
- "What would [competitor] do here?"
- "What if we approached this from [different angle]?"
- "What's the most unconventional solution you can imagine?"

### Testing Feasibility
- "What's the hardest part to build?"
- "What could go wrong?"
- "What would we do if [dependency] failed?"

### Defining Success
- "How will we know this succeeded?"
- "What metric would tell us to kill this?"
- "What does 'good enough' look like vs 'exceptional'?"

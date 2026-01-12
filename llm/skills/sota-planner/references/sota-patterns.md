# SOTA Patterns Library

Reference patterns from industry-leading products. Use these as inspiration when elevating user ideas.

## UX Delight Patterns

### Optimistic Updates (Linear, Notion)
**Standard**: Wait for server response before updating UI
**SOTA**: Update UI immediately, reconcile in background
```
User action → Instant UI update → Background sync → Silent reconciliation
```
**User benefit**: Feels instant, no loading spinners
**Implementation notes**: Requires conflict resolution strategy, rollback capability

### Command Palette (Raycast, Linear, Vercel)
**Standard**: Navigate through menus
**SOTA**: Cmd+K opens instant search for any action
```
Features:
- Fuzzy search across all actions
- Recent commands
- Keyboard-first navigation
- Context-aware suggestions
```
**User benefit**: Power users become 10x faster
**Implementation notes**: Index all actions, weight by recency/frequency

### Skeleton States (Facebook, Stripe)
**Standard**: Loading spinner
**SOTA**: Content-shaped placeholders that pulse
```
Instead of: [Spinner]
Show: [████████ ████]
       [████ ████████]
```
**User benefit**: Perceived performance improves, less jarring
**Implementation notes**: Match actual content layout exactly

### Inline Editing (Notion, Airtable)
**Standard**: Edit in modal/separate page
**SOTA**: Double-click to edit in place
**User benefit**: Reduced context switching
**Implementation notes**: Escape to cancel, Enter to save, blur to save

---

## Performance Patterns

### Edge Computing (Vercel, Cloudflare)
**Standard**: Single origin server
**SOTA**: Compute at edge closest to user
```
Latency: 200ms → 20ms for global users
Use cases: Auth checks, redirects, personalization
```
**Implementation notes**: Workers/Edge Functions, careful with database access

### Streaming Responses (ChatGPT, Vercel AI SDK)
**Standard**: Wait for complete response
**SOTA**: Stream partial results as they're ready
```
User sees: [Typing...] → [Partial response] → [Complete response]
```
**User benefit**: Perceived instant response, can read while generating
**Implementation notes**: Server-Sent Events or WebSocket, chunk boundaries

### Prefetching (Next.js, Remix)
**Standard**: Fetch on navigation
**SOTA**: Fetch on hover/viewport intersection
```
Link hover → Prefetch data → Navigation instant
```
**User benefit**: Navigations feel instant
**Implementation notes**: Balance prefetch volume with bandwidth

### Virtual Scrolling (Discord, Slack)
**Standard**: Render all items
**SOTA**: Only render visible items + buffer
```
10,000 items → Only ~50 DOM nodes at any time
```
**User benefit**: Smooth scrolling regardless of list size
**Implementation notes**: Fixed height rows simplify, variable height requires measurement

---

## Data Patterns

### Real-time Sync (Figma, Notion)
**Standard**: Refresh to see changes
**SOTA**: Live updates via WebSocket/CRDT
```
Levels of sophistication:
1. Presence (who's viewing)
2. Live cursors
3. Collaborative editing
```
**Implementation notes**: CRDTs for conflict-free, OT for text-heavy

### Local-First (Linear, Obsidian)
**Standard**: Server is source of truth
**SOTA**: Local database syncs with server
```
Benefits:
- Works offline
- Instant interactions
- Reduced server load
```
**Implementation notes**: SQLite in browser, sync engine complexity

### Incremental Static Regeneration (Next.js)
**Standard**: Build all pages at build time OR fetch at request time
**SOTA**: Build popular pages, generate rest on-demand, revalidate stale
```
First request: Generate and cache
Subsequent: Serve cache, revalidate in background
```
**User benefit**: Fast loads + fresh content
**Implementation notes**: Define revalidation strategy per-route

---

## Architecture Patterns

### Feature Flags (LaunchDarkly pattern)
**Standard**: Deploy = release
**SOTA**: Decouple deployment from release
```
Benefits:
- Gradual rollout (1% → 10% → 100%)
- Kill switch for broken features
- A/B testing built-in
```
**Implementation notes**: Client-side SDK, analytics integration

### Event Sourcing (Banking, Audit-heavy)
**Standard**: Store current state
**SOTA**: Store all events, derive state
```
Benefits:
- Complete audit trail
- Time-travel debugging
- Easy undo/redo
```
**Implementation notes**: Event schema versioning, snapshotting for performance

### Plugin Architecture (VSCode, Figma)
**Standard**: Monolithic, all features built-in
**SOTA**: Core + extensible via plugins
```
Core: Essential functionality
Plugins: Community/custom extensions
API: Well-defined extension points
```
**Implementation notes**: Sandboxing, API versioning, marketplace

---

## AI Integration Patterns

### Contextual AI Assist (GitHub Copilot, Notion AI)
**Standard**: AI in separate interface
**SOTA**: AI suggestions inline where user is working
```
User types → AI suggests completion → Tab to accept
```
**Implementation notes**: Debounce requests, cache common patterns

### AI-Powered Search (Algolia, Perplexity)
**Standard**: Keyword matching
**SOTA**: Semantic search + natural language queries
```
"Show me bugs from last week" → Filters + results
```
**Implementation notes**: Embeddings, query parsing, hybrid search

### Progressive AI Enhancement
**Standard**: AI or nothing
**SOTA**: Graceful degradation when AI unavailable
```
Best: AI-enhanced response
Good: Rule-based fallback
Minimum: Manual workflow still works
```
**Implementation notes**: Feature detection, graceful timeout handling

---

## Security Patterns

### Zero Trust Architecture
**Standard**: Trust internal network
**SOTA**: Verify every request regardless of source
```
Every request: Auth + Authorization + Rate limit + Audit
```
**Implementation notes**: Short-lived tokens, principle of least privilege

### Progressive Authentication
**Standard**: Full auth upfront
**SOTA**: Start anonymous, auth when needed
```
Browse: Anonymous
Save: Email only
Purchase: Full auth + 2FA
```
**User benefit**: Lower friction to start
**Implementation notes**: Session elevation, clear UX for auth prompts

---

## Quick Reference: SOTA Upgrades

| Standard | SOTA | Key Tech |
|----------|------|----------|
| Loading spinner | Skeleton + optimistic | React Suspense |
| Full page reload | SPA with prefetch | Next.js/Remix |
| REST polling | WebSocket/SSE | Socket.io/Ably |
| Server rendering | Edge + Streaming | Vercel Edge |
| Manual save | Auto-save + sync | Debounce + CRDT |
| Search box | Command palette | cmdk/kbar |
| Modal forms | Inline editing | contenteditable |
| Pagination | Virtual scroll | TanStack Virtual |

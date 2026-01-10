## 🚨 CRITICAL PROTOCOL: READ BEFORE WRITING CODE

You are acting as a Senior Frontend Architect. Your goal is to balance **Developer Experience (DX)** with **User Performance (UX)**.

**The Golden Rule:** Use the lightest tool capable of solving the problem.

---

## 🛠️ The "SOTA" Tech Stack

| Domain            | Tool                 | Usage Rule                                                   |
| ----------------- | -------------------- | ------------------------------------------------------------ |
| **Framework**     | **Astro 5.0+**       | The "Shell". Handles Routing, SEO, and Layouts.              |
| **Server Islands** | **server:defer/stream** | Dynamic server-rendered content (0KB JS). User Avatar, Cart Count. |
| **Interactivity** | **React 19**         | **ONLY** for complex state (Carousels, Forms, Dashboards).   |
| **Simple Logic**  | **Vanilla TS**       | For toggles, menus, and intersection observers (< 50 lines). |
| **State**         | **Nanostores**       | For sharing state between Islands (e.g., Cart, Auth).        |
| **Data Mutations** | **Astro Actions**    | Type-safe backend functions for forms. No manual API endpoints. |
| **Environment**   | **astro:env**         | Type-safe environment variables with validation.            |
| **Page Motion**   | **View Transitions** | Native Astro/Browser API for page navigation animations.     |
| **Comp. Motion**  | **Framer Motion**    | Complex component gestures/physics inside React Islands.     |
| **Styling**       | **Tailwind v4**      | Utility-first. Use `cn()` for class merging.                 |

---

## 📐 Architecture Patterns

### 1. The "Interactivity Tier" Decision Matrix (Strict)

Before choosing `.tsx` or `.astro`, classify the feature:

| Tier       | Complexity  | Solution              | Example                                                  |
| ---------- | ----------- | --------------------- | -------------------------------------------------------- |
| **Tier 1** | **Static**  | `.astro` (HTML/CSS)   | Footer, Hero Text, Grid Layout                           |
| **Tier 1.5** | **Dynamic (Server)** | `.astro` + `server:defer` | User Avatar, Cart Count, Personalized Greeting |
| **Tier 2** | **Trivial** | `.astro` + `<script>` | Dark Mode Toggle, Mobile Menu, simple Scroll-to-Top      |
| **Tier 3** | **Complex** | `.tsx` (React Island) | Infinite Testimonial Slider, Multi-step Form, Search Bar |

**⚠️ Rules for Tier 2 (Vanilla JS):**

- Must be **TypeScript** (use JSDoc or strict casting if inside `.astro`).
- Must check `if (!element) return;`.
- Must handle `astro:page-load` for View Transitions compatibility.
- **Limit:** If logic exceeds ~50 lines, refactor to **Tier 3 (React)**.

### 2. State Management (Nanostores)

Do **NOT** use React Context or massive prop drilling across islands. Use **Nanostores**.

- **Scenario:** A "Buy" button in a React Island updates a "Cart Count" in the Astro Header.
- **Pattern:**

```typescript
// src/stores/cart.ts
import { atom } from 'nanostores';
export const isCartOpen = atom(false);
```

_Use it in React:_ `const $isOpen = useStore(isCartOpen);`
_Use it in Astro Script:_ `isCartOpen.subscribe(...)`

### 3. Animations Strategy

- **Page Navigation:** Use Astro's `<ViewTransitions />`. Do not use Framer Motion for full-page transitions.
- **Complex UI (Tier 3):** Use **Framer Motion** for "layout", "drag", or "enter/exit" animations _inside_ React components.

### 4. Data & Logic (Astro 5)

- **Mutations:** Use **Astro Actions** (`src/actions/`) for all form submissions and backend logic. Do not write manual API endpoints.
  - **Example:** Contact form, login, signup, newsletter subscription.
  - **Pattern:**
    ```typescript
    // src/actions/contact.ts
    import { defineAction } from 'astro:actions';
    export const contact = defineAction({
      input: z.object({ email: z.string().email() }),
      handler: async ({ email }) => { /* ... */ }
    });
    ```

- **Environment Variables:** Use `astro:env` for type-safe environment variables.
  - ❌ `process.env.API_KEY` or `import.meta.env.API_KEY`
  - ✅ `import { API_KEY } from 'astro:env/server'`
  - **Configuration:** Define schema in `astro.config.mjs` `env` block.
  - **Validation:** Astro validates at build time and provides autocomplete.

---

## ⚡ Workflow: The Component-Driven Loop

### Step 1: Define the Interface (The Contract)

```typescript
// src/types/testimonials.ts
export interface Testimonial {
  id: string;
  quote: string;
  author: string;
}
```

### Step 2: Choose the Tier

- _Task:_ "Create a testimonial slider with infinite loop and drag support."
- _Decision:_ **Tier 3 (Complex)** -> Use React + Framer Motion.

### Step 3: Implementation (React Island)

```tsx
// src/components/islands/TestimonialSlider.tsx
import { motion } from 'framer-motion';
// ...implementation...
```

### Step 4: Integration (Astro Page)

```astro
---
// src/pages/index.astro
import { TestimonialSlider } from '../components/islands/TestimonialSlider';
---

<TestimonialSlider client:visible />
```

---

## 🛑 Quality Control Checks

Before outputting code, verify:

1. [ ] **Hydration Check:** Did I use `client:load` on a static footer? (Bad). Did I use it on the critical hero LCP element? (Bad - use CSS/Server Render where possible).
2. [ ] **Cleanup:** If writing Tier 2 scripts, did I remove event listeners on `astro:before-swap` or use specific cleanup logic?
3. [ ] **Type Safety:** No `any`. No `// @ts-nocheck`.
4. [ ] **Styles:** Used `cn()` for conditional classes. No template literal concatenation.
5. [ ] **Conflict Check:** Does this contradict `astro.mdc`? **This file (`AGENTS.md`) is the Source of Truth.**

---

## 💡 Example: Tier 2 vs Tier 3

**Tier 2 (Valid Vanilla):**

```html
<button id="menu-btn" aria-label="Toggle Menu">Menu</button>
<script>
  document.addEventListener('astro:page-load', () => {
    const btn = document.getElementById('menu-btn');
    btn?.addEventListener('click', () => document.body.classList.toggle('menu-open'));
  });
</script>
```

**Tier 3 (Valid React):**

```tsx
// SearchBar.tsx
export const SearchBar = () => {
  const [query, setQuery] = useState('');
  // Complex filtering logic, API calls, loading states...
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
};
```


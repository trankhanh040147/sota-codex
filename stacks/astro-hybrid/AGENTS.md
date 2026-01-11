## 🚨 CRITICAL PROTOCOL: READ BEFORE WRITING CODE

You are acting as a Senior Frontend Architect. Your goal is to balance **Developer Experience (DX)** with **User Performance (UX)**.

**The Golden Rule:** Use the lightest tool capable of solving the problem.

---

## 🛠️ The "SOTA" Tech Stack

| Domain             | Tool                    | Usage Rule                                                                                                   |
| ------------------ | ----------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Framework**      | **Astro 5.0+**          | The "Shell". Handles Routing, SEO, and Layouts.                                                              |
| **Server Islands** | **server:defer/stream** | Dynamic server-rendered content (0KB JS). User Avatar, Cart Count. Use `slot="fallback"` for loading states. |
| **Interactivity**  | **React 19**            | **ONLY** for complex state (Carousels, Forms, Dashboards).                                                   |
| **Simple Logic**   | **Vanilla TS**          | For toggles, menus, and intersection observers (< 50 lines).                                                 |
| **State**          | **Nanostores**          | For sharing state between Islands (e.g., Cart, Auth).                                                        |
| **Data Mutations** | **Astro Actions**       | Type-safe backend functions for forms. No manual API endpoints.                                              |
| **Content**        | **Content Collections** | File-based content with type-safe queries (`getCollection`, `getEntry`).                                     |
| **Environment**    | **astro:env**           | Type-safe environment variables with validation.                                                             |
| **Page Motion**    | **View Transitions**    | Native Astro/Browser API for page navigation animations. Use `<ClientRouter />` (Astro 5).                   |
| **Comp. Motion**   | **Framer Motion**       | Complex component gestures/physics inside React Islands.                                                     |
| **Styling**        | **Tailwind v4**         | Utility-first. Use `cn()` for class merging.                                                                 |

---

## 📐 Architecture Patterns

### 1. The "Interactivity Tier" Decision Matrix (Strict)

Before choosing `.tsx` or `.astro`, classify the feature:

| Tier         | Complexity           | Solution                  | Example                                                                                   |
| ------------ | -------------------- | ------------------------- | ----------------------------------------------------------------------------------------- |
| **Tier 1**   | **Static**           | `.astro` (HTML/CSS)       | Footer, Hero Text, Grid Layout                                                            |
| **Tier 1.5** | **Dynamic (Server)** | `.astro` + `server:defer` | User Avatar, Cart Count, Personalized Greeting. Use `slot="fallback"` for loading states. |
| **Tier 2**   | **Trivial**          | `.astro` + `<script>`     | Dark Mode Toggle, Mobile Menu, simple Scroll-to-Top                                       |
| **Tier 3**   | **Complex**          | `.tsx` (React Island)     | Infinite Testimonial Slider, Multi-step Form, Search Bar                                  |

**⚠️ Critical: React Hooks and SSR in Astro**

React hooks (e.g., `useState`, `useEffect`, `useSyncExternalStore`) **cannot be called during SSR** even with `client:visible` or `client:load` directives. Astro still evaluates component code during SSR to generate initial HTML, which causes "Invalid hook call" errors.

**Solution: Use `client:only="react"` for components with hooks**

- ❌ **`client:visible`** - Still evaluates during SSR → Hook errors
- ❌ **`client:load`** - Still evaluates during SSR → Hook errors
- ✅ **`client:only="react"`** - Completely skips SSR → Hooks only run on client

**When to use each directive:**

| Directive             | SSR?   | Use Case                                                    |
| --------------------- | ------ | ----------------------------------------------------------- |
| `client:only="react"` | ❌ No  | Components with React hooks, complex state, or custom hooks |
| `client:visible`      | ✅ Yes | Components without hooks that benefit from lazy loading     |
| `client:load`         | ✅ Yes | Critical above-the-fold components without hooks            |

**Example - Hook Error Fix:**

```astro
<!-- ❌ WRONG: Causes "Invalid hook call" error -->
<PricingCard client:visible plan={plan} />

<!-- ✅ CORRECT: Skips SSR, hooks only run on client -->
<PricingCard client:only="react" plan={plan} />
```

**Why `client:only` works:**

- Completely skips server-side rendering
- Component code is never evaluated during SSR
- React hooks only execute on client where React is fully initialized
- Trade-off: No initial HTML (component renders after hydration)

**⚠️ Rules for Tier 2 (Vanilla JS):**

- Must be **TypeScript** (use JSDoc or strict casting if inside `.astro`).
- Must check `if (!element) return;`.
- Must handle `astro:page-load` for View Transitions compatibility.
- Use `transition:persist` attribute to maintain state during view transitions (e.g., video players, form inputs).
- Use `transition:persist-props` on islands to preserve props across navigation (Astro 4.5+).
- **Limit:** If logic exceeds ~50 lines, refactor to **Tier 3 (React)**.

**View Transitions Events & Attributes:**

- `astro:page-load`: Fires after navigation completes. Use for re-initializing scripts.
- `astro:before-swap`: Fires before DOM swap. Use for preserving theme/state during transitions.
- `data-astro-rerun`: Attribute to force inline script re-execution on navigation.
- `data-astro-reload`: Attribute on links to force full-page navigation (disables client router).
- **Custom swap functions:** Use `swapFunctions` object (Astro 4.15+) for custom transition animations.

### 2. State Management (Nanostores)

Do **NOT** use React Context or massive prop drilling across islands. Use **Nanostores**.

- **Scenario:** A "Buy" button in a React Island updates a "Cart Count" in the Astro Header.
- **Pattern:**
  ript
  // src/stores/cart.ts
  import { atom } from 'nanostores';
  export const isCartOpen = atom(false);
  _Use it in React:_ `const $isOpen = useStore(isCartOpen);`
  _Use it in Astro Script:_ `isCartOpen.subscribe(...)`

### 3. Animations Strategy

- **Page Navigation:** Use Astro's `<ClientRouter />` (Astro 5 breaking change - replaces `<ViewTransitions />`). Do not use Framer Motion for full-page transitions.
- **Complex UI (Tier 3):** Use **Framer Motion** for "layout", "drag", or "enter/exit" animations _inside_ React components.

### 4. Data & Logic (Astro 5)

- **Mutations:** Use **Astro Actions** (`src/actions/`) for all form submissions and backend logic. Do not write manual API endpoints.
  - **Example:** Contact form, login, signup, newsletter subscription.
  - **Pattern:**cript
    // src/actions/contact.ts
    import { defineAction, ActionError, isInputError } from 'astro:actions';
    import { z } from 'zod';

    export const contact = defineAction({
    accept: 'form', // Required for form data handling
    input: z.object({ email: z.string().email() }),
    handler: async ({ email }, { locals }) => {
    // Security: Always validate auth in handler
    if (!locals.user) {
    throw new ActionError({ code: "UNAUTHORIZED" });
    }

        // Handle errors
        try {
          // ... logic ...
        } catch (error) {
          if (isInputError(error)) {
            throw error; // Re-throw validation errors
          }
          throw new ActionError({ code: "INTERNAL_ERROR" });
        }

    }
    });
    - **Error Handling:**
    - Use `ActionError({ code: "..." })` for structured errors.
    - Use `isInputError()` to check validation errors.
    - Use `.orThrow()` pattern for optional actions.

  - **Security:** Actions are public endpoints (`/_actions/*`). Always validate auth using `context.locals` in the handler.

- **Environment Variables:** Use `astro:env` for type-safe environment variables.
  - ❌ `process.env.API_KEY` or `import.meta.env.API_KEY`
  - ✅ `import { API_KEY } from 'astro:env/server'`
  - **Configuration:** Define schema in `astro.config.mjs` `env` block.
  - **Validation:** Astro validates at build time and provides autocomplete.

- **Server Islands Caching:** Server Islands support automatic caching via `Cache-Control` headers (GET requests only, props < 2KB). Use `ASTRO_KEY` environment variable for encryption. Use for frequently accessed server-rendered content.

### 5. Content Collections (Astro 5 Content Layer)

- **Configuration:** Define in `src/content.config.ts` using `defineCollection()`.
- **Loaders:**
  - Use `glob()` for file-based content (Markdown, MDX).
  - Use `file()` for JSON/YAML content.
- **Querying:**
  - Use `getCollection()` to query collections.
  - Use `getEntry()` to fetch a single entry.
  - Use `render(entry)` to render Markdown content.
- **Important:** `compiledContent()` is **async** in Astro 5 (breaking change).
- **Client Restriction:** Cannot use `astro:content` on client. Pass content via props to islands.

// src/content.config.ts
import { defineCollection, z } from 'astro:content';

export const blog = defineCollection({
loader: glob('./blog/\*_/_.md'),
schema: z.object({
title: z.string(),
date: z.date(),
}),
});

## // Usage in .astro file

import { getCollection } from 'astro:content';
const posts = await getCollection('blog');
const entry = await getEntry('blog', 'my-post');
const { Content } = await entry.render();

---

<Content />---

## ⚡ Workflow: The Component-Driven Loop

### Step 1: Define the Interface (The Contract)

t
// src/types/testimonials.ts
export interface Testimonial {
id: string;
quote: string;
author: string;
}### Step 2: Choose the Tier

- _Task:_ "Create a testimonial slider with infinite loop and drag support."
- _Decision:_ **Tier 3 (Complex)** -> Use React + Framer Motion.

### Step 3: Implementation (React Island)

// src/components/islands/TestimonialSlider.tsx
import { motion } from 'framer-motion';
// ...implementation...### Step 4: Integration (Astro Page)

---

// src/pages/index.astro
import { TestimonialSlider } from '../components/islands/TestimonialSlider';

---

<!-- Use client:only for components with hooks -->

<TestimonialSlider client:only="react" items={testimonials} />---

## 🛑 Quality Control Checks

Before outputting code, verify:

1. [ ] **Hydration Check:** Did I use `client:load` on a static footer? (Bad). Did I use it on the critical hero LCP element? (Bad - use CSS/Server Render where possible).
2. [ ] **React Hooks & SSR:** Did I use `client:only="react"` for components with React hooks? (Required to prevent "Invalid hook call" errors).
3. [ ] **Cleanup:** If writing Tier 2 scripts, did I remove event listeners on `astro:before-swap` or use specific cleanup logic?
4. [ ] **Type Safety:** No `any`. No `// @ts-nocheck`.
5. [ ] **Styles:** Used `cn()` for conditional classes. No template literal concatenation.
6. [ ] **View Transitions:** Did I use `<ClientRouter />` instead of `<ViewTransitions />`? (Astro 5 breaking change).
7. [ ] **View Transitions Attributes:** Did I use `data-astro-reload` when full-page navigation is needed? Did I use `data-astro-rerun` for inline script re-execution?
8. [ ] **Actions Security:** Did I validate auth in Action handlers using `context.locals`?
9. [ ] **Content Collections:** Did I await `compiledContent()` if using it? (Astro 5 async breaking change).
10. [ ] **Conflict Check:** Does this contradict `astro.mdc`? **This file (`AGENTS.md`) is the Source of Truth.**

---

## 💡 Example: Tier 2 vs Tier 3

**Tier 2 (Valid Vanilla):**

<button id="menu-btn" aria-label="Toggle Menu">Menu</button>

<script>
  document.addEventListener('astro:page-load', () => {
    const btn = document.getElementById('menu-btn');
    btn?.addEventListener('click', () => document.body.classList.toggle('menu-open'));
  });
</script>**Tier 3 (Valid React):**

// SearchBar.tsx
export const SearchBar = () => {
const [query, setQuery] = useState('');
// Complex filtering logic, API calls, loading states...
return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
};

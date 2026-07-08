# SvelteKit Development Principles

Reference this document at the start of every SvelteKit task. Target Svelte 5 exclusively — do not write Svelte 4 patterns (`export let`, `on:event`, `$:` reactive declarations, `<slot>`).

---

## Svelte 5 Runes

Runes are compiler signals, not runtime imports. Never import them.

| Rune | Purpose |
|---|---|
| `$state(value)` | Reactive state; objects/arrays are deeply reactive |
| `$state.raw(value)` | Reactive reference without deep reactivity |
| `$state.snapshot(val)` | Plain non-reactive clone for passing to non-Svelte code |
| `$derived(expr)` | Computed value; lazy and cached |
| `$derived.by(() => {...})` | Multi-line derived computation |
| `$effect(() => {...})` | Side effects after DOM update; return cleanup fn |
| `$effect.pre(() => {...})` | Runs before DOM update (for measuring) |
| `$props()` | Declare component props with destructuring |
| `$bindable(default)` | Opt a prop into two-way binding |
| `$inspect(val)` | Dev-only: log value changes to console |

**Rules:**
- Use `$derived` to compute values — never `$effect` to set state from state.
- Never mutate state inside `$effect` without a condition; causes infinite loops.
- `$effect` does not run during SSR.

### Props

```svelte
<script lang="ts">
  interface Props {
    title: string;
    count?: number;
    onchange?: (value: string) => void;
    children?: import('svelte').Snippet;
  }
  let { title, count = 0, onchange, children }: Props = $props();
</script>
```

- Event callbacks are plain function props (`onchange`, `onclick`) — not `createEventDispatcher`.
- Use `...rest` to forward attributes: `let { label, ...rest }: Props = $props()`.
- Only use `$bindable` when two-way binding is the explicit contract; prefer callbacks otherwise.

### Snippets

```svelte
<!-- Definition -->
{#snippet footer(label: string)}
  <p>{label}</p>
{/snippet}

<!-- Render -->
{@render footer('Done')}
```

- Type snippets as `Snippet` or `Snippet<[ArgType]>` in `Props`.
- `children` is the default snippet for content between component tags.

---

## Routing Conventions

| File | Purpose |
|---|---|
| `+page.svelte` | Page component |
| `+page.ts` | Universal load (server + client) |
| `+page.server.ts` | Server-only load + form actions |
| `+layout.svelte` / `.ts` / `.server.ts` | Layout equivalents |
| `+server.ts` | HTTP API handlers |
| `+error.svelte` | Error boundary for this segment |

- Route groups `(group)/` organise routes without affecting the URL.
- Params: `[slug]`, optional `[[slug]]`, rest `[...path]`.

---

## Data Loading

**Universal load** (`+page.ts`) — runs on server and client; use for data not requiring secrets.

**Server load** (`+page.server.ts`) — server only; use for DB queries, cookies, secrets.

```typescript
// +page.server.ts
import type { PageServerLoad } from './$types';
import { error } from '@sveltejs/kit';

export const load: PageServerLoad = async ({ params, locals }) => {
  const post = await db.post.findUnique({ where: { slug: params.slug } });
  if (!post) throw error(404, 'Not found');
  return { post };
};
```

```svelte
<!-- +page.svelte -->
<script lang="ts">
  import type { PageData } from './$types';
  let { data }: { data: PageData } = $props();
</script>
```

- Always use generated types: `PageLoad`, `PageServerLoad`, `LayoutLoad`, `PageData`, `LayoutData`.
- Return nested promises from load to stream deferred data.
- Use `depends('app:key')` in load and `invalidate('app:key')` in components for fine-grained invalidation.
- Use `$app/state` (not `$app/stores`) for `page`, `navigating`, `updated`.

---

## Form Actions

```typescript
// +page.server.ts
import type { Actions } from './$types';
import { fail, redirect } from '@sveltejs/kit';

export const actions: Actions = {
  default: async ({ request }) => {
    const data = await request.formData();
    const email = data.get('email');
    if (typeof email !== 'string' || !email) return fail(422, { error: 'Required' });
    await db.user.create({ data: { email } });
    redirect(303, '/dashboard');
  },
};
```

```svelte
<script lang="ts">
  import { enhance } from '$app/forms';
  import type { ActionData } from './$types';
  let { form }: { form: ActionData } = $props();
</script>

<form method="POST" use:enhance>
  <input name="email" />
  {#if form?.error}<p role="alert">{form.error}</p>{/if}
  <button type="submit">Submit</button>
</form>
```

- Always add `use:enhance` — it prevents full-page reloads when JS is available.
- Use `fail()` (not `throw`) for validation errors; throw `error()` for unexpected failures.
- Validate with Zod on the server; return field errors via `fail()`.

---

## API Routes

```typescript
// +server.ts
import { json, error } from '@sveltejs/kit';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async ({ params }) => {
  const item = await db.find(params.id);
  if (!item) throw error(404);
  return json(item);
};
```

- Use `json()` helper — sets `Content-Type` automatically.
- Throw `error()` from `@sveltejs/kit`, not `new Error()`.

---

## Error Handling

| Helper | Use case |
|---|---|
| `throw error(status, message)` | Non-recoverable — renders `+error.svelte` |
| `redirect(303, url)` | Post-action redirect |
| `return fail(422, data)` | Form validation failure |

Global handler: `export const handleServerError: HandleServerError` in `hooks.server.ts`.

---

## Control Flow

Prefer early returns and defensive guards in load functions, actions, and API routes. Validate preconditions first; keep the happy path as the final unindented statement. Never write an `else` branch after `throw error()`, `return fail()`, or `redirect()`.

```typescript
// Preferred — guard at the top, happy path last
export const load: PageServerLoad = async ({ params, locals }) => {
  if (!locals.user) throw error(401, 'Unauthorized');
  const post = await db.post.findUnique({ where: { slug: params.slug } });
  if (!post) throw error(404, 'Not found');
  return { post };
};

// Avoid — unnecessary else after throw
export const load: PageServerLoad = async ({ params }) => {
  const post = await db.post.findUnique({ where: { slug: params.slug } });
  if (post) {
    return { post };
  } else {
    throw error(404, 'Not found');
  }
};
```

---

## TypeScript Patterns

```typescript
// Typing component generics
// <script lang="ts" generics="T">

// Utility type for a component's props
import type { ComponentProps } from 'svelte';
import Button from './Button.svelte';
type ButtonProps = ComponentProps<typeof Button>;
```

- Gate browser-only code: `import { browser } from '$app/environment'; if (browser) { ... }`.
- Never access `window`, `document`, or `localStorage` outside `$effect` or a `browser` guard.
- Use `$lib` alias for imports from `src/lib`; use `$lib/server/` for server-only modules.

---

## Testing

Use Vitest + `@testing-library/svelte` + `@testing-library/jest-dom`.

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import { sveltekit } from '@sveltejs/kit/vite';

export default defineConfig({
  plugins: [sveltekit()],
  test: { environment: 'jsdom', setupFiles: ['./src/test/setup.ts'] },
});
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
```

### Component tests

```typescript
import { render, screen, fireEvent } from '@testing-library/svelte';
import Counter from './Counter.svelte';

it('calls onchange when incremented', async () => {
  const onchange = vi.fn();
  render(Counter, { props: { count: 0, onchange } });
  await fireEvent.click(screen.getByRole('button', { name: /increment/i }));
  expect(onchange).toHaveBeenCalledWith(1);
});
```

### Load function tests

Test as plain async functions — no browser needed:

```typescript
it('throws 404 for unknown slug', async () => {
  vi.spyOn(db.post, 'findUnique').mockResolvedValue(null);
  await expect(load({ params: { slug: 'x' } } as any)).rejects.toMatchObject({ status: 404 });
});
```

### Form action tests

```typescript
it('returns 422 when email is missing', async () => {
  const request = new Request('http://localhost', { method: 'POST', body: new FormData() });
  const result = await actions.default({ request } as any);
  expect(result?.status).toBe(422);
});
```

**What to test:** component render/interaction, load function auth guards and not-found cases, action validation and success paths, pure helpers in `$lib`.

**What not to test:** internal reactivity wiring — test observable output via the DOM or return values.

---

## Self-Review Checklist

- [ ] No magic numbers or magic strings — all literals extracted into named constants
- [ ] No Svelte 4 patterns: no `export let`, `on:event`, `$:`, `<slot>`
- [ ] All props typed via `$props()` with an explicit `Props` interface
- [ ] `$derived` used for computed values — not `$effect`
- [ ] Load functions typed with generated `PageLoad` / `PageServerLoad`
- [ ] Page components receive `data` typed as `PageData` from `./$types`
- [ ] Forms use `use:enhance`; validation returns `fail()`, success uses `redirect()`
- [ ] `$app/state` used (not `$app/stores`)
- [ ] No browser globals outside `$effect` or `browser` guard
- [ ] Tests cover load functions, actions, and component behaviour

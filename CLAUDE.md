# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IT-Tools is a Vue 3 + TypeScript web application that provides a collection of handy online tools for developers. It's built with Vite and uses the Naive UI component library.

## Development Commands

All commands use `pnpm` (packageManager is pinned to pnpm@9.11.0):

```bash
# Install dependencies
pnpm install

# Start development server
pnpm dev

# Build for production
pnpm build

# Run unit tests (Vitest)
pnpm test

# Run unit tests with coverage
pnpm coverage

# Run e2e tests (Playwright)
pnpm test:e2e

# Run e2e tests against dev server (faster)
pnpm test:e2e:dev

# Lint (ESLint with @antfu config)
pnpm lint

# Type check (vue-tsc)
pnpm typecheck

# Create a new tool
pnpm run script:create:tool my-tool-name
```

## Architecture

### Tool Structure

Each tool is a self-contained feature in `src/tools/<tool-name>/` with these files:
- `index.ts` - Tool definition using `defineTool()`
- `<tool-name>.vue` - The Vue component
- `<tool-name>.service.ts` - Business logic (optional)
- `<tool-name>.service.test.ts` - Unit tests (optional)
- `<tool-name>.e2e.spec.ts` - E2E tests (optional)

Tools are registered in `src/tools/index.ts` and organized by category. The router (`src/router.ts`) dynamically generates routes from the tools array.

### Key Directories

- `src/tools/` - All tool implementations (~88 tools)
- `src/ui/` - Shared UI components (c-input-text, c-button, c-card, etc.)
- `src/composable/` - Vue composables (useCopy, useValidation, etc.)
- `src/utils/` - Utility functions
- `src/components/` - Vue components (ToolCard, FavoriteButton, etc.)
- `src/stores/` - Pinia stores
- `src/plugins/` - App plugins (i18n, naive-ui, plausible)
- `src/layouts/` - Page layouts (base, toolLayout)
- `locales/` - i18n translation files (en.yml, fr.yml, etc.)

### Auto-Imports

The following are auto-imported (see `vite.config.ts`):
- Vue (ref, computed, watch, etc.)
- Vue Router (useRoute, useRouter)
- VueUse (@vueuse/core)
- Vue I18n (useI18n)
- Naive UI (useDialog, useMessage, useNotification, useLoadingBar)
- Components (via unplugin-vue-components)

### Styling

- **UnoCSS** for atomic CSS with custom shortcuts (see `unocss.config.ts`)
- Common shortcuts: `pretty-scrollbar`, `bg-surface`, `bg-background`, `divider`
- **Less** for scoped component styles
- Naive UI theming via `src/themes.ts`

### Important Patterns

**Tool Definition** (`src/tools/tool.ts`):
```typescript
export const tool = defineTool({
  name: translate('tools.bcrypt.title'),  // Use i18n
  path: '/bcrypt',
  description: '',
  keywords: ['bcrypt', 'hash'],
  component: () => import('./bcrypt.vue'),  // Lazy load
  icon: LockSquare,  // From @vicons/tabler
  createdAt: new Date('2024-01-01'),  // Marks as "new" for 2 weeks
});
```

**Copy to Clipboard**:
Always use the local `useCopy` composable, not `@vueuse/core`'s `useClipboard`:
```typescript
import { useCopy } from '@/composable/copy';
const { copy } = useCopy({ source: computedValue, text: 'Copied!' });
```

**Input Components**:
Use the custom `c-input-text` component instead of Naive UI's n-input directly:
```vue
<c-input-text
  v-model:value="inputValue"
  label="Input label"
  placeholder="Enter text..."
  :validation-rules="[requiredStringRule]"
/>
```

**Card Components**:
Wrap tool sections in `c-card`:
```vue
<c-card title="Section Title">
  <!-- content -->
</c-card>
```

### i18n

Translations are in YAML files in `locales/`. Use the `translate()` helper from `@/plugins/i18n.plugin` for tool metadata. In components, use `useI18n()` and `{{ $t('key') }}`.

Key translation paths:
- `home.*` - Home page
- `about.*` - About page
- `tools.<tool-name>.*` - Individual tool translations

### Testing

Unit tests use Vitest with jsdom. Place tests next to the source files with `.test.ts` suffix.

E2E tests use Playwright. Place in tool directories with `.e2e.spec.ts` suffix or in `src/` root.

### ESLint Rules

- Curly braces required for all control statements
- Semicolons required
- No use of `@vueuse/core`'s `useClipboard` (use local `useCopy` instead)

### Environment Variables

Key env vars (see `src/config.ts`):
- `BASE_URL` - App base URL
- `VITE_TRACKER_ENABLED` - Enable Plausible analytics
- `VITE_VERCEL_GIT_COMMIT_SHA` - Commit SHA
- `PACKAGE_VERSION` - Set automatically during build

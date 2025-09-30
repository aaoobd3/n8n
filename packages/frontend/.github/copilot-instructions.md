# n8n Frontend Development Guide

## Architecture Overview

**Monorepo Structure**: This is part of n8n's pnpm workspace-based monorepo. The frontend consists of multiple interconnected packages in `packages/frontend/`:

- **`editor-ui/`** - Main Vue 3 application (workflow editor)
- **`@n8n/design-system/`** - Shared Vue component library with Storybook
- **`@n8n/stores/`** - Pinia stores for cross-package state management
- **`@n8n/composables/`** - Reusable Vue composition functions
- **`@n8n/i18n/`** - Internationalization system
- **`@n8n/chat/`** - Chat/assistant components
- **`@n8n/rest-api-client/`** - API client abstractions

## Key Development Patterns

### State Management
Uses **Pinia stores** with workspace-scoped packages. Store files follow `[feature].store.ts` naming:
- `ui.store.ts` - Global UI state, modals, sidebars
- `workflows.store.ts` - Workflow data and operations
- `ndv.store.ts` - Node Detail View state
- `canvas.store.ts` - Workflow canvas interactions

### Component Architecture
- **Design System**: All UI components extend `@n8n/design-system` base components (N8nButton, N8nInput, etc.)
- **Vue 3 + Composition API**: Uses `<script setup>` syntax consistently
- **Async Components**: Heavy views use `async () => import()` for code splitting

### CSS & Styling
Uses CSS custom properties system defined in `CLAUDE.md`. Key patterns:
```scss
// Use design tokens, not hardcoded values
color: var(--color-primary);
spacing: var(--spacing-s);
font-size: var(--font-size-m);
```

### Routing & Views
- **Route-based code splitting**: Views in `src/views/` use async imports
- **Nested layouts**: MainHeader/MainSidebar components wrap content
- **Route middleware**: RBAC permissions and auth guards in `router.ts`

## Development Commands

```bash
# Main development server (editor-ui)
cd editor-ui && pnpm dev

# Design system development
cd @n8n/design-system && pnpm storybook

# Run all frontend packages
pnpm dev

# Type checking
pnpm typecheck
pnpm typecheck:watch

# Testing
pnpm test         # Vitest unit tests
pnpm test:dev     # Watch mode
```

## Critical Integration Points

### API Communication
- **REST Client**: Uses `@n8n/rest-api-client` for backend calls
- **Push Connection**: Real-time updates via WebSocket (`pushConnection.store.ts`)
- **Type Safety**: Shared types from `@n8n/api-types` workspace package

### Node Detail View (NDV)
Central component for node editing. Key files:
- `NodeDetailsView.vue` - Main NDV container
- `ParameterInput*.vue` - Dynamic parameter rendering system
- `RunData.vue` - Execution result display

### Canvas System
- **Vue Flow**: Uses `@vue-flow/core` for workflow canvas
- **Canvas Store**: Handles node positioning, connections, selections
- **Real-time Collaboration**: Multi-user editing via `collaboration.store.ts`

## Build System Specifics

### Vite Configuration
- **Path Aliases**: `@` maps to `src/`, `@n8n/*` resolves to workspace sources
- **Legacy Support**: Browser compatibility via `@vitejs/plugin-legacy`
- **Asset Processing**: SVG loader, static file copying for icons/themes

### TypeScript
- **Strict Mode**: Full type checking enabled
- **Vue TSC**: Uses `vue-tsc` for Vue component type checking
- **Workspace Types**: Shared types across packages

## Common Gotchas

1. **CSS Variables**: Always use design tokens from `CLAUDE.md`, never hardcode colors/spacing
2. **Store Access**: Import stores via `@n8n/stores` for cross-package compatibility
3. **Component Registration**: Auto-imported via `unplugin-vue-components` - no manual imports needed for N8n* components
4. **Route Guards**: Check `middleware` array in route configs for auth/permission requirements
5. **Asset Loading**: Use Vite's asset pipeline - import images as modules, not string paths

## Testing Patterns

- **Vitest + Testing Library**: Component testing with Vue Testing Library
- **Store Testing**: Use `@pinia/testing` for isolated store tests
- **Mock Strategy**: Utilities in `__tests__/` directories for common mocks
- **E2E Context**: Separate test configuration for end-to-end scenarios

## Performance Considerations

- **Bundle Splitting**: Route-level splitting via async imports
- **Tree Shaking**: Lodash methods imported individually (`lodash/camelCase`)
- **Virtual Scrolling**: Large lists use `vue-virtual-scroller`
- **Lazy Loading**: Components and stores loaded on-demand where possible

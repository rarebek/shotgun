# agents.md

use the maximum amount of ultrathink. take all the time you need. it's much better if you do too much research and thinking than not enough.

## build commands
- **frontend dev**: `cd frontend && pnpm run dev`
- **frontend build**: `cd frontend && pnpm run build`
- **wails dev**: `wails dev` (hot reload)
- **wails build**: `wails build` (production)
- **install deps**: `cd frontend && pnpm install`

## testing
no test framework currently configured. add tests before implementing new features.

## code style

### go backend
- use `gofmt` for formatting
- pascalcase for exported types/functions
- proper error handling with error propagation
- context-based cancellation for async operations
- structured logging with runtime.log* functions

### vue frontend  
- composition api with `<script setup>`
- camelcase for variables, pascalcase for components
- kebab-case for css classes and html attributes
- reactive refs and computed properties
- event emission patterns for parent-child communication
- tailwind css for styling with css variables for theming

### general
- descriptive naming conventions
- responsive design required
- debounced operations for performance
- no comments unless explicitly requested
- follow existing patterns in codebase
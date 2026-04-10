# Project Guidelines

## Tech Stack & Environment
- **Framework:** Astro (v6.1.5)
- **Environment:** Node.js (>= 22.12.0)
- **Package Manager:** `pnpm`
- **Project Type:** Static Site / Server-Side Rendering (default static)

## Architecture & Directory Structure
- `src/pages/`: File-based routing. Each `.astro` file corresponds to a route.
- `src/layouts/`: Reusable layout components (e.g., global `<head>`, header/footer, `<slot />`).
- `src/components/`: Reusable UI components to keep pages clean.
- `src/assets/`: Processed assets (e.g., SVG, images) imported via ESM in components.
- `public/`: Unprocessed static assets served at the root `/`.

## Component Conventions
- Astro components use the standard three-part structure:
  1. **Component Script:** Inside `---` fences (server-side code, imports, data fetching).
  2. **Component Template:** HTML/JSX mix. Page content wrapper using `<Layout>` and `<slot />`.
  3. **Component Styles:** Scoped styles inside `<style>` tags at the bottom.

## Styling Approach
- **Scoped CSS:** Default approach. Use `<style>` inside `.astro` files; styles apply only to that component.
- **Global CSS:** Placed in layout components (like `src/layouts/Layout.astro`) or imported global stylesheets.

## Build and Test Commands
Use `pnpm` for all scripts:
- **Dev Server:** `pnpm run dev`
- **Build:** `pnpm run build`
- **Preview:** `pnpm run preview`
- **Install dependencies:** `pnpm install`

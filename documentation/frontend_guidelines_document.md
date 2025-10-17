# Frontend Guidelines Document

This document outlines the frontend architecture, design principles, and technologies used in the `apps/web` portion of the `vibecode-clone` monorepo. It is written in everyday language so anyone—technical or not—can understand how the frontend is set up and why certain choices were made.

## 1. Frontend Architecture

### 1.1 Stack Overview
- **Next.js (App Router)**: Provides the framework for file-based routing, server and client components, API routes, and built-in performance optimizations.
- **React & TypeScript**: Enables building reusable UI pieces with strong type safety, reducing bugs and improving developer confidence.
- **Tailwind CSS**: A utility-first CSS framework for rapid styling without writing custom CSS classes.
- **shadcn/ui**: A collection of prebuilt, accessible React components on top of Radix UI, ensuring a consistent look and feel.
- **next-themes**: Handles light/dark mode switching at runtime.
- **Better Auth**: Powers user sign-up, sign-in, and protected routes within Next.js API routes.

### 1.2 Monorepo & Workspace
- **PNPM Workspace**: `apps/web` lives alongside other packages (`apps/server`, `packages/ui`, etc.). Shared configs (TypeScript, ESLint, Prettier) sit at the root.
- **Modular Structure**: Common UI components will be extracted into `packages/ui`, making them available to both web and future mobile or desktop apps.

### 1.3 Supporting Scalability, Maintainability, and Performance
- **Scalability**: File-based routing and modular packages let us add features without disrupting existing code. Shared packages reduce duplication.
- **Maintainability**: TypeScript catches errors at compile time. Component isolation means you can update one piece without side effects.
- **Performance**: Next.js server components reduce client bundle size. Automatic code splitting and lazy loading keep initial load times low.

## 2. Design Principles

### 2.1 Key Principles
- **Usability**: Clear navigation, predictable behavior, and minimal steps to accomplish tasks.
- **Accessibility**: All components meet WCAG 2.1 guidelines—ARIA labels, keyboard navigation, focus states.
- **Responsiveness**: Layouts adapt gracefully from mobile to desktop using Tailwind’s responsive utilities.
- **Consistency**: Shared design tokens (colors, spacing, typography) enforce a unified experience.

### 2.2 Applying the Principles
- **Navigation**: The sidebar and header use consistent spacing and visual cues (active link highlighting).
- **Forms**: Input fields have clear labels, error messages, and support keyboard-only users.
- **Dark Mode**: next-themes ensures color contrast remains accessible in both light and dark.

## 3. Styling and Theming

### 3.1 Styling Approach
- **Utility-First CSS**: Tailwind CSS is our primary styling tool—no separate CSS or SCSS files. Classes like `p-4`, `bg-primary`, `flex` handle most layouts.
- **Component Tokens**: Custom design tokens (colors, font sizes) are defined in `tailwind.config.js`.

### 3.2 Theming
- **Light & Dark Modes**: next-themes toggles a `class` on the `<html>` element. Tailwind’s `dark:` modifier adjusts colors accordingly.
- **Extensible Tokens**: New theme values (e.g., high-contrast mode) can be added to the theme config.

### 3.3 Visual Style
- **Overall Style**: Modern, flat design with subtle glassmorphism on overlay panels (semi-transparent backgrounds with a slight blur).
- **Color Palette**:
  - Primary: `#4F46E5` (indigo)
  - Secondary: `#10B981` (emerald)
  - Accent: `#F59E0B` (amber)
  - Background Light: `#F9FAFB`
  - Background Dark: `#111827`
  - Surface Light: `#FFFFFF`
  - Surface Dark: `#1F2937`
  - Error: `#EF4444`
  - Success: `#22C55E`
  - Text Primary Light: `#111827`
  - Text Primary Dark: `#F3F4F6`

### 3.4 Typography
- **UI Font**: Inter, system-font stack (`-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif`).
- **Code Font**: JetBrains Mono for terminal and editor components.

## 4. Component Structure

### 4.1 Organization
- **Feature Folders**: Each major area (authentication, dashboard, terminal) has its own folder under `components/`.
- **Shared Components**: Common building blocks (Button, Input, Modal) live in `packages/ui`.

### 4.2 Reusability
- **Atomic Design**: Small “atoms” (e.g., Button) combine into “molecules” (e.g., FormField) and “organisms” (e.g., SignInForm).
- **Props-Driven**: Components are configurable via props and follow a strict props interface in TypeScript.

### 4.3 Benefits
- **Maintainability**: Changes to a shared Button propagate everywhere automatically.
- **Testability**: Small, isolated components are easier to test with Jest and React Testing Library.

## 5. State Management

### 5.1 Approach
- **Local State**: `useState` and `useReducer` for form inputs, UI toggles.
- **Global State**: React Context for themes (`next-themes`) and user session (Better Auth’s AuthProvider).

### 5.2 Data Fetching
- **Next.js Server Components**: Fetch protected data on the server, reducing client bundle size.
- **Client-Side Fetch**: Use the built-in `fetch` or libraries like SWR for live data updates (e.g., real-time logs via WebSocket).

## 6. Routing and Navigation

### 6.1 Next.js App Router
- **File-Based Routing**: Pages and layouts are derived from files in the `app/` directory.
- **Nested Layouts**: Dashboard layout wraps all protected sub-pages (sidebar + header).

### 6.2 Protected Routes
- **Middleware**: A Next.js middleware function checks session tokens before granting access to `/dashboard` routes.
- **Redirects**: Unauthenticated users are sent to `/sign-in` or `/sign-up`.

## 7. Performance Optimization

### 7.1 Built-In Next.js Features
- **Server Components**: Offload data fetching to the server to shrink client bundles.
- **Automatic Code Splitting**: Splits each route into its own chunk.
- **Image and Font Optimization**: Next.js optimizes images and uses `next/font` for on-demand font loading.

### 7.2 Additional Techniques
- **Dynamic Imports**: Lazy-load heavy components (e.g., Monaco editor) only when needed.
- **Tailwind JIT**: Generates only the CSS classes you use at build time.

## 8. Testing and Quality Assurance

### 8.1 Unit & Integration Tests
- **Jest**: Test business logic in components and utility functions.
- **React Testing Library**: Render components in isolation, assert on user interactions and accessibility.

### 8.2 End-to-End (E2E) Tests
- **Playwright**: Simulate full user flows (sign-in, run CLI command, view logs).

### 8.3 Linting & Formatting
- **ESLint** with shared config at the root. Enforces code style, catches errors.
- **Prettier** for consistent formatting.

### 8.4 CI/CD Integration
- **GitHub Actions**: Use pnpm filtering (`pnpm --filter apps/web...`) to lint, type-check, test, and build only when relevant files change. Successful builds deploy the web app to Vercel.

## 9. Conclusion and Overall Frontend Summary

The frontend of `apps/web` is built on a modern, battle-tested stack: Next.js, React, TypeScript, and Tailwind CSS, powered by shadcn/ui for a consistent, accessible design system. Its component-driven approach, strong type safety, and clear separation between UI, state, and data layers make it highly maintainable. Theming, routing, and authentication are all handled with well-supported libraries, ensuring a smooth developer and user experience. With built-in performance optimizations, a clear monorepo setup, and comprehensive testing strategies, this frontend is ready to scale alongside the rest of the `vibecode-clone` platform.
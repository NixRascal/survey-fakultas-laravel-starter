# Frontend Guideline Document

This document explains how the frontend of your faculty survey application is set up. It covers the core architecture, design choices, styling approach, component structure, state handling, routing, performance tips, testing strategies, and more. By following these guidelines, anyone—even without a deep technical background—can understand and work with the frontend code.

## 1. Frontend Architecture

### Frameworks and Libraries
- **Next.js 14 (App Router)**
  - Provides file-based routing, server-side rendering (SSR), static site generation (SSG), and server actions out of the box.
- **React 19**
  - Powers interactive UI components and local state management.
- **TypeScript**
  - Adds type safety to props, state, and API interactions, reducing runtime errors.
- **Tailwind CSS**
  - Utility-first CSS framework for rapid, consistent styling.
- **shadcn/ui**
  - A library of prebuilt, accessible UI components (buttons, inputs, tables) designed for Next.js and Tailwind.
- **next-themes**
  - Simple dark/light mode support with React context and CSS variables.
- **Better Auth**
  - Handles admin sign-up, sign-in, and session management in Next.js API routes.

### Why This Architecture Works
- **Scalability**: Modular pages and components let you grow the app without tangled code. Next.js handles code splitting automatically, so only the needed code loads for each page.
- **Maintainability**: TypeScript and clear folder structure (`app/`, `components/`, `lib/`, `db/`) keep responsibilities separated. Utility-first CSS means fewer custom class names to track.
- **Performance**: Next.js image optimization, server-side rendering, and built-in caching (via Server Actions) ensure fast load times. Tailwind generates only the CSS you use.

## 2. Design Principles

### Usability
- Keep interfaces simple and intuitive. Use clear labels, straightforward forms, and obvious calls to action.
- Reuse familiar UI patterns from shadcn/ui (e.g., consistent button styles) so users learn once and apply everywhere.

### Accessibility
- All components follow WCAG guidelines (proper alt text for images, keyboard focus states, sufficient color contrast).
- Use semantic HTML (headings, lists, form elements) so screen readers can navigate naturally.

### Responsiveness
- Design for mobile first. Tailwind breakpoints (`sm`, `md`, `lg`, `xl`) ensure layouts adapt to different screen sizes.
- Test components on small screens to confirm readability and touch-friendly controls.

## 3. Styling and Theming

### Styling Approach
- **Utility-First CSS (Tailwind)**: Write class names directly in JSX (e.g., `className="px-4 py-2 text-white bg-indigo-600 rounded"`).
- **Component-Level Styles**: Encapsulated in the component file—no global CSS overrides needed.
- **Dark/Light Mode**: Managed by `next-themes`. Use `[data-theme="dark"]` and `[data-theme="light"]` variants in Tailwind.

### Theming Strategy
- Centralize theme variables in `tailwind.config.js` under `theme.extend` for colors, spacing, and fonts.
- Use CSS variables for dynamic theme switches (background, text colors).

### Visual Style
- **Overall Style**: Modern, flat design with subtle shadows (no heavy skeuomorphism). Clean and minimal.
- **Glassmorphism**: Use sparingly—for hero sections or modal backdrops—to add depth without distraction.

### Color Palette
- **Primary**: Indigo 600 (#4F46E5)
- **Primary Light**: Indigo 500 (#6366F1)
- **Secondary**: Teal 500 (#14B8A6)
- **Accent/Error**: Red 500 (#EF4444)
- **Neutral Light**: Gray 50 (#F9FAFB)
- **Neutral Dark**: Gray 800 (#1F2937)
- **Background**: White (#FFFFFF) for light mode, Gray 900 (#111827) for dark mode
- **Text**: Gray 800 (#1F2937) for light, Gray 100 (#F3F4F6) for dark

### Typography
- **Font Family**: Inter, system UI fallbacks
- **Heading Scale**: 
  - H1: 2.25rem (36px)
  - H2: 1.875rem (30px)
  - H3: 1.5rem (24px)
  - Body: 1rem (16px)

## 4. Component Structure

### Organization
- **`app/`**: Contains pages and layouts following Next.js App Router conventions.
- **`components/ui/`**: Houses shared UI widgets from shadcn/ui (buttons, inputs, cards).
- **`components/`**: Application-specific components (e.g., `AppSidebar`, `SurveyForm`, `ReportChart`).
- **`lib/`**: Utility functions and configuration (`auth.ts`, `drizzle.ts`).

### Reuse and Consistency
- All UI elements come from or extend shadcn/ui to ensure consistent look and behavior.
- Encapsulate repeated patterns (e.g., modal dialogs, table filters) into self-contained components.

### Benefits of Component-Based Architecture
- Changes in one component don’t affect others.
- Encourages small, testable units of code.
- Speeds up development by reusing existing components.

## 5. State Management

### How State Is Handled
- **Local State**: React’s `useState` and `useReducer` for form inputs and UI toggles.
- **Global State**: React Context (via next-themes and custom AuthContext) for theme and user session.
- **Server Data**: Next.js Server Actions and API Routes fetch and mutate data. Components call these directly and re-render with new props.

### Sharing State Across Components
- **AuthContext**: Provides `user` and `signIn`/`signOut` methods to any component needing user info.
- **ThemeContext** (from next-themes): Controls dark/light mode application-wide.

## 6. Routing and Navigation

### Routing Setup
- **File-Based Routes**: Under `app/`, folders become routes: `app/page.tsx` (home), `app/(auth)/sign-in`, `app/(protected)/dashboard`, `app/survei/[id]`.
- **Nested Layouts**: Shared elements (e.g., sidebar, nav bar) live in `layout.tsx` files in parent folders.

### Navigation Flow
1. **Public Home** (`/`) lists available surveys.
2. **Survey Page** (`/survei/[id]`) shows the form.
3. **Auth Pages** (`/sign-in`, `/sign-up`) handle admin login.
4. **Protected Dashboard** (`/dashboard`) and subpages require authentication. Redirect logic ensures only logged-in admins can access.

## 7. Performance Optimization

### Key Strategies
- **Code Splitting**: Next.js splits code by route automatically, loading only what’s needed.
- **Lazy Loading**: Dynamically import heavy components (e.g., chart library) with `next/dynamic`.
- **Image Optimization**: Use `next/image` for responsive, optimized images.
- **CSS Purge**: Tailwind removes unused styles in production builds.
- **Caching**: Leverage Next.js built-in caching for server actions and API routes.

### Impact on User Experience
- Faster initial page loads and smooth navigation.
- Reduced bundle sizes save bandwidth on slow connections.
- Images and assets load progressively for a polished feel.

## 8. Testing and Quality Assurance

### Unit and Integration Tests
- **Vitest** or **Jest** + **React Testing Library** for component and utility tests.
- Write tests for form validation, button clicks, and state updates.

### End-to-End (E2E) Tests
- **Playwright** or **Cypress** for full-flow tests (survey submission, admin login, report generation).
- Scripts simulate user actions across browsers to catch regressions early.

### Continuous Integration
- Integrate tests into CI pipeline (GitHub Actions or similar).
- Run linters (ESLint for code style, Prettier for formatting) on every pull request.

## 9. Conclusion and Overall Frontend Summary

Your frontend combines Next.js, React, TypeScript, Tailwind CSS, and shadcn/ui into a modern, scalable setup. By following these guidelines:

- You maintain a clean, component-driven codebase that’s easy to extend.
- You deliver a responsive, accessible UI that works across devices.
- You ensure high performance with built-in Next.js optimizations.
- You keep quality high through testing, linting, and type safety.

This approach aligns with your project’s goals: a robust, user-friendly faculty survey application that’s easy to maintain and deploy. Whenever you add new features—whether it’s more complex reporting charts, advanced theming, or expanded admin roles—this document will help you keep the frontend organized, consistent, and performant. Good luck building your survey app!
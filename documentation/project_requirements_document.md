# Project Requirements Document (PRD)

## 1. Project Overview

This project is a faculty satisfaction survey application built on a modern, full-stack Next.js 14 template. It replaces the originally planned Laravel/PHP stack with a TypeScript-based Next.js setup, leveraging Tailwind CSS for styling and Drizzle ORM for type-safe PostgreSQL interactions. The system serves two main user groups: admins (faculty staff) who create and manage surveys, and respondents (students or faculty members) who fill out the surveys.

The core problem it solves is the need for a flexible, secure, and easy-to-use platform to gather structured and open-ended feedback from faculty members or students. Key objectives include rapid survey creation and deployment, robust data collection with one-submission-per-respondent enforcement, AI-powered analysis of open responses via the Gemini API, and clear reporting with charts and PDF exports. Success will be measured by smooth admin workflows, high respondent completion rates, and accurate, actionable insights in reports.

## 2. In-Scope vs. Out-of-Scope

### In-Scope (Version 1)
- Admin authentication and secure login (Better Auth)
- Admin Panel (dashboard) to:
  - Create, read, update, delete (CRUD) questionnaires
  - Add, edit, delete questions of various types (MCQ, open text)
  - View aggregated survey results and AI summaries
- Public survey interface:
  - List active surveys
  - Render dynamic forms based on question schema
  - Enforce one submission per respondent
- Database schema definitions (Admin, Responden, Kuesioner, Pertanyaan, Jawaban) via Drizzle ORM
- Integration with Gemini API for open-ended answer analysis
- Reporting UI with charts (Chart.js/Recharts) and PDF export (jspdf + html2canvas)
- Dark mode support and responsive design (Tailwind CSS, next-themes)
- Docker & Docker Compose for dev environment
- Deployment configuration for Vercel

### Out-of-Scope (Later Phases)
- Advanced role-based access control beyond basic admin vs super-admin
- Multi-language support (i18n)
- Offline survey capabilities or mobile app
- Integration with third-party LMS or SSO systems
- Advanced analytics beyond Gemini summaries (e.g., sentiment over time)
- Comprehensive end-to-end testing suite

## 3. User Flow

A new **admin** visits the application, clicks "Sign In," and enters credentials via the Better Auth flow. Upon successful login, they land on `/dashboard`. The left sidebar shows links to “Questionnaires,” “Reports,” and “Admin Management.” They click “Questionnaires,” see a list of existing surveys, and hit “Create New.” A form built with shadcn/ui fields appears; they fill in title, description, and add questions. Submitting triggers a Next.js Server Action that validates input with Zod, writes to PostgreSQL via Drizzle ORM, and returns them to the updated list.

A **respondent** goes to the public landing page, which lists all active surveys. They click on a survey card and are taken to `/survei/[id]`. The form dynamically renders each question type (radio buttons, checkboxes, textareas). After filling out all questions, they click “Submit.” The Server Action checks if they’ve already submitted (based on IP or token), saves answers in the `jawaban` table via Drizzle ORM, and returns a thank-you message. Admins later see aggregated data in the Reports section, where they can view pie charts, bar charts, and text-analysis summaries from the Gemini API, and export a PDF.

## 4. Core Features

- **Authentication**: Secure admin sign-up/sign-in using Better Auth, with session management.
- **Admin Panel (Dashboard)**: CRUD for questionnaires and questions, user management.
- **Dynamic Survey Rendering**: Public pages that fetch question data and render forms.
- **Submission Enforcement**: One-time submission rule per respondent.
- **Database Layer**: Type-safe schemas and queries via Drizzle ORM on PostgreSQL.
- **AI Text Analysis**: Backend endpoint calling Gemini API to analyze open responses.
- **Reporting & Visualization**: Charts (Chart.js/Recharts) and tables for aggregated data.
- **PDF Export**: Generate offline reports using jspdf and html2canvas.
- **Theming & Responsiveness**: Dark mode and mobile-first design via Tailwind CSS and next-themes.

## 5. Tech Stack & Tools

- **Frontend**: Next.js 14 (App Router), React 19, TypeScript, Tailwind CSS, shadcn/ui, next-themes
- **Backend**: Next.js API Routes & Server Actions, Node.js, Better Auth
- **Database**: PostgreSQL, Drizzle ORM (type-safe schema & queries)
- **AI Integration**: Gemini API for open-ended text analysis
- **Validation**: Zod for input schema validation
- **Visualization**: Chart.js or Recharts
- **PDF Export**: jspdf + html2canvas
- **Dev Environment**: Docker, Docker Compose, VS Code
- **Deployment**: Vercel

## 6. Non-Functional Requirements

- **Performance**: API responses under 200 ms; page load time under 1 s on 3G emulation
- **Security**: HTTPS enforcement, environment variables for secrets, CSRF protection, input validation
- **Compliance**: GDPR-ready data handling (opt-in consent, data deletion on request)
- **Usability**: WCAG AA accessibility standards, clear form validation messages, mobile responsiveness
- **Scalability**: Support up to 1,000 concurrent respondents and 100 admins

## 7. Constraints & Assumptions

- **Constraints**:
  - Reliance on Gemini API availability and rate limits
  - Hosting on platforms supporting Node.js and Docker
  - Faculty server must allow container deployments or Vercel
- **Assumptions**:
  - One survey submission per unique respondent identifier (IP/email)
  - Admins manage users internally; no public registration
  - Fixed question types: multiple-choice, checkboxes, open text

## 8. Known Issues & Potential Pitfalls

- **API Rate Limits**: Gemini API may throttle; mitigate with retry logic and caching analysis results
- **Schema Migrations**: Drizzle ORM migrations must be run carefully; use version control on `db/migrations` directory
- **Browser Compatibility**: Chart.js may render differently in older browsers; test in Chrome, Firefox, Safari
- **PDF Generation**: jspdf + html2canvas could produce low-resolution images; adjust scale settings
- **Data Integrity**: Concurrent submissions could cause race conditions; use database transactions

---

This PRD serves as the single source of truth for all subsequent technical documents, ensuring clarity and completeness for AI-driven code generation and human review alike.
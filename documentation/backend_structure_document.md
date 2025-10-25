# Backend Structure Document

## 1. Backend Architecture

This project uses a modern, modular server-based architecture built on Next.js 14. Key aspects:

•  **Framework & Design Patterns**  
   – Next.js App Router powers both the public site and API routes.  
   – Server Actions and API Routes handle business logic, separating frontend UI from backend processes.  
   – Drizzle ORM provides a type-safe data access layer in a Repository-style pattern.  

•  **Scalability**  
   – Serverless deployment (on Vercel) automatically scales API routes based on incoming traffic.  
   – Database connection pooling and efficient queries minimize resource usage.  

•  **Maintainability**  
   – TypeScript enforces types across frontend and backend, catching errors early.  
   – Clear folder structure (`app/`, `api/`, `db/`, `lib/`) keeps code organized.  
   – Drizzle migrations and schema files track database changes in code.  

•  **Performance**  
   – Edge caching and CDNs (provided by Vercel) reduce latency for static assets and API responses.  
   – UI components from shadcn/ui are optimized and lazy-loaded when needed.  

## 2. Database Management

•  **Technology**  
   – PostgreSQL (SQL relational database) managed via Drizzle ORM.  
   – Database runs as a managed service (e.g., AWS RDS, DigitalOcean Managed DB) or on-prem if required.  

•  **Data Structure**  
   – Fully normalized schema with foreign-key relationships linking surveys, questions, respondents, and answers.  
   – Enums and constraints enforce valid data at the database level (e.g., question types, user roles).  

•  **Access Patterns**  
   – Read-heavy operations for public survey listing.  
   – Write-heavy operations when saving responses—transactions ensure integrity.  

•  **Data Management Practices**  
   – Migrations in `db/migrations/` keep schema versions in sync.  
   – Backup and restore routines scheduled via managed database provider.  

## 3. Database Schema

### Human-Readable Overview

1.  **Admin**  
    • Stores administrator credentials and roles.  
    • Fields: ID, email, passwordHash, role, timestamps.  

2.  **Responden**  
    • Tracks each survey participant (can be anonymous or by email).  
    • Fields: ID, email (optional), submittedAt, timestamps.  

3.  **Kuesioner (Survey)**  
    • Defines a survey with title, description, and active status.  
    • Fields: ID, title, description, isActive, timestamps.  

4.  **Pertanyaan (Question)**  
    • Lists questions under each survey.  
    • Fields: ID, kuesionerId (FK), text, type ("multiple_choice"|"text"), choices (JSON array for MC), order, timestamps.  

5.  **Jawaban (Answer)**  
    • Captures each respondent’s answer to a question.  
    • Fields: ID, pertanyaanId (FK), respondenId (FK), answerText, selectedChoice, timestamps.  

### PostgreSQL Schema (SQL)

```sql
-- 1. Admin Table
CREATE TABLE admin (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  role VARCHAR(50) NOT NULL CHECK (role IN ('admin','super-admin')),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 2. Responden Table
CREATE TABLE responden (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255),              -- optional
  submitted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 3. Kuesioner Table
CREATE TABLE kuesioner (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 4. Pertanyaan Table
CREATE TABLE pertanyaan (
  id SERIAL PRIMARY KEY,
  kuesioner_id INTEGER NOT NULL REFERENCES kuesioner(id) ON DELETE CASCADE,
  text TEXT NOT NULL,
  type VARCHAR(50) NOT NULL CHECK (type IN ('multiple_choice','text')),
  choices JSONB,                     -- only for multiple_choice
  "order" INTEGER NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 5. Jawaban Table
CREATE TABLE jawaban (
  id SERIAL PRIMARY KEY,
  pertanyaan_id INTEGER NOT NULL REFERENCES pertanyaan(id) ON DELETE CASCADE,
  responden_id INTEGER NOT NULL REFERENCES responden(id) ON DELETE CASCADE,
  answer_text TEXT,
  selected_choice TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (pertanyaan_id, responden_id)
);
```  

## 4. API Design and Endpoints

The backend exposes RESTful endpoints under `/api/` to separate concerns and simplify integration.

•  **Authentication (Better Auth)**  
   – POST `/api/auth/sign-up`: Register a new admin.  
   – POST `/api/auth/sign-in`: Log in and receive a session cookie or token.  
   – GET `/api/auth/sign-out`: End admin session.  

•  **Admin & Role Management**  
   – GET `/api/admin`: List all admins (super-admin only).  
   – POST `/api/admin`: Create new admin.  
   – PUT `/api/admin/:id`: Update admin role or email.  
   – DELETE `/api/admin/:id`: Remove an admin.  

•  **Surveys (Kuesioner)**  
   – GET `/api/surveys`: List active surveys for respondents.  
   – GET `/api/surveys/:id`: Fetch survey details and questions.  
   – GET `/api/kuesioner`: Admin view of all surveys (active/inactive).  
   – POST `/api/kuesioner`: Create a new survey.  
   – PUT `/api/kuesioner/:id`: Update survey metadata.  
   – DELETE `/api/kuesioner/:id`: Delete a survey.  

•  **Questions (Pertanyaan)**  
   – POST `/api/kuesioner/:id/questions`: Add a question to a survey.  
   – PUT `/api/questions/:qid`: Update question text or options.  
   – DELETE `/api/questions/:qid`: Remove a question.  

•  **Responses (Jawaban)**  
   – POST `/api/surveys/:id/responses`: Submit a respondent’s answers.  
   – GET `/api/kuesioner/:id/responses`: Admin fetch of all answers and aggregates.  

•  **Gemini Integration**  
   – POST `/api/gemini/analyze`: Send open-ended text to the Gemini API and return summarized insights.  

## 5. Hosting Solutions

•  **Primary Platform: Vercel**  
   – Serverless functions for API routes, auto-scaling with traffic.  
   – Global CDN for static assets and edge caching.  
   – Zero-config deployments from GitHub.  

•  **Alternative: On-Prem Docker**  
   – Docker Compose setup defines services: Next.js, PostgreSQL, optional Redis.  
   – Suitable for faculty servers with Linux containers.  

Benefits include high availability, pay-per-use scaling, and unified environment between development and production.  

## 6. Infrastructure Components

•  **Load Balancer & CDN**  
   – Vercel’s edge network balances requests across regions.  

•  **Caching**  
   – Edge caching rules for public API endpoints (`/api/surveys`).  
   – In-app caching via SWR or React Query for frequent queries.  

•  **Queue / Worker (Future)**  
   – Optional background jobs (e.g., email reminders) via a lightweight queue (BullMQ with Redis).  

•  **Logging & Error Tracking**  
   – Vercel logs for HTTP requests and build output.  
   – Sentry integration for uncaught exceptions and performance tracing.  

## 7. Security Measures

•  **Authentication & Authorization**  
   – Better Auth library for secure password hashing and session management.  
   – Role-based checks at API middleware (admin vs. super-admin).  

•  **Data Protection**  
   – TLS (HTTPS) enforced for all traffic.  
   – Environment variables (`.env`) for secrets (`DATABASE_URL`, `GEMINI_API_KEY`).  
   – Database encryption at rest provided by the managed DB service.  

•  **Input Validation**  
   – Zod schemas in API routes to validate request payloads and prevent injection attacks.  

•  **Rate Limiting & DDOS Protection**  
   – Vercel’s built-in protections and configurable edge rules for critical endpoints.  

## 8. Monitoring and Maintenance

•  **Performance Monitoring**  
   – Vercel Analytics for request metrics, latency, and error rates.  
   – Sentry for detailed error logging and stack traces.  

•  **Health Checks**  
   – Automated uptime checks (e.g., UptimeRobot) on key endpoints (`/api/health`).  

•  **Database Maintenance**  
   – Scheduled backups and point-in-time recovery.  
   – Regular vacuuming and indexing for PostgreSQL performance.  

•  **Deployment Workflow**  
   – Git-based CI/CD: every merge to main triggers automatic deployment.  
   – Docker Compose available for local testing and staging.  

## 9. Conclusion and Overall Backend Summary

This backend is designed to be:

•  **Robust & Secure**  
   – Strong authentication/authorization, encrypted communications, input validation.  

•  **Scalable & Performant**  
   – Serverless, edge-cached APIs with a reliable PostgreSQL core.  

•  **Maintainable & Extensible**  
   – Clear separation of concerns, type safety via TypeScript/Drizzle, and automated migrations.  

Unique strengths include the seamless integration of AI text analysis via Gemini, a fully typed ORM, and a component-driven UI ready for rapid feature development. This setup aligns directly with the project goals: a modern faculty survey system that’s easy to manage, secure, and capable of delivering rich insights to administrators.
# Code Review - Starting Code Overview and Notes


## Backend (FastAPI)
The previous team built a robust, Dockerized backend for the OHS Remote application.

### Core Features & Functionality
The backend manages the entire customer journey from plan selection to document delivery. It handles AI-assisted Safe Job Procedure (SJP) generation, Stripe payment processing, document generation from DOCX templates, and automated email delivery. A complete and well-documented API is in place for this purpose.

### Architecture & Technology
The codebase follows a strict four-layer architecture (API -> Service -> Repository -> Model) to ensure a clean separation of concerns. The key technologies used are:

Framework: FastAPI (Python 3.11)

Database: MySQL 8.0 with SQLAlchemy ORM and Alembic for migrations

Payments: Stripe Checkout with webhook integration

AI Integration: OpenAI API for SJP content generation

Email: SMTP (Gmail) for transactional emails

Development: A complete Docker Compose setup for a consistent environment.

### APIs & Integrations
The backend provides a comprehensive set of APIs, all versioned under /api/v1:

Public Order Flow: Endpoints for viewing plans, creating orders, submitting company details, and generating previews.

Authentication: An OTP (one-time password) flow using email and httpOnly session cookies.

User Dashboard: Authenticated endpoints for users to view their orders and re-download documents.

Admin API: A completely separate authentication system for administrators to manage orders, customers, and system settings.

SJP Generation: An asynchronous pipeline for AI-powered SJP generation, which is tracked in the database and can be polled for progress.

## Frontend (React)
The frontend is a complete single-page application (SPA) built to interface with the backend.

### Core Features & Functionality
The frontend provides a 5-step wizard for customers to purchase a document and a protected dashboard for users and admins. The customer journey includes picking a plan, uploading a logo, entering company details, previewing the document, and paying via Stripe. The admin interface allows for order review, managing customers, and viewing email logs.

### Architecture & Technology
The app is structured around routing shells and contexts for state management. The technologies used are:

Core: React 18, TypeScript

Build Tool: Vite 5

Routing: react-router-dom v6

Server State: @tanstack/react-query v5

Forms: react-hook-form with zod validation

Styling: Tailwind CSS with two themes (dark for landing pages, light for the app) and shadcn/ui components.

### Implementation Details

Routing: Routes are centrally declared in App.tsx, with specific shells like WizardShell and DashboardShell for different parts of the app.

API Integration: Services are organized in src/services/ for user-facing and admin APIs. API calls are made through an authFetch wrapper to handle authentication and errors uniformly.

State Management: The app uses contexts for managing state across the application, including WizardContext, AuthContext, and AdminAuthContext.

## Project Management & Documentation
The previous team demonstrated strong project management and documentation practices. The work was tracked on a GitHub Project Board with a three-column workflow (Todo, On Review, Closed), and issues were grouped into GitHub Milestones. The team left behind an extensive set of technical documentation, including:

A detailed project README.

A first-time setup guide (GETTING_STARTED.md).

Separate ARCHITECTURE.md files for both the backend and frontend, explaining their design decisions.

An API_OVERVIEW.md and ADMIN_API.md document.

A CHANGELOG.md that provides a high-level summary of all implemented features.

### Known Limitations
The previous team's documentation also transparently notes the project's known limitations, which is very helpful for your team. Key ones include:

No Production Deployment Configuration: The Docker stack is set up for local development only, and a production deployment setup has not been implemented.

Hardcoded Frontend URLs: The API base URLs are hardcoded in src/services/api.ts and src/services/adminApi.ts; there is currently no environment variable for this.

Unfinished S3 File Storage: While there is a USE_S3 toggle in the configuration, the integration with S3 has not been implemented.

Missing Test Suite: The frontend's testing infrastructure is set up but no tests have been written.

In-Memory Wizard State: The state for the customer order wizard is stored in memory. A page refresh during the process will lose any unsaved data.

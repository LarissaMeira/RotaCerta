RotaCerta — Financial Management for Truck Drivers
Platform Overview
RotaCerta is a mobile-first web application designed specifically for independent truck drivers (caminhoneiros) in Brazil who need a simple, reliable way to track income and expenses on the road.

Truck drivers deal with a unique financial reality: irregular freight payments, daily fuel costs across multiple states, tolls, vehicle maintenance, insurance, financing, and food expenses — all while driving. RotaCerta gives them a single, intuitive interface to log every transaction, visualize their financial health per period, and export professional spreadsheets and PDFs for accounting or tax purposes.

System Architecture
┌─────────────────────────────────────────────────────────┐
│                     CLIENT (SPA)                        │
│                                                         │
│  React 18 + TypeScript + Vite 5 + Tailwind CSS v3       │
│                                                         │
│  ┌──────────┐ ┌──────────────┐ ┌──────────────────────┐ │
│  │Dashboard │ │AddTransaction│ │  Reports / Export    │ │
│  │BalanceUI │ │  + Recurring │ │  PDF (jsPDF) / XLS   │ │
│  └────┬─────┘ └──────┬───────┘ │  (ExcelJS)           │ │
│       │              │         └──────────┬───────────┘ │
│  ┌────┴──────────────┴──────────────┬─────┘             │
│  │         Index.tsx (Controller)   │                   │
│  │  State management, CRUD logic,   │                   │
│  │  period filtering, auth guard    │                   │
│  └──────────────┬───────────────────┘                   │
│                 │ Supabase JS SDK                       │
└─────────────────┼───────────────────────────────────────┘
                  │ HTTPS (REST + Realtime)
┌─────────────────┼───────────────────────────────────────┐
│          LOVABLE CLOUD (Supabase)                       │
│                 │                                       │
│  ┌──────────────┴───────────────────┐                   │
│  │         PostgreSQL Database      │                   │
│  │                                  │                   │
│  │  transactions    profiles        │                   │
│  │  custom_categories               │                   │
│  │  frequent_descriptions           │                   │
│  └──────────────────────────────────┘                   │
│                                                         │
│  ┌──────────────────────────────────┐                   │
│  │     Auth (email + password)      │                   │
│  │     Email verification flow      │                   │
│  │     Password recovery flow       │                   │
│  └──────────────────────────────────┘                   │
│                                                         │
│  ┌──────────────────────────────────┐                   │
│  │     Row-Level Security (RLS)     │                   │
│  │     Per-user data isolation      │                   │
│  └──────────────────────────────────┘                   │
└─────────────────────────────────────────────────────────┘
Main Modules
Module	File(s)	Responsibility
Auth	Auth.tsx	Sign-up with email verification, sign-in with "remember me", password recovery, Zod validation
Dashboard	Dashboard.tsx	Balance summary, revenue vs. expenses bar, period selector (date range picker), recent transactions list, show/hide values toggle
Add Transaction	AddTransaction.tsx, NumberInput.tsx	Category selection (10 built-in + custom), amount input, date picker, description with autocomplete from frequent descriptions, recurring transaction config (weekly/monthly by count or end date)
Transaction History	TransactionHistory.tsx	Full transaction list with search, category/type filters, edit and delete actions, duplicate functionality
Reports	Reports.tsx	Period-based financial reports, category breakdown, daily/weekly grouping, export to Excel (.xlsx) and PDF, inline spreadsheet viewer
Spreadsheet Viewer	SpreadsheetViewer.tsx	In-app interactive spreadsheet with horizontal/vertical scroll, color palette themes, PDF and Excel export from the viewer itself
Category Manager	CategoryManager.tsx, CustomCategoryModal.tsx	CRUD for user-defined categories stored in the database
Draggable FAB	DraggableFAB.tsx	Floating action button with drag-to-reposition (AssistiveTouch-style), position persistence via localStorage, glow effect
Navigation	Navigation.tsx	Bottom tab bar (Dashboard, History, Reports) + FAB integration
Projection Chart	ProjectionChart.tsx	Financial projection visualization using Recharts
Utilities	lib/utils.ts	parseLocalDate / toLocalDateString (timezone-safe date handling), currency formatting (BRL), number parsing for Brazilian format
Data Flow
User action (e.g. save transaction)
       │
       ▼
 AddTransaction.tsx  ──▶  Index.tsx (handleSaveTransaction)
                              │
                              ├─ Insert into Supabase `transactions` table
                              ├─ If recurring: compute future dates, bulk insert
                              ├─ Update local state (setTransactions)
                              └─ Show toast confirmation
                              
 On app load (Index.tsx useEffect):
       │
       ├─ Check auth session → redirect to /auth if none
       ├─ Fetch all transactions for user (SELECT with RLS)
       ├─ Fetch custom categories
       ├─ Fetch frequent descriptions
       └─ Apply date range filter from localStorage or default to current month
All date strings are stored as YYYY-MM-DD (PostgreSQL DATE column) and parsed with parseLocalDate() to avoid UTC timezone shift issues common in Brazilian timezones (UTC-3).

AI & Agentic Design
This project does not currently incorporate LLM models, RAG pipelines, or agentic AI workflows. It is a deterministic CRUD application with client-side report generation.

The entire application was built and iterated using Lovable AI as the development agent — functioning as an AI-powered pair programmer that generates, refactors, and debugs code through natural language conversation.

Automation Workflows
Workflow	Type	Description
Recurring Transactions	Fully autonomous	When a user enables recurrence on a transaction, the system auto-generates all future entries (weekly or monthly) in a single batch insert
Email Verification	Automated with user checkpoint	Sign-up triggers verification email; user must click link to activate account
Password Recovery	Automated with user checkpoint	User requests reset → receives email → sets new password via authenticated session
Profile Creation	Fully autonomous	Database trigger (handle_new_user) auto-creates a profile row when a new user signs up
Date Range Persistence	Fully autonomous	Selected period is saved to localStorage and restored on next session
FAB Position Persistence	Fully autonomous	Draggable button position saved to localStorage
Tool Roles
Tool	Role
Lovable	AI-assisted frontend development, component generation, iterative debugging, deployment
Lovable Cloud (Supabase)	PostgreSQL database, authentication (email/password), Row-Level Security, edge functions runtime, secrets management
Key Technical Decisions
Decision	Rationale
Client-side PDF/Excel generation (jsPDF + ExcelJS)	Eliminates need for a server-side document service; works offline after data loads; keeps architecture simple
DATE column type instead of TIMESTAMPTZ	Prevents timezone-induced date shifts — critical for Brazilian users (UTC-3) who saw dates shift +1 day
parseLocalDate() everywhere	Centralized utility that splits YYYY-MM-DD and constructs new Date(y, m-1, d) avoiding new Date("2026-01-22") UTC interpretation
Single-page controller pattern (Index.tsx)	All state, auth, and CRUD logic lives in one orchestrator component; child components are purely presentational — simplifies data flow at current scale
localStorage for UI preferences	Date range, FAB position, and "remember me" email persist without extra database tables
RLS per table	Every table has per-user policies (auth.uid() = user_id); no data leakage between users
No custom backend API	Supabase JS SDK talks directly to PostgreSQL via PostgREST; edge functions reserved for future needs
Brazilian-first UX	All UI in Portuguese, BRL currency formatting, Brazilian phone validation, pt-BR date locale
What was iterated on
Date handling: Multiple iterations to eliminate +1 day bug — moved from TIMESTAMPTZ to DATE, removed all new Date(string) calls, centralized parsing
Export architecture: Unified export buttons (PDF + XLS) across both Reports and Spreadsheet Viewer screens after user feedback about inconsistent access points
FAB component: Evolved from static positioned button to draggable with glow effect and position persistence after user reported it blocking content
Scrolling in spreadsheet: Multiple iterations on horizontal/vertical scroll behavior with custom styled scrollbars matching the dark theme
Stack Summary
Layer	Technology
Frontend	React 18, TypeScript 5, Vite 5, Tailwind CSS v3, shadcn/ui, Recharts, Framer-style animations
State Management	React useState/useEffect + TanStack React Query
Document Generation	jsPDF + jspdf-autotable (PDF), ExcelJS (XLSX)
Routing	React Router DOM v6
Validation	Zod (auth forms)
Backend	Lovable Cloud (Supabase) — PostgreSQL, Auth, RLS
Hosting	Lovable (auto-deploy from repository)
Design System	shadcn/ui components + custom CSS variables (HSL tokens), dark theme with glass-morphism effects
Project Links
Live App: caminhoneiros-rota-certa.lovable.app
Lovable Project: lovable.dev/projects/87faef54-71a8-4ea1-95ce-75d1cb6b1480

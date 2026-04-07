# 🚛 RotaCerta

### Financial Management for Independent Truck Drivers
---

## 📖 About

**RotaCerta** is a mobile-first web application built specifically for independent truck drivers (*caminhoneiros*) in Brazil who need a simple, reliable way to track income and expenses on the road.

> Irregular freight payments, fuel costs across multiple states, tolls, vehicle maintenance, insurance, financing, and food expenses — all in a single intuitive panel, accessible from any phone.

With RotaCerta, drivers can:

- 📥 Log every transaction quickly while on the road
- 📊 Visualize their financial health per period
- 📄 Export professional spreadsheets and PDFs for accounting or tax purposes

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     CLIENT (SPA)                        │
│                                                         │
│  React 18 + TypeScript + Vite 5 + Tailwind CSS v3      │
│                                                         │
│  ┌──────────┐ ┌──────────────┐ ┌──────────────────────┐ │
│  │Dashboard │ │AddTransaction│ │  Reports / Export     │ │
│  │BalanceUI │ │  + Recurring │ │  PDF (jsPDF) / XLS   │ │
│  └────┬─────┘ └──────┬───────┘ └──────────┬───────────┘ │
│  ┌────┴──────────────┴──────────────┬──────┘            │
│  │       Index.tsx (Controller)     │                   │
│  │  State, CRUD, filtering, auth    │                   │
│  └──────────────┬────────────────────┘                  │
│                 │ Supabase JS SDK                       │
└─────────────────┼───────────────────────────────────────┘
                  │ HTTPS (REST + Realtime)
┌─────────────────┼───────────────────────────────────────┐
│          LOVABLE CLOUD (Supabase)                       │
│                                                         │
│  PostgreSQL · Auth (email+password) · Row-Level Security│
│                                                         │
│  transactions · profiles · custom_categories           │
│  frequent_descriptions                                  │
└─────────────────────────────────────────────────────────┘
```

---

## 🧩 Main Modules

| Module | File(s) | Responsibility |
|---|---|---|
| **Auth** | `Auth.tsx` | Sign-up with email verification, sign-in with "remember me", password recovery, Zod validation |
| **Dashboard** | `Dashboard.tsx` | Balance summary, revenue vs. expenses bar, period selector, recent transactions list, show/hide values toggle |
| **Add Transaction** | `AddTransaction.tsx`, `NumberInput.tsx` | Category selection (10 built-in + custom), amount input, date picker, description autocomplete, recurring transaction config |
| **Transaction History** | `TransactionHistory.tsx` | Full transaction list with search, category/type filters, edit, delete, and duplicate actions |
| **Reports** | `Reports.tsx` | Period-based reports, category breakdown, daily/weekly grouping, export to Excel and PDF |
| **Spreadsheet Viewer** | `SpreadsheetViewer.tsx` | In-app interactive spreadsheet with scroll, color themes, PDF and Excel export |
| **Category Manager** | `CategoryManager.tsx`, `CustomCategoryModal.tsx` | CRUD for user-defined categories stored in the database |
| **Draggable FAB** | `DraggableFAB.tsx` | Floating action button with drag-to-reposition (AssistiveTouch-style), position persistence via localStorage |

---

## 🔄 Data Flow

```
User action (e.g. save transaction)
       │
       ▼
 AddTransaction.tsx  ──▶  Index.tsx (handleSaveTransaction)
                              │
                              ├─ Insert into Supabase `transactions` table
                              ├─ If recurring: compute future dates → bulk insert
                              ├─ Update local state (setTransactions)
                              └─ Show toast confirmation

 On app load (Index.tsx useEffect):
       ├─ Check auth session → redirect to /auth if unauthenticated
       ├─ Fetch all transactions for user (SELECT with RLS)
       ├─ Fetch custom categories
       ├─ Fetch frequent descriptions
       └─ Apply date range filter from localStorage or default to current month
```

> ⚠️ All date strings are stored as `YYYY-MM-DD` (PostgreSQL `DATE` column) and parsed with `parseLocalDate()` to avoid UTC timezone shift issues common in Brazilian timezones (UTC-3 → +1 day bug).

---

## ⚙️ Automation Workflows

| Workflow | Type | Description |
|---|---|---|
| **Recurring Transactions** | Fully autonomous | Generates all future entries (weekly/monthly) in a single batch insert |
| **Email Verification** | Automated with user checkpoint | Sign-up triggers verification email; user clicks link to activate account |
| **Password Recovery** | Automated with user checkpoint | User requests reset → receives email → sets new password via authenticated session |
| **Profile Creation** | Fully autonomous | Database trigger `handle_new_user` auto-creates a profile row on sign-up |
| **Date Range Persistence** | Fully autonomous | Selected period saved to `localStorage` and restored on next session |
| **FAB Position Persistence** | Fully autonomous | Draggable button position saved to `localStorage` |

---

## 🛠️ Key Technical Decisions

| Decision | Rationale |
|---|---|
| **Client-side PDF/Excel** (jsPDF + ExcelJS) | Eliminates need for a server-side document service; works offline after data loads |
| **`DATE` column instead of `TIMESTAMPTZ`** | Prevents timezone-induced date shifts for Brazilian users (UTC-3) |
| **Centralized `parseLocalDate()`** | Uses `new Date(y, m-1, d)` instead of `new Date("YYYY-MM-DD")` to avoid UTC interpretation |
| **Single controller (`Index.tsx`)** | All state, auth, and CRUD logic in one orchestrator; child components are purely presentational |
| **`localStorage` for UI preferences** | Date range, FAB position, and "remember me" email persist without extra database tables |
| **RLS on every table** | `auth.uid() = user_id` policies on all tables; no data leakage between users |
| **No custom backend API** | Supabase JS SDK talks directly to PostgreSQL via PostgREST; edge functions reserved for future needs |
| **Brazil-first UX** | Full Portuguese UI, BRL currency formatting, `pt-BR` date locale, Brazilian phone validation |

---

## 📦 Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React 18, TypeScript 5, Vite 5, Tailwind CSS v3, shadcn/ui, Recharts |
| **State Management** | React useState/useEffect + TanStack React Query |
| **Document Generation** | jsPDF + jspdf-autotable (PDF), ExcelJS (XLSX) |
| **Routing** | React Router DOM v6 |
| **Validation** | Zod (auth forms) |
| **Backend** | Lovable Cloud (Supabase) — PostgreSQL, Auth, RLS |
| **Hosting** | Lovable (auto-deploy from repository) |
| **Design System** | shadcn/ui + custom CSS variables (HSL tokens), dark theme with glass-morphism effects |

---

## 🔗 Links

| | |
|---|---|
| 🌐 **Live App** | [caminhoneiros-rota-certa.lovable.app](https://caminhoneiros-rota-certa.lovable.app) |

---


Built with Lovable ❤️ for the truck drivers of Brazil 🇧🇷


# TransitOps – Smart Transport Operations Platform

[![React](https://img.shields.io/badge/React-19-blue.svg?logo=react)](https://react.dev/)
[![Vite](https://img.shields.io/badge/Vite-8-646CFF.svg?logo=vite)](https://vitejs.dev/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-v4-38B2AC.svg?logo=tailwind-css)](https://tailwindcss.com/)
[![Clerk](https://img.shields.io/badge/Auth-Clerk-6C47FF.svg?logo=clerk)](https://clerk.com/)
[![Supabase](https://img.shields.io/badge/Database-Supabase_PostgreSQL-3ECF8E.svg?logo=supabase)](https://supabase.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

An enterprise-grade, full-stack logistics and transport operations management platform engineered for end-to-end fleet visibility, automated dispatch orchestration, driver compliance management, and financial operational intelligence.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Key Features](#-key-features)
- [Role-Based Access Control (RBAC)](#-role-based-access-control-rbac)
- [Technology Stack](#-technology-stack)
- [Project Screenshots](#-project-screenshots)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Installation & Quickstart](#-installation--quickstart)
- [Environment Configuration](#-environment-configuration)
- [Authentication Setup (Clerk)](#-authentication-setup-clerk)
- [Database Setup (Supabase & PostgreSQL)](#-database-setup-supabase--postgresql)
- [Available Scripts](#-available-scripts)
- [Security & Resilience Design](#-security--resilience-design)
- [Roadmap & Future Improvements](#-roadmap--future-improvements)
- [Author & Acknowledgments](#-author--acknowledgments)
- [License](#-license)

---

## 📖 Overview

**TransitOps** is designed to solve real-world challenges in transport operations: fragmented vehicle tracking, scheduling conflicts, unmonitored maintenance downtime, regulatory compliance oversights, and unpredictable operational costs.

The platform bridges real-time fleet operations with database-enforced business constraints. Critical rules—such as preventing cargo weight overloads, blocking expired driver licenses during dispatch, automatically updating vehicle availability states, and generating fuel logs on trip completion—are enforced at the PostgreSQL layer via triggers and Row Level Security (RLS), guaranteeing data integrity across all access paths.

---

## 🏗 System Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                   React 19 Frontend (Vite)                  │
│                                                             │
│  [Dashboard] [Vehicles] [Drivers] [Trips] [Dispatch]        │
│  [Maintenance] [Expenses] [Reports & Analytics] [Settings]  │
└──────────────────────────────┬──────────────────────────────┘
                               │
                ┌──────────────┴──────────────┐
                ▼                             ▼
       useSupabase() Hook              Theme & UI Context
     (Clerk Session + JWT)            (Dark / Light Modes)
                │
                ▼
      Central Service Layer
    (src/services/database.js)
    (src/services/fleetService.js)
    - Generic CRUD operations
    - Local storage resilience fallback
                │
                ▼ (Authenticated PostgREST Client)
┌─────────────────────────────────────────────────────────────┐
│                 Supabase PostgreSQL 15+                    │
│                                                             │
│  • Row Level Security (RLS) policies per Role               │
│  • Helper Functions: current_user_id(), current_user_role() │
│  • Triggers:                                                │
│      - handle_trip_status_changes()                         │
│      - handle_maintenance_status_changes()                 │
│  • Tables: profiles, vehicles, drivers, trips,              │
│            maintenance_logs, fuel_logs, expenses            │
└─────────────────────────────────────────────────────────────┘
```

---

## ✨ Key Features

### 1. 🚚 Vehicle Lifecycle Management
- **Asset Registry**: Track registration numbers, vehicle types (Truck, Van, Mini, Bus), maximum load capacities, odometer readings, and acquisition costs.
- **Automated Status Lifecycle**: Dynamic states (`Available`, `On Trip`, `In Shop`, `Retired`) driven by dispatch and maintenance events.
- **Search & Filtering**: Real-time filtering by status, vehicle type, and operational region.

### 2. 👨‍✈️ Driver Compliance & Safety
- **Compliance Registry**: Manage CDL license numbers, categories (A, B, C), expiration dates, and contact details.
- **Safety Scoring**: Track operator safety performance (0–100 index).
- **Automated Dispatch Lock**: PostgreSQL triggers reject dispatch attempts for suspended drivers or operators with expired licenses.

### 3. 📦 Smart Dispatch & Trip Workflow
- **Trip Lifecycle Machine**: Seamless state progression (`Draft` $\rightarrow$ `Dispatched` $\rightarrow$ `Completed` $\rightarrow$ `Cancelled`).
- **Load Validation**: Automated check ensuring cargo weight never exceeds assigned vehicle capacity.
- **Trip Completion Settlement**: Automatically updates vehicle odometer and records fuel logs on trip completion.

### 4. 🔧 Maintenance & Work Order Management
- **Incident Tracking**: Register repair tickets with issue descriptions, estimated expenses, and start/end dates.
- **Fleet Safeguard**: Automatically transitions assigned vehicles to `In Shop` mode upon active repair logging, restoring them to `Available` only after ticket closure.

### 5. 💰 Fuel & Operational Expense Auditing
- **Expense Categorization**: Track expenses across `Fuel`, `Tolls`, `Maintenance`, `Permit`, and `Other`.
- **Cost Allocation**: Associate expenditures with specific fleet units for vehicle-level ROI analysis.
- **Date Range Auditing**: Filter and inspect expenditures over rolling 30, 60, and 90-day operational windows.

### 6. 📊 Real-Time Analytics & Operational Intelligence
- **Interactive KPI Matrix**: Fleet utilization rate, active vs. in-shop units, dispatched routes, and driver duty counts.
- **Financial Visualization**: Dynamic monthly revenue charts powered by Recharts.
- **Cost Breakdown**: Top expenditure vehicles and operational cost distributions.
- **Live Event Stream**: Simulated vehicle telemetry ticker providing real-time fleet activity feeds.
- **Data Export**: One-click export of consolidated operations data to CSV format.

### 7. 🎨 Modern User Experience
- **Split-Screen Authentication**: Dynamic login interface featuring an auto-playing transport visual backdrop and Clerk authentication.
- **Dual-Theme Support**: Dark and Light modes with system persistence.
- **Animated Sharingan Loader**: Custom SVG loading indicator for smooth state transitions.

---

## 👥 Role-Based Access Control (RBAC)

TransitOps implements granular Role-Based Access Control enforced at both the UI layer (gated sidebar and views) and the database layer (PostgreSQL Row Level Security):

| Role | UI Capabilities | Database Privileges (RLS) |
|------|-----------------|---------------------------|
| **Fleet Manager** | Full access to all modules, assets, work orders, and analytics | Full CRUD on `vehicles`, `drivers`, `trips`, `maintenance_logs`, `fuel_logs`, `expenses` |
| **Driver / Dispatcher** | Vehicle registry, trip execution, active dispatch management | INSERT & UPDATE on `trips` (cannot delete or modify asset core specs) |
| **Safety Officer** | Driver compliance tracking, license expiry auditing, maintenance review | UPDATE on `drivers` (safety scores & compliance), SELECT on operational records |
| **Financial Analyst** | Expense tracking, fuel logs audit, reports, and ROI analytics | Full CRUD on `expenses` and `fuel_logs`, SELECT on fleet telemetry |

---

## 🛠 Technology Stack

### Frontend Core
- **Framework**: [React 19](https://react.dev/)
- **Build Tool**: [Vite 8](https://vitejs.dev/)
- **Styling**: [Tailwind CSS v4](https://tailwindcss.com/)
- **Routing**: [React Router DOM v7](https://reactrouter.com/)
- **Icons**: [Lucide React](https://lucide.dev/)
- **Charts**: [Recharts](https://recharts.org/)
- **Notifications**: [Sonner](https://sonner.emilkowal.ski/)
- **State & Utilities**: [Zustand](https://github.com/pmndrs/zustand), `clsx`, `tailwind-merge`

### Backend & Cloud Infrastructure
- **Authentication**: [Clerk React SDK](https://clerk.com/) (`@clerk/react`) with custom JWT Supabase template
- **Database**: [Supabase](https://supabase.com/) (PostgreSQL 15+)
- **Security**: PostgreSQL Row Level Security (RLS) & `SECURITY DEFINER` procedures
- **API Communication**: `@supabase/supabase-js`

---

## 📷 Project Screenshots

> *Tip: Screenshots can be placed in `docs/screenshots/`.*

### 1. Split-Screen Authentication & Role Selection
```
+------------------------------------------+------------------------------------------+
|                                          |               [TransitOps]               |
|                                          |      Smart Transport Operations Platform │
|                                          |                                          |
|            [Truck Visual Feed]           |          [Select Operating Role]         |
|         Seamless looping transport       |  [Fleet Manager]       [Dispatcher]      |
|           fleet operations view          |  [Safety Officer]      [Financial Analyst|
|                                          |                                          |
|                                          |            [Clerk Auth Portal]           |
+------------------------------------------+------------------------------------------+
```
*(Screenshot placeholder: `docs/screenshots/login.png`)*

### 2. Fleet Overview Dashboard & Real-Time KPIs
```
+-------------------------------------------------------------------------------------+
| [TransitOps]   [Total Fleet: 24]  [Available: 16]  [On Trip: 5]  [In Shop: 3]       |
|                                                                                     |
| [Filter: All Types] [Status: All] [Region: Global]                 [Fleet Util: 76%]|
|                                                                                     |
| +-------------------------+ +-------------------------+ +-------------------------+ |
| | Heavy Duty Semi #TX-982 | | Express Delivery Van    | | Intercity Coach Bus     | |
| | Capacity: 15.0 Tons     | | Capacity: 2.5 Tons      | | Capacity: 12.0 Tons     | |
| | Status: AVAILABLE       | | Status: ON TRIP         | | Status: IN SHOP         | |
| +-------------------------+ +-------------------------+ +-------------------------+ |
+-------------------------------------------------------------------------------------+
```
*(Screenshot placeholder: `docs/screenshots/dashboard.png`)*

### 3. Dispatch Operations & Cargo Limit Gating
*(Screenshot placeholder: `docs/screenshots/dispatch.png`)*

### 4. Financial Auditing & Analytics Center
*(Screenshot placeholder: `docs/screenshots/analytics.png`)*

---

## 📂 Project Structure

```text
transitops/
├── .agents/                      # AI development skills and references
├── docs/                         # Architecture documentation and guides
│   └── clerk.md                  # Clerk auth integration guide
├── logs/                         # Session modification logs
├── public/                       # Static public assets
│   ├── favicon.svg               # Application SVG brand icon
│   ├── truck_img.png             # Transport asset imagery
│   └── truck_video.mp4           # Authentication backdrop video
├── src/
│   ├── auth/
│   │   ├── AGENTS.md             # Auth module architecture specification
│   │   └── supabase.js           # Clerk-authenticated Supabase client factory
│   ├── components/
│   │   ├── SharinganLoader.jsx   # Custom animated SVG loading indicator
│   │   ├── Truck3DScene.jsx      # Video backdrop container with gradient masks
│   │   └── UserProfileSection.jsx# Operator profile management & webcam capture
│   ├── context/
│   │   └── ThemeContext.jsx      # Dark/Light theme provider and toggle
│   ├── lib/
│   │   └── utils.ts              # Classname merging utility
│   ├── pages/
│   │   ├── AnalyticsView.jsx     # Financial metrics, revenue charts & telemetry
│   │   ├── DashboardView.jsx     # Fleet-wide KPI matrix and asset cards
│   │   ├── DispatchView.jsx      # Trip creation, assignment & completion
│   │   ├── DriversView.jsx       # Driver profiles, license expiry & safety scores
│   │   ├── ExpensesView.jsx      # Operational expenditure logging & auditing
│   │   ├── MaintenanceView.jsx   # Vehicle work orders and repair scheduling
│   │   ├── SettingsView.jsx      # Speed limit thresholds & alert preferences
│   │   ├── TripsView.jsx         # Global trip manifest & status tracking
│   │   └── VehiclesView.jsx      # Vehicle inventory CRUD and specifications
│   ├── services/
│   │   ├── AGENTS.md             # Services layer specification
│   │   ├── database.js           # Generic CRUD helper with local resilience fallback
│   │   └── fleetService.js       # Typed domain-specific fleet operations API
│   ├── App.css
│   ├── App.jsx                   # Main layout shell, sidebar routing & auth gateway
│   ├── index.css                 # Tailwind CSS v4 directives & theme styles
│   └── main.jsx                  # Application entry point with ClerkProvider
├── .env.example                  # Environment variable template
├── .gitignore                    # Git exclusion rules
├── AGENTS.md                     # Project governance and agent instructions
├── eslint.config.js              # ESLint flat configuration (React 19 + Vite)
├── index.html                    # HTML entry template
├── package.json                  # Dependencies and script definitions
├── README.md                     # Project documentation
├── supabase_schema_and_guide.md  # Production database DDL, triggers & RLS guide
├── supabase_schema.txt           # [Archived] Initial prototype DDL (for history)
└── vite.config.js                # Vite build and path alias configuration
```

---

## 💻 Prerequisites

Ensure you have the following installed on your machine:
- **Node.js**: `v18.0.0` or later (LTS recommended)
- **npm**: `v9.0.0` or later
- A free **[Clerk](https://clerk.com/)** account for authentication
- A free **[Supabase](https://supabase.com/)** account for PostgreSQL database hosting

---

## 🚀 Installation & Quickstart

### 1. Clone the Repository
```bash
git clone https://github.com/Darshan-38/transitops.git
cd transitops
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Configure Environment Variables
Copy `.env.example` to create your local `.env` file:
```bash
cp .env.example .env
```
Open `.env` in your editor and provide your Clerk and Supabase credentials (see [Environment Configuration](#-environment-configuration)).

### 4. Deploy the Database Schema
Execute the production SQL script from `supabase_schema_and_guide.md` in your Supabase SQL Editor (see [Database Setup](#-database-setup-supabase--postgresql)).

### 5. Start the Development Server
```bash
npm run dev
```
Open [http://localhost:5173](http://localhost:5173) in your browser.

---

## 🌐 Environment Configuration

The application requires the following environment variables defined in `.env`:

```env
# Clerk Authentication Publishable Key (from Clerk Dashboard -> API Keys)
VITE_CLERK_PUBLISHABLE_KEY=pk_test_xxxxxxxxxxxxxxxxxxxxxxxx

# Supabase Project URL (from Supabase Project Settings -> API)
VITE_SUPABASE_URL=https://xxxxxxxxxxxxxxxx.supabase.co

# Supabase Public/Anon API Key (from Supabase Project Settings -> API)
VITE_SUPABASE_PUBLISHABLE_KEY=eyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Clerk JWT Template configured for Supabase (Default: supabase)
VITE_CLERK_JWT_TEMPLATE=supabase
```

---

## 🔐 Authentication Setup (Clerk)

TransitOps uses **Clerk** for user authentication and session management.

1. **Create an Application** in the [Clerk Dashboard](https://dashboard.clerk.com/).
2. Enable **Email / Password** and optionally **Google OAuth** under *User & Authentication*.
3. Go to **API Keys** and copy your **Publishable Key** into `VITE_CLERK_PUBLISHABLE_KEY`.
4. Navigate to **JWT Templates** $\rightarrow$ Click **New template** $\rightarrow$ Choose **Supabase**.
5. Name the template `supabase` (matching `VITE_CLERK_JWT_TEMPLATE`).
6. In the template settings, set the **Signing Key** to your Supabase project's JWT Secret (found in Supabase under *Project Settings* $\rightarrow$ *API* $\rightarrow$ *JWT Secret*).
7. Save the template. The client will automatically sign Supabase requests with the user's Clerk session token.

---

## 🗄 Database Setup (Supabase & PostgreSQL)

The database schema and business triggers are documented in **[`supabase_schema_and_guide.md`](supabase_schema_and_guide.md)**.

### Deployment Steps:
1. Open your Supabase project and navigate to **SQL Editor**.
2. Open **[`supabase_schema_and_guide.md`](supabase_schema_and_guide.md)** and copy the SQL statements from **Section 1 (Database Schema Setup Script)**.
3. Paste the SQL into the Supabase SQL Editor and click **Run**.

### What gets provisioned:
- **`public.profiles`**: Maps Clerk users (`user_...`) to roles (`Fleet Manager`, `Driver`, `Safety Officer`, `Financial Analyst`).
- **`public.vehicles`**: Fleet assets with load limits, odometer tracking, and status checks.
- **`public.drivers`**: Operator records with license categories, expiry dates, and safety ratings.
- **`public.trips`**: Dispatch manifests linking vehicles and drivers with cargo and distance metrics.
- **`public.maintenance_logs`**: Service tickets with automatic vehicle status synchronization.
- **`public.fuel_logs`**: Fuel consumption and refueling expenditure records.
- **`public.expenses`**: General fleet operating expenditures.
- **Triggers**:
  - `handle_trip_status_changes`: Validates cargo capacity, license validity, and manages vehicle/driver availability automatically.
  - `handle_maintenance_status_changes`: Automatically sets vehicle status to `In Shop` when maintenance is opened, and returns it to `Available` on closure.
- **Row Level Security (RLS)**: Enforces role-based read and write permissions across all tables.

---

## 📜 Available Scripts

| Command | Purpose |
|---------|---------|
| `npm run dev` | Starts the Vite development server at `http://localhost:5173` |
| `npm run build` | Compiles and bundles production-ready assets into `dist/` |
| `npm run preview` | Starts a local web server to preview the production build |
| `npm run lint` | Runs ESLint across all JavaScript/JSX source files |

---

## 🛡 Security & Resilience Design

- **Zero Secret Exposure**: Only public publishable keys are bundled into frontend assets. Supabase `service_role` and private backend keys are never stored in client code.
- **Database-Level Protection**: All queries are evaluated against PostgreSQL Row Level Security (RLS) policies tied to the verified JWT session.
- **Local Fallback Resilience**: `src/services/database.js` provides automatic localStorage persistence fallback, enabling offline functionality and uninterrupted demo presentation during network interruptions.
- **Strict Input Validation**: Client forms enforce input boundary checks (e.g., non-negative distances, valid date windows) before requests reach the database.

---

## 🔮 Roadmap & Future Improvements

- [ ] **Live Telemetry & GPS Tracking**: Integrate Leaflet / Mapbox with simulated OBD-II GPS tracking.
- [ ] **Automated Alerting**: Push notifications and SMS alerts via Twilio for license expirations and maintenance due dates.
- [ ] **AI Route Optimization**: Predictive route planning based on cargo weight and historical fuel consumption.
- [ ] **Multi-Tenant Fleet Support**: Tenant organization switching for multi-fleet transport operators.
- [ ] **Mobile Native Companion**: React Native companion app for drivers to update trip milestones and scan bills of lading.

---

## 👨‍💻 Author & Acknowledgments

**Darshan Rathod**
- **GitHub**: [@Darshan-38](https://github.com/Darshan-38)
- **LinkedIn**: [linkedin.com/in/rathod-darshan-6459bb2b8](https://www.linkedin.com/in/rathod-darshan-6459bb2b8/)

Developed as a modern transport operations solution demonstrating modular React architecture, full-stack database enforcement, and enterprise-grade authentication.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).

# Frontend Overview

The Lingtarn Forge frontend is a React 18 application that dynamically renders UI from DSL definitions.

## Technology Stack

| Component | Technology |
|-----------|-----------|
| Framework | React 18 |
| Language | TypeScript |
| Build Tool | Vite |
| Styling | Tailwind CSS |
| UI Components | shadcn/ui (Radix-based) |
| Routing | React Router v6 |
| State | React Context (AuthContext, AppContext, MenuContext, DepartmentContext) |

## Directory Structure

```
frontend/src/
├── main.tsx                    # App entry point
├── App.tsx                     # Root component with routing
├── api/
│   └── client.ts               # API client (auth, apps, data CRUD)
├── components/
│   ├── renderer/                # DSL-driven rendering engine
│   │   ├── Registry.tsx         # Field type → component mapping
│   │   ├── FormCanvas.tsx       # Dynamic form renderer
│   │   ├── ListCanvas.tsx       # Dynamic table renderer
│   │   ├── WorkflowCanvas.tsx   # Workflow state & transitions
│   │   ├── PluginCanvas.tsx     # Plugin rendering
│   │   └── EmbedPageCanvas.tsx  # Iframe embedding
│   ├── layout/                  # App shell
│   │   ├── Sidebar.tsx          # Navigation sidebar
│   │   ├── WorkbenchHeader.tsx  # Top header bar
│   │   ├── AiAssistant.tsx      # Chat assistant panel
│   │   ├── CommandPalette.tsx   # Ctrl+K command palette
│   │   ├── ProtectedRoute.tsx   # Auth guard
│   │   └── NotificationBell.tsx # Notification indicator
│   ├── forge/                   # AI Forge components
│   │   ├── DslPreviewCard.tsx   # DSL preview (read-only)
│   │   └── ChartPreview.tsx     # Chart rendering
│   └── ui/                      # Base UI primitives (shadcn)
│       ├── button.tsx, input.tsx, card.tsx, ...
├── contexts/
│   ├── AuthContext.tsx           # Token, user, login/logout
│   ├── AppContext.tsx            # App list, refreshApps
│   ├── MenuContext.tsx           # Menu refresh trigger
│   └── DepartmentContext.tsx     # Current department for data scoping
├── pages/
│   ├── AppDetail.tsx             # Generic DSL-rendered app page
│   ├── Workbench.tsx             # Admin dashboard
│   ├── Marketplace.tsx           # App marketplace
│   ├── Forge.tsx                 # AI Forge interface
│   ├── Login.tsx, Register.tsx   # Auth pages
│   └── ...
├── types/
│   └── dsl.ts                   # LingtarnDSL TypeScript interfaces
├── lib/
│   ├── dsl.ts                   # DSL parsing helpers
│   └── utils.ts                 # General utilities
└── plugins/
    └── registry.ts              # Frontend plugin registry
```

## Core Rendering Pipeline

The frontend's most important feature is its ability to render any DSL-defined app without app-specific code:

```
1. User navigates to /app/{appId}
        │
        ▼
2. AppDetail fetches app DSL via API
        │
        ▼
3. DSL is parsed by lib/dsl.ts helpers
   ├─ getSchemaRecords(dsl) → field definitions
   ├─ getLayoutList(dsl)    → list column config
   └─ getLayoutForm(dsl)    → form layout config
        │
        ▼
4. ListCanvas renders table view
   ├─ Columns from layout.list
   ├─ Cells rendered by Registry (type → component)
   ├─ Action buttons from logic.actions
   └─ Pagination, sorting, filtering
        │
        ▼
5. FormCanvas renders form dialog
   ├─ Grid layout from layout.form
   ├─ Inputs rendered by Registry (type → component)
   ├─ Validation from schema (required, etc.)
   └─ Save → POST/PATCH to Data API
        │
        ▼
6. WorkflowCanvas renders state & transitions
   ├─ State badge (current status)
   ├─ Transition buttons (allowed for current user)
   └─ History timeline
```

## State Management

No Redux or Zustand — the app uses React Context for global state:

| Context | Provides |
|---------|----------|
| `AuthContext` | `user`, `token`, `login()`, `logout()`, role helpers |
| `AppContext` | `apps[]`, `refreshApps()` |
| `MenuContext` | `menuRefreshKey`, `triggerMenuRefresh()` |
| `DepartmentContext` | `currentDepartmentId`, `setDepartmentId()` |

Local component state (`useState`, `useCallback`) handles page-level concerns.

## Detailed Documentation

- [DSL Renderer](dsl-renderer.md) — How the rendering engine works
- [Form Canvas](form-canvas.md) — Dynamic form rendering
- [List Canvas](list-canvas.md) — Dynamic table rendering

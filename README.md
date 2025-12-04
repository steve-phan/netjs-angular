# Learning Module Platform

A production-grade end-to-end feature for an Education-as-a-Service platform that allows learners to view learning modules, track their progress, and filter by category with real-time updates.

## Features Implemented ✅

### Backend (NestJS + TypeScript)

- **RESTful API** with query parameter validation
  - `GET /api/modules` — fetch modules with optional filtering
  - `PATCH /api/modules/:id` — update module completion status
- **Server-side filtering & pagination**
  - Filter by category (AI, Digital Skills, Sustainability)
  - Paginate results (configurable page size)
  - Validated query parameters (enum, integer, bounds checking)
- **Type-safe DTOs**
  - `LearningModuleQueryParamsDto` — validates query inputs with Transform decorators for edge-case coercion
  - `LearningModulesReponseDto` — consistent response structure `{ total, modules }`
- **Unit tests** — service and controller tests (Jest) with >90% coverage
- **Global ValidationPipe** with implicit type conversion and transformation

### Frontend (Angular 17 + RxJS)

- **Reactive service architecture**
  - Single source of truth (`BehaviorSubject` cache)
  - Optimistic UI updates — toggle completion status immediately, reconcile on server response
  - Automatic error recovery (reverts changes if request fails)
- **Filtering & sorting**
  - Category dropdown filter (server-side)
  - Automatic sorting by category order (AI → Digital Skills → Sustainability) + title
  - Real-time category options derived from all-modules cache
- **Pagination**
  - Page navigation component with bounds checking
  - Configurable page size (default: 10)
  - Total page count computed from server response
- **Progress tracking**
  - Completion summary (completed vs. total modules)
  - Real-time updates as modules are toggled
- **Component structure**
  - Modular standalone components (LearningModuleList, ModuleCard, Pagination, ProgressSummary, etc.)
  - Feature-folder pattern (`features/learning-modules/`)
  - Clean separation of concerns (presentation logic in components, data management in service)
- **Memory leak prevention**
  - `takeUntilDestroyed()` for manual subscriptions
  - `async` pipe for template subscriptions
- **Unit & E2E tests** — Karma/Chrome with HttpClientTestingModule

### Shared

- **Type safety** — TypeScript interfaces and enums across frontend and backend
- **Error handling** — typed errors (unknown → Error), proper HTTP exception responses
- **Validation** — enum-based category validation, numeric bounds checking

---

## Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/steve-phan/xu
   cd xu
   ```

2. **Install backend dependencies**

   ```bash
   cd backend
   yarn install
   ```

3. **Install frontend dependencies**
   ```bash
   cd ../frontend
   yarn install
   ```

---

## Running the Application

### Start the Backend (Port 8080)

```bash
cd backend
yarn start:dev
```

API will be available at `http://localhost:8080`

### Start the Frontend (Port 4200)

```bash
cd frontend
yarn start
```

Application will be available at `http://localhost:4200`

---

## Testing

### Backend Tests

```bash
cd backend
yarn test
```

Runs Jest tests for service and controller with coverage reporting.

### Frontend Tests

```bash
cd frontend
yarn test
```

Runs Karma tests with Chrome headless mode.

---

## Architecture Highlights

### Service Pattern (Single Source of Truth)

The `LearningModuleService` exposes:

- `modules$` — Observable of current module list (filtered & sorted)
- `categories$` — Observable of available category options
- `load(params)` — fetch fresh data and update cache
- `toggleCompletion(id)` — optimistic update with server reconciliation
- `getFilteredModules(params)` — delegates to load() for server-backed filtering

### Reactive Patterns Used

- **BehaviorSubject** — cache that holds the latest state for immediate subscribers
- **combineLatest** — derive categories$ from modules$ + static list
- **switchMap** — cancel in-flight requests when a new filter/page is selected
- **tap** — side effects (update cache, logging)
- **takeUntilDestroyed** — clean up subscriptions to prevent memory leaks
- **shareReplay** — share cached streams across multiple subscribers

### Optimistic Updates

1. User toggles a module → UI updates immediately (cache written)
2. PATCH request sent to backend in the background
3. If successful → no action (already cached)
4. If error → revert cache to previous state + show error toast

---

## Future Enhancements

- **Search functionality** — wire search-filter component to service (component scaffolded)
- **Authentication** — JWT-based auth with role-based access control
- **Rate limiting** — protect API from abuse
- **Persistent database** — replace in-memory seed with MongoDB/Prisma
- **Grouping UI** — render modules grouped by category for improved discoverability
- **Docker** — containerize backend and frontend for easy deployment
- **GraphQL** — consider for more flexible client-driven queries (future phase)

---

## Project Structure

```
xu/
├── backend/                 # NestJS API
│   ├── src/
│   │   └── app/
│   │       └── learning-module/
│   │           ├── controllers/
│   │           ├── services/
│   │           ├── repositories/
│   │           ├── dto/
│   │           ├── types/
│   │           └── data/
│   └── test/               # Jest tests
│
└── frontend/               # Angular 17 application
    └── src/
        └── app/
            ├── features/
            │   └── learning-modules/
            │       ├── models/
            │       ├── services/
            │       └── components/
            └── assets/
```

---

## Key Technologies

- **Backend**: NestJS, TypeScript, Jest, class-validator
- **Frontend**: Angular 17, RxJS, Karma, Chrome
- **Type Safety**: TypeScript (strict mode)
- **Testing**: Jest (backend), Karma/Chrome (frontend)
- **HTTP**: HttpClient (frontend), @nestjs/common (backend)

---
name: ddd-react-ole
description: Apply Domain Driven Design folder structure and architectural boundaries to a React + TypeScript project.
---

# Domain Driven Design for React

You are an expert in Domain Driven Design (DDD) applied to React + TypeScript frontend applications. Your role is to enforce a consistent folder structure and architectural boundaries.

## Key Principles

- **Domain is the core** — holds entities, types, and pure business logic. Zero dependencies on React, infrastructure, or any framework.
- **Infrastructure is the only layer** allowed to communicate with external APIs or services. It maps raw API responses to domain types.
- **Features are use-case-driven UI slices.** Each feature only imports from `domain` and `infrastructure`, never from other features directly.
- **Shared** holds truly reusable, generic UI and utilities with no business logic.
- Keep components small and focused. Pages orchestrate data fetching; components are purely presentational.
- Avoid barrel-file hell — prefer explicit imports over deeply nested `index.ts` re-exports.

## Base Structure

```
src/
├── domain/
│   └── {entity}/
│       ├── {entity}.types.ts       # Interfaces, enums, and type aliases
│       └── {entity}.helpers.ts     # Pure functions (no side effects)
│
├── infrastructure/
│   ├── http.ts                     # Base HTTP client (axios instance, fetch wrapper, etc.)
│   └── {entity}.service.ts     # API calls — returns domain types directly
│
├── features/
│   └── {useCase}/
│       ├── components/
│       │   └── {ComponentName}.tsx
│       ├── hooks/
│       │   └── use{UseCase}.ts     # Data fetching + state management for this feature
│       ├── pages/
│       │   └── {UseCase}Page.tsx   # Route-level component, composes hooks + components
│       └── {useCase}.types.ts      # UI-only types (form state, props) local to this feature
│
├── shared/
│   ├── components/
│   │   └── ui/                     # Generic UI: Button, Modal, Input, Spinner...
│   ├── hooks/
│   │   └── useDebounce.ts          # Generic reusable hooks
│   └── utils/
│       └── formatDate.ts           # Generic utility functions
│
├── router/
│   └── index.tsx                   # Route definitions (React Router, TanStack Router, etc.)
│
└── main.tsx
```

## Layer Rules

| Layer            | Can import from                      | Cannot import from              |
| ---------------- | ------------------------------------ | ------------------------------- |
| `domain`         | nothing                              | everything else                 |
| `infrastructure` | `domain` (freely)                    | `features`, `shared` components |
| `features`       | `domain`, `infrastructure`, `shared` | other `features`                |
| `shared`         | nothing (or `domain` types only)     | `features`, `infrastructure`    |

## Naming Conventions

| Thing        | Convention                                 | Example            |
| ------------ | ------------------------------------------ | ------------------ |
| Folders      | `kebab-case`                               | `agent-list/`      |
| Components   | `PascalCase.tsx`                           | `AgentCard.tsx`    |
| Other files  | `camelCase.ts`                             | `formatDate.ts`    |
| Hooks        | Prefixed with `use`                        | `useAgentList.ts`  |
| Services     | Suffixed with `.service.ts`                | `agent.service.ts` |
| Type files   | Suffixed with `.types.ts`                  | `agent.types.ts`   |
| Helper files | Suffixed with `.helpers.ts` or `.utils.ts` | `agent.helpers.ts` |
| Pages        | Suffixed with `Page`                       | `AgentPage.tsx`    |

## Feature Hook Pattern

Each feature exposes a dedicated hook that encapsulates data fetching and local state, keeping pages clean:

```ts
// features/agent/hooks/useAgent.ts
import { getAgents } from "@/infrastructure/agent/agent.service";
import { Agent } from "@/domain/agent/agent.types";

export const useAgent = () => {
  const [agents, setAgents] = useState<Agent[]>([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    setLoading(true);
    getAgents()
      .then(setAgents)
      .finally(() => setLoading(false));
  }, []);

  return { agents, loading };
};
```

## Dependencies

| Package                        | Purpose                                                |
| ------------------------------ | ------------------------------------------------------ |
| React + TypeScript             | Required                                               |
| React Router / TanStack Router | Routing                                                |
| TanStack Query or SWR          | Async state (recommended)                              |
| Tailwind CSS or Bootstrap      | Styling                                                |
| Axios or native fetch          | HTTP                                                   |
| Zod                            | Runtime type validation of API responses (recommended) |

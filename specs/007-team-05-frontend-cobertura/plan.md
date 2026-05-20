# Implementation Plan: 007-team-05-frontend-cobertura

**Branch**: `007-team-05-frontend-cobertura` | **Date**: 2026-05-20 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/007-team-05-frontend-cobertura/spec.md`

## Summary

Construir la web PWA que consuma los 4 endpoints REST de análisis institucional, estrategias de cobertura y Chat IA provistos por la API backend (TEAM-05). Esto incluye routing SPA con React Router, state management con `useSyncExternalStore` para persistencia del chat IA (mismo patrón que `signals.ts`), manejo de errores y escenarios de degradación, gráficos interactivos con `lightweight-charts`, y el diseño de la UI respetando las variables CSS existentes en el proyecto.

## Technical Context

**Language/Version**: TypeScript 5.x, React 18, HTML5, CSS Variables  
**Primary Dependencies**: Vite, React Router v6, lightweight-charts  
**Storage**: `useSyncExternalStore` (in-memory para chat session, mismo patrón que `signals.ts`)  
**Testing**: Vitest + React Testing Library (80% min coverage)  
**Target Platform**: Navegadores Desktop (Responsive) y PWA  
**Project Type**: Web Application / PWA frontend  
**Performance Goals**: Tiempo de carga inicial < 2s en entorno dev  
**Constraints**: fetch nativo (sin axios), degradación IA limpia, polling cada 2s máx 30s (15 intentos)  
**Scale/Scope**: 4 páginas nuevas (`/institutional/analysis`, `/institutional/positions`, `/coverage/strategies`, `/ai/chat`), 3 servicios API frontend, wrapper de layout

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- [x] **Idioma Oficial**: El plan, spec, UI y documentación están especificados en español técnico.
- [x] **Modelo Semi-Automático**: IA no ejecuta operaciones, solo recomienda estrategias de cobertura. Las pre-alertas de indisponibilidad del modelo reflejan la soberanía humana al dar fallbacks manuales.
- [x] **PWA - Stack Base Obligatorio**: Vite, React, TypeScript.
- [x] **Backend - Stack Base Obligatorio**: Consumiendo una REST API en Node.js/Express.

## UX Architecture & Control Strategy

- **Target Experience**: SPA de gestión financiera ágil y sin recargas de página. Multi-tabs (análisis, posiciones, estrategias, chat IA local a la sesión).
- **Critical Controls**: 
  - `Selectors`: Dropdowns cerrados para `period` y `horizon` en vez de inputs de texto para ajustarse al contrato.
  - `Comparador de Estrategias`: Uso intensivo de visualización a través de graficado 2D real mediante `lightweight-charts`, no solo numérico.
  - `Polling Indicator`: Spinner y timer para requests asíncronas extendidas del LLM.
- **State Strategy**: 
  - `Server State`: Consumido por REST con `fetch` en capas de servicio.
  - `Client State`: Store con `useSyncExternalStore` para historial local del chat de IA (mismo patrón que `src/store/signals.ts`, evitando instalar librerías externas), en `src/store/chat.ts`.
- **Performance Boundaries**: Polling delimitado a (máximo) 15 intentos (30 segundos), evitando saturación de conexiones. Carga inicial ágil.

## Data Source Routing & Runtime Modes

- **Source Domains**: `Institutional Analysis`, `Coverage Strategies`, `AI Institutional Chat`.
- **Routing Rules**: Redirigido localmente como `/api` a `http://localhost:3000` vía Vite proxy.
- **Runtime Modes**: PWA development mode (`npm run dev`) / Production mode.
- **Credential/Account Strategy**: Reúso estricto del helper `getAuthHeaders()` usando `localStorage('inversions.dev.token')` o la variable de entorno `VITE_DEV_BEARER_TOKEN`.

## Project Structure

### Documentation (this feature)
```text
specs/007-team-05-frontend-cobertura/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── contracts/
├── checklists/
└── tasks.md
```

### Source Code (repository root)

```text
projects/rest-api/inversions_api/src/
├── routes/
│   └── coverage/
│       └── coverageRouter.ts          # POST /api/coverage/analyze, /compare, /simulate

projects/pwa/inversions_app/
├── package.json
├── src/
│   ├── main.tsx
│   ├── layouts/
│   │   └── MainLayout.tsx
│   ├── pages/
│   │   ├── institutional/
│   │   │   ├── InstitutionalAnalysisPage.tsx
│   │   │   └── RegulatoryPositionsPage.tsx
│   │   ├── coverage/
│   │   │   └── CoverageStrategiesPage.tsx
│   │   └── ai/
│   │       └── AIChatPage.tsx
│   ├── services/
│   │   ├── institutional/
│   │   │   └── institutionalApi.ts
│   │   ├── coverage/
│   │   │   └── coverageApi.ts
│   │   └── ai/
│   │       └── aiChatApi.ts
│   ├── store/
│   │   └── chat.ts
│   └── components/
│       ├── coverage/
│       │   └── PayoffChart.tsx
│       └── ai/
│           ├── ChatHistory.tsx
│           └── ScenarioAnalysisCards.tsx
└── tests/
    ├── pages/
    ├── services/
    └── components/
```
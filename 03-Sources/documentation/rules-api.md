---
trigger:
  includes: ["src/services/**", "src/lib/axios.ts", "src/lib/constant/endpoints.ts", "src/pages/**/services/**"]
description: Rules for API integration, service layer, Axios, and security.
---

# API & Service Layer Rules

## Architecture & Flow
- **Data Flow**: `UI → Custom Hook → Service Layer → Axios Client → Backend API` (Mandatory: UI never calls backend directly).
- **Service Layer (`src/services/`)**: Typed API calls only. Framework-independent. No JSX, toast, navigation, or UI state.
- **Central Endpoints (`src/lib/constant/endpoints.ts`)**: All API paths centralized. Never hardcode URLs.

## Client, Auth & Errors
- **Axios Client (`src/lib/axios.ts`)**: Single gateway for HTTP requests, interceptors, tokens, signatures.
- **Authentication**: Auth state managed centrally via `useLocalStorage`. Never access `localStorage` directly. Never construct manual Bearer headers in components/services.
- **Error Normalization**: Process API errors through `src/lib/error-utils.ts`. Display user-friendly messages, never raw backend errors.
- **Security**: Never log sensitive data or expose API keys/secrets.

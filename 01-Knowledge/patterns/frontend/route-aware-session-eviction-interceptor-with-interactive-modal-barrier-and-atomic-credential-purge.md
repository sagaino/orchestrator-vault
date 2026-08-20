---
title: "Route-Aware Session Eviction Interceptor with Interactive Modal Barrier and Atomic Credential Purge"
type: pattern
tags: [pattern, frontend, auth, security, axios-interceptor, session-management, error-handling]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787127791371-cc78a949
sources: ["Harvest 1787127791371 Cc78a949.json"]
---

# Route-Aware Session Eviction Interceptor with Interactive Modal Barrier and Atomic Credential Purge

Route-Aware Session Eviction Interceptor with Interactive Modal Barrier and Atomic Credential Purge.

## 1. Overview & Architecture

An HTTP response interceptor pipeline that intercepts 401 Unauthorized responses, guards against infinite redirect loops by inspecting current location state, atomically purges authentication credentials from local storage, and displays a user-blocking modal barrier before evicting the session to the login page.

## 2. Implementation & Code Structure

src/
├── config/
│   ├── axios.ts             # Axios response interceptor with 401 eviction logic
│   └── constant/
│       └── localstorage.ts  # Session token & user metadata keys

## 3. Key Implementation Points

- Route-aware suppression guard detecting `/login` via pathname and hash to eliminate infinite reload loops.
- Atomic credential removal across token and user profile storage keys.
- Polymorphic response message parsing handling localized backend error objects (`msg_ind`) and raw strings.
- Modal barrier with outside-click and escape-key locks preventing user interaction with stale protected views.
- Hard window redirect ensuring memory sandbox and WebSocket connections are completely reset.

## 4. Code Examples

### Route-aware 401 response interceptor with SweetAlert modal barrier and atomic cache clearance

```typescript
import Axios from "axios";
import Swal from "sweetalert2";
import { LOCALSTORAGE_KEY } from "./constant/localstorage";

export const axios = Axios.create({
  baseURL: import.meta.env.VITE_API_URL || "http://localhost:3000",
});

axios.interceptors.response.use(
  (response) => response,
  async (error) => {
    const status = error.response?.status;
    const data = error.response?.data;

    if (status === 401) {
      const isLoginPage =
        window.location.pathname.includes("/login") ||
        window.location.hash.includes("/login");

      if (!isLoginPage) {
        // 1. Atomically purge expired session credentials
        localStorage.removeItem(LOCALSTORAGE_KEY.TOKEN);
        localStorage.removeItem(LOCALSTORAGE_KEY.USER);

        // 2. Extract polymorphic error message
        let msg = "Sesi telah kedaluwarsa.";
        if (data?.message) {
          msg =
            typeof data.message === "string"
              ? data.message
              : data.message.msg_ind || JSON.stringify(data.message);
        }

        // 3. Render blocking interactive modal barrier
        await Swal.fire({
          title: msg.split(",")[0],
          text: "Sesi telah kedaluwarsa.",
          icon: "warning",
          confirmButtonText: "Go to Login",
          allowOutsideClick: false,
          allowEscapeKey: false,
          confirmButtonColor: "#FFD700",
        });

        // 4. Force hard eviction redirect to clean up application state
        window.location.href = "/login";
      }
    } else {
      console.error("API Error:", data?.message || error.message || "An error occurred");
    }

    return Promise.reject(error);
  }
);

export default axios;
```

## 5. Considerations & Best Practices

- Multiple concurrent API requests failing with 401 simultaneously could trigger duplicate alert modals if not debounced with an active lock flag.
- Using `window.location.href` forces a full page reload, purging in-memory React state; this is safer for security than soft router push but loses transient form states.
- For seamless user experiences, an automatic token refresh queue (401 -> refreshToken -> retry original request) should be placed ahead of the hard eviction barrier.

## 6. Related Knowledge

- [[01-Knowledge/patterns/frontend/declarative-navigation-with-deep-link-state-preservation-guards.md]]
- Session Lifecycle And Token Eviction

## 7. Source

- Harvest 1787127791371 Cc78a949.json

---
title: "unified-session-expiration-guard-interceptor-auth-eviction-flow"
type: pattern
tags: [pattern, frontend, security, auth, session-management, http-interceptor, axios]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787132260416-42b894f9
sources: ["Harvest 1787132260416 42b894f9.json"]
---

# unified-session-expiration-guard-interceptor-auth-eviction-flow

Unified Session Expiration Guard & Interceptor-Driven Auth Eviction Flow

## 1. Overview & Architecture

Pola session management dan interceptor response untuk menangani 401 Unauthorized secara terpusat dengan pencegahan infinite alert loop, dialog penguncian interaksi (modal lock), pembersihan kredensial sesi secara atomik, dan pengalihan navigasi ke login.

## 2. Implementation & Code Structure

src/config/axios.ts (response interceptor 401 handler) -> src/hooks/useLogout.ts (client session purge hook) -> src/components/common/UserDropdown.tsx (UI trigger)

## 3. Key Implementation Points

- Route-aware 401 suppression to isolate login failures from expired authenticated sessions
- Atomic removal of security credentials (TOKEN & USER) upon unauthorized response
- Blocking modal dialog requiring explicit user acknowledgement before redirection
- Parity between programmatic interceptor session eviction and explicit user logout hooks

## 4. Code Examples

### Response Interceptor handling 401 Unauthorized with loop suppression and credential eviction

```typescript
// Add a response interceptor to handle errors
axios.interceptors.response.use(
  (response) => response,
  async (error) => {
    const status = error.response?.status;
    const data = error.response?.data;

    if (status === 401) {
      const isLoginPage = window.location.pathname.includes("/login") || window.location.hash.includes("/login");

      if (!isLoginPage) {
        localStorage.removeItem(LOCALSTORAGE_KEY.TOKEN);
        localStorage.removeItem(LOCALSTORAGE_KEY.USER);

        let msg = "Sesi telah kedaluwarsa.";
        if (data?.message) {
          msg = typeof data.message === "string" ? data.message : (data.message.msg_ind || JSON.stringify(data.message));
        }

        await Swal.fire({
          title: msg.split(",")[0],
          text: "Sesi telah kedaluwarsa.",
          icon: "warning",
          confirmButtonText: "Go to Login",
          allowOutsideClick: false,
          allowEscapeKey: false,
          confirmButtonColor: "#FFD700",
        });

        window.location.href = "/login";
      }
    } else {
      console.error("API Error:", data?.message || error.message || "An error occurred");
    }

    return Promise.reject(error);
  }
);
```

### Unified Session Purge Hook for User Actions and Interceptor Parity

```typescript
export const useLogout = () => {
  const navigate = useNavigate();
  const { t } = useTranslation();

  const handleLogout = useCallback(() => {
    // Clear the auth state from localStorage
    localStorage.removeItem(LOCALSTORAGE_KEY.TOKEN);
    localStorage.removeItem(LOCALSTORAGE_KEY.USER);
    // Navigate to login page
    navigate("/login", { replace: true });

    // Show success toast
    CustomToast.success(t("general.youre_logout"));
  }, [navigate, t]);

  return {
    handleLogout,
  };
}
```

## 5. Considerations & Best Practices

- Infinite loop guard: Checks current URL to prevent re-triggering 401 alerts when login attempt fails on /login route.
- Modal lockdown: SweetAlert2 modal sets allowOutsideClick: false and allowEscapeKey: false, forcing explicit user acknowledgement before navigation.
- Multi-lingual error fallback: Handles backend error messages whether returned as raw strings or localized objects (e.g. msg_ind).

## 6. Related Knowledge

- Session Management
- Auth Flow

## 7. Source

- Harvest 1787132260416 42b894f9.json

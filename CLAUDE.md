# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start        # Start development server (port 3000)
npm test         # Run tests in watch mode
npm run build    # Production build
```

To run a single test file:
```bash
npm test -- --testPathPattern=App.test
```

Linting runs automatically via `react-scripts` (ESLint with `react-app` preset). No separate lint command.

## Architecture

This is a React 19 SPA frontend that connects to a Spring Boot backend at `http://localhost:8080`.

**Tech stack:** React 19, React Router v7, Zustand 5, Axios

**API base URL** is configured via environment variables:
- `.env.development`: `http://localhost:8080`
- `.env.production`: `http://54.116.190.160:8080`

## Code Organization

### `src/api/`
- `Http.jsx` — Axios instance configured with `baseURL` from env and `withCredentials: true`
- `Auth.jsx` — Auth API calls (`register`, `login`, `logout`, `myPage`, `deleteMember`, `updateMember`) plus Axios request/response interceptors
- `GuestBook.jsx` — Guestbook API calls (CRUD operations)

**JWT auth flow:** Login response returns `{accessToken, refreshToken, user}`, stored in localStorage under key `tokens`. The request interceptor in `Auth.jsx` injects `Authorization: Bearer <token>` for all requests except `/members/login`, `/members/register`, and `/members/refresh`. The response interceptor handles 401 errors by refreshing the token and retrying failed requests.

### `src/store/`
Four Zustand stores:
- `useAuthStore.jsx` — `user`, `isLoggedIn` state; `zu_login()`, `zu_logout()` actions (no persistence, tokens managed directly in localStorage)
- `useTodoStore.jsx` — Todo CRUD, persisted to localStorage key `todo-storage`
- `useMemoStroe.jsx` — Memo CRUD, persisted to localStorage key `memo-storage` (note: filename typo "Stroe")
- `useGuestbookStore.jsx` — Guestbook state (no persistence, data fetched from backend)

### `src/components/`
- `PrivateRoute.jsx` — Route guard component that checks `isLoggedIn` (Zustand) OR presence of `tokens` in localStorage; redirects to `/login` if unauthorized

### `src/pages/`
Route-level components. Protected routes (`/memo`, `/profile`) are wrapped with `PrivateRoute` in `App.js`.

### `src/index.css`
Global dark-theme styles and utility classes (`.row`, `.col`, `.card`, `.muted`, `.empty`).

## Session Restoration

On page reload, `App.js` checks localStorage for `tokens` and restores login state to Zustand via `zu_login()`.

# AI-Powered RESTful API Server — Frontend

Front End of Term Assignment for COMP 4537 - AI Powered Classroom Web Application

React · Vite · React Router · Axios

---

## Quick Start

```bash
# 1. Install dependencies
npm install

# 2. Set up environment
cp .env.example .env
# Edit .env — fill in VITE_API_BASE_URL with the backend server URL

# 3. Start the dev server
npm run dev        # development (hot module reload)
npm run build      # production build → dist/
npm run preview    # preview the production build locally
```

The app starts on **http://localhost:5173** by default.

---

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `VITE_API_BASE_URL` | Base URL of the backend API — no trailing slash | `http://localhost:3000/api` |

> **Note:** All Vite environment variables must be prefixed with `VITE_` to be accessible in the browser.

---

## Connecting to the Backend

Open `.env` and set `VITE_API_BASE_URL` to wherever the backend is running:

```bash
# Local development (backend running on your machine)
VITE_API_BASE_URL=http://localhost:3000/api

# Deployed backend
VITE_API_BASE_URL=https://your-backend.azurewebsites.net/api
```

If no backend is available yet, see [Testing Without a Backend](#testing-without-a-backend) below.

---

## Project Structure

```
frontend/
├── index.html                   ← HTML entry point
├── vite.config.js               ← Vite + React plugin config
├── .env.example                 ← Copy to .env and fill in values
└── src/
    ├── main.jsx                 ← React root — mounts <App />
    ├── App.jsx                  ← Router setup, route definitions
    ├── models/                  ← OOP data classes (no framework deps)
    │   ├── User.js              ← class User — getters: remainingCalls, isAdmin, isLimitReached
    │   ├── AuthToken.js         ← class AuthToken — JWT decode, localStorage save/load/clear
    │   ├── ApiUsage.js          ← class ApiUsageRecord, ApiUsageSummary
    │   └── Classroom.js         ← class Question, StudentAnswer
    ├── services/                ← API service layer (inheritance)
    │   ├── BaseApiService.js    ← Abstract base: axios instance, interceptors, class ApiError
    │   ├── AuthService.js       ← extends BaseApiService — register, login, logout, restoreSession
    │   ├── ClassroomService.js  ← extends BaseApiService — questions, answers, AI chat, usage
    │   └── AdminService.js      ← extends BaseApiService — user management, usage summaries
    ├── hooks/
    │   ├── useAuth.jsx          ← AuthProvider context + useAuth() hook
    │   └── useApi.js            ← Generic async hook: { execute, data, loading, error }
    ├── components/
    │   └── common/
    │       ├── UI.jsx           ← Button, Input, Textarea, Alert, Card, UsageMeter, Badge, Spinner
    │       ├── Navbar.jsx       ← Top nav — renders differently for student vs. admin
    │       └── ProtectedRoute.jsx ← Auth + role guard for React Router
    ├── pages/
    │   ├── LandingPage.jsx      ← Public home page with hero, features, CTA
    │   ├── AuthPages.jsx        ← LoginPage + RegisterPage + class FormValidator
    │   ├── StudentDashboard.jsx ← Student view: questions, answer submission, AI chat
    │   └── AdminDashboard.jsx   ← Admin view: post questions, review answers, user table
    └── styles/
        └── global.css           ← Full design system — tokens, components, layouts
```

---

## Pages & Routes

| Route | Page | Access |
|-------|------|--------|
| `/` | Landing page | Public |
| `/login` | Login form | Public — redirects to dashboard if already logged in |
| `/register` | Registration form | Public — redirects to dashboard if already logged in |
| `/dashboard` | Student dashboard | Student only |
| `/admin` | Admin dashboard | Admin only |
| `*` | Catch-all | Redirects to `/` |

Authenticated users are redirected to their correct dashboard automatically on login. Accessing a route for the wrong role (e.g. a student hitting `/admin`) redirects back to their own dashboard.

---


**Models** — pure data classes with no React dependency

| Class | Responsibility |
|-------|---------------|
| `User` | Encapsulates user data; exposes `remainingCalls`, `usagePercentage`, `isAdmin`, `isLimitReached` as getters |
| `AuthToken` | Wraps a raw JWT; handles decode, expiry check, `save()` / `AuthToken.load()` / `AuthToken.clear()` |
| `ApiUsageSummary` | Aggregates usage records; exposes `remaining`, `percentage`, `isNearLimit`, `isAtLimit` |
| `Question` | Classroom question with `isOpen`, `isClosed`, `hasEvaluation`, `formattedDate` |
| `StudentAnswer` | Student response with `isGraded`, `scoreLabel`, `formattedDate` |

**Services** — HTTP layer using inheritance

```
BaseApiService          ← axios instance, interceptors, get/post/put/delete, ApiError
    ├── AuthService     ← register, login, logout, restoreSession
    ├── ClassroomService← questions, answers, AI ask, usage summary
    └── AdminService    ← getAllUsers, getAllUsageSummaries, resetUserUsage, deleteUser
```

Each service is exported as a singleton instance shared across the app.

**FormValidator** — static class in `AuthPages.jsx`

```js
FormValidator.validateLogin({ email, password })     // → { field: 'error message' }
FormValidator.validateRegister({ name, email, ... }) // → { field: 'error message' }
FormValidator.hasErrors(errorsObj)                   // → boolean
```

---

## Test Credentials

These match the credentials specified in the assignment deliverables:

| Role | Email | Password |
|------|-------|----------|
| Student | `john@john.com` | `123` |
| Admin | `admin@admin.com` | `111` |

---

## Testing Without a Backend

To test the UI before the backend is deployed, temporarily replace the `login` method in `src/services/AuthService.js`:

```js
async login(email, password) {
  // MOCK — remove when backend is ready
  const isAdmin = email.includes('admin');
  const user = new User({
    id: 1,
    name: isAdmin ? 'Admin User' : 'John Student',
    email,
    role: isAdmin ? 'admin' : 'student',
    apiCallsUsed: 3,
    apiCallsLimit: 20,
  });
  const fakeToken = new AuthToken(
    'eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwicm9sZSI6InVzZXIiLCJleHAiOjk5OTk5OTk5OTl9.mock'
  );
  fakeToken.save();
  this._saveUser(user);
  return { user, token: fakeToken };
}
```

Log in with `john@john.com` to see the student dashboard and `admin@admin.com` to see the admin dashboard. Revert this change before connecting to the real backend.

---

## Building for Production

```bash
npm run build
```

Output goes to `dist/`. Deploy this folder to any static host (Azure Static Web Apps, Vercel, Netlify, etc.).

> Make sure `VITE_API_BASE_URL` is set to the **deployed** backend URL before building — environment variables are baked in at build time.

---

## Deploying to Azure Static Web Apps

1. Push this repo to GitHub
2. In the Azure portal, create a **Static Web App** and link it to this repo
3. Set the build output location to `dist` and the app location to `/`
4. Add `VITE_API_BASE_URL` as an environment variable in the Azure portal under **Configuration**
5. Azure will auto-deploy on every push to `main`

Make sure the backend has CORS configured to allow requests from the frontend's Azure domain.
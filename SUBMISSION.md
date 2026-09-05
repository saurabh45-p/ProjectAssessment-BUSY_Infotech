# Submission — codevolveX

> This is the first file a reviewer opens. Fill in every placeholder and commit before the deadline.

---

## Links

| | URL |
|---|---|
| **GitHub repository** | https://github.com/saurabh45-p/ProjectAssessment-BUSY_Infotech |
| **Live application** | `https://codevolvx.vercel.app` |
|

---

## Notes for the reviewer

The backend is hosted on Render's free tier — the first request after inactivity can take **30–60 seconds** while the container wakes up. After that initial cold start, all subsequent requests respond normally.



---

## Demo credentials

| Role | Email | Password |
|------|-------|----------|
| Student | *(add demo email)* | *(add demo password)* |
| Instructor | *(add demo email)* | *(add demo password)* |
| Admin | *(add demo email)* | *(add demo password)* |

---

## Stack

| Layer | What was used | Why |
|-------|--------------|-----|
| **Frontend framework** | React 18 + Vite 6 | Fast HMR during development; optimised production bundles via Rollup |
| **Routing** | React Router v7 | Declarative nested routes, protected route wrappers, loader-based data fetching |
| **State management** | Redux Toolkit 2 | Centralised, predictable state for auth, cart, and course progress slices |
| **Styling** | Tailwind CSS v4 | Utility-first; co-located styles remove the need for separate CSS files |
| **Animation** | Framer Motion 12 | Physics-based spring animations; `AnimatePresence` for page transitions |
| **Charts** | Recharts 3 + MUI X Charts | Instructor revenue and enrolment analytics dashboards |
| **Video player** | video-react | Lightweight HTML5 player with seek and progress callbacks |
| **Forms** | React Hook Form 7 | Uncontrolled inputs; built-in validation reduces re-renders |
| **Smooth scroll** | Lenis 1 | Momentum-based scroll on the landing page |
| **HTTP client** | Axios 1 | Interceptors for attaching JWT and centralising error handling |
| **Backend runtime** | Node.js ≥18 + Express 4 (ES Modules) | Non-blocking I/O; ES Module syntax kept parity with the frontend |
| **Database** | MongoDB Atlas + Mongoose 8 | Flexible document model suited to a nested course → section → sub-section hierarchy |
| **Authentication** | JWT (jsonwebtoken 9) + bcrypt 5 | Stateless auth; bcrypt with a cost factor of 10 for password hashing |
| **File uploads** | Multer 1 (disk / memory storage) | Parses `multipart/form-data` before handing files to Cloudinary |
| **Media storage** | Cloudinary SDK 2 | Automatic video transcoding, adaptive streaming, and CDN delivery |
| **Payments** | Razorpay Node SDK 2 | Order creation → checkout → HMAC signature verification on the server |
| **Email / OTP** | Brevo (`@getbrevo/brevo`) + Nodemailer fallback | Transactional email for OTP verification and password-reset links |
| **Dev tooling** | Nodemon 3, Concurrently 9, ESLint 9 | Hot reload, parallel scripts, and consistent code style |
| **Frontend hosting** | Vercel | Zero-config deployment; `vercel.json` rewrites all paths to `index.html` for SPA routing |
| **Backend hosting** | Render (Node.js web service) | Free tier; auto-deploys from the `main` branch |
| **Database hosting** | MongoDB Atlas (shared M0) | Managed backups, IP allowlist, connection pooling |

---

## Goal checklist

| # | Goal | Status | Notes |
|---|------|--------|-------|
| 1 | User registration, login, and JWT-based auth | ✅ Done | OTP email verification on signup; httpOnly cookie + Bearer token both handled |
| 2 | Role-based access control (Student / Instructor / Admin) | ✅ Done | `auth` + `isStudent` / `isInstructor` middleware on all protected routes; UI routes guarded via `PrivateRoute` and `accountType` checks |
| 3 | Course creation, section & sub-section management | ✅ Done | Three-step wizard: metadata → curriculum → publish; inline editing of sections and sub-sections |
| 4 | Video upload and Cloudinary streaming | ✅ Done | Multer streams file buffer → Cloudinary; signed URLs returned to player |
| 5 | Student enrolment and course progress tracking | ✅ Done | `CourseProgress` document per student; sub-section completion toggled from the video player |
| 6 | Secure payment flow (Razorpay) | ✅ Done | Server creates order → client opens checkout → server verifies HMAC signature before enrolment |
| 7 | Instructor analytics dashboard | ✅ Done | Revenue and enrolment charts (Recharts + MUI X Charts); per-course breakdown |
| 8 | Course ratings and reviews | ✅ Done | One review per student per course; average rating computed server-side |
| 9 | OTP-based password reset | ✅ Done | Time-limited OTP stored in `OtpGenerator` collection; reset link emailed via Brevo |
| 10 | Responsive, accessible UI | ✅ Done | Mobile-first Tailwind layout is responsive; ARIA attributes and keyboard navigation are incomplete on some modal components |

---

## How much time did you actually spend?

| Phase | Hours |
|-------|-------|
| Planning, schema design, and architecture | ~2 h |
| Backend API (auth, courses, payments, email) | ~10 h |
| Frontend — core pages, auth, and routing | ~6 h |
| Frontend — dashboard, analytics, course player | ~7 h |
| Integration, debugging, and end-to-end testing | ~3 h |
| Deployment and environment configuration | ~1 h |
| Documentation and this submission | ~1 h |
| **Total** | **~30 h** |

---

## What would you do next, with another 12 hours?

1. **Admin panel** — the `Admin` role exists in the schema and JWT guard but the dashboard UI is not built yet. I would implement course and user moderation views.
2. **Course search and filtering** — the catalogue currently lists all courses. Adding full-text search (MongoDB Atlas Search) and filter-by-category would make discovery practical at scale.
3. **ARIA and keyboard accessibility** — close modals on `Escape`, trap focus in dialogs, and add `aria-label` to all icon-only buttons.
4. **Automated tests** — write Jest unit tests for the payment verification logic and Playwright e2e tests for the enrolment flow.

---

## What are you least happy with in this codebase, and why?

**Error handling consistency on the backend.** Most controllers wrap logic in `try/catch` but there is no centralised Express error-handling middleware. This means error response shapes vary across routes (`{ success, message }` vs raw status codes), which makes the frontend error-handling code messier than it needs to be. I would introduce a single `errorHandler(err, req, res, next)` middleware and a custom `AppError` class so every controller only needs to `throw` or call `next(err)`.

---

*Submitted by — SAURABH PATEL · 04/09/2026*

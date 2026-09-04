# Key Technical Decisions — codevolveX

Each entry records: the decision, the alternatives considered, why this option was chosen, and any trade-offs accepted.

---

## 1. Monorepo with Concurrently

**Decision:** Keep `frontend/` and `server/` in a single repository with a root `package.json` that runs both via `concurrently`.

**Alternatives considered:**
- Separate repos — cleaner separation but requires managing two GitHub repos, two deployment pipelines, and cross-repo co-ordination for every shared env-var change.
- Turborepo / Nx — powerful but excessive setup overhead for a two-package project.

**Why this:** One `git clone`, one `npm run dev`, one PR. The added complexity of a monorepo tool is not justified at this scale.

**Trade-offs:** Root `node_modules` installs packages that only one workspace needs. CI pipelines need to be explicit about which workspace to build.

---

## 2. ES Modules on the Backend

**Decision:** Use `"type": "module"` in `server/package.json`, write all server files with `import/export`.

**Alternatives considered:**
- CommonJS (`require`) — default Node.js; no config needed but inconsistent with the frontend.
- TypeScript — adds type safety but increases setup time and compilation step.

**Why this:** ES Modules kept the codebase visually consistent with the Vite/React frontend. Node.js ≥18 supports ESM natively without a build step.

**Trade-offs:** Some older npm packages (e.g. `fs` shim installed unnecessarily) behave differently. `__dirname` is not available — replaced with `import.meta.url` patterns where needed.

---

## 3. JWT Delivered as Both Cookie and Body Token

**Decision:** Login sets an `httpOnly` cookie and also returns the token in the response body. The `auth` middleware reads from `req.cookies.token || req.body.token || Authorization Bearer`.

**Alternatives considered:**
- Cookie only — more secure (XSS-resistant) but harder to test with Postman / Thunder Client.
- Body only — easier to develop against but exposes token to XSS if stored in `localStorage`.
- Session-based auth — stateful; harder to scale horizontally.

**Why this:** The dual approach let the frontend store the token in Redux (for convenience during development) while the production browser uses the cookie automatically. The `auth` middleware handles both transparently.

**Trade-offs:** The body-token fallback weakens the security model slightly. A production hardening pass should remove the body-token path and force cookie-only.

---

## 4. Cloudinary via Multer Disk Storage (Temp File Pattern)

**Decision:** Multer writes uploads to `server/public/temp/`. `uploadOnCloudinary` reads the file, uploads it, then `fs.unlink` removes the local copy.

**Alternatives considered:**
- Multer `memoryStorage` with a Buffer — avoids writing to disk; simpler cleanup. But large video files (hundreds of MB) would exhaust RAM on a free-tier host.
- Direct client-to-Cloudinary upload — removes the server as a middleman; faster. But requires exposing Cloudinary credentials to the browser, which is a security risk.
- AWS S3 — more control; higher complexity and cost.

**Why this:** Disk storage keeps memory consumption flat regardless of file size. The temp file is always cleaned up, whether the upload succeeds or fails (the `catch` block also calls `fs.unlink`).

**Trade-offs:** Render's free tier has ephemeral disk — temp files survive only for the lifetime of the request (which is fine, as they are deleted synchronously within it). On a multi-instance deployment, the temp directory would need to be a shared volume.

---

## 5. Razorpay Server-Side Order + HMAC Verification

**Decision:** Payment flow is two-step: (1) server creates Razorpay order and returns `order_id`; (2) after client-side checkout, server re-derives the HMAC-SHA256 signature and compares it with the one Razorpay sent before enrolling the student.

**Alternatives considered:**
- Trust the client — client sends a "payment succeeded" flag; server enrols immediately. Completely insecure; bypassed by any HTTP client.
- Razorpay webhook — event-driven; more reliable for async payment outcomes. Requires a publicly accessible endpoint and webhook secret configuration.

**Why this:** The synchronous verify-then-enrol approach is simpler to implement and debug. The HMAC check provides cryptographic proof that the payment was genuine. Webhooks were deferred as a future improvement.

**Trade-offs:** If the client crashes between checkout completion and the `verifyPayment` POST, the student's payment is captured by Razorpay but they are not enrolled. A reconciliation cron job or webhook fallback would fix this.

---

## 6. Redux Toolkit for All Client State

**Decision:** Use `@reduxjs/toolkit` with five slices: `auth`, `profile`, `cart`, `course`, `viewCourse`.

**Alternatives considered:**
- React Context + `useReducer` — built-in; no extra dependency. Gets unwieldy across 17+ pages.
- Zustand — minimal boilerplate; good DX. Less ecosystem tooling (DevTools, middleware) out of the box.
- React Query / TanStack Query — excellent for server state (caching, background refetch). Would replace most slice logic for async data but requires a mental model shift.

**Why this:** Redux DevTools make debugging state mutations across the cart, progress, and auth flows straightforward. The team had prior familiarity with RTK, making it the lowest-risk choice.

**Trade-offs:** Boilerplate is higher than Zustand. Async actions use `createAsyncThunk` which is verbose. For a future rewrite, replacing server-state slices with React Query is recommended.

---

## 7. `react-hook-form` for the Course Wizard

**Decision:** Use `react-hook-form` for the three-step AddCourse wizard instead of controlled `useState` fields.

**Alternatives considered:**
- Formik — popular, but `react-hook-form` has better performance (uncontrolled inputs) and a smaller bundle size.
- Plain `useState` — sufficient for simple forms but becomes unwieldy with 10+ fields, nested arrays (tags, instructions), and cross-step validation.

**Why this:** Uncontrolled inputs mean React does not re-render the entire wizard on every keystroke. The `watch()` API enables dynamic field display without state coupling.

**Trade-offs:** Integration with custom UI components (dropzone, OTP input, star rating) requires wrapping them in `Controller`. This adds boilerplate for non-standard inputs.

---

## 8. Tailwind CSS v4 + Framer Motion (No Component Library for Core Layout)

**Decision:** Use Tailwind for all layout and styling. Framer Motion for animations. MUI is used only for the data-grid and charts in the instructor dashboard.

**Alternatives considered:**
- Full MUI — faster to scaffold but opinionated styling conflicts with a custom design system.
- Chakra UI / shadcn — good DX but adds a dependency layer on top of Tailwind.
- Plain CSS modules — maximum control but slowest to write.

**Why this:** Tailwind v4 with the Vite plugin offers the fastest iteration loop for utility-first styling. Framer Motion provides physics-based animations that would be complex to replicate in CSS alone. MUI's data grid was used pragmatically for the instructor course table because building a sortable, paginated table from scratch was out of scope.

**Trade-offs:** MUI and Tailwind CSS-in-JS can conflict on specificity. Care was taken to isolate MUI components in the dashboard so their styles do not bleed into Tailwind-styled pages.

---

## 9. Brevo (`@getbrevo/brevo`) for Transactional Email

**Decision:** Use the official Brevo SDK as the primary transactional email provider.

**Alternatives considered:**
- Nodemailer + SMTP (Gmail) — free but hits Gmail's 500/day send limit quickly; app passwords are fragile.
- Resend — modern DX; the `resend` package is present in `server/package.json` but is unused.
- SendGrid — industry standard but requires domain verification for production.

**Why this:** Brevo offers a generous free tier (300 emails/day), a clean SDK, and does not require DNS changes on a custom domain for sandbox testing.

**Trade-offs:** The `resend` package is a dead dependency — it was explored but abandoned in favour of Brevo. Should be removed from `package.json` to reduce install size.

---

## 10. No Centralised Error-Handling Middleware

**Decision (by omission):** Each controller has its own `try/catch` that calls `res.status(xxx).json(...)` directly.

**Alternatives considered:**
- Express `(err, req, res, next)` error handler + custom `AppError` class — standard production pattern.
- `express-async-errors` package — patches Express to catch thrown errors automatically without needing `next(err)`.

**Why this happened:** Incremental development — each controller was written in isolation without establishing a global error contract first.

**Impact:** Error response shapes vary across routes (`{ success, message }` vs `{ error }` vs raw status codes). The frontend's Axios error handler has to special-case multiple shapes.

**Future fix:** Introduce `utils/AppError.js`, wrap controllers with `asyncHandler`, add a single `app.use(errorHandler)` at the bottom of `index.js`.

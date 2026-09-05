# AI Prompts Log — codevolveX

This file logs every meaningful prompt given to an AI tool during development, the output received, and how it was used.

---

## Summary

| Metric | Value |
|---|---|
| AI tools used | ChatGPT-4o, GitHub Copilot (inline) |
| Total prompts logged | 9 |
| Output used as-is | 2 |
| Output modified before use | 6 |
| Output discarded | 1 |
| Estimated % of codebase AI-assisted | ~25% |

---

## Prompt Log

---

### Entry 1 — OTP TTL Schema

| Field | Value |
|---|---|
| **Date** | 2026-09-01 |
| **Tool** | ChatGPT-4o |
| **Phase** | Auth — OTP verification |
| **Prompt** | `I'm building an OTP verification flow in Mongoose. I want the OTP document to expire automatically after about 4 hours without a cron job. Show me the schema and explain how MongoDB handles TTL indexes.` |
| **Output summary** | Generated a Mongoose schema using `expires` on the `createdAt` field and explained that MongoDB's TTL monitor runs every 60 seconds, so exact expiry is approximate. Also included a `pre("save")` hook pattern for triggering side effects on creation. |
| **Used?** | Modified |
| **What was changed** | Swapped the `pre("save")` example (which used nodemailer directly) to call our own `mailSender` utility. Changed the `expires` value from `300` (5 min) to `15000` (~4 h) to match the project requirement. |
| **File(s) affected** | `server/models/OtpGenerator.model.js` |

---

### Entry 2 — Razorpay HMAC Verification

| Field | Value |
|---|---|
| **Date** | 2026-09-02 |
| **Tool** | ChatGPT-4o |
| **Phase** | Payment — signature verification |
| **Prompt** | `Write a Node.js Express controller that verifies a Razorpay payment signature using HMAC-SHA256. The inputs are razorpay_order_id, razorpay_payment_id, and razorpay_signature. Show how to construct the body string and compare the digest.` |
| **Output summary** | Returned a concise code snippet using Node's built-in `crypto` module: `crypto.createHmac('sha256', secret).update(order_id + '|' + payment_id).digest('hex')` and a strict equality check against the incoming signature. |
| **Used?** | Modified |
| **What was changed** | The generated snippet returned a `200` with `success: false` on failure — kept that pattern. Added the `enrollStudents()` call, `Order.create()`, and `sendPaymentSuccessEmail()` pipeline after a passing verification. The original snippet had no post-verification logic. |
| **File(s) affected** | `server/controller/payment.js` |

---

### Entry 3 — Instructor Analytics Aggregation Pipeline

| Field | Value |
|---|---|
| **Date** | 2026-09-03 |
| **Tool** | ChatGPT-4o |
| **Phase** | Analytics — instructor dashboard |
| **Prompt** | `I have an Order collection where each document has a 'courses' array (ObjectIds) and an 'amount' field. An order can contain courses from multiple instructors. Write a MongoDB aggregation that computes per-course revenue for a specific instructor, splitting the order amount evenly across all courses in that order.` |
| **Output summary** | Proposed a pipeline using `$unwind` on the courses array, `$lookup` to join with the Courses collection, and `$group` to sum revenue per course with the split amount. Used `$divide` to split. |
| **Used?** | Discarded |
| **What was changed** | The aggregation assumed a single `$lookup` would filter by instructor, but our schema keeps instructor on the Course document, not the Order. Rewrote the logic in plain JavaScript: fetched the instructor's course IDs first, then queried `Order.find({ courses: { $in: courseIds } })` and computed the revenue split in a `forEach` loop. This was simpler to debug and test. The AI-generated pipeline produced incorrect results when a single order contained courses from different instructors. |
| **File(s) affected** | `server/controller/course.js` — `instructorDashboard` function |

---

### Entry 4 — Cloudinary Upload Utility

| Field | Value |
|---|---|
| **Date** | 2026-09-01 |
| **Tool** | ChatGPT-4o |
| **Phase** | Media — file upload |
| **Prompt** | `Write a Node.js utility function that takes a local file path, uploads it to Cloudinary using the v2 SDK with resource_type auto, deletes the local temp file after upload (success or failure), and returns the Cloudinary response object.` |
| **Output summary** | Generated the `uploadOnCloudinary` function with `try/catch`, cleanup in both branches, and `cloudinary.uploader.upload()`. |
| **Used?** | Modified |
| **What was changed** | Added `await fs.access(localFilePath)` before upload to guard against Multer saving a zero-byte file on validation error. Also added `.replace(/\\\\/g, '/')` to the file path string to fix a Windows-specific backslash bug discovered during local testing. |
| **File(s) affected** | `server/utils/cloudinary.js` |

---

### Entry 5 — Email Verification Template (HTML)

| Field | Value |
|---|---|
| **Date** | 2026-09-02 |
| **Tool** | ChatGPT-4o |
| **Phase** | Auth — transactional email |
| **Prompt** | `Write a clean HTML email template function for OTP verification. It should accept an OTP string and return an HTML string. Use inline CSS only (no external stylesheets). Brand name: codevolveX. Primary colour: dark navy. Keep it under 30 lines.` |
| **Output summary** | Returned a self-contained HTML function with a centred card layout, large OTP display, and brand footer. Used inline styles for compatibility with email clients. |
| **Used?** | Modified |
| **What was changed** | Adjusted brand colours to match the Tailwind theme used in the frontend. Added an expiry note ("This OTP expires in 4 hours"). Changed the font stack to `Arial, sans-serif` for maximum email client support. |
| **File(s) affected** | `server/mail/templates/emailVerificationTemplate.js` |

---

### Entry 6 — Course Enrollment Email Template

| Field | Value |
|---|---|
| **Date** | 2026-09-02 |
| **Tool** | ChatGPT-4o |
| **Phase** | Payment — post-enrolment email |
| **Prompt** | `Write a JavaScript function that returns an HTML email string for a course enrolment confirmation. Parameters: courseName (string), studentName (string). Should feel warm and professional.` |
| **Output summary** | Generated a two-paragraph template with a greeting, course name in bold, and a CTA button linking to the dashboard. |
| **Used?** | As-is |
| **What was changed** | None. Template was used directly. The CTA button href was set to the live Vercel URL. |
| **File(s) affected** | `server/mail/templates/courseEnrollmentEmail.js` |

---

### Entry 7 — Redux Cart Slice

| Field | Value |
|---|---|
| **Date** | 2026-09-04 |
| **Tool** | GitHub Copilot (inline) |
| **Phase** | Frontend — state management |
| **Prompt** | *(Inline autocomplete — no explicit prompt. Copilot suggested the full `addToCart`, `removeFromCart`, and `resetCart` reducers after the slice boilerplate was typed.)* |
| **Output summary** | Suggested all three reducers with `localStorage.setItem` side effects inside each case to persist the cart across page refreshes. |
| **Used?** | Modified |
| **What was changed** | The Copilot suggestion used `JSON.stringify(state)` on the whole state object, which serialises the entire slice including metadata. Changed to persist only `state.cart` (the items array) and `state.total`. Added a guard against duplicate course IDs before pushing to the cart array. |
| **File(s) affected** | `frontend/src/slices/cart.slice.js` |

---

### Entry 8 — PrivateRoute Component

| Field | Value |
|---|---|
| **Date** | 2026-09-03 |
| **Tool** | ChatGPT-4o |
| **Phase** | Frontend — routing |
| **Prompt** | `Write a React PrivateRoute component using react-router-dom v6+ that reads the auth token from a Redux store and redirects to /login if the user is not authenticated.` |
| **Output summary** | Generated a wrapper component using `useSelector` to read `state.auth.token` and `<Navigate to="/login" replace />` when null. |
| **Used?** | As-is |
| **What was changed** | None. The generated component matched the project's Redux slice structure exactly. |
| **File(s) affected** | `frontend/src/components/core/auth/PrivateRoute.jsx` |

---

### Entry 9 — Payment Success Email Template

| Field | Value |
|---|---|
| **Date** | 2026-09-04 |
| **Tool** | GitHub Copilot (inline) |
| **Phase** | Payment — receipt email |
| **Prompt** | *(Inline autocomplete — triggered after typing the function signature `paymentSuccessEmail(studentName, amount, orderId, paymentId)`.)* |
| **Output summary** | Suggested an HTML template listing all four parameters in a receipt-style table. |
| **Used?** | Modified |
| **What was changed** | Reformatted the amount display to divide by 100 at the template level and append "₹" currency symbol. Added Razorpay order ID and payment ID in monospace font for easy copy-paste during support queries. |
| **File(s) affected** | `server/mail/templates/payementSuccessEmail.js` |

---

## Copilot / Inline Autocomplete

| Tool | Which files | How heavily relied on |
|---|---|---|
| GitHub Copilot | Controllers, Redux slices, email templates | Accepted roughly 15–20% of suggestions; most were used as a starting point and revised |
| ChatGPT-4o | Architecture decisions, complex algorithms, email HTML | Used for targeted problem-solving; always reviewed and tested before committing |

---

## Reflections

### Did AI assistance speed up development?

Yes — meaningfully for boilerplate-heavy work. The OTP schema with TTL, the HMAC verification snippet, and the PrivateRoute component each saved 20–40 minutes of documentation reading. HTML email templates (which require inline CSS and cross-client quirks) were the single biggest time save: what would have taken 2–3 hours of trial-and-error was scaffolded in minutes and then refined.

### Where did AI output require significant correction?

The instructor dashboard revenue aggregation (Entry 3) was the most expensive failure. The pipeline looked plausible but was semantically wrong — it could not correctly split revenue when a single order spanned courses from multiple instructors. Debugging it took longer than writing the plain-JavaScript alternative from scratch. The lesson: aggregation pipelines are hard to get right with a prompt alone; always validate against real data.

The Cloudinary utility (Entry 4) also had a subtle Windows-only bug with backslash path separators that the AI had no way to anticipate. Local testing on Windows caught it before deployment.

### Would you use AI assistance differently on the next project?

Yes — two changes:

1. **Use AI for scaffolding, not for logic.** Let it generate the function signature, type hints, and happy-path skeleton. Write the error handling, edge cases, and business logic manually.
2. **Prompt with constraints.** Vague prompts produce generic code. Adding "this runs on Windows with Node 18 ES Modules and Mongoose 8" to every backend prompt produced noticeably more accurate output.

# Work Plan — codevolveX

This document records the actual build sequence and time allocation used during development.

---

## Phase Overview

| # | Phase | Goal | Est. Hours | Actual Hours |
|---|---|---|---|---|
| 1 | Foundation | Repo setup, env config, DB connection, Express skeleton | 1 | ~1 |
| 2 | Auth system | Signup/OTP/login/JWT/reset-password | 2 | ~3 |
| 3 | Course data layer | All Mongoose models + relations | 1 | ~1 |
| 4 | Course CRUD API | Create / Read / Update / Delete + Cloudinary upload | 3 | ~4 |
| 5 | Payment API | Razorpay order + verify + enrolment + email | 2 | ~2 |
| 6 | Analytics API | Instructor dashboard aggregation pipeline | 1 | ~1 |
| 7 | Frontend foundation | Vite setup, Tailwind, Redux store, Axios, React Router | 1 | ~1 |
| 8 | Auth UI | Signup, login, OTP verify, forgot / reset password flows | 2 | ~3 |
| 9 | Student UI | Home, Catalogue, CourseDetails, Cart, Checkout, ViewCourse | 4 | ~4 |
| 10 | Instructor UI | AddCourse wizard, MyCourses, EditCourse, InstructorDashboard | 3 | ~3 |
| 11 | Shared UI | Navbar, Footer, Settings, MyProfile, PurchaseHistory | 2 | ~2 |
| 12 | Integration & debugging | End-to-end testing, cross-origin fixes, env alignment | 2 | ~3 |
| 13 | Deployment | Vercel (frontend) + Render (backend) + Atlas IP allowlist | 1 | ~1 |
| 14 | Documentation | README, SUBMISSION.md, docs/ | 1 | ~1 |
| **Total** | | | **26 h** | **~30 h** |

---

## Detailed Task Breakdown

### Phase 1 — Foundation
- [x] Initialise mono-repo: `root/`, `server/`, `frontend/`
- [x] Root `package.json` with `concurrently` dev script
- [x] `server/.env` template (PORT, BASE_URL, JWT_SECRET, Cloudinary, Razorpay, Brevo)
- [x] `frontend/.env` template (VITE_BASE_URL, VITE_RAZORPAY_KEY)
- [x] MongoDB Atlas cluster + IP allowlist + connection string
- [x] `server/config/database.js` — Mongoose connect with retry log
- [x] Express app skeleton with CORS, bodyParser, cookieParser

---

### Phase 2 — Authentication
- [x] `OtpGenerator` model with TTL index (15 000 s) and `pre("save")` email hook
- [x] `sendOTP` controller — generate 6-digit numeric OTP, collision-check, persist
- [x] `signUp` controller — validate OTP, bcrypt hash, create Profile + User atomically
- [x] `login` controller — bcrypt compare, jwt.sign (24 h), httpOnly cookie + body token
- [x] `changePassword` controller — verify old password, re-hash, send confirmation email
- [x] `resetPasswordToken` — `crypto.randomUUID()`, store in User.token, email link
- [x] `resetPassword` — verify token + expiry, bcrypt update
- [x] `auth` middleware — reads cookie / Bearer / body token, `jwt.verify`
- [x] `isStudent`, `isInstructor`, `isAdmin` role guards

---

### Phase 3 — Data Layer
- [x] `User` model (10 fields + timestamps)
- [x] `Profile` model (4 optional fields + timestamps)
- [x] `Course` model (Draft/Published status, instructor ref, studentsEnrolled array)
- [x] `Section` model → SubSection ref array
- [x] `SubSection` model (videoUrl, timeDuration as string)
- [x] `CourseProgress` model (completedVideo array of SubSection refs)
- [x] `RatingsAndReview` model (course index for aggregation)
- [x] `Category` model (admin-seeded)
- [x] `Order` model (Razorpay IDs, amount, status enum)

---

### Phase 4 — Course API
- [x] Multer middleware — `diskStorage` for images and videos to `server/public/temp/`
- [x] `uploadOnCloudinary` utility — upload + delete local temp file
- [x] `createCourse` — Multer → Cloudinary thumbnail → Course.create → update Instructor + Category
- [x] `showAllCourses` — filter `status: "Published"`, populate instructor + category
- [x] `getCourseDetails` — deep populate (instructor → profile, sections → subsections minus videoUrl)
- [x] `getFullCourseDetails` — same but includes videoUrl + CourseProgress for enrolled student
- [x] `editCourse` — patch any field, replace thumbnail via Cloudinary
- [x] `deleteCourse` — cascade delete sections, subsections; pull from instructor + enrolled students
- [x] `getInstructorCourses` — filter by instructor ID
- [x] Section CRUD (`createSection`, `updateSection`, `deleteSection`)
- [x] SubSection CRUD (`createSubSection`, `updateSubSection`, `deleteSubSection`) + video upload
- [x] Category endpoints (`createCategory` Admin-only, `showAllCategory`, `categoryPageDetails`)
- [x] Ratings endpoints (`createRating` Student-only, `averageRatings`, `gettingAllRatings`, `reviewsAndRatingForCourse`)
- [x] `updateCourseProgress` — toggle SubSection ID in CourseProgress.completedVideo

---

### Phase 5 — Payment API
- [x] Razorpay SDK instance in `config/razorpay.js`
- [x] `capturePayment` — validate cart IDs, sum prices, check not already enrolled, create Razorpay order
- [x] `verifySignature` — HMAC-SHA256 verification → `enrollStudents` → `Order.create`
- [x] `enrollStudents` — atomic updates: Course.$push(studentId), CourseProgress.create, User.$push(courseId, progressId)
- [x] `sendPaymentSuccessEmail` — receipt email via Brevo
- [x] `getPurchaseHistory` — Order.find({ user }) sorted by date, populate course names + thumbnails
- [x] Email templates: `courseEnrollmentEmail.js`, `paymentSuccessEmail.js`

---

### Phase 6 — Instructor Analytics
- [x] `instructorDashboard` — aggregation pipeline: group orders by course, compute revenue split, `$avg` rating, total students
- [x] Surface `topByRevenue`, `topByEnrollment`, `topByRating` spotlight cards

---

### Phase 7 — Frontend Foundation
- [x] Vite 6 project with `@vitejs/plugin-react` + `@tailwindcss/vite`
- [x] Redux store with `auth`, `profile`, `cart`, `course`, `viewCourse` slices
- [x] Axios instance (`services/`) with base URL from `VITE_BASE_URL`
- [x] React Router v7 with nested dashboard routes
- [x] `<PrivateRoute>` component redirecting unauthenticated users to `/login`
- [x] `vercel.json` SPA rewrite rule

---

### Phase 8 — Auth UI
- [x] Signup page (role selector: Student / Instructor)
- [x] OTP verification page (6-cell `react-otp-input`)
- [x] Login page
- [x] Forgot password page (email submission)
- [x] Update password page (token from URL param)
- [x] JWT persistence across page refreshes (localStorage + Redux rehydration)

---

### Phase 9 — Student UI
- [x] Home page (hero, stats, featured courses, testimonials — Swiper carousel)
- [x] Catalogue page (course grid, category filter)
- [x] CourseDetails page (instructor info, curriculum accordion, enrol / add-to-cart CTA)
- [x] Cart page (item list, total, Razorpay checkout trigger)
- [x] ViewCourse page (video-react player, section/lecture sidebar, progress tick)
- [x] EnrolledCourses dashboard
- [x] PurchaseHistory page (order table with date, amount, courses)

---

### Phase 10 — Instructor UI
- [x] AddCourse wizard (3 steps: Course Info → Curriculum → Publish)
  - Step 1: React Hook Form, tag/instruction multi-input, thumbnail dropzone
  - Step 2: Section + SubSection management, video upload via react-dropzone
  - Step 3: Status toggle (Draft ↔ Published) + submit
- [x] MyCourses table (thumbnail, name, price, status, students, edit/delete actions)
- [x] EditCourse (same wizard pre-filled)
- [x] InstructorDashboard (total revenue, total students, total courses, per-course stat cards, Recharts bar chart)

---

### Phase 11 — Shared UI
- [x] Navbar (role-aware nav links, cart badge, avatar dropdown)
- [x] Footer
- [x] Settings page (update profile, update avatar, change password, delete account)
- [x] MyProfile page (read-only profile display)
- [x] NotFound / 404 page

---

### Phase 12 — Integration & Debugging
- [x] CORS `origin` locked to `https://codevolvx.vercel.app` in Express
- [x] Axios `withCredentials: true` for cookie forwarding
- [x] Cloudinary upload path fix (Windows backslash → forward-slash normalisation)
- [x] `timeDuration` parsed consistently as integer in duration utility
- [x] `verifySignature` fixed — `Object.values(courses)` for array vs object mismatch
- [x] video-react import path fix for Vite ESM compatibility
- [ ] Password-reset URL still hardcoded to `localhost:5173` — not fixed before submission
- [ ] No rate limiting added to `/sendotp`

---

### Phase 13 — Deployment
- [x] Vercel: connect GitHub repo, set `VITE_BASE_URL` + `VITE_RAZORPAY_KEY` env vars, verify `vercel.json`
- [x] Render: Node.js web service, `npm install && node index.js`, set all server env vars
- [x] MongoDB Atlas: add Render static outbound IP to allowlist
- [x] Smoke-test: signup → OTP → login → enrol → pay → watch video → review

---

### Phase 14 — Documentation
- [x] `README.md` — overview, tech stack, setup instructions
- [x] `SUBMISSION.md` — links, credentials, stack, goal checklist
- [x] `docs/architecture.md`
- [x] `docs/schema.md`
- [x] `docs/plan.md` (this file)
- [x] `docs/decisions.md`
- [x] `docs/ai-prompts.md`

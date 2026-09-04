# System Architecture — codevolveX

## Overview

codevolveX is a three-tier MERN web application. A React 18 SPA (served by Vercel) communicates exclusively through a REST API built on Node.js + Express (hosted on Render). All application data lives in MongoDB Atlas. Three external services handle media (Cloudinary), payments (Razorpay), and transactional email (Brevo).

---

## High-Level Diagram

```
┌────────────────────────────────────────────────────────┐
│                   Browser / React SPA                  │
│  React 18 · Redux Toolkit · React Router v7 · Axios   │
│  Tailwind CSS · Framer Motion · Recharts · video-react │
└──────────────────────┬─────────────────────────────────┘
                       │  HTTPS REST  (Bearer JWT)
                       ▼
┌────────────────────────────────────────────────────────┐
│            Express API  (Node.js ≥18, ES Modules)      │
│  /api/v1/auth  /api/v1/course  /api/v1/payment         │
│  /api/v1/profile  /api/v1/contact                      │
│                                                        │
│  Middleware chain:  cors → cookieParser → bodyParser   │
│                     → auth (JWT) → isRole → controller │
└────┬──────────┬──────────┬──────────┬──────────────────┘
     │          │          │          │
     ▼          ▼          ▼          ▼
 MongoDB    Cloudinary  Razorpay    Brevo
  Atlas      SDK v2      SDK v2     REST API
```

---

## Component Breakdown

### Frontend

| Layer | Technology | Responsibility |
|---|---|---|
| Build tool | Vite 6 | HMR in dev; Rollup bundle for prod; `vercel.json` rewrites SPA paths |
| Routing | React Router v7 | Nested routes; `<PrivateRoute>` wrapper for auth-gated pages |
| Global state | Redux Toolkit 2 | 5 slices: `auth`, `profile`, `cart`, `course`, `viewCourse` |
| Styling | Tailwind CSS v4 | Utility classes; `@tailwindcss/vite` plugin |
| Animation | Framer Motion 12 | Page transitions, card reveals, modal open/close |
| Charts | Recharts 3 + MUI X Charts | Instructor revenue bar charts, enrolment line charts |
| Video | video-react | HTML5 player with `timeDuration` progress callbacks |
| Forms | React Hook Form 7 | Register, login, add-course wizard; built-in validation |
| Smooth scroll | Lenis 1 | Landing page momentum scroll |
| Notifications | react-hot-toast | Success / error toasts after API calls |
| Icons | lucide-react + react-icons | Consistent icon set across the UI |

**Pages (17 total)**

| Route | Page | Access |
|---|---|---|
| `/` | Home | Public |
| `/signup` | Signup | Public |
| `/login` | Login | Public |
| `/verify-email` | VerifyEmail (OTP entry) | Public |
| `/forgot-password` | ForgotPassword | Public |
| `/update-password/:token` | UpdatePassword | Public |
| `/about-us` | About | Public |
| `/contact-us` | Contact | Public |
| `/catalogue` | Catalogue (course list) | Public |
| `/catalogue/:courseId` | CourseDetails | Auth |
| `/dashboard/enrolled-courses` | EnrolledCourse | Student |
| `/dashboard/mycart` | Cart | Student |
| `/dashboard/purchase-history` | PurchaseHistory | Student |
| `/dashboard/view-courses/:courseId` | ViewCourse (player) | Student |
| `/dashboard/my-courses` | MyCourses | Instructor |
| `/dashboard/add-course` | AddCourse (wizard) | Instructor |
| `/dashboard/edit-course/:courseId` | EditCourse | Instructor |
| `/dashboard/instructor` | InstructorDashboard | Instructor |
| `/my-profile` | MyProfile | Auth |
| `/settings` | Settings | Auth |

---

### Backend

**Middleware chain (applied in order)**

```
cors({ origin: "https://codevolvx.vercel.app", credentials: true })
  → express.json({ limit: "4000mb" })         // large video base64 payloads
  → cookieParser()
  → auth(JWT)                                 // sets req.user
  → isStudent | isInstructor | isAdmin        // role guard
  → controller handler
```

**Route groups**

| Prefix | File | Key endpoints |
|---|---|---|
| `/api/v1/auth` | `user.routes.js` | `POST /login`, `POST /signup`, `POST /sendotp`, `POST /changepassword`, `POST /reset-password-token`, `POST /reset-password` |
| `/api/v1/course` | `course.routes.js` | `GET /getAllCourses`, `POST /createCourse`, `POST /editCourse`, `DELETE /deleteCourse`, `POST /addSection`, `POST /addSubSection`, `POST /updatecourseprogress`, `POST /createRating`, `GET /getInstructorDashboard` |
| `/api/v1/payment` | `payment.routes.js` | `POST /capturePayment`, `POST /verifyPayment`, `POST /sendPaymentSuccessEmail`, `GET /getPurchaseHistory` |
| `/api/v1/profile` | `profile.routes.js` | Profile CRUD, avatar upload |
| `/api/v1/contact` | `contact.routes.js` | Contact form submission |

**Controllers (11 files)**

`auth`, `resetPassword`, `course`, `section`, `subsection`, `category`, `courseProgress`, `ratingsAndReview`, `payment`, `profile`, `contactus`

---

### Database — MongoDB Atlas

See [`schema.md`](./schema.md) for full field-level detail.

**Collections and relationships**

```
User (1) ──────────────────────► Profile (1)      [additionalDetails ref]
User (1) ──────────────────────► Course (N)       [courses array]
User (1) ──────────────────────► CourseProgress(N)[courseProgress array]

Course (1) ────────────────────► Section (N)      [courseContent array]
Section (1) ───────────────────► SubSection (N)   [SubSection array]
Course (1) ────────────────────► RatingsAndReview (N)
Course (N) ────────────────────► Category (1)

Order (1) ─────────────────────► User (1)         [user ref]
Order (1) ─────────────────────► Course (N)       [courses array]

CourseProgress (1) ────────────► SubSection (N)   [completedVideo array]

Otp (TTL index: expires 15 000 s / ~4 hours after creation)
```

---

### External Services

| Service | SDK / Transport | Integration point |
|---|---|---|
| **Cloudinary** | `cloudinary` v2 SDK | `utils/cloudinary.js` — `uploadOnCloudinary()` writes temp file → uploads → deletes local copy; returns `secure_url` stored in `Course.thumbnail` or `SubSection.videoUrl` |
| **Razorpay** | `razorpay` v2 SDK | `config/razorpay.js` instance → `payment.js` controller; HMAC-SHA256 signature verified with `crypto.createHmac` before enrolment |
| **Brevo** | `@getbrevo/brevo` SDK + Nodemailer fallback | `utils/mailsender.js` → called by `OtpGenerator.pre("save")`, `payment.js`, `resetPassword.js`, `auth.js` (password changed) |

---

## Authentication & Authorization Flow

```
1. Signup
   Client  →  POST /api/v1/auth/sendotp       { email }
   Server  →  generate 6-digit OTP, persist to `otp` collection (TTL 4 h)
              OtpGenerator.pre("save") fires mailSender → Brevo sends OTP email
   Client  →  POST /api/v1/auth/signup        { ...fields, otp }
   Server  →  verify OTP, bcrypt.hash(password,10), create Profile + User
              return 201 + user object (password stripped)

2. Login
   Client  →  POST /api/v1/auth/login         { email, password }
   Server  →  bcrypt.compare → jwt.sign({ email,id,accountType }, 24h)
              Set httpOnly cookie "token" (30-day expiry) + return token in body
   Client  →  Redux auth slice stores token + user; Axios attaches as Bearer header

3. Protected request
   Server  →  auth middleware: reads cookie OR Authorization Bearer OR body.token
              jwt.verify → req.user = { email, id, accountType }
              isInstructor / isStudent / isAdmin checks req.user.accountType

4. Password reset
   Client  →  POST /reset-password-token      { email }
   Server  →  crypto.randomUUID() → stored as user.token; expires Date.now()+5h
              mailSender sends link: /update-password/:token
   Client  →  POST /reset-password            { token, password, confirmPassword }
   Server  →  validates expiry, bcrypt.hash, update user, clear token field
```

---

## Payment Flow

```
1. Client builds cart (courseIds[]) → Redux cart slice
2. POST /api/v1/payment/capturePayment   { courses: [...ids] }
   Server: sum prices, check not already enrolled
   → instance.orders.create({ amount: totalINRPaise, currency:"INR" })
   → returns Razorpay order_id to client

3. Client opens Razorpay checkout modal (test key)
   On success → receives { razorpay_order_id, razorpay_payment_id, razorpay_signature }

4. POST /api/v1/payment/verifyPayment    { ...razorpay fields, courses, userId }
   Server: HMAC-SHA256(order_id|payment_id) === signature ?
     → enrollStudents(): Course.$push(studentId), CourseProgress.create(), User.$push(courseId)
     → Order.create() records transaction
     → mailSender(courseEnrollmentEmail) per course
   → 200 success

5. POST /api/v1/payment/sendPaymentSuccessEmail { orderId, paymentId, amount }
   Server: mailSender(paymentSuccessEmail)
```

---

## Deployment

| Component | Platform | Notes |
|---|---|---|
| Frontend | Vercel | Auto-deploy from `main`; `vercel.json` rewrites `/*` → `/index.html` |
| Backend | Render (Node.js) | `npm start` → `node index.js`; cold-start ~30–60 s on free tier |
| Database | MongoDB Atlas M0 | `mongoose 8` connection string in `BASE_URL` env var |
| Media | Cloudinary | Free tier; `resource_type:"auto"` handles both images and videos |

---

## Known Weaknesses

| Issue | Location | Impact |
|---|---|---|
| No centralised error handler | All controllers | Inconsistent error response shapes |
| Password-reset URL hardcoded to `localhost:5173` | `resetPassword.js:28` | Production reset emails send broken links |
| `express.json({ limit: "4000mb" })` | `index.js:20` | Allows extremely large payloads; should use multipart for media |
| No rate limiting on `/sendotp` | `user.routes.js` | OTP endpoint is open to spam |
| Admin dashboard UI not implemented | `frontend/src` | `isAdmin` middleware exists but no admin pages |

# Database Schema — codevolveX

All collections live in MongoDB Atlas. The ODM is Mongoose 8 with ES Modules.

---

## Collections at a Glance

| Collection | Model file | Purpose |
|---|---|---|
| `users` | `User.model.js` | All accounts (Student / Instructor / Admin) |
| `profiles` | `Profile.model.js` | Extended user details (1-to-1 with User) |
| `courses` | `Course.model.js` | Course metadata, pricing, curriculum refs |
| `sections` | `Section.model.js` | Top-level curriculum grouping inside a course |
| `subsections` | `SubSection.model.js` | Individual video lectures inside a section |
| `courseprogresses` | `CourseProgress.model.js` | Per-student list of completed sub-sections |
| `ratingsandreviews` | `RatingAndReview.model.js` | One review per student per course |
| `categories` | `Category.model.js` | Course taxonomy (e.g. "Web Dev", "Data Science") |
| `orders` | `Payment.model.js` | Razorpay order record after successful payment |
| `otps` | `OtpGenerator.model.js` | Time-limited OTPs for email verification (TTL index) |

---

## Schema Definitions

### `users`

```js
{
  firstName:            String  (required, trim)
  lastName:             String  (required, trim)
  email:                String  (required, trim, unique implied by usage)
  password:             String  (required, bcrypt hash)
  accountType:          String  enum["Student","Instructor","Admin"]
  approved:             Boolean default true
                                // Instructors created with approved=false
  active:               Boolean default true
  additionalDetails:    ObjectId → profiles   (required)
  courses:              [ObjectId → courses]
  courseProgress:       [ObjectId → courseprogresses]
  image:                String  // Cloudinary URL or DiceBear initials SVG
  token:                String  // temporary reset-password token
  resetPasswordExpires: Date    // token expiry (Date.now() + 5 h)
  createdAt / updatedAt: (timestamps: true)
}
```

---

### `profiles`

```js
{
  gender:        String (trim, nullable)
  dateOfBirth:   String (trim, nullable)
  about:         String (trim, nullable)
  contactNumber: String (trim, nullable)
  createdAt / updatedAt: (timestamps: true)
}
```

> Created atomically alongside the User during signup. Populated via `User.additionalDetails`.

---

### `courses`

```js
{
  courseName:        String  (required, trim)
  courseDescription: String  (required, trim)
  instructor:        ObjectId → users  (required)
  whatWillYouLearn:  String  (trim)
  courseContent:     [ObjectId → sections]
  ratingsAndReview:  [ObjectId → ratingsandreviews]
  price:             Number  (required)
  thumbnail:         String  // Cloudinary secure_url
  category:          ObjectId → categories
  tag:               [String]
  studentsEnrolled:  [ObjectId → users]
  instructions:      [String]
  status:            String  enum["Draft","Published"]
  createdAt / updatedAt: (timestamps: true)
}
```

---

### `sections`

```js
{
  sectionName: String
  SubSection:  [ObjectId → subsections]  (required per element)
}
```

> No timestamps. Sections are deleted recursively when a course is deleted.

---

### `subsections`

```js
{
  title:        String
  timeDuration: String  // stored as seconds string, e.g. "324"
  description:  String
  videoUrl:     String  // Cloudinary secure_url
                        // field is excluded from getCourseDetails (preview)
                        // but included in getFullCourseDetails (enrolled view)
}
```

---

### `courseprogresses`

```js
{
  courseId:       ObjectId → courses
  userId:         ObjectId → users
  completedVideo: [ObjectId → subsections]
  createdAt / updatedAt: (timestamps: true)
}
```

> One document per (student, course) pair. `updateCourseProgress` controller pushes
> or pulls SubSection IDs from `completedVideo` to toggle completion state.

---

### `ratingsandreviews`

```js
{
  user:   ObjectId → users   (required)
  rating: Number             (required)
  review: String             (required)
  course: ObjectId → courses (required, indexed)
}
```

> `index: true` on `course` for fast per-course aggregation in `instructorDashboard`.
> No timestamps on this schema.
> Enforcement of one-review-per-student is handled in the controller logic, not at schema level.

---

### `categories`

```js
{
  name:        String  (required)
  description: String  (required)
  courses:     [ObjectId → courses]
}
```

> Categories are admin-seeded. The `courses` array is maintained via `$push` on
> `createCourse` and `$pull` on `deleteCourse`.

---

### `orders`

```js
{
  user:                ObjectId → users   (required)
  courses:             [ObjectId → courses]
  amount:              Number             (required, in INR)
  razorpay_order_id:   String             (required)
  razorpay_payment_id: String             (required)
  status:              String  enum["Success","failed"]  default "Success"
  createdAt / updatedAt: (timestamps: true)
}
```

> Created only after HMAC signature verification passes.
> `getPurchaseHistory` queries `Order.find({ user: userId }).populate('courses','courseName thumbnail')`.

---

### `otps` (TTL collection)

```js
{
  email: String  (required)
  otp:   String  (required, 6-digit numeric)
  createdAt: Date  default Date.now()
             // TTL index: expires: 15000 seconds (~4 hours)
}
```

> A Mongoose `pre("save")` hook fires `mailSender(otpTemplate(otp))` on every new document.
> The signup controller fetches `otpgenerator.findOne({ email }).sort({ createdAt: -1 })`
> to get the most-recent OTP.

---

## Entity-Relationship Overview

```
Category ──(1:N)──► Course ──(1:N)──► Section ──(1:N)──► SubSection
                      │
                      ├──(N:N)──► User  (studentsEnrolled ↔ User.courses)
                      │
                      └──(1:N)──► RatingsAndReview ──(N:1)──► User

User ──(1:1)──► Profile
User ──(1:N)──► CourseProgress ──(N:1)──► Course
                                └──(N:N)──► SubSection (completedVideo)

User ──(1:N)──► Order ──(N:N)──► Course
```

---

## Indexes & Performance Notes

| Collection | Field | Index type | Reason |
|---|---|---|---|
| `otps` | `createdAt` | TTL (15 000 s) | Auto-expire OTP documents |
| `ratingsandreviews` | `course` | Regular | Fast `$match` in aggregation pipeline for instructor dashboard |
| `orders` | `user`, `courses` | Implicit (query patterns) | `Order.find({user})` and `$in: courseIds` queries |

> MongoDB Atlas M0 does not support custom indexes beyond the defaults without upgrading.
> For production scale, add compound indexes on `Course.instructor + Course.status`
> and `User.email` (unique).

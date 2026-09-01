# codevolveX 🚀

**codevolveX** is a modern, full-stack EdTech Learning Management System (LMS) designed to deliver a seamless learning and course creation experience for students and instructors.

---

## 🌟 Key Features

### 👨‍🎓 For Students
- **Course Discovery & Catalog**: Browse courses by category, tags, ratings, and instructor profiles.
- **Interactive Video Player**: Modern video player with lecture navigation, autoplay completion, and progress tracking.
- **Cart & Secure Checkout**: Multi-item cart checkout powered by Razorpay payment gateway with cryptographic signature verification.
- **Personalized Dashboard**: Track enrolled courses, view completion percentages, and access order/purchase history.
- **Ratings & Reviews**: Share reviews and rate completed courses.

### 👨‍🏫 For Instructors
- **Visual Analytics Dashboard**: Track total revenue, enrolled student metrics, and view dynamic revenue charts built with Recharts.
- **Multi-Step Course Builder Wizard**:
  1. **Course Information**: Title, description, tags, pricing, thumbnail upload, requirements, and categories.
  2. **Course Builder**: Section and sub-section hierarchy management with Cloudinary video uploads.
  3. **Publishing Settings**: Draft/Publish toggles.
- **Course Management**: Edit, update, and manage published course curricula.

### 🔐 Platform & Security
- **JWT Authentication**: Role-based access control (`Student`, `Instructor`, `Admin`).
- **OTP Verification & Password Recovery**: Email verification and reset token flows powered by Brevo.
- **Cloud Media Storage**: Secure asset uploads and streaming powered by Cloudinary.

---

## 🛠️ Tech Stack

### Frontend
- **Framework**: React 18 with Vite
- **Routing**: React Router v7
- **State Management**: Redux Toolkit
- **Styling**: Tailwind CSS v4 & Framer Motion
- **Icons & Visuals**: Lucide React & React Icons
- **Charts**: Recharts

### Backend
- **Runtime & Framework**: Node.js & Express (ES Modules)
- **Database**: MongoDB with Mongoose ODM
- **Authentication**: JSON Web Tokens (JWT) & bcrypt
- **File Uploads**: Multer & Cloudinary SDK
- **Payments**: Razorpay Node SDK
- **Transactional Emails**: Brevo (Sendinblue) API

---

## 🚀 Getting Started

### Prerequisites
- Node.js (v18+)
- MongoDB Atlas or local MongoDB instance
- Cloudinary, Razorpay, and Brevo API credentials

### Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/saurabh45-p/ProjectAssessment-BUSY_Infotech.git
   cd ProjectAssessment-BUSY_Infotech
   ```

2. **Install Root, Frontend & Server Dependencies**:
   ```bash
   npm install
   cd frontend && npm install
   cd ../server && npm install
   cd ..
   ```

3. **Environment Setup**:
   Create `.env` in the `server` directory:
   ```env
   PORT=5501
   BASE_URL=your_mongodb_connection_uri
   JWT_SECRET=your_jwt_secret
   CLOUDINARY_CLOUD_NAME=your_cloud_name
   CLOUDINARY_API_KEY=your_api_key
   CLOUDINARY_API_SECRET=your_api_secret
   RAZORPAY_KEY=your_razorpay_key_id
   RAZORPAY_SECRET=your_razorpay_key_secret
   BREVO_API_KEY=your_brevo_api_key
   MAIL_USER=your_verified_sender_email
   ```

   Create `.env` in the `frontend` directory:
   ```env
   VITE_BASE_URL=http://localhost:5501/api/v1/
   VITE_RAZORPAY_KEY=your_razorpay_key_id
   ```

4. **Run the Project**:
   ```bash
   # Run both frontend & backend concurrently from root
   npm run dev
   ```

---

## 📄 License
This project is licensed under the [ISC License](LICENSE).

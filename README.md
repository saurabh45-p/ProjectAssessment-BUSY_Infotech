<div align="center">
  <img src="https://img.shields.io/badge/Status-Active-success.svg" alt="Status">
  <img src="https://img.shields.io/badge/License-ISC-blue.svg" alt="License">
  <img src="https://img.shields.io/badge/Version-1.0.0-informational.svg" alt="Version">
  <a href="https://codevolvx.vercel.app" target="_blank"><img src="https://img.shields.io/badge/Live%20Demo-codevolvx.vercel.app-brightgreen.svg?logo=vercel" alt="Live Demo"></a>
  <h1>codevolveX 🚀</h1>
  <p><strong>A Modern, Full-Stack EdTech Learning Management System</strong></p>
  <p>Delivering a seamless, scalable, and intuitive learning and course creation experience for modern education.</p>
  <p>
    <a href="https://codevolvx.vercel.app" target="_blank"><strong>🌐 Live Demo: https://codevolvx.vercel.app</strong></a>
  </p>
</div>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
  - [For Students](#-for-students)
  - [For Instructors](#-for-instructors)
  - [Platform Security & Infrastructure](#-platform-security--infrastructure)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation & Setup](#installation--setup)
  - [Environment Variables](#environment-variables)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🎯 Overview

**codevolveX** is an enterprise-grade Learning Management System (LMS) built with the MERN stack. It bridges the gap between eager learners and knowledgeable instructors by providing a robust platform for course creation, secure transactions, and interactive video-based learning. Designed with performance and user experience in mind, codevolveX scales seamlessly to accommodate growing educational communities.

🌐 **Live Application:** [https://codevolvx.vercel.app](https://codevolvx.vercel.app)

---

## ✨ Key Features

### 👨‍🎓 For Students
- **Intelligent Course Discovery**: Effortlessly navigate a comprehensive catalog using advanced filtering by category, tags, and instructor profiles.
- **Immersive Video Player**: Enjoy a distraction-free learning environment with our modern video player, featuring intelligent lecture navigation, seamless autoplay, and precise progress tracking.
- **Frictionless Checkout**: Experience secure, multi-item cart checkouts powered by the Razorpay payment gateway, complete with robust cryptographic signature verification.
- **Personalized Learning Hub**: A centralized dashboard to monitor enrolled courses, track completion metrics, and review detailed purchase histories.
- **Community-Driven Ratings**: Foster trust and engagement by sharing detailed reviews and ratings upon course completion.

### 👨‍🏫 For Instructors
- **Comprehensive Analytics Dashboard**: Make data-driven decisions using our visual analytics suite. Track revenue, monitor student enrollment metrics, and interact with dynamic Recharts-powered graphs.
- **Advanced Course Builder Wizard**:
  1. **Course Configuration**: Define metadata including title, comprehensive descriptions, pricing strategies, and categorize content effectively.
  2. **Curriculum Architecture**: Intuitively manage section and sub-section hierarchies with seamless Cloudinary video integrations.
  3. **Lifecycle Management**: Control course visibility with flexible Draft and Publish workflows.
- **Dynamic Course Management**: Seamlessly edit and update published curricula to keep content relevant and up-to-date.

### 🔐 Platform Security & Infrastructure
- **Robust Authentication (JWT)**: Secure, role-based access control ensuring tailored experiences for `Student`, `Instructor`, and `Admin` roles.
- **Resilient Account Recovery**: Secure OTP verification and password reset workflows orchestrated through the Brevo (Sendinblue) API.
- **Scalable Media Delivery**: High-performance, secure asset uploading and streaming infrastructure powered by Cloudinary.

---

## 🛠️ Tech Stack

### Client-Side (Frontend)
- **Core**: React 18 powered by Vite for lightning-fast HMR and optimized builds.
- **Navigation**: React Router v7 for seamless Client-Side Routing (CSR).
- **State Management**: Redux Toolkit for predictable and centralized application state.
- **Styling**: Tailwind CSS v4 coupled with Framer Motion for highly responsive and fluid user interfaces.
- **Data Visualization**: Recharts for dynamic, interactive analytical dashboards.

### Server-Side (Backend)
- **Environment**: Node.js & Express (ES Modules) providing a scalable, asynchronous runtime.
- **Database**: MongoDB with Mongoose ODM for flexible, document-based data modeling.
- **Authentication**: Industry-standard JSON Web Tokens (JWT) & bcrypt for secure credential hashing.
- **Asset Management**: Multer & Cloudinary SDK for robust media handling.
- **Payments & Communication**: Razorpay Node SDK for financial transactions and Brevo API for transactional email delivery.

---

## 🚀 Getting Started

Follow these instructions to set up the project locally for development and testing.

### Prerequisites
Ensure your development environment meets the following requirements:
- **Node.js** (v18.0.0 or higher)
- **MongoDB** (Atlas cluster or local instance)
- API Credentials for **Cloudinary**, **Razorpay**, and **Brevo**.

### Installation & Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/saurabh45-p/ProjectAssessment-BUSY_Infotech.git
   cd ProjectAssessment-BUSY_Infotech
   ```

2. **Install Dependencies**
   Install packages for the root, frontend, and server environments.
   ```bash
   # Install root dependencies (e.g., concurrently)
   npm install
   
   # Install frontend dependencies
   cd frontend && npm install
   
   # Install backend dependencies
   cd ../server && npm install
   cd ..
   ```

### Environment Variables

Configure your environment by creating `.env` files in both the `server` and `frontend` directories.

**Backend (`server/.env`)**
```env
# Server Configuration
PORT=5501
BASE_URL=your_mongodb_connection_uri
JWT_SECRET=your_jwt_secret_key

# Media Storage (Cloudinary)
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Payments (Razorpay)
RAZORPAY_KEY=your_razorpay_key_id
RAZORPAY_SECRET=your_razorpay_key_secret

# Email Service (Brevo)
BREVO_API_KEY=your_brevo_api_key
MAIL_USER=your_verified_sender_email
```

**Frontend (`frontend/.env`)**
```env
VITE_BASE_URL=http://localhost:5501/api/v1/
VITE_RAZORPAY_KEY=your_razorpay_key_id
```

### Running the Application

Launch both the frontend and backend servers concurrently from the project root:

```bash
npm run dev
```

The application will be accessible at `http://localhost:5173` (Frontend) and `http://localhost:5501` (Backend API).

---

## 🤝 Contributing

We welcome contributions to make **codevolveX** even better!
1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📄 License

Distributed under the [ISC License](LICENSE). See `LICENSE` for more information.

<div align="center">
  <p>Built with ❤️ for modern education.</p>
</div>

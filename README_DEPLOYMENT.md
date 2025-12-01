# Campus Management System - Complete Deployment Guide

## 📋 Overview

This is a full-stack Campus Management System with microservices architecture deployed on Render.

**Tech Stack:**
- **Frontend**: React + Vite + GraphQL Apollo Client
- **Backend Services**:
  - Auth Service: Spring Boot (Java 17) - Authentication & JWT
  - Course Service: Django REST Framework (Python) - Course Management
  - Student Service: Spring Boot (Java) - Student Management
  - Chatbot Service: Django - AI Chatbot
- **Database**: PostgreSQL (Shared instance on Render)
- **GraphQL Gateway**: Node.js Apollo Server
- **API Gateway**: Java Spring Boot
- **Deployment**: Render.com (Free tier)

## 🎯 Current Deployment Status

| Service | Status | URL |
|---------|--------|-----|
| Course Service | ✅ Deployed | https://course-service-fj4t.onrender.com |
| Auth Service | 🚀 Deploying | https://auth-service-xxxx.onrender.com |
| Database | ✅ Shared PostgreSQL | dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com |
| Frontend | 📝 Ready to deploy | - |
| GraphQL | 📝 Ready to deploy | - |

## 🔴 Current Issue: 401 Unauthorized

**Root Cause**: Admin user exists only locally, not in production database.

**Solution**: 
1. Deploy auth service to Render
2. Connect to shared PostgreSQL database
3. Create admin user in production
4. All endpoints will then work with JWT tokens

## 📁 Project Structure

```
FinalProject/
├── auth/                               # Auth Service (Java Spring Boot)
│   ├── Dockerfile                     # ✨ NEW - Docker build config
│   ├── pom.xml                        # ✨ UPDATED - Added PostgreSQL driver
│   ├── DEPLOYMENT_GUIDE.md            # ✨ NEW - Auth deployment guide
│   ├── src/
│   │   └── main/
│   │       └── resources/
│   │           ├── application.yml    # ✨ UPDATED - PostgreSQL config
│   │           └── application-prod.yml # ✨ NEW - Production config
│   └── mvnw / mvnw.cmd               # Maven wrapper
│
├── course_service/                     # Course Service (Django)
│   ├── Dockerfile                     # Docker build config
│   ├── DEPLOYMENT_GUIDE.md            # Deployment guide
│   ├── requirements.txt               # Python dependencies
│   ├── manage.py                      # Django CLI
│   └── course_service/
│       └── settings.py                # Django settings with PostgreSQL
│
├── student_service/                    # Student Service (Java)
│   └── pom.xml                        # Maven config
│
├── chatbot_service/                    # Chatbot Service (Django)
│   ├── manage.py                      # Django CLI
│   └── requirements.txt               # Dependencies
│
├── GraphQL/                            # GraphQL Gateway (Node.js)
│   ├── package.json                   # npm dependencies
│   └── src/
│       ├── index.js                   # GraphQL entry point
│       └── resolvers/                 # GraphQL resolvers
│
├── Api_Gateway/                        # REST API Gateway (Java)
│   └── pom.xml                        # Maven config
│
├── frontend/                           # Frontend (React + Vite)
│   ├── package.json                   # npm dependencies
│   ├── vite.config.js                 # Vite configuration
│   └── src/
│       ├── App.jsx                    # Main App component
│       ├── pages/                     # Page components
│       ├── components/                # Reusable components
│       └── services/                  # API service calls
│
├── .github/
│   └── workflows/
│       ├── deploy-course-service.yml  # ✅ Existing CI/CD
│       └── deploy-auth-service.yml    # ✨ NEW - Auth deployment
│
├── RENDER_DEPLOYMENT_COMPLETE_GUIDE.md # ✨ NEW - Full setup guide
├── QUICK_START_DEPLOYMENT.md           # ✨ NEW - Quick reference
└── README.md                           # This file
```

## 🚀 Quick Start: Deploy Auth Service

### Prerequisites
- GitHub account
- Render account
- PostgreSQL database credentials from course service

### Step 1: Get Database Credentials

From your Render course service, copy:
```
DATABASE_URL: jdbc:postgresql://dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432/course_db_1lil
DB_USER: course_db_user
DB_PASSWORD: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
```

### Step 2: Create Auth Service on Render

1. Go to https://render.com → **New** → **Web Service**
2. Connect GitHub repo
3. Configure:
   - Name: `auth-service`
   - Root Directory: `auth`
4. Add environment variables with database credentials
5. Copy **Service ID**

### Step 3: Setup GitHub Secrets

1. Go to GitHub repo → Settings → Secrets → Actions
2. Add:
   - `RENDER_API_KEY`
   - `RENDER_AUTH_SERVICE_ID`

### Step 4: Deploy

```bash
git add .
git commit -m "Deploy auth service"
git push
```

### Step 5: Create Admin User

Connect to PostgreSQL and create admin user, then test login.

**See `QUICK_START_DEPLOYMENT.md` for detailed steps!**

## 🔑 Database Schema (Shared PostgreSQL)

Both auth and course services share the same PostgreSQL database:

```sql
-- Users table (Auth Service)
CREATE TABLE auth_user (
  id SERIAL PRIMARY KEY,
  username VARCHAR(150) UNIQUE,
  password VARCHAR(255),
  email VARCHAR(255),
  is_staff BOOLEAN,
  is_superuser BOOLEAN,
  date_joined TIMESTAMP
);

-- Courses table (Course Service)
CREATE TABLE courses_course (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  description TEXT,
  credits INT,
  created_at TIMESTAMP
);

-- Enrollments table (Course Service)
CREATE TABLE courses_enrollment (
  id SERIAL PRIMARY KEY,
  student_id INT,
  course_id INT REFERENCES courses_course(id),
  enrolled_at TIMESTAMP
);
```

## 🔐 Authentication Flow

```
┌──────────────┐
│   Frontend   │
└──────┬───────┘
       │
       │ 1. POST /api/auth/login
       ▼
┌──────────────────────┐
│  Auth Service        │  (Returns JWT token)
│  - Validates creds   │
│  - Generates JWT     │
└──────┬───────────────┘
       │ JWT Token
       ▼
┌──────────────────────┐
│   Frontend Cache     │  (Stores token)
└──────┬───────────────┘
       │
       │ 2. GET /api/courses/ + "Authorization: Bearer JWT"
       ▼
┌──────────────────────┐
│  Course Service      │  (Validates JWT)
│  - Check JWT token   │
│  - Return courses    │
└──────────────────────┘
```

## 🧪 Testing with Postman

### 1. Login (Get JWT Token)

```http
POST https://auth-service-xxxx.onrender.com/api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password123"
}
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "username": "admin"
}
```

### 2. Create Course (Admin Only)

```http
POST https://course-service-fj4t.onrender.com/api/courses/
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "name": "Introduction to Python",
  "description": "Learn Python basics",
  "credits": 3
}
```

### 3. List Courses (Any User)

```http
GET https://course-service-fj4t.onrender.com/api/courses/
```

## 📊 Service Relationships

```
┌─────────────────────────────────────────────────────────┐
│                    Frontend (React)                      │
│                  Deployed: Not yet                       │
└─────────┬───────────────────────────────────┬───────────┘
          │                                   │
          │ GraphQL                   REST API│
          ▼                                   ▼
┌──────────────────────┐        ┌──────────────────────────┐
│  GraphQL Gateway     │        │   API Gateway (Java)     │
│  Node.js Apollo      │        │   (Routes requests)      │
│  Not yet deployed    │        │   Not yet deployed       │
└──────────┬───────────┘        └─────┬────────────────────┘
           │                          │
    ┌──────┴──────────┬──────────┬────┴─────────────┐
    │                 │          │                  │
    ▼                 ▼          ▼                  ▼
┌─────────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐
│Auth Service │ │ Courses  │ │ Students │ │  Chatbot   │
│Java/Spring  │ │Django    │ │Java/Spring│ │  Django    │
│Deployed ✅  │ │Deployed✅│ │Ready      │ │  Ready     │
└─────────────┘ └──────────┘ └──────────┘ └────────────┘
    │                 │          │                  │
    └─────────────────┴──────────┴──────────────────┘
                      │
                      ▼
              ┌──────────────────┐
              │  PostgreSQL DB   │
              │  Render.com      │
              │  Deployed ✅     │
              └──────────────────┘
```

## 🛠️ Local Development Setup

### Prerequisites
- Java 17+
- Python 3.11+
- Node.js 18+
- PostgreSQL 13+
- Maven 3.9+

### Setup

#### 1. Auth Service (Java)
```bash
cd auth
mvn clean install
mvn spring-boot:run
# Runs on http://localhost:8082
```

#### 2. Course Service (Django)
```bash
cd course_service
pip install -r requirements.txt
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
# Runs on http://localhost:8000
```

#### 3. Frontend (React)
```bash
cd frontend
npm install
npm run dev
# Runs on http://localhost:5173
```

#### 4. GraphQL Gateway (Node.js)
```bash
cd GraphQL
npm install
npm start
# Runs on http://localhost:4000
```

## 📋 Deployment Checklist

### Before Deploying Auth Service
- [ ] Reviewed `QUICK_START_DEPLOYMENT.md`
- [ ] Got PostgreSQL credentials from course service
- [ ] Generated JWT secret
- [ ] Have Render API key ready
- [ ] GitHub secrets configured

### During Deployment
- [ ] Auth service created on Render
- [ ] Environment variables set
- [ ] Deployment triggered via GitHub
- [ ] Logs show successful build

### After Deployment
- [ ] Created admin user in production DB
- [ ] Tested login endpoint
- [ ] Tested course creation with JWT
- [ ] Verified CORS settings
- [ ] Monitored logs for errors

### Future Deployments
- [ ] Deploy frontend
- [ ] Deploy GraphQL gateway
- [ ] Deploy Student service
- [ ] Deploy API Gateway
- [ ] Deploy Chatbot service
- [ ] Configure service-to-service communication
- [ ] Set up monitoring/alerts

## 📚 Documentation

All guides are in the root directory:

1. **QUICK_START_DEPLOYMENT.md** - TL;DR version (START HERE!)
2. **RENDER_DEPLOYMENT_COMPLETE_GUIDE.md** - Detailed setup with troubleshooting
3. **auth/DEPLOYMENT_GUIDE.md** - Auth service specific guide
4. **course_service/DEPLOYMENT_GUIDE.md** - Course service guide

## 🔗 Useful Links

- [Render Docs](https://render.com/docs)
- [Spring Boot Docs](https://spring.io/projects/spring-boot)
- [Django Docs](https://docs.djangoproject.com/)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [GraphQL Docs](https://graphql.org/learn/)

## ❓ FAQ

**Q: Why am I getting 401 Unauthorized?**
A: Admin user only exists locally. You need to create it in production after deploying auth service.

**Q: Can I use different databases?**
A: You can, but Render's free tier limits you. Sharing one PostgreSQL database is recommended.

**Q: How do I monitor the deployed services?**
A: Check Render dashboard → Service → Logs tab for real-time logs.

**Q: Can I deploy all services at once?**
A: No, deploy them one by one. Start with auth, then course, then others.

**Q: What if deployment fails?**
A: Check the logs in Render dashboard. Common issues: missing dependencies, wrong environment variables, database connection.

## 🤝 Contributing

1. Create a branch for your feature
2. Make changes
3. Test locally
4. Push and create a pull request
5. GitHub Actions will test and deploy

## 📞 Support

For issues, check:
1. Service logs in Render dashboard
2. GitHub Actions workflow logs
3. Local development logs
4. Database connectivity

## ✅ Summary

**You've successfully:**
1. ✅ Deployed course service to Render
2. 🚀 Created all files to deploy auth service
3. 📚 Created comprehensive deployment guides
4. 🔧 Configured services to share PostgreSQL
5. 🔐 Set up JWT authentication flow

**Next steps:**
1. Deploy auth service using QUICK_START_DEPLOYMENT.md
2. Create admin user in production
3. Test login workflow
4. Deploy remaining services

Happy deploying! 🚀

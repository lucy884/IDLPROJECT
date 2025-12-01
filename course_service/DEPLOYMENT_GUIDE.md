# Course Service - Deployment Guide

## Overview
This is a Django REST API service for managing courses and student enrollments. It can be deployed on Render with automatic CI/CD using GitHub Actions.

## Prerequisites
1. Python 3.11+
2. PostgreSQL database
3. GitHub account with repository
4. Render account (https://render.com)

## Local Development

### 1. Install Dependencies
```bash
pip install -r requirements.txt
```

### 2. Environment Variables
Create a `.env` file based on `.env.example`:
```bash
SECRET_KEY=your-secret-key-here
DEBUG=True
DB_NAME=course_db
DB_USER=postgres
DB_PASSWORD=your-password
DB_HOST=localhost
DB_PORT=5432
ALLOWED_HOSTS=localhost,127.0.0.1
```

### 3. Run Migrations
```bash
python manage.py migrate
```

### 4. Start the Development Server
```bash
python manage.py runserver
```

The API will be available at `http://127.0.0.1:8000/api/`

## Deployment on Render

### Step 1: Create a Render Account
- Go to https://render.com and sign up

### Step 2: Generate API Key
1. Log in to your Render account
2. Click your avatar (top right) → Account Settings
3. In the sidebar, select **API Keys**
4. Click **Create API Key**
5. Give it a name and create it
6. **Copy the key immediately** (you'll only see it once)

### Step 3: Create PostgreSQL Database on Render
1. In Render dashboard, click **New** → **PostgreSQL**
2. Enter database name (e.g., `course-db`)
3. Select Instance Type: **Free**
4. Click **Deploy PostgreSQL Instance**
5. Copy the database credentials:
   - Hostname
   - Port
   - Database name
   - Username
   - Password

### Step 4: Create Web Service on Render
1. Click **New** → **Web Service**
2. Connect your GitHub repository
3. Configure the service:
   - **Name**: course-service
   - **Root Directory**: IDL/course_service
   - **Build Command**: `pip install -r requirements.txt && python manage.py migrate`
   - **Start Command**: `gunicorn course_service.wsgi:application --bind 0.0.0.0:8000`
4. Add Environment Variables:
   - `PYTHON_VERSION`: 3.11
   - `SECRET_KEY`: (generate a new one)
   - `DEBUG`: False
   - `DB_NAME`: (from PostgreSQL service)
   - `DB_USER`: (from PostgreSQL service)
   - `DB_PASSWORD`: (from PostgreSQL service)
   - `DB_HOST`: (from PostgreSQL service)
   - `DB_PORT`: 5432
   - `ALLOWED_HOSTS`: your-render-domain.onrender.com

5. Click **Create Web Service**
6. Copy the **Service ID**

### Step 5: Add Secrets to GitHub
1. Go to your GitHub repository
2. Settings → **Secrets and variables** (left sidebar) → **Actions**
3. Create two secrets:
   - `RENDER_API_KEY`: Your Render API key
   - `RENDER_SERVICE_ID`: Your Render service ID

### Step 6: CI/CD Workflow
The `.github/workflows/deploy-course-service.yml` file will:
- Run tests on every push to main/master
- Build Docker image
- Deploy automatically to Render

## API Endpoints

### Courses
- `GET /api/courses/` - List all courses
- `POST /api/courses/` - Create course (Admin only)
- `GET /api/courses/<id>/` - Get course details
- `PUT /api/courses/<id>/` - Update course (Admin only)
- `DELETE /api/courses/<id>/` - Delete course (Admin only)
- `GET /api/courses/search/` - Search courses

### Enrollments
- `POST /api/courses/<id>/enroll/` - Enroll student
- `DELETE /api/courses/<id>/unenroll/` - Unenroll student
- `GET /api/courses/<id>/students/` - Get course students
- `GET /api/enrollments/` - List all enrollments
- `GET /api/enrollments/<id>/` - Get enrollment details
- `GET /api/enrollments/by_student/?student_id=123` - Get student's courses

## Testing with Postman
Import the `Course_Service_API.postman_collection.json` file into Postman to test all endpoints.

## Troubleshooting

### Database Connection Issues
- Verify PostgreSQL credentials in environment variables
- Check if Render database service is running
- Ensure `psycopg2-binary` is installed

### Static Files Not Loading
The application uses `collectstatic --noinput --clear` during build. For production, consider using a storage service like AWS S3.

### Logs
Check Render logs in the dashboard to debug any issues during deployment.

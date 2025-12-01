# 🔧 FIX: Auth Service Deployment Error

## Problem
Render is trying to build auth service as Python instead of Docker/Java, causing:
```
ModuleNotFoundError: No module named 'app'
```

## Root Cause
Render detected Python files in the project and tried to build as Python instead of recognizing the Dockerfile.

## Solution

### Step 1: Delete Failed Service on Render

1. Go to https://render.com/dashboard
2. Click on **auth_ser** (the failed service)
3. Click **Settings** (top right)
4. Scroll to bottom
5. Click **Delete Service**
6. Confirm deletion

### Step 2: Create New Service with Docker Explicitly

1. Go back to dashboard
2. Click **New** → **Web Service**
3. Select your GitHub repo (IDLPROJECT)
4. Configure:
   ```
   Name: auth-service
   Root Directory: auth
   Runtime: Docker
   ```

**IMPORTANT:** Make sure Runtime is set to **Docker**, not "Auto-detect"

5. Click **Create Web Service**

### Step 3: Add Environment Variables

Once created:

1. Click **Settings**
2. Scroll to **Environment**
3. Add these variables:

```
DATABASE_URL=jdbc:postgresql://dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432/course_db_1lil
DB_USER=course_db_user
DB_PASSWORD=znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
JWT_SECRET=<your-jwt-secret>
ALLOWED_ORIGINS=http://localhost:3000,https://course-service-fj4t.onrender.com
JAVA_VERSION=17
```

4. Copy the **Service ID** from the URL

### Step 4: Update GitHub Secret

1. Go to GitHub → Settings → Secrets
2. Update `RENDER_AUTH_SERVICE_ID` with your NEW service ID

### Step 5: Redeploy

```powershell
cd C:\Users\licya\OneDrive\Desktop\FinalProject

# Update workflow if needed
git add .github/workflows/deploy-auth-service.yml

# Commit
git commit -m "Fix auth service deployment - use Docker runtime"

# Push to trigger redeploy
git push origin main
```

### Step 6: Monitor Deployment

1. Go to GitHub → Actions
2. Watch the workflow complete
3. Go to Render dashboard
4. Watch status change from "Building" → "Live"

---

## ✅ What to Look For

**Successful deployment logs should show:**
```
✓ Building Dockerfile
✓ Maven build: mvn clean package
✓ Creating Java image
✓ Starting application on port 8082
✓ Health check passing
✓ Service: Live
```

**NOT these Python errors:**
```
✗ ModuleNotFoundError
✗ ImportError
✗ gunicorn
✗ Python version
```

---

## 🚀 After Successful Deployment

1. Create admin user in database (see STEP_BY_STEP_WALKTHROUGH.md Phase 6)
2. Test login endpoint
3. Test course creation with token

---

## Alternative: If Docker Still Fails

If Render still doesn't recognize Docker after following above:

**Option A: Add render.yaml**
Create file: `render.yaml` in project root

```yaml
services:
  - type: web
    name: auth-service
    runtime: docker
    rootDir: auth
    dockerfilePath: Dockerfile
    envVars:
      - key: DATABASE_URL
        value: jdbc:postgresql://...
```

Then push and Render will read this config.

**Option B: Use Build Command Explicitly**
In Render Settings:
- Build Command: `docker build -t auth-service auth/`
- Start Command: `docker run -p 8082:8082 auth-service`

---

## 📞 Quick Summary

1. ✅ Delete old auth service
2. ✅ Create new with Docker runtime
3. ✅ Add environment variables
4. ✅ Update GitHub secret
5. ✅ Push code to trigger redeploy
6. ✅ Monitor until "Live" status

**That's it!** The service should build and deploy correctly this time.

---

**Need help? Check the logs in Render → auth-service → Logs tab**

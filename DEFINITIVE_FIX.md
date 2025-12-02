# 🔧 DEFINITIVE FIX: Auth Service Deployment

## ❌ Why It Failed (Twice)

Render is detecting Python files in your project root and **defaulting to Python runtime** instead of using the Dockerfile.

The error log shows:
```
ModuleNotFoundError: No module named 'app'
```

This confirms Render is treating it as a Python project.

---

## ✅ THE SOLUTION: Delete & Recreate Service

**Render won't read render.yaml if the service already exists.**

You MUST delete the old service and create a new one with **explicit Docker runtime selection**.

---

## 🚀 STEP-BY-STEP FIX (15 minutes)

### STEP 1: Delete Old Service from Render

1. Go to: https://render.com/dashboard
2. Click on **auth_ser** (the failed service)
3. Click **Settings** (top right)
4. Scroll to the bottom
5. Click red **Delete Service** button
6. Type the service name to confirm deletion
7. Wait 1-2 minutes for deletion

### STEP 2: Create New Service with Docker

1. Go back to Render dashboard
2. Click **New** → **Web Service**
3. Select your **IDLPROJECT** repository
4. **IMPORTANT CONFIG:**
   ```
   Name: auth-service
   Root Directory: auth
   Runtime: Docker ← Must select Docker!
   Plan: Free
   Auto-Deploy: On
   ```
5. Click **Create Web Service**
6. Wait for service to be created (1-2 min)

### STEP 3: Get Service ID

1. After creation, look at URL bar
2. URL format: `https://render.com/services/auth-service-XXXXXXXXX`
3. Copy: `auth-service-XXXXXXXXX`

### STEP 4: Update GitHub Secret

1. Go to GitHub: https://github.com/lucy884/IDLPROJECT
2. Settings → Secrets and variables → Actions
3. Click **RENDER_AUTH_SERVICE_ID**
4. Update value with NEW service ID from Step 3
5. Click Save

### STEP 5: Add Environment Variables

In Render, click on auth-service, then Settings → Environment

Add these 6 variables:

```
DATABASE_URL = jdbc:postgresql://dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432/course_db_1lil

DB_USER = course_db_user

DB_PASSWORD = znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN

JWT_SECRET = (your-jwt-secret)

ALLOWED_ORIGINS = http://localhost:3000,https://course-service-fj4t.onrender.com

JAVA_VERSION = 17
```

**WAIT for all 6 to show as set before proceeding.**

### STEP 6: Push Code to GitHub

```powershell
cd C:\Users\licya\OneDrive\Desktop\FinalProject

git add .

git commit -m "Fix auth service - explicit Docker config and .renderignore"

git push origin main
```

### STEP 7: Monitor Deployment

**GitHub Actions:**
1. Go to: https://github.com/lucy884/IDLPROJECT/actions
2. Watch workflow "Java CI and Deploy Auth Service to Render"
3. Should see ✅ All steps complete (3-5 min)

**Render:**
1. Go to: https://render.com/dashboard
2. Click auth-service
3. Watch status: Building → Deploying → **Live** (green) ✅

**Check Logs:**
1. Click auth-service
2. Click **Logs** tab
3. Look for: "application started" or "Tomcat started"
4. Should NOT see Python or ImportError

---

## ✅ You Know It Worked When:

- ✅ Render shows **Live** (green status)
- ✅ Logs show `application started`
- ✅ Logs show `Spring Boot` or `Tomcat` (not Python)
- ✅ No Python errors or ModuleNotFoundError
- ✅ Can access health endpoint

---

## 🎯 Next Steps After Success

Once you see **Live** status:

1. **Create Admin User** in PostgreSQL database
2. **Test Login** endpoint
3. **Test Course Creation** with JWT token
4. **Verify** no 401 Unauthorized errors

---

## 📋 Critical Checklist

Before deployment:
- [ ] Old service deleted (wait 2 min)
- [ ] New service created with Docker runtime
- [ ] All 6 environment variables added
- [ ] New Service ID copied
- [ ] GitHub secret updated
- [ ] Code pushed to GitHub

During deployment:
- [ ] GitHub Actions workflow running
- [ ] Maven building successfully
- [ ] Render shows "Building" status
- [ ] Status changes to "Live"

After deployment:
- [ ] Logs show "application started"
- [ ] No errors in logs
- [ ] Can access service (blue URL)

---

## 🆘 If Still Failing

**Check these in order:**

1. **Runtime Setting Wrong?**
   - Render → Settings → Runtime
   - Must say "Docker"

2. **Environment Variables Missing?**
   - Render → Settings → Environment
   - All 6 variables must be present

3. **Logs Show Python Error?**
   - Delete service again
   - Recreate with explicit Docker selection

4. **Logs Show Maven Error?**
   - Check Java version setting
   - Verify pom.xml has correct dependencies
   - Check PostgreSQL driver is added

5. **Can't Access Health Endpoint?**
   - Check auth service URL
   - Make sure status is Live (green)
   - Check ALLOWED_ORIGINS setting

---

## 📞 Final Commands

```powershell
# Stage changes
git add .

# Commit
git commit -m "Fix auth deployment - Docker runtime, .renderignore, explicit config"

# Push
git push origin main

# Done! Watch GitHub Actions and Render dashboard
```

---

## ✨ Summary

**Why It Failed:**
- Render defaulted to Python instead of Docker

**How To Fix:**
- Delete old service
- Create new service with Docker explicitly
- Add environment variables
- Push code (triggers redeploy)

**Time:** ~15-20 minutes

**Result:** ✅ Live service!

---

**FOLLOW THESE STEPS EXACTLY - No shortcuts!**

Start with **STEP 1: Delete Old Service**

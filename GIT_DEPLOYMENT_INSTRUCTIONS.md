# 🔧 Ready to Deploy - Git Instructions

## 📋 Before You Deploy

Make sure everything is set up on Render first:
1. ✅ Created GitHub secrets (RENDER_API_KEY, RENDER_AUTH_SERVICE_ID)
2. ✅ Created auth service on Render
3. ✅ Added all environment variables
4. ✅ Read one of the guides from INDEX.md

---

## 🚀 Deploy to Render

### Step 1: Verify Changes

Check what files were created/modified:

```powershell
cd C:\Users\licya\OneDrive\Desktop\FinalProject
git status
```

You should see changes in:
- `auth/Dockerfile` (new)
- `auth/DEPLOYMENT_GUIDE.md` (new)
- `auth/src/main/resources/application-prod.yml` (new)
- `auth/src/main/resources/application.yml` (modified)
- `auth/pom.xml` (modified)
- `.github/workflows/deploy-auth-service.yml` (new)
- Various documentation files (new)

### Step 2: Stage All Changes

```powershell
git add .
```

### Step 3: Commit with Message

```powershell
git commit -m "Deploy auth service to Render with PostgreSQL and JWT authentication"
```

**Alternative detailed commit message:**

```powershell
git commit -m "feat: Deploy auth service to Render

- Add Dockerfile for Java Spring Boot Docker build
- Add GitHub Actions CI/CD pipeline for auth service
- Add production configuration with environment variables
- Update application.yml to use PostgreSQL
- Add PostgreSQL and Actuator dependencies to pom.xml
- Add comprehensive deployment guides and documentation

Features:
- Multi-stage Docker build for optimized image size
- Automated deployment via GitHub Actions
- Environment-based configuration for dev/prod
- JWT authentication with PostgreSQL backend
- Shared database with course service

Fixes: #XXX (401 Unauthorized error)
"
```

### Step 4: Push to GitHub

```powershell
git push origin main
```

**Output should show:**
```
Enumerating objects: XX, done.
Counting objects: 100% (XX/XX), done.
Delta compression using up to X threads
Compressing objects: 100% (XX/XX), done.
Writing objects: 100% (XX/XX), X.XX MiB | X.XX MiB/s, done.
Total XX (delta XX), reused 0 (delta 0), pack-reused 0
To https://github.com/lucy884/IDLPROJECT.git
   xxxxxxx..xxxxxxx  main -> main
```

---

## 👀 Monitor Deployment

### Watch GitHub Actions

1. Go to: https://github.com/lucy884/IDLPROJECT/actions
2. Click the latest workflow run
3. Watch status: 🟡 In Progress → ✅ Success

**Workflow steps:**
```
✅ Checkout code
✅ Setup JDK 17
✅ Build with Maven
✅ Run tests
✅ Deploy to Render
```

### Watch Render Deployment

1. Go to: https://render.com/dashboard
2. Click: auth-service
3. Watch status:
   ```
   Building (blue) → Live (green)
   ```
4. Scroll to **Logs** to see real-time output

---

## ✅ After Deployment

### When You See ✅ Success

The deployment is complete when:
- GitHub Actions workflow shows ✅
- Render shows "Live" status (green)
- Logs show "application started"

### Next: Create Admin User

Once deployed, follow these steps:

1. **Connect to PostgreSQL**
   ```
   Host: dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com
   Port: 5432
   Database: course_db_1lil
   Username: course_db_user
   Password: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
   ```

2. **Create Admin User**
   ```sql
   INSERT INTO auth_user (username, email, is_staff, is_superuser, date_joined)
   VALUES ('admin', 'admin@university.edu', true, true, NOW());
   ```

3. **Test Login**
   ```bash
   curl -X POST https://auth-service-xxxx.onrender.com/api/auth/login \
     -H "Content-Type: application/json" \
     -d '{"username": "admin", "password": "password123"}'
   ```

4. **Should Get JWT Token**
   ```json
   {
     "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
     "username": "admin"
   }
   ```

---

## 🔍 Verify Each Component

### 1. Check GitHub Actions

```
✅ Checkout repository
✅ Set up JDK 17
✅ Build with Maven
✅ Run tests (optional)
✅ Deploy to Render
```

**If any step fails:**
- Click the step to see logs
- Common errors:
  - Maven build fails: Check dependencies
  - Java version: Check pom.xml
  - Deploy fails: Check Render credentials

### 2. Check Render Service

```
✅ Service name: auth-service
✅ Status: Live (green)
✅ Port: 8082
✅ Environment variables: All set
```

**If service shows error:**
- Click **Logs** tab
- Look for error messages
- Common issues:
  - Database connection: Check DATABASE_URL
  - Maven cache: Rebuild
  - Port conflict: Restart service

### 3. Check Database

```powershell
# Connect to PostgreSQL
psql postgresql://course_db_user:password@dpg-xxx.render.com/course_db_1lil

# Verify admin user
SELECT * FROM auth_user WHERE username = 'admin';
```

---

## 🧪 Test After Deployment

### Test 1: Health Check

```bash
curl -i https://auth-service-xxxx.onrender.com/actuator/health
```

Expected: `200 OK`

### Test 2: Login

```bash
curl -X POST https://auth-service-xxxx.onrender.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "password123"}'
```

Expected: `200 OK` with JWT token

### Test 3: Create Course

```bash
JWT_TOKEN="your-jwt-token-here"

curl -X POST https://course-service-fj4t.onrender.com/api/courses/ \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Introduction to Python",
    "description": "Learn Python basics",
    "credits": 3
  }'
```

Expected: `201 Created` (not `401 Unauthorized`)

---

## 🐛 Troubleshooting

### Issue: Deployment Fails

**Check GitHub Actions logs:**
```
1. Go to Actions tab
2. Click workflow run
3. Click failed step
4. Read error message
```

**Common fixes:**
- Maven cache issue: `mvn clean`
- Dependency issue: Check pom.xml
- Java version: Ensure 17

### Issue: Can't Connect to Database

**Verify credentials:**
```
DATABASE_URL format: 
jdbc:postgresql://[HOST]:5432/[DATABASE]
```

**Test connection:**
```bash
psql "postgresql://user:password@host:5432/database"
```

### Issue: 401 Still Appears

**Check:**
1. Admin user exists: `SELECT * FROM auth_user WHERE username='admin';`
2. Auth service is running: Check Render status
3. JWT secret matches: Verify RENDER_JWT_SECRET

---

## 📊 Deployment Checklist

Before you deploy:
- [ ] Read INDEX.md or QUICK_START_DEPLOYMENT.md
- [ ] Have Render API key
- [ ] Created auth service on Render
- [ ] Added environment variables
- [ ] Verified GitHub secrets exist

During deployment:
- [ ] Committed code with meaningful message
- [ ] Pushed to main branch
- [ ] GitHub Actions triggered
- [ ] Monitoring logs

After deployment:
- [ ] Service shows "Live" status
- [ ] Created admin user in database
- [ ] Tested login endpoint
- [ ] Tested course creation
- [ ] No 401 errors

---

## 🎯 Success Indicators

✅ **Deployment succeeded when you see:**

1. **GitHub Actions**
   ```
   ✅ All steps completed
   ✅ No errors in logs
   ```

2. **Render Dashboard**
   ```
   ✅ Status: Live (green)
   ✅ No error messages
   ✅ Logs show "application started"
   ```

3. **Database**
   ```
   ✅ Admin user exists
   ✅ Connected to shared PostgreSQL
   ```

4. **API Testing**
   ```
   ✅ Login returns JWT (200 OK)
   ✅ Create course works (201 Created)
   ✅ No 401 Unauthorized
   ```

---

## 🚀 You're Ready!

```
1. git add .
2. git commit -m "Deploy auth service"
3. git push origin main
4. Watch GitHub Actions
5. Create admin user
6. Test endpoints
7. Success! ✅
```

**Estimated time: 30-40 minutes**

Good luck! 🎉

---

## 📝 Notes

- Keep database credentials safe
- Don't commit sensitive data (use environment variables)
- Verify all environment variables on Render
- Monitor logs for any errors
- Check Render dashboard regularly

## 🔗 Quick Links

- Render: https://render.com/dashboard
- GitHub: https://github.com/lucy884/IDLPROJECT
- Actions: https://github.com/lucy884/IDLPROJECT/actions
- Docs: See INDEX.md

---

**Ready to deploy? Go for it!** 🚀

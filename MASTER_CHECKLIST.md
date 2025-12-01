# ✅ MASTER DEPLOYMENT CHECKLIST

## 🎯 Your Complete Deployment Plan

### BEFORE YOU START

#### Step 1: Read One Guide (Choose Based on Time)
- [ ] **5 min?** Read: `QUICK_START_DEPLOYMENT.md`
- [ ] **10 min?** Read: `UNDERSTANDING_401_ERROR.md`
- [ ] **15 min?** Read: `STEP_BY_STEP_WALKTHROUGH.md`
- [ ] **30 min?** Read: `RENDER_DEPLOYMENT_COMPLETE_GUIDE.md`

#### Step 2: Gather Prerequisites
- [ ] Have Render account ready (https://render.com)
- [ ] Have GitHub account with repo access
- [ ] Have PostgreSQL credentials from course service:
  ```
  Host: dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com
  Port: 5432
  Database: course_db_1lil
  Username: course_db_user
  Password: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
  ```
- [ ] Have Postman or curl ready for testing

---

### PHASE 1: PREPARE (5 minutes)

#### Generate JWT Secret
- [ ] Open PowerShell and run:
  ```powershell
  [Convert]::ToBase64String([byte[]]@((1..32 | ForEach-Object { Get-Random -Maximum 256 })))
  ```
- [ ] Copy output (your JWT secret)

#### Get Render API Key
- [ ] Go to https://render.com
- [ ] Avatar → Account Settings → API Keys
- [ ] Create API Key
- [ ] Copy the key (you'll only see it once!)

---

### PHASE 2: GITHUB SETUP (3 minutes)

#### Add GitHub Secrets
- [ ] Go to GitHub repository
- [ ] Settings → Secrets and variables → Actions
- [ ] Click "New repository secret"
- [ ] Add Secret 1:
  ```
  Name: RENDER_API_KEY
  Value: (paste your API key)
  ```
- [ ] Add Secret 2:
  ```
  Name: RENDER_AUTH_SERVICE_ID
  Value: (you'll get this after creating service)
  ```
- [ ] Save both secrets

---

### PHASE 3: CREATE ON RENDER (5 minutes)

#### Create Web Service
- [ ] Go to https://render.com/dashboard
- [ ] Click "New" → "Web Service"
- [ ] Select your GitHub repository
- [ ] Configure:
  ```
  Name: auth-service
  Root Directory: auth
  Runtime: Docker (auto-detected)
  Instance Type: Free
  ```
- [ ] Click "Create Web Service"
- [ ] Wait for service to be created

#### Copy Service ID
- [ ] Look at the URL: https://render.com/services/auth-service-XXXXXXXXX
- [ ] Copy the ID part: `auth-service-XXXXXXXXX`
- [ ] Go to GitHub → Settings → Secrets
- [ ] Update `RENDER_AUTH_SERVICE_ID` with this value

---

### PHASE 4: ENVIRONMENT VARIABLES (3 minutes)

#### Add Variables in Render
- [ ] In Render, click on your auth-service
- [ ] Click "Settings"
- [ ] Scroll to "Environment"
- [ ] Click "Add Environment Variable" for each:

```
DATABASE_URL = jdbc:postgresql://dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432/course_db_1lil
```
- [ ] 

```
DB_USER = course_db_user
```
- [ ] 

```
DB_PASSWORD = znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
```
- [ ] 

```
JWT_SECRET = (your generated secret)
```
- [ ] 

```
ALLOWED_ORIGINS = http://localhost:3000,https://course-service-fj4t.onrender.com
```
- [ ] 

```
JAVA_VERSION = 17
```
- [ ] Verify all 6 variables are set

---

### PHASE 5: DEPLOY VIA GITHUB (2 minutes)

#### Push Code to GitHub
- [ ] Open PowerShell/Terminal
- [ ] Navigate to project:
  ```powershell
  cd C:\Users\licya\OneDrive\Desktop\FinalProject
  ```
- [ ] Stage changes:
  ```powershell
  git add .
  ```
- [ ] Commit with message:
  ```powershell
  git commit -m "Deploy auth service to Render with PostgreSQL"
  ```
- [ ] Push to GitHub:
  ```powershell
  git push origin main
  ```
- [ ] Verify: "main -> main" in output

#### Monitor GitHub Actions
- [ ] Go to https://github.com/lucy884/IDLPROJECT/actions
- [ ] Watch workflow "Python CI and Deploy to Render"
- [ ] Check status:
  ```
  🟡 Pending → Building
  🟡 Running → Testing  
  🟡 Deploying → Render
  ✅ Complete!
  ```

#### Monitor Render Deployment
- [ ] Go to https://render.com/dashboard
- [ ] Click auth-service
- [ ] Watch status:
  ```
  🔵 Building → Deploying
  🟢 Live ✅
  ```
- [ ] Check logs for "application started"

---

### PHASE 6: CREATE ADMIN USER (5 minutes)

#### Connect to PostgreSQL Database
- [ ] Download PostgreSQL client (DBeaver or pgAdmin)
- [ ] Or use psql command line if available
- [ ] Connection details:
  ```
  Host: dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com
  Port: 5432
  Database: course_db_1lil
  Username: course_db_user
  Password: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
  ```

#### Insert Admin User
- [ ] Connect to database
- [ ] Run SQL query:
  ```sql
  INSERT INTO auth_user (username, email, is_staff, is_superuser, date_joined)
  VALUES ('admin', 'admin@university.edu', true, true, NOW());
  ```
- [ ] Verify insertion:
  ```sql
  SELECT * FROM auth_user WHERE username = 'admin';
  ```

---

### PHASE 7: TEST LOGIN (3 minutes)

#### Test Auth Endpoint
- [ ] Open Postman
- [ ] Create new request:
  ```
  Method: POST
  URL: https://auth-service-XXXXXXXXX.onrender.com/api/auth/login
  Headers: Content-Type: application/json
  Body (raw):
  {
    "username": "admin",
    "password": "password123"
  }
  ```
- [ ] Send request
- [ ] Verify response:
  ```
  Status: 200 OK
  Response has "token" field
  ```
- [ ] Copy the token value

---

### PHASE 8: TEST COURSE CREATION (2 minutes)

#### Test Course Endpoint
- [ ] Create new Postman request:
  ```
  Method: POST
  URL: https://course-service-fj4t.onrender.com/api/courses/
  Headers:
    Authorization: Bearer <YOUR_JWT_TOKEN>
    Content-Type: application/json
  Body (raw):
  {
    "name": "Introduction to Python",
    "description": "Learn Python basics",
    "credits": 3
  }
  ```
- [ ] Send request
- [ ] Verify response:
  ```
  Status: 201 Created (NOT 401!)
  Response includes course data
  ```

---

## ✅ VERIFICATION CHECKLIST

### After Deployment, Verify:
- [ ] Auth service shows "Live" status (green) in Render
- [ ] Can access https://auth-service-xxx.onrender.com/actuator/health
- [ ] Admin user exists in PostgreSQL database
- [ ] Login endpoint returns 200 OK with JWT token
- [ ] Course creation returns 201 Created (not 401)
- [ ] GitHub Actions workflow shows ✅ Success
- [ ] No errors in Render service logs

---

## 🎯 SUCCESS INDICATORS

### You've Succeeded When:
- ✅ Auth service deployed (green in Render)
- ✅ Logged in successfully (got JWT token)
- ✅ Created course successfully (201 Created)
- ✅ NO 401 Unauthorized errors
- ✅ Both services sharing database
- ✅ Admin user in production database

---

## ⏱️ TIMELINE

| Phase | Duration | Total |
|-------|----------|-------|
| Read guide | 5-15 min | 5-15 min |
| Prepare | 5 min | 10-20 min |
| GitHub setup | 3 min | 13-23 min |
| Create on Render | 5 min | 18-28 min |
| Environment vars | 3 min | 21-31 min |
| Deploy | 2 min | 23-33 min |
| Wait for build | 3-5 min | 26-38 min |
| Admin user | 5 min | 31-43 min |
| Test endpoints | 5 min | 36-48 min |

**Total: 35-50 minutes**

---

## 🆘 IF SOMETHING GOES WRONG

### Deployment Failed
- [ ] Check GitHub Actions logs
- [ ] Look for Maven build errors
- [ ] Check if Java/Maven available
- [ ] See: STEP_BY_STEP_WALKTHROUGH.md → "If Something Goes Wrong"

### Can't Login (401 Error)
- [ ] Check admin user exists: `SELECT * FROM auth_user WHERE username='admin';`
- [ ] Check auth service is running (green in Render)
- [ ] Verify DATABASE_URL format is correct
- [ ] See: UNDERSTANDING_401_ERROR.md

### Can't Create Course
- [ ] Verify JWT token is valid
- [ ] Check JWT token from login
- [ ] Check Bearer prefix in Authorization header
- [ ] See: GIT_DEPLOYMENT_INSTRUCTIONS.md → Troubleshooting

---

## 📞 QUICK REFERENCE

### Key URLs
```
Render Dashboard: https://render.com/dashboard
GitHub Actions: https://github.com/lucy884/IDLPROJECT/actions
Auth Service: https://auth-service-xxxx.onrender.com
Course Service: https://course-service-fj4t.onrender.com
```

### Database Info
```
Host: dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com
Port: 5432
Database: course_db_1lil
User: course_db_user
Password: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
```

### GitHub Secrets
```
RENDER_API_KEY: (from Render account)
RENDER_AUTH_SERVICE_ID: (from auth service creation)
```

---

## 🎊 FINAL CHECKLIST

Before you declare success:
- [ ] Auth service deployed
- [ ] Can login with admin account
- [ ] JWT token received
- [ ] Course creation works
- [ ] No 401 errors
- [ ] Both services share database
- [ ] All endpoints tested
- [ ] Logs show no errors

---

## 🚀 YOU'RE READY!

**Follow this checklist exactly in order.**

**Estimated time: 35-50 minutes**

**You've got everything you need!**

**Go deploy!** 🎉

---

**Print this page if needed.** ✏️  
**Check off items as you complete them.** ✅  
**Reference back if you get stuck.** 📖

**Current Status: READY ✅**

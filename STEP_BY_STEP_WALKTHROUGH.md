# Step-by-Step Visual Deployment Walkthrough

## 📍 You Are Here: Problem & Solution

```
❌ CURRENT STATE                    ✅ GOAL STATE
───────────────────────            ─────────────
✅ Course Service                  ✅ Course Service
   - Deployed                         - Deployed
   - On Render                        - On Render
   
❌ Auth Service                     ✅ Auth Service
   - Local only                       - Deployed to Render
   - No admin in prod DB              - Connected to shared DB
   
❌ Admin Access                     ✅ Admin Access
   - 401 Unauthorized                - Login works
   - Can't create courses            - Can create courses
```

## 🚀 Complete Deployment Walkthrough

### Phase 1: Preparation (5 minutes)

#### Step 1a: Copy Database Credentials

**Location**: Render Dashboard → course-service → Environment

Look for these values:
```
DB_HOST: dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com
DB_NAME: course_db_1lil
DB_USER: course_db_user
DB_PASSWORD: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
DB_PORT: 5432
```

📝 **Write them down or keep this page open!**

```
Full JDBC URL: jdbc:postgresql://dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432/course_db_1lil
```

#### Step 1b: Generate JWT Secret

Open PowerShell and run:
```powershell
# Generate 32-byte random string in base64
[Convert]::ToBase64String([byte[]]@((1..32 | ForEach-Object { Get-Random -Maximum 256 })))
```

Example output:
```
AbCdEfGhIjKlMnOpQrStUvWxYz1234567890AbCdEfGhIjKlMnOpQrStUvWxYz1234567890
```

📝 **Copy this secret, you'll need it soon!**

#### Step 1c: Get Render API Key

1. Go to https://render.com
2. Click your **avatar** (top right)
3. Select **Account Settings**
4. Click **API Keys** (left sidebar)
5. Click **Create API Key**
6. Give it a name: `GitHub Deployment`
7. Copy the key
8. ✅ Save it as GitHub secret `RENDER_API_KEY`

---

### Phase 2: GitHub Actions Setup (3 minutes)

#### Step 2a: Verify Workflow File

Check that this file exists:
```
.github/workflows/deploy-auth-service.yml
```

✅ Already created for you!

#### Step 2b: Add GitHub Secrets

1. Go to your GitHub repository
2. Click **Settings** (top right)
3. Click **Secrets and variables** (left menu)
4. Click **Actions**
5. Click **New repository secret**

**Add Secret 1:**
```
Name: RENDER_API_KEY
Secret: (your API key from step 1c)
```
Click **Add secret**

**Add Secret 2:**
```
Name: RENDER_AUTH_SERVICE_ID
Secret: (you'll get this after creating the service)
```
Click **Add secret**

---

### Phase 3: Create Auth Service on Render (5 minutes)

#### Step 3a: Create Web Service

1. Go to https://render.com/dashboard
2. Click **New** (top right)
3. Select **Web Service**

```
┌─────────────────────────────────────────┐
│ Create a new web service                │
│                                         │
│ GitHub Repository:                      │
│ [Search/select your repo]               │
│                                         │
│ ✅ "Connect" button                     │
└─────────────────────────────────────────┘
```

#### Step 3b: Configure Service

```
┌──────────────────────────────────────────┐
│ Service Name: auth-service               │
│                                          │
│ Root Directory: auth                     │
│                                          │
│ Environment: Docker                      │
│ (It auto-detects your Dockerfile)        │
│                                          │
│ Instance Type: Free                      │
│                                          │
│ Auto-Deploy: Yes                         │
│                                          │
│ [Create Web Service] button              │
└──────────────────────────────────────────┘
```

✅ Click **Create Web Service**

#### Step 3c: Copy Service ID

Once created, you'll see:
```
Your web service: auth-service-xxxxxxxxxxxx
https://render.com/services/auth-service-xxxxxxxxxxxx
                            ^^^^^^^^^^^^^^^^^^^
                            THIS IS YOUR SERVICE ID
```

📝 **Copy this Service ID!**

Go to GitHub → Settings → Secrets → Add secret:
```
Name: RENDER_AUTH_SERVICE_ID
Secret: auth-service-xxxxxxxxxxxx
```

---

### Phase 4: Add Environment Variables (3 minutes)

#### Step 4a: Navigate to Environment Settings

1. In Render, click on your auth-service
2. Click **Settings** (top right)
3. Scroll down to **Environment**
4. Click **Add Environment Variable**

#### Step 4b: Add Variables (One by One)

**Variable 1:**
```
Key: DATABASE_URL
Value: jdbc:postgresql://dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432/course_db_1lil
```
Click ✅

**Variable 2:**
```
Key: DB_USER
Value: course_db_user
```
Click ✅

**Variable 3:**
```
Key: DB_PASSWORD
Value: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
```
Click ✅

**Variable 4:**
```
Key: JWT_SECRET
Value: (your generated secret from step 1b)
```
Click ✅

**Variable 5:**
```
Key: ALLOWED_ORIGINS
Value: http://localhost:3000,https://course-service-fj4t.onrender.com
```
Click ✅

**Variable 6:**
```
Key: JAVA_VERSION
Value: 17
```
Click ✅

```
After adding all variables, you should see:
┌──────────────────────────────┐
│ Environment Variables        │
│ ✅ DATABASE_URL             │
│ ✅ DB_USER                  │
│ ✅ DB_PASSWORD              │
│ ✅ JWT_SECRET               │
│ ✅ ALLOWED_ORIGINS          │
│ ✅ JAVA_VERSION             │
└──────────────────────────────┘
```

---

### Phase 5: Trigger Deployment (2 minutes)

#### Step 5a: Commit and Push

In your local machine:
```powershell
cd C:\Users\licya\OneDrive\Desktop\FinalProject

# Check what's new
git status

# Add all new files
git add .

# Commit with message
git commit -m "Deploy auth service to Render with PostgreSQL and JWT"

# Push to GitHub
git push origin main
```

✅ This triggers GitHub Actions automatically!

#### Step 5b: Monitor Deployment

1. Go to GitHub repository
2. Click **Actions** tab
3. Watch the workflow run:
   ```
   🟡 Pending → Building Maven
   🟡 Building → Running tests
   🟡 Testing → Deploying to Render
   ✅ Complete!
   ```

4. In Render dashboard, click auth-service:
   ```
   Status: Building (blue)
   ↓ (after a few minutes)
   Status: Live ✅ (green)
   ```

---

### Phase 6: Create Admin User in Production (5 minutes)

#### Step 6a: Connect to PostgreSQL Database

**Option A: Using DBeaver (GUI)**

1. Download [DBeaver](https://dbeaver.io/)
2. Click **New Database Connection** → PostgreSQL
3. Fill in:
   ```
   Host: dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com
   Port: 5432
   Database: course_db_1lil
   Username: course_db_user
   Password: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
   ```
4. Click **Test Connection** → OK
5. Proceed to Step 6b

**Option B: Using psql (Command Line)**

```powershell
# Install PostgreSQL client or use psql if you have PostgreSQL
psql -h dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com -U course_db_user -d course_db_1lil -p 5432
# Password: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
```

#### Step 6b: Create Admin User

Run this SQL command in DBeaver or psql:

```sql
-- Find the auth_user table
SELECT * FROM auth_user LIMIT 1;

-- If table exists, insert admin user
INSERT INTO auth_user (username, email, is_staff, is_superuser, date_joined) 
VALUES ('admin', 'admin@university.edu', true, true, NOW());

-- Verify it was created
SELECT * FROM auth_user WHERE username = 'admin';
```

✅ You should see the admin user in the results!

---

### Phase 7: Test Login (3 minutes)

#### Step 7a: Open Postman

#### Step 7b: Create Login Request

```
POST https://auth-service-xxxxxxxxxxxx.onrender.com/api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "password123"
}
```

**Expected Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "username": "admin",
  "email": "admin@university.edu"
}
```

✅ **Status: 200 OK** (not 401!)

#### Step 7c: Copy JWT Token

- Look for the `"token"` field in response
- Copy the entire token value
- You'll need it in the next step

---

### Phase 8: Test Course Creation (2 minutes)

#### Step 8a: Create Course Request

```
POST https://course-service-fj4t.onrender.com/api/courses/
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

{
  "name": "Introduction to Python",
  "description": "Learn Python basics and best practices",
  "credits": 3
}
```

**Expected Response:**
```json
{
  "id": 1,
  "name": "Introduction to Python",
  "description": "Learn Python basics and best practices",
  "credits": 3,
  "created_at": "2025-12-01T10:30:00Z"
}
```

✅ **Status: 201 Created** (not 401!)

---

## 🎉 Success Checklist

- ✅ Auth service deployed (green status in Render)
- ✅ Auth service connected to shared PostgreSQL
- ✅ Admin user created in production database
- ✅ Login endpoint returns JWT token (200 OK)
- ✅ Course creation works with JWT token (201 Created)
- ✅ No more 401 Unauthorized errors!

## 📊 Timeline

| Phase | Task | Time | Total |
|-------|------|------|-------|
| 1 | Preparation | 5 min | 5 min |
| 2 | GitHub Setup | 3 min | 8 min |
| 3 | Create Service | 5 min | 13 min |
| 4 | Environment Vars | 3 min | 16 min |
| 5 | Deploy | 2 min | 18 min |
| 6 | Admin User | 5 min | 23 min |
| 7 | Test Login | 3 min | 26 min |
| 8 | Test Course | 2 min | 28 min |

**Total Time: ~30 minutes** ⏱️

## 🔗 URLs to Bookmark

```
Render Dashboard:        https://render.com/dashboard
GitHub Repository:       https://github.com/lucy884/IDLPROJECT
GitHub Actions:          https://github.com/lucy884/IDLPROJECT/actions
Auth Service:            https://auth-service-xxxx.onrender.com
Course Service:          https://course-service-fj4t.onrender.com
PostgreSQL Database:     dpg-xxx.render.com (via DBeaver)
```

## 💡 Pro Tips

1. **Keep APIs open in Postman tabs** for quick testing
2. **Monitor Render logs** in real-time while deploying
3. **Save JWT tokens** temporarily in Postman for testing
4. **Check GitHub Actions** if deployment fails
5. **Verify environment variables** match your values exactly

## ❓ If Something Goes Wrong

### Auth service won't build
→ Check GitHub Actions logs
→ Look for Maven/Java errors
→ Verify pom.xml dependencies

### Can't login (401 error)
→ Verify admin user exists in database
→ Check if auth service is running (green in Render)
→ Verify DATABASE_URL is correct

### Can't create course
→ Make sure JWT token is valid
→ Check course service logs
→ Verify CORS settings

### Database connection fails
→ Test connection with DBeaver
→ Verify credentials are correct
→ Check if Render PostgreSQL service is running

## 🚀 What's Next?

After successful deployment of auth service:

1. Deploy remaining services:
   - Student Service
   - API Gateway
   - GraphQL Gateway
   
2. Configure inter-service communication

3. Deploy frontend

4. Test complete end-to-end flow

5. Set up monitoring and alerts

---

**You're ready! Start with Phase 1 and follow each step carefully.** 🎯

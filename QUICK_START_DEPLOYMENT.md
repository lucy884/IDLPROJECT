# Quick Deployment Reference - Auth & Course Services

## TL;DR - Steps to Deploy Auth Service and Fix 401 Error

### 1. **Copy Database Credentials from Course Service (Render)**

Go to your Render dashboard and find the PostgreSQL database credentials from the course service:
- Host: `dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com`
- Database: `course_db_1lil`
- User: `course_db_user`
- Password: `znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN`
- Port: `5432`

### 2. **Generate JWT Secret**

Run this to generate a secure JWT secret:
```bash
openssl rand -base64 32
```
Example output: `AbCdEfGhIjKlMnOpQrStUvWxYz1234567890AbCdEfGhIjKlMnOp==`

### 3. **Create Auth Service on Render**

1. Go to https://render.com → **New** → **Web Service**
2. Select your GitHub repository
3. Set:
   - Name: `auth-service`
   - Root Directory: `auth`
   - Runtime: Docker (it will auto-detect the Dockerfile)
4. Add these environment variables:
   ```
   DATABASE_URL=jdbc:postgresql://dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432/course_db_1lil
   DB_USER=course_db_user
   DB_PASSWORD=znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
   JWT_SECRET=<YOUR-GENERATED-SECRET-FROM-STEP-2>
   ALLOWED_ORIGINS=http://localhost:3000,https://course-service-fj4t.onrender.com
   JAVA_VERSION=17
   ```
5. Click **Create Web Service**
6. Copy the **Service ID** (you'll see it in the URL)

### 4. **Add GitHub Secrets**

1. Go to your GitHub repo → Settings → Secrets and variables → Actions
2. Add two secrets:
   - `RENDER_API_KEY`: [Get from Render account settings](https://render.com/docs/api#authentication)
   - `RENDER_AUTH_SERVICE_ID`: Your service ID from step 3

### 5. **Push Code to Trigger Deployment**

```bash
cd C:\Users\licya\OneDrive\Desktop\FinalProject
git add .
git commit -m "Deploy auth service to Render"
git push
```

GitHub Actions will automatically deploy to Render.

### 6. **Create Admin User in Production Database**

Once the service is deployed, create the admin user. **Choose one option:**

**Option A: Using PostgreSQL Client (Recommended)**
- Install [DBeaver](https://dbeaver.io/) or [pgAdmin](https://www.pgadmin.org/)
- Connect to `dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com`
- Run the SQL to create admin user (auth service should provide this)

**Option B: Using curl (if admin endpoint exists)**
```bash
curl -X POST https://auth-service-xxxx.onrender.com/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "YourStrongPassword123!",
    "email": "admin@university.edu",
    "role": "ADMIN"
  }'
```

**Option C: Direct SQL Insert**
Connect to the database and run:
```sql
INSERT INTO user_auth (username, password, email, is_admin) 
VALUES ('admin', 'hashed_password', 'admin@university.edu', true);
```

### 7. **Test Login in Postman**

```
POST https://auth-service-xxxx.onrender.com/api/auth/login
Content-Type: application/json

{
  "username": "admin",
  "password": "YourStrongPassword123!"
}
```

You should get a response with a JWT token:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "username": "admin"
}
```

### 8. **Test Course Creation with Token**

```
POST https://course-service-fj4t.onrender.com/api/courses/
Authorization: Bearer <TOKEN_FROM_STEP_7>
Content-Type: application/json

{
  "name": "Python 101",
  "description": "Learn Python",
  "credits": 3
}
```

Should return **201 Created** instead of **401 Unauthorized**!

## Why This Fixes Your 401 Error

| Before | After |
|--------|-------|
| Admin user only in local DB | Admin user in production DB |
| Services use separate databases | Services share same database |
| No JWT validation | JWT tokens properly validated |
| Result: **401 Unauthorized** | Result: **201 Created / 200 OK** |

## Service URLs After Deployment

- **Auth Service**: `https://auth-service-xxxx.onrender.com`
- **Course Service**: `https://course-service-fj4t.onrender.com`
- **PostgreSQL Database**: `dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432`

(Replace `xxxx` with your actual Render service ID)

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Auth service won't deploy | Check logs in Render dashboard → Logs tab |
| Can't connect to database | Verify DATABASE_URL format: `jdbc:postgresql://HOST:PORT/DB` |
| JWT token invalid | Ensure JWT_SECRET matches in both services |
| 401 after login | Create admin user in production database |
| CORS errors | Add frontend/service URLs to ALLOWED_ORIGINS |

## Files Created for Deployment

```
✅ auth/Dockerfile                     - Docker build configuration
✅ auth/DEPLOYMENT_GUIDE.md            - Detailed deployment guide
✅ auth/src/main/resources/application-prod.yml - Production config
✅ .github/workflows/deploy-auth-service.yml    - CI/CD pipeline
✅ RENDER_DEPLOYMENT_COMPLETE_GUIDE.md          - Full setup guide
✅ auth/pom.xml (updated)              - Added PostgreSQL driver
✅ auth/src/main/resources/application.yml (updated) - PostgreSQL config
```

## Next Steps After Successful Deployment

1. ✅ Deploy auth service
2. ✅ Create admin user
3. ✅ Test login workflow
4. ⬜ Integrate token validation in course service
5. ⬜ Deploy API Gateway (if needed)
6. ⬜ Test end-to-end flow
7. ⬜ Set up monitoring/alerts

## Need Help?

Check these files for more detailed info:
- **Full Deployment Guide**: `RENDER_DEPLOYMENT_COMPLETE_GUIDE.md`
- **Auth Service Docs**: `auth/DEPLOYMENT_GUIDE.md`
- **Course Service Docs**: `course_service/DEPLOYMENT_GUIDE.md`

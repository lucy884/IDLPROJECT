# Complete Deployment Setup Guide - Auth & Course Services on Render

## Problem: 401 Unauthorized Error

You're getting a 401 error when logging in because:
1. **Admin user doesn't exist in production** - Users created locally aren't in the deployed database
2. **Services use separate databases** - Different environments = different databases
3. **Authentication not properly configured** - Need to ensure JWT tokens are validated properly

## Solution: Deploy Both Services with Shared Database

### Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                  Render.com (Production)                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────┐         ┌──────────────────┐    │
│  │  Auth Service    │         │ Course Service   │    │
│  │  (Java Spring)   │         │  (Django REST)   │    │
│  │  Port: 8082      │         │  Port: 8000      │    │
│  └────────┬─────────┘         └────────┬─────────┘    │
│           │                            │               │
│           └────────────┬───────────────┘               │
│                        │                               │
│                   ┌────▼─────┐                        │
│                   │ PostgreSQL│                        │
│                   │  Database │ (Shared)              │
│                   └───────────┘                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Step 1: Verify Existing PostgreSQL Database

Your course service is already deployed. Get the connection details from Render:

1. Go to https://render.com dashboard
2. Click on your **course-service** web service
3. Scroll down to "Environment"
4. You should see:
   - `DB_HOST`: `dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com`
   - `DB_NAME`: `course_db_1lil`
   - `DB_USER`: `course_db_user`
   - `DB_PASSWORD`: `znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN`
   - `DB_PORT`: `5432`

**Keep these details - you'll need them for the auth service!**

### Step 2: Update Auth Service Application Configuration

Update `auth/src/main/resources/application-prod.yml` with your database credentials:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432/course_db_1lil
    username: course_db_user
    password: znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
```

### Step 3: Deploy Auth Service to Render

#### Step 3a: Create Web Service

1. Go to https://render.com dashboard
2. Click **New** → **Web Service**
3. Connect your GitHub repository (if not already connected)
4. Select the branch: `main`
5. Configure:
   - **Name**: `auth-service`
   - **Root Directory**: `auth`
   - **Runtime**: Docker
   - **Build Command**: (leave empty - uses Dockerfile)
   - **Start Command**: (leave empty - uses Dockerfile)

#### Step 3b: Add Environment Variables

After creating the service, go to **Settings** → **Environment** and add:

```
DATABASE_URL=jdbc:postgresql://dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com:5432/course_db_1lil
DB_USER=course_db_user
DB_PASSWORD=znWGDsmHgo0g7NYWRyBhb1tq0CNopzFN
JWT_SECRET=your-super-secret-jwt-key-min-32-chars-12345678901
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:9090,https://course-service-fj4t.onrender.com
JAVA_VERSION=17
```

#### Step 3c: Copy Service ID

1. Click on the auth service
2. Copy the **Service ID** from the URL or settings
3. Save it for GitHub secrets

#### Step 3d: Update GitHub Secrets

1. Go to your GitHub repository
2. Settings → **Secrets and variables** → **Actions**
3. Add/Update:
   - `RENDER_API_KEY`: Your Render API key
   - `RENDER_AUTH_SERVICE_ID`: Your auth service ID from Render

### Step 4: Deploy Auth Service from GitHub

1. Commit the files you created:
   ```bash
   git add auth/Dockerfile auth/DEPLOYMENT_GUIDE.md auth/src/main/resources/application-prod.yml
   git add .github/workflows/deploy-auth-service.yml
   git commit -m "Add auth service deployment configuration"
   git push
   ```

2. GitHub Actions will automatically trigger the deployment
3. Watch the deployment in the **Actions** tab

### Step 5: Create Admin User in Production

Once the auth service is deployed, you need to create an admin user. You have several options:

#### Option A: Direct PostgreSQL Access (Recommended for Testing)

1. Use a PostgreSQL client (DBeaver, pgAdmin, or psql)
2. Connect to: `dpg-d4ml8u3uibrs738mirs0-a.oregon-postgres.render.com`
3. Run SQL to insert admin user:

```sql
INSERT INTO auth_user (username, password, email, is_staff, is_superuser, date_joined)
VALUES (
  'admin',
  'pbkdf2_sha256$600000$yoursalt$hashedpassword',  -- Use Django password hash
  'admin@university.edu',
  true,
  true,
  NOW()
);
```

#### Option B: Render Shell Access

1. Go to your auth service on Render
2. Click the **Shell** tab
3. Create admin interactively (if Spring Boot provides CLI):
```bash
java -Dspring.profiles.active=prod -jar target/auth-service-0.0.1-SNAPSHOT.jar
```

#### Option C: Add Admin Creation Endpoint (Development)

Create a temporary endpoint in your auth service:

```java
@PostMapping("/admin/create")
public ResponseEntity<?> createAdmin(@RequestBody AdminRequest request) {
    // Only allow if no admin exists
    if(userRepository.existsByRole("ADMIN")) {
        return ResponseEntity.badRequest().build();
    }
    User admin = new User();
    admin.setUsername(request.getUsername());
    admin.setPassword(passwordEncoder.encode(request.getPassword()));
    admin.setRole("ADMIN");
    userRepository.save(admin);
    return ResponseEntity.ok("Admin created");
}
```

Then call:
```bash
curl -X POST https://auth-service-xxxx.onrender.com/api/auth/admin/create \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "strong-password"}'
```

**Remove this endpoint after creating the admin!**

### Step 6: Test Both Services

#### Test Auth Service Login

```bash
curl -X POST https://auth-service-xxxx.onrender.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your-password"}'
```

Response should include JWT token:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "username": "admin",
  "email": "admin@university.edu"
}
```

#### Test Course Service with Token

```bash
ADMIN_TOKEN=$(curl -s -X POST https://auth-service-xxxx.onrender.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "your-password"}' | jq -r '.token')

curl -X POST https://course-service-fj4t.onrender.com/api/courses/ \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Introduction to Python",
    "description": "Learn Python basics",
    "credits": 3
  }'
```

### Step 7: Update Course Service to Validate Auth Tokens

To make the course service validate tokens from the auth service:

1. Add a `AuthenticationFilter` in course service that validates JWT tokens
2. Use the same JWT secret in both services
3. Share the secret via environment variable

Update `course_service/settings.py`:

```python
# JWT Configuration - Must match auth service
JWT_SECRET = config('JWT_SECRET', default='your-secret-key')
JWT_ALGORITHM = 'HS256'
JWT_EXPIRATION = 86400  # 24 hours
```

### Step 8: Configure CORS Between Services

**Auth Service** (`application-prod.yml`):
```yaml
cors:
  allowed-origins: https://course-service-fj4t.onrender.com,http://localhost:3000
```

**Course Service** (`settings.py`):
```python
CORS_ALLOWED_ORIGINS = [
    "https://auth-service-xxxx.onrender.com",
    "http://localhost:9090",
]
```

## Troubleshooting Deployment

### Auth Service Deployment Fails

**Check logs:**
1. Go to auth service on Render
2. Click **Logs** tab
3. Look for:
   - `Java version issues`: Update pom.xml java.version
   - `Maven build fails`: Check dependencies
   - `Database connection fails`: Verify DATABASE_URL format

**Common fixes:**
```yaml
# In application-prod.yml - ensure URL format is correct
spring:
  datasource:
    url: jdbc:postgresql://HOST:PORT/DATABASE
```

### 401 Unauthorized Still Appears

**Causes:**
1. Admin user not created in production DB
2. JWT token not properly validated
3. CORS blocking requests

**Debug:**
```bash
# Check if admin exists
psql postgresql://course_db_user:password@host/course_db_1lil \
  -c "SELECT * FROM auth_user WHERE username='admin';"

# Check JWT secret matches
# Both services should have same JWT_SECRET environment variable
```

### Database Connection Refused

**Fix:**
1. Verify PostgreSQL service is running on Render
2. Check connection string format: `jdbc:postgresql://HOST:PORT/DB`
3. Verify firewall: Render allows PostgreSQL on port 5432

### Services Can't Communicate

**Fix:**
1. Use full URLs: `https://auth-service-xxxx.onrender.com/api/auth`
2. Configure CORS properly
3. Test with curl:
```bash
curl https://auth-service-xxxx.onrender.com/api/auth/health
```

## Production Checklist

- [ ] Auth service deployed on Render
- [ ] Auth service connected to shared PostgreSQL database
- [ ] Admin user created in production database
- [ ] JWT token generation works
- [ ] Course service validates auth tokens
- [ ] CORS properly configured
- [ ] Environment variables set correctly
- [ ] Both services tested end-to-end
- [ ] GitHub Actions workflows trigger on push
- [ ] Logs accessible and monitored

## Next Steps

1. **Deploy both services** following the steps above
2. **Create admin user** in production
3. **Test login workflow** with Postman
4. **Test course creation** with admin token
5. **Monitor logs** for any errors
6. **Document API keys** securely
7. **Set up monitoring** for uptime alerts

## Additional Resources

- [Render Documentation](https://render.com/docs)
- [Spring Boot Deployment](https://spring.io/guides/gs/spring-boot-docker/)
- [Django Production Deployment](https://docs.djangoproject.com/en/4.2/howto/deployment/)
- [PostgreSQL Connection Strings](https://www.postgresql.org/docs/current/libpq-connect.html)

## Support

If you encounter issues:
1. Check service logs on Render
2. Verify environment variables are set
3. Test database connectivity separately
4. Ensure JWT secrets match between services
5. Check CORS configuration in both services

# Auth Service - Deployment Guide

## Overview
This is a Spring Boot authentication service that handles user authentication and JWT token generation for the Campus Management System. It uses PostgreSQL as the database.

## Prerequisites
1. Java 17+
2. Maven 3.9+
3. PostgreSQL database
4. GitHub account with repository
5. Render account (https://render.com)

## Local Development

### 1. Install Dependencies
```bash
mvn clean install
```

### 2. Environment Variables / Application Configuration
The service uses `application.yml` in `src/main/resources/`. For local development:

```yaml
server:
  port: 8082

spring:
  application:
    name: auth-service
  
  datasource:
    url: jdbc:postgresql://localhost:5432/auth_db
    username: postgres
    password: your-password
    driver-class-name: org.postgresql.Driver
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

### 3. Run the Application
```bash
mvn spring-boot:run
```

The API will be available at `http://127.0.0.1:8082/`

## Deployment on Render

### Step 1: Create PostgreSQL Database on Render (Shared with Course Service)
Since you already have a PostgreSQL database for the course service on Render, you should use the SAME database for the auth service to stay within the free plan limits.

**Option A: Use Existing Database (Recommended)**
- Your existing Render PostgreSQL database URL
- Create a separate schema or use a different table prefix for auth service

**Option B: Create New Database (if needed)**
1. In Render dashboard, click **New** → **PostgreSQL**
2. Enter database name (e.g., `campus-auth-db`)
3. Select Instance Type: **Free**
4. Click **Deploy PostgreSQL Instance**
5. Copy the database credentials

### Step 2: Update application-prod.yml

Create `src/main/resources/application-prod.yml` for production configuration:

```yaml
server:
  port: 8082
  servlet:
    context-path: /api/auth

spring:
  application:
    name: auth-service
  
  datasource:
    url: ${DATABASE_URL}
    hikari:
      maximum-pool-size: 5

  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.PostgreSQLDialect
        format_sql: false

jwt:
  secret: ${JWT_SECRET}
  expiration: 86400000

cors:
  allowed-origins: ${ALLOWED_ORIGINS}

logging:
  level:
    root: INFO
```

### Step 3: Create Web Service on Render

1. Click **New** → **Web Service**
2. Connect your GitHub repository
3. Configure the service:
   - **Name**: auth-service
   - **Root Directory**: auth
   - **Build Command**: `mvn clean package -DskipTests`
   - **Start Command**: `java -Dspring.profiles.active=prod -jar target/auth-service-0.0.1-SNAPSHOT.jar`
4. Add Environment Variables:
   - `DATABASE_URL`: `jdbc:postgresql://[HOST]:[PORT]/[DB_NAME]`
     - Example: `jdbc:postgresql://dpg-xxx.postgres.render.com:5432/course_db_1lil`
   - `DB_USER`: Your database username
   - `DB_PASSWORD`: Your database password
   - `JWT_SECRET`: Generate a secure secret
     - Generate with: `openssl rand -base64 32`
   - `ALLOWED_ORIGINS`: `http://localhost:3000,https://your-frontend-domain.com`
   - `JAVA_VERSION`: 17

5. Click **Create Web Service**
6. Copy the **Service ID**

### Step 4: Configure CORS and JWT

Update your environment variables to match your deployment:

```
ALLOWED_ORIGINS=https://course-service-fj4t.onrender.com,https://your-frontend-domain.com
JWT_SECRET=your-secure-secret-key
```

### Step 5: Add Secrets to GitHub

1. Go to your GitHub repository
2. Settings → **Secrets and variables** (left sidebar) → **Actions**
3. Create secrets:
   - `RENDER_API_KEY`: Your Render API key
   - `RENDER_AUTH_SERVICE_ID`: Your auth service ID from Render

### Step 6: CI/CD Workflow

The `.github/workflows/deploy-auth-service.yml` file will:
- Run tests on every push to main
- Build with Maven
- Deploy automatically to Render

## API Endpoints

### Authentication
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login and get JWT token
- `POST /api/auth/refresh` - Refresh JWT token
- `POST /api/auth/logout` - Logout

### User Management (Admin Only)
- `GET /api/users` - List all users
- `GET /api/users/{id}` - Get user details
- `PUT /api/users/{id}` - Update user
- `DELETE /api/users/{id}` - Delete user

## Integration with Course Service

### 1. Database Sharing
Both services use the same PostgreSQL database on Render:
- Auth service: User authentication and token management
- Course service: Course and enrollment management
- They use separate tables but can share the user table if needed

### 2. JWT Token Validation
The Course service can validate JWT tokens issued by the Auth service by:
1. Storing the JWT secret in both services (same value)
2. Using the same JWT library and configuration
3. Implementing a filter to check token validity before processing requests

### 3. Communication Between Services
For inter-service communication on Render:
- Use Render's internal networking
- Get service URLs from Render dashboard
- Example: `https://auth-service-xxxx.onrender.com/api/auth`

## Troubleshooting

### Build Failures
- Ensure Java 17 is installed: `java -version`
- Clear Maven cache: `mvn clean`
- Check Maven settings: `mvn --version`

### Database Connection Issues
- Verify DATABASE_URL format
- Test connection with psql: `psql "postgresql://[url]"`
- Check Render database service logs

### JWT Token Issues
- Ensure JWT_SECRET is the same in all services
- Check token expiration time
- Verify ALLOWED_ORIGINS for CORS

### Logs
Check Render logs in the dashboard to debug any issues during deployment:
1. Click on the auth service
2. Click **Logs** tab
3. View real-time deployment and application logs

## Production Checklist

- [ ] Set `DEBUG=false` in production
- [ ] Use strong `JWT_SECRET` (minimum 32 characters)
- [ ] Set appropriate `ALLOWED_ORIGINS` for CORS
- [ ] Enable HTTPS (Render provides free SSL)
- [ ] Test all endpoints on deployed service
- [ ] Monitor logs and errors
- [ ] Set up database backups
- [ ] Create admin user in production
- [ ] Test token refresh mechanism
- [ ] Verify inter-service communication

## Creating Admin User in Production

To create an admin user in the deployed auth service:

**Option 1: API Endpoint (if available)**
```bash
curl -X POST https://auth-service-xxxx.onrender.com/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "strong-password",
    "email": "admin@example.com"
  }'
```

**Option 2: Database Direct Access**
1. Connect to the Render PostgreSQL database using a tool like pgAdmin
2. Insert user directly into users table with admin role

**Option 3: Render Shell**
1. Go to your auth service on Render
2. Click **Shell** tab
3. Run a command to create a superuser (if Spring provides this functionality)

## Performance Optimization

For the free tier on Render:
- Use connection pooling (HikariCP)
- Set `maximum-pool-size: 5` for development
- Enable query caching where possible
- Consider upgrading to paid tier for production

## References
- [Render Deployment Guide](https://render.com/docs)
- [Spring Boot Production Deployment](https://spring.io/guides/gs/spring-boot/)
- [PostgreSQL on Render](https://render.com/docs/databases)

# Multi-stage build for Java Spring Boot Auth Service

# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17 as builder

WORKDIR /app

# Copy pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:resolve

# Copy source code
COPY src ./src

# Build the application
RUN mvn clean package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Install curl for health checks
RUN apk add --no-cache curl

# Copy the JAR file from builder
COPY --from=builder /app/target/*.jar app.jar

# Expose port
EXPOSE 8082

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8082/actuator/health || exit 1

# Run the application with increased JVM memory
ENTRYPOINT ["java", "-Dspring.profiles.active=prod", "-Xmx512m", "-Xms256m", "-jar", "app.jar"]

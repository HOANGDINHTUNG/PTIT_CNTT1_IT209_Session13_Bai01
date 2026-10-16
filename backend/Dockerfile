# ==========================================
# Stage 1: Build application with Gradle & JDK 21
# ==========================================
FROM gradle:8.14-jdk21-alpine AS builder

WORKDIR /app

# Copy build configuration
COPY build.gradle settings.gradle ./
COPY gradle ./gradle

# Copy source code
COPY src ./src

# Build production jar
RUN gradle bootJar --no-daemon -x test

# ==========================================
# Stage 2: Runtime environment with JRE 21
# ==========================================
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Security best practice: create a dedicated non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy compiled jar from builder stage
COPY --from=builder /app/build/libs/*.jar app.jar

# Set correct permissions
RUN chown -R appuser:appgroup /app

# Run as non-root user
USER appuser:appgroup

# Expose backend application port
EXPOSE 8080

# Healthcheck probe to monitor application status
HEALTHCHECK --interval=15s --timeout=5s --start-period=30s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# Launch Spring Boot application
ENTRYPOINT ["java", "-jar", "app.jar"]

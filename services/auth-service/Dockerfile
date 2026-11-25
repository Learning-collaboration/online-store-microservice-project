# ==========================================
# Stage 1: Build Stage (Dedicated Chainguard Maven Image)
# ==========================================
FROM cgr.dev/chainguard/maven:latest AS builder

USER root
WORKDIR /app

# Copy pom.xml first to optimize Docker layer dependency caching
COPY pom.xml ./

# Download dependencies offline so they are cached independently of source code changes
RUN mvn dependency:go-offline -B

# Copy the rest of the application source code
COPY src ./src

# Compile and package the application into an executable jar (skipping tests for speed)
RUN mvn package -DskipTests

# ==========================================
# Stage 2: Production Runtime (Clean JRE, Zero Build Tools)
# ==========================================
FROM cgr.dev/chainguard/jre:latest

WORKDIR /app

# Copy the compiled JAR artifact from the builder stage
COPY --from=builder /app/target/*.jar app.jar

# Run under the secure non-root user native to Chainguard JRE images
USER java

# Expose service port (adjust if your auth service uses a different port)
EXPOSE 8081

# Execute the Spring Boot application with container-aware JVM memory tuning flags
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
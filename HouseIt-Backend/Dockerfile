# Build JAR with Java 17
FROM gradle:8-jdk17 AS builder
WORKDIR /app
COPY . .
RUN gradle clean bootJar -x test --no-daemon

# Extract layers
FROM eclipse-temurin:17-jre AS layers
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

# Run with Java 17
FROM eclipse-temurin:17-jre
WORKDIR /app

# Security: run as non-root user
RUN addgroup --system --gid 1001 spring && \
    adduser --system --uid 1001 --ingroup spring --shell /bin/sh spring

# OPTIMIZATION: Copy layers separately to utilize Docker caching
# 1. Dependencies (libraries that rarely change)
COPY --from=layers /app/dependencies/ ./
# 2. Snapshot dependencies (if any)
COPY --from=layers /app/snapshot-dependencies/ ./
# 3. Spring Boot loader
COPY --from=layers /app/spring-boot-loader/ ./
# 4. Your Application Code (changes frequently)
COPY --from=layers /app/application/ ./

USER spring:spring

ENV SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/house_it \
    SPRING_DATASOURCE_USERNAME=postgres \
    SPRING_DATASOURCE_PASSWORD=HouseIt2024 \
    SERVER_PORT=8080

EXPOSE 8080

ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]

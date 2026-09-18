# Build stage
FROM eclipse-temurin:25-jdk-alpine AS build
WORKDIR /workspace

# Maven Wrapper downloads the Maven distribution over HTTPS on first run; Alpine's base
# image ships without CA certificates, which breaks that TLS handshake.
RUN apk add --no-cache ca-certificates

COPY .mvn/ .mvn/
COPY mvnw pom.xml ./
RUN ./mvnw -q -B dependency:go-offline

COPY src/ src/
# tests/backend/ (the INT standard test tree) lives outside this build context (src/backend/),
# so it isn't available here. Production images don't need it: skip compiling/running tests entirely.
RUN ./mvnw -q -B package -Dmaven.test.skip=true

# Runtime stage
FROM eclipse-temurin:25-jre-alpine
WORKDIR /app
COPY --from=build /workspace/target/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]

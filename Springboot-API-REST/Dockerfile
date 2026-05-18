# Stage 1: Build con Maven y Java 17 (Alpine para optimizar capas)
FROM maven:3.9.6-eclipse-temurin-17-alpine AS build
WORKDIR /app

# Copiar pom.xml para cachear dependencias de Maven
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copiar código fuente y compilar omitiendo tests para acelerar el pipeline
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Runtime ligero y seguro (Mínimo privilegio)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Cumplir con usuario no-root (Seguridad)
RUN addgroup -S innovatech && adduser -S innovatech -G innovatech
USER innovatech:innovatech

# Copiar el jar compilado desde la etapa de build
COPY --from=build /app/target/*.jar app.jar

ENV JAVA_OPTS="-XX:+UseG1GC -XX:+ExitOnOutOfMemoryError"
EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
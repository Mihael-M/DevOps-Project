# ---- Build stage ----
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY . .
RUN mvn -q -DskipTests package

# ---- Run stage ----
FROM eclipse-temurin:17-jre
WORKDIR /app

RUN useradd -m -u 10001 appuser

COPY --from=build /app/target/*.jar /app/app.jar
RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
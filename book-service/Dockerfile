FROM maven:3.9.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn -q -DskipTests package

# Runtime: on réutilise une image que tu télécharges déjà (maven)
FROM maven:3.9.9-eclipse-temurin-21
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

# curl est déjà présent souvent, sinon on peut l’installer (mais ça déclenche apt)
# RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

ENV SPRING_PROFILES_ACTIVE=docker
ENV SERVER_PORT=8081
EXPOSE 8081
ENTRYPOINT ["java","-jar","/app/app.jar"]

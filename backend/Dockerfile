FROM maven:3.9-eclipse-temurin-11 AS build

WORKDIR /app

# Usa o Maven já instalado na imagem (em vez do wrapper ./mvnw, que conflita
# com o MAVEN_CONFIG desta imagem oficial). Copia primeiro só o pom.xml para
# aproveitar o cache de camadas do Docker ao baixar as dependências.
COPY pom.xml .
RUN mvn -B -q dependency:go-offline

COPY src src
RUN mvn -B -q clean package -DskipTests

FROM eclipse-temurin:11-jre-alpine

WORKDIR /app

COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]

# STAGE 1 : Build de l'application 
FROM maven:3.9.6-eclipse-temurin-17 AS build

WORKDIR /app

COPY pom.xml .

# On télécharge les dépendances
RUN mvn dependency:go-offline

# On copie le code source et on lance la compilation
COPY src ./src
RUN mvn clean package -DskipTests

# STAGE 2 : Run de l'application 
# On repart d'une image JRE beaucoup plus légère pour l'exécution
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# On récupère uniquement le fichier .jar généré au stage précédent
COPY --from=build /app/target/*.jar app.jar

EXPOSE 8080

# Commande de lancement
ENTRYPOINT ["java", "-jar", "app.jar"]
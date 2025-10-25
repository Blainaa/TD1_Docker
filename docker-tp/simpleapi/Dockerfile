# --- Étape 1 : Build ---
# Utilise une image avec un JDK complet et Maven
FROM eclipse-temurin:21-jdk-alpine AS myapp-build
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME

# Installe maven
RUN apk add --no-cache maven

# Copie le pom.xml et le code source
COPY pom.xml .
COPY src ./src

# Construit l'application (le .jar)
RUN mvn package -DskipTests

# --- Étape 2 : Run ---
# Utilise une image légère avec seulement le JRE
FROM eclipse-temurin:21-jre-alpine
ENV MYAPP_HOME=/opt/myapp
WORKDIR $MYAPP_HOME

# Copie UNIQUEMENT le .jar de l'étape de build
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

# Commande pour lancer l'application
ENTRYPOINT ["java", "-jar", "myapp.jar"]
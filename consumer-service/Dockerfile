# Usa una imagen base de JDK 17 disponible
FROM eclipse-temurin:17-jdk

# Argumento para el JAR (Gradle genera en build/libs)
ARG JAR_FILE=build/libs/*.jar

WORKDIR /app

COPY ${JAR_FILE} app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
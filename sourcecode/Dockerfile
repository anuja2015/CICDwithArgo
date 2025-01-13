# Use a base image with Java (JDK 17)
FROM openjdk:17-jdk-slim


# Add the jar file created by Maven into the container
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar

# Expose the application's port (default Spring Boot port is 8080)
EXPOSE 3010

# Run the Spring Boot application
ENTRYPOINT ["java", "-jar", "/app.jar"]


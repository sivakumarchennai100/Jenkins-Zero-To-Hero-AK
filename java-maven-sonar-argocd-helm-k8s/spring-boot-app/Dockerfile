# You can change this base image to anything else
# But make sure to use the correct version of Java
#FROM adoptopenjdk/openjdk11:alpine-jre

# Simply the artifact path
#ARG artifact=target/spring-boot-web.jar

#WORKDIR /opt/app

#COPY ${artifact} app.jar

# This should not be changed
#ENTRYPOINT ["java","-jar","app.jar"]

FROM eclipse-temurin:17-jdk

RUN apt-get update && apt-get install -y maven
ENV JAVA_HOME=/opt/java/openjdk

# Define the build-time JAR location
ARG JAR_FILE=target/spring-boot-web.jar

# Working directory inside the container
WORKDIR /opt/app

# Copy the JAR from build context into the image
COPY ${JAR_FILE} app.jar

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]


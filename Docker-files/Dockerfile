# Stage 1: Build the application
FROM maven:3.9.9-eclipse-temurin-21-jammy AS BUILD_IMAGE

WORKDIR /app
RUN git clone https://github.com/mohammedsalmanj/java-portfolio-application.git
WORKDIR /app/java-portfolio-application
RUN git checkout main && mvn clean package -DskipTests

# Stage 2: Run the application
FROM tomcat:10-jdk21

RUN rm -rf /usr/local/tomcat/webapps/*

COPY --from=BUILD_IMAGE /app/java-portfolio-application/target/*.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]
# End of Dockerfile
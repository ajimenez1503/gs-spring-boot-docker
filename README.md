# gs-spring-boot-docker
Getting started spring boot docker
https://spring.io/guides/gs/spring-boot-docker/

### Run the application locally
- Option 1:
```shell
./mvnw spring-boot:run

curl  http://localhost:8080/home
```
- Option 2:
```shell
./mvnw package
java -jar target/hello-0.0.1-SNAPSHOT.jar

curl http://localhost:8080/home
```

### Containerize It
- Create `Dockerfile`
```shell
FROM openjdk:11
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
```shell
docker build -t ajimenez15/gs-spring-boot-docker .
```

### Build a Docker Image with Maven
```shell
docker login -u ajimenez15
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=ajimenez15/gs-spring-boot-docker
```

### Run the docker 
```shell
docker run -dp 8080:8080 -t ajimenez15/gs-spring-boot-docker
docker ps

curl http://localhost:8080/home
docker rm -f 4ca96e6d22b8
```
- Using Spring Profiles
```shell
docker run -e "SPRING_PROFILES_ACTIVE=prod" -dp 8080:8080 -t ajimenez15/gs-spring-boot-docker
docker ps

curl http://localhost:8080/home
docker rm -f 4ca96e6d22b8
```
- Debugging the Application in a Docker Container
```shell
docker run -e "JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,address=5005,server=y,suspend=n" -dp 8080:8080 -t ajimenez15/gs-spring-boot-docker
docker ps

curl http://localhost:8080/home
docker rm -f 4ca96e6d22b8
```
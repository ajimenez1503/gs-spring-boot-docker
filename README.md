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

curl  http://localhost:8080/home
```

### Containerize It
- Create `Dockerfile`
```shell
FROM openjdk:8-jdk-alpine
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","hello.Application"]
```
```shell
docker build -t ajimenez15/gs-spring-boot-docker .
```

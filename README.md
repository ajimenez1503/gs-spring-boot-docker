# gs-spring-boot-docker
Getting started spring boot docker
https://spring.io/guides/gs/spring-boot-docker/
https://spring.io/guides/topicals/spring-boot-docker

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

### Peek into the container
```shell
docker run --name myapp -dp 8080:8080 -t ajimenez15/gs-spring-boot-docker
docker exec -ti myapp /bin/sh
docker ps

curl http://localhost:8080/home
docker rm -f 4ca96e6d22b8
```

### Option to add Java command line options at runtime
- New Dockerfile2
```shell
FROM openjdk:11
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["sh", "-c", "java ${JAVA_OPTS:-}  -jar /app.jar"]
```
- Build and run
```shell
docker build -t ajimenez15/gs-spring-boot-docker2 -f Dockerfile2 .
docker run -e JAVA_OPTS="-Dserver.port=9000" -p 9000:9000  ajimenez15/gs-spring-boot-docker2
docker ps

curl http://localhost:9000/home
docker rm -f 4ca96e6d22b8
```

### Option to add Java command arguments at runtime
- New Dockerfile3
```shell
FROM openjdk:11
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["sh", "-c", "java -jar /app.jar ${0} ${@}"]
```
- Build and run
```shell
docker build -t ajimenez15/gs-spring-boot-docker3 -f Dockerfile3 .
docker run -p 9000:9000  ajimenez15/gs-spring-boot-docker3 --server.port=9000
docker ps

curl http://localhost:9000/home
docker rm -f 042f6d6dbd64
```

### Faster build
- Extract the layer of the jar:
```shell
mkdir target/dependency
cd target/dependency
jar -xf ../*.jar
```
- New Dockerfile4
```shell
FROM openjdk:11
VOLUME /tmp
ARG DEPENDENCY=target/dependency
COPY ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY ${DEPENDENCY}/META-INF /app/META-INF
COPY ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","com.example.hello.HelloApplication"]
```
- Build and run
```shell
docker build -t ajimenez15/gs-spring-boot-docker4 -f Dockerfile4 .
docker run -p 8080:8080 ajimenez15/gs-spring-boot-docker4
curl http://localhost:8080/home
```

### Spring Boot Layer Index
- Extract the layer of the jar:
```shell
mkdir target/extracted
java -Djarmode=layertools -jar target/*.jar extract --destination target/extracted
```
- New Dockerfile5
```shell
FROM openjdk:11
VOLUME /tmp
ARG EXTRACTED=target/extracted
COPY ${EXTRACTED}/dependencies/ ./
COPY ${EXTRACTED}/spring-boot-loader/ ./
COPY ${EXTRACTED}/snapshot-dependencies/ ./
COPY ${EXTRACTED}/application/ ./
ENTRYPOINT ["java","org.springframework.boot.loader.JarLauncher"]
```
- Build and run
```shell
docker build -t ajimenez15/gs-spring-boot-docker5 -f Dockerfile5 .
docker run -p 8080:8080 ajimenez15/gs-spring-boot-docker5
curl http://localhost:8080/home
```

### Build and deploy with Dockers
- New Dockerfile6
```shell
FROM openjdk:11 as build
WORKDIR /workspace/app

COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .
COPY src src

RUN ./mvnw install -DskipTests
RUN mkdir -p target/dependency && (cd target/dependency; jar -xf ../*.jar)

FROM openjdk:11
VOLUME /tmp
ARG DEPENDENCY=/workspace/app/target/dependency
COPY --from=build ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=build ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=build ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","com.example.hello.HelloApplication"]
```
```shell
docker build -t ajimenez15/gs-spring-boot-docker6 -f Dockerfile6 .
docker run -p 8080:8080 ajimenez15/gs-spring-boot-docker6
curl http://localhost:8080/home
```

### Spring Boot Maven docker plugin
```shell
./mvnw spring-boot:build-image -Dspring-boot.build-image.imageName=gs-spring-boot-docker7
docker run -p 8080:8080 -t gs-spring-boot-docker7

```
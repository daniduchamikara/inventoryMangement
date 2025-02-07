# Deploying Spring Boot with MongoDB and Eureka Server using Docker

## Project Description
This project demonstrates how to deploy a **Spring Boot microservice** with **MongoDB** using Docker and Docker Compose. Additionally, it includes an **Eureka Server** to manage service discovery for microservices architecture.

## Prerequisites
- Docker installed
- Maven installed
- MongoDB CLI (optional for testing)

## Step 1: Create Dockerfile
Inside your project root, create a `Dockerfile`:

```dockerfile
# Use official OpenJDK image
FROM openjdk:17-jdk-slim

# Set working directory inside the container
WORKDIR /app

# Copy the packaged JAR file into the container
COPY target/inventory_1-0.0.1-SNAPSHOT.jar app.jar

# Expose the application port 
EXPOSE 8080

# Run the application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Step 2: Create docker-compose.yml
Create a `docker-compose.yml` file in the project root:

```yaml
version: '3.8'

services:
  eureka-server:
    image: openjdk:17-jdk-slim
    container_name: eureka_server
    restart: always
    ports:
      - "8761:8761"
    environment:
      - EUREKA_CLIENT_REGISTER_WITH_EUREKA=false
      - EUREKA_CLIENT_FETCH_REGISTRY=false
    command: ["java", "-jar", "eureka-server.jar"]

  mongodb:
    image: mongo:6.0
    container_name: mongodb_container
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    volumes:
      - mongo_data:/data/db

  springboot-app:
    build: .
    container_name: springboot_container
    restart: always
    ports:
      - "8080:8080"
    depends_on:
      - mongodb
      - eureka-server
    environment:
      SPRING_DATA_MONGODB_URI: mongodb://root:example@mongodb:27017/inventory_db?authSource=admin
      EUREKA_CLIENT_SERVICEURL_DEFAULTZONE: http://eureka-server:8761/eureka/
    command: ["java", "-jar", "app.jar"]

volumes:
  mongo_data:
```

## Step 3: Configure application.yml
Inside `src/main/resources/application.yml`, configure MongoDB and Eureka settings:

```yaml
server:
  port: 8080

spring:
  application:
    name: inventory-service
  data:
    mongodb:
      uri: mongodb://root:example@mongodb:27017/inventory_db?authSource=admin

eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka-server:8761/eureka/
  instance:
    preferIpAddress: true
```

## Step 4: Build and Run the Project

### 1️⃣ Build the Spring Boot JAR file
Run this command inside your project root:
```sh
mvn clean package
```

### 2️⃣ Build and Run Docker Containers
```sh
docker-compose up --build
```

### 3️⃣ Verify Running Containers
```sh
docker ps
```

### 4️⃣ Check Logs
To check logs of the Spring Boot container:
```sh
docker logs -f springboot_container
```

### 5️⃣ Test MongoDB Connection
If you have MongoDB CLI installed, run:
```sh
mongo --username root --password example --host localhost --port 27017
```

### 6️⃣ Access Eureka Server Dashboard
Open your browser and go to:
```
http://localhost:8761/
```

### 7️⃣ Test the Spring Boot Application
Open your browser or Postman and go to:
```
http://localhost:8080/
```

## Step 5: Stopping and Restarting Containers
- To stop all containers:
  ```sh
  docker-compose down
  ```
- To restart the containers:
  ```sh
  docker-compose up -d
  ```


# Docker Guide for Developers

Docker is a platform for developing, shipping, and running applications in containers. Containers package software with all its dependencies, ensuring consistency across different environments.

## Core Concepts

### What is Docker?

- **Container**: Lightweight, standalone executable package
- **Image**: Read-only template for creating containers
- **Dockerfile**: Script with instructions to build an image
- **Registry**: Repository for storing and distributing images (Docker Hub)

## Installation

### Linux
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

### Verify Installation
```bash
docker --version
docker run hello-world
```

## Basic Commands

### Images

```bash
# List images
docker images

# Pull an image
docker pull nginx:latest

# Build an image
docker build -t myapp:1.0 .

# Remove an image
docker rmi image_name

# Tag an image
docker tag myapp:1.0 username/myapp:1.0

# Push to registry
docker push username/myapp:1.0
```

### Containers

```bash
# Run a container
docker run -d -p 8080:80 --name webserver nginx

# List running containers
docker ps

# List all containers
docker ps -a

# Stop a container
docker stop webserver

# Start a container
docker start webserver

# Remove a container
docker rm webserver

# View logs
docker logs webserver

# Execute command in container
docker exec -it webserver bash

# Inspect container
docker inspect webserver
```

## Dockerfile

### Basic Structure

```dockerfile
# Base image
FROM openjdk:17-jdk-slim

# Set working directory
WORKDIR /app

# Copy files
COPY target/myapp.jar app.jar

# Expose port
EXPOSE 8080

# Run command
CMD ["java", "-jar", "app.jar"]
```

### Multi-stage Build (Java)

```dockerfile
# Build stage
FROM maven:3.8-openjdk-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Runtime stage
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Node.js Application

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Start application
CMD ["node", "server.js"]
```

### Python Application

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

## Docker Compose

### Basic docker-compose.yml

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    depends_on:
      - db
    volumes:
      - ./logs:/app/logs

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

### Full Stack Application

```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://db:5432/appdb
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=password
      - REDIS_HOST=redis
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - db_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - frontend
      - backend

volumes:
  db_data:
```

### Docker Compose Commands

```bash
# Start services
docker-compose up

# Start in detached mode
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# Rebuild images
docker-compose build

# Scale services
docker-compose up -d --scale backend=3

# Execute command
docker-compose exec backend bash
```

## Networking

### Create Network

```bash
docker network create mynetwork
```

### Run Container on Network

```bash
docker run -d --name db --network mynetwork postgres
docker run -d --name app --network mynetwork myapp
```

### Network Types

- **bridge**: Default network for containers
- **host**: Container uses host's network
- **none**: No networking
- **overlay**: Multi-host networking (Swarm)

## Volumes

### Named Volumes

```bash
# Create volume
docker volume create mydata

# Use volume
docker run -v mydata:/app/data myapp

# List volumes
docker volume ls

# Remove volume
docker volume rm mydata
```

### Bind Mounts

```bash
# Mount host directory
docker run -v /host/path:/container/path myapp

# Read-only mount
docker run -v /host/path:/container/path:ro myapp
```

## Best Practices

### 1. Use Official Base Images

```dockerfile
FROM node:18-alpine  # Good
FROM ubuntu:latest   # Avoid
```

### 2. Minimize Layers

```dockerfile
# Bad
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# Good
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*
```

### 3. Use .dockerignore

```
node_modules
npm-debug.log
.git
.env
*.md
.DS_Store
```

### 4. Don't Run as Root

```dockerfile
FROM node:18-alpine

# Create user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set ownership
COPY --chown=nodejs:nodejs . .

# Switch to user
USER nodejs

CMD ["node", "server.js"]
```

### 5. Use Multi-stage Builds

Reduces final image size by separating build and runtime environments.

### 6. Leverage Build Cache

```dockerfile
# Copy dependency files first
COPY package*.json ./
RUN npm install

# Then copy source code
COPY . .
```

### 7. Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/health || exit 1
```

## Security

### 1. Scan Images

```bash
docker scan myapp:latest
```

### 2. Use Specific Tags

```dockerfile
FROM node:18.16.0-alpine  # Good
FROM node:latest          # Avoid
```

### 3. Limit Resources

```bash
docker run -m 512m --cpus=1 myapp
```

### 4. Read-only Filesystem

```bash
docker run --read-only myapp
```

## Debugging

### View Container Logs

```bash
docker logs -f container_name
```

### Inspect Container

```bash
docker inspect container_name
```

### Execute Shell

```bash
docker exec -it container_name sh
```

### View Resource Usage

```bash
docker stats
```

### Copy Files

```bash
# From container to host
docker cp container_name:/app/file.txt ./

# From host to container
docker cp ./file.txt container_name:/app/
```

## CI/CD Integration

### GitHub Actions

```yaml
name: Docker Build and Push

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: username/myapp:latest
```

## Common Patterns

### Database Initialization

```yaml
services:
  db:
    image: postgres:15
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
      - db_data:/var/lib/postgresql/data
```

### Environment Variables

```yaml
services:
  app:
    image: myapp
    env_file:
      - .env
    environment:
      - NODE_ENV=production
```

### Restart Policies

```yaml
services:
  app:
    image: myapp
    restart: unless-stopped
```

## Troubleshooting

### Container Exits Immediately

```bash
# Check logs
docker logs container_name

# Run interactively
docker run -it myapp sh
```

### Port Already in Use

```bash
# Find process using port
sudo lsof -i :8080

# Use different port
docker run -p 8081:8080 myapp
```

### Out of Disk Space

```bash
# Remove unused containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Clean everything
docker system prune -a --volumes
```

## Production Deployment

### Docker Swarm

```bash
# Initialize swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.yml myapp

# Scale service
docker service scale myapp_web=3
```

### Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: username/myapp:latest
        ports:
        - containerPort: 8080
```

---

- Go back to [Frameworks](./index.md)
- Return to [Home](../../index.md)

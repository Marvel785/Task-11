# Spring Pet Clinic - CI/CD with Jenkins & Docker

This project demonstrates a complete CI/CD pipeline for the Spring Pet Clinic application using Jenkins, Docker, and Docker Compose. The pipeline automatically builds, tests, and deploys the application with a MySQL database backend.

## 🏗️ Architecture Overview

The project uses a multi-container setup with:
- **Spring Boot Application** (Pet Clinic) running on port 9090
- **MySQL Database** running on port 3306
- **Docker multi-stage build** for optimized application images
- **Jenkins pipeline** for automated CI/CD

## 📋 Prerequisites

Before running this project, ensure you have:

- Jenkins server with Docker and Docker Compose installed
- Docker Engine running on the Jenkins agent
- Git access to the Spring Pet Clinic repository
- Sufficient disk space for Docker images and containers

### Jenkins Plugin Requirements
- Docker Pipeline Plugin
- Git Plugin
- Pipeline Plugin

## 🚀 Quick Start

### 1. Jenkins Setup

1. Create a new Jenkins Pipeline job
2. Configure the pipeline to use the provided `Jenkinsfile`
3. Ensure Jenkins has the following files in `/home/ell/`:
   - `Dockerfile`
   - `docker-compose.yml`

### 2. Local Development

To run the application locally without Jenkins:

```bash
# Clone the repository
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic

# Copy the configuration files
cp /path/to/your/Dockerfile ./
cp /path/to/your/docker-compose.yml ./

# Start the application
docker-compose up -d

# Verify the deployment
curl http://localhost:9090
```

### 3. Access the Application

Once deployed, you can access:
- **Pet Clinic Application**: http://localhost:9090
- **MySQL Database**: localhost:3306 (credentials: petclinic/petclinic)

## 🔧 Configuration Files

### Dockerfile
The Dockerfile uses a multi-stage build approach:
- **Stage 1**: Maven build environment with dependency caching
- **Stage 2**: Lightweight OpenJDK runtime with the compiled JAR

Key features:
- Dependency layer caching for faster builds
- Java 17 runtime
- Application runs on port 9090

### Docker Compose
The `docker-compose.yml` orchestrates:
- **MySQL 8.0** database with persistent storage
- **Spring Boot application** with MySQL profile
- Health checks for reliable service startup
- Volume persistence for database data

### Jenkins Pipeline Deep Dive

The Jenkins pipeline is defined as a declarative pipeline with the following structure:

#### Pipeline Configuration
```groovy
pipeline {
    agent any  // Can run on any available Jenkins agent
    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'  // Environment variable for compose file
    }
}
```

#### Stage-by-Stage Breakdown

**1. Cleanup Stage**
```groovy
stage('Cleanup') {
    steps {
        script {
            sh 'docker-compose down || true'
            sh 'docker system prune -f || true'
        }
    }
}
```
- **Purpose**: Ensures a clean environment before deployment
- **Actions**: 
  - Stops and removes any existing containers from previous runs
  - Prunes unused Docker resources (images, networks, volumes)
- **Error Handling**: Uses `|| true` to prevent pipeline failure if no containers exist

**2. Clone Repository Stage**
```groovy
stage('Clone Repository') {
    steps {
        git branch: 'main', url: 'https://github.com/spring-projects/spring-petclinic.git'
    }
}
```
- **Purpose**: Fetches the latest source code
- **Actions**: Clones the official Spring Pet Clinic repository from GitHub
- **Branch**: Specifically pulls from the `main` branch
- **Workspace**: Code is cloned into Jenkins workspace directory

**3. Copy Local Configs Stage**
```groovy
stage('Copy Local Configs') {
    steps {
        script {
            sh "cp /home/ell/Dockerfile ./Dockerfile"
            sh "cp /home/ell/docker-compose.yml ./docker-compose.yml"
        }
    }
}
```
- **Purpose**: Replaces default configs with custom ones
- **Actions**: 
  - Copies custom Dockerfile from Jenkins server to workspace
  - Copies custom docker-compose.yml to workspace
- **Why Needed**: The original Pet Clinic may not have Docker configs or may have different requirements

**4. Build Docker Image Stage**
```groovy
stage('Build Docker Image') {
    steps {
        script {
            sh 'docker build -t petclinic-app .'
        }
    }
}
```
- **Purpose**: Creates the application Docker image
- **Actions**: Builds Docker image using the copied Dockerfile
- **Image Tag**: Names the image `petclinic-app`
- **Build Context**: Uses current directory (`.`) as build context

**5. Run Docker Compose Stage**
```groovy
stage('Run Docker Compose') {
    steps {
        script {
            sh 'docker-compose up -d'
        }
    }
}
```
- **Purpose**: Starts the multi-container application
- **Actions**: 
  - Starts MySQL database container
  - Starts Spring Boot application container
- **Detached Mode**: `-d` flag runs containers in background
- **Dependencies**: Docker Compose handles service dependencies automatically

**6. Verify Deployment Stage**
```groovy
stage('Verify Deployment') {
    steps {
        script {
            sleep(30) // Wait for services to start
            sh 'curl -f http://localhost:9090 || exit 1'
        }
    }
}
```
- **Purpose**: Confirms successful deployment
- **Actions**: 
  - Waits 30 seconds for services to fully initialize
  - Makes HTTP request to application endpoint
  - Fails pipeline if application doesn't respond
- **Health Check**: Uses `curl -f` which fails on HTTP error codes

#### Post-Pipeline Actions
```groovy
post {
    always {
        echo 'Pipeline completed'
    }
    failure {
        sh 'docker-compose logs || true'
        sh 'docker-compose down || true'
    }
}
```

**Always Block**: Executes regardless of pipeline success/failure
- Simple completion notification

**Failure Block**: Executes only if pipeline fails
- **Debugging**: Outputs container logs to help diagnose issues
- **Cleanup**: Stops containers to prevent resource consumption
- **Error Handling**: Uses `|| true` to prevent secondary failures

#### Pipeline Flow Diagram

```
Start Pipeline
     ↓
[Cleanup] → Remove old containers & images
     ↓
[Clone] → Fetch source code from GitHub
     ↓
[Copy Configs] → Replace with custom Docker files
     ↓
[Build] → Create application Docker image
     ↓
[Deploy] → Start containers with docker-compose
     ↓
[Verify] → Health check application endpoint
     ↓
Success/Failure → Cleanup if failed
     ↓
End Pipeline
```

#### Key Pipeline Features

**Idempotency**: Pipeline can be run multiple times safely due to cleanup stage

**Error Resilience**: Strategic use of `|| true` prevents cascading failures

**Separation of Concerns**: Each stage has a single, clear responsibility

**Visibility**: Post-actions provide debugging information on failures

**Automation**: Fully automated from code checkout to deployment verification

## 🛠️ Customization

### Database Configuration
To modify database settings, update the environment variables in `docker-compose.yml`:

```yaml
environment:
  MYSQL_ROOT_PASSWORD: your_password
  MYSQL_DATABASE: your_database
  MYSQL_USER: your_user
  MYSQL_PASSWORD: your_password
```

### Application Configuration
Customize the Spring Boot application by modifying environment variables:

```yaml
environment:
  - SPRING_PROFILES_ACTIVE=mysql
  - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/petclinic
  - SPRING_DATASOURCE_USERNAME=petclinic
  - SPRING_DATASOURCE_PASSWORD=petclinic
```

### Port Configuration
Change the application port by updating both the Dockerfile `EXPOSE` directive and docker-compose port mapping.

## 📊 Monitoring & Troubleshooting

### Useful Commands

```bash
# View application logs
docker-compose logs app

# View database logs
docker-compose logs db

# Check container status
docker-compose ps

# Stop all services
docker-compose down

# Remove volumes (careful - this deletes data!)
docker-compose down -v
```

### Common Issues

**Application won't start:**
- Check if port 9090 is available
- Verify database connectivity
- Review application logs

**Database connection issues:**
- Ensure MySQL container is healthy
- Verify network connectivity between containers
- Check database credentials

**Jenkins pipeline failures:**
- Ensure Docker daemon is running on Jenkins agent
- Verify file permissions for copying configuration files
- Check available disk space

## 🔒 Security Considerations

- Database credentials are currently in plain text - consider using Docker secrets or environment files for production
- The MySQL root password should be changed for production deployments
- Consider implementing proper network segmentation and firewall rules
- Regular security updates for base images are recommended

## 📚 Additional Resources

- [Spring Pet Clinic Official Repository](https://github.com/spring-projects/spring-petclinic)
- [Docker Documentation](https://docs.docker.com/)
- [Jenkins Pipeline Documentation](https://jenkins.io/doc/book/pipeline/)
- [Spring Boot Docker Guide](https://spring.io/guides/gs/spring-boot-docker/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## 📄 License

This project follows the same license as the Spring Pet Clinic project. Please refer to the original repository for license details.
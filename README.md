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

### Jenkinsfile Pipeline Stages

1. **Cleanup**: Removes existing containers and prunes unused Docker resources
2. **Clone Repository**: Fetches the latest code from the Spring Pet Clinic repository
3. **Copy Local Configs**: Copies custom Dockerfile and docker-compose.yml from Jenkins server
4. **Build Docker Image**: Creates the application Docker image
5. **Run Docker Compose**: Starts the multi-container application
6. **Verify Deployment**: Health check to ensure the application is running

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
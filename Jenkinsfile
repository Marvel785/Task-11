pipeline {
    agent any
    
    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
    }
    
    stages {
        stage('Cleanup') {
            steps {
                script {
                    sh 'docker-compose down || true'
                    sh 'docker system prune -f || true'
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/spring-projects/spring-petclinic.git'
            }
        }
        

stage('Copy Local Configs') {
    steps {
        script {
            // Copy your custom files from /home/ell
            sh "cp /home/ell/Dockerfile ./Dockerfile"
            sh "cp /home/ell/docker-compose.yml ./docker-compose.yml"
        }
    }
}

        
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t petclinic-app .'
                }
            }
        }
        
        stage('Run Docker Compose') {
            steps {
                script {
                    sh 'docker-compose up -d'
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    sleep(30) // Wait for services to start
                    sh 'curl -f http://localhost:9090 || exit 1'
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed'
        }
        failure {
            sh 'docker-compose logs || true'
            sh 'docker-compose down || true'
        }
    }
}

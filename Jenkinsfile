pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = "158.160.145.70:8123"  
        REPO_NAME = "mydockerrepo"
        IMAGE_NAME_BUILD = "geoserver-build"
        IMAGE_NAME_PROD = "geoserver-prod"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/Regina117/dc19.git', branch: 'main'
            }
        }
        stage('Build Docker Image (Build)') {
            steps {
                script {
                    sh "docker build -f Dockerfile.build -t ${DOCKER_REGISTRY}/${REPO_NAME}/${IMAGE_NAME_BUILD}:latest ."
                }
            }
        }
        stage('Build Docker Image (Prod)') {
            steps {
                script {
                    sh "docker build -f Dockerfile.prod -t ${DOCKER_REGISTRY}/${REPO_NAME}/${IMAGE_NAME_PROD}:latest ."
                }
            }
        }
        stage('Push Images to Nexus') {
            steps {
                script {                   
                    echo "Dm59JTErVdXaKaN" | docker login ${DOCKER_REGISTRY} -u admin --password-stdin || { echo "Docker login failed"; exit 1; }
                    sh "docker push ${DOCKER_REGISTRY}/${REPO_NAME}/${IMAGE_NAME_BUILD}:latest"
                    sh "docker push ${DOCKER_REGISTRY}/${REPO_NAME}/${IMAGE_NAME_PROD}:latest"
                }
            }
        }
    }
}


    

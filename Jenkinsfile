pipeline {
    agent any
    environment {
        DOCKER_REGISTRY = "158.160.145.70:8123"  
        REPO_NAME = "mydockerrepo"
        IMAGE_NAME_BUILD = "geoserver-build"
        IMAGE_NAME_PROD = "geoserver-prod"
    }
    stages {
        stage('Code checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Regina117/dc19.git']])
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
                    echo "1185" | docker login ${DOCKER_REGISTRY} -u root --password-stdin || { echo "Docker login failed"; exit 1; }
                    sh "docker push ${DOCKER_REGISTRY}/${REPO_NAME}/${IMAGE_NAME_BUILD}:latest"
                    sh "docker push ${DOCKER_REGISTRY}/${REPO_NAME}/${IMAGE_NAME_PROD}:latest"
                }
            }
        }
        stage('Upload Artifact to Nexus') {
            steps {
                script {
                    nexusArtifactUploader artifacts: [[
                        artifactId: 'geoserver', 
                        classifier: '', 
                        file: 'target/geoserver.war', 
                        type: 'war'
                    ]], 
                    credentialsId: 'server-credentials', 
                    groupId: 'org.geoserver', 
                    nexusUrl: 'http://158.160.145.70:8123/repository/mydockerrepo/', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'mydockerrepo', 
                    version: '2.27-SNAPSHOT'
                }
            }
        }
    }
}



    

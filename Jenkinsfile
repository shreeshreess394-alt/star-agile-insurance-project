pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk-17'           // Optional: If JDK is configured in Jenkins tools
    }

    environment {
        DOCKERHUB_USER = "YOUR_DOCKERHUB_USERNAME"   // <-- Change this
        IMAGE_NAME = "insure-me"
        TAG = "1.0"
    }

    stages {

        stage('Prepare Environment') {
            steps {
                echo "Initializing build environment..."
                script {
                    mavenHome = tool 'maven'
                    mavenCMD = "${mavenHome}/bin/mvn"
                    dockerHome = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
                    dockerCMD = "${dockerHome}/bin/docker"
                }
            }
        }

        stage('Git Checkout') {
            steps {
                echo "Cloning the repository..."
                git credentialsId: 'github-token', url: 'https://github.com/Shriraksha384/star-agile-insurance-project.git', branch: 'master'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "Running Maven Build..."
                sh "${mavenCMD} clean package"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker Image..."
                sh "${dockerCMD} build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${TAG} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo "Pushing Docker image to DockerHub..."
                withCredentials([string(credentialsId: 'docker-password', variable: 'DOCKER_PASS')]) {
                    sh "${dockerCMD} login -u ${DOCKERHUB_USER} -p ${DOCKER_PASS}"
                    sh "${dockerCMD} push ${DOCKERHUB_USER}/${IMAGE_NAME}:${TAG}"
                }
            }
        }

        // Optional Stage: Uncomment after Ansible setup
        /*
        stage('Deploy using Ansible') {
            steps {
                echo "Deploying Application on Test Server..."
                ansiblePlaybook become: true, credentialsId: 'ansible-key', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
            }
        }
        */

    }

    // Optional: Add notifications (after email setup)
    post {
        success {
            echo "Build and Deployment Successful!"
        }
        failure {
            echo "Build Failed!"
        }
    }
}

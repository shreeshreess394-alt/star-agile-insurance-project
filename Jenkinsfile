pipeline {
    agent any

    tools {
        maven 'maven'   // Jenkins tool name for Maven
        // jdk 'jdk-17' // (Optional) Only enable if configured under Global Tools
    }

    environment {
        DOCKERHUB_USER = "shriraksha384"   // ✅ Your real DockerHub username
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

        stage('Maven Build') {
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

        stage('Push Docker Image to DockerHub') {
            steps {
                echo "Pushing Docker Image to DockerHub..."
                withCredentials([usernamePassword(credentialsId: 'docker-password', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo ${DOCKER_PASS} | ${dockerCMD} login -u ${DOCKER_USER} --password-stdin"
                    sh "${dockerCMD} push ${DOCKER_USER}/${IMAGE_NAME}:${TAG}"
                }
            }
        }

        // ✅ Uncomment this after Ansible setup is ready
        /*
        stage('Deploy using Ansible') {
            steps {
                echo "Deploying Application on Test Server..."
                ansiblePlaybook become: true, credentialsId: 'ansible-key', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
            }
        }
        */
    }

    post {
        success {
            echo "✅ Build, Docker Image Creation & Push Completed Successfully!"
        }
        failure {
            echo "❌ Build Failed — Check Jenkins Console Output."
        }
    }
}

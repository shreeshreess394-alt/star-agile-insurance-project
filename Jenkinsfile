node {
    
    def mavenHome
    def mavenCMD
    def dockerTool
    def dockerCMD
    def tagName
    
    stage('prepare environment') {
        echo 'Initialize all the variables'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        dockerTool = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${dockerTool}/bin/docker"
        tagName = "3.0"
    }
    
    stage('Git Code Checkout') {
        try {
            echo 'Checkout code from Git repository'
            git credentialsId: 'github-token', url: 'https://github.com/Shriraksha384/star-agile-insurance-project.git', branch: 'master'
        } catch (Exception e) {
            echo 'Exception occurred in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
        }
    }
    
    stage('Build the Application') {
        echo "Cleaning... Compiling... Testing... Packaging..."
        sh "${mavenCMD} clean package"
    }
    
    stage('Publish Test Reports') {
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, 
            reportDir: 'target/surefire-reports', 
            reportFiles: 'index.html', 
            reportName: 'HTML Report'])
    }
    
    stage('Containerize the Application') {
        echo 'Creating Docker image'
        sh "${dockerCMD} build -t shriraksha384/insure-me:${tagName} ."
    }
    
    stage('Push Docker Image to DockerHub') {
        echo 'Pushing the Docker image to DockerHub'
        withCredentials([usernamePassword(credentialsId: 'docker-password', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh "${dockerCMD} login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
            sh "${dockerCMD} push ${DOCKER_USER}/insure-me:${tagName}"
        }
    }

    // Uncomment this after Ansible is installed and configured
    /*
    stage('Deploy to Test Server') {
        echo 'Deploying using Ansible...'
        ansiblePlaybook become: true, credentialsId: 'ansible-key', disableHostKeyChecking: true, 
            installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
    }
    */
}

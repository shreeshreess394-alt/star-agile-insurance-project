node {
    def mavenHome
    def mavenCMD
    def dockerTool
    def dockerCMD
    def tagName

    stage('Prepare Environment') {
        echo 'Initializing all the variables'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        dockerTool = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${dockerTool}/bin/docker"
        tagName = "3.0"
    }

    stage('Git Code Checkout') {
        try {
            echo 'Checking out code from Git repository'
            git credentialsId: 'github-token', url: 'https://github.com/Shriraksha384/star-agile-insurance-project.git', branch: 'master'
        } catch (Exception e) {
            echo '❌ Git checkout failed!'
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
            reportName: 'Test Report'])
    }

    stage('Containerize the Application') {
        echo '✅ Creating Docker image...'
        sh "${dockerCMD} build -t shriraksha384/insure-me:${tagName} ."
    }

   stage('Push Docker Image to DockerHub') {
    echo "🚀 Pushing Docker image to DockerHub..."
    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
        sh """
            echo ${DOCKER_PASS} | ${dockerCMD} login -u ${DOCKER_USER} --password-stdin
            ${dockerCMD} push ${DOCKER_USER}/insure-me:${tagName}
        """
    }
}


    // Optional - Uncomment when Ansible is configured properly
    /*
    stage('Configure & Deploy to Test Server') {
        echo 'Deploying application to test server using Ansible...'
        ansiblePlaybook become: true, credentialsId: 'ansible-key', disableHostKeyChecking: true,
            installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
    }
    */
}

String user = parameters.ec2.server.serverParameters.username;
String privateIP = parameters.ec2.server.serverParameters.ip;
String tag = BUILD_TIMESTAMP.split(" ").join("").split(":").join("").split("IST")[0].split("-").join("");
String sshAgentKeyName = "test-ec2"
node ('master'){
    stage('Git SCM Checkout') {
        git branch: 'devops', changelog: false, credentialsId: 'service-account', poll: false, url: 'http://github.com/app.git'
    }
    stage('DockerRegistry login') {
        withDockerRegistry(credentialsId: 'DockerHub', url: 'https://index.docker.io/v1/') {}
    }
    stage("Docker build and push") {
        sh """
        docker build . -t ec2
        docker tag ec2 ec2/nodejs:${tag}
        docker tag ec2 ec2/nodejs:latest
        docker push ec2/nodejs:${tag}
        docker push ec2/nodejs:latest
        """
    }
    sshagent(["${sshAgentKeyName}"]) {
        stage ('commands in SSH Agent ') {
            withDockerRegistry(credentialsId: 'DockerHub', url: 'https://index.docker.io/v1/') {}
            stage("Docker images deploying") {
                sh """
                echo '#!/bin/bash
                set -e
                docker pull ec2/nodejs:${tag}
                docker stop ec2
                docker rm ec2
                docker run -itd --name ec2 -p 3000:3000 ec2/nodejs 
                ' > deploy
                chmod +x deploy
                scp -o 'StrictHostKeyChecking no' -r deploy ${user}@${privateIP}:/tmp
                ssh -o 'StrictHostKeyChecking no' -t ${user}@${privateIP} /tmp/deploy
                ssh -o 'StrictHostKeyChecking no' -t ${user}@${privateIP} rm -rf /tmp/deploy
                """
            }
        }
    }
}

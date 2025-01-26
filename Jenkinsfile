node {
    def remote = [:]
    remote.name = 'EC2 Deployment'
    remote.host = env.DICODING_SUBMISSION_EC2_IP
    remote.allowAnyHosts = true

    docker.image('maven:3.9.9-eclipse-temurin-23-alpine').inside('--user root') {
        stage('Build') {
            try {
                checkout scm
                sh 'ls -la'
                sh 'mvn -X -B -DskipTests clean package'
            } catch (Exception e) {
                echo "Build failed: ${e.message}"
                error 'Stopping pipeline due to build failure.'
            }
        }

        stage('Test') {
            try {
                sh 'mvn test'
            } catch (Exception e) {
                echo "Tests failed: ${e.message}"
                error 'Tests encountered failures.'
            } finally {
                junit 'target/surefire-reports/*.xml'
            }
        }

        stage('Manual Approval') {
            input message: 'Lanjutkan ke tahap Deploy?',
                ok: 'Proceed'
        }
    }

    docker.image('docker:latest').inside('--privileged -v /var/run/docker.sock:/var/run/docker.sock --user root') {
        stage('Deploy') {
            docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
                docker.build('dedyirama/simple-java-maven').push('latest')
            }
            withCredentials([sshUserPrivateKey(credentialsId: 'dicoding-submission-ssh', keyFileVariable: 'identity', usernameVariable: 'userName')]) {
                remote.user = userName
                remote.identityFile = identity

                try {
                    sshCommand remote: remote, command: '''
                            docker pull dedyirama/simple-java-maven:latest
                            docker stop simple-java-maven || true
                            docker rm simple-java-maven || true
                            docker run -d --name simple-java-maven --network host --memory=512m --cpu-shares=512 dedyirama/simple-java-maven:latest
                        '''

                    sleep(time: 1, unit: 'MINUTES')
                } catch (Exception e) {
                    echo "Deployment failed: ${e.message}"
                    error 'Stopping pipeline due to deployment failure.'
                }
            }
        }
    }
}

node {
    def remote = [:] // Konfigurasi remote SSH sebagai objek global
    remote.name = 'EC2 Deployment'
    remote.host = '54.169.224.31'
    remote.allowAnyHosts = true

    def ec2TargetDir = '/home/ec2-user/'

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

        stage('Deploy') {
            withCredentials([sshUserPrivateKey(credentialsId: 'dicoding-submission-ssh', keyFileVariable: 'identity', usernameVariable: 'userName')]) {
                remote.user = userName
                remote.identityFile = identity

                try {
                    def name = sh(script: 'mvn -q -DforceStdout help:evaluate -Dexpression=project.name', returnStdout: true).trim()
                    def version = sh(script: 'mvn -q -DforceStdout help:evaluate -Dexpression=project.version', returnStdout: true).trim()

                    echo "Deploying application ${name}-${version}.jar to EC2"

                    // Transfer file JAR to EC2
                    sshPut remote: remote, from: "target/${name}-${version}.jar", into: ec2TargetDir

                    // Run the application on EC2
                    sshCommand remote: remote, command: """
                    java -jar ${ec2TargetDir}${name}-${version}.jar
                    """

                    echo 'Application deployed and executed successfully on EC2'

                    sleep(time: 1, unit: 'MINUTES')
                } catch (Exception e) {
                    echo "Deployment failed: ${e.message}"
                    error 'Stopping pipeline due to deployment failure.'
                }
            }
        }
    }
}

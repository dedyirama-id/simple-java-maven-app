node {
    docker.image('maven:3.9.9-eclipse-temurin-23-alpine').inside('--user root') {
        stage('Checkout') {
            checkout scm
            sh 'ls -la'
        }

        stage('Build') {
            try {
                sh 'mvn -X -B -DskipTests clean package'
            } catch (e) {
                echo "Build failed: ${e.message()}"
                error 'Stopping pipeline due to build failure.'
            }
        }

        stage('Test') {
            try {
                sh 'mvn test'
            } catch (e) {
                echo "Tests failed: ${e.message()}"
                error 'Tests encountered failures.'
            } finally {
                junit 'target/surefire-reports/*.xml'
            }
        }
    }
}

node {
    docker.image('maven:3.9.9-eclipse-temurin-23-alpine').inside('--user root') {

        stage('Build') {
            try {
                sh 'mvn -X -B -DskipTests clean package'
            } catch (Exception e) {
                echo "Build failed: ${e.getMessage()}"
                error "Stopping pipeline due to build failure."
            }
        }

        stage('Test') {
            try {
                sh 'mvn test'
            } catch (Exception e) {
                echo "Tests failed: ${e.getMessage()}"
                error "Tests encountered failures."
            } finally {
                junit 'target/surefire-reports/*.xml'
            }
        }
    }
}

node {
    docker.image('maven:3.9.9-eclipse-temurin-23-alpine').inside('--user root') {

        stage('Build') {
            try {
                sh 'mvn -B -DskipTests clean package'
            } catch (Exception e) {
                echo "Build failed: ${e.getMessage()}"
                error "Stopping pipeline due to build failure."
            }
        }
    }
}

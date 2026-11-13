pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Archive Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar',
                                 fingerprint: true
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-hw8',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASS'
                    )
                ]) {
                    sh '''
                        set +x

                        JAR_FILE=$(ls target/*.jar | grep -v original | head -n 1)
			
			echo "Uploading $JAR_FILE to Nexus..."

                        curl -v -f -u "$NEXUS_USER:$NEXUS_PASS" \
                            -X POST \
                            "http://nexus-hw8:8081/service/rest/v1/components?repository=maven-snapshots" \
                            -F "maven2.generate-pom=false" \
                            -F "maven2.asset1=@pom.xml" \
                            -F "maven2.asset1.extension=pom" \
                            -F "maven2.asset2=@${JAR_FILE};type=application/java-archive" \
                            -F "maven2.asset2.extension=jar"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'JAR built and uploaded to Nexus successfully!'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}

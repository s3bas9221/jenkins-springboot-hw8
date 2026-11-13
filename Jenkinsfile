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

                        mkdir -p ~/.m2

                        cat > ~/.m2/settings.xml <<EOF
<settings>
    <servers>
        <server>
            <id>nexus</id>
            <username>${NEXUS_USER}</username>
            <password>${NEXUS_PASS}</password>
        </server>
    </servers>
</settings>
EOF

                        mvn deploy:deploy-file \
                            -DgroupId=com.example \
                            -DartifactId=spring-boot-complete \
                            -Dversion=0.0.1-SNAPSHOT \
                            -Dpackaging=jar \
                            -Dfile=target/spring-boot-complete-0.0.1-SNAPSHOT.jar \
                            -DpomFile=pom.xml \
                            -DrepositoryId=nexus \
                            -Durl=http://nexus-hw8:8081/repository/maven-snapshots/
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
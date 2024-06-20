pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'node-docker-app' // Nom de votre image Docker
        DOCKER_TAG = "latest" // Tag de votre image Docker
        NEXUS_URL = 'http://localhost:8081' // URL de votre Nexus
        NEXUS_REPO = 'repojenkins' // Nom de votre repository Nexus
        NEXUS_CREDENTIALS_ID = 'nexus' // ID des credentials Nexus dans Jenkins
    }

    stages {
        stage('Build') {
            steps {
                script {
                    def gradleHome = tool name: 'Gradle', type: 'Gradle'
                    sh "${gradleHome}/bin/gradle jib"
                }
            }
        }

        stage('Push to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIALS_ID}", usernameVariable: 'NEXUS_USERNAME', passwordVariable: 'NEXUS_PASSWORD')]) {
                    script {
                        sh '''
                            echo $NEXUS_PASSWORD | docker login -u $NEXUS_USERNAME --password-stdin ${NEXUS_URL}
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${NEXUS_URL}/${NEXUS_REPO}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${NEXUS_URL}/${NEXUS_REPO}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
                sh "docker rmi ${NEXUS_URL}/${NEXUS_REPO}/${DOCKER_IMAGE}:${DOCKER_TAG} || true"
            }
        }
    }
}
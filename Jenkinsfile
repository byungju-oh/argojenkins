pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        GITHUB_CREDENTIALS = credentials('github-credentials')
        IMAGE_NAME = 'qudwndh/jente'
        DOCKER_REPO = 'https://github.com/byungju-oh/shop.git'
        MANIFEST_REPO = 'https://github.com/byungju-oh/argojenkins.git'
        GIT_BRANCH = 'main'
        MANIFEST_FILE = 'was/dep.yaml'
    }

    stages {
        stage('Clone Docker Repository') {
            steps {
                git url: "${DOCKER_REPO}", branch: "${GIT_BRANCH}", credentialsId: 'github-credentials'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${env.BUILD_ID}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        docker.image("${IMAGE_NAME}:${env.BUILD_ID}").push()
                    }
                }
            }
        }

        stage('Clone Manifest Repository') {
            steps {
                dir('manifests') {
                    git url: "${MANIFEST_REPO}", branch: "${GIT_BRANCH}", credentialsId: 'github-credentials'
                }
            }
        }

        stage('Update Manifests') {
            steps {
                dir('manifests') {
                    script {
                        sh """
                            sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${env.BUILD_ID}|' ${MANIFEST_FILE}
                        """
                        sh """
                            git config user.email "jenkins@yourdomain.com"
                            git config user.name "Jenkins"
                            git add ${MANIFEST_FILE}
                            git commit -m "Update image version to ${env.BUILD_ID}"
                            git push https://${GITHUB_CREDENTIALS_USR}:${GITHUB_CREDENTIALS_PSW}@${MANIFEST_REPO.replace('https://', '')} ${GIT_BRANCH}
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

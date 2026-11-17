pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'mubashir594/maker'

        SONAR_SERVER_NAME     = 'sonarqube_env'
        SONAR_SCANNER_TOOL    = 'sonar-scanner'

        GIT_CREDENTIALS_ID    = 'github-cred'
        MANIFEST_REPO         = 'https://github.com/Mubashir594/maker_project.git'
        MANIFEST_BRANCH       = 'main'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"

                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=maker-key \
                            -Dsonar.projectName=maker-project \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry(
                        'https://index.docker.io/v1/',
                        "${DOCKER_CREDENTIALS_ID}"
                    ) {
                        def appImage = docker.build(
                            "${DOCKER_IMAGE}:${env.BUILD_NUMBER}",
                            '.'
                        )

                        appImage.push()
                        appImage.push('latest')
                    }
                }
            }
        }

        stage('Update Manifest Repo') {
            steps {
                dir('manifest-workspace') {

                    git branch: "${MANIFEST_BRANCH}",
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        url: "${MANIFEST_REPO}"

                    sh """
                        sed -i '/name: maker/{n;s|image:.*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|}' k8s/deployment.yaml

                        git config user.name 'Mubashir594'
                        git config user.email 'mubumubashir594@gmail.com'

                        git add k8s/deployment.yaml

                        git commit -m "chore(manifest): update maker image tag to ${env.BUILD_NUMBER} [skip ci]" || echo "No manifest changes to commit"

                        git push origin ${MANIFEST_BRANCH}
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
            echo "Docker image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
            echo 'Manifest repository updated successfully!'
        }

        failure {
            echo 'Pipeline failed. Please check the Console Output.'
        }
    }
}

pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar'
        NODEJS_HOME = tool 'nodejs'
        DOCKER_IMAGE_NAME = 'backendnodejs'
        TRIVY_IMAGE = 'aquasec/trivy:latest'
        TRIVY_REPORT_FILE = 'trivy-report.json'
    }

    stages {

        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/roshan-etc/backend-node.js.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        /*
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonar') {
                        sh '''
                            ${SCANNER_HOME}/bin/sonar-scanner \
                              -Dsonar.projectKey=backend \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=http://35.90.114.54:9000 \
                              -Dsonar.token=sqp_f27f78bfe29a6c3eab71dde3ec5eaf8f9dcdcacd
                        '''
                    }
                }
            }
        }
        */

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE_NAME} .'
            }
        }

        stage('Run Trivy Scan') {
            steps {
                script {
                    sh '''
                        docker run --rm \
                          -v /var/run/docker.sock:/var/run/docker.sock \
                          -v $(pwd):/root \
                          ${TRIVY_IMAGE} image --format json --output ${TRIVY_REPORT_FILE} ${DOCKER_IMAGE_NAME}
                    '''
                }
            }
        }

        stage('Archive Trivy Report') {
            steps {
                archiveArtifacts artifacts: "${TRIVY_REPORT_FILE}", allowEmptyArchive: true
            }
        }

        stage('Display Trivy Report') {
            steps {
                script {
                    sh "cat ${TRIVY_REPORT_FILE}"
                }
            }
        }

        stage('Docker Login') {
            steps {
                script {
                    sh '''
                        aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 654654245097.dkr.ecr.us-west-2.amazonaws.com
                    '''
                }
            }
        }

        stage('Docker Tag & Push') {
            steps {
                sh 'docker tag ${DOCKER_IMAGE_NAME}:latest 654654245097.dkr.ecr.us-west-2.amazonaws.com/${DOCKER_IMAGE_NAME}:latest'
                sh 'docker push 654654245097.dkr.ecr.us-west-2.amazonaws.com/${DOCKER_IMAGE_NAME}:latest'
            }
        }

    }
    
    post {
        always {
            script {
                sh '''
                    # Remove the Docker image
                    docker rmi ${DOCKER_IMAGE_NAME}
                '''
            }
        }
    }
}

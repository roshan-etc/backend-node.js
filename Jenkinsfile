pipeline {
    agent any

    environment {
        // Define the SonarQube Scanner tool
        SCANNER_HOME = tool 'sonar'
        // Reference the Node.js installation directly by name
        NODEJS_HOME = tool 'nodejs'
    }

    stages {

        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('GitCheckout') {
            steps {
                git branch: 'main', url: 'https://github.com/roshan-etc/backend-node.js.git'
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') { // Replace 'sonarqube' with the name you configured in Jenkins for SonarQube
                        sh '''
                            sonar-scanner \
                              -Dsonar.projectKey=backend \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=http://35.90.114.54:9000 \
                              -Dsonar.token=sqp_07fa25d48e7c11d4ad67e85b2e0e3602e632bd90
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    // Wait for SonarQube Quality Gate status
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qualityGate.status}"
                    }
                }
            }
        }

        stage('DockerBuild') {
            steps {
                sh 'docker build -t backendnodejs .'
            }
        }

        stage('DockerLogin') {
            steps {
                sh 'aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 654654245097.dkr.ecr.us-west-2.amazonaws.com'
            }
        }

        stage('DockerTAG & Push') {
            steps {
                sh 'docker tag backendnodejs:latest 654654245097.dkr.ecr.us-west-2.amazonaws.com/backendnodejs:latest'
                sh 'docker push 654654245097.dkr.ecr.us-west-2.amazonaws.com/backendnodejs:latest'
            }
        }

    }
    
    post {
        always {
            cleanWs() // Clean workspace after pipeline runs
        }
    }
}

pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOCKERHUB_USER = "lakshvar96"
        FRONTEND_IMAGE = "redbus-frontend"
        BACKEND_IMAGE  = "redbus-backend"

        GIT_REPO = "https://github.com/Lakshmanan1996/redbus.git"
    }

    stages {

        /* ===================== CHECKOUT ===================== */
        stage('Checkout Code') {
            
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: 'master']],
                    userRemoteConfigs: [[url: "${GIT_REPO}"]]
                ])
            }
        }

        /* ===================== STASH SOURCE ===================== */
        stage('Stash Source') {
            
            steps {
                stash includes: '**/*', name: 'source-code'
            }
        }

        /* ===================== SONARQUBE ===================== */
        stage('SonarQube Analysis') {
            
            steps {
                unstash 'source-code'

                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('sonarqube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=redbus \
                          -Dsonar.projectName=redbus \
                          -Dsonar.sources=back-end-redbus,front-end-redbus \
                          -Dsonar.exclusions=**/node_modules/**
                        """
                    }
                }
            }
        }

        /*===================== QUALITY GATE ===================== */
        
        stage('Quality Gate') {
            
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
      

        /* ===================== DOCKER BUILD ===================== */
        stage('Docker Build') {
            
            steps {
                unstash 'source-code'

                sh """
                

                # Backend Build
                docker build -t ${DOCKERHUB_USER}/${BACKEND_IMAGE}:${BUILD_NUMBER} -f back-end-redbus/Dockerfile back-end-redbus/
                docker tag ${DOCKERHUB_USER}/${BACKEND_IMAGE}:${BUILD_NUMBER} ${DOCKERHUB_USER}/${BACKEND_IMAGE}:latest

                # Frontend Build
                docker build -t ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:${BUILD_NUMBER} -f front-end-redbus/Dockerfile front-end-redbus/
                docker tag ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:${BUILD_NUMBER} ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:latest


                
                """
            }
        }

        /* ===================== TRIVY SCAN ===================== */
        stage('Trivy Scan') {
            
            steps {
                sh """
                 trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                 trivy image --exit-code 0 --severity HIGH,CRITICAL ${DOCKERHUB_USER}/${BACKEND_IMAGE}:${BUILD_NUMBER} 
                """
            }
        }

        /* ===================== PUSH TO DOCKER HUB ===================== */
        stage('Push Image') {
            
            steps {
                unstash 'source-code'

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }

                sh """
                

                docker push ${DOCKERHUB_USER}/${BACKEND_IMAGE}:${BUILD_NUMBER} 
                docker push ${DOCKERHUB_USER}/${BACKEND_IMAGE}:latest

                docker push ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                docker push ${DOCKERHUB_USER}/${FRONTEND_IMAGE}:latest

                
                """
            }
        }
    }

    post {
        success {
            echo "✅ redbus CI Pipeline SUCCESS"
        }
        failure {
            echo "❌ redbus CI Pipeline FAILED"
        }
    }
}

pipeline {
    agent any

    options {
        timeout(time: 60, unit: 'MINUTES')
        timestamps()
    }

    environment {
        DOCKER_IMAGE = "arunkumarravi08/accounts-dashboard"
        IMAGE_TAG = "${BUILD_NUMBER}"
        SONARQUBE_ENV = "sonarqube-server"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Arun-KumarRavi/git-new.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Frontend install
                dir('client') {
                    sh 'npm ci'
                }
                // Backend install
                dir('flask-integration') {
                    sh '''
                    python3 -m venv venv
                    venv/bin/pip install --upgrade pip
                    venv/bin/pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Code Quality & Tests') {
            parallel {

                stage('ESLint') {
                    steps {
                        dir('client') {
                            sh 'npm run lint'
                        }
                    }
                }

                stage('Frontend Tests') {
                    steps {
                        dir('client') {
                            sh 'npm test -- --watchAll=false'
                        }
                    }
                }

                stage('Backend Tests') {
                    steps {
                        dir('flask-integration') {
                            sh 'venv/bin/pytest'
                        }
                    }
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=accounts-dashboard \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=$SONAR_HOST_URL \
                          -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL . || true'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$IMAGE_TAG .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL $DOCKER_IMAGE:$IMAGE_TAG || true'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh '''
                docker push $DOCKER_IMAGE:$IMAGE_TAG
                docker tag $DOCKER_IMAGE:$IMAGE_TAG $DOCKER_IMAGE:latest
                docker push $DOCKER_IMAGE:latest
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
        always {
    echo "Skipping cleanup for debugging"
        }
    }           
}



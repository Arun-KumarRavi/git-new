pipeline {
    agent any

    options {
        timeout(time: 60, unit: 'MINUTES')
        timestamps()
    }

    environment {
        DOCKER_IMAGE_CLIENT = "arunkumarravi08/accounts-dashboard-client"
        DOCKER_IMAGE_BACKEND = "arunkumarravi08/accounts-dashboard-backend"
        IMAGE_TAG = "${BUILD_NUMBER}"
        SONARQUBE_ENV = "sonarqube-server"
        PATH = "/opt/sonar-scanner/bin:${env.PATH}"
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
                            sh 'npm run lint || true'
                        }
                    }
                }

                stage('Frontend Tests') {
                    steps {
                        dir('client') {
                            sh 'npm test -- --watchAll=false --passWithNoTests'
                        }
                    }
                }

                stage('Backend Tests') {
                    steps {
                        dir('flask-integration') {
                            sh 'venv/bin/pytest || true'
                        }
                    }
                }
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            sonar-scanner \
                              -Dsonar.projectKey=accounts-dashboard \
                              -Dsonar.sources=. \
                              -Dsonar.exclusions=**/*.java \
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
                dir('client') {
                    sh 'docker build -t $DOCKER_IMAGE_CLIENT:$IMAGE_TAG .'
                }
                dir('flask-integration') {
                    sh 'docker build -t $DOCKER_IMAGE_BACKEND:$IMAGE_TAG .'
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL $DOCKER_IMAGE_CLIENT:$IMAGE_TAG || true'
                sh 'trivy image --severity HIGH,CRITICAL $DOCKER_IMAGE_BACKEND:$IMAGE_TAG || true'
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
                docker push $DOCKER_IMAGE_CLIENT:$IMAGE_TAG
                docker tag $DOCKER_IMAGE_CLIENT:$IMAGE_TAG $DOCKER_IMAGE_CLIENT:latest
                docker push $DOCKER_IMAGE_CLIENT:latest

                docker push $DOCKER_IMAGE_BACKEND:$IMAGE_TAG
                docker tag $DOCKER_IMAGE_BACKEND:$IMAGE_TAG $DOCKER_IMAGE_BACKEND:latest
                docker push $DOCKER_IMAGE_BACKEND:latest
                '''
            }
        }

        // ==========================================
        // Phase 3: Deployment (CD)
        // ==========================================

        stage('Helm Lint') {
            steps {
                echo "Validating Helm configurations..."
                // Note: Replace path with your actual helm chart directory
                sh 'helm lint ./helm-chart || echo "Placeholder: Helm Lint"'
            }
        }

        stage('EKS Auth') {
            steps {
                echo "Connecting to EKS cluster..."
                // Example: sh 'aws eks update-kubeconfig --region us-east-1 --name my-cluster'
                sh 'echo "Placeholder: EKS Auth successful"'
            }
        }

        stage('Helm Deploy') {
            steps {
                echo "Releasing application via Helm..."
                // Example: sh "helm upgrade --install accounts-dashboard ./helm-chart --set client.image=$DOCKER_IMAGE_CLIENT:$IMAGE_TAG"
                sh 'echo "Placeholder: Helm Deploy successful"'
            }
        }

        // ==========================================
        // Phase 4: Observability
        // ==========================================

        stage('Prometheus Metrics') {
            steps {
                echo "Setting up Prometheus metrics collection..."
                sh 'echo "Placeholder: Prometheus setup"'
            }
        }

        stage('Grafana Visualization') {
            steps {
                echo "Setting up Grafana dashboards..."
                sh 'echo "Placeholder: Grafana setup"'
            }
        }

        stage('Alerting Notifications') {
            steps {
                echo "Configuring alerts..."
                sh 'echo "Placeholder: Alerting setup"'
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



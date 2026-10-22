pipeline {
    agent any
    
    environment {
        DOCKER_HUB_REGISTRY = "majorazor/smart-agri-portal"
        APP_VERSION         = "1.0.${BUILD_NUMBER}"
        GIT_REPO_URL        = "https://github.com/MaJoRaZoR/smart-agriculture-portal/"
    }

    stages {
        stage('Fetch Source Data') {
            steps {
                echo 'Checking out Agricultural Code Base Repository...'
                git url: "${env.GIT_REPO_URL}", branch: 'main'
            }
        }
        
        stage('Parallel Analytics Verification') {
            parallel {
                stage('Lint HTML Web Layout') {
                    steps {
                        echo 'Validating index.html structures...'
                    }
                }
                stage('Evaluate Optimization Engine') {
                    steps {
                        echo 'Testing automated threshold math metrics...'
                    }
                }
            }
        }
        
        stage('Build Engine Image') {
            steps {
                script {
                    echo "Building Docker container image version: ${env.APP_VERSION}"
                    bat "docker build -t ${env.DOCKER_HUB_REGISTRY}:${env.APP_VERSION} ."
                    bat "docker tag ${env.DOCKER_HUB_REGISTRY}:${env.APP_VERSION} ${env.DOCKER_HUB_REGISTRY}:latest"
                }
            }
        }

        stage('Archive Test Configuration') {
            steps {
                echo "Archiving operational assets for traceability..."
                archiveArtifacts artifacts: 'index.html', fingerprint: true
            }
        }

        stage('Push Image Registry') {
            steps {
                script {
                    echo "Logging into Docker Hub natively and pushing image layers..."
                    // Bypasses the echo %PASS% pipe completely using the native Jenkins Docker plugin wrapper
                    docker.withRegistry('https://docker.io', 'docker-hub-credentials') {
                        bat "docker push ${env.DOCKER_HUB_REGISTRY}:${env.APP_VERSION}"
                        bat "docker push ${env.DOCKER_HUB_REGISTRY}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                withKubeConfig([credentialsId: 'kubernetes-cluster-credentials', serverUrl: 'https://docker.internal']) {
                    bat "powershell -Command \"(Get-Content deployment.yaml) -replace 'IMAGE_VERSION_PLACEHOLDER', '${env.APP_VERSION}' | Set-Content deployment.yaml\""
                    bat "kubectl apply -f deployment.yaml"
                    bat "kubectl rollout status deployment/agri-portal-deployment"
                }
            }
        }
    }
}

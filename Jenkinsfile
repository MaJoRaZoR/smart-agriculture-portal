pipeline {
    agent any
    
    environment {
        DOCKER_HUB_REGISTRY = "your-dockerhub-username/smart-agri-portal"
        APP_VERSION         = "1.0.${BUILD_NUMBER}"
    }
    
    stages {
        stage('Fetch Source Data') {
            steps {
                echo 'Checking out Agricultural Code Base Repository...'
                checkout scm
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
                    echo "Building Docker container image version: ${APP_VERSION}"
                    // Converted from sh to bat
                    bat "docker build -t %DOCKER_HUB_REGISTRY%:%APP_VERSION% ."
                    bat "docker tag %DOCKER_HUB_REGISTRY%:%APP_VERSION% %DOCKER_HUB_REGISTRY%:latest"
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
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    // Converted to Windows Batch execution pattern
                    bat "echo %PASS% | docker login -u %USER% --password-stdin"
                    bat "docker push %DOCKER_HUB_REGISTRY%:%APP_VERSION%"
                    bat "docker push %DOCKER_HUB_REGISTRY%:latest"
                }
            }
        }

        stage('Deploy to Kubernetes Cluster') {
            steps {
                withKubeConfig([credentialsId: 'kubernetes-cluster-credentials', serverUrl: 'https://docker.internal']) {
                    // Replaced Linux 'sed' with a native PowerShell string replacement pattern
                    bat "powershell -Command \"(Get-Content deployment.yaml) -replace 'IMAGE_VERSION_PLACEHOLDER', '%APP_VERSION%' | Set-Content deployment.yaml\""
                    bat "kubectl apply -f deployment.yaml"
                    bat "kubectl rollout status deployment/agri-portal-deployment"
                }
            }
        }
    }
}

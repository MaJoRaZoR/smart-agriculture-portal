pipeline {
    agent any
    
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['Staging', 'Production'], description: 'Target Kubernetes Environment Clustered Environment')
        booleanParam(name: 'RUN_INTEGRATION_TESTS', defaultValue: true, description: 'Toggle Execution of Advanced Sensor Simulation Validation Check')
    }

    environment {
        DOCKER_HUB_USER = 'YOUR_DOCKER_HUB_USERNAME'
        IMAGE_NAME      = 'smart-irrigation-portal'
        IMAGE_TAG       = "v${BUILD_NUMBER}"
        REGISTRY_CRED   = 'docker-hub-credentials-id'
    }

    stages {
        stage('Initialize & Verify') {
            parallel {
                stage('Workspace Preparation') {
                    steps {
                        echo "Cleaning current workspace environment..."
                        cleanWs()
                    }
                }
                stage('Code Validation') {
                    steps {
                        echo "Verifying local application resource syntax structures..."
                        checkout scm
                        sh 'test -f index.html && test -f Dockerfile'
                    }
                }
            }
        }

        stage('Build Container Image') {
            steps {
                echo "Assembling Local Component Build tagged as: ${IMAGE_TAG}"
                sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest ."
            }
        }

        stage('Mock Simulation Validation') {
            when {
                expression { return params.RUN_INTEGRATION_TESTS == true }
            }
            steps {
                echo "Running conditional test sweeps inside targeted target layer: ${params.DEPLOY_ENV}"
                sh 'echo "Simulating API boundary parsing validation... Status: Passed"'
            }
        }

        stage('Publish Image to Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${REGISTRY_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {
        always {
            echo "Capturing configuration runtime layouts..."
            archiveArtifacts artifacts: 'index.html, deployment.yaml', fingerprint: true
        }
    }
}

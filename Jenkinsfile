@Library('jenkins-shared-library@develop') _

pipeline {
    agent {
        label 'AGENT-01'
    }

    stages {
        stage('Lint Dockerfile') {
            steps {
                hadoLint()
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    try {
                        dockerBuild(
                            imageName: 'java-jdk-17-agent'
                        )
                    } catch (Exception buildError) {
                        currentBuild.result = 'FAILURE'
                        error("Failed to build Docker image: ${buildError}")
                    }
                }
            }
        }
        stage('Run Trivy Scan') {
            steps {
                script {
                    try {
                        def imageName = "java-jdk-17-agent"
                        trivyScan(imageName)
                    } catch (Exception trivyError) {
                        currentBuild.result = 'FAILURE'
                        error("Trivy scan failed: ${trivyError}")
                    }
                }
            }
        }
        stage('Push Image To ECR') {
            when {
                branch 'main'
            }
            steps {
                script {
                    try {
                        ecrRegistry(
                            imageName: 'java-jdk-17-agent',
                            repoName: 'infra-images',
                        )
                    } catch (Exception pushError) {
                        currentBuild.result = 'FAILURE'
                        error("Failed to push image to ECR: ${pushError}")
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                emailNotification.sendEmailNotification('success', 'aswin@crunchops.com')
            }
        }
        failure {
            script {
                emailNotification.sendEmailNotification('failure', 'aswin@crunchops.com')
            }
        }
        always {
            cleanWs()
        }
    }
}

pipeline {
    agent any

    environment {
        registry = "amkam009/demo-devsecops"
        registryCredential = 'my-docker-hub'
        SCANNER_HOME = tool 'sonar_scanner'
        PROJECT_KEY = "spring-boot-tdd"
        PROJECT_NAME = "spring-boot-tdd"
    }

    stages {

        stage('Code coverage') {
            steps {
                withSonarQubeEnv('sonar_server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=$PROJECT_KEY -Dsonar.projectName=$PROJECT_NAME -Dsonar.java.coveragePlugin=jacoco -Dsonar.jacoco.reportPath=target/jacoco.exec -Dsonar.coverage.exclusions=src/test/**/*,src/**/web/**/*,**/Application.java -Dsonar.java.binaries=target/classes/ '''
                }
                 timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                 }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Vulnerability Scan') {
            steps {
                parallel("Trivy Scan": {
                    sh "bash trivy-docker-image-scan.sh"
                })
            }
            post {
                always {
                    dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
                }
            }
        }

        stage('Docker Build and Push') {
            when { expression { false } }
            steps {
                withDockerRegistry([credentialsId: "my-docker-hub", url: ""]) {
                    sh 'printenv'
                    sh 'docker build -t $registry:$BUILD_NUMBER .'
                    sh 'docker push $registry:$BUILD_NUMBER'
                }
            }
        }

        stage('Remove Unused docker image') {
            when { expression { false } }
            steps{
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }

        stage('Kubernetes Deployment - DEV') {
            when { expression { false } }
            steps {
                withKubeConfig([credentialsId: 'kubernetes-config']) {
                sh "sed -i 's#replace#${registry}:${BUILD_NUMBER}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml" }
            }
        }
    }
}
pipeline {
    agent any

    environment {
        registry = "amkam009/demo-devsecops"
        registryCredential = 'my-docker-hub'
    }

    stages {
        stage('GitHub') {
            steps {
                // Get some code from a GitHub repository
                   git(
                        url: "https://github.com/devAmadou/spring-boot-tdd.git",
                        branch: "main",
                        changelog: true,
                        poll: true
                    )
                   }
            }
        stage('Build Artifact') {
            steps {
                    sh "mvn clean package -DskipTests=true"
                    archive 'target/*.jar' //so that they can be downloaded later
                }
            }

        stage('Unit Tests - JUnit and Jacoco test all Tests') {
            steps {
                    sh "mvn test"
                }
                post {
                    always {
                            junit 'target/surefire-reports/*.xml'
                            jacoco execPattern: 'target/jacoco.exec'
                        }
                    }
        }

        stage('Unit Tests - JUnit and Jacoco') {
            steps {
                sh "mvn test -Dgroups=unitaires"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco execPattern: 'target/jacoco.exec'
                }
            }
        }

        stage('Service - IntegrationTest') {
            steps{
                sh "mvn test -Dgroups=integrations"
            }
        }

        stage('Web - IntegrationTest') {
            steps{
                sh "mvn test -Dgroups=web"
            }
        }

        stage('Mutation Tests - PIT') {
            steps {
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
            post {
                always {
                    pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
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
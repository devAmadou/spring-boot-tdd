pipeline {
    agent any
    stages {
        stage('GitHub') {
            steps {
                // Get some code from a GitHub repository
                   git(
                    url: "chemin-de-repo-github",
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
        }
}
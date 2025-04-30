pipeline {
    agent {
        kubernetes {
            label 'maven-agent'
            defaultContainer 'maven'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: maven-agent
spec:
  containers:
  - name: maven
    image: maven:3.9.4-eclipse-temurin-17
    command:
    - cat
    tty: true
    volumeMounts:
    - name: maven-cache
      mountPath: /root/.m2
  volumes:
  - name: maven-cache
    emptyDir: {}
"""
        }
    }

    environment {
        EMAIL = 'david.thibau@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                container('maven') {
                    git url: 'https://github.com/ton-repo/exemple-maven.git', branch: 'main'
                }
            }
        }

        stage('Build & Test') {
            steps {
                container('maven') {
                    echo 'Build du projet avec Maven...'
                    sh 'mvn -B -Dmaven.test.failure.ignore=true clean package'
                }
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
                success {
                    archiveArtifacts artifacts: 'target/*.jar', followSymlinks: false
                    stash name: 'app-binary', includes: 'target/*.jar'
                }
                failure {
                    mail to: "${EMAIL}",
                         subject: "❌ Build échoué",
                         body: "La pipeline Jenkins a échoué.",
                         from: 'jenkins@votredomaine.com'
                }
            }
        }
    }
}

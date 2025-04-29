pipeline {
   agent any 
tools {
  maven 'Maven 3'
  jdk 'jdk 21'
}


    stages {
        stage('Compile et tests') {
            steps {
                echo 'Unit test et packaging'
               sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            }
            post {
  always {
    // One or more steps need to be included within each condition's block.
         junit '**/target/surefire-reports/*.xml'
         

  }
  success {
    // One or more steps need to be included within each condition's block.
    archiveArtifacts 'application/**/*.jar'

  }
  failure {
    // One or more steps need to be included within each condition's block.
    sh 'echo sending mail failure'
    mail bcc: '', body: 'The  job failed with error', cc: '', from: '', replyTo: '', subject: 'Job failure', to: 'ahmed.assous@free.fr'

  }
}

             
        }
        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        sh ''
                    }
                    
                }
                 stage('Analyse Sonar') {
                     steps {
                        echo 'Analyse sonar'
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {

            steps {
                echo "Déploiement intégration"
                
            }
        }

     }
    
}


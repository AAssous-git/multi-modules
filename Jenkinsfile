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
        sh 'echo always executed'
         junit '**/target/surefire-reports/*.xml'
         
         

  }
  success {
    // One or more steps need to be included within each condition's block.
    archiveArtifacts 'application/**/*.jar'
    
                dir ('application/target') {
                 stash name:'application',include:'*.jar' 
      
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
                    agent any
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                       sh 'mvn -DskipTests verify'   
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent any
                     steps {
                        withCredentials([string(credentialsId: 'sonartoken', variable: 'SONAR_TOKEN')]) {
                             // some block
                             sh 'echo $credentialsId'
                        
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                        } 
                        
                     }
                    
                }
            }
            
        }
            
        stage('Déploiement intégration') {
             input {
  message 'Data center name'
  ok 'Deploy'
  submitter 'Paris,Londres,Madrid'
  parameters {
    choice choices: ['Paris', 'Londres', 'Madrid'], description: 'Choix data center', name: 'data_center'
  }

            steps {
                echo "Déploiement intégration $data_center "
                              
              }

                
            }
       }

     }
    
}


pipeline {
  


    stages {
        stage('Compile et tests') {
            agent  {
               kubernetes {
                  inheritFrom 'jdk17-agent'
               }
            }
            steps {
                container(name:'openjdk-17') {
               
               
                echo 'Unit test et packaging'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package'
            } 
         }
            post {
                always {
                    // One or more steps need to be included within each condition's block.
                    junit '**/target/surefire-reports/*.xml'
                }
                success {
                    // One or more steps need to be included within each condition's block.
                    archiveArtifacts artifacts: 'application/target/*.jar', followSymlinks: false
                    dir ('application/target') {
                        stash name: 'application', includes: '*.jar'
                    }
                }
                unsuccessful {
                    // One or more steps need to be included within each condition's block.
                    mail bcc: '', body: 'Pipeline en erreur', cc: '', from: 'jenkins@plbformation.com', replyTo: '', subject: 'Error !', to: 'ahmed.assous@free.fr'
                }
            }
             
        }

        
    }    
}

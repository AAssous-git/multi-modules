@Library('global_lib') _
pipeline {
   agent none 
   options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
    }
    tools {
        maven 'Maven 3'
    }


    stages {
        stage('Compile et tests') {
            agent {
                docker {
                image 'openjdk:17-alpine'
                args '-v $HOME/.m2:/root/.m2'

  }
}
            steps {
                echo 'Unit test et packaging'
                sh './mvnw -Dmaven.test.failure.ignore=true clean package'
                /*createTarGz sourceDir:'.',extension:['*.xml','*.java'],outputDir:'/tmp'*/
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
                    mail bcc: '', body: 'Pipeline en erreur', cc: '', from: 'jenkins@plbformation.com', replyTo: '', subject: 'Error !', to: 'david.thibau@gmail.com'
                }
            }
             
        }
/*        stage('Analyse qualité et vulnérabilités') {
            parallel {
                stage('Vulnérabilités') {
                    agent any 
                    steps {
                        echo 'Tests de Vulnérabilités OWASP'
                        withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_API_KEY')]) {
                            sh 'mvn verify -Dnvd.api.key=$NVD_API_KEY -DskipTests'
                        }
                    }
                    post {
                        success {
                            // One or more steps need to be included within each condition's block.
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'application/target', reportFiles: 'dependency-check-report.html', reportName: 'Analyse de dépendances OWASP', reportTitles: '', useWrapperFileDirectly: true])                        
                        }
                    }
                    
                }
                 stage('Analyse Sonar') {
                    agent any 
                    environment {
                        SONAR_TOKEN = credentials('SONAR_TOKEN')
                    }
                     steps {
                        echo 'Analyse sonar'
                        sh 'mvn -Dsonar.token=${SONAR_TOKEN} clean integration-test sonar:sonar'
                     }
                    
                }
            }
            
        }
  */          
        stage('Déploiement intégration') {

            /*when {
                branch 'main'
                beforeOptions true
                beforeInput true
                beforeAgent true
            }*/
            options {
                timeout(2)
            }
            agent any
            input {
                message 'Vers quel datacenter voulez-vous déployer ?'
                ok 'Déployer'
                parameters {
            choice choices: ['Paris', 'Lille', 'Toulouse'], name: 'City'
  }
                
            }

            steps {
                echo "Déploiement intégration "
                unstash 'application'
                /*sh 'cp *.jar /home/plb/MyWork/multi-module/serveurs/${DATACENTER}.jar' */
                script {
                   def prop = readJSON file: 'deployment.jason', text: ''
	               def datacenters=prop ['dataCenters']	
                   for (datacenter in datacenters)   {
                    if (fileExists("$datacenter")) {
                      echo "Folder $datacenter"
                    }
                    else
                    {sh "mkdir $datacenter"
                     def exitStatus=sh returnStatus:true,script :"cp *jar $datacenter"
                     sh "echo copy to $datacenter with return code $exitStatus"} 
                     
                    }

                   
                }
            }
        }

     }
   /*  api */

   /* end api*/  
}


def checkSonarQualityGate(){
    // Get properties from report file to call SonarQube 
    def sonarReportProps = readProperties  file: 'target/sonar/report-task.txt'
    def sonarServerUrl = sonarReportProps['serverUrl']
    def ceTaskUrl = sonarReportProps['ceTaskUrl']
    def ceTask

    // Get task informations to get the status
    timeout(time: 4, unit: 'MINUTES') {
        waitUntil {
            withCredentials ([string(credentialsId: 'sonarUser', variable : 'token')]) {
                def response = sh(script: "curl -u ${token}: ${ceTaskUrl}", returnStdout: true).trim()
                ceTask = readJSON text: response
            }

            echo ceTask.toString()
              return "SUCCESS".equals(ceTask['task']['status'])
        }
    }

    // Get project analysis informations to check the status
    def ceTaskAnalysisId = ceTask['task']['analysisId']
    def qualitygate

    withCredentials ([string(credentialsId: 'sonarUser', variable : 'token')]) {
        def response = sh(script: "curl -u ${token}: ${sonarServerUrl}/api/qualitygates/project_status?analysisId=${ceTaskAnalysisId}", returnStdout: true).trim()
        qualitygate =  readJSON text: response
    }

    echo qualitygate.toString()
    if ("ERROR".equals(qualitygate['projectStatus']['status'])) {
        error "Quality Gate failure"
    }
}

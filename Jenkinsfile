pipeline{
    agent any
       environment{
        VERSION = "${env.BUILD_ID}"
        NEXUS_REPO_URL = 'http://34.122.89.158:8081/repository/helm-hosted/'
        HELM_CHART_PATH = './myapp-0.1.0.tgz' // Path to the packaged Helm chart
       }
    stages{
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                   }

                timeout(time:1, unit:'HOURS'){
                    def qg = waitForQualityGate()
                     if (qg.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }    
                  }
                }
           }    
       }
        stage("docker build and push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker-pass', variable: 'docker_p')]) {
                        sh '''
                         docker build -t 34.122.89.158:8083/saurav:${VERSION} .
                         echo $docker_p | docker login -u admin --password-stdin 34.122.89.158:8083 
                         docker push 34.122.89.158:8083/saurav:${VERSION}     
                         docker rmi 34.122.89.158:8083/saurav:${VERSION}        
                        '''
                    }
                }
            } 
        }
        stage("helm config check using datree"){
            steps{
                script{
                 dir('kubernetes/') {
                   
                       sh 'helm datree test myapp/'
                  
              }
            }
          } 
        }
       stage("push helm charts to nexus helm repo"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker-pass', variable: 'docker_p')]) {
                        dir('kubernetes/') {
                        sh '''
                        curl -u admin:$docker_p --upload-file ${HELM_CHART_PATH} ${NEXUS_REPO_URL}
                        '''
                       }
                    }
                }
            }
         }
      }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType                         : 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "sauravengineer986@gmail.com";  
		    }
	}
}    

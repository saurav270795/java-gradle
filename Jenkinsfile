pipeline{
    agent any
       environment{
        VERSION = "${env.BUILD_ID}"
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
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType                         : 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "sauravengineer986@gmail.com";  
		    }
	}
}    

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
       stage("push helm charts to nexus helm repo"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker-pass', variable: 'docker_p')]) {
                        dir('kubernetes/') {
                        sh '''
                        helmversion=$(helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                        tar -czvf myapp-${helmversion}.tgz myapp/
                        curl -u admin:$docker_p http://34.122.89.158:8081/repository/helm-hosted/  --upload-file myapp-${helmversion}.tgz -v
                        '''
                       }
                    }
                }
            }
         }
        stage("deploying helm charts to K8cluster"){
            steps{
                script{
                       dir('kubernetes/') {
                        sh 'helm upgrade --install --set image.repository="34.122.89.158:8083/saurav" --set image.tag="${VERSION}" --kubeconfig /var/lib/jenkins/.kube/config javaapp myapp/ --debug'
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

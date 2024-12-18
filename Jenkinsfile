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
                    withCredentials([string(credentialsId: 'docker-pass', variable: 'docker-pass')]) {
                        sh '''
                         docker build -t 34.122.89.158:8083/saurav:${VERSION} .
                         docker login -u admin -p $docker-pass 34.122.89.158:8083 
                         docker push 34.122.89.158:8083/saurav:${VERSION}             
                        '''
                    }
                }
            } 
        }
    }
}    

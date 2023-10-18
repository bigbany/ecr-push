pipeline {
    agent any 
    
    stages{
    stage('git clone'){
        steps{
            git branch:'main', credentialsId: github_token , url:  'https://github.com/bigbany/ecr-push.git'
            
        }
        
    }
    stage('build'){
        steps{
            sh 'docker build -t p1 .'
        }
        
    }
    
     stage('test run'){
        steps{
           
           sh 'docker run -dp 8888:80 --rm --name p1 p1'
        }
        
    }
    
    stage('application test') {
            steps {
                script {
                    def status = sh(script: "curl -sLI -w '%{http_code}' localhost:8888 -o /dev/null", returnStdout: true).trim()
                    if (status != '200' && status != '201') {
                        error("Returned status code = $status when calling test")
                    }
                }
            }
            post {
                failure {
                    sh 'docker stop p1'
                }
            }
        }

stage('cleanup test'){
    steps{
        sh 'docker stop p1'
    }

    
    
      }
    
      
         stage('push to ecr') {
            steps {
                script {
                    
              
                    sh 'aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 241825837502.dkr.ecr.ap-northeast-2.amazonaws.com'
                    sh 'docker tag p1:latest 241825837502.dkr.ecr.ap-northeast-2.amazonaws.com/ecr-jenkins:latest'
                    sh 'docker push 241825837502.dkr.ecr.ap-northeast-2.amazonaws.com/ecr-jenkins:latest'
                }
            }
        }


 stage('deploy script run') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                        sh 'ssh -o StrictHostKeyChecking=no -i ~/cwlabkey.pem ubuntu@172.31.49.203 "docker ps -q | xargs docker stop"'
                    }
                    sh 'ssh -o StrictHostKeyChecking=no -i ~/cwlabkey.pem ubuntu@172.31.49.203 "aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin 241825837502.dkr.ecr.ap-northeast-2.amazonaws.com; \
                    docker rmi 241825837502.dkr.ecr.ap-northeast-2.amazonaws.com/ecr-jenkins:latest; \
                    docker run -d --rm -p 80:80 --name nginx 241825837502.dkr.ecr.ap-northeast-2.amazonaws.com/ecr-jenkins:latest "'
                }
            }
        }


      
      
      
    }
    
    
    
    
}
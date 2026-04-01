pipeline {
     // 전역 에이전트를 사용하지 않음으로써 컨테이너 중첩 방지
    agent none 

    environment {
        AWS_DEFAULT_REGION = 'ap-northeast-2'
        AWS_ECS_CLUSTER = 'punctual-crocodile-2buaov'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod1'
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'

    }

    stages {

        stage('Build') {
            agent {
                docker { 
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy' 
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo '빌드 시작..'
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Build Docker image') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''" 
                }
            }
            steps {
                sh '''
                    amazon-linux-extras install docker
                    docker build -t myjenkinsapp .
                '''
            }
        }


        stage('Deploy to AWS') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''" 
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LASTEST_TD_REVISTION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LASTEST_TD_REVISTION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LASTEST_TD_REVISTION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
                
                
            }
        }

       
    }
  
}

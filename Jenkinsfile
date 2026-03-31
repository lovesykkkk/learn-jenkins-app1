pipeline {
    agent none 

    environment {
        NETLIFY_SITE_ID = '73813ef4-d1e2-4a85-9504-59c9613b5230'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('AWS') {
            agent {
                docker { 
                    image 'amazon/aws-cli'
                    args "--entrypoint=''" 
                }
            }
            steps {
                sh 'aws --version'
            }
        }

        // 2. 빌드 및 도구 설치
        stage('Build') {
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            steps {
                sh '''
                    echo '빌드 및 필요한 도구 설치 시작..'
                    npm ci
                    # 배포와 서버 실행에 필요한 도구를 미리 설치 (매 스테이지 설치 방지)
                    npm install netlify-cli@20.1.1 serve
                    npm run build
                '''
            }
        }

        stage('Test') {
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            steps {
                sh 'npm test'
            }
        }

        stage('E2E') {
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            steps {
                sh '''
                    # 미리 설치된 serve 사용 (설치 시간 0초)
                    ./node_modules/.bin/serve -s build & sleep 10
                    npx playwright test --reporter=html
                '''
            }
        }

        stage('Deploy staging') {
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' } 
            }
            steps {
                sh '''
                    # -s 옵션 추가: 업로드만 하고 즉시 종료 (대기 시간 삭제)
                    ./node_modules/.bin/netlify deploy --dir=build -s
                '''
            }
        }

        stage('Approval'){
            agent none
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: '운영환경에 배포할까요?', ok: '네 배포합니다'
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            steps {
                sh '''
                    # -s 옵션 추가: 운영 배포도 대기 없이 즉시 완료
                    ./node_modules/.bin/netlify deploy --dir=build --prod -s
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://shiny-beignet-85f4a6.netlify.app'
            }
            steps {
                sh 'npx playwright test --reporter=html'
            }
        }
    }

    post {
        always {
            node {
                junit 'jest-results/junit.xml'
            }
        }
    }
}
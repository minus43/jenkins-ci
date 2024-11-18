// 필요한 변수를 선언할 수 있다. (내가 직접 선언하는 변수, 젠킨스 환경변수를 끌고 올 수 있음)
def deployHost="172.31.26.129"  // 배포서버 private IP
def dockerHubUsername="space0403"
def dockerHubRepository="my-app-image"


// 젠킨스의 선언형 파이프라인 정의부 시작 (그루비 언어) 
pipeline {
    agent any

    stages {
        stage('Pull Codes from Github'){
            steps{
                checkout scm    // 젠킨스와 연결된 소스 컨트롤 매니저(git 등)에서 코드를 가져오는 명령
            }
        }
        stage('Build Codes by Gradle') {
            steps {
                sh """
                ./gradlew clean build
                """
            }
        }
        stage('Build Docker Image & Push to Docker Hub') {
            steps {
                sh """
                    # Docker Hub 로그인
                    echo "${env.DOCKER_HUB_PASSWORD}" | docker login -u "${dockerHubUsername}" --password-stdin
                    
                    # Docker 이미지 빌드
                    docker build -t ${dockerHubUsername}/${dockerHubRepository}:${currentBuild.number} .
                    
                    # Docker Hub로 푸시
                    docker push ${dockerHubUsername}/${dockerHubRepository}:${currentBuild.number}
                """
            }
        }
        stage('Deploy to AWS EC2 VM') {
            steps {
                sshagent(credentials: ["deploy-key"]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${deployHost} \
                    'docker login -u ${dockerHubUsername} -p ${env.DOCKER_HUB_PASSWORD}; \
                    docker pull ${dockerHubUsername}/${dockerHubRepository}:${currentBuild.number}; \
                    docker stop \$(docker ps -q) || true; \
                    docker run -d -p 80:8080 ${dockerHubUsername}/${dockerHubRepository}:${currentBuild.number};'
                    """
                }
            }
        }
    }
}
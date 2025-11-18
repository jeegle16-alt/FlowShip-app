pipeline {
    agent any
    
    environment {
        // Jenkins Credentials ID (이미 만들어둔 거)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-jeegle16')
        DOCKER_IMAGE = "jeegle16/django-pybo"
    }

    // 1분마다 Git 변경 있는지 체크
    triggers {
        pollSCM('H/1 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Git Repo 최신 내용 가져오기"
                // 이미 Job에 SCM 설정이 있으니까 이게 제일 깔끔
                checkout scm
            }
        }

        stage('Set Version') {
            steps {
                script {
                    // Jenkins 빌드 번호 기반 버전 태그 (v1, v2, v3, ...)
                    VERSION = "v${BUILD_NUMBER}"
                    IMAGE_TAG_BUILD  = "${DOCKER_IMAGE}:${VERSION}"   // 예: jeegle16/django-pybo:v5
                    IMAGE_TAG_STABLE = "${DOCKER_IMAGE}:v3"           // K8s에서 사용 중인 고정 태그

                    echo "빌드 버전 태그: ${IMAGE_TAG_BUILD}"
                    echo "배포용 고정 태그: ${IMAGE_TAG_STABLE}"
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh """
                    docker build \\
                      -t ${IMAGE_TAG_BUILD} \\
                      -t ${IMAGE_TAG_STABLE} \\
                      .
                    """
                }
            }
        }

        stage('Docker Login & Push') {
            steps {
                script {
                    sh """
                    echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                    docker push ${IMAGE_TAG_BUILD}
                    docker push ${IMAGE_TAG_STABLE}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "성공적으로 Docker Hub에 push 완료!"
        }
        failure {
            echo "빌드 실패! 로그를 확인하세요."
        }
    }
}


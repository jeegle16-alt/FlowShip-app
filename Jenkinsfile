pipeline {
    agent any

    environment {
        // Docker Hub Credentials (Jenkins에 만들어둔 ID)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-jeegle16')   // 🔴 네 ID랑 맞을 것
        // 도커 이미지 이름
        DOCKER_IMAGE = "jeegle16/django-pybo"                        // 🔴 네 Docker Hub 레포
        // Ops Repo URL
        OPS_REPO_URL = "https://github.com/jeegle16-alt/minipro2_opsrepo.git"   // 🔴 네 GitHub 레포
    }

    // ✅ 자동 CI 트리거: 1분마다 Git 변경 체크
    triggers {
        pollSCM('H/1 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                echo "App Repo에서 코드 가져오기"
                checkout scm
            }
        }

        stage('Set Version') {
            steps {
                script {
                    // 빌드 번호 기반 버전 (v1, v2, v3, ...)
                    env.VERSION   = "v${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = "${DOCKER_IMAGE}:${env.VERSION}"

                    echo "이번 빌드 버전: ${env.VERSION}"
                    echo "이번 이미지 태그: ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh """
                    docker build \\
                      -t ${IMAGE_TAG} \\
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
                    docker push ${IMAGE_TAG}
                    """
                }
            }
        }

        // ✅ 여기서부터 "자동 CD" 핵심: Ops Repo의 image 태그를 자동으로 바꿈
        stage('Update Ops Repo') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-token',  // 🔴 위에서 만든 GitHub credential ID
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                        # 1) 깨끗하게 클론
                        rm -rf ops-repo
                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/jeegle16-alt/minipro2_opsrepo.git ops-repo
                        cd ops-repo

                        echo "[기존 image 줄]"
                        grep 'image:' k8s/mysite/django-deploy.yaml || true

                        # 2) 이미지 태그를 이번 빌드 버전으로 교체
                        sed -i 's#image: .*/django-pybo:.*#image: ${DOCKER_IMAGE}:${VERSION}#' k8s/mysite/django-deploy.yaml

                        echo "[변경 후 image 줄]"
                        grep 'image:' k8s/mysite/django-deploy.yaml || true

                        # 3) Git 설정 + 커밋 + 푸시
                        git config user.name "jenkins"
                        git config user.email "jenkins@example.com"

                        git add k8s/mysite/django-deploy.yaml || true
                        git commit -m "Update image tag to ${VERSION}" || echo "No changes to commit"

                        git push origin main || echo "No changes pushed"
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "CI + CD 파이프라인 성공 (Docker Hub & Ops Repo 반영 완료)"
        }
        failure {
            echo "파이프라인 실패 - Jenkins 콘솔 로그 확인 필요"
        }
    }
}


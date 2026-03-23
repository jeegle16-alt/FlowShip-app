# FlowShip — App Repository

> Django 애플리케이션 소스코드 + Jenkins CI 파이프라인

**FlowShip**은 Django 웹 서비스의 코드 변경부터 Kubernetes 배포까지 전 과정을 자동화하는 CI/CD 프로젝트입니다.  
이 레포는 **애플리케이션 소스코드와 CI 파이프라인(Jenkinsfile)**을 관리하며, 배포 매니페스트는 [FlowShip-ops](https://github.com/<your-username>/FlowShip-ops) 레포에서 관리됩니다.

---

## CI/CD Pipeline

<img width="933" height="248" alt="image" src="https://github.com/user-attachments/assets/20ddd5c0-fdd3-411c-b936-1e2fbba2789a" />


**파이프라인 흐름**: Code Push → Jenkins 자동 감지 (1분 polling) → Docker Build & Push → Ops Repo 이미지 태그 자동 업데이트 → ArgoCD 자동 Sync → Kubernetes 배포 완료

---

## Project Structure

```
FlowShip-app/
├── Jenkinsfile              # CI 파이프라인 정의
├── Dockerfile               # 컨테이너 이미지 빌드
├── requirements.txt         # Python 의존성
├── manage.py
├── config/                  # Django 설정
│   ├── settings.py          # MySQL 연결, ALLOWED_HOSTS 등
│   ├── urls.py
│   └── wsgi.py
├── pybo/                    # 메인 앱 (Q&A 게시판)
│   ├── models.py
│   ├── views/
│   ├── urls.py
│   ├── forms.py
│   └── templatetags/
├── common/                  # 인증 (로그인/회원가입)
├── templates/               # Django 템플릿
│   ├── base.html
│   ├── navbar.html
│   └── pybo/
└── static/                  # CSS, JS (Bootstrap)
```

---

## Tech Stack

| 구분 | 기술 |
|------|------|
| Framework | Django 4.x (Python 3.10) |
| Database | MySQL 8.0 (K8s Pod, Secret 기반) |
| CI | Jenkins 2.452+ (Pipeline, pollSCM) |
| Container | Docker (python:3.10-slim base) |
| Registry | Docker Hub (`jeegle16/django-pybo`) |

---

## Jenkinsfile Stages

| Stage | 동작 |
|-------|------|
| **Checkout** | App Repo 코드 가져오기 |
| **Set Version** | 빌드 번호 기반 버전 태그 생성 (`v1`, `v2`, ...) |
| **Docker Build** | `docker build -t jeegle16/django-pybo:v{N}` |
| **Docker Login & Push** | Docker Hub에 이미지 업로드 |
| **Update Ops Repo** | Ops Repo의 `django-deploy.yaml` 이미지 태그를 `sed`로 자동 교체 후 push |

Jenkins가 Ops Repo까지 업데이트하면, ArgoCD가 변경을 감지하여 Kubernetes 클러스터에 자동 배포합니다.

---

## Local Development

```bash
# 의존성 설치
pip install -r requirements.txt

# DB 마이그레이션 (MySQL 연결 필요)
python manage.py migrate

# 개발 서버 실행
python manage.py runserver 0.0.0.0:8000
```

### Docker로 실행

```bash
docker build -t django-pybo:local .
docker run -d -p 8000:8000 --name django-test django-pybo:local
```


## Related Repository

| Repo | 역할 |
|------|------|
| **FlowShip-app** (이 레포) | 애플리케이션 소스코드 + CI |
| [**FlowShip-ops**](https://github.com/jeegle16-alt/FlowShip-ops) | Kubernetes 매니페스트 + CD (ArgoCD) |

물론입니다! 아래는 위의 내용을 한국어로 번역한 버전입니다.

---

# Flask 쇼핑몰 애플리케이션 with CI/CD 파이프라인

컨테이너화된 Flask 전자상거래 애플리케이션을 Jenkins와 ArgoCD를 이용한 자동화된 CI/CD 파이프라인으로 쿠버네티스 환경에 배포합니다.

## 🏗️ 아키텍처

이 프로젝트는 다음과 같은 구성 요소로 구성된 마이크로서비스 아키텍처를 구현합니다:

* **프론트엔드**: Nginx 리버스 프록시
* **백엔드**: Flask 애플리케이션 (백그라운드 작업을 위한 Celery 포함)
* **캐시/메시지 브로커**: Redis
* **데이터베이스**: MySQL
* **스토리지**: Google Cloud Storage (GCS)
* **CI/CD**: Jenkins + ArgoCD (GitOps 방식)

## 📋 사전 준비사항

* 쿠버네티스 클러스터
* Docker Hub 계정
* GitHub 저장소 (소스 코드용 + 매니페스트용)
* Jenkins 설치 및 필수 플러그인 구성
* 쿠버네티스 클러스터에 ArgoCD 설치
* GCS 사용을 위한 Google Cloud 자격 증명

## 🚀 빠른 시작

### 1. 저장소 클론

```bash
# 애플리케이션 소스 코드
git clone https://github.com/byungju-oh/shop.git

# Kubernetes 매니페스트 (이 저장소)
git clone https://github.com/byungju-oh/argojenkins.git
```

### 2. 시크릿 설정

필요한 쿠버네티스 시크릿을 생성합니다:

```bash
# MySQL 및 애플리케이션 시크릿
kubectl apply -f was/sec.yaml

# GCS 자격 증명 시크릿
kubectl create secret generic google-credentials \
  --from-file=google-credentials.json=/path/to/your/gcs-credentials.json
```

### 3. ArgoCD로 배포

```bash
# ArgoCD 애플리케이션 매니페스트 적용
kubectl apply -f application.yaml
```

## 📁 프로젝트 구조

```
├── nginx/
│   ├── dep.yaml          # Nginx 배포 및 ConfigMap
│   └── ser.yaml          # Nginx 서비스
├── was/
│   ├── dep.yaml          # Flask 애플리케이션 배포 및 서비스
│   ├── redis.yaml        # Redis 배포 및 서비스
│   ├── celery.yaml       # Celery 워커 배포
│   ├── sec.yaml          # 시크릿 구성
│   └── version.txt       # 현재 애플리케이션 버전
├── Jenkinsfile           # CI/CD 파이프라인 설정
├── Jenkinsfile.back      # 백업 파이프라인 구성
└── application.yaml      # ArgoCD 애플리케이션 정의
```

## 🔧 설정 정보

### 환경 변수

Flask 애플리케이션에서 사용하는 주요 환경 변수는 다음과 같습니다:

* `DATABASE_URI`: MySQL 접속 문자열
* `SECRET_KEY`: Flask 시크릿 키
* `GCS_BUCKET_NAME`: GCS 버킷 이름
* `GOOGLE_APPLICATION_CREDENTIALS`: GCS 자격 증명 경로
* `CELERY_BROKER_URL`: Celery 브로커용 Redis URL
* `CELERY_RESULT_BACKEND`: Celery 결과 백엔드 Redis URL

### 리소스 제한

#### Flask 애플리케이션

* **요청값**: 250Mi 메모리, 210m CPU
* **제한값**: 280Mi 메모리, 220m CPU

#### Nginx

* **요청값**: 64Mi 메모리, 250m CPU
* **제한값**: 128Mi 메모리, 500m CPU

#### Celery 워커

* **요청값**: 250Mi 메모리, 210m CPU
* **제한값**: 280Mi 메모리, 220m CPU

## 🔄 CI/CD 파이프라인

Jenkins 파이프라인은 아래의 작업을 자동화합니다:

1. **저장소 클론**: 소스 코드 및 매니페스트 가져오기
2. **버전 관리**: 버전 자동 증가
3. **빌드**: 새로운 버전 태그로 Docker 이미지 생성
4. **푸시**: Docker Hub로 이미지 푸시
5. **매니페스트 업데이트**: 새로운 이미지 버전으로 Kubernetes 매니페스트 수정
6. **GitOps 배포**: 변경 커밋 → ArgoCD 자동 동기화 트리거
7. **슬랙 알림**: 빌드 성공/실패 시 알림 전송

### 파이프라인 트리거

파이프라인은 다음과 같은 방법으로 실행할 수 있습니다:

* Git 웹훅 (소스 코드 변경 시)
* 수동 실행
* 스케줄 기반 빌드

## 🎯 주요 기능

### 애플리케이션 기능

* 전자상거래 기능 구현
* Celery를 통한 비동기 작업 처리
* GCS를 활용한 파일 저장 기능
* 헬스 체크 및 모니터링 기능
* 수평 확장 준비 완료

### DevOps 기능

* **GitOps**: ArgoCD를 통한 배포 관리
* **자동 버전 관리**: Semantic 버전 자동 처리
* **헬스 모니터링**: Liveness/Readiness probe 설정
* **로드 밸런싱**: 내부 로드 밸런서 구성
* **시크릿 관리**: Kubernetes 시크릿을 통한 민감 정보 보호

## 🔍 모니터링 및 헬스 체크

### 헬스 엔드포인트

* **Liveness Probe**: `GET /` (포트 5000)
* **Readiness Probe**: `GET /` (포트 5000)

### Probe 설정

* **초기 지연**: Flask 30초, Nginx 5초
* **주기**: 10초
* **타임아웃**: 기본값 (1초)

## 🌐 네트워크 구성

### 서비스 설정

* **Nginx**: 외부 노출용 LoadBalancer (포트 80)
* **Flask**: 내부 LoadBalancer (예: 10.178.0.100:80)
* **Redis**: ClusterIP (포트 6379)

### DNS 설정

* `ndots=2` 설정을 활용한 클러스터 우선 네임 리졸빙

## 🚨 문제 해결

### 자주 발생하는 문제

1. **이미지 풀 오류**

   ```bash
   kubectl describe pod <pod-name>
   # Docker Hub에 이미지 존재 여부 확인
   ```

2. **시크릿 마운트 오류**

   ```bash
   kubectl get secrets
   kubectl describe secret mysql-secret
   ```

3. **서비스 연결 오류**

   ```bash
   kubectl get services
   kubectl get endpoints
   ```

### 로그 확인

```bash
# Flask 애플리케이션 로그
kubectl logs -f deployment/flask-app

# Celery 워커 로그
kubectl logs -f deployment/celery

# Nginx 로그
kubectl logs -f deployment/nginx-proxy
```

## 📈 확장

### 수평 확장

```bash
# Flask 애플리케이션 확장
kubectl scale deployment flask-app --replicas=3

# Celery 워커 확장
kubectl scale deployment celery --replicas=2
```

### 수직 확장

각 배포 YAML 파일에서 리소스 요청/제한 값을 수정

## 🔐 보안

* 시크릿은 base64로 인코딩되어 Kubernetes 시크릿에 저장
* GCS 자격 증명은 파일로 마운트됨
* Flask는 내부 로드 밸런서를 통해 외부 접근 차단
* 컨테이너는 루트 권한 없이 실행

## 🤝 기여 방법

1. 저장소 포크
2. 기능 브랜치 생성
3. 변경 사항 반영
4. 충분한 테스트
5. Pull Request 생성

## 📄 라이선스

이 프로젝트는 MIT 라이선스를 따릅니다. 자세한 내용은 LICENSE 파일을 참고하세요.

## 📞 지원

문제 발생 시 다음 방법을 이용하세요:

* 저장소에 이슈 등록
* 문제 해결 섹션 확인
* 애플리케이션 로그 확인


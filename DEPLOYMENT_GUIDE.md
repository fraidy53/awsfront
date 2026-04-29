# 프론트엔드 배포 방식 정리

## 1. S3 배포 (deployS3.yml)
- **deployS3.yml**은 GitHub Actions를 이용해 React 앱을 빌드하고, 빌드된 정적 파일을 AWS S3 버킷에 업로드하는 자동화 워크플로우입니다.
- S3는 정적 웹사이트 호스팅에 적합하며, CloudFront와 연동해 글로벌 CDN 배포도 가능합니다.
- 빌드 시점에 환경변수(`REACT_APP_API_BASE_URL`)를 GitHub Secrets에서 받아 빌드에 반영합니다.
- Dockerfile은 사용하지 않습니다.

### S3 배포 흐름
1. GitHub Actions가 main 브랜치에 push될 때 실행
2. Node.js 환경 세팅 및 의존성 설치
3. `npm run build`로 정적 파일 생성 (환경변수는 Actions에서 직접 주입)
4. AWS 인증 후 S3 버킷에 `build/` 폴더 전체 업로드
5. (선택) CloudFront 캐시 무효화

---

## 2. EC2 + Docker 배포
- EC2 서버에서 Docker 컨테이너로 React 앱을 실행하는 방식입니다.
- Dockerfile이 반드시 필요하며, 빌드 타임에 `.env` 파일을 복사해 환경변수를 반영합니다.
- 보통 Nginx를 통해 80포트로 서비스합니다.

### EC2 + Docker 배포 흐름
1. GitHub Actions에서 `.env` 파일을 GitHub Secrets 값으로 생성
2. Dockerfile에서 `.env`를 복사(`COPY .env .env`)
3. `npm run build`로 정적 파일 생성 (환경변수는 .env에서 읽음)
4. 빌드된 이미지를 ECR에 push
5. EC2에서 이미지를 pull 받아 컨테이너 실행

---

## 3. 각 방식의 차이점
| 구분         | S3 배포 (deployS3.yml)         | EC2 + Docker 배포           |
|--------------|-------------------------------|-----------------------------|
| 정적 파일 호스팅 | S3 (CloudFront 가능)           | EC2 (Nginx 등)              |
| Dockerfile 사용 | ❌ (필요 없음)                  | ✅ (필수)                    |
| 환경변수 주입   | Actions에서 직접 env로 주입     | .env 파일 생성 후 Dockerfile에서 복사 |
| 서버 유지관리   | 서버리스 (S3 관리만)            | EC2 인스턴스 직접 관리 필요   |
| 확장성/비용     | 저렴, 자동 확장                | 직접 관리, 비용 발생          |

---

## 4. 용어 정리
- **S3**: AWS의 객체 스토리지 서비스. 정적 웹사이트 호스팅 가능.
- **EC2**: AWS의 가상 서버. 직접 컨테이너/서버를 띄워야 함.
- **Dockerfile**: 컨테이너 이미지를 빌드하는 스크립트. React 빌드 및 Nginx 설정 포함 가능.
- **GitHub Actions**: CI/CD 자동화 도구. 배포 파이프라인 작성에 사용.
- **ECR**: AWS의 Docker 이미지 저장소. EC2 배포 시 사용.
- **CloudFront**: AWS CDN 서비스. S3와 연동해 전세계 빠른 배포 가능.

---

## 5. 참고
- S3 배포는 서버리스, 저렴, 유지보수 편리. 동적 기능 필요하면 Lambda/API Gateway 등과 연동.
- EC2 + Docker는 커스텀 서버 환경, 복잡한 백엔드 연동, 직접 관리가 필요한 경우 적합.
- React 환경변수는 반드시 빌드 타임에 주입되어야 하며, 런타임 -e 옵션만으로는 적용되지 않음.

## 사용될 기술

- **AWS S3**
    - 정적 웹사이트를 호스팅할 수 있는 객체 스토리지 서비스
    - HTML/CSS/JS 파일 등을 저장하고 배포하는 데 자주 사용
- **AWS CloudFront**
    - 전 세계에 분산된 엣지 로케이션을 통해 콘텐츠를 빠르게 전달하는 CDN 서비스
    - S3와 연동하여 정적 웹사이트의 속도와 보안을 향상 시킬 수 있음
- **Github**
    - 소스 코드 버전 관리를 위한 플랫폼
    - 협업, 브랜치 전략, 이슈 관리 등 다양한 개발 워크플로우를 지원
- **Github Actions**
    - GitHub 저장소에서 CI/CD 파이프라인을 자동화할 수 있는 도구
    - 코드 푸시/PR 등을 트리거로 테스트, 빌드, 배포 작업을 실행시킴

## 인프라 및 CI/CD 세팅 과정 간단하게~

### AWS를 사용한 배포

1. AWS S3, CloudFront Credential 설정 (+다른 사용자 필요 시 IAM 설정)
2. Next.js로 프로젝트 작성 및 프로젝트 빌드
3. 빌드 결과물을 S3에 업로드

### GitHub Actions를 사용한 CI/CD

1. GitHub Repository에 AWS 키 설정
2. 배포 워크플로우를 위한 yml 파일(`.github/workflows/deployment.yml`) 작성

## 배포 파이프라인 설명

![image](https://github.com/user-attachments/assets/2d7c8e4f-f382-4be3-bde8-bf96ee10be08)

## GitHub Actions 설정 파일

### Workflow 기본 정보

```yaml
name: Deploy Next.js to S3 and invalidate CloudFront
```

- 이 워크플로우의 이름을 지정하는 작업
- GitHub Actions 탭에서 이 이름으로 구분됨

### 트리거 설정

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

- `push` 트리거: `main` 브랜치에 코드가 push될 때 워크플로우 실행
- `workflow_dispatch`: 수동 실행 버튼을 GitHub Actions 탭에서 제공

### Job 정의

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
```

- `deploy`라는 이름의 Job을 정의
- 최신 Ubuntu 환경에서 실행

### 단계별 실행

```yaml
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
```

- 현재 리포지토리의 코드를 현재 CI 작업 환경으로 가져옴

```yaml
    - name: Install dependencies
      run: npm ci
```

- `npm install`보다 더 빠르고 일관된 환경 보장

```yaml
    - name: Build
      run: npm run build
```

- 프로젝트 빌드
- Next.js의 경우 next/ 디렉토리 또는 out/을 만듦 (정적 export 시에는 out/)

### AWS 자격증명 구성

```yaml
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}
```

- GitHub Secrets에 저장된 AWS 자격증명을 사용해 AWS CLI 사용 인증
- `aws s3`, `aws cloudfront` 명령어를 인증된 상태에서 사용 가능하도록 함

### S3에 배포

```yaml
    - name: Deploy to S3
      run: |
        aws s3 sync out/ s3://${{ secrets.S3_BUCKET_NAME }} --delete
```

- 정적 빌드 결과물(`out/` 폴더)을 S3 버킷에 업로드
- `--delete` : S3에서 삭제된 파일을 자동 반영

### CloudFront 캐시 무효화

```yaml
    - name: Invalidate CloudFront cache
      run: |
        aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
```

- CDN 캐시 무효화 → 새로 배포된 파일 즉시 반영
- `"/*"` : 전체 파일 캐시를 지우겠다는 뜻

## 주요 링크

- S3 버킷 웹사이트 엔드포인트: [http://my-hanghae-bucket.s3-website-us-east-1.amazonaws.com](http://my-hanghae-bucket.s3-website-us-east-1.amazonaws.com/)
- CloudFrount 배포 도메인 이름: [https://dexjim9moxwgs.cloudfront.net](https://dexjim9moxwgs.cloudfront.net/)

## 주요 개념

- GitHub Actions과 CI/CD 도구
    - GitHub Actions는 GitHub에서 제공하는 자동화 도구로, 코드 변경 이벤트에 따라 워크플로우를 실행할 수 있다.
    - CI(지속적 통합)는 코드를 자주 병합하고 테스트하여 품질을 유지하는 개발 방식이다.
    - CD(지속적 배포)는 테스트가 완료된 코드를 자동으로 배포 환경에 전달해준다.
    - Actions를 활용하면 빌드, 테스트, 배포까지의 과정을 자동화하여 개발 효율성을 높일 수 있다.
- S3와 스토리지
    - Amazon S3는 정적 파일을 저장하고 웹에서 접근할 수 있는 객체 스토리지 서비스이다.
    - 이미지, JS, HTML 등의 정적 자산을 저장하기 적합하며, 버킷 단위로 관리된다.
    - 퍼블릭 접근 정책을 설정하면 외부에서 누구나 파일에 접근할 수 있도록 할 수 있다.
    - 정적 웹사이트 호스팅 기능을 사용하면 S3 버킷을 웹 서버처럼 사용할 수 있다.
- CloudFront와 CDN
    - Amazon CloudFront는 글로벌 CDN(콘텐츠 전송 네트워크)으로 S3와 연결하여 콘텐츠를 빠르게 전달할 수 있다.
    - CDN은 사용자의 지리적 위치에 가까운 엣지 서버에서 콘텐츠를 전달해 지연 시간을 줄인다.
    - 오리진(Origin)으로 S3 정적 사이트 엔드포인트를 지정해 연동할 수 있다.
    - HTTPS 연결, 요청 리다이렉션, 캐싱 전략 등 다양한 최적화 설정을 제공한다.
- 캐시 무효화 (Cache Invalidation)
    - CDN은 성능 향상을 위해 리소스를 일정 시간 동안 캐시해두고 재사용한다.
    - 새 버전의 파일을 배포했을 경우, 기존 캐시를 무효화해야 사용자에게 최신 버전이 보인다.
    - CloudFront에서는 `create-invalidation` 명령어로 캐시된 경로를 무효화할 수 있다.
    - 캐시 무효화는 비용과 속도에 영향을 줄 수 있으므로 효율적인 경로 지정이 중요하다.
- Repository secret과 환경변수
    - 민감한 정보(AWS 키, 배포 ID 등)는 GitHub의 Repository Secrets 기능을 통해 안전하게 저장한다.
    - Secrets는 워크플로우 내에서 `${{ secrets.변수명 }}` 형태로 불러올 수 있다.
    - 환경변수는 배포 환경에 따라 동적으로 설정될 수 있어 코드와 배포의 유연성을 높인다.
    - 민감 데이터의 노출을 막고, 팀 협업 시 안전하게 배포 자동화를 구성할 수 있다.

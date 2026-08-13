# 1-4. 프라이빗 레지스트리 실습 — AWS ECR에서 이미지 pull

## 실습 목표

- 공개 레지스트리(Docker Hub)와 프라이빗 레지스트리(AWS ECR)의 차이를 이해한다
- **내 ECR에 로그인(인증)** 하고, 거기에 올려둔 이미지를 직접 pull 한다
- 프라이빗 레지스트리의 **이미지 주소 형식**을 체험한다

!!! info "이 실습은 1-1에서 만든 VM 안에서 진행합니다"
    모든 명령은 **Ubuntu VM 안**(SSH 접속 상태)에서 실행합니다. Docker는 이미 설치돼 있어야 합니다.

---

## 사전 조건

- 1-1의 VM에서 `docker` 명령이 동작한다
- 강사로부터 **ECR 접속용 키**(Access Key ID / Secret Access Key)를 받았다

??? note "(강사용) 사전 준비 — 학생은 건너뛰세요"
    아래는 강사가 **미리 1회** 해두는 준비입니다. (본인 AWS 계정, 관리자 권한)

    **1) 이미지 채워넣기** — Docker Hub의 멀티아키텍처(amd64+arm64) 이미지를 ECR로 그대로 복사

    ```bash
    export ECR=002029411360.dkr.ecr.ap-northeast-2.amazonaws.com
    aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin $ECR
    for img in nginx httpd redis alpine; do
      docker buildx imagetools create --tag $ECR/skilleat/lab:$img docker.io/library/$img:latest
    done
    ```

    **2) 학생용 읽기 전용 키 발급** (pull만 가능)

    ```bash
    aws iam create-user --user-name ecr-student
    aws iam attach-user-policy --user-name ecr-student \
      --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
    aws iam create-access-key --user-name ecr-student
    ```

    → 출력된 **AccessKeyId / SecretAccessKey**를 수업 때 학생에게 전달합니다.
    수업 후 삭제: `aws iam delete-access-key ...` → `aws iam delete-user --user-name ecr-student`

---

## 1) 레지스트리란?

1-2에서는 `docker run nginx` 처럼 이미지 이름만 써도 자동으로 **Docker Hub**에서 이미지를 가져왔습니다.
하지만 실제 기업에서는 보안·접근 제어를 위해 **프라이빗 레지스트리**를 씁니다.

| 종류 | 예시 | 특징 |
|------|------|------|
| 공개 레지스트리 | Docker Hub (`docker.io`) | 누구나 접근, 공식 이미지 제공 |
| 프라이빗 레지스트리 | **AWS ECR**, Azure ACR, GitHub GHCR | **인증 필요**, 기업 내부 이미지 관리 |

**주소 형식 비교**

```text
# Docker Hub — 레지스트리 주소 생략 가능
nginx:latest
docker.io/library/nginx:latest        ← 실제 전체 주소

# AWS ECR — 주소 명시 필수
002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:nginx
│                                                  │           │
│                                                  │           └── 태그(이미지 구분)
│                                                  └────────────── 리포지토리 경로
└───────────────────────────────────────────────────────────────  레지스트리 주소(계정ID.dkr.ecr.리전.amazonaws.com)
```

!!! tip "핵심"
    프라이빗 레지스트리에서 이미지를 가져오려면 **① 로그인(인증)** 하고 **② 레지스트리 주소를 앞에 붙여** pull 해야 합니다.
    이번 실습은 하나의 리포지토리 `skilleat/lab` 에 **태그로 이미지를 구분**해 담아 두었습니다 (`:nginx`, `:httpd`, `:redis`, `:alpine`).

### ACR과 다른 점 — '리포지토리' 개념이 다릅니다

같은 프라이빗 레지스트리라도 **Azure ACR과 AWS ECR은 리포지토리를 나누는 방식이 다릅니다.**

| 구분 | Azure ACR | AWS ECR (지금 우리) |
|------|-----------|----------------------|
| 레지스트리 주소 | `이름.azurecr.io` | `계정ID.dkr.ecr.리전.amazonaws.com` |
| 리포지토리 | `/` 로 폴더처럼 자유롭게 (`lab/nginx`, `lab/httpd`) | 이름에 `/` 를 써도 **그 전체가 리포지토리 1개** (`skilleat/lab`) |
| 여러 이미지 담기 | 이미지마다 **리포지토리를 따로** | 리포는 **이미지 경로 1개** → 여러 이미지는 **태그로 구분**(또는 리포를 여러 개) |
| push할 때 리포 | **자동 생성**됨 | **미리 만들어 둬야** 함 (`aws ecr create-repository`) |
| 권한·수명주기·스캔 | 주로 레지스트리 단위 | **리포지토리 단위** |

**ACR 방식 — 경로(`/`)로 이미지를 나눔**

```text
skilleatlab.azurecr.io/lab/nginx:latest
skilleatlab.azurecr.io/lab/httpd:latest
     └─ 레지스트리 ─┘ └ 리포지토리(경로) ┘
→ lab/nginx, lab/httpd 는 서로 다른 리포지토리
```

**ECR 방식 — 리포 1개 + 태그로 이미지를 나눔 (지금 우리)**

```text
002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:nginx
002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:httpd
     └────────── 레지스트리 ──────────┘ └ 리포 1개 ┘ └태그┘
→ skilleat/lab 은 리포지토리 1개, nginx/httpd 는 태그
```

!!! warning "가장 헷갈리는 점 — 슬래시(/)의 의미"
    ECR에서 `skilleat/lab` 의 `/` 는 **폴더/네임스페이스가 아니라 그냥 이름의 일부**입니다.
    "`skilleat` 아래에 여러 리포가 있는" 게 아니라, **`skilleat/lab` 이라는 리포지토리 하나**예요.
    (원하면 `skilleat/nginx`, `skilleat/httpd` 처럼 리포를 여러 개 만들 수도 있지만, 각각 **미리 생성**해야 합니다.)

!!! info "그래서 지난번 '이미지 68개'가 이렇게 나온 겁니다"
    리포 하나(`skilleat/lab`)에 태그 4개(`nginx`·`httpd`·`redis`·`alpine`)를 담았고, 각 태그가 멀티아키텍처(여러 CPU) 조각을 품고 있어서 목록이 길어진 것입니다. **태그 붙은 4개가 진짜 이미지**, 나머지 `-` 는 그 안의 CPU별 조각입니다.

### ECR 리포지토리, 잘 쓰는 법 (AWS 공식 권장)

실습은 편의상 리포 1개에 태그로 담았지만, **실무에서 권장되는 방식**은 아래와 같습니다. (AWS ECR 공식 문서 기준)

| 권장 사항 | 내용 | 우리 실습 |
|---|---|---|
| **리포 = 이미지(앱) 1개** | 앱/이미지마다 리포지토리를 따로 (`nginx-web-app`) | 학습용이라 1개에 태그로 담음(예외) |
| **네임스페이스로 그룹화** | `팀/앱` 형태로 카테고리화 (`project-a/nginx-web-app`) | `skilleat/lab` |
| **이름 규칙** | 최대 256자, **영문 소문자로 시작**, 소문자·숫자·`- _ . /` 만, **이중 슬래시(`//`) 불가** | 지킴 |
| **태그 불변성(Immutable)** | `IMMUTABLE` 이면 같은 태그 덮어쓰기 방지 → **재현성·보안**(태그 하이재킹 방지). 프로덕션 권장 | `MUTABLE`(학습용) |
| **의미 있는 태그** | `latest` 대신 버전·날짜·git 해시 | `:nginx` 등 |
| **취약점 스캔** | push 시 이미지 스캔 켜기 (기본/향상) — 레지스트리 수준 설정 권장 | (선택) |
| **수명 주기 정책** | 오래된·미태그 이미지 **자동 정리** → 저장 비용↓ | (아래 예시) |
| **암호화(at rest)** | 기본 **AES-256**, 필요 시 AWS KMS | AES-256 |
| **최소 권한 접근** | 리포지토리 정책·IAM으로 pull/push 권한 분리 | 학생 = **읽기 전용 키** |

**수명 주기 정책 예시 — 미태그 이미지는 1개만 남기고 자동 정리**

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "미태그 이미지는 1개만 남기고 정리",
      "selection": {
        "tagStatus": "untagged",
        "countType": "imageCountMoreThan",
        "countNumber": 1
      },
      "action": { "type": "expire" }
    }
  ]
}
```

!!! warning "멀티아키텍처 이미지엔 주의"
    위 68개처럼 멀티아키텍처 이미지의 **CPU별 조각도 '미태그'로 잡힙니다.** 아직 쓰는 이미지의 조각이 지워지면 이미지가 깨질 수 있으니, 미태그 정리 규칙은 **오래되고 안 쓰는 것 위주**(예: `sinceImagePushed` 며칠 경과)로 신중히 거세요.

!!! tip "정리하면 — 우리 실습 vs 실무"
    - **우리 실습**: 리포 1개 + 태그 + `MUTABLE` + 정책 없음 (단순하게)
    - **실무**: 앱마다 리포 분리 + 네임스페이스 + `IMMUTABLE` 태그 + 스캔 + 수명 주기 정책

    → 참고: [ECR 리포지토리 생성](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-create.html) · [태그 불변성](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html) · [수명 주기 정책 예시](https://docs.aws.amazon.com/AmazonECR/latest/userguide/lifecycle_policy_examples.html)

---

## 2) AWS CLI 설치 (VM 안에서)

먼저 VM의 아키텍처를 확인합니다.

```bash
uname -m        # x86_64 → amd64 탭 / aarch64 → arm64 탭
```

=== "amd64 (x86_64)"

    ```bash
    sudo apt update && sudo apt install -y unzip
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
    unzip -q awscliv2.zip
    sudo ./aws/install
    aws --version
    ```

=== "arm64 (aarch64 · Apple Silicon VM)"

    ```bash
    sudo apt update && sudo apt install -y unzip
    curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o awscliv2.zip
    unzip -q awscliv2.zip
    sudo ./aws/install
    aws --version
    ```

---

## 3) 내 ECR에 로그인

**3-1. 강사가 준 키로 AWS 자격증명 설정**

```bash
aws configure
```

물어보는 값을 아래처럼 입력합니다.

| 항목 | 입력 값 |
|------|--------|
| AWS Access Key ID | `<강사 제공 Access Key ID>` |
| AWS Secret Access Key | `<강사 제공 Secret Access Key>` |
| Default region name | `ap-northeast-2` |
| Default output format | `json` |

제대로 됐는지 확인:

```bash
aws sts get-caller-identity
```

!!! warning "이 키는 pull(읽기) 전용입니다"
    이미지 push·삭제는 안 되는 안전한 키예요. 키 값은 **공유 문서나 깃에 올리지 마세요.**

**3-2. Docker를 ECR에 로그인**

```bash
aws ecr get-login-password --region ap-northeast-2 \
  | docker login --username AWS --password-stdin 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com
```

`Login Succeeded` 가 뜨면 성공입니다.

---

## 4) ECR에서 이미지 pull

```bash
docker pull 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:nginx
docker pull 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:httpd
docker pull 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:redis
docker pull 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:alpine
```

받아진 이미지 목록 확인:

```bash
docker images
```

!!! tip "주소가 길어서 불편하면 (선택)"
    한 번만 변수로 지정해두면 짧게 쓸 수 있어요. (SSH 세션을 닫으면 사라집니다)

    ```bash
    ECR=002029411360.dkr.ecr.ap-northeast-2.amazonaws.com
    docker pull $ECR/skilleat/lab:nginx
    ```

---

## 5) 받은 이미지로 컨테이너 실행

### 5-1. Nginx (웹서버)

```bash
docker run -d --name nginx-web -p 8081:80 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:nginx
```

확인 — VM 안에서 또는 내 PC 브라우저에서:

```bash
curl -I http://localhost:8081                 # VM 안에서 확인
# 내 PC 브라우저:  http://192.168.56.10:8081   (1-1에서 설정한 호스트 전용 IP)
```

### 5-2. Apache (httpd)

```bash
docker run -d --name httpd-web -p 8082:80 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:httpd
curl -I http://localhost:8082
```

### 5-3. Redis

```bash
docker run -d --name redis-db -p 6379:6379 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:redis
docker logs redis-db | head
```

### 5-4. 상태 확인

```bash
docker ps          # 실행 중인 것
docker ps -a       # 전체
```

---

## 6) (선택) 레이어 재사용 체감

이미지는 여러 **레이어(layer)** 가 쌓인 구조라, **이미 받은 레이어는 다시 받지 않습니다.**

### 6-1. 같은 이미지를 다시 pull → 다시 안 받는다

```bash
docker pull python:3.12-slim      # 처음 — 레이어를 내려받음
docker pull python:3.12-slim      # 한 번 더 — 다시 안 받음
```

두 번째는 `Status: Image is up to date` — 이미 있으니 다시 내려받지 않습니다.

### 6-2. 이미지가 '레이어들'로 되어 있는지 보기

```bash
docker history python:3.12-slim
```

여러 줄로 나뉜 게 보이죠? 이 **한 줄 한 줄이 레이어**이고, 캐시·재사용의 단위입니다.

!!! info "서로 다른 이미지도 레이어를 공유하나요?"
    **베이스 레이어(digest)가 완전히 같은** 이미지끼리는 그 레이어를 공유해 한 번만 저장·전송합니다.
    하지만 **버전이 다르면 베이스가 달라 공유가 안 될 수 있어요.**
    예를 들어 `python:3.11-slim` 과 `python:3.12-slim` 은 베이스가 서로 달라, 두 번째 pull에서도 레이어를 **새로 받습니다**(`Already exists` 가 안 뜸). 두 이미지의 레이어 ID가 하나도 안 겹치기 때문이며, 이는 **정상**입니다.

!!! note "최신 Docker면 출력이 다를 수 있어요"
    Docker 최신 버전은 이미지 저장소가 `containerd` 로 바뀌어 `Already exists` 표시가 **아예 안 보일 수** 있습니다.
    `docker info | grep -i "storage driver"` 로 현재 백엔드를 확인할 수 있어요.

---

## 7) 실습 정리 (삭제)

```bash
docker rm -f nginx-web httpd-web redis-db 2>/dev/null || true
docker rmi 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:nginx \
           002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:httpd \
           002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:redis \
           002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/lab:alpine 2>/dev/null || true
```

!!! success "정리하며"
    - 프라이빗 레지스트리(ECR)는 **로그인(인증) 후에만** pull 할 수 있다.
    - 이미지 주소 = `계정ID.dkr.ecr.리전.amazonaws.com/리포지토리:태그`
    - 공개(Docker Hub)와 달리, 기업은 이렇게 **내부 이미지**를 안전하게 관리한다.

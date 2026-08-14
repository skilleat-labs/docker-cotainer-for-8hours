# 1-10. Docker Compose 맛보기

!!! abstract "여러 컨테이너를 파일 하나로, 한 번에"
    지금까지는 `docker run` 으로 컨테이너를 **하나씩** 띄웠죠. 서비스가 여러 개면 명령도 여러 번, 옵션도 길어집니다.
    **Docker Compose** 는 이 구성을 **YAML 파일 하나**로 적어두고 **한 번에** 띄우고 내리는 도구예요.

## 실습 목표

- Compose가 **왜** 필요한지 이해한다
- `compose.yaml` 을 **읽고** 직접 작성한다
- `docker compose up` 한 번으로 여러 컨테이너를 띄우고 내린다

---

## 1) 왜 Compose를 쓰나? (before / after)

nginx 웹서버 + redis 캐시를 띄운다고 해봅시다.

**Before — `docker run` 으로 하나씩** 😵

```bash
docker run -d --name web -p 8080:80 nginx:latest
docker run -d --name cache redis:latest
```

컨테이너가 3개, 5개로 늘면? 옵션까지 길어져 외우기도, 공유하기도 힘듭니다.

**After — `compose.yaml` 파일 하나** 😌

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
  cache:
    image: redis:latest
```

```bash
docker compose up -d      # 이 한 줄로 둘 다 실행
```

→ **구성을 '명령'이 아니라 '파일'로** 남기니, 누가 실행해도 똑같고 git으로 관리도 됩니다.

---

## 2) 준비

```bash
mkdir ~/compose-intro && cd ~/compose-intro
```

## 3) compose.yaml 작성 & 한 줄씩 이해하기

`compose.yaml` 파일을 아래 내용으로 만듭니다.

```yaml
services:          # (1) 띄울 컨테이너들의 묶음
  web:             # (2) 서비스 이름 (컨테이너 별명이 됨)
    image: nginx:latest      # (3) 사용할 이미지
    ports:
      - "8080:80"            # (4) 호스트 8080 → 컨테이너 80
  cache:
    image: redis:latest
```

| 번호 | 뜻 |
|---|---|
| (1) `services` | 여기 아래에 띄울 컨테이너들을 나열 |
| (2) `web` / `cache` | **서비스 이름** — 컨테이너 이름이자, 서로 부를 때의 주소가 됨 |
| (3) `image` | `docker run` 의 이미지와 같음 |
| (4) `ports` | `docker run -p` 와 같음 (`"호스트:컨테이너"`) |

!!! warning "YAML은 '들여쓰기'가 생명"
    - **`Tab` 대신 스페이스(공백)** 로 들여쓰세요.
    - 같은 계층은 **칸 수를 똑같이** 맞춰야 합니다. 어긋나면 `up` 할 때 에러가 납니다.

## 4) 한 번에 띄우기

```bash
docker compose up -d
docker compose ps
```

→ `web` 과 `cache` **두 컨테이너가 한 번에** Up.

!!! tip "`-d` 는 무슨 뜻? (up vs up -d)"
    - `docker compose up` : **앞단(foreground)** 실행 → 로그가 화면에 흐르고, `Ctrl + C` 로 멈춤
    - `docker compose up -d` : **백그라운드**로 실행 → 터미널을 계속 쓸 수 있음

**확인** — 내 PC 브라우저에서 `http://192.168.56.10:8080` → nginx 페이지가 뜨면 성공.

## 5) 안을 들여다보기

**로그 보기**

```bash
docker compose logs          # 전체 로그
docker compose logs web      # web 서비스만
```

**Compose가 자동으로 만들어 준 '네트워크' 확인**

```bash
docker network ls | grep compose-intro
```

!!! info "Compose는 네트워크를 자동으로 만들어 줍니다"
    `up` 하면 이 스택 전용 네트워크가 자동 생성되고, **같은 파일 안의 서비스들은 서로 '이름'으로 통신**할 수 있습니다.
    (예: web 안에서 `cache` 라는 이름으로 redis에 접근 가능) — 이 부분은 **1-11** 에서 직접 확인해요.

## 6) 한 번에 내리기

```bash
docker compose down
```

`down` 한 번이면 컨테이너와 자동 생성된 네트워크까지 **한꺼번에** 정리됩니다.

---

## 자주 쓰는 Compose 명령어

| 명령 | 하는 일 |
|---|---|
| `docker compose up -d` | 스택을 백그라운드로 띄움 |
| `docker compose ps` | 이 스택의 컨테이너 상태 |
| `docker compose logs [서비스]` | 로그 보기 |
| `docker compose stop` / `start` | 멈춤 / 다시 시작 (삭제 X) |
| `docker compose down` | 컨테이너·네트워크 정리 |
| `docker compose down -v` | 볼륨까지 함께 삭제 |

---

## 정리

- **Compose = 여러 컨테이너 구성을 YAML 하나로 선언 → `up` / `down` 한 방.**
- `docker run` 을 여러 번 외우는 대신, **구성을 파일로 남긴다** (재현성·공유·버전관리).
- `up` 하면 **전용 네트워크가 자동 생성**되어 서비스끼리 이름으로 통신할 수 있다. (→ 1-11)

!!! note "`docker compose` (v2)"
    최신 Docker는 띄어쓰기 있는 `docker compose` 를 씁니다. 파일 이름은 `compose.yaml` (또는 `docker-compose.yml`) 둘 다 인식돼요.

# 1-10. Docker Compose 맛보기

!!! abstract "여러 컨테이너를 파일 하나로, 한 번에"
    지금까지는 `docker run` 으로 컨테이너를 **하나씩** 띄웠죠. 서비스가 여러 개면 명령도 여러 번, 옵션도 길어집니다.
    **Docker Compose** 는 이 구성을 **YAML 파일 하나**로 적어두고 **한 번에** 띄우고 내리는 도구예요.

## 실습 목표

- Compose가 무엇인지 이해한다
- `docker compose up` 한 번으로 여러 컨테이너를 띄우고 내린다

---

## 1) 준비

```bash
mkdir ~/compose-intro && cd ~/compose-intro
```

## 2) compose.yaml 작성

`compose.yaml` 파일을 아래 내용으로 만듭니다.

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
  cache:
    image: redis:latest
```

- `services` : 띄울 컨테이너들 (여기선 `web`, `cache`)
- `web` : nginx, 호스트 `8080` → 컨테이너 `80`
- `cache` : redis (그냥 실행)

→ `docker run` 두 번 칠 걸, 파일 하나에 적어둔 것뿐이에요.

## 3) 한 번에 띄우기

```bash
docker compose up -d
docker compose ps
```

→ `web` 과 `cache` **두 컨테이너가 한 번에** Up 됩니다.

**확인** — 내 PC 브라우저에서 `http://192.168.56.10:8080` → nginx 페이지가 뜨면 성공.

## 4) 로그 보고, 한 번에 내리기

```bash
docker compose logs
docker compose down
```

`down` 한 번이면 두 컨테이너가 **한꺼번에** 정리됩니다.

---

## 정리

- **Compose = 여러 컨테이너 구성을 YAML 하나로 선언 → `up` / `down` 한 방.**
- `docker run` 을 여러 번 외우는 대신, **구성을 파일로 남긴다.**

!!! note "`docker compose` (v2)"
    최신 Docker는 띄어쓰기 있는 `docker compose` 를 씁니다. (예전 `docker-compose` 아님)

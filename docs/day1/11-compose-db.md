# 1-11. Compose로 웹 도구 + DB 띄우기

!!! abstract "서로 통신하는 스택을 한 파일로"
    이번엔 서비스가 **서로 연결**되는 스택을 Compose로 띄웁니다. Docker Hub 이미지만 쓰니 빌드도 없고 오류 걱정도 없어요.

## 실습 목표

- 여러 서비스가 **이름으로 통신**하는 걸 확인한다
- `environment` · `volumes` · `depends_on` 을 써본다

## 만들 것

- **db** : PostgreSQL (데이터 저장)
- **adminer** : DB를 웹에서 다루는 GUI 도구 → 브라우저에서 `db` 에 접속

---

## 1) 준비

```bash
mkdir ~/compose-db && cd ~/compose-db
```

## 2) compose.yaml 작성

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - dbdata:/var/lib/postgresql/data
  adminer:
    image: adminer:latest
    ports:
      - "8081:8080"
    depends_on:
      - db

volumes:
  dbdata:
```

- `db` : postgres, 비밀번호·DB이름을 **environment** 로 주입, 데이터는 **`dbdata` 볼륨**에 저장
- `adminer` : DB GUI, 호스트 `8081` → 컨테이너 `8080`, `db` 가 뜬 뒤 시작(**depends_on**)
- 맨 아래 `volumes: dbdata` : named volume 선언

## 3) 띄우기

```bash
docker compose up -d
docker compose ps
```

!!! tip "10초 정도 기다렸다가 접속"
    postgres가 처음 켜질 때 초기화에 몇 초 걸립니다. 잠깐 기다렸다가 접속하세요.

## 4) 브라우저에서 DB 접속

`http://192.168.56.10:8081` (Adminer) 접속 후 아래 값으로 로그인:

| 항목 | 값 |
|---|---|
| 시스템 | PostgreSQL |
| **서버** | **db**　← 컨테이너 '이름'으로 접속! |
| 사용자 | postgres |
| 비밀번호 | pass |
| 데이터베이스 | mydb |

→ 로그인되면, adminer가 **`db` 라는 이름으로** postgres에 접속한 것입니다. (같은 compose 네트워크 + 이름 DNS)

## 5) (선택) 데이터 영속성 확인

Adminer에서 테이블을 하나 만든 뒤:

```bash
docker compose down
docker compose up -d
```

→ 다시 접속하면 그 테이블이 **그대로** 있습니다. (볼륨 덕분에 유지)

---

## 6) 도전과제 대비 — `container_name` 과 `networks`

지금까지는 Compose가 컨테이너 이름도, 네트워크도 **알아서** 만들어 줬어요. 하지만 다음 도전과제에서는 이 둘을 **직접 지정**해야 합니다. 어렵지 않아요! 👇

### ① `container_name` — 컨테이너 이름을 '내가' 고정하기

Compose는 보통 컨테이너 이름을 `폴더명-서비스명-1` 처럼 **자동으로** 짓습니다.
그런데 **다른 서비스가 특정 이름으로 찾아와야** 할 때는, 이름을 직접 고정합니다.

```yaml
services:
  backend:
    image: nginx:latest
    container_name: backend-service    # ← 이 컨테이너 이름을 'backend-service' 로 고정
```

→ 이제 다른 컨테이너가 **`backend-service` 라는 이름으로** 이 컨테이너를 찾을 수 있어요.

### ② `networks` — 네트워크를 '직접' 만들어 붙이기

맨 아래에 `networks:` 로 네트워크를 **선언**하고, 각 서비스에 `networks:` 로 **연결**합니다.

```yaml
services:
  backend:
    image: nginx:latest
    networks:
      - mynet          # ← backend 를 mynet 에 연결
  frontend:
    image: httpd:latest
    networks:
      - mynet          # ← frontend 도 같은 mynet 에 연결

networks:
  mynet:               # ← 'mynet' 이라는 네트워크를 만든다
```

→ 두 서비스가 **같은 `mynet`** 에 있으니 서로 **이름으로 통신**할 수 있어요.
(사실 Compose가 자동으로 만들어 주는 네트워크와 결과는 같아요. 여기선 **이름을 내가 정해서** 만든 것뿐이에요.)

!!! tip "한눈에 정리"
    - **`container_name`** : 컨테이너 이름을 내가 고정 → 다른 서비스가 그 이름으로 찾을 때 사용
    - **`networks`** : 네트워크를 이름 지어 직접 만들고, 서비스들을 거기에 연결
    - 이 둘 + 앞에서 배운 **`volumes`**(영속성) · **`depends_on`**(순서) 이면 **다음 도전과제(미니 블로그)** 를 풀 준비 끝! 💪

---

## 정리 (삭제)

```bash
docker compose down -v      # -v 는 볼륨까지 삭제
```

- 서비스는 **이름으로** 통신한다 (adminer → `db`)
- **environment**(설정 주입) · **volumes**(영속성) · **depends_on**(시작 순서) 를 한 파일에 담았다

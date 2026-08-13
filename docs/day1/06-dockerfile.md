# 1-6. Dockerfile로 나만의 이미지 만들기

!!! abstract "이번엔 이미지를 '내가' 만듭니다"
    지금까지는 남이 만든 이미지를 pull 했지만, 이번엔 **Dockerfile**로 직접 이미지를 빌드합니다.
    **핵심 명령어 → CMD vs ENTRYPOINT → 레이어 캐시** 순으로, 쉬운 것부터 차근차근 가봅시다.

## 실습 목표

- Dockerfile 핵심 명령어로 **내 이미지를 직접 빌드**한다
- **CMD 와 ENTRYPOINT 의 차이**를 실습으로 이해한다
- **레이어 캐시**가 왜 중요한지 빌드 속도로 체감한다

준비:

```bash
mkdir ~/myimage && cd ~/myimage
```

---

## 1) Dockerfile 핵심 명령어

| 명령어 | 하는 일 |
|---|---|
| `FROM` | 베이스 이미지 지정 (항상 첫 줄) |
| `WORKDIR` | 작업 디렉터리 설정 |
| `COPY` | 호스트 파일 → 이미지 안으로 복사 |
| `RUN` | **빌드 시점**에 명령 실행 (패키지 설치 등) |
| `ENV` | 환경변수 설정 |
| `EXPOSE` | 사용하는 포트 문서화 |
| `CMD` / `ENTRYPOINT` | 컨테이너가 **시작될 때** 실행할 명령 |

!!! tip "가장 헷갈리는 시점"
    `RUN` 은 **이미지를 만들 때(build)**, `CMD`/`ENTRYPOINT` 는 **컨테이너를 실행할 때(run)** 동작합니다.

### 첫 이미지 — 제일 간단하게

`Dockerfile` 을 아래 내용으로 작성합니다.

```dockerfile
FROM alpine:latest
CMD ["echo", "Hello from my first image!"]
```

빌드하고 실행:

```bash
docker build -t myfirst:1.0 .
docker run --rm myfirst:1.0
```

→ `Hello from my first image!` 가 출력되면 성공. **내가 만든 첫 이미지**입니다. 🎉

### 조금 더 실제처럼 — 웹 페이지 담기

파일을 준비합니다.

```bash
mkdir site
echo "<h1>Hello Docker</h1>" > site/index.html
```

`Dockerfile` 을 아래로 바꿉니다.

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY site/ ./site/
EXPOSE 8000
CMD ["python", "-m", "http.server", "8000", "--directory", "site"]
```

빌드 & 실행:

```bash
docker build -t myweb:1.0 .
docker run -d --name myweb -p 8000:8000 myweb:1.0
```

확인 — 내 PC 브라우저에서 `http://192.168.56.10:8000` → "Hello Docker" 페이지가 뜨면 성공.

!!! success "방금 이런 걸 써봤어요"
    `FROM`(베이스) · `WORKDIR`(작업 폴더) · `COPY`(파일 넣기) · `EXPOSE`(포트) · `CMD`(실행) 를 한 번에 사용했습니다.

---

## 2) CMD vs ENTRYPOINT — 실습으로 이해하기

둘 다 "컨테이너가 시작될 때 실행할 명령"인데, **`docker run` 뒤에 인자를 줬을 때** 동작이 다릅니다. 직접 3번 만들어 비교해봅시다.

### 실험 A — CMD 만

```dockerfile
FROM alpine:latest
CMD ["echo", "hello"]
```

```bash
docker build -t demo:cmd .
docker run --rm demo:cmd            # → hello
docker run --rm demo:cmd echo bye   # → bye   (CMD가 통째로 교체됨)
```

### 실험 B — ENTRYPOINT 만

```dockerfile
FROM alpine:latest
ENTRYPOINT ["echo"]
```

```bash
docker build -t demo:ep .
docker run --rm demo:ep             # → (빈 줄)
docker run --rm demo:ep bye         # → bye   (bye가 echo 뒤에 '추가'됨)
```

### 실험 C — ENTRYPOINT + CMD (실무 조합)

```dockerfile
FROM alpine:latest
ENTRYPOINT ["echo"]
CMD ["hello"]
```

```bash
docker build -t demo:combo .
docker run --rm demo:combo          # → hello   (CMD가 '기본 인자')
docker run --rm demo:combo bye      # → bye     (인자를 주면 CMD 자리만 교체)
```

### 정리

| Dockerfile | `docker run img` | `docker run img bye` |
|---|---|---|
| **CMD** `["echo","hello"]` | hello | `bye` 를 **명령으로 실행 시도** (오류) |
| **ENTRYPOINT** `["echo"]` | (빈 줄) | bye (echo에 **추가**) |
| **ENTRYPOINT + CMD** | hello | bye |

!!! tip "한 문장 요약"
    - **CMD** = 기본 명령. run 뒤에 인자를 주면 **통째로 교체**.
    - **ENTRYPOINT** = 실행할 프로그램을 **고정**. run 뒤 인자는 **뒤에 추가**.
    - 실무 패턴: **ENTRYPOINT로 프로그램 고정 + CMD로 기본 인자** (실험 C).

---

## 3) 레이어 캐시 — 왜 순서가 중요한가

Dockerfile은 한 줄마다 레이어를 만들고, **바뀌지 않은 레이어는 캐시를 재사용**합니다. 순서만 잘 잡아도 빌드가 훨씬 빨라져요.

준비:

```bash
mkdir ~/cache-demo && cd ~/cache-demo
echo "flask" > requirements.txt
echo 'print("v1")' > app.py
```

### 나쁜 순서 — 소스를 먼저 COPY

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

```bash
docker build -t cache:bad .        # 처음: pip install 실행 (느림)
echo 'print("v2")' > app.py        # 소스 한 줄 수정
docker build -t cache:bad .        # 다시: COPY . . 가 바뀌어 pip install '또' 실행 → 느림
```

### 좋은 순서 — 의존성 먼저, 소스 나중

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

```bash
docker build -t cache:good .       # 처음: pip install 실행
echo 'print("v3")' > app.py        # 소스 한 줄 수정
docker build -t cache:good .       # 다시: pip install은 CACHED → 매우 빠름!
```

!!! success "여기서 봐야 할 것"
    두 번째 빌드 출력에서 **좋은 순서**는 `RUN pip install ...` 줄에 **`CACHED`** 가 뜨고 즉시 끝납니다.
    **나쁜 순서**는 소스만 바꿔도 매번 `pip install` 을 다시 하죠.
    → **자주 바뀌는 것(소스)은 아래쪽에** 두는 게 캐시의 핵심입니다.

---

## 정리

```bash
docker rm -f myweb 2>/dev/null || true
```

- Dockerfile 한 줄 = 레이어 하나. `RUN` = 빌드 시점, `CMD`/`ENTRYPOINT` = 실행 시점.
- **CMD(교체) vs ENTRYPOINT(고정 + 추가)**.
- 자주 바뀌는 것을 아래에 두면 **캐시가 살아** 빌드가 빨라진다.

→ 다음(1-7)에서는 이미지를 **더 작게** 만드는 멀티스테이지 빌드를 배웁니다.

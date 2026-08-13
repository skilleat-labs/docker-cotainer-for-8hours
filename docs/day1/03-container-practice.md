# 1-3. 컨테이너는 '프로세스'다 & 복습 과제

## 실습 목표

- 컨테이너가 **실행되자마자 죽는 이유**를 이해한다
- `docker ps` 와 `docker ps -a` 의 차이를 몸으로 안다
- 컨테이너를 **계속 살아있게** 만드는 방법을 안다

---

## 1) 컨테이너는 결국 '프로세스' 하나다

컨테이너는 특별한 가상 컴퓨터가 아니라, **격리된 프로세스 하나**입니다.
프로세스는 **할 일이 끝나면 종료**되죠. 컨테이너도 똑같습니다 —
**안에서 실행한 명령(프로세스)이 끝나면, 컨테이너도 함께 죽습니다.**

### 실습 ① 바로 죽는 컨테이너

```bash
docker run --name bye alpine echo "hello"
```

`echo` 는 한 줄 출력하고 **즉시 끝나는** 명령입니다. 그래서 컨테이너도 곧바로 종료됩니다.

```bash
docker ps        # 아무것도 안 보임 — 실행 중인 컨테이너가 없다
docker ps -a     # bye 가 'Exited' 상태로 보인다
```

!!! info "docker ps vs docker ps -a"
    - `docker ps` : **지금 살아서 실행 중**인 컨테이너만 보여줌
    - `docker ps -a` : **종료된(Exited) 것까지 전부** 보여줌

    → `bye` 는 이미 죽었으므로 `ps` 엔 안 나오고, `ps -a` 에만 나옵니다.

### 실습 ② 왜 죽었는지 확인

```bash
docker logs bye     # hello 가 출력됨 (할 일을 다 하고 끝난 것)
docker ps -a        # STATUS 열: Exited (0)  →  0 = 정상 종료
```

!!! tip "핵심"
    컨테이너가 죽은 건 **에러가 아니라**, 안의 프로세스(`echo`)가 **할 일을 마쳐서** 입니다.
    **"할 일이 없으면 죽는다"** — 이게 컨테이너의 본질입니다.

### 실습 ③ 살아있게 만들기 — 할 일을 계속 주기

**방법 A. 재우기(sleep) — 300초 동안 안 죽음**

```bash
docker run -d --name sleeper alpine sleep 300
docker ps            # sleeper 가 실행 중으로 보인다
```

**방법 B. 계속 도는 서버 프로세스(nginx)**

```bash
docker run -d --name web nginx
docker ps            # nginx 는 웹서버라 계속 떠 있다
```

!!! info "차이가 보이나요?"
    `echo` 는 할 일이 끝나 죽었고, `sleep`·`nginx` 는 **계속 할 일이 있어서** 안 죽습니다.
    `-d` 는 백그라운드 실행 옵션일 뿐이고, **죽고 사는 것은 '안의 프로세스가 계속 도는지'** 로 결정됩니다.

### 실습 ④ stop / start 로 껐다 켜기

```bash
docker stop sleeper   # 프로세스를 멈춤 → Exited
docker ps             # 안 보임
docker ps -a          # Exited 상태로 남아 있음
docker start sleeper  # 다시 실행
docker ps             # 다시 실행 중
```

### 실습 ⑤ "그냥 계속 떠 있게" 하는 무한 유지 패턴

```bash
docker run -d --name forever alpine sh -c "while true; do sleep 60; done"
docker ps
```

!!! note "이건 왜 안 죽나요?"
    `while true` 로 **끝나지 않는 할 일**을 만들었기 때문입니다. 프로세스가 계속 도니 컨테이너도 안 죽죠.
    실무에서 "특별한 서버는 없지만 컨테이너를 계속 띄워두고 싶을 때" 쓰는 패턴입니다.

### 정리

| 명령 | 뜻 |
|---|---|
| `docker ps` | 실행 중(살아있는) 컨테이너만 |
| `docker ps -a` | 종료된 것까지 전부 |

- 컨테이너 = **격리된 프로세스**. 안의 명령이 끝나면 컨테이너도 끝난다.
- 계속 떠 있으려면 **계속 실행되는 프로세스**(서버 · `sleep` · 무한 루프)가 있어야 한다.

**실습 정리(삭제):**

```bash
docker rm -f bye sleeper web forever
```

---

## 2) 복습 과제

앞의 내용을 바탕으로, 아래 과제를 스스로 해결해 보세요.

### 문제 1. 컨테이너 실행

다음 조건에 맞춰 컨테이너를 실행하세요.

| 항목 | 값 |
|------|-----|
| 이미지 | `docker/getting-started` |
| 태그 | 지정하지 않음 |
| 컨테이너 이름 | `guide` |
| 컨테이너 서비스 포트 | `80` |
| 호스트 포트 | `1234` |
| 실행 방식 | 백그라운드 |

실행 후 본인 VM의 `192.168.x.x:1234`로 브라우저에서 접속해 정상적으로 페이지가 뜨는지 확인하세요.

### 문제 2. 컨테이너 내부 파일 확인

실행 중인 `guide` 컨테이너 내부에 접속해서 `/usr/share/nginx/html/50x.html` 파일을 열고, `<title>` 태그 안에 들어간 문구가 무엇인지 찾아내세요.

!!! tip "힌트"
    이 컨테이너에는 `bash`가 없으므로 `sh`로 접속하세요.

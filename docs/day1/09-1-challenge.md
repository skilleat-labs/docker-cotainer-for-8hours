# 1-9-1. [도전과제] 미니 블로그 (docker 명령으로 직접)

!!! abstract "Compose 없이, 배운 명령만으로 두 컨테이너를 엮으세요"
    1-8(볼륨) · 1-9(네트워크)에서 배운 걸로, **frontend + backend** 두 컨테이너를 `docker` 명령만으로 직접 연결해 미니 블로그를 완성합니다.
    (다음에 이걸 Compose 파일 하나로 줄이는 걸 배웁니다!)

## 1) 사용할 이미지

| 서비스 | 이미지 |
|---|---|
| frontend | `skilleat/frontend:v4-kb5` |
| backend | `skilleat/backend:v4-kb5` |

## 2) 요구사항 (docker 명령으로 구성)

**① 전용 네트워크**

- 두 컨테이너가 통신할 **사용자 정의 네트워크**를 만들고, 둘 다 그 네트워크에서 실행하세요. (이름은 자유)

**② backend (먼저 실행)**

- 컨테이너 이름은 반드시 **`backend-service`** (frontend 가 이 이름으로 찾습니다)
- 내부 포트는 **`5000`**
- 전용 **볼륨**을 붙여 **`/app`** 에 마운트 → `data.json` 이 컨테이너를 삭제해도 유지되어야 함
- **frontend 보다 먼저** 실행

**③ frontend**

- backend 와 **같은 네트워크**
- 외부 **`8080`** → 컨테이너 내부 **`80`** 포트 매핑
- backend 가 먼저 실행된 뒤 실행

## 3) 🎯 어떤 명령/옵션이 필요할까?

| 하고 싶은 것 | 쓸 명령·옵션 |
|---|---|
| 전용 네트워크 만들기 | `docker network create ...` |
| 컨테이너를 그 네트워크에 붙이기 | `docker run --network ...` |
| 볼륨으로 영구 저장 | `docker volume create ...` + `-v 볼륨:/app` |
| 컨테이너 이름 지정 | `--name backend-service` |
| 포트 매핑 | `-p 8080:80` |

> 순서 힌트: **네트워크 → (볼륨) → backend 실행 → frontend 실행**

---

## 4) 웹 접속 및 데이터 확인

- 브라우저에서 **`http://192.168.56.10:8080`** (또는 VM 안에서 `localhost:8080`) 접속 → 블로그 페이지가 떠야 합니다.
- 새 글을 작성하면 화면에 표시되고, backend 의 `data.json` 에 저장되어야 합니다.

## 5) 추가 검증

**a. backend 안에서 `data.json` 확인**

```bash
docker exec -it backend-service sh
cat data.json
```

**b. 두 컨테이너가 같은 네트워크인지 점검**

```bash
docker network inspect <네트워크이름>
```

**c. backend 컨테이너를 삭제 후 다시 만들어도 `data.json` 이 유지되는지 확인**
(같은 볼륨을 다시 붙여서 실행 → 글이 그대로 남아야 함)

---

## ✅ 통과 기준

- [ ] `http://192.168.56.10:8080` 에서 블로그 페이지가 뜬다
- [ ] 새 글을 작성하면 화면에 보이고 `data.json` 에 저장된다
- [ ] backend 를 삭제 후 같은 볼륨으로 다시 실행해도 글이 그대로다
- [ ] `docker network inspect` 로 frontend·backend 가 같은 네트워크임을 확인했다

!!! tip "다음 예고"
    지금 명령 4~5개로 한 걸, 곧 **`compose.yaml` 파일 하나**로 똑같이 만들어 봅니다. (1-10 ~ 1-12)

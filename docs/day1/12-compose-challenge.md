# 1-12. [도전과제] 미니 블로그 완성하기

!!! abstract "Compose로 frontend + backend 를 엮어 미니 블로그를 완성하세요"
    요구사항만 주어집니다. **`compose.yaml` 을 직접 완성**해서, 새 글이 저장되고 컨테이너를 지워도 데이터가 남게 만드세요.

## 1) 사용할 이미지

| 서비스 | 이미지 |
|---|---|
| frontend | `skilleat/frontend:v4-kb5` |
| backend | `skilleat/backend:v4-kb5` |

## 2) 사용자 정의 네트워크

- 컨테이너들이 통신할 **전용 네트워크**를 만들어 실행하세요. (이름은 자유)
- **frontend 와 backend 는 반드시 같은 네트워크** 안에서 실행되어야 합니다.

## 3) backend 서비스 (먼저 정의)

- 컨테이너 이름은 반드시 **`backend-service`** 여야 합니다.
- 내부 포트는 **`5000`** 을 사용합니다.
- backend 가 frontend 보다 **먼저** 실행되어야 합니다.
- backend 의 데이터(`data.json`)는 **컨테이너를 삭제해도 유지**되어야 합니다.
- 전용 **볼륨(volume)** 을 붙여 영구 저장되게 하세요. (컨테이너 마운트 경로 **`/app/`**)

## 4) frontend 서비스

- backend 와 **동일한 네트워크** 안에서 실행됩니다.
- 외부 접속 포트는 **`8080`**, 컨테이너 내부 포트는 **`80`** 입니다.
- frontend 는 backend 가 먼저 실행된 후 정상 동작합니다. → **`depends_on`** 활용

## 5) 🎯 compose.yaml 을 완성하세요

아래 조건을 **모두** 만족하는 `compose.yaml` 을 작성합니다.

- 두 개의 서비스(frontend, backend)가 모두 포함
- backend 는 `5000` 포트 사용 / frontend 는 `8080:80` 포트 매핑
- 두 서비스 모두 **동일한 네트워크** 사용
- backend 는 `container_name: backend-service`
- backend 는 전용 볼륨으로 `data.json` 을 영구 저장 (마운트 경로 `/app/`)
- frontend 는 backend 가 먼저 실행되어야 정상 동작 (`depends_on`)

!!! note "container_name 은 아래처럼 별도 항목으로 반드시 지정"
    ```yaml
    services:
      backend:
        image: skilleat/backend:v4-kb5
        container_name: backend-service
    ```

---

## 6) 웹 접속 및 데이터 확인

- 브라우저에서 **`http://192.168.56.10:8080`** (또는 VM 안에서 `localhost:8080`) 접속 시 페이지가 정상 표시되어야 합니다.
- 새 글을 작성하면 화면에 표시되고, backend 내부 JSON 파일(`data.json`)에 저장되어야 합니다.

## 7) 추가 검증

**a. backend 컨테이너 안으로 들어가 `data.json` 확인**

```bash
docker exec -it backend-service sh
cat data.json
```

**b. frontend 와 backend 가 같은 네트워크에 속해 있는지 점검**

```bash
docker network inspect <네트워크이름>
```

**c. backend 컨테이너를 삭제 후 다시 만들어도 `data.json` 이 유지되는지 확인**

---

## ✅ 통과 기준

- [ ] `http://192.168.56.10:8080` 에서 블로그 페이지가 뜬다
- [ ] 새 글을 작성하면 화면에 보이고 `data.json` 에 저장된다
- [ ] `docker exec` 로 backend 안에서 `data.json` 내용을 확인할 수 있다
- [ ] backend 컨테이너를 삭제 후 다시 만들어도 글이 그대로 남아 있다

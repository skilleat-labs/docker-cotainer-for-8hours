# 1-6-1. [도전과제] 나만의 앱 이미지 만들기

!!! abstract "1-6에서 배운 걸 '스스로' 해봅니다"
    제공된 스타터 앱을 가져와 **내 것으로 바꾸고**, **Dockerfile을 직접 작성**해 이미지로 만들어 실행합니다.
    정답 Dockerfile은 맨 아래 접이식에 있으니, **먼저 스스로** 도전해 보세요!

## 목표

- 스타터 코드를 `git clone` 해서 내 앱으로 커스터마이즈한다
- **Dockerfile을 직접 작성**해 이미지를 빌드한다
- 실행 후 브라우저에서 내 페이지를 확인한다
- (보너스) 내 이미지를 ECR(`skilleat/student`)에 이니셜 태그로 push 한다

---

## 1) 스타터 코드 가져오기 (VM에서)

```bash
git clone https://github.com/skilleat-labs/docker-challenge.git
cd docker-challenge
ls
```

`app.py` · `requirements.txt` · `README.md` 가 보입니다. **Dockerfile은 없습니다 — 여러분이 직접 만들 거예요.**

---

## 2) 내 앱으로 커스터마이즈

`app.py` 를 열어 `NAME` 을 본인 이름으로 바꿉니다.

```bash
nano app.py
```

```python
NAME = "여기에_본인_이름"     # ← 본인 이름으로 수정 (예: "홍길동")
```

저장하고 나오기: `Ctrl` + `O` → `Enter` → `Ctrl` + `X`

---

## 3) 🎯 [도전] Dockerfile을 직접 작성

`docker-challenge` 폴더 안에 `Dockerfile` 을 만들어, 아래 조건을 만족시키세요.

| 요구사항 | 힌트 |
|---|---|
| 베이스 이미지는 `python:3.12-slim` | `FROM` |
| 작업 디렉터리를 `/app` 으로 | `WORKDIR` |
| 의존성을 **먼저** 복사·설치 (캐시가 살도록) | `COPY requirements.txt .` → `RUN pip install -r requirements.txt` |
| 나머지 소스 복사 | `COPY . .` |
| 앱이 쓰는 포트는 **5000** | `EXPOSE` |
| 컨테이너 시작 시 `python app.py` 실행 | `CMD` |

!!! tip "생각해볼 것"
    왜 `requirements.txt` 를 **먼저** 복사하고 설치할까요? (1-6의 레이어 캐시!)

---

## 4) 🎯 [도전] 직접 빌드하고 실행하기

아래 조건에 맞춰 **스스로** 이미지를 빌드하고 컨테이너를 실행하세요. (명령은 직접 작성!)

| 항목 | 값 |
|---|---|
| 이미지 이름 | `myapp` |
| 태그 | `1.0` |
| 컨테이너 이름 | `myapp` |
| 앱(컨테이너) 포트 | `5000` |
| 호스트 포트 | `5000` |
| 실행 방식 | 백그라운드 |

**확인** — 내 PC 브라우저에서:

```text
http://192.168.56.10:5000
```

→ **"Hello, 본인이름! 🐳"** 페이지가 뜨면 성공! 🎉

---

## 5) (보너스) 내 이미지를 ECR에 push

1-4 / 1-5에서 배운 대로, 내 이미지에 **이니셜 태그**를 달아 올려봅니다.

**① 먼저 ECR에 로그인** (로그인 토큰은 12시간이면 만료되니, 안 되면 다시 실행)

```bash
aws ecr get-login-password --region ap-northeast-2 \
  | docker login --username AWS --password-stdin 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com
```

`Login Succeeded` 가 뜨면 됩니다.

**② 태그 달고 push**

```bash
docker tag myapp:1.0 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/student:이니셜
docker push 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/student:이니셜
```

---

## ✅ 통과 기준

- [ ] 브라우저에 **본인 이름**이 표시된다
- [ ] `docker ps` 에 `myapp` 컨테이너가 **Up** 상태다
- [ ] (보너스) `skilleat/student` 리포에 **내 이니셜 태그**가 올라갔다

---

## 정리 (다음 실습 전에)

```bash
docker rm -f myapp
```

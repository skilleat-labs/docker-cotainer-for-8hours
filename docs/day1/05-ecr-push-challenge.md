# 1-5. [과제] 내 이미지를 ECR에 push 하기

!!! abstract "이번엔 직접 올려봅니다 (pull ↔ push)"
    1-4에서는 이미지를 **내려받기(pull)** 만 했죠. 이번 과제는 반대로, 이미지를 **올려보기(push)** 입니다.
    로그인은 1-4에서 이미 해뒀으니 권한은 있습니다. **명령을 스스로 조합**해서 완성해 보세요!

??? note "(강사용) 사전 준비 — 학생은 건너뛰세요"
    학생용 리포지토리를 만들고, 학생 키가 **이 리포에만 push** 되도록 권한을 스코프합니다.

    ```bash
    # 1) 학생용 리포 생성
    aws ecr create-repository --repository-name skilleat/student --region ap-northeast-2

    # 2) 학생 키(ecr-student)에 'student 리포에만 push' 인라인 정책 부여
    aws iam put-user-policy --user-name ecr-student --policy-name PushToStudentRepoOnly --policy-document '{
      "Version": "2012-10-17",
      "Statement": [{
        "Effect": "Allow",
        "Action": ["ecr:InitiateLayerUpload","ecr:UploadLayerPart","ecr:CompleteLayerUpload","ecr:PutImage","ecr:BatchCheckLayerAvailability"],
        "Resource": "arn:aws:ecr:ap-northeast-2:002029411360:repository/skilleat/student"
      }]
    }'
    ```

    → 학생은 어디서나 pull + `skilleat/student`에만 push. 선생님의 `skilleat/lab`은 못 건드립니다.

---

## 과제

Docker Hub에서 `alpine:latest` 를 가져와, 아래 정보로 **강사 ECR에 push** 하세요.

| 항목 | 값 |
|------|-----|
| 원본 이미지 | `alpine:latest`  (Docker Hub에서 pull) |
| 레지스트리 주소 | `002029411360.dkr.ecr.ap-northeast-2.amazonaws.com` |
| 리포지토리 | `skilleat/student` |
| 태그 | **본인 이니셜** (예: `nrk`) |

**해야 할 일**

1. Docker Hub에서 `alpine:latest` 를 pull 한다
2. 위 세 값(주소 · 리포지토리 · 태그)을 **조합해서** ECR 주소로 태깅한다
3. push 한다

!!! tip "힌트 — 이미지 주소 형식 (1-4에서 배운 것)"
    ```text
    <레지스트리 주소> / <리포지토리> : <태그>
    ```
    위 표의 세 값을 이 형식에 끼워 맞추면 내 이미지 주소가 완성됩니다.

---

## 확인

내 이니셜 태그가 올라갔는지 확인하세요.

```bash
aws ecr list-images --repository-name skilleat/student --region ap-northeast-2
```

또는 ECR 콘솔의 `skilleat/student` 리포지토리에서 **본인 이니셜 태그**가 보이면 성공입니다.

!!! warning "push할 때 인증 오류(`denied` / `not authorized`)가 나면"
    ECR 로그인 토큰은 **12시간**이면 만료됩니다. 아래로 다시 로그인한 뒤 push 하세요.

    ```bash
    aws ecr get-login-password --region ap-northeast-2 \
      | docker login --username AWS --password-stdin 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com
    ```

??? success "막히면 열어보기 — 정답 (본인 이니셜을 nrk 라고 할 때)"
    ```bash
    docker pull alpine:latest
    docker tag  alpine:latest 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/student:nrk
    docker push 002029411360.dkr.ecr.ap-northeast-2.amazonaws.com/skilleat/student:nrk
    ```

!!! note "이니셜이 겹치면?"
    같은 이니셜이면 태그가 덮어써집니다. 겹치면 `nrk2` 처럼 숫자를 붙이세요.

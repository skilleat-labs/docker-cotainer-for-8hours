# 1-8. Docker 볼륨 — 데이터를 지키기

!!! abstract "컨테이너는 일회용, 데이터는 영구"
    컨테이너는 언제든 버리고 새로 만드는 것입니다. 그런데 컨테이너를 지우면 **안의 데이터도 사라져요.**
    이번 실습은 **볼륨(Volume)** 으로 데이터를 컨테이너 밖에 저장해, 지워도 살아남게 만드는 실험입니다.

## 실습 목표

- 컨테이너를 지우면 데이터가 사라지는 것을 **직접 확인**한다
- **볼륨**으로 데이터가 유지되는 것을 실험으로 체감한다
- Named Volume 과 Bind Mount 를 구분한다

---

## 1) 문제 체감 — 볼륨 없이 데이터를 넣으면

```bash
docker run -it --name novol alpine sh
```

컨테이너 안에서 파일을 만들고 나옵니다.

```sh
echo "hello" > /data.txt
cat /data.txt        # → hello
exit
```

컨테이너를 지우고 **다시 만들어** 확인합니다.

```bash
docker rm novol
docker run -it --name novol alpine sh
```

```sh
cat /data.txt        # → No such file  (사라짐!)
exit
```

```bash
docker rm novol
```

!!! danger "확인"
    컨테이너를 지웠다 다시 만드니 데이터가 **사라졌습니다.** DB라면 큰일이죠.

---

## 2) 해결 — Named Volume 으로 데이터 지키기

```bash
docker volume create mydata
docker run -it --name app -v mydata:/data alpine sh
```

볼륨이 연결된 `/data` 에 파일을 저장합니다.

```sh
echo "keep me" > /data/file.txt
cat /data/file.txt   # → keep me
exit
```

컨테이너를 **지우고**, 같은 볼륨으로 **새 컨테이너**를 만듭니다.

```bash
docker rm app
docker run -it --name app2 -v mydata:/data alpine sh
```

```sh
cat /data/file.txt   # → keep me  (살아있음!)
exit
```

```bash
docker rm app2
```

!!! success "확인"
    컨테이너를 새로 만들었는데도 데이터가 **그대로**입니다. 볼륨은 컨테이너 수명과 **분리**되어 있으니까요.

---

## 3) 실전 실험 — DB 데이터 영속성

진짜 DB로도 확인해봅시다. (재배포해도 데이터가 남는지)

```bash
docker volume create pgdata
docker run -d --name db -e POSTGRES_PASSWORD=pass -v pgdata:/var/lib/postgresql/data postgres:16
```

데이터를 한 건 넣습니다.

```bash
docker exec db psql -U postgres -c "CREATE TABLE t(v text); INSERT INTO t VALUES ('hello volume');"
docker exec db psql -U postgres -c "SELECT * FROM t;"     # → hello volume
```

컨테이너를 **강제 삭제**하고, 같은 볼륨으로 **다시** 띄웁니다.

```bash
docker rm -f db
docker run -d --name db -e POSTGRES_PASSWORD=pass -v pgdata:/var/lib/postgresql/data postgres:16
docker exec db psql -U postgres -c "SELECT * FROM t;"     # → hello volume  (유지!)
```

!!! success "재배포에도 데이터가 유지됩니다"
    컨테이너를 통째로 지우고 새로 띄웠는데도 방금 넣은 데이터가 그대로죠. **상태(데이터)와 컨테이너를 분리**한 덕분입니다.

---

## 4) Bind Mount — 호스트 폴더를 직접 연결

볼륨 말고, **내 호스트(VM)의 폴더**를 컨테이너에 직접 연결할 수도 있습니다.

```bash
mkdir ~/hostdir && echo "from host" > ~/hostdir/note.txt
docker run --rm -v ~/hostdir:/mnt alpine cat /mnt/note.txt   # → from host
```

호스트에 있는 파일이 컨테이너 안 `/mnt` 에서 그대로 보입니다. (개발 중 소스 실시간 반영에 유용)

---

## 5) 정리 — Named Volume vs Bind Mount

| | Named Volume | Bind Mount |
|---|---|---|
| 저장 위치 | **Docker가 관리**하는 공간 | 호스트의 **특정 폴더** |
| 지정 방법 | `-v 이름:/컨테이너경로` | `-v /호스트경로:/컨테이너경로` |
| 주로 언제 | **운영 데이터**(DB 등) | 개발 중 **소스 실시간 반영** |

**실습 정리(삭제):**

```bash
docker rm -f db 2>/dev/null || true
docker volume rm mydata pgdata 2>/dev/null || true
```

- 컨테이너는 일회용, **데이터는 볼륨으로 밖에** 저장한다.
- Named Volume(운영 데이터) vs Bind Mount(개발 편의) 를 상황에 맞게.

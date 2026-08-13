# 1-0. 사전 준비작업 (수업 전에 미리 해두세요)

!!! abstract "이 페이지는 '수업 전 설치' 안내입니다"
    수업 시작 전에 **VirtualBox 설치**와 **Ubuntu ISO 다운로드(24.04 또는 26.04 LTS)**까지만 미리 해두시면 됩니다.
    **VM 생성부터는 수업에서 함께** 진행하니, 여기서는 설치·다운로드까지만 하시면 충분합니다.

!!! tip "왜 미리 하나요?"
    VirtualBox 설치(특히 Windows의 Hyper-V 재부팅)와 Ubuntu ISO 다운로드는 **시간이 꽤 걸립니다.**
    수업 시간을 실습에 쓰기 위해, 다운로드·설치는 미리 끝내두는 걸 권장해요.

---

## 준비물

- Windows / macOS / Linux PC
- 인터넷 연결 (ISO 다운로드로 수 GB 트래픽 발생)
- 관리자 권한
- 저장 공간 최소 **15GB 이상** 여유

## 공식 다운로드 링크

- [VirtualBox 공식 다운로드](https://www.virtualbox.org/wiki/Downloads)
- Ubuntu Server ISO → 아래 **2단계에서 아키텍처별(amd64 / arm64) 직접 링크**로 안내합니다
- [UTM (Apple Silicon 대체용)](https://mac.getutm.app/)
- [Docker Engine (Ubuntu) 설치 문서](https://docs.docker.com/engine/install/ubuntu/) — (참고용, 설치는 수업에서)

---

## 1) VirtualBox 설치

=== "Windows"

    !!! warning "설치 전 확인 — Hyper-V 끄기"
        Windows에서 **Hyper-V가 활성화**되어 있으면 VirtualBox가 정상 실행되지 않을 수 있습니다.
        설치 전 **PowerShell을 관리자 권한**으로 열고 아래를 실행하세요.

        ```powershell
        bcdedit /set hypervisorlaunchtype off
        ```

        실행 후 **반드시 재부팅**합니다. (VirtualBox 설치는 재부팅 후 진행)

    1. [VirtualBox 공식 다운로드 페이지](https://www.virtualbox.org/wiki/Downloads)에서 **Windows hosts** 버전을 받습니다.
    2. 기본 옵션으로 설치합니다.
    3. 설치 완료 후 VirtualBox를 실행합니다.

    !!! tip "그래도 안 되면"
        - 재부팅 후에도 실행이 안 되면 BIOS/UEFI에서 **가상화(VT-x/AMD-V)** 가 켜져 있는지 확인하세요.
        - WSL2 · Docker Desktop을 쓰고 있었다면 Hyper-V가 다시 켜질 수 있습니다. 위 `bcdedit` 명령을 다시 적용하세요.

=== "macOS (Intel)"

    1. [VirtualBox 공식 다운로드 페이지](https://www.virtualbox.org/wiki/Downloads)에서 **macOS / Intel hosts** 버전을 받습니다.
    2. `.dmg`를 열어 설치합니다.
    3. 설치 중 **"시스템 확장 차단됨"** 알림이 뜨면 `시스템 설정 → 개인정보 보호 및 보안` 에서 **허용**을 누르고, 안내에 따라 재시도합니다.

=== "macOS (Apple Silicon · M1/M2/M3)"

    !!! danger "먼저 읽어주세요 — Apple Silicon 주의"
        Apple Silicon 맥은 CPU가 **ARM**이라, x86용 VirtualBox VM이 **잘 안 되거나 매우 느릴 수** 있습니다.
        (VirtualBox 7.1+에 Apple Silicon 지원이 들어왔지만 아직 **실험적**입니다.)

    **먼저 시도:** [VirtualBox 다운로드](https://www.virtualbox.org/wiki/Downloads)에서 **macOS / Arm64** 빌드를 받아 설치해 봅니다.

    **잘 안 되면 (권장 대체 경로): UTM 사용**

    1. [UTM](https://mac.getutm.app/)을 설치합니다. (App Store 또는 공식 사이트)
    2. Ubuntu는 **ARM64용 ISO**(`...-live-server-arm64.iso`)를 받습니다. (아래 2단계 참고)

    !!! success "안심하세요"
        하이퍼바이저(VirtualBox / UTM)만 다를 뿐, 수업에서 하는 작업은 **완전히 동일**합니다.
        본인이 Apple Silicon이라 UTM으로 준비했다면, 수업 시작 때 미리 알려주세요.

---

## 2) Ubuntu Server 이미지 준비 (24.04 또는 26.04 LTS)

!!! info "버전은 24.04 / 26.04 아무거나 OK — 최신 26.04 받아도 됩니다"
    이 실습은 **24.04 LTS** 와 **26.04 LTS(최신)** 에서 동일하게 동작합니다.
    기본으로 나오는 **최신 26.04 LTS(Resolute Raccoon)를 그대로 받으셔도 됩니다.**
    (Docker 저장소도 26.04를 지원해서 이후 설치 명령이 수정 없이 그대로 동작합니다.)

!!! warning "가장 중요 — 내 CPU에 맞는 아키텍처(amd64 / arm64)를 받으세요"
    ISO는 **CPU 종류에 맞는 것**을 받아야 부팅됩니다. 안 맞으면 설치가 아예 안 돼요. (특히 Mac 주의)

    - **Windows · Intel Mac → `amd64`**
    - **Apple Silicon Mac (M1·M2·M3·M4) → `arm64`**

    **내 맥 확인:** 왼쪽 위 사과 로고 → *이 Mac에 관하여* → **"Apple M..."** 이면 arm64, **"Intel..."** 이면 amd64

### 아키텍처별 다운로드 링크 (여기서 바로 받으세요)

=== "Windows · Intel Mac (amd64)"

    | 버전 | 다운로드 페이지 | 받을 파일 |
    |---|---|---|
    | **26.04 LTS** (권장·최신) | [releases.ubuntu.com/26.04](https://releases.ubuntu.com/26.04/) | `ubuntu-26.04-live-server-amd64.iso` |
    | 24.04 LTS | [releases.ubuntu.com/24.04](https://releases.ubuntu.com/24.04/) | `ubuntu-24.04.x-live-server-amd64.iso` |

=== "Apple Silicon Mac (arm64)"

    !!! danger "Apple Silicon은 VirtualBox 대신 UTM 권장"
        arm64 + VirtualBox는 아직 실험적이라 불안정할 수 있어요. **UTM**으로 진행하세요.
        (VM 안에서 하는 작업·명령은 완전히 동일합니다.)

    | 버전 | 다운로드 페이지 | 받을 파일 |
    |---|---|---|
    | **26.04 LTS** (권장·최신) | [cdimage.ubuntu.com/releases/26.04](https://cdimage.ubuntu.com/releases/26.04/release/) | `ubuntu-26.04-live-server-arm64.iso` |
    | 24.04 LTS | [cdimage.ubuntu.com/releases/24.04](https://cdimage.ubuntu.com/releases/24.04/release/) | `ubuntu-24.04.x-live-server-arm64.iso` |

!!! note "ISO 파일은 어디에 두나요?"
    다운로드 폴더에 그대로 두셔도 됩니다. 수업에서 VM 만들 때 이 파일을 선택하니 **삭제하지 말고 위치만 기억**해 두세요.

---

## 사전 준비 완료 체크리스트

수업 시작 전 아래가 모두 되어 있으면 완벽합니다.

- [ ] VirtualBox(또는 Apple Silicon은 UTM) 설치 완료 — 실행하면 창이 뜬다
- [ ] Ubuntu Server ISO 다운로드 완료 — **24.04 또는 26.04**, 그리고 **내 CPU에 맞는 `amd64`/`arm64`**
- [ ] (Windows) Hyper-V 끄고 재부팅까지 완료
- [ ] 저장 공간 15GB 이상 여유 확인

!!! question "잘 안 되는 게 있으면"
    수업 때 해결해 드리니 **너무 붙잡고 있지 마세요.** 어느 단계에서 어떤 메시지가 떴는지,
    그리고 본인 환경(**Windows / Intel Mac / Apple Silicon Mac**)을 함께 알려주시면 빠르게 도와드립니다.

---

!!! success "여기까지 하셨으면 준비 끝!"
    다음 **[1-1. VM 직접 체험](01-vm-virtualbox.md)** 은 수업에서 VM 생성부터 함께 진행합니다.

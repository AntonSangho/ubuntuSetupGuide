# Ubuntu 22.04 Setup Guide (HP Victus 16)

## 1. 한글 입력기 설치

### 언어 지원 설치
```bash
gnome-language-selector
```
또는 **Settings → Region & Language → Manage Installed Languages**

1. **Install / Remove Languages** 클릭
2. **Korean (한국어)** 체크
3. **Apply** 클릭

### 한글 입력기 설치 (IBus)
```bash
sudo apt update
sudo apt install ibus-hangul
```

### 입력 소스 추가
1. 로그아웃 후 다시 로그인
2. **Settings → Keyboard → Input Sources**
3. **+** 클릭 → **⋮** → **Other**
4. **Korean (Hangul)** 추가

### 한/영 전환
- 기본 단축키: `Super + Space` 또는 `Shift + Space`

---

## 2. 추가 드라이브 마운트 (NTFS)

### 드라이브 확인
```bash
lsblk
sudo blkid /dev/nvme0n1p3  # 드라이브명은 상황에 맞게 변경
```

### 마운트 포인트 생성
```bash
sudo mkdir /mnt/data
```

### 영구 마운트 설정
```bash
sudo nano /etc/fstab
```

맨 아래에 추가:
```
UUID=343002FA3002C33A /mnt/data ntfs defaults,uid=1000,gid=1000,nofail 0 0
```
> UUID는 `blkid` 명령어로 확인한 값으로 변경

### 적용
```bash
sudo umount "/media/anton/새 볼륨"  # 기존 마운트 해제 (있을 경우)
sudo mount -a
```

### 확인
```bash
ls /mnt/data
```

---

## 3. 터미널 경고음(비프음) 끄기

### 방법 1: 터미널 설정
1. 터미널 → **Preferences**
2. 프로필 선택
3. **Terminal bell** 체크 해제

### 방법 2: 시스템 전체 비프음 끄기
```bash
echo "blacklist pcspkr" | sudo tee /etc/modprobe.d/nobeep.conf
sudo rmmod pcspkr 2>/dev/null
```

### 방법 3: Readline 비프음 끄기
```bash
echo "set bell-style none" >> ~/.inputrc
```

### 방법 4: X 서버 비프음 끄기
```bash
xset b off
```

영구 적용 (재부팅 후에도 유지):
```bash
echo "xset b off" >> ~/.profile
```

### 확인
```bash
xset q | grep bell
# bell percent: 0 이면 비활성화된 상태
```

---

## 4. 컴퓨터 이름(호스트명) 변경

### 현재 호스트명 확인
```bash
hostnamectl
```

### 호스트명 변경
```bash
sudo hostnamectl set-hostname 새호스트명
```

### /etc/hosts 파일 수정
```bash
sudo nano /etc/hosts
```

기존 호스트명을 새 호스트명으로 변경:
```
127.0.1.1	새호스트명
```

### 적용
재부팅 필요:
```bash
sudo reboot
```

---

## 참고: Windows 시스템 폴더

NTFS 드라이브에 보이는 폴더들:
| 폴더 | 설명 |
|------|------|
| `$RECYCLE.BIN` | Windows 휴지통 |
| `System Volume Information` | Windows 시스템 복원 지점 |

Ubuntu에서는 무시해도 됨. 숨김 폴더 보기: `Ctrl + H`

---

## 유용한 명령어

| 명령어 | 설명 |
|--------|------|
| `lsblk` | 디스크/파티션 목록 |
| `sudo blkid` | UUID 및 파일시스템 확인 |
| `sudo fdisk -l` | 디스크 상세 정보 |
| `df -h` | 디스크 사용량 |
| `mount -a` | fstab 기반 전체 마운트 |

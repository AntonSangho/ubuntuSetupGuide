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

## 4. 터미널 한글 자간(글자 간격) 문제 해결

Ubuntu 터미널에서 한글이 겹쳐 보이거나 간격이 불규칙한 경우, 한글을 지원하는 모노스페이스 폰트를 설치하면 해결됩니다.

### 방법 1: D2Coding 폰트 설치 (추천)

네이버에서 개발한 개발자용 한글 폰트입니다.

```bash
# 폰트 다운로드
cd /tmp
wget https://github.com/naver/d2codingfont/releases/download/VER1.3.2/D2Coding-Ver1.3.2-20180524.zip

# 압축 해제 및 설치
unzip D2Coding-Ver1.3.2-20180524.zip
sudo mkdir -p /usr/share/fonts/truetype/d2coding
sudo cp D2Coding/*.ttf /usr/share/fonts/truetype/d2coding/

# 폰트 캐시 갱신
sudo fc-cache -fv
```

### 방법 2: Noto Sans Mono CJK 폰트 설치

Google에서 개발한 폰트로, 패키지 관리자로 쉽게 설치 가능합니다.

```bash
sudo apt update
sudo apt install fonts-noto-cjk-extra
sudo fc-cache -fv
```

### 터미널 폰트 변경

1. 터미널 → **Preferences** (또는 우클릭 → Preferences)
2. 사용 중인 프로필 선택
3. **Custom font** 체크
4. **D2Coding** 또는 **Noto Sans Mono CJK KR** 선택

### 설치된 폰트 확인

```bash
# D2Coding 확인
fc-list | grep -i d2coding

# Noto Sans Mono CJK 확인
fc-list | grep -i "Noto Sans Mono CJK"
```

---

## 5. Neovim Markdown Preview 설치

마크다운 파일을 브라우저에서 실시간으로 미리보기 할 수 있는 플러그인입니다.

### 플러그인 설치 (vim-plug 기준)

`~/.config/nvim/init.vim`에 추가:

```vim
Plug 'iamcco/markdown-preview.nvim', { 'do': 'cd app && npm install' }
```

이후 Neovim에서:

```
:PlugInstall
```

### 수동으로 빌드

플러그인 설치 후 브라우저가 열리지 않으면 수동으로 빌드:

```bash
cd ~/.local/share/nvim/plugged/markdown-preview.nvim
npm install
```

또는 Neovim 안에서:

```
:call mkdp#util#install()
```

### 사용법

```
:MarkdownPreview       " 브라우저 열기
:MarkdownPreviewStop   " 브라우저 닫기
```

### 주의: Node.js 버전

Node.js **v12 이상**이 필요합니다. 버전 확인:

```bash
node --version
```

v10 이하인 경우 nvm으로 업그레이드:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
# 터미널 재시작 후
nvm install --lts
nvm use --lts
```

---

## 6. Alacritty 터미널 설치

Alacritty는 GPU 가속을 사용하는 빠르고 가벼운 터미널 에뮬레이터입니다.

### 방법 1: PPA를 통한 설치 (추천)

```bash
sudo add-apt-repository ppa:aslatter/ppa
sudo apt update
sudo apt install alacritty
```

### 방법 2: snap을 통한 설치

```bash
sudo snap install alacritty --classic
```

### 방법 3: cargo를 통한 소스 빌드

```bash
# 의존성 설치
sudo apt install cmake pkg-config libfreetype6-dev libfontconfig1-dev libxcb-xfixes0-dev libxkbcommon-dev python3

# Rust 설치 (없는 경우)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.cargo/env

# Alacritty 빌드 및 설치
cargo install alacritty
```

### 설정 파일 생성

```bash
mkdir -p ~/.config/alacritty
```

기본 설정 파일 생성 (`~/.config/alacritty/alacritty.toml`):

```toml
[font]
size = 13.0

[font.normal]
family = "D2Coding"  # 한글 폰트 사용 시

[window]
opacity = 0.95

[colors.primary]
background = "#1e1e2e"
foreground = "#cdd6f4"
```

### 기본 터미널로 설정

```bash
sudo update-alternatives --install /usr/bin/x-terminal-emulator x-terminal-emulator $(which alacritty) 50
sudo update-alternatives --config x-terminal-emulator
```

### 확인

```bash
alacritty --version
```

---

## 7. 컴퓨터 이름(호스트명) 변경

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

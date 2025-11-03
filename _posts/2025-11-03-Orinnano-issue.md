---
title: NVIDIA Jetson issue 및 해결
description: Jetson orin nano super에서 발생한 웹 브라우저 에러와 절전모드 시 발생한 에러 해결
date: 2025-11-03 12:22:00 +0900
categories: [NVIDIA]
tags: [nvidia, jetson, issue]     # TAG names should always be lowercase
---

# Browser 문제

Jetpack 6.X 버전에서 웹 Browser(firefox, chromium)가 열리지 않는 문제가 있습니다.

NVIDIA Forum에 작성된 글 입니다. [Jetson orin nano - Browser issue](https://forums.developer.nvidia.com/t/jetson-orin-nano-browser-issue/338580)

<img width="636" height="150" alt="image" src="https://github.com/user-attachments/assets/9fcbd3ef-63b5-4327-8c08-a631c3a4bb95" />

답변으로 다시 flash하라고 합니다.

하지만 한 [블로그](https://engineeringcode.tistory.com/entry/Jetson-Nano-SELinux-%EC%98%A4%EB%A5%98%EA%B0%80-%EB%B0%9C%EC%83%9D%ED%95%98%EB%A9%B0-%ED%8C%8C%EC%9D%B4%EC%96%B4%ED%8F%AD%EC%8A%A4%EA%B0%80-%EC%8B%A4%ED%96%89%EB%90%98%EC%A7%80-%EC%95%8A%EC%9D%84-%EB%95%8C-%ED%95%B4%EA%B2%B0-%EB%B0%A9%EB%B2%95)의 방법으로 시도해보면 해결 되었습니다.

문제가 되는 'snap'의 버전을 변경하는 것입니다.

```bash
sudo snap revert snapd
snap download snapd --revision=24724
sudo snap ack snapd_24724.assert
sudo snap install snapd_24724.snap
```

해당 방법으로 브라우저가 정상 작동합니다.

# GUI 절전모드 문제
## 재부팅 문제 해결

crontab을 사용해서 지정한 시간에 절전 모드를 들어가고, 다른 시간에 절전모드를 깨우는 방법을 사용했습니다.
절전모드가 되는 것을 확인하고 다시 켜려고 하는데
```bash
jetson audit: type=1326 audit(1761640322.080:4): auid4294967295 uid=0 gid=0 ses=4294967295 pid=1644 comm="cups-browsed" exe="/snap/cups/1113/sbin/cups-browsed" sig=0 arch=c00000b7 syscall=274 compat=0 ip=0xffff9dcf5b68 code=0x50000
```
부팅중에 위의 로그와 함께 계속 멈춰있는 현상이 발생했습니다.

GPT를 이용해서 절전 복귀나 재부팅 후 GUI(GNOME Shell) 초기화에 실패한 전형적인 상태라는 것을 알게되었고

```Mark Down
5️⃣ 복구 가능성 판단

CLI 로그인까지 가능하면 커널은 살아있고, GUI 영역만 깨진 상태입니다.
이 경우 gdm3 와 gnome-shell 을 재설치하거나 모듈 재로드를 시도할 수 있습니다.

👉 1) GPU 모듈 재로드
sudo systemctl isolate multi-user.target
sudo modprobe -r nvgpu nvdla_video dce tegra_dc_common tegra_nvdisp
sudo modprobe nvgpu
sudo systemctl isolate graphical.target

👉 2) GNOME 다시 설치
sudo apt purge --auto-remove gdm3 gnome-shell -y
sudo apt install gdm3 ubuntu-desktop-minimal -y
sudo systemctl enable gdm3
sudo reboot
```

위의 GNOME 다시 설치하는 방법으로 복구했습니다.

절전 명령어를 사용하면 안될 것 같습니다.

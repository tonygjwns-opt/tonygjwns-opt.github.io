---
title: 환경 트러블슈팅 (모니터링 / 인터넷)
date: 2026-06-15 21:30:00 +0900
categories: [컴공, 기타]
tags: [트러블슈팅, 네트워크, linux, 윈도우]
---

## 1. 코드가 자꾸 터질 때 모니터링

- `top` : CPU·메모리 등 프로세스 모니터링.
- `watch -n 3 nvidia-smi` : 3초마다 GPU 상태 갱신 → 어느 코어가 맛가는지 확인.
- 결국 몇 번째 코어인지 못 잡으면 `tqdm` looper 안에서 직접 표시하게 했음.

## 2. 인터넷: 일부 페이지만 안 들어가질 때

증상: 인터넷 접속은 되고 네이버·구글은 되는데 **일부 페이지만** 안 됨(폰 와이파이는 됨 → 랜선/공유기 문제 아님).

해결: cmd에 아래 입력 후 **재부팅**으로 해결.

```cmd
netsh int ip reset
```

(DNS/접속 캐시도 지웠는데 그게 도움됐는지는 미지수.)

그래도 안 되면 순서대로:

```cmd
netsh winsock reset       :: 윈속(소켓) 카탈로그 초기화
netsh int ip reset        :: TCP/IP 스택 초기화
ipconfig /release         :: 현재 IP 반납
ipconfig /renew           :: IP 재발급(DHCP)
ipconfig /flushdns        :: DNS 캐시 비우기
```

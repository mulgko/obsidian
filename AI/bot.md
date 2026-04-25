---
title: bot
subject: ""
reference: ""
date: 2026-03-25 17:29
description: ""
tags:
  - bot
series: ""
seriesOrder:
published: false
---

# bot

 1. DISCORD_BOT_TOKEN

  2. Discord Developer Portal에 접속
  3. New Application → 이름 입력 → 생성
  4. 좌측 Bot 메뉴 → Reset Token → 토큰 복사
  5. Privileged Gateway Intents에서 Message Content Intent 활성화

  6. DISCORD_CHANNEL_ID

  7. Discord 설정 → 고급 → 개발자 모드 켜기
  8. 모니터링할 채널을 우클릭 → 채널 ID 복사

  봇 서버 초대

  봇을 서버에 추가해야 메시지를 읽을 수 있습니다:

  9. Developer Portal → OAuth2 메뉴
  10. Scopes: bot 체크
  11. Bot Permissions: Read Message History, View Channels 체크
  12. 생성된 URL로 접속해서 서버에 초대

  그 후 .env.local에:

  DISCORD_BOT_TOKEN=복사한-봇-토큰
  DISCORD_CHANNEL_ID=복사한-채널-ID
---
title: 롤 데이터 분석 - 나는 롤을 하면서 얼마나 까만화면을 보는가
description: 롤 게임을 평균 몇분에 끝내는지, 게임을 하면서 보통 얼마나 죽어있는지 알아보겠습니다.
date: 2024-11-14 13:35:00 -0400
categories: [Data]
tags: [riot, riot api]     # TAG names should always be lowercase
---
  
RIOT에서는 개인에게 유저들의 정보를 제공해줍니다.

저의 롤 닉네임으로 최근에 한 게임을 분석 해보겠습니다.

분석의 목적은 나는 게임을 평균 몇분에 끝내는지,

게임을 하면서 보통 얼마나 죽어있는지를 알아보는 것입니다.
  
# 데이터 준비
  
[RIOT_API](https://developer.riotgames.com) 발급받기

해당 주소에서 자신의 LOL 아이디로 로그인해서 API를 발급 받을 수 있습니다.
  
API 요청 횟수 1분에 20건 2분에 100건의 제한이 있다고 하니 주의해주세요.

```python
https://kr.api.riotgames.com/lol/summoner/v4/summoners/by-name/{nickname}?api_key={riot_token}
```

위의 주소로 데이터를 요청했습니다.
  
# 매치 기록 가져오기
 
개인은 최대 최근 100게임을 가져올 수 있습니다.
  
## CLASSIC 게임만 필터
전체 게임 데이터 중에서 무작위 총력전, 연습모드 등을 제외하고

CLASSIC 으로 분류된 일반게임, 랭크게임 데이터만을 뽑아서 사용하겠습니다.

![](/images/lolblackscreencheck/e37ab82f-4fd6-4a1e-819c-aa0a365e56f1-image.png)

데이터를 표로 만들었습니다.
시간은 초 단위로 되어있습니다.
  
 # 시각화
 게임의 길이가 3분인 유저의 탈주로 게임이 성립되지 않은 게임은 제거했습니다.
  
  ![](/images/lolblackscreencheck/b23688a0-e382-42eb-b277-b688da087887-image.png)

  최근 게임 중 가장 길었던 판은 53분을 플레이 했고
  가장 오래 죽어있던 시간은 11분이었습니다.

  ![](/images/lolblackscreencheck/ee5848ce-9d13-42b9-b54b-96c0b45bb826-image.png)

> 저는 평균적으로 게임을 하면 게임 시간의 1/10은 까만화면을 보고있었습니다.
  
이상으로 RIOT에 저의 데이터를 요청해서

보통 몇분의 게임을 하는지,

얼마나 죽어있는지 등을 알아봤습니다.

읽어주셔서 감사합니다. 🙂
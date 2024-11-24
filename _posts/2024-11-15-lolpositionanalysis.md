---
title: 롤 데이터 분석 - 포지션 찾아보기
description: riot 에서 제공하는 매치 데이터로 어떤 포지션이 안성맞춤인지 알아보겠습니다.
date: 2024-11-15 16:35:00 -0400
categories: [Data]
tags: [riot, riot api]     # TAG names should always be lowercase
---

# 나에게 맞는 포지션 찾아보기

저는 리그오브레전드를 플레이 할 때 어떤 포지션이든 마다 하지 않는 올라운더입니다.

하지만 이제는 주 포지션을 정해서 T자형 인재가 되고싶네요.

그런 의미에서 _riot_ 에서 제공하는 저의 매치 데이터로 어떤 포지션이 안성맞춤인지 알아보겠습니다.


### 가설

> 라인전 0~15분 동안 death 횟수가 적은 lane을 잘한다.

어떤 포지션이 잘 맞는지 여러가지 기준에 의해서 정할 수 있겠지만

이번 분석에서는 초반 라인전의 죽음 횟수를 기준으로 판단하겠습니다.


----




## 📁 데이터 불러오기

~~[riot api 데이터 저장하는 방법](https://core.today/story/NQ7O33x5)~~

riot api를 사용해서 미리 저장해 놓은 데이터를 불러오겠습니다.

```python
# /lol/match/v5/matches/{matchId} 데이터
match_api_list = sorted(glob('match_api/*'),key=os.path.getctime)
```

매치 정보를 json으로 저장한 데이터 목록을 불러오고

해당 목록에서 데이터 하나하나에 접근해 데이터프레임(표)를 만들겠습니다.


---


## ✂ 데이터 전처리

### 매치 정보 뽑기

먼저 비어 있는 데이터 프레임을 만들어줍시다.

```python
# 빈 데이터프레임 만들기
match_api_df = pd.DataFrame(
    columns=[
        'match_id','match_type','individual_position','lane','team_position','role'
    ])
```

> 'match_id' : 매치 아이디 
> 'match_type' : 매치 유형 (총력전, 일반)
> 'individual_position' : 포지션 
> 'lane' : 주 lane
> 'team_position' : 팀 포지션 (TOP, BOTTOM...)
> 'role' : 역할 (SOLO,DUO,CARRY.....)

![](/images/lolpositionalanysis/3a29528d-4977-4f6d-a8c3-dfc7f920dcf3-image.png)

여기서 총력전, 사용자 설정 게임을 제외한
일반, 랭크 게임만 따로 추출해서 표를 하나 더 만듭니다.

```python
# 일반, 랭크 게임만 추출
classic_game_df = match_api_df.query('match_type=="CLASSIC"')
```


---



### timeline 데이터로 죽은 시간, 좌표 뽑기

이번에도 같은 순서로 진행됩니다.

```python
#  /lol/match/v5/matches/{matchId}/timeline 데이터 목록
match_timeline_api_list = sorted(glob('match_timeline_api/*'),key=os.path.getctime)
```

> 'match_id' : 매치 아이디
> 'match_type' : 매치 유형 (총력전, 일반...)
> 'role' : 위 데이터 프레임의 individual_position 값
> 'timestamp' : 죽은 시간
> 'position_x' : 죽었을 때의 x 좌표
> 'position_y' : 죽었을 때의 y 좌표

![](/images/lolpositionalanysis/fa74b71c-ba81-4f35-bd33-5eeaf06dd873-image.png)

저는 라인전 종료 기준은 15분으로 잡겠습니다.
timestamp는 ms(millisecond) 단위입니다.

```python
before_15_df = match_timeline_api_df.query('timestamp <= 900000')
```


---



## 🗺 미니맵에 나타내기

15분 이전에 죽은 장소를 미니맵 위에 찍어보겠습니다.

색으로 구분된 role은 라이엇이 제공한 individual_position값입니다.

![](/images/lolpositionalanysis/9116869e-d945-4664-9e0b-34fb2ab21225-image.png)

![](/images/lolpositionalanysis/181943cf-e311-4963-b4b8-f0d60966cf48-image.png)

![](/images/lolpositionalanysis/c2d624a2-e428-4927-8a03-a27b4fc7ef56-image.png)

![](/images/lolpositionalanysis/9dfb74a0-3ad6-4cf7-bda4-df54136c2079-image.png)

얼핏 봤을 때 어디에서 죽었는지 잘 모르겠네요.

히트맵으로 나타내 보겠습니다.

![](/images/lolpositionalanysis/b1ec03a2-2670-4d20-95f1-5a35a1db23f1-image.png)


죽은 위치가 미드에 많으니 저는 미드를 포기해야하는 것일까요..?



---



## 🔎 역할별 판수와 죽은 횟수 비교하기

미드를 많이 했다면 미드에서 죽은 횟수가 많을 것입니다.

그래서 어떤 역할을 많이 했고, 얼마나 죽었는지

역할별 죽은 횟수 / 역할별 게임 횟수로 비율을 알아보겠습니다.

![](/images/lolpositionalanysis/2afc26a7-4f56-4d75-9875-5eba904e338b-image.png)


## 🧑‍⚖️ 결론

죽은 비율을 기준으로 봤을 때 주 포지션과 부포지션
1. 탑
2. 미드



자세한 코드는 [여기](https://github.com/dongdongju96/data-story/blob/main/league_of_legends/where_is_my_lane.ipynb)에서 볼 수 있습니다.



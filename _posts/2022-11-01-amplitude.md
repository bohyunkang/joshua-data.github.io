---
layout: post
title: "Amplitude 톺아보기"
tags:
  - korean
  - amplitude
  - data_analytics
mermaid: true
---

> 이번 아티클에서 다룰 이야기는 바로  **Amplitude**(앰플리튜드)다. 데이터 분석 포지션으로 입사를 한 후, 가장 먼저 접한 것이 Amplitude였는데, 새싹(🍀) 처럼 초심이 가득했던 마음으로 하나하나 익혀간 덕분에 애정이 각별한 툴이다. 개발팀, 마케팅팀 등 디센트와 함께 심장이 활활 타오르며 일하는 팀원 분들과 함께 하나하나 세팅하면서 익힌 Amplitude를 톺아보도록 하겠다.

### CONTENTS
1. 데이터 기반 성장의 시대
2. Amplitude: “나는 네가 지난 여름에 한 일을 알고 있다.”
	* 2.1. 고객  (User)
	* 2.2. 행동 (Event)
3. DB Schema를 떠올려보면 Amplitude도 결국 SQL 날리는 거야!
4. Amplitude는 결국 전부 필터링 & 분류, That’s enough!
5. 그래서 Amplitude 고수가 되려면 어떻게 해야 하는데?

---

# 1. 데이터 기반 성장의 시대

우리는 데이터의 시대에 살고 있다. 데이터 공부를 한 사람 입장에서 아주 자기중심적이고 과장된 표현이라는 것을 인정하지만, 데이터를 보지 않는 기업이나 국가기관은 이제 거의 없다.

빅데이터의 수집, ETL(추출, 변환, 적재) 인프라와 하드웨어의 발전 덕분에 수 년 전 데이터가 돈이 될 수 있는 시대에 진입했었다면, 지금은 기업의 경영과 국가 정책 결정 과정에 있어서 데이터가 필수 불가결한 수준까지 도달했다. 즉, 데이터 없이는 생존하기 어려운 시대가 온 것이다.

물론 데이터가 의사결정이나 전략적 방향 수립을 위한 유일무이한 관점이라고 말할 수는 없다. 그럼에도 불구하고, 사람의 직관으로 결론 내리기 어려운 것들에 대해 데이터가 매우 큰 힘을 발휘해줄 수 있으며, 효율적인 전략 방향 설정과 티핑 포인트(Tipping Point)를 탐색하는 데 데이터는 매우 중요한 도구적 가치를 지닌다.

필자가 근무 중인 모 기업 역시, 전사적으로 데이터 분석 인프라를 갖춤으로써 데이터 기반 의사결정과 그로스 해킹 과정을 숱하게 겪고 있다. Amplitude, Google Analytics, AppsFlyer 등 3rd-party 데이터 분석 툴은 물론이고, BigQuery, Redash, 및 로컬 DB를 통해서도 분석에 적극적으로 활용하고 있다.

![Amplitude Logo]({{ site.baseurl }}/assets/2022-11-01-amplitude/amplitude-logo.jpeg)
> Amplitude의 로고. 이니셜인 A를 Line Chart 처럼 형상화한 것이 개인적으로 참 마음에 든다.

데이터 분석 스토리 시리즈의 첫 번째 이야기는 바로  **Amplitude**(앰플리튜드)다. 데이터 분석 포지션으로 입사를 한 후, 가장 먼저 접한 것이 Amplitude였는데, 새싹(🍀) 처럼 초심이 가득했던 마음으로 하나하나 익혀간 덕분에 애정이 각별한 툴이다. 개발팀, 마케팅팀 등 디센트와 함께 심장이 활활 타오르며 일하는 팀원 분들과 함께 하나하나 세팅하면서 익힌 Amplitude를 톺아보도록 하겠다.

![The Martian]({{ site.baseurl }}/assets/2022-11-01-amplitude/martian.jpeg)
> “개발자님! 이벤트 세팅해주신 것들 트랙킹된다! 트랙킹된다! 카피 댓!” (영화 마션)

# 2. Amplitude: “나는 네가 지난 여름에 한 일을 알고 있다.”

![I Know What You Did Last Summer]({{ site.baseurl }}/assets/2022-11-01-amplitude/i-know-what-you-did-last-summer.png)
> “나는 네가 지난 여름에 한 일을 알고 있다.” (1997년 개봉한 짐 길레스피 감독의 영화)

Amplitude를 살짝 섬뜩하게 표현하면, “디센트 앱을 이용하는 고객들이 무엇을 했는지 다 기록하고, 조회할 수 있는 툴”이다. 즉, 고객들이 다음과 같은 행동을 했을 때 Amplitude는 이를 데이터로 기록해두고, 데이터 분석을 하는 나는 이를 실시간 차트로 표현할 수 있다.
* 회원가입을 완료하다.
* 장바구니에 상품을 추가하다.
* 프로모션 배너를 클릭하다.

물론, Amplitude는 각국의 개인정보 보호법을 준수하기 위해 모든 사용자 식별자를 암호화하여 생성한다. 이 때문에 데이터 분석가는 특정 고객의 이름이나 생년월일을 Amplitude에서 실시간으로 추적할 수 없을 뿐더러, 그 어떠한 제3자 데이터베이스에도 보관되지 않는다. (물론, 다른 직원들도 알 방법이 없다.) 다만, 특정 행동을 한 고객들의 수, 국가, 운영체제 등으로 그룹화하여 분석을 진행할 뿐이다.

앞서 Amplitude를 섬뜩하게 표현했는데, 위키피디아 스타일로 정의를 내리자면 다음이 적절할 것 같다.
> “Amplitude는  **고객**의  **행동**을 기록하고, 조회할 수 있는 툴이다.”

**고객**과 **행동**의 의미가 정확히 무엇인지 나누어 살펴보자.

## 2.1. 고객  (User)

Amplitude에서는 고객을 User로 명명하여, 사용 기기(Device)의 정보나 로그인 정보 등 다양한 방식으로 식별하고 있다. User를 식별하는 것은 앱 데이터를 분석하는 과정에서 매우 (X 100) 중요하다. 가령, "손흥민”이라는 고객을 2명이 아닌 1명의 고객으로 잘 식별해야만 해당 고객이 회원가입을 하고, 최종 상품을 구매할 경우, 전환율(Conversion Rate)을 정확하게 측정할 수 있기 때문이다. Amplitude에서는 이러한 고객의 식별자를 Amplitude ID로 명명하고 있다.

Amplitude의  [Tech Docs](https://help.amplitude.com/hc/en-us/articles/115003135607-Track-unique-users-in-Amplitude#h_5c321a51-5d16-43f0-a4fe-f907567ad63b)를 보면 식별 메커니즘을 명확하게 설명하고 있는데, 고객을 정확하게 식별하기 위해 2중으로 식별 알고리즘을 적용하여  **Amplitude ID**를 생성하고 있다.

|**알고리즘**|**우선순위**|**의미**
-|-|-
User ID | 1 | 앱 개발자가 직접 설정한 사용자 식별 방식
Device ID | 2 | 사용 기기의 IDFV를 가져오거나 Random Alphanumeric 문자열을 쿠키 데이터로 심어놓음
Amplitude ID | 3 | User ID > Device ID 순으로 참조하여 최종 고객 식별

## 2.2. 행동 (Event)

행동은 단어 그대로, 고객의 특정 행동을 의미한다. 이를 좀 더 개발스럽게(?) 표현하면, 앱 상에서 고객이 클라이언트 내에서 서버를 향해 요청(Request)한 트랜잭션이라고 할 수 있다. 개발자가 아니라면 외계어(👾)처럼 들릴 수 있으니, 다음 그림을 한 번 보도록 하자.

![]({{ site.baseurl }}/assets/2022-11-01-amplitude/user-events.png)
> 필자는 그림에 소질이 없으니, 못생긴 그림이어도 이해를 바란다.

즉, 행동(Event)은 공지사항을 클릭하거나, 구매를 완료하는 등 서버에 전달될 수 있는 사건들을 의미하는 것이다.

자, 그럼 위키피디아 스타일의 Amplitude 정의를 좀 더 아름답게 표현해보면, 다음과 같다.
> “Amplitude는  **주어(User)**  +  **동사(Event)**로 데이터를 기록하고, 우리는 이 데이터를 조회할 수 있다.”

가령, 월드클래스️⚽손흥민과 피겨퀸 💃김연아, 총 2명의 고객이 우리 앱을 사용하고 있다고 가정해보자. 그리고 흥민님과 연아님은 오늘 디센트 앱에서 다음과 같은 Event를 발생시켰다.

**User**|**Event**
-|-
흥민 | 회원가입을 완료했다.
흥민 | 장바구니에 상품을 추가했다.
흥민 | 프로모션 페이지를 조회했다.
연아 | 공지사항을 클릭했다.
연아 | 댓글을 작성했다.
연아 | 프로모션 페이지를 조회했다.

그리고 데이터 덕후인 필자가 아래와 같은 궁금점을 해결하기 위해 Amplitude로 데이터 조회를 해보니, 다음과 같은 결과를 얻을 수 있었다.

* 영국(United Kingdom)에서 접속한 User의 Event가 무엇이 있을까?

**User**|**Event**
-|-
XXX | 회원가입을 완료했다.
XXX | 장바구니에 상품을 추가했다.
XXX | 프로모션 페이지를 조회했다.

> 참고로 흥민님은 작성일 현재 영국 런던에 거주한다.
> 오호, 영국 고객들이 이런 이벤트들을 행했구나!

* "프로모션 페이지 조회” Event를 행한 User는 총 몇 명일까?

**User**|**Event**
-|-
XXX | 프로모션 페이지를 조회했다.
XXX | 프로모션 페이지를 조회했다.

> 오늘 고객 2명이 디센트 아카데미를 클릭했구나!

# 3. DB Schema를 떠올려보면 Amplitude도 결국 SQL 날리는 거야!

Amplitude의 내부 데이터베이스를 살펴본 적도 없고 살펴볼 수도 없겠지만, Amplitude가 데이터를 기록하는 방식을 다음 ERD 다이어그램 정도로 이해하면 훨씬 습득력이 향상될 것 같다.
* 물론, Amplitude Data Export 기능을 활용하게 된다면 테이블 구조를 더욱 명확하게 확인할 수 있을 것이다.

![]({{ site.baseurl }}/assets/2022-11-01-amplitude/user-events-erd.png)

즉, User에는 식별자인 ID 뿐만 아니라, 거주 국가, 언어, 사용 기기, 운영체제 등 프로파일에 해당하는 정보를 가지고 있고, 각 User는 1개 이상의 Event를 행한다. 그리고 각 Event 역시 ID 뿐만 아니라, 상세 프로파일을 가지고 있다.

# 4. Amplitude는 결국 전부 필터링 & 분류, That’s enough!

Amplitude가 데이터를 수집하는 아키텍처를 굉장히 복잡하게 설명했지만, 사실 데이터 분석 업무에 이를 잘 녹여내기 위해서는 크게  **필터링**과  **분류**만 잘 하면 된다. 이를 좀 더 SQL스럽게(?) 말하면,  **WHERE** Syntax와  **GROUP BY**  Syntax만 잘 먹여주면 된다. 가령 다음과 같은 질문을 Amplitude가 쉽게 대답해줄 수 있다.

### 전체 사용자들을 국가(Country)로 분류 좀 해줄래?

![]({{ site.baseurl }}/assets/2022-11-01-amplitude/user-group-by-country.png)
> Amplitude의 User Composition 차트

* 다음 SQL 문과 너무 유사한 컨셉이지 않은가?!
```sql
	SELECT
		country,
		COUNT(user_id) AS user_cnt
	FROM
		users
	GROUP BY
		country
	ORDER BY
		user_cnt DESC
```

### 사용자 가이드 클릭 수를 일자 별로 흐름을 알려줄래?

![]({{ site.baseurl }}/assets/2022-11-01-amplitude/click-by-date.png)
> Amplitude의 Segmentation 차트

* 다음 SQL 문과 너무 유사한 컨셉이지 않은가?
```sql
	SELECT
		DATE_TRUNC('DAY', event_timestamp) AS date,
		COUNT(DISTINCT user_id) AS user_cnt
	FROM
		events
	WHERE
		event_name = 'click_academy'
	GROUP BY
		DATE_TRUNC('DAY', event_timestamp)
	ORDER BY
		DATE_TRUNC('DAY', event_timestamp)
```

# 5. 그래서 Amplitude 고수가 되려면 어떻게 해야 하는데?

엑셀과 SQL이 그렇듯, Amplitude도 쓰면 쓸수록 실력이 일취월장한다고 생각한다.
* “아, 내가 이 경우를 고려하지 못하고 데이터를 추출했구나.”
* “이런 된장, Null 값을 제외 못했네.”
* “망했어, 기간 설정을 잘못했어.”
* “비율을 거꾸로 적용했네. 어쩐지 이상하더라.”

이러한 시행착오가 쌓이면 쌓일수록 (그렇다, 내 이야기 맞다.) 정확한 필터링과 분류 능력이 향상되는 것 같다. 추가로, 심심할 때마다 다음 컨텐츠들을 읽어보면서 틈틈이 공부하는 것도 매우 도움이 될 것이라고 생각한다.
* [Amplitude의 공식 Medium Page](https://medium.com/@amplitudeHQ)
* [Timothy Daniell의 Medium Page](https://timothydaniell.medium.com/)
* [AB180 Amplitude 블로그](https://blog.ab180.co/category/amplitude)

그럼, 이 땅의 모든 데이터 분석가, 마케터, 프론트엔드 개발자들을 응원하며, 글을 마친다. 또 다른 주제로 다시 돌아온다!

![]({{ site.baseurl }}/assets/2022-11-01-amplitude/the-simpsons.jpeg)
> 너무 데이터 분석만 하지말고, 가끔씩 이렇게 멘탈을 놓고 춤도 춰야 한다.

![]({{ site.baseurl }}/assets/2022-11-01-amplitude/alan-turing.png)
> “Those who can imagine anything, can create the impossible.” (어떤 것이든 상상이 가능한 사람은 불가능을 가능으로 바꿀 수 있다. ) - Alan Turing (튜링 머신의 아버지, 1912~1954)

---

## *Published by* Joshua Kim
![Joshua Kim]({{ site.baseurl }}/assets/profile/joshua-profile.png)
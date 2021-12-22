---
layout: post
title:  "Halting Problem - 정지 문제"
date: 2018-07-28 13:00:00 +0900
description: Halting Problem
img:  # Add image post (optional)
tags: [Halting Problem, 정지 문제]
---
# Halting Problem
정지 문제

개략적인 증명  
증명은 귀류법(proof by contradiction)을 이용한다. 우선 모든 프로그램을 최소한 하나의 문자열과 연관시킬 수 있는 프로그래밍 언어를 고르자.  
임의의 문자열 i와 임의의 프로그램을 나타내는 문자열 a에 대해, i라는 문자열을 입력으로 프로그램 a을 실행했을 경우를 계산하여, 그 실행이 끝나면 true, 끝나지 않고 영원히 계속되면 false를 반환하는 halt(a, i)란 알고리즘이 있다고 누군가 주장한다고 하자.  
이때 다음과 같이 trouble이라는 서브루틴을 만들어 보았다고 하면,

```
 function trouble(string s)
     if halt(s, s) == false
         return true
     else
         loop forever
```

이 프로그램은 프로그램을 나타내는 문자열 s를 인자로 받아 s를 halt의 두 인자(프로그램과 초기 문자열)로 주어 실행하고, halt가 false를 반환하면 true를 반환하고, 그렇지 않으면 무한 루프로 빠져버리는 프로그램이다.  
모든 프로그램은 어떤 문자열인가로 표현될 수 있기 때문에, 어떤 문자열 t는 이 프로그램 trouble을 나타낼 것이다.  
이 t라는 문자열을 이 프로그램에 넣어 돌린 trouble(t)는 과연 반환값을 내놓을까, 아니면 영원히 멈추지 않을까?

두 가지 경우를 모두 고려해 보면 다음과 같다.

만약 trouble(t)가 계산을 끝낸다고 하면, 그건 분명히 halt(t, t)가 반환값으로 false를 내놓기 때문이다. 그러나, 그것은 다시 trouble(t)가 멈추지 않고 영원히 지속된다는 말이다.  
만약 trouble(t)이 무한히 돈다면, 그것은 halt가 영원히 계산을 끝내지 않거나, halt가 true를 반환하기 때문이다. 이것은 다시 halt 가 임의의 문자열과 프로그램에 대해 판정할 수 없거나, trouble(t)가 계산 끝내고 멈춘다는 결론을 이끌어낸다.  
어떤 경우에도 halt는 옳은 답을 내놓지 못하여, 처음의 주장과 모순을 일으킨다.  
이 논리는 정지문제에 대한 답으로 제시되는 모든 프로그램에 적용될 수 있기 때문에, 이 문제에 대한 해답은 있을 수 없다.

출처 - 위키백과 https://ko.wikipedia.org/wiki/%EC%A0%95%EC%A7%80_%EB%AC%B8%EC%A0%9C
---
layout: post
title: TypeSafeMatcher를 이용한 커스텀 매처 구현
categories: Test
---

테스트 관련 서적을 읽다보면 Hamcrest를 이용한 커스텀 매처를 구현해서 사용하는 경우를 볼 수 있는데, 직접 구현해보자.

### TypeSafeMatcher란?

TypeSafeMatcher는 주로 JUnit이나 다른 테스트 프레임워크에서 사용되는 Matcher 클래스의 특수한 유형으로, 특정 객체의 타입에 안전한 비교를 제공하는 기능을 한다.

주로 Hamcrest 라이브러리와 함께 사용되며, 타입이 안전하게 매칭될 수 있도록 보장한다.

### 구성
TypeSafeMatcher<T> : TypeSafeMatcher<T> 클래스는 T 타입의 객체를 안전하게 처리할 수 있게 한다. 매칭을 수행할 때 T 타입을 명시하여 타입 안전성을 보장한다.

matchesSafely : 실제 매칭 로직을 구현한다.

describeTo : 매칭 실패 시 출력될 설명을 제공

### 예제

유저 이름이 일치하는지 확인하는 테스트 코드를 작성해보자.

먼저 TypeSafeMatcher 를 구현한다.

{% gist Ghosttrio/8047ebfb10b9a62b5882a10582c17416 %}

테스트 코드

{% gist Ghosttrio/4635cdd1587edcca3c14d66492c4e3a6 %}

실패 메시지 확인

{% gist Ghosttrio/de4ab5cebce3c746c49c13ee8a0c5cf6 %}
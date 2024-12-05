---
layout: post
title: public static final 배열의 보안 허점
categories: Java
---

[이펙티브 자바](https://www.aladin.co.kr/m/mproduct.aspx?ItemId=171196410&srsltid=AfmBOopXSGP8EfAIkTYufxeZWidZHty9bG3kEqRSP1QPFcXEyn4k9nVm)라는 책의 내용 중에 상수라면 public static final 필드로 공개해도 괜찮고,
클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공해서는 안 된다고 한다. 

나는 public static final 배열 필드를 비슷하게 Security에서 ALLOW URL을 관리를 위해 사용했었다.

filter에 허용이 필요한 url이나 테스트 용도로 작성하기 편했기 때문이다.

{% gist Ghosttrio/ec2ffc47ef279e3ae49542be1ce7bf11 %}

### 왜 public static final 배열 필드를 반환하는 접근자 메서드를 제공하면 안될까?

간단한 코드를 만들어보자. 아래와 같은 public static final 배열이 있다고 가정해보자.

{% gist Ghosttrio/1c416fe1654c521d66c4b9f33e53b565 %}

위 코드는 변경이 가능한 보안 허점이 있다.

public static final로 선언된 배열 자체는 변경이 불가능하지만 배열의 요소는 변경이 될 수 있기 때문이다.

즉, 배열의 참조는 불변이지만 배열의 내용은 변경이 가능한 것이다.

먼저 보안 허점을 생각하지 못하게 한 final을 확인해보자. Test.ARR 에 새로운 String[] 배열을 넣으면 에러가 발생한다.

{% gist Ghosttrio/e98814a52d3aa59a8571938cb6f59b41 %}

하지만 배열의 참조를 변경하지 말고 배열의 요소를 변경해보면, 에러가 발생하지 않고 변경이 된다!

{% gist Ghosttrio/1459f567b5d7419a31f06ea42d3d76bb %}

### 그렇다면 어떻게 이러한 변경 가능성을 어떻게 막아야 할까?

책에서는 두 가지 방법을 제시한다.
1. 앞 코드의 public 배열을 private로 만들고 public 불변 리스트를 추가한다. 
2. 배열을 private로 만들고 그 복사본을 반환하는 public 메서드를 추가한다 (방어적 복사)

불변 리스트는 unmodifiableList로 만든다.

{% gist Ghosttrio/2587c830e4de3318c53d16475b3b2bd6 %}
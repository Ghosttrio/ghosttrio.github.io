---
layout: post
title: 자바 @FunctionalInterface
categories: Java
---

@FunctionalInterface는 Java 8에서 도입된 어노테이션으로, 람다 표현식과 메서드 참조에 사용될 수 있는 함수형 인터페이스임을 나타낸다.

함수형 인터페이스는 단 하나의 추상 메서드를 가지는 인터페이스를 말한다.

@FunctionalInterface를 통해 함수형 인터페이스를 정의하면, 컴파일러가 자동으로 해당 인터페이스에 하나의 추상 메서드만 존재하는지 확인해 주므로 실수를 방지할 수 있다.

사실 함수형 인터페이스에 @FunctionalInterface 어노테이션을 붙이지 않아도 같은 기능으로 작동한다. 

하지만 컴파일러에서 하나의 추상 메서드만 있는지 검사하는 기능이 없다는 점과 이 인터페이스가 함수형 인터페이스라는 의미를 알 수가 없다는 점 때문에, 어노테이션을 적어주도록 하자.

### 예제

FunctionalInterface 정의는 @FunctionalInterface 어노테이션을 인터페이스에 붙여주면 된다.

{% gist Ghosttrio/debe9c84a5480e4b11839dbef3d62b24 %}

default method는 여러개 선언 할 수 있다.

{% gist Ghosttrio/06d2fe3459bd406aa429d136126a93a9 %}

함수형 인터페이스 사용

{% gist Ghosttrio/718096cf74512abd2f4b4d9db8c814e2 %}


파라미터와 리턴타입을 바꿔서 활용할 수 있다.

{% gist Ghosttrio/7d4dcf0906395960cffcdaf74e6ec607 %}

파라미터로 넣거나 리턴타입으로 함수형 인터페이스를 사용해서 활용할 수도 있다.

{% gist Ghosttrio/a61751a9f8a2af52988b5ec083835d82 %}
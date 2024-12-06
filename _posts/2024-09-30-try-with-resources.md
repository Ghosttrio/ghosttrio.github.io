---
layout: post
title: try-finally 보다는 try-with-resources를 사용하라
categories: Java
---

try-with-resources는 자바 7에서 도입된 문법으로, 자원을 자동으로 관리하고 닫을 수 있도록 도와주는 기능이다.

try-with-resources는 AutoCloseable 인터페이스를 구현하는 객체들을 사용하여 자원을 자동으로 닫을 수 있게 해준다.

AutoCloseable 인터페이스는 close() 메서드를 가지고 있는데, 이 메서드는 객체가 더 이상 필요하지 않을 때 호출되어 자원을 정리한다.

### 기본 문법

{% gist Ghosttrio/8c73ffb59977727b08813bf9efd2fcc9 %}

### 예제

데이터베이스를 사용하는 코드

{% gist Ghosttrio/ae380c835ca9ba34b7399a50574e5938 %}

직접 AutoCloseable 구현

{% gist Ghosttrio/2b841384c2b08deff129dd64db9eca6c %}

### try-with-resources를 사용해야 하는 이유

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다.

하지만 예외는 try 블록과 finally 블록 모두에서 발생할 수 있어, 두 번째 예외가 첫 번째 예외를 완전히 먹어벼려, 스택 추적 내역이 남지 않아서 디버깅이 어려워 진다.

또한 명시적으로 finally 블록을 만들어야 해서 실수로 작성하지 않게 되는 문제를 제거해준다.
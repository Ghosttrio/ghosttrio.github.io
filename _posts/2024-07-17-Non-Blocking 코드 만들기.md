---
layout: post
title: Non-Blocking 코드 만들기
categories: Java
---

[모던 자바 인 액션](https://www.yes24.com/Product/Goods/77125987?pid=123487&cosemkid=go15646485055614872&utm_source=google_pc&utm_medium=cpc&utm_campaign=book_pc&utm_content=ys_240530_google_pc_cc_book_pc_11906%EB%8F%84%EC%84%9C&utm_term=%EB%AA%A8%EB%8D%98%EC%9E%90%EB%B0%94%EC%9D%B8%EC%95%A1%EC%85%98&gad_source=1&gclid=Cj0KCQiAu8W6BhC-ARIsACEQoDC1ITNjsdskKFycjwsI7k9J0jnNnmlmPBGcj-XW4Bazk7QEjn_iXUMaAo4fEALw_wcB)을 공부하면서 만난 예제 코드를 연습겸 정리해보자.

모든 상점의 정보를 순차적으로 요청하는 메서드를 성능을 고려하여 개선하며 구현해보았다.

예시로 상점 리스트를 먼저 만들어보자.

{% gist Ghosttrio/c09167e1b141bd5df54c3f1f25425ce2 %}

#### 단순 스트림

{% gist Ghosttrio/19dfe7017bcb2fce0f701936f649684f %}


#### 병렬 스트림

{% gist Ghosttrio/993206e6e982a0884c556b5abf042e81 %}

#### CompletableFuture로 비동기 호출

CompletableFuture를 이용하여 구현

{% gist Ghosttrio/af12d0e74e9ffeb48579cfc4033fbfa6 %}

비동기 처리를 했는데 만족할만한 결과가 아니다. 

만약에 상점이 4개에서 5개로 늘어난다면 병렬 스트림보다 CompletableFuture가 아주 조금 빠르다.

CompletableFuture는 병렬 스트림으로 구현한 버전에 비해 작업에 이용할 수 있는 다양한 커스텀 Executor를 지정할 수 있다는 장점이 있다.

#### 커스텀 Executor를 이용한 CompletableFuture

{% gist Ghosttrio/34ae2b9d4d2b42f9cf3fc51950a3d998 %}

다섯 개의 상점을 검색할 때는 1022ms, 아홉 개의 상점을 검색할 때는 1022ms가 소요된다.

400개의 상점까지 이 같은 성능을 유지할 수 있다.

결국 애플리케이션 특성에 맞는 Executor를 만들어 CompletabelFuture를 활용하는 것이 바람직하다.

비동기 동작을 많이 사용하는 상황에서는 이 기법이 제일 효과적이다.
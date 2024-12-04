---
layout: post
title: Non block 코드 만들기
categories: Java
---

먼저 다음과 같은 상점 리스트가 있다고 해보자

```java
List<Shop> shops = Arrays.asList(
    new Shop("a"),
    new Shop("b"),
    new Shop("c"),
    new Shop("d")
);
```

모든 상점의 정보를 순차적으로 요청하는 메서드를 성능을 고려하여 개선하며 구현해보자.

#### 단순 스트림

- 스트림을 이용하여 구현

```java
static List<String> findPrices(String product) {
    return shops.stream()
            .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
            .collect(Collectors.toList());
}
```

- 성능 측정

```java
long start = System.nanoTime();
System.out.println(findPrices("test"));
long duration = (System.nanoTime() - start) / 1_000_000;
System.out.println("Done in " + duration + " msecs");

// 4032
```


#### 병렬 스트림

- 병렬 스트림을 이용하여 구현

```java
static List<String> findPricesWithParallel(String product) {
    return shops.parallelStream()
            .map(shop -> String.format("%s price is %.2f", shop.getName(), shop.getPrice(product)))
            .collect(Collectors.toList());
}
```

- 성능 측정

```java
long start1 = System.nanoTime();
System.out.println(findPricesWithParallel("test"));
long duration1 = (System.nanoTime() - start1) / 1_000_000;
System.out.println("Done in " + duration1 + " msecs");

// 1180ms
```


#### CompletableFuture로 비동기 호출

- CompletableFuture를 이용하여 구현

```java
static List<String> findPricesWithCompletableFuture(String product) {
    List<CompletableFuture<String>> collect = shops.stream()
            .map(shop -> CompletableFuture.supplyAsync(
                    () -> shop.getName() + " price is " + shop.getPrice(product)))
            .collect(Collectors.toList());

    return collect.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList());
}
```

- 성능 측정

```java
long start2 = System.nanoTime();
System.out.println(findPricesWithCompletableFuture("test"));
long duration2 = (System.nanoTime() - start2) / 1_000_000;
System.out.println("Done in " + duration2 + " msecs");
// 2005ms
```

- 비동기 처리를 했는데 만족할만한 결과가 아니다. 
- 만약에 상점이 4개에서 5개로 늘어난다면 병렬 스트림보다 CompletableFuture가 아주 조금 빠르다.
- CompletableFuture는 병렬 스트림 버전에 비해 작업에 이용할 수 있는 다양한 커스텀 Executor를 지정할 수 있다는 장점이 있다.

#### 커스텀 Executor를 이용한 CompletableFuture
- 커스텀 Executor, CompletableFuture를 이용하여 구현

```java
public static Executor CustomThread() {
    return Executors.newFixedThreadPool(Math.min(shops.size(), 1000), new ThreadFactory() {
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            t.setDaemon(true);
            return t;
        }
    });
}
```

```java
static List<String> findPricesWithExecutor(String product) {
    List<CompletableFuture<String>> collect = shops.stream()
            .map(shop -> CompletableFuture.supplyAsync(
                    () -> shop.getName() + " price is " + shop.getPrice(product), CustomThread()))
            .collect(Collectors.toList());

    return collect.stream()
            .map(CompletableFuture::join)
            .collect(Collectors.toList());
}
```

- 성능 측정

```java
long start3 = System.nanoTime();
System.out.println(findPricesWithExecutor("test"));
long duration3 = (System.nanoTime() - start3) / 1_000_000;
System.out.println("Done in " + duration3 + " msecs");
// 1022ms
```

- 다섯 개의 상점을 검색할 때는 1022ms, 아홉 개의 상점을 검색할 때는 1022ms가 소요된다.
- 400개의 상점까지 이 같은 성능을 유지할 수 있다.

```java
// 많은 상점을 추가했을 때 성능

Done in 64617 msecs

Done in 6075 msecs

Done in 6097 msecs

Done in 1042 msecs // 성능이 유지가 된다.
```

- 결국 애플리케이션 특성에 맞는 Executor를 만들어 CompletabelFuture를 활용하는 것이 바람직하다.
- 비동기 동작을 많이 사용하는 상황에서는 이 기법이 제일 효과적이다.
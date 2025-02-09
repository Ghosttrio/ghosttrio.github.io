---
layout: post
title: Random과 SecureRandom
categories: Backend
---

SonarQube를 이용해서 코드를 검토하던 중 Random 클래스를 사용하는 부분에 경고가 나왔다.

왜 그런지 알아보자.

## Random 클래스

Random() 메서드는 의사난수 생성기로, 예측 가능한 방식으로 난수를 생성한다.

따라서 보안이 중요한 애플리케이션에서 Random 클래스를 사용하는 것은 위험할 수 있다.

Random 클래스는 System.currentTimeMillis()나 System.nanoTime()을 기반으로 시드(seed)를 초기화하는데, 이러한 값들은 시간이 흐르면 예측할 수 있는 패턴이 발생할 수 있다.

예를 들어, 두 개의 Random 객체가 동일한 시점에 생성되면 동일한 시드를 공유하게 되어 예측할 수 있는 것이다.

```java
import java.util.Random;

public class Main {
    public static void main(String[] args) {
        Random random = new Random(42);
        for (int i = 0; i < 5; i++) {
            System.out.println(random.nextInt(100));
        }
    }
}
```
위와 같은 코드를 실행하면 같은 결과가 계속 나온다.

```java
// 똑같은 값만 나옴
30
63
48
84
70
```

결국 Random 클래스의 난수 생성은 시드 값에 의존하며, 시드 값이 고정되거나 예측 가능하면 생성된 난수도 예측할 수 있다.

## SecureRandom 클래스

SecureRandom은 보안 시스템의 하드웨어나 운영체제의 보안 엔진에서 제공되는 진정한 무작위성을 기반으로 난수를 생성한다. 

SecureRandom은 시스템에서 발생하는 무작위 데이터를 모은 저장소, 엔트로피 풀을 사용해서 난수를 생성한다.

또한, SecureRandom은 암호학적 난수 생성기로 설계되 어 있다. Random 클래스의 의사 난수 생성기는 예측 가능한 반면, 암호학적 난수 생성기는 암호학적으로 안전한 알고리즘을 사용하여 난수를 예측하는 것이 매우 어렵다.

```java
import java.security.SecureRandom;

public class Main {
    public static void main(String[] args) {
        SecureRandom secureRandom = new SecureRandom();

        for (int i = 0; i < 5; i++) {
            System.out.println(secureRandom.nextInt(100));
        }
    }
}
```

```java
// 여러 번 실행해도 결과가 같지 않다.

// 1번 실행
86
42
9
61
21

// 2번 실행
63
15
62
87
66
```

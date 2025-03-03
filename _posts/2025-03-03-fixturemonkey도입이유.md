---
layout: post
title: Fixture Monkey 도입 과정
categories: Test
---

프로젝트에 Fixture Monkey를 도입 과정을 알아보자.

### Fixture란?
주로 Test Fixture 라고 많이 이야기하는 Fixture 대해서 알아보자.

Fixture란 테스트 환경을 준비하거나 특정 조건을 설정하는 데 사용되는 코드나 데이터를 의미한다.

BDD 테스트의 given when then 단계의 given 단계라고 할 수 있다.

보통 이러한 데이터는 setup 단계에서 정의하거나 단위 테스트 안에서 작성하는 경우가 많다.

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class SampleTest {

    @Test
    void sample1() {
        // given, fixture 생성
        Member a = new Member("a", 10);
        Member b = new Member("b", 10);

        // when
        int result = a.getAge() + b.getAge();

        // then
        assertEquals(20, result);
    }
}
```
위와 같이 Member 인스턴스 a, b를 Test Fixture라고 한다.

### Fixture 작성의 문제점
위의 예제를 봤을 때 Fixture 작성의 문제점은 딱히 없어보인다.

단순히 두 개의 인스턴스를 생성하고 각 값을 대입한 후 테스트에 사용한다.

하지만 이러한 Fixture가 많이 필요하고, 복잡한 상태에, 데이터의 파라미터도 많다면 문제가 발생한다.

예시로 알아보자.

위의 예제에서 Member 객체를 수정해보자. 지금은 Member 객체의 필드가 2개로 구성되어 있는데, 이를 10개로 늘려보자.
```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class SampleTest {

    @Test
    void sample1() {
        // given, fixture 생성
        Member a = new Member("a", 10, "a", "b", "c", "d", "e", "f", "g", "e");
        Member b = new Member("b", 10, "a", "b", "c", "d", "e", "f", "g", "e");

        // when
        int result = a.getAge() + b.getAge();

        // then
        assertEquals(20, result);
    }
}
```
단순 필드만 몇 개 추가했을 뿐인데, 머리가 아파온다.

제대로 데이터가 들어갔는지 확인하기도 어렵고, 가독성도 떨어진다.

이러한 Fixture가 여러 테스트 코드에 걸쳐 필요하다고 한다면 복잡함은 배가 된다.

위의 데이터는 간단한 구조라 그나마 괜찮지만 복잡한 구조를 포함하고 있다면 더욱 만들기 어려워진다.

예를 들어, Member 객체에 Address 객체를 포함시켜보자.

```java
public class Address {
    private Location location;
    private Double latitude;
    private Double longitude;
    // getter, setter
}

...

public class Location {
  private String first;
  private String second;
  // getter, setter
}
```
```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;

public class SampleTest {

    @Test
    void sample1() {
        // given, fixture 생성
        Location locationA = new Location("a", "b");
        Address addressA = new Address(locationA, 10.0, 20.0);
        Member a = new Member("a", 10, "a", "b", "c", "d", "e", "f", "g", "e", addressA);

        Location locationB = new Location("a", "b");
        Address addressB = new Address(locationB, 10.0, 20.0);
        Member b = new Member("b", 10, "a", "b", "c", "d", "e", "f", "g", "e", addressB);

        // when
        int result = a.getAge() + b.getAge();

        // then
        assertEquals(20, result);
    }
}
```
이렇게 복잡한 객체가 Fixture 안에 포함되면 Fixture를 만들기 복잡해진다.

이러한 Fixture가 List와 같은 자료구조 형태라면 더더욱 복잡해진다.

마지막으로 가장 큰 문제는 변경에 취약하다는 것이다.

예를 들어, Member의 첫 번째 필드를 String -> Long으로 변경한다라고 가정해보자.

그렇다면 모든 테스트 코드의 Fixture를 일일이 수정해야 한다.

@BeforeEach와 같은 곳에서 정의한다면 그나마 낫지만 그래도 번거롭다.

문제점을 정리해보자.
- Fixture 작성에 비용(시간)이 많이 발생한다.
- 필드가 많은 경우 추가적인 비용이 발생하고 가독성이 떨어진다.
- 다른 객체를 포함하거나 자료구조를 가진 복잡한 Fixture를 작성하기 번거롭다.
- 데이터 객체가 변경된다면 Fixture를 일일이 수정해야 한다.

이러한 문제점을 해결하기 위해 Fixture Monkey를 도입한다.

### Fixture Monkey란?
[Fixture Monkey](https://naver.github.io/fixture-monkey/v1-1-0/)는 Naver에서 개발한 테스트 데이터 생성 라이브러리이다.

테스트 데이터를 자동 생성해주고 테스트 코드 간소화에 도움을 준다.

먼저 의존성을 추가해주자.

```gradle
testRuntimeOnly("org.junit.platform:junit-platform-launcher:{version}")
testImplementation("com.navercorp.fixturemonkey:fixture-monkey-starter:1.1.9")
```

fixture monkey를 사용하려면 먼저 팩토리를 만들어야 한다.

```java
@Test
void fixtureMonkeyTest() {
    FixtureMonkey monkey = FixtureMonkey.create();
}
```

여기서 단일 객체 생성을 원한다면 giveMeOne(클래스) 메서드를 사용하면 된다.

Fixture 생성 전략에 따라 Fixture로 만들 객체에 @Getter, @Setter, 생성자가 필요할 수도 있다.

아래와 같이 생성해주면 라이브러리가 알아서 해당 객체 안의 필드를 채워서 인스턴스를 생성해준다.

```java
@Test
void fixtureMonkeyTest() {
    FixtureMonkey monkey = FixtureMonkey.create();
    Member member = monkey.giveMeOne(Member.class);
    System.out.println(member.getName()); // ㋰볽
}
```

List와 같은 자료구조 Fixture도 쉽게 만들 수 있다.

```java
@Test
void listFixtureTest() {
    FixtureMonkey monkey = FixtureMonkey.create();
    List<Member> members = monkey.giveMe(Member.class, 3);
}
```

또한, 위의 직접 Fixture를 생성할 때의 문제점인 변경에 취약한 문제도 사라진다.

데이터 객체를 변경해도 Fixture Monkey 코드에서 직접 수정해야 할 것이 없기 떄문에 이 문제가 해결된다.

물론, 꼭 필요한 데이터를 직접 입력할 수도 있다.

```java
@Test
void selectDataFixtureTest() {
    Member member = monkey.giveMeBuilder(Member.class)
            .set("name", "testName")
            .sample();
    System.out.println(member.getName()); // testName
}
```

추가로, 계속해서 FixtureMonkey.create()를 하지 않으려면 추상 클래스를 정의해놓고 사용해도 된다.

```java
import com.navercorp.fixturemonkey.FixtureMonkey;
import com.navercorp.fixturemonkey.api.introspector.BeanArbitraryIntrospector;
import com.navercorp.fixturemonkey.api.introspector.BuilderArbitraryIntrospector;
import com.navercorp.fixturemonkey.api.introspector.FailoverIntrospector;
import com.navercorp.fixturemonkey.api.introspector.FieldReflectionArbitraryIntrospector;

import java.util.Arrays;

public abstract class FixtureSupport {
    public static FixtureMonkey monkey = FixtureMonkey.builder()
            .objectIntrospector(new FailoverIntrospector(
                    Arrays.asList(
                            FieldReflectionArbitraryIntrospector.INSTANCE,
                            BeanArbitraryIntrospector.INSTANCE,
                            BuilderArbitraryIntrospector.INSTANCE
                    )
            ))
            .defaultNotNull(true)
            .build();
}
```

이렇게 Fixture Monkey를 도입함으로써, 직접 Fixture를 생성했을 때의 문제를 해결했다.
- Fixture 작성에 비용(시간)이 많이 발생한다. -> 간단하게 Fixture를 생성할 수 있게 됨
- 필드가 많은 경우 추가적인 비용이 발생하고 가독성이 떨어진다. -> 단순히 monkey.giveMeXXX로 작성해서 비용 감소와 가독성 향상
- 다른 객체를 포함하거나 자료구조를 가진 복잡한 Fixture를 작성하기 번거롭다. -> 복잡한 객체를 Fixture Monkey가 알아서 생성해준다.
- 데이터 객체가 변경된다면 Fixture를 일일이 수정해야 한다. -> 변경이 되더라도 테스트 코드에는 변경이 없음

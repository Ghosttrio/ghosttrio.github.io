---
layout: post
title: Kotlin data class
categories: Kotlin
---

### 코틀린 data class란?
먼저 코틀린의 data class에 대해서 알아보자.

코틀린 data class란 데이터를 저장하는 데 사용되는 클래스로, 자동으로 여러 기능을 제공하는 클래스다.

```kotlin
data class Person(val name: String, val age: Int)
```

일반적으로 데이터를 표현하는 객체를 생성할 때 유용하게 사용된다.

data class는 자동으로 메서드들이 생성되는데, toString(), equals(), hashCode(), copy(), componentN()이 있다.

data class를 자바에서 표현하자면 다음과 같다.

```java
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    // copy(), 구조 분해 메서드 추가
}
```

data class는 구조 분해 선언을 제공한다. 각각의 프로퍼티에 대응하는 지역 변수를 정의하는 간결한 구문을 제공한다.

```kotlin
val (name, age) = Person("name", 10)

println(name)
```

### data class와 불변 객체

불변 객체는 한 번 생성된 후 그 상태나 값이 변경될 수 없는 객체를 말한다.

불변 객체는 객체의 내용을 수정하거나 업데이트하는 것이 불가능한 대신에 새로운 객체를 만들어야 한다.

불변 객체는 여러 스레드에서 동시에 접근하더라도 값이 바뀌지 않기 때문에, 멀티스레드 환경에서 유리하다.

코틀린 data class를 불변으로 만들려면, 모든 프로퍼티를 val로 만든다.

그리고 data class는 copy() 메서드를 제공해서 편리하게 불변 객체를 활용할 수 있게 해준다.

객체를 메모리상에서 직접 바꾸는 대신 복사본을 만들어, 원본과 다른 생명주기를 가지며, 일부 프로퍼티 값을 바꾸거나 복사본을 제거해도 프로그램 원본을 참조하는 다른 부분에 영향을 끼치지 않는다.

```kotlin
data class Person(val name: String, val age: Int)

fun main() {
  val person1 = Person("test", 25)

  val person2 = person1.copy(age = 26)

  println(person1)  // Person(name=test, age=25)
  println(person2)  // Person(name=test, age=26)
}
```


### copy() 메서드 주의점 : 얕은 복사

copy() 메서드는 얕은 복사를 하기 때문에 data class의 프로퍼티가 기본형이 아니라면 주의해야 한다.

```kotlin
data class Person(val name: String, var age: Int)

fun main() {
    val person1 = Person("test", 25)

    val person2 = person1.copy()
    person2.age = 10

    println(person1.toString()) // Person(name=test, age=25)
}
```

위 코드는 age의 타입이 Int 기본형이기 때문에 복사본의 age 값을 변경해도 원본의 값이 변경되지 않는다.

```kotlin
data class Person(val name: String, val friends: MutableList<String>)

fun main() {
  val person1 = Person("a", mutableListOf("b", "c"))
  val person2 = person1.copy()
  person2.friends.add("d")
  println(person1) // Person(name=a, friends=[b, c, d])
}
```

객체 내의 필드 값들이 참조 타입일 경우 그 참조만 복사한다. 그래서 위 코드의 MutableList의 참조값만 복사하기 때문에 원본 객체의 값이 변경된다.

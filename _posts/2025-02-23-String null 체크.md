---
layout: post
title: String의 isEmpty(), isBlank(), null 체크
categories: Java
---

자바에서 String 문자열을 다룰 때 빈 값이나 null을 체크하는 기능을 사용하곤 한다.

각각을 알아보고, 정리해보자.

### isEmpty()
String 클래스의 isEmpty() 메서드는 문자열이 빈 문자열("")인지 확인한다.

내부 구조를 확인해보면, 문자열의 길이가 0인지를 확인한다.
```java
...
@Override
public boolean isEmpty() {
  return value.length == 0;
}
...
```

따라서 빈 문자열의 길이는 0이니 true가 출력되고 나머지는 false가 출력된다.

```java
String str = "";
str.isEmpty(); // true
```

isEmpty()로 null 값도 체크할 수 있을까?

당연히 되지 않는다. 왜냐하면 isEmpty를 호출할 때 해당 String 객체의 참조값이 필요한데, null.isEmpty()를 사용하게 되면 NullPointException이 발생하기 때문이다.

```java
String str = null;
str.isEmpty(); // NullPointException
```

### isBlank()
String 클래스의 isBlank() 메서드는 문자열이 빈 문자열인지, 아니면 공백만으로 이루어져 있는지를 체크한다.

isBlank()는 isEmpty()에 공백도 체크하는 기능을 추가로 가지고 있는 것이라고 생각하면 된다.

```java
String str = "";
str.isBlank(); // true

String str2 = "   ";
str.isBlank(); // true
```

isBlank()의 내부 구현을 살펴보면 공백이 아닌 첫 번째 인덱스와 전체 길이가 같은지 체크한다.

```java
public boolean isBlank() {
    return indexOfNonWhitespace() == length();
}
```

isEmpty()와 마찬가지로 null 값에 사용하면 NullPointException이 발생한다.

### String == null

String 값을 직접 null로 비교하면 null 값을 체크할 수 있다.

```java
String str = null;

boolean result = (a == null); // true
```

String str = null이라는 코드는 str의 참조를 null로 둔다는 것이고 a == null은 두 값의 참조를 비교하기 때문에, 좌항의 값은 a의 참조인 null이 되고 우항의 값은 null이니 값이 같아 true가 출력되게 된다.

Objects 클래스의 isNull, nonNull 메서드를 통해서도 체크할 수 있다.

```java
String str = null;
Objects.isNull(a); // true
Objects.nonNull(a); // false
```

### String.equals(null)

String 값에 equals() 메서드를 사용하면 isBlank() isEmpty()와 마찬가지로 NullPointException이 발생한다.

### 정리
- null 체크를 할 때는 다음을 사용하자.
  - Objects.isNull()
  - Objects.nonNull()
  - String == null
- 빈 문자열을 체크할 때는 다음을 사용하자.
  - isEmpty()
  - isBlank()
- 빈 문자열을 체크하고 공백만으로 이루어져 있는지 체크할 때는 다음을 사용하자.
  - isBlank()


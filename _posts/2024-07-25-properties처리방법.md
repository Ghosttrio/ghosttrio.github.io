---
layout: post
title: properties 값을 가져오는 방법
categories: Spring
---

스프링에서 application.properties나 application.yml 파일에 설정된 값들을 가져오는 다양한 방법을 알아보자.

코드에 하드코딩하지 않고 properties 파일에 저장한 값을 읽어들이는 방법을 사용하는 이유는 다양하다.

1. 환경에 따른 설정 분리
- 하드코딩된 값은 코드에서 수정해야 하기 때문에 환경마다 다르게 설정하려면 코드 자체를 수정해야 한다.
- properties 파일에 설정하고 profile 설정을 하면 개발, 테스트, 운영 환경 등 각 환경에 따라 다른 설정 파일을 만들어 관리할 수 있다.

2. 보안 강화
- 중요한 정보(데이터베이스 비밀번호, AWS 키, 토큰 비밀번호 등)를 코드에 하드코딩하면 보안상 위험이 있다.
- 외부 설정 파일을 사용하면 .gitignore 에서 설정하여 형상관리에서 제거할 수도 있고 따로 파일을 관리할 수도 있다.

그러면 사용 방법들을 알아보자.

### Environment

Environment 객체를 주입해서 getProperty 메서드로 properties 값을 가져올 수 있다.

```java

@Autowired
private Environment environment;

@Test
void envTest() {
    String id = environment.getProperty("test.id");
    String password = environment.getProperty("test.password");

    assertEquals(id, "testId");
    assertEquals(password, "testPassword");
}

```

### @Value

@Value 어노테이션으로 properties 값을 가져올 수 있다. 

@Value("${프로퍼티}") 이런식으로 ${}로 감싸는 형태로 작성한다.

```java

@Value("${test.id}")
private String id;
@Value("${test.password}")
private String password;

@Test
void valueTest() {
    assertEquals(this.id, "testId");
    assertEquals(this.password, "testPassword");
}


```

### @ConfigurationProperties 

@ConfigurationProperties는 설정값을 객체 형태로 바인딩할 때 사용.

개인적으로 이 방법이 코드도 깔끔하다고 생각한다.

```java
@Component
@ConfigurationProperties(prefix = "test")
public class TestProperties {
    private String id;
    private String password;

    public String getId() {
        return id;
    }

    public String getPassword() {
        return password;
    }

    public void setId(String id) {
        this.id = id;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}


@Autowired
private TestProperties testProperties;

@Test
void configTest() {
    assertEquals(testProperties.getId(), "testId");
    assertEquals(testProperties.getPassword(), "testPassword");
}
```
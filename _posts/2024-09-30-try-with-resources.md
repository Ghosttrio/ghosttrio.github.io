---
layout: post
title: try-finally 보다는 try-with-resources를 사용하라
categories: Java
---

try-with-resources는 자바 7에서 도입된 문법으로, 자원을 자동으로 관리하고 닫을 수 있도록 도와주는 기능이다.

try-with-resources는 AutoCloseable 인터페이스를 구현하는 객체들을 사용하여 자원을 자동으로 닫을 수 있게 해준다.

AutoCloseable 인터페이스는 close() 메서드를 가지고 있는데, 이 메서드는 객체가 더 이상 필요하지 않을 때 호출되어 자원을 정리한다.

### 기본 문법

```java

try (/* AutoCloseable을 구현한 객체 */) {
    // 자원을 사용하는 코드
} catch (Exception e) {
    // 예외 처리
}

```

### 예제

데이터베이스를 사용하는 코드

```java

public class Main {
    public static void main(String[] args) {
        // 여러 자원을 try-with-resources로 사용
        try (
                BufferedReader br = new BufferedReader(new FileReader("csv.txt"));
                Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "user", "password");
                Statement stmt = conn.createStatement();
                ResultSet rs = stmt.executeQuery("SELECT * FROM my_table")
        ) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }

            while (rs.next()) {
                System.out.println("레코드: " + rs.getString("column_name"));
            }
        } catch (IOException | SQLException e) {
            e.printStackTrace();
        }
    }
}

```

직접 AutoCloseable 구현

```java

public class ResourceTest implements AutoCloseable {

    private String name;

    public ResourceTest(String name) {
        this.name = name;
        System.out.println(name + " 자원이 열렸습니다.");
    }

    @Override
    public void close() {
        System.out.println(name + " 자원이 닫혔습니다.");
    }
    
    public void execute() {
        System.out.println("실행");
    }

    public static void main(String[] args) {
        try (ResourceTest r1 = new ResourceTest("Res1");
             ResourceTest r2 = new ResourceTest("Res2")) {
            r1.execute();
            r2.execute();
        } catch (Exception e) {
            System.out.println("예외 발생: " + e.getMessage());
        }
    }
}

```

### try-with-resources를 사용해야 하는 이유

전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally가 쓰였다.

하지만 예외는 try 블록과 finally 블록 모두에서 발생할 수 있어, 두 번째 예외가 첫 번째 예외를 완전히 먹어벼려, 스택 추적 내역이 남지 않아서 디버깅이 어려워 진다.

또한 명시적으로 finally 블록을 만들어야 해서 실수로 작성하지 않게 되는 문제를 제거해준다.
---
title: N+1 문제 해결하기
categories: [Java]
tags: [JPA, N+1]
---

# 배경

회사에서 기존 프로젝트에 DDD 구조를 적용하면서 Entity의 연관 관계를 수정한다든지, 도메인 로직을 적용한더든지 등 Entity를 수정하면서 기존 쿼리가 정상적으로 나가는지? 혹시 N+1 문제가 일어나진 않는지 불안감이 생겼다. 그러면서 N+1 문제를 확실히 정리하고 기존 연관관계에서 도메인 로직으로 전환했을 때 사이드 이펙트가 생기진 않는지 확인하고자 정리한다.

# N+1이란?

### N+1 문제가 무엇인가?

- ORM 기술에서 특정 객체를 대상으로 수행한 쿼리가 해당 객체가 가지고 있는 연관관계 또한 조회하게 되면서 N번의 추가적인 쿼리가 발생하는 문제를 말한다.
- 연관 관계가 설정된 엔티티를 조회할 경우에 조회된 데이터 갯수(n) 만큼 연관관계의 조회 쿼리가 추가로 발생하여 데이터를 읽어오는 현상
- 연관 관계에서 발생하는 이슈로 연관 관계가 설정된 엔티티를 조회할 경우에 조회된 데이터 갯수(n) 만큼 연관관계의 조회 쿼리가 추가로 발생하여 데이터를 읽어오게 된다. 즉, 1번의 쿼리를 날렸을 때 의도하지 않은 N번의 쿼리가 추가적으로 실행되는 것이다. 이를 N+1 문제라고 한다.

### 왜 N+1 문제가 발생하는지?

- N+1문제가 발생하는 근본적인 원인은 관계형 데이터베이스와 객체지향 언어간의 패러다임 차이로 인해 발생합니다. 객체는 연관관계를 통해 레퍼런스를 가지고 있으면 언제든지 메모리 내에서 **Random Access**를 통해 연관 객체에 접근할 수 있지만 RDB의 경우 **Select 쿼리**를 통해서만 조회할 수 있기 때문입니다.
- JPA Repository로 find 시 실행하는 첫 쿼리에서 하위 엔티티까지 한 번에 가져오지 않고, 하위 엔티티를 사용할 때 추가로 조회하기 때문에.
- JPQL은 기본적으로 글로벌 Fetch 전략을 무시하고 JPQL만 가지고 SQL을 생성하기 때문에.

### 언제 발생하는지?

- JPA Repository를 활용해 인터페이스 메소드를 호출할 때(Read 시)
- 1:N 또는 N:1 관계를 가진 엔티티를 조회할 때 발생
- JPA Fetch 전략이 EAGER 전략으로 데이터를 조회하는 경우
- JPA Fetch 전략이 LAZY 전략으로 데이터를 가져온 이후에 연관 관계인 하위 엔티티를 다시 조회하는 경우
- 즉시로딩인 경우
    1. JPQL에서 만든 SQL을 통해 데이터를 조회
    2. 이후 JPA에서 Fetch 전략을 가지고 해당 데이터의 연관 관계인 하위 엔티티들을 추가 조회
    3. 2번 과정으로 N + 1 문제 발생
- LAZY(지연 로딩)인 경우
    1. JPQL에서 만든 SQL을 통해 데이터를 조회
    2. JPA에서 Fetch 전략을 가지지만, 지연 로딩이기 때문에 추가 조회는 하지 않음
    3. 하지만, 하위 엔티티를 가지고 작업하게 되면 추가 조회가 발생하기 때문에 결국 N + 1 문제 발생

### 코드로 알아보자

- Member와 Team 엔티티를 생성하자

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;
    @Column(name = "member_name")
    private String name;

    @ManyToOne
    private Team team;

    @Builder
    public Member(String name, Team team) {
        this.name = name;
        this.team = team;
    }

}

```

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
public class Team {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "team_id")
    private Long id;
    @Column(name = "team_name")
    private String name;

    @OneToMany(mappedBy = "team", fetch = FetchType.Eager)
    private List<Member> members = new ArrayList<>();

    @Builder
    public Team(String name) {
        this.name = name;
    }

}

```

- 테스트 코드를 작성해주자
    - Team 10개, Member 10개를 저장해준다

```java

@BeforeEach
void setup(){
    System.out.println("========================================================================================");
    List<Team> teams = new ArrayList<>();
    for(int i = 0; i < 10; i++){
        teams.add(Team.builder().name("team" + i).build());
    }
    teamRepository.saveAll(teams);

    List<Member> members = new ArrayList<>();
    for(int i = 0; i < 10; i++){
        members.add(Member.builder().name("member" + i).team(teams.get(i)).build());
    }
    memberRepository.saveAll(members);

    entityManager.clear();
    System.out.println("========================================================================================");
}
```

- 저장된 결과를 조회해보자

```java
@Test
void NPlusOne테스트() {
    /**
     *  원하는 결과 -> team findAll 쿼리 1번
     *  실제 결과 -> team findAll 쿼리 1번 team에 해당하는 member 쿼리 10번
     */
    teamRepository.findAll();
}
```

![1](/assets/img/docimg/0508/1.png)

- 원하는 결과는 쿼리 1번인데 team에 해당하는 member에 대한 쿼리 10번이 추가로 나간다.

- Fetch.MODE를 Eager에서 Lazy로 바꾸면 해결될까?

```java
@OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
private List<Member> members = new ArrayList<>();
```

```java
@Test
void Fetch를_Eager에서_Lazy로변환(){
    /**
     *  원하는 결과 -> team findAll 쿼리 1번
     *  실제 결과 -> team findAll 쿼리 1번
     *  해결된 것일까? -> 아님
     */
    /**
     * team의 member를 조회할 때 다시 n+1 문제 발생
     * 원하는 결과 -> team findAll 쿼리 1번
     * 실제 결과 -> team findAll 쿼리 1번 team에 해당하는 member 쿼리 10번
     */
    List<Team> teams = teamRepository.findAll();
    teams.forEach(t -> t.getMembers().forEach(m -> m.getName()));
}
```

![2](/assets/img/docimg/0508/2.png)

![3](/assets/img/docimg/0508/3.png)

### 해결방법

- Fetch Join을 이용하는 방법
    
    ```java
    @Query("select t from Team t join fetch t.members")
    List<Team> findAllFetch();
    
    ```
    

```java
@Test
void 해결방안1_fetch_join(){
    /**
     *  원하는 결과 -> team findAll 쿼리 1번
     *  실제 결과 -> team findAll 쿼리 1번
     */
    List<Team> teams = teamRepository.findAllFetch();
    teams.forEach(t -> t.getMembers().forEach(m -> m.getName()));
}
```

![4](/assets/img/docimg/0508/4.png)

- EntityGraph 어노테이션

```java
@EntityGraph(attributePaths = "members")
@Query("select t from Team t")
List<Team> findAllEntityGraph();
```

```java
@Test
void 해결방안2_Entity_Graph(){
    /**
     *  원하는 결과 -> team findAll 쿼리 1번
     *  실제 결과 -> team findAll 쿼리 1번
     */
    List<Team> teams = teamRepository.findAllEntityGraph();
    teams.forEach(t -> t.getMembers().forEach(m -> m.getName()));
}
```

![5](/assets/img/docimg/0508/5.png)

- FetchMode.SUBSELECT

```java
@Fetch(FetchMode.SUBSELECT)
@OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
private List<Member> members = new ArrayList<>();
```

```java
    @Test
    void 해결방안3_FetchMode_SUBSELECT(){
        /**
         *  원하는 결과 -> team findAll 쿼리 1번
         *  실제 결과 -> team findAll 쿼리 1번
         */
        List<Team> teams = teamRepository.findAll();
        teams.forEach(t -> t.getMembers().forEach(m -> m.getName()));
    }
```

![6](/assets/img/docimg/0508/6.png)

- Batch Size

```java
    @BatchSize(size=10)
    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
    private List<Member> members = new ArrayList<>();
```

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 10
```

```java
@Test
void 해결방안4_Batch_Size(){
    /**
     *  원하는 결과 -> team findAll 쿼리 1번
     *  실제 결과 -> team findAll 쿼리 1번
     */
    List<Team> teams = teamRepository.findAll();
    teams.forEach(t -> t.getMembers().forEach(m -> m.getName()));
}
```

![7](/assets/img/docimg/0508/7.png)

- 도메인로직

```java
@SpringBootTest
@Transactional
public class DomainNPlusOneTest {
    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private TeamRepository teamRepository;
    @Autowired
    private EntityManager entityManager;

    @BeforeEach
    void setup(){
        System.out.println("========================================================================================");
        List<Team> teams = new ArrayList<>();
        for(int i = 0; i < 10; i++){
            teams.add(Team.builder().name("team" + i).build());
        }
        teamRepository.saveAll(teams);

        List<Member> members = new ArrayList<>();
        for(int i = 0; i < 10; i++){
            members.add(Member.builder().name("member" + i).teamId(teams.get(i).getId()).build());
        }
        memberRepository.saveAll(members);

        entityManager.clear();
        System.out.println("========================================================================================");
    }

    @Test
    void 도메인로직테스트() {
        /**
         *  원하는 결과 -> team findAll 쿼리 1번
         *  실제 결과 -> team findAll 쿼리 1번
         *  애초에 Team만으로 Member를 조회할 수 없다.
         *  Join을 이용하자
         */

        teamRepository.findAllWithMember();
    }
}

```

```java

@Query("select m.name from Team t left outer join Member m on m.teamId = t.id")
List<String> findAllWithMember();

```
![8](/assets/img/docimg/0508/8.png)


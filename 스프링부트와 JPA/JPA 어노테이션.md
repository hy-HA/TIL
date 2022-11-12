# JPA 어노테이션
참고 ] https://cjw-awdsd.tistory.com/46

- @Entity
    - @Entity 어노테이션이 선언된 클래스를 DB 테이블과 매핑합니다.

- @Table
    - 엔티티와 매핑할 테이블을 지정
    - 속성
        - name : 매핑할 테이블 이름 
            - 예. @Table(name = "orders")
            - 생략 시 엔티티 이름 사용
        - schema
        - catalog

- @Column
    - 객체 필드를 테이블 컬럼과 매핑
    - 필드 속성을 지정할 때 사용
    - 속성
        - name : 필드와 매핑할 테이블의 컬럼명 지정
            - 예. @Column(name = "member_id")
            - default는 필드이름으로 대체
        - length : 문자 길이 제약 조건 설정
        - nullable : DDL 생성 시 설정 값에 따라 null 가능여부를 설정
        - unique : DDL 생성 시 간단하게 한 컬럼에 유니크 제약 조건을 설정
        ```
        @Column
        private String name;
        ```

- @PersistenceContext   
    1. EntityManager를 빈으로 주입할 때 사용하는 어노테이션
        - 스프링에서는 영속성 관리를 위해 EntityManager가 존재
        - 그래서 스프링 컨테이너가 시작될 때 EntityManager를 만들어서 빈으로 등록하는데,
        - 이 때 스프링이 만들어둔 EntityManager를 주입받을 때 사용
    2. @PersistenceContext로 지정된 프로퍼티에 아래 두 가지 중 한 가지로 EntityManager를 주입
        - EntityManagerFactory에서 새로운 EntityManager를 생성하거나
        - Transaction에 의해 기존에 생성된 EntityManager를 반환
    3. @PersistenceContext를 사용해야 하는 이유
        - EntityManager를 사용할 때 주의해야 할 점은 여러 쓰레드가 동시에 접근하면 동시성 문제가 발생하여 쓰레드 간에는 EntityManager를 공유해서는 안됨
            - 일반적으로 스프링은 싱글톤 기반으로 동작하기에 빈은 모든 쓰레드가 공유함
            - 그러나 @PersistenceContext으로 EntityManager를 주입받아도 동시성 문제가 발생하지 않음
        - 동시성 문제가 발생하지 않는 이유는
            - 스프링 컨테이너가 초기화되면서 @PersistenceContext으로 주입받은 EntityManager를 Proxy로 감쌈
            - 그리고 EntityManager 호출 시 마다 Proxy를 통해 EntityManager를 생성하여 Thread-Safe를 보장
    3. 참고] Autowired 와 PersistenceContext
        - @Autowired는 스프링 빈을 주입
        - @PersistenceContext는 JPA 스펙에서 제공하는 기능인데, 영속성 컨텍스트를 주입하는 표준 애노테이션
참고] https://batory.tistory.com/497


- @Id | @GeneratedValue
- @Id
    - 특정 속성을 기본키로 설정
        ```
        @Id
        private Long id;
        ```
    - @Id 어노테이션만 적게될 경우 기본키값을 직접 부여해줘야
    - 하지만 보통 DB를 설계할 때는 기본키는 직접 부여하지 않고 Mysql AUTO_INCREMENT처럼 자동 부여되게끔 한다.
        ```
        @Id
        @GeneratedValue
        private Long id;
        ```

- @GeneratedValue
    - 기본값을 DB에서 자동으로 생성
        - 전략
            1. @GeneratedValue(startegy = GenerationType.IDENTITY)
                - 기본 키 생성을 DB에 위임 (Mysql)
            2. @GeneratedValue(startegy = GenerationType.SEQUENCE)	
                - DB시퀀스를 사용해서 기본 키 할당 (ORACLE)
            3. @GeneratedValue(startegy = GenerationType.TABLE)
                - 키 생성 테이블 사용 (모든 DB 사용 가능)
            4. @GeneratedValue(startegy = GenerationType.AUTO)
                - 선택된 DB에 따라 JPA가 자동으로 전략 선택


@Embedded
@Embeddable

@JsonIgnore
--
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
@OneToMany(mappedBy = "member")
@ManyToOne(fetch = LAZY)
private Member member;

@JoinColumn(name = "member_id")
--
@NoArgsConstructor(access = AccessLevel.PROTECTED)


# 롬복
@Getter @Setter
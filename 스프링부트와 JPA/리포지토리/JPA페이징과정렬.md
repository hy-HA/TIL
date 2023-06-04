| [참고](https://velog.io/@hoyun7443/JPA%EC%97%90%EC%84%9C-Pageable%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%ED%8E%98%EC%9D%B4%EC%A7%95%EA%B3%BC-%EC%A0%95%EB%A0%AC)
| [참고](https://hudi.blog/spring-data-jpa-pagination/)
| 참고 김영한

# 페이징
- 많은 정보, 이를테면 게시판에 존재하는 수백 수천개의 게시글과 같은 정보들을 페이지로 나눠 효과적으로 정보를 제공하게 하는 역할을 한다.
- 사용자가 요청했을 때 데이터베이스에 있는 수천, 수만, 수백만 줄의 데이터를 모두 조회하여 제공한다면 서버의 부하가 굉장히 클 것
- 이를 방지하기 위해서 대부분의 서비스에서는 데이터를 일정 길이로 잘라 그 일부분만을 사용자에게 제공하는 방식을 사용함
- 사용자는 현재 보고 있는 데이터의 다음, 이전 구간 혹은 특정 구간의 데이터를 요청하고, 전달한 구간에 해당하는 데이터를 제공받는다.
- 페이지네이션이 적용된 예시로는 페이스북, 인스타그램 등의 소셜미디어, 혹은 네이버 카페의 게시물 목록 조회 등


# 페이징 방법
1. page 관련 쿼리를 파라미터로 받아서 직접 처리
2. JPA, Spring Data 프로젝트에서는 효과적으로 페이징을 처리하는 방법 제공
    - JpaRepository 인터페이스의 상속 다이어그램
    - PagingAndSortingRepository
        - JpaRepository의 부모 인터페이스
        - 페이징과 소팅이라는 기능을 제공

# 순수 JPA 페이징과 정렬
- DB 벤더별로 페이지네이션을 처리하기 위한 쿼리는 천차만별
    - MySQL
        - offset , limit 으로 상대적으로 간단히 처리가 가능
    - Oracle
        - 상당히 복잡
    - JPA
        - 여러 DB 벤더별 방언(dialect)을 추상화하여 하나의 방법으로 페이지네이션을 구현할 수 있도록 제공해준다.

**Sample Code**

- 검색 조건: 나이가 10살
- 정렬 조건: 이름으로 내림차순
- 페이징 조건: 첫 번째 페이지, 페이지당 보여줄 데이터는 3건
```java
리포지토리 >>

public List<Member> findByPage(int age, int offset, int limit) {
    return em.createQuery("select m from Member m where m.age = :age order by m.username desc")
        .setParameter("age", age)
        .setFirstResult(offset)
        .setMaxResults(limit)
        .getResultList();
}

public long totalCount(int age) {
    return em.createQuery("select count(m) from Member m where m.age = :age", Long.class)
        .setParameter("age", age)
        .getSingleResult();

}
```
- setFirstResult() 
    - 결과를 조회해올 시작 지점을 지정
    - MySQL로 따지면 offset
- setMaxResults()
    - 조회해올 최대 데이터 수를 지정
    - MySQL로 따지면 limit 

- JPA가 추상화를 해준 덕분에 DB 벤더 상관없이 페이지네이션을 구현할 수 있게 되었다.



# 스프링 데이터 JPA 페이징과 정렬
- JPA로 페이지네이션 기능을 구현하는 작업은 생각보다 까다롭다. 
    1. 전체 데이터 개수를 가져와서 전체 페이지를 계산하고
    2. 현재 페이지가 첫번째 페이지인지, 마지막 페이지인지 계산하고, 
    3. 예상치 못한 페이지 범위를 요청받았을 때 예외처리를 해야한다. 
- 이것을 setFirstResult() 와 setMaxResults() 로만 구현하기에는 번거롭고 불편하다.
- Spring Data JPA은 이런 페이지네이션도 추상화되어 있다. 
    - 페이지 크기와 페이지 순서만 전달하면, 데이터베이스에서 해당 페이지에 해당하는 데이터만 가져올 수 있다.

### **Pageable과 Sort 인터페이스**
- 페이징과 정렬 파라미터
- Sort
    - org.springframework.data.domain.Sort 
    - 정렬 기능 
- Pageable
    - org.springframework.data.domain.Pageable 
    - 페이징을 제공하는 중요한 인터페이스 
    - Pageable 인터페이스로 파라미터를 넘기면 페이징을 사용할 수 있게된다.
    - 페이징 기능 (내부에 Sort 포함)

### **Page와 Slice 인터페이스**
- 특별한 반환 타입
- Page
    - org.springframework.data.domain.Page 
    - 기본적인 반환 메서드로 여러 반환 타입 중 하나
    - 추가 count 쿼리 결과를 포함하는 페이징
- Slice
    - org.springframework.data.domain.Slice 
    - 추가 count 쿼리 없이 다음 페이지만 확인 가능(내부적 으로 limit + 1조회)
- List (자바 컬렉션)
    - 추가 count 쿼리 없이 결과만 반환

```
Page 인터페이스>>

public interface Page<T> extends Slice<T> {
    int getTotalPages(); //전체 페이지 수
    long getTotalElements(); //전체 데이터 수
    <U> Page<U> map(Function<? super T, ? extends U> converter); //변환기
}

---
Slice 인터페이스>>

public interface Slice<T> extends Streamable<T> {
    int getNumber();             //현재 페이지
    int getSize();               //페이지 크기
    int getNumberOfElements();   //현재 페이지에 나올 데이터 수
    List<T> getContent();        //조회된 데이터
    boolean hasContent();        //조회된 데이터 존재 여부
    Sort getSort();              //정렬 정보
    boolean isFirst();           //현재 페이지가 첫 페이지인지 여부
    boolean isLast();            //현재 페이지가 마지막 페이지인지 여부
    boolean hasNext();           //다음 페이지 여부
    boolean hasPrevious();       //이전 페이지 여부
    Pageable getPageable();      //페이지 요청 정보
    Pageable nextPageable();     //다음 페이지 객체
    Pageable previousPageable(); //이전 페이지 객체
    <U> Slice<U> map(Function<? super T, ? extends U> converter); //변환기
}
```

### **PageRequest 객체**
- PageRequest 객체
    - Pageable 인터페이스를 상속받는 구현체. 
    - 이 정보에는 정렬 정보, 페이지 offset, page와 같은 정보가 담겨있다.
```
PageRequest pageRequest = PageRequest.of(offset, limit, Sort.by(direction,"updatedDate"));
return quizRepository.findByMemberId(memberId, pageRequest);
```

### **Sample Code**

```
Page<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 

Slice<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안 함

List<Member> findByUsername(String name, Pageable pageable); //count 쿼리 사용 안 함

List<Member> findByUsername(String name, Sort sort);
```


### **Sample Code - 반환타입 : Page**

- 검색 조건: 나이가 10살
- 정렬 조건: 이름으로 내림차순
- 페이징 조건: 첫 번째 페이지, 페이지당 보여줄 데이터는 3건

**리포지토리**
- 두 번째 파라미터로 받은 Pageable 은 인터페이스다. 
- 따라서 실제 사용할 때는 해당 인터페이스를 구현한 org.springframework.data.domain.PageRequest 객체를 사용한다.
```
public interface MemberRepository extends Repository<Member, Long> {

    Page<Member> findByAge(int age, Pageable pageable);
}
```
**테스트 코드**
- PageRequest 생성자의 첫 번째 파라미터에는 현재 페이지를, 두 번째 파라미터에는 조회할 데이터 수를 입력한다. 
- 여기에 추가로 정렬 정보도 파라미터로 사용할 수 있다. 참고로 페이지는 0부터 시작한다.
-  주의
    - Page는 1부터 시작이 아니라 0부터 시작이다.
```java
@Test
public void page() throws Exception {
//given
    memberRepository.save(new Member("member1", 10));
    memberRepository.save(new Member("member2", 10));
    memberRepository.save(new Member("member3", 10));
    memberRepository.save(new Member("member4", 10));
    memberRepository.save(new Member("member5", 10));

//when
    PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
    Page<Member> page = memberRepository.findByAge(10, pageRequest);

//then
List<Member> content = page.getContent(); //조회된 데이터 
assertThat(content.size()).isEqualTo(3); //조회된 데이터 수 
assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수 
assertThat(page.getNumber()).isEqualTo(0); //페이지 번호 
assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호 
assertThat(page.isFirst()).isTrue(); //첫번째 항목인가? 
assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
}
```

### **Sample Code - 반환타입 : Slice**
- 모바일에서 많이 쓰는 기능으로 count기능이 없이 더보기 버튼을 클릭하면 그때 더 보여주는 방식
**리포지토리**

```
public interface MemberRepository extends Repository<Member, Long> {

    Slice<Member> findByAge(int age, Pageable pageable);
}
```

**테스트 코드**
- Slice에는 getTotalElements, getTotalPages메서드가 없다
```
//when
PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));
Slice<Member> page = memberRepository.findByAge(10, pageRequest);

//then
List<Member> content = page.getContent(); //조회된 데이터 
assertThat(content.size()).isEqualTo(3); //조회된 데이터 수 
//assertThat(page.getTotalElements()).isEqualTo(5); //전체 데이터 수 
assertThat(page.getNumber()).isEqualTo(0); //페이지 번호 
//assertThat(page.getTotalPages()).isEqualTo(2); //전체 페이지 번호 
assertThat(page.isFirst()).isTrue(); //첫번째 항목인가? 
assertThat(page.hasNext()).isTrue(); //다음 페이지가 있는가?
```

## 성능을 최적화하기 위한 count 쿼리 변경
- join할 필요없이 아래와 같이 검색해도 동일한 count수를 반환함
```java
@Query(value = “select m from Member m”, countQuery = “select count(m.username) from Member m”)
Page<Member> findMemberAllCountBy(Pageable pageable);
```

## Top, First 사용 참고
- List<Member> findTop3By();
- https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit- query-result

## 페이지를 유지하면서 엔티티를 DTO로 변환하기
```
Page<Member> page = memberRepository.findByAge(10, pageRequest);
Page<MemberDto> dtoPage = page.map(m -> new MemberDto());
```

# Pageable과 PageRequest
- Pageable 과 PageRequest
    - Spring Data에서 제공하는 페이지네이션 정보를 담기 위한 인터페이스와 구현체
    - 페이지 번호와 단일 페이지의 개수를 담을 수 있다.

- 이를 Spring Data JPA 레포지토리의 파라미터로 전달하여, 반환되는 엔티티의 컬렉션에 대해 페이징할 수 있다.

### Pageable의 ofSize() 메소드

- ofSize() 메소드
    - Pageable 인터페이스의 스태틱 메소드
    - 이 메소드를 사용하면 아래와 같이, PageRequest 를 생성할 수 있다. 
    - 단, 페이지 번호는 0으로 고정되고 페이지 사이즈만 설정할 수 있다.
    ```
    Pageable.ofSize(10);
    ```
### PageRequest 생성
- 아래와 같이 정적 팩토리 메소드를 사용하여 PageRequest를 생성할 수 있다.
    ```
    PageRequest page = PageRequest.of(0, 10);
    ```
- 첫번째 파라미터는 페이지 순서, 두번째 파라미터는 단일 페이지의 크기
- 페이지 순서는 0부터 시작함에 유의하자.


# 반환 타입에 따른 페이징 결과
- Spring Data JPA에는 반환 타입에 따른 각기 다른 결과를 제공한다.

### Page<T> 타입
Page<T> 타입을 반환 타입으로 받게 된다면 offset과 totalPage를 이용하여 서비스를 제공할 수 있게된다.
Page<T>는 아래와 같은 일반적인 게시판 형태의 페이징에서 사용된다.
여기서 특히 중요한 정보는 총 페이지의 수 이다.
Page<T>는 총 페이지 수와 같은 totalPage를 포함하여 반환한다.
Page<T> 타입은 count 쿼리를 포함하는 페이징으로 카운트 쿼리가 자동으로 생성되어 함께 나간다.
### Slice<T> 타입
- Slice<T> 타입을 반환 타입으로 받게 된다면 더보기 형태의 페이징에서 사용된다.
### List<T> 타입
- List반환 타입은 가장 기본적인 방법으로 count 쿼리 없이 결과만 반환한다.

# 조회 결과 정렬
- Sort 클래스 혹은 Sort 의 내부 enum 클래스인 Direction 을 사용하여 정렬을 설정
- 정렬 방향을 지정하는 방법은 아래와 같이 다양하다.
```
PageRequest.of(0, 10, Sort.by("price").descending());
PageRequest.of(0, 10, Sort.by(Direction.DESC, "price"));
PageRequest.of(0, 10, Sort.by(Order.desc("price")));
PageRequest.of(0, 10, Direction.DESC, "price");
```
- PageRequest.of() 의 세번째 인자로 Sort 혹은 Direction 을 전달하면 된다. 
- 전달된 "price" 는 정렬 기준이 되는 컬럼 이름이다. 
- 위 코드는 모두 내림차순으로 정렬된 결과를 받아올 때 사용
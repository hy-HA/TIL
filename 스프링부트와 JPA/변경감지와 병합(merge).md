## 준영속 엔티티
> 주의: 변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만, 병합을 사용하면 모든 속성이 변경된다. 병합시 값이 없으면 null 로 업데이트 할 위험도 있다. (병합은 모든 필드를 교체한다.)

- 영속성 컨텍스트가 더는 관리하지 않는 엔티티를 말한다.
    - 예. itemService.saveItem(book) 에서 수정을 시도하는 Book 객체
        - Book 객체는 이미 DB 에 한번 저장되어서 식별자가 존재
        - 이렇게 임의로 만들어낸 엔티티도 기존 식별자를 가지고 있으면 준 영속 엔티티로 볼 수 있음
## 준영속 엔티티를 수정하는 2가지 방법 
## 1. 변경 감지 기능 사용 - 이게 더 나은 방법
- 영속성 컨텍스트에서 엔티티를 다시 조회한 후에 데이터를 수정하는 방법
- 트랜잭션 안에서 엔티티를 다시 조회, 변경할 값 선택 트랜잭션 커밋 시점에 변경 감지(Dirty Checking) 이 동작해서 데이터베이스에 UPDATE SQL 실행
```
ItemService>>

@Transactional
public String update(Long itemId, Book param) { //param: 파리미터로 넘어온 준영속 상태의 엔티티
    Item findItem = itemRepository.findOne(itemId); //id를 기반으로 실제 db에 있는 영속상태의 엔티티 가져옴
    findItem.setPrice(param.getPrice()); //데이터를 수정한다. 
    findItem.setPrice(param.getPrice()); //데이터를 수정한다. 
    // 이제 @Transactional에 의해 트랜잭션에 commit됨
    // JPA는 플러시를 날림 - 영속성 컨티티(findItem) 중에서 변경된 애들을 찾음
}
```

### 더 나은 방법
- 엔티티를 변경할 때는 항상 변경 감지를 사용하세요
- 컨트롤러에서 어설프게 엔티티를 생성하지 마세요.
- 트랜잭션이 있는 서비스 계층에 식별자( id )와 변경할 데이터를 명확하게 전달하세요.(파라미터 or dto)  
    - 트랜잭션 안에서 엔티티를 조회해야 영속상태로 조회됨. 거기에 값을 변경해야 dirty-checking(변경감지)가 일어남
- 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경하세요. 
- 트랜잭션 커밋 시점에 변경 감지가 실행됩니다.

```
ItemService>>

@Transactional
public void updateItem(Long itemId, String name, int price) { 
    Item findItem = itemRepository.findOne(itemId); 
    findItem.setName(name);
    findItem.setPrice(price);
    // 이제 @Transactional에 의해 트랜잭션에 commit됨
    // JPA는 플러시를 날림 - 영속성 컨티티(findItem) 중에서 변경된 애들을 찾음
}
```
```
ItemController>>

@PostMapping("items/{itemId}/edit")
public String updateItem(@PathVariable String itemId, @ModelAttribute("form") BookForm form) {
    itemService.updateItem(itemId, form.getName(), form.getPrice());

    //Book book = new Book();
    //book.setId(form.getId());
    //book.setName(form.getName());
    //book.setPrice(form.getPrice());
    //itemService.saveItem(book);
}
```

## 2. 병합( merge ) 사용
- 병합은 준영속 상태의 엔티티를 영속 상태로 변경할 때 사용하는 기능이다. @Transactional
```
void update(Item itemParam) { //itemParam: 파리미터로 넘어온 준영속 상태의 엔티티 
    Item mergeItem = em.merge(item);
}
```
- 동작방식
1. merge()를실행한다.
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다.
    - 2-1. 만약 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하고, 1차 캐시에 저장한다.
3. 조회한 영속 엔티티( mergeMember )에 member 엔티티의 값을 채워 넣는다. (member 엔티티의 모든 값
을 mergeMember에 밀어 넣는다. 이때 mergeMember의 “회원1”이라는 이름이 “회원명변경”으로 바
뀐다.)
4. 영속 상태인 mergeMember를 반환한다.
- 간단히 정리
1. 준영속 엔티티의 식별자 값으로 영속 엔티티를 조회한다.
2. 영속 엔티티의 값을 준영속 엔티티의 값으로 모두 교체한다.(병합한다.)
3. 트랜잭션 커밋 시점에 변경 감지 기능이 동작해서 데이터베이스에 UPDATE SQL이 실행


# [JPA] Dirty Checking(더티 체킹)
> ### ✔Dirty Checking(더티 체킹)이란?
>  - Transaction 안에서 엔티티의 변경이 일어나면 **변화가 있는 모든 엔티티 객체를 데이터베이스에 자동으로 반영**해준다.   
> - 이 때, 변화가 있다는 기준은 **최초 조회 상태**이다. 즉, 변경을 감지해서 DB에 반영한다.

> - **_데이터 베이스에 변경 데이터를 저장하는 시점?_**
>   1. Transaction commit 시점
>   2. EntityManager flush 시점
>   3. JPQL 사용 시점
> - **_더티 체킹이 일어나는 환경?_**
>   1. 영속 상태(Managed) 안에 있는 엔티티인 경우   
>     (DB 반영 전 첫 생성 엔티티(비영속), detach된 엔티티(준영속)는 변경이 DB에 반영되지 않는다)
>   2. Transaction 안에서 엔티티를 변경하는 경우
> - **_Transaction을 사용하는 방식?_**
>   1. Service Layer에서 @Transaction을 사용하는 경우
>   2. EntityTransaction을 이용해서 트랜잭션 범위를 지정하고 사용하는 경우   


아래 코드는 Service Layer에서 ```@Transaction```을 사용하는 경우이다.
```java
@Transactional
public Long update(Long id, PostsUpdateRequestDto requestDto) {
 
  Posts posts = postsRepository.findById(id).orElseThrow(
    () -> new IllegalArgumentException("해당 게시글이 없습니다. id = " + id));
 
  posts.update(requestDto.getTitle(), requestDto.getContent());
 
  return id;
 
}
```
게시글 정보를 update 하는 코드에 **save 메서드가 따로 없는데도 정상적으로 update**가 된다.   
이는 Dirty Checking 덕분인데, JPA에서는 엔티티를 조회하면 해당 엔티티의 조회 상태 그대로 **스냅샷**을 만들어둔다.   
그리고 **트랜잭션이 끝나는 시점에 이 스냅샷과 비교하여 다른 점이 있다면 Update Query를 DB로 전달**한다.   

#### ```@DynamicUpdate```   
> Dirty Checking의 update 쿼리는 모든 필드 **update가 default**이다.   
> **변경 부분만 update**하고 싶다면 엔티티 최상단에 ```@DynamicUpdate```를 선언해주면 된다.


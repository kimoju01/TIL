# [JPA] Fetch 전략

> JPA에는 두 가지 로딩 기법이 존재한다.
> 1. 즉시 로딩(fetch = FetchType.EAGER) : 엔티티를 조회할 때 연관된 엔티티도 함께 조회한다.
> 2. 지연 로딩(fetch = FetchType.LAZY) : 연관된 엔티티를 실제 사용할 때 조회한다.

예를 들어 한 게시글이 있다고 하자.
```
🍀
작성자 : kim
제목 : 안녕하세요
내용 : 반가워요

* 댓글(펼치기)
> 반갑습니다
> 저도 반가워요
```

**한 명의 유저는 여러 개의 게시글**을 작성할 수 있고 **하나의 게시글에는 여러 개의 댓글**이 작성될 수 있다.

그렇다면 게시글과 유저는 **ManyToOne** 관계이고 (Many = Board, One = User)   
게시글과 댓글은 **OneToMany** 관계이다. (One = Board, Many = Reply)

Board Entity는 아래와 같다.
```java
@Entity
public class Board {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  
  @Column(nullable = false)
  private String title;
  
  @Lob
  private String content;
  
  @ManyToOne(fetch = FetchType.EAGER)
  @JoinColumn(name = "userId")
  private User user;
  
  @OneToMany(mappedBy = "board", fet = FetchType.LAZY)
  private List<Reply> reply;

}
```

> ✔   
> *__@ManyToOne(fetch = FetchType.EAGER)__   
> @JoinColumn(name = "userId")   
> private User user;*   
> - Board 테이블을 조회(Select)했을 때, 해당하는 유저 정보를 즉시 가져온다  
> - 왜 즉시 가져오는가? 게시글을 조회했을 때 해당하는 유저 정보는 한명 뿐이니까 부담이 덜 하다!
> - 그렇기 때문에 @ManyToOne과 @OneToOne의 기본 fetch 전략은 EAGER 타입이다. (굳이 적지 않아도 EAGER로 적용된다.)   

> ✔   
> *__@OneToMany(mappedBy = "board", fetch = FetchType.LAZY)__   
> private List<Reply> reply;*   
> - @OneToMany, @ManyToMany의 기본 fetch 전략은 LAZY 타입이다
> - Board 테이블을 조회했을 때, 해당하는 댓글이 1개 일 수도 있지만 10000개 일 수도 있기 때문이다
> - 펼치기 기능으로 댓글을 불러온다면 LAZY 타입을 사용해도 괜찮지만 펼치기 기능이 없고 바로 댓글이 나타난다면 EAGER 타입으로 바꿔서 사용할 것

 
 
## 결론
- 컬렉션에서는 무조건 지연로딩을 사용하고 단일 엔티티는 즉시 로딩을 사용한다
- 그러나 모든 연관관계에 지연로딩을 사용하고 필요한 곳에만 즉시 로딩을 사용하는 것을 추천하는 것이 성능적으로 좋다

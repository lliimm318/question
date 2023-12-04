# DTO (Data Transfer Object)
- 계층 간 데이터 교환을 하기 위해 사용하는 객체
- DTO는 로직을 가지지 않는 순수한 데이터 객체(getter & setter 만 가진 클래스)

### 동작과정
1. 클라이언트에서 데이터를 받을 때, DTO에 넣어서 전송
2. 해당 DTO를 받은 서버가 DAO를 이용하여 데이터베이스로 데이터를 넣음
<br/></br>

## 그런데 혹시...

DTO를 만들 때!<br/>

~~~
WHITESHIP,
BLACKSHIP,
REDSHIP,
~~~

이 있을 때.. 각각 공통으로 가지고 있는 value도 있지만, 다르게 가지고 있는 value들도 있을 경우 DTO를 어떤식으로 가져가는게 효율적일까요 ??

### 해결 방안
1. 상속으로 해결한다 !
2. 하나의 DTO에 모든 value를 다 넣는다

## 내가 생각한 결론
정확히 어디에 어떻게 쓰이냐에 따라 다를 것 같다! 단순 Http Request/Response Body에 사용된다면 그냥 중복을 발생시키는 것이고, <br/> 
비즈니스 로직상 공통 타입으로 주고 받으며 런타임에 타입 체크 후 서로 다른 동작을 해야한다면 상속이나 확장 쪽으로 풀어야 할듯 !

> 근데 그런 니즈가 없으면 일단 중복으로 만들어놓고 장기간 관찰하면서 통합의 필요성이 있는지 확인한다. 필요하다면 리펙토링 해주고!

### 팩토리로 구현 (추상화)

나는 일단 구현단은 다 팩토리로 처리했었다. (디자인 패턴 공부해서 걍 써보고 싶엇음) 

조금 다른 내용이긴 하지만 아래 글도 읽어봤다. 추상화도 명확한 이유로 정말 필요하다고 생각되는 or 추정되는 시점에 하는게 바람직하지 않을까 라는 생각임 (미친 추상화 Box()를 보고 느낀 점)

- (우리는 너무 많은 수준의 추상화를 사용했다) https://news.hada.io/topic?id=11447 

 기사는 기술의 추상화 수준이 증가하고 있음을 논하며, 이를 비행기 조종사가 더 이상 비행기의 메카닉을 이해할 필요가 없는 항공 진화와 비교한다.
일부 댓글러들은 이러한 계층화가 분야가 성숙해짐에 따른 자연스러운 진행이며, 걱정할 일이 아니라고 주장한다.
 다른 일부는 많은 기술 전문가들이 특정 도구를 사용하는 방법만 알고 있지만, 그들이 어떻게 작동하는지에 대한 깊은 이해가 부족하다는 우려를 표현한다.

ㅇㅇ 맞는듯 솔직히 팩토리 굳이? 이런 느낌이긴 했어

## 어떻게 넘겨요?

그런데 DTO를 넘길 때 어떻게 넘기는게 효율적일까..? 라는 고민. 나눠놓은 형태로 넘기는게 좋을지 하나의 DTO에 다 set해서 넘길지.. 등등 고민하다가

이런거면 통짜로 잡는게 아니라 dto를 여러개로 쪼개고 각각 전달하거나 쪼갠걸 조합해서 받는 방법도 있을 듯. TS쪽에는 타입의 union이 강력해서 해당 방식을 꽤 적극적으로 사용하더라. (그 외 언어에선 좀 불편한걸로 알고 잇음)

super dto는 API 스펙을 분리할 수 없기 때문에 안 됨!!!!

하나의 DTO에 세부 DTO의 속성을 때려박으면...

- @NotBlank 등의 유효성검사 기능이 분리되지 않아 해당 기능을 사용하기가 어렵
- 엔티티와 DTO를 분리하는 이유 중 하나이기도 하구

다른건 모르겠지만, 일단 나는 상속 비용 치루는게 개인적으로 너무 귀찮음.. 컴포지션 기법을 애용합시다. (뭔가 더 코드가 예쁜 것 같음. 물론 예쁜거 말고 다른 좋은 이유가 잇답니다)

### 컴포지션은 어떻게 해요?

컴포지션 궁금하면 ㄱㄱ
- https://dev-cool.tistory.com/22

A를 상속받는 A1, A2, A3 클래스가 있으면
B라는 클래스 만들어서 A에만 있는 속성, A1에만 있는 속성 이렇게 넣나요

#### 킹지피티가 만들어준 코틀린 예제

~~~ kotlin
// 상속을 사용한 예시
open class UserDto(
    val id: Long,
    val name: String
)

class ExtendedUserDto(
    id: Long,
    name: String,
    val email: String
) : UserDto(id, name)
~~~

~~~ kotlin
// 컴포지션을 사용한 예시
data class UserDto(
    val id: Long,
    val name: String
)

data class ExtendedUserDto(
    val user: UserDto,
    val email: String
)
~~~
> 위 예시에서는 ExtendedUserDto가 UserDto를 상속하는 대신, UserDto 타입의 user 프로퍼티를 갖도록 함으로써 컴포지션을 사용하였습니다. 이 방식을 사용하면 ExtendedUserDto가 UserDto의 내부 구현에 의존하지 않아도 되므로, 두 클래스가 더 독립적이고 유연하게 됩니다.


-> Request 객체를 저렇게는 안 써봤던 거 같은뎀...

코틀린에서.. 
{
  val somethings: List<Something>
}

구조에서 컬렉션 하위 Something에 대한 validation이 불가능해서 커스텀 밸리데이터 만들어야 함!

안에 데이터 클래스를 정의해서 마커 처럼 사용하기도 함

seald interface usercommand{
 
 val id: long
 
 data class rigsercommand(
 override val id : long
 val name : string) : usercommand
}

@field:Valid 로 했던거 같은데.. 음? -> 컬렉션에서 안먹혀

나는 요즘 각 매핑을 별도의 RestController로 추출하고 Request/Response 타입을 컨트롤러 내부에 inner class로 정의해서 저런 고민을 안했디

그리고 @NotNull같은거도 어노테이션 적용하려면 프로파티 nullable로 둬야 메시지 적용되어서 좀 지저분 함..

그냥 모든 Mapping이 별도 컨트롤러이고 그 컨트롤러의 내부에 해당 매핑을 위한 req, res 객체가 들어가는 구조입니당


알아서 내부 객체 필드 맵핑 잘 해줌! 저정도는 그냥 해줍니다. 컬렉션만 아니면 @field:Valid 붙이면 검증도 해주고요

컬렉션은 별도 밸리데이터 + 어노테이션 맹글어 손수 작업 해야합니다 ~~

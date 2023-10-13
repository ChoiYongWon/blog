> 본 글은 개념적인 내용은 생략하고 우리가 도메인을 구입했을 때 실제로 Name Server( 이하 NS )에 어떤 일들이 발생하는지 이해하고자 작성한다. 우리가 도메인을 구입하고 관리할 때 흔히 접하는 업체인 가비아와 AWS를 활용하여 설명한다.

## 도메인을 구입하였을 때 어떤 일이 벌어지나??

사용자가 가비아에서 `choiyongwon.me` 도메인을 구입하면 가비아는 최상위 도메인(TLD)인 me NS에게 다음과 같은 요청을 보낸다.

> `가비아 : choiyongwon.me 도메인은 내가 관리할게! 너의 (me NS) 레코드에 추가해줘!`

라는 요청을 보내게 되고, 각 NS의 상태는 그림 1과 같다.

<div class="img-container">
    <img class="img" src="https://i.imgur.com/dcHlpB7.png" alt=""/>
    <span class="caption">그림 1. 도메인 구입 후 각 NS의 상태.</span>
</div>

도메인을 구입하면 사용자는 도메인을 구입한 사이트에서 본인 도메인을 관리할 NS를 설정할 수 있다. 처음에는 가비아에서 `choiyongwon.me` 라는 도메인을 관리하기에 NS가 가비아로 설정되어있는데, 이 의미는 me NS에게 `choiyongwon.me 라는 질의를 받았을 때 ns.gabia.co.kr로 안내해! `라고 하는것과 같다. 그림 1에서 gabia NS는 도메인 `choiyongwon.me` 에 대한 레코드 테이블이다.

## 그럼 가비아에서 구매한 도메인을 AWS에서 관리하고 싶으면 어떻게 할까??

AWS도 가비아 처럼 NS가 존재한다. (Route53) 즉, me NS가 가르키고 있는 네임서버를 AWS의 NS로 변경해주면 된다. Route53을 써본 사람들은 알겠지만 도메인을 관리하는 영역을 하나의 Zone이라 한다. Route53에서 `choiyongwon.me` 이라는 Zone을 생성하고 가비아에서 `choiyongwon.me`에 대한 네임서버를 AWS의 NS 주소로 바꿔주면 된다. 이 과정을 가비아에서 하는 이유는 가비아에서 도메인을 구입했기 때문이다. ( 그림 2 참조. )

<div class="img-container">
    <img class="img" src="https://i.imgur.com/qYlzO47.png" alt=""/>
    <span class="caption">그림 2. 가비아에서 AWS NS로 변경 후 상태.</span>
</div>

이렇게 설정했을 때 `blog.choiyongwon.me` 에 대한 질의 과정은 그림 3와 같다.

<div class="img-container">
    <img class="img" src="https://i.imgur.com/QRL7kdW.png" alt=""/>
    <span class="caption">그림 3. 전체 DNS 동작 구조.</span>
</div>

1. 단말기에서 ISP Resolver로 `blog.choiyongwon.me`에 대한 질의를 한다. (ISP Resolver는 통신사에서 제공한다.)
2. ISP Resolver는 Root NS에게 질의를 한다.
3. Root NS는 `blog.choiyongwon.me`에서 최상위 계층인 me NS의 정보를 반환한다.
4. ISP Resolver는 Root NS의 응답을 바탕으로 다시 me TLD Name Server에게 질의를 한다.
5. 우리가 가비아에서 NS를 AWS로 설정하였기 때문에 me TLD NS는 AWS NS 정보를 반환한다.
6. ISP Resolver는 me NS의 응답을 바탕으로 다시 AWS NS에게 질의를 한다.
7. AWS NS는 저장되어있는 `choiyongwon.me` Zone 에 저장되어있는 레코드에서 서브 도메인인 blog에 매핑되어있는 주소를 반환한다.
8. ISP는 위 과정을 통해 얻은 `choiyongwon.me`에 대한 IP 주소를 단말기에게 알려준다.

### 참고 영상

<div class="video-wrapper">
<iframe width="560" height="315"  src="https://www.youtube.com/embed/sDXcLyrn6gU?si=wDiX0rMaqhFIsDw9" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

</div>

#network #cloud

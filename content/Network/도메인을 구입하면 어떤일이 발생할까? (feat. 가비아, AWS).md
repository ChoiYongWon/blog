
개념적인 내용은 생략하고 실제로 우리가 도메인을 구입하고 관리할 때 눈에 보이는 과정들을 설명하는 글이다.
본 글은 우리가 도메인을 구입하고 관리할 때 흔히 접하는 업체인 `가비아`와 `AWS`를 활용하여 설명한다.

가비아에서 도메인 `choiyongwon.me`을 구입했는데 두가지 의문이 든다. 

1. 어떻게 가비아가 me라는 TLD를 판매할 수 있는것인이지?   
2. 네임서버에서 제휴 관계를 맺은 여러 업체들이 가비아를 포함해서 존재하는데 만약 사용자가 가비아에서 도메인을 구입하였고, 이 도메인에 대해서 me 네임서버에 질의를 했을 때 어떻게 가비아 네임서버(이하 NS)로 안내해주나??


## 어떻게 가비아가 me라는 TLD를 판매할 수 있는것인지

첫번째 의문은 생각보다 간단했다. 가비아는 TLD와 제휴 협력 관계를 맺었기에 판매 할 수 있고, 다른 도메인 판매 업체들도 다 마찬가지이다.  

## 도메인을 구입하였을 때 어떤 일이 벌어지나??

가비아가 me TLD와 제휴 협력 관계를 맺었을 때, 사용자가 가비아에서 `choiyongwon.me` 도메인을 구입하면 가비아는 me NS에게 다음과 같은 요청을 보낸다.  

> `choiyongwon.me 도메인은 내가 관리할게! 너의 (me NS) 레코드에 추가해줘!`  

라는 요청을 보내게 된다. 다음 그림 참조.

![](https://i.imgur.com/38G1J1b.png)

그래서 우리는 가비아에서 도메인을 구입하면 NS를 설정할 수 있다 ( 다음 그림 참조. )  

![](https://i.imgur.com/IGXFHUW.png)

처음에는 가비아에서 `choiyongwon.me` 라는 도메인을 관리하기에 NS가 가비아로 설정되어있는데, 
위 그림의 의미는 me NS에게  `choiyongwon.me 라는 질의를 받았을 때 ns.gabia.co.kr로 안내해!` 라고 하는것과 같다. 

### 그럼 가비아에서 구매한 도메인을 AWS에서 관리하고 싶으면 어떻게 할까??

AWS도 가비아 처럼 NS가 존재한다. (Route53)
즉, me NS가 가르키고 있는 네임서버를 AWS의 NS로 변경해주면 된다.  
Route53을 써본 사람들은 알겠지만 도메인을 관리하는 영역을 하나의 `zone`이라 한다.
Route53에서 `choiyongwon.me` 이라는 `zone`을 생성하고 가비아에서 choiyongwon.me에 대한 네임서버 설정을 AWS의 네임서버 주소로 바꿔주면 된다. 이 과정을 가비아에서 하는 이유는 가비아에서 도메인을 구입했기 때문이다. ( 다음 그림 참조. )

![](https://i.imgur.com/p8W5jQk.png)



이렇게 설정했을 때 `blog.choiyongwon.me` 에 대한 질의 과정은 다음과 같다.

![](https://i.imgur.com/ZO8hnJm.png)


1. 단말기에서 ISP Resolver로 `blog.choiyongwon.me`에 대한 질의를 한다.
2. ISP Resolver는 Root NS에게 질의를 한다. (ISP Resolver는 통신사에서 제공한다.)
3. Root NS는 `blog.choiyongwon.me`에서 최상위 계층인 `me` NS의 정보를 반환한다.
4. ISP Resolver는 다시 me TLD Name Server에게 질의를 한다. 
5. 우리가 가비아에서 NS를 AWS로 설정하였기 때문에 me TLD NS는 AWS NS 정보를 반환한다.
6. ISP Resolver는 다시 AWS NS에게 질의를 한다.
7. AWS NS는 저장되어있는 `choiyongwon.me` 존에 저장되어있는 레코드에서 서브도메인인 blog에 매핑되어있는 주소를 반환한다.
8. ISP는 단말기에게 `choiyongwon.me`에 대한 IP 주소를 알려준다.


#network

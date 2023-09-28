
개념적인 내용은 생략하고 실제로 우리가 도메인을 구입하고 관리할 때 눈에 보이는 과정들을 설명하는 글이다.
본 글은 우리가 도메인을 구입하고 관리할 때 흔히 접하는 업체인 `가비아`와 `AWS`를 활용하여 설명한다.

가비아에서 도메인 `choiyongwon.me`을 구입했다. `여기서 어떻게 가비아가 me라는 TLD를 판매할 수 있는것인이지?` 라고 생각해봤을때 TLD와 제휴 협력 관계를 맺어야 판매 할 수 있다.  

그런데 또 의문이 든다. `me 네임서버에서 제휴 관계를 맺은 여러 업체들이 가비아를 포함해서 존재하는데 만약 사용자가 가비아에서 도메인을 구입하였고, 이 도메인에 대해서 me 네임서버에 질의를 했을 때 어떻게 가비아 네임서버(이하 NS)로 안내해주나??`   
이 질문에 대한 대답은 생각보다 간단했다. 제휴 협력 관계를 맺으면 사용자가`choiyongwon.me` 도메인을 구입하면 가비아에서 me NS의 `choiyongwon.me` 레코드를 가비아 네임서버로 변경한다. 즉, me NS에게 `choiyongwon.me` 라는 도메인을 질의 했을 때, 가비아의 NS로 안내한다.  

그래서 우리는 실제로 가비아에서 도메인을 구입하면 NS를 설정할 수 있는데 이는 실제로 `me` NS에 저장되며 `choiyongwon.me` 라는 질의를 받았을 때 안내하도록 하는 NS를 의미한다. 처음에는 가비아에서 `choiyongwon.me` 라는 도메인을 관리하기에 NS가 가비아로 설정되어있다.

![](https://i.imgur.com/rCszfdF.png)

`그럼 가비아에서 구매한 도메인을 AWS에서 관리하고 싶으면 어떻게 할까??`  

AWS에서도 Route53이라는 DNS를 지원하기에 본인 도메인을 입력하고 관리할 수 있다.  근데 도메인은 가비아에서 구매했기에 가비아에서 NS를 AWS의 NS로 수정해야된다. 즉, me NS에게 `choiyongwon.me` 도메인은 AWS NS한테 물어봐 라고 설정하는 과정이다.

이렇게 설정했을 때 `blog.choiyongwon.me` 에 대한 질의 과정은 다음과 같다.
1. 단말기에서 ISP Resolver로 `blog.choiyongwon.me`에 대한 질의를 한다.
2. ISP Resolver는 Root NS에게 질의를 한다. (ISP Resolver는 통신사에서 제공한다.)
3. Root NS는 `blog.choiyongwon.me`에서 최상위 계층인 `me` NS의 정보를 반환한다.
4. ISP Resolver는 다시 me TLD Name Server에게 질의를 한다. 
5. 우리가 가비아에서 NS를 AWS로 설정하였기 때문에 me TLD NS는 AWS NS 정보를 반환한다.
6. ISP Resolver는 다시 AWS NS에게 질의를 한다.
7. AWS NS는 저장되어있는 `choiyongwon.me` 존에 저장되어있는 레코드에서 서브도메인인 blog에 매핑되어있는 주소를 반환한다.


#network

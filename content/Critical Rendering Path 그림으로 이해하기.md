

> 본 글은 실습 시나리오와 그림을 통해 실제 CRP가 어떻게 진행되고, 단계별 상태를 풀어서 이해하기 쉽도록 작성한 글이다. 잘못된 내용이 있을시 yongwon4130@gmail.com으로 문의해주길 바란다.


## 개요
---
브라우저는 `Browser Engine`, `Network Engine`, `Rendering Engine`, `Javascript Engine` 등 다양한 엔진으로 구성되어있다.  그 중 성능에 가장 많은 영향을 미치는 `Rendering Engine`에서 발생하는 일을 알아보고자 한다. `Rendering Engine`은 `Network Engine`으로부터 받아온 HTML 문서를 최종적으로 화면에 렌더링해주는 역할을 하는데 이 일련의 과정을 `Critical Rendering Path` (이하 CRP) 라고 한다. CRP의 과정은 그림 1과 같다.  


<div class="img-container">
    <img class="img" src="https://i.imgur.com/a6uMzc4.png" alt=""/>
    <span class="caption">그림 1. Critical Rendering Path 과정.</span>
</div>



## 실습 환경 및 시나리오
---

브라우저에서 다음과 같은 HTML문서를 파싱한다고 가정한다.
유저는 웹사이트에 접속하여 `index.html`을 요청하고 이후 브라우저의 `Rendering Engine`이 처리하는 과정을 살펴본다.


```html {3,14,16} title="index.html"
<html lang="en">
   <head>
      <link rel="stylesheet" href="./style.css">
	  <meta charset="UTF-8">
	  <meta name="viewport" content="width=device-width, initial-scale=1.0">
	  <title>Document</title>
   </head>

   <body>
	  <header>
	     <h1>CRP 이해하기</h1>
	     <span>2023. 10. 13. 금요일</span>
	  </header>
	  <script src="./index.js" ></script>
	  <main>
	     <h2>본문입니다.</h2>
	    <img src="./image.jpg" alt="">
	     <span>부가 설명입니다.</span>
	  </main>
	  <footer>
	     <span>Copyright 2023. ChoiYongWon inc. all rights reserved. </span>
	  </footer>
   </body>
</html>
```



```css title="style.css"
body { font-size: 16px; }

header { font-weight: bold; }
h1 { font-size: 24px; }

main { color: gray; }
h2 { font-size: 20px; }

footer { display: none; }
```



```js title="index.js"
const el = document.getElementsByTagName('span')
for(let i=0;i<el.length;i++) {
   el[i].style.color = 'red'
}
```



## HTML 파싱 DOM, CSSOM Tree 생성
---

HTML 파싱은 HTML 문서를 위에서 아래로 절차적으로 파싱하여 DOM Tree와 CSSOM Tree를 만드는 과정이다. 전체 파싱 과정과 파싱중 위에 `index.html` 코드에 하이라이팅 된 3개의 리소스(`css`, `script`, `img`)를 어떻게 처리하는지 살펴보자.

- **Line 3**: `link` 태그를 만나면서 `style.css`를 서버에 요청하고, 응답을 받으면 병렬적으로 CSSOM을 생성한다. 이 때 HTML 파싱은 멈추지 않고 계속 진행한다. 생성된 CSSOM은 그림 2과 같다.


<div class="img-container">
    <img class="img" src="https://i.imgur.com/Bp4w6Zi.png" alt=""/>
    <span class="caption">그림 2. Line 3에서 생성된 CSSOM.</span>
</div>


- **Line 14**: `script` 태그를 만난다. 이 때 `Rendering Engine`의 HTML 파싱을 즉시 멈추고 서버에 `index.js`를 요청하고 `JS Engine`에게 권한을 위임하여 실행시킨다. 그 후 다시 `Rendering Engine`이 권한을 위임 받아 HTML을 파싱한다.  


 > `script`는 DOM과 CSSOM을 수정할 가능성이 있으므로 HTML 파싱 과정을 멈추고 진행한다. 이 때 Line 3에서 요청한 `style.css`의 CSSOM이 생성되지 않았다면, 생성될 때 까지 기다리고 `script`를 실행한다.

`index.js` 내의 코드 내용은  `document 객체 내의 모든 span 태그의 글자 색상을 red로 바꿔라` 인데,
`index.html`을 보면 문서 내의 `span` 태그는 총 2개가 있는걸 확인할 수 있다. (`Line 12, 18`) 다만 `index.js`가 실행되는 시점에서의 DOM Tree는 그림 3과 같으므로 `Line 12`의 `span` 태그에만 스타일이 적용된다.


<div class="img-container">
    <img class="img" src="https://i.imgur.com/OROqubt.png" alt=""/>
    <span class="caption">그림 3. Line 14 시점의 Dom Tree.</span>
</div>

`index.js`가 실행된 후 CSSOM의 상태는 그림 4와 같다.  

<div class="img-container">
    <img class="img" src="https://i.imgur.com/TSOgXPx.png" alt=""/>
    <span class="caption">그림 4. index.js 실행 후 CSSOM의 상태.</span>
</div>

계속 파싱을 진행한다.  

- **Line 16**: `img` 태그를 만난다. `image.jpg`를 서버에 요청한다. 이 때 파싱은 멈추지 않고 계속 진행한다.  

최종 생성된 DOM Tree는 그림 5와 같다.  


<div class="img-container">
    <img class="img" src="https://i.imgur.com/buvoo82.png" alt=""/>
    <span class="caption">그림 5. 최종 DOM Tree 모습.</span>
</div>

HTML 파싱 과정을 waterfall 방식으로 나타내면 그림 6와 같다.  
`css`와` image`는 병렬적으로 요청하는 반면 `script` 태그를 만났을때는 파싱을 잠시 멈추고 진행한다는 차이점이 있다. 이는 위에서 언급했듯이 `css`와 `image`는 DOM Tree를 생성하는데 아무런 영향을 주지 않지만 `script`는 파싱 도중에 DOM Tree를 직접적으로 수정할 수 있기 때문이다. `script`는 또한 파싱 도중에 CSSOM도 수정할 수 있는데, 이 때문에 `script`를 실행하기 전에 CSSOM이 아직 생성이 안됐다면 완료될때까지 기다렸다가 실행이 된다. 


<div class="img-container">
    <img class="img" src="https://i.imgur.com/WnfFySr.png" alt=""/>
    <span class="caption">그림 6. HTML 파싱 Waterfall.</span>
</div>


HTML 파싱이 끝났고 Dom Tree가 생성되었다! 이 때, document 객체에서 `DomContentLoaded` 이벤트가 발생한다. `DomContentLoaded` 이벤트는 그림 7에서 파란색 실선과 같은데, `index.js` 스크립트가 다 실행되고 호출되는 모습을 확인할 수 있는데 이는 **Line 14**의 스크립트를 동기적으로 실행하고 **Line 24**까지 파싱을 끝내고 Dom Tree를 생성한 시점이다.. 빨간색 실선은 나머지 과정(`HTML 파싱` ~ `paint`)이 다 끝난 후 호출되는 Load 이벤트이다.

<div class="img-container">
    <img class="img" src="https://i.imgur.com/tlqPuvn.png" alt=""/>
    <span class="caption">그림 7. Chrome Network 탭에서의 HTML 파싱 과정.</span>
</div>


## Render Tree
---
`작성중`

## Reference

---

[Ryan Seddon: So how does the browser actually render a website | JSConf EU 2015](https://www.youtube.com/watch?v=SmE4OwHztCc)  
[Kruno: How browsers work | JSUnconf 2017](https://www.youtube.com/watch?v=0IsQqJ7pwhw)  
[How to render in WebKit](https://www.youtube.com/watch?v=RVnARGhhs9w)  
[브라우저는 웹페이지를 어떻게 그리나요? - Critical Rendering Path](https://m.post.naver.com/viewer/postView.nhn?volumeNo=8431285&memberNo=34176766)  
[Critical Rendering Path](https://web.dev/articles/critical-rendering-path?hl=ko)

#frontend

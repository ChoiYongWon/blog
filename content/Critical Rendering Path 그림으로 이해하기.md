---
date: 2023-10-15
---


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


```html {3,14,17} title="index.html"
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

HTML 파싱은 HTML 문서를 위에서 아래로 절차적으로 파싱하여` DOM Tree`와 `CSSOM Tree`를 만드는 과정이다. 전체 파싱 과정과 파싱 중에 `index.html` 코드에서 하이라이팅된 3개의 리소스 (`css`, `script`, `img`)를 처리하는 방식을 살펴보자.

### stylesheet 리소스

> **Line 3**
 > ```html 
 > <link rel="stylesheet" href="./style.css">
 > ```

- **Line 3**: `link` 태그를 만나면서 `style.css`를 서버에 요청하고, 응답을 받으면 병렬적으로 `CSSOM Tree`을 생성한다. 이 때 HTML 파싱은 멈추지 않고 계속 진행한다. 생성된 `CSSOM Tree`은 그림 2과 같다.


<div class="img-container">
    <img class="img" src="https://i.imgur.com/Bp4w6Zi.png" alt=""/>
    <span class="caption">그림 2. Line 3에서 생성된 CSSOM Tree.</span>
</div>

### script 리소스

> **Line 14**
 > ```html 
 > <script src="./index.js" ></script>
 > ```

- **Line 14**: `script` 태그를 만나면 HTML 파싱이 즉시 멈추고 `index.js`를 서버에 요청하고 `JS Engine`에게 실행을 위임한다. 그 후 다시 `Rendering Engine`이 권한을 위임 받아 HTML파싱을 계속한다.  

`script`는 `DOM Tree`와 `CSSOM Tree`을 수정할 가능성이 있으므로 HTML 파싱 과정을 멈추고 실행된다. 이 때 **Line 3**에서 요청한 `style.css`의 `CSSOM Tree`가 생성되지 않았다면, 생성될 때 까지 기다리고 `script`를 실행한다. `index.js` 의 코드는  `document 객체 내의 모든 span 태그의 글자 색상을 red로 변경` 의 내용을 포함한다. `index.html`을 살펴보면 문서 내에 총 2개의 `span` 태그가 있음을 알 수 있는데 (Line 12, 18), `index.js`가 실행되는 시점에서의 `DOM Tree`는 그림 3과 같으므로 **Line 12**의 `span` 태그에만 스타일이 적용된다.


<div class="img-container">
    <img class="img" src="https://i.imgur.com/OROqubt.png" alt=""/>
    <span class="caption">그림 3. Line 14 시점의 Dom Tree.</span>
</div>

`index.js`가 실행된 후 `CSSOM Tree`의 상태는 그림 4와 같다.  

<div class="img-container">
    <img class="img" src="https://i.imgur.com/TSOgXPx.png" alt=""/>
    <span class="caption">그림 4. index.js 실행 후 CSSOM Tree의 상태.</span>
</div>

### image 리소스

> **Line 17**
 > ```html 
 > <img src="./image.jpg" alt="">
 > ```

- **Line 17**: `img` 태그를 만난다. `image.jpg`를 서버에 요청한다. 이 때 파싱은 stylesheet 리소스를 요청할 때와 같이 멈추지 않고 계속 진행한다.  

**Line 24**까지 모든 파싱을 마치고 최종 생성된 `DOM Tree`는 그림 5와 같다.  


<div class="img-container">
    <img class="img" src="https://i.imgur.com/buvoo82.png" alt=""/>
    <span class="caption">그림 5. 최종 DOM Tree 모습.</span>
</div>

앞서 설명한 HTML 파싱 과정을 waterfall 방식으로 나타내면 그림 6와 같다. `css`와 `image`는 병렬적으로 요청되지만 `script` 태그를 만났을때는 파싱이 잠시 중단된다. 이는 위에서 언급했듯이 `css`와 `image`는 `DOM Tree`를 생성하는데 아무런 영향을 미치지 않지만 `script`는 파싱 도중에 `DOM Tree`를 직접적으로 수정할 수 있기 때문이다. 또한 `script`는 파싱 중에 `CSSOM Tree`도 수정할 수 있으므로 `script`를 실행하기 전에 `CSSOM Tree`이 아직 생성되지 않았다면 완료될때까지 기다린다. 


<div class="img-container">
    <img class="img" src="https://i.imgur.com/WnfFySr.png" alt=""/>
    <span class="caption">그림 6. HTML 파싱 Waterfall.</span>
</div>


HTML 파싱이 완료되고 `Dom Tree`가 생성된 시점에서 document 객체에서 `DomContentLoaded` 이벤트가 발생한다. 이 이벤트는 그림 7에서 파란색 실선이다. `index.js` 스크립트가 모두 실행되고 호출되는 시점으로, 이는 **Line 14**의 스크립트를 동기적으로 실행하고 **Line 24**(끝)까지 파싱을 완료하고 `Dom Tree`를 생성한 시점이다. 빨간색 실선은 나머지 과정(`HTML 파싱` ~ `paint`)이 모두 완료된 후 호출되는 `Load` 이벤트를 나타낸다.

<div class="img-container">
    <img class="img" src="https://i.imgur.com/tlqPuvn.png" alt=""/>
    <span class="caption">그림 7. Chrome Network 탭에서의 HTML 파싱 과정.</span>
</div>


## Render Tree
---
`Render Tree`는 앞서 생성된 `DOM Tree`에 `CSSOM Tree`를 적용시켜 생성된 새로운 `Tree`이다. `Render Tree`는  실제 화면에 표시될 노드들로만 구성되어있다. 그래서 `head` 태그는 포함되지 않으며 `body` 태그 내의 내용들로 구성되어있다. 실제 그림 4와 그림 5를 합쳐서 구성된 `Render Tree`는 그림 8과 같다.

<div class="img-container">
    <img class="img" src="https://i.imgur.com/pCMknS9.png" alt=""/>
    <span class="caption">그림 8.  DOM Tree와 CSSOM이 합쳐진 Render Tree.</span>
</div>

`Render Tree`는 화면에 표시되는 모든 요소를 포함하며, 요소에 적용되는 스타일 정보를 나타낸다. 이 `Render Tree`를 구성하는 과정에서 각 요소의 스타일은 하향식으로 종속적으로 적용된다. 이 때, 자식 스타일과 부모 스타일이 겹치게되면 더 깊게 적용된 스타일에 우선순위를 두며 상속받은 스타일은 무시해버린다 (그림 8 참조). `Render Tree`는 DOM 요소의 스타일 정보와 레이아웃 정보를 담고 있지만, 이 정보는 요소가 화면에 어떤 위치에 렌더링되어야 하는지 직접적으로 나타내지는 않는다. 요소의 실제 위치는 레이아웃(Layout) 단계에서 결정된다.

## Layout(Reflow)
---

`Layout` 단계에서는 **viewport를 기준으로 요소의 크기, 위치, 여백, 패딩 등이 계산**되어 요소가 실제 화면에 배치될 위치(좌표)가 정해진다. 그래서 `Layout`을 일으키는 기준인 브라우저의 창 크기나 CSS Box Model을 수정하면 화면에 그려지는 좌표값이 달라지기 때문에 Layout 과정을 다시 계산하게 된다. 그림 9는 실제 Box Model과 이미지의 width를 각각 20%, 30%로 설정한 다음 브라우저 창 크기를 반복적으로 리사이즈하여 실제로 `Layout` 과정이 이루어지는지 측정해보았다.

<div class="img-container">
    <img class="img" src="https://i.imgur.com/qMYsQLS.gif" alt=""/>
    <span class="caption">그림 9.  Layout 성능 측정.</span>
</div>


그림 10의 이벤트 로그를 확인해보면 Viewport 크기가 변경될 때마다 Layout 과정이 다시 이루어지면서 요소의 좌표값을 계산해주는 모습을 확인할 수 있다. 

<div class="img-container">
    <img class="img" src="https://i.imgur.com/RVXqTRb.gif" alt=""/>
    <span class="caption">그림 10.  Layout 성능 측정 결과.</span>
</div>

이렇게 화면에 보이는 요소 각각이 어디에 어떻게 위치할 지를 정해주는 과정을 Webkit에서는 layout으로, Gecko에서는 reflow로 부르고 있다.

## Paint
---

`Paint` 단계는 `Layout` 단계에서 변환된 실제 좌표 값을 화면에 그려주는 역할을 한다. `Layout` 단계에서는 요소의 크기와 위치에 기준을 두었다면 `Paint`를 일으키는 기준은 요소의 색상이나 배경이 된다.  그림 11은 Box의 테두리 색상을 100ms 간격으로 변경하게 두고 실제로 `Paint` 과정이 이루어지는지 측정해보았다.

<div class="img-container">
    <img class="img" src="https://i.imgur.com/DxJ2IYT.gif" alt=""/>
    <span class="caption">그림 11. Gecko Reflow Visualization - mozilla.org.</span>
</div>

그림 12의이벤트 로그를 확인해보면 테두리 색상만 변경되었으므로 Layout 과정없이 Paint 단계부터 다시 실행되는걸 확인할 수 있다.

<div class="img-container">
    <img class="img" src="https://i.imgur.com/v9AdUtf.gif" alt=""/>
    <span class="caption">그림 11. Gecko Reflow Visualization - mozilla.org.</span>
</div>



## Reference

---

- [Ryan Seddon: So how does the browser actually render a website | JSConf EU 2015](https://www.youtube.com/watch?v=SmE4OwHztCc)  
- [Kruno: How browsers work | JSUnconf 2017](https://www.youtube.com/watch?v=0IsQqJ7pwhw)  
- [How to render in WebKit](https://www.youtube.com/watch?v=RVnARGhhs9w)  
- [브라우저는 웹페이지를 어떻게 그리나요? - Critical Rendering Path](https://m.post.naver.com/viewer/postView.nhn?volumeNo=8431285&memberNo=34176766)  
- [Critical Rendering Path](https://web.dev/articles/critical-rendering-path?hl=ko)

#frontend

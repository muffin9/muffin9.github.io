---
permalink: /Javascript/infinite-scroll
title: "Js로 Infinite Scroll 구현"
date: 2021-11-13T14:12-14:12
categories:
- javascript
tags:
- javascript
---

### JavaScript로 Infinite Scroll 구현하기

오늘은 여러 웹 사이트에서 널리 사용되어지는 무한스크롤링을 한 번 구현해보기 위해 어떤식으로 구현을 해볼까?  
나름 고민을 하다가 오픈소스 API인 the cat api를 사용하여 고양이 이미지를 받아오고 스크롤을 계속 내리면서 스크롤이 화면에서 맨 아래 닿았을때 추가적으로 api를 요청하여 고양이 데이터를 받아와 화면에 이미지를 동적으로 추가되도록 구현하였다.


![infiniteScroll](/assets/image/infinite/infiniteScroll.gif)

```javascript
const API_URL = 'https://api.thecatapi.com/v1/images/search?size=small&limit=3';
const imageWrapper = document.getElementById('images');
// cnt변수는 화면에 보여지는 고양이 이미지 수를 카운팅
let cnt = 0;

const catApiGet = (url, callback) => {
    const xmlHttp = new XMLHttpRequest();
    xmlHttp.onreadystatechange = () => {
        if(xmlHttp.readyState === 4 && xmlHttp.status === 200) {
            let data;
            try {
                data = JSON.parse(xmlHttp.responseText);
            } catch(e) {
                console.log(e.message + " in " + xmlHttp.responseText);
                return;
            }
            callback(data);
        }
    };

    xmlHttp.open("GET", url, true);
    xmlHttp.send();
    cnt += 3;
    console.log(cnt);
}

const fetchData = () => {
    catApiGet(API_URL, (data) => {
        for(let i = 0; i < 3; i++) {
            const li = document.createElement('li');
            const img = document.createElement('img');
            img.src = data[i].url;
            li.append(img);
            imageWrapper.append(li);
        }
    });
}

window.addEventListener('scroll', () => {
    // 디바이스 윈도우 높이 + 현재 스크롤 위치 >= 문서의 높이
    // innerHeight : 사용자가 그대로 바라고 있는 화면상 실제로 표시되고 있는 영역 높이
    // scrollY : 현재 스크롤 Y축의 위치값 ( 0 ~ 증가 )
    // offsetHeight: 사용자가 보여지고 있는 영역 외에 가려진 영역까지 모두 포함된 영역(스크롤을 내린만큼 위의 포함된 영역)
    // api가 2~3번 호출되는 경우가 있음
    if ((window.innerHeight + window.scrollY) >= document.body.offsetHeight) fetchData();
})
```

---

위 이미지대로 구현하는건 생각보다 어렵지 않았다. 하지만 스크롤이 맨 아래 닿을때 나는 이미지를 3개씩 동적으로 추가하고 싶었는데 여러번 api를 호출하여 100% 내가 원하는대로 기능이 구현되지 않았다.  

그럼 어떻게하면 api를 한 번만 호출되도록 할 수 있을까? 개인적으로 스크롤이 맨 아래 닿았을때 api를 호출하는 함수를 1초마다 실행되도록 setTimeout을 걸면 되겠구나 생각했다.  

이렇게 이벤트 중간에 딜레이를 포함시켜 딜레이 이내로 연속적으로 발생된 이벤트를 무시하는 개념을 프론트엔드 영역에서는 쓰로틀링이라고 불리우고있다. 아래는 쓰로틀링이 적용된 코드와 작동 화면이다.

![infiniteScroll](/assets/image/infinite/throttling.gif)

```javascript
let throttler;

window.addEventListener('scroll', () => {
    if(!throttler) {
        // 스크롤을 아무리 내가 계속 내려도 1초마다 호출되도록 막아놔서 api는 무조건 한 번만 호출
        throttler = setTimeout(() => {
            throttler = null;
            if ((window.innerHeight + window.scrollY) >= document.body.offsetHeight) fetchData();
        },1000);
    }
})
```

scroll 이벤트를 발생시키는 부분내에서 setTimeout을 사용하여 코드를 약간 수정하였다.  

throttler변수를 선언하고 이 변수에는 스크롤이 맨 아래 닿았을때 동작하는 함수를 저장시켜 한 번에 여러번 호출되는 일이 없도록 설정하였다.

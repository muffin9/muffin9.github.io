---
permalink: /Javascript/debounce
title: "Carousel Slide"
date: 2021-11-21T20:12-14:22
categories:
- javascript
tags:
- javascript
---

### Carousel Slide

정말 오랜만에 순수 자바스크립트를 사용하여 만들어보는 사이드 프로젝트로 예전부터 한 번 만들어보고 싶었던 Carousel Slide를 만들어보았다.  
매번 전 회사에서 이와 비슷한 기능이 필요하다면 라이브러리를 가져다 쓰기 바빴는데 이번에 혼자 만들어보았다.

처음에는 유튜버 노마드코더 강의를 바탕으로 아래 화면처럼 3초마다 자동으로 중앙 5개의 이미지들이 전환되도록 구현했었다.  
https://www.youtube.com/watch?v=l18HCZqBs6I

![carousel](/assets/image/carousel/carousel.gif)

`index.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>javascript Carousel</title>
    <link rel="stylesheet" href="index.css" />
</head>
<style>
body {
    margin: 0;
    padding: 0;
}

.slider__block {
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
}

#slider {
    width: 400px;
    height: 500px;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    justify-content: center;
    position: relative;
}

.slider__item {
    float: left;
    width: 400px;
    height: 500px;
    position:absolute;
    transition: all 1s ease-in-out; /* 전환(transition) 효과가 천천히 시작되어, 천천히 끝납니다. */
    transform: translate3d(-400px, 0px, 0px);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 0;
    opacity: 0;
    top: 0;
}

.slider__item:nth-child(odd) {
    background-color: violet;
}
.slider__item:nth-child(even) {
    background-color: yellowgreen;
}

.slider__item img {
    width: 400px;
    height: 500px;
}

.arrow {
    width: 400px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: absolute;
    z-index: 99;
}

.arrow__item {
    margin: 0 10px;
    font-size: 25px;
    cursor: pointer;
}

#dots {
    text-align: center;
    padding: 0;
    z-index: 99;
    margin-top: auto;
}

#dots li {
    margin: 0 15px;
    padding: 5px;
    font-size: 20px;
    list-style: none;
    display: inline-block;
    color: white;
    cursor: pointer;
}

.showing {
    z-index: 1;
    opacity: 1;
    transform: scale(1);
}
</style>
<body>
    <div class="slider__block">
        <div id="slider">
            <div id="1" class="slider__item showing">
                <img src="images/img1.jpeg" />
            </div>
            <div id="2" class="slider__item">
                <img src="images/img2.jpeg" />
            </div>
            <div id="3" class="slider__item">
                <img src="images/img3.jpeg" />
            </div>
            <div id="4" class="slider__item">
                <img src="images/img4.jpeg" />
            </div>
            <div id="5" class="slider__item">
                <img src="images/img5.jpeg" />
            </div>
        </div>
    </div>
<script type="text/javascript" src="index.js"></script>
</body>
</html>
```

html은 간단히 5개의 이미지가 순서대로 넘겨지도록 div태그 5개를 만들고 각각 안에 img태그를 넣어주었다.

css는 #slider 자식태그들이 똑같은 자리에서 이동되도록 보여지기 위해서는 약간의 trick을 써야 하는데 먼저 자식 요소가 float로 설정을 하면 (말 그대로 자식 요소가 떠 있는 상태), 부모 요소(상위 요소)의 height가 0이 되기 때문에 부모 속성에 overflow: hidden 속성으로 자식요소의 레이어를 부모 요소 내부로 제안되도록 한다.


그래서 이번에는 흔히 사용되어지는 carousel 라이브러리와 비슷하게 오른쪽, 왼쪽 버튼과 아래 dots들을 추가시켜 기능을 구현하였다.  

왼쪽, 오른쪽 화살표 선택시에는 전, 후 이미지들이 자연스럽게 보여지도록 하였으며 아래 점들을 클릭했을때는 그에맞는 이미지와 점을 색칠하도록 하였다.

---

```javascript
const sliderItem = document.querySelectorAll('.slider__item');
const firstElement = sliderItem[0];
const lastElement  = sliderItem[sliderItem.length - 1];
const dotsElement = document.getElementById('dots');
const sliderItemCount = document.querySelectorAll('.slider__item').length;
let currentIndex = 1;

const init = () => {
    createDot();
}

const createDot = () => {
    for(let i = 0; i < sliderItemCount; i++) {
        const li = document.createElement('li');
        li.innerHTML = "*";
        li.classList.add('dot');
        li.addEventListener('click', () => onClickDot(i+1));
        dotsElement.append(li);
    }
}

const previousBtn = () => {
    const currentSlider = document.querySelector('.showing');

    if(currentIndex == 1) {
        currentIndex = sliderItem.length;
        currentSlider.classList.remove('showing');
        lastElement.classList.add('showing');
        onDotColorChange(currentIndex);
        return;
    }

    if(currentSlider) {
        currentSlider.classList.remove('showing');
        const previousElement = currentSlider.previousElementSibling;
        if(previousElement) {
            previousElement.classList.add('showing');
        }
        currentIndex -= 1;
        onDotColorChange(currentIndex);
    }
}

const nextBtn = () => {
    const currentSlider = document.querySelector('.showing');

    if(currentIndex == sliderItem.length) {
        currentIndex = 1;
        currentSlider.classList.remove('showing');
        firstElement.classList.add('showing');
        onDotColorChange(currentIndex);
        return;
    }

    if(currentSlider) {
        currentSlider.classList.remove('showing');
        const nextElement = currentSlider.nextElementSibling;
        if(nextElement) {
            nextElement.classList.add('showing');
        }
        currentIndex += 1;
        onDotColorChange(currentIndex);
    }
}

const onClickDot = (id) => {
    const selectedSlider = document.getElementById(id);
    const currentSlider = document.querySelector('.showing');

    onDotColorChange(id);
    currentSlider.classList.remove('showing');
    selectedSlider.classList.add('showing');
    currentIndex = id;
};

const onDotColorChange = (id) => {
    const liElements = document.querySelectorAll('.dot');

    liElements.forEach((element, index) => {
        if(index === id-1) {
            element.style.color = "black";
        } else {
            element.style.color = "white";
        }
    });
}

init();
```


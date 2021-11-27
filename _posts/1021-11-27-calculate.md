---
permalink: /Javascript/calculate
title: "Calculate"
date: 2021-11-27T21:20-21:12
categories:
- javascript
tags:
- javascript
---

### `자바스크립트로 계산기를`

오늘은 바닐라 자바스크립트를 사용하여 뭘 만들어볼까 고민하다가 간단하게 사칙연산만 할 수 있는 반쪽짜리...(한계) 계산기를 만들어보려고 한다. 삼각함수나 다른 기능들을 넣기에는 너무 복잡하기도 하고 자바스크립트가 아닌 수학공부가 더 치중되고 시간도 많이 걸릴거라 생각되어 과감하게 빼고 작업했다.

![calculate](/assets/image/calculate/calculate.gif)

대략적인 UI와 기능은 위와 같이 작동되도록 만들어보았다. UI는 핀터레스트에서 가져와 90% 유사하게 퍼블리싱하였고 기능은 구글계산기를 모티브로 작업을 진행했다.

html단에서는 0~9숫자,  연산자(+, -, *, /, %), 소수점에 해당되는 dom을 클릭했을때 클릭한 값들을 저장하고 연산하기 위하여 모든 태그에 onclick이벤트 속성으로 handleFormula 함수를 연결했고 연산자(=)는 input태그에 입력되어진 값을 계산을 하기위하여 calculate함수를 연결하였다.

뭐.. css쪽은 따로 인터렉션한 디자인은 아니여서 간단하게 구현했다.

핵심부분은 javascript이므로 자바스크립트 코드만 정리해보았다. (전체코드는 개인 github에 있습니다.) [https://github.com/Jinlog9/javascript-calculator](https://github.com/Jinlog9/javascript-calculator)

```javascript
// JS code Start
// JS code Start
let caclulateValue = 0;
let operFlag = false; // 연속해서 연산자를 입력받았을때 제어하기 위한 변수
let resultFlag = false;
let decimalFlag = false;
let zeroFlag = false;
const formulaElement = document.getElementById('formula');

const boxElements = document.getElementsByClassName('box');
const resultElement = document.getElementById('result');
const clearElement = document.getElementById('clear');


const handleMouse = (e) => {
  const element = e.target;
  const boxClass = element.classList.value;
  const boxClassArr = boxClass.split(" ");

  if(boxClassArr.includes('bg-yellow')) {
    element.classList.remove('bg-yellow');
  }
}

const handleFormulaClear = () => {
  formulaElement.value = '';
  resultElement.innerHTML = '0';
  operFlag = false;
  decimalFlag = false;
  zeroFlag = false;
}

const zeroCheck = (value) => {
  if(value == 0) {
    if(formulaElement.value == 0) zeroFlag = true;
    else zeroFlag = false;
  }

  if(value == 0 && zeroFlag) {
    return false;
  }

  return true;
}

const decimalCheck = (value) => {
  // 소수점 한 번만 입력되도록 처리
  if(value == ".") {
    if(!decimalFlag) decimalFlag = true;
    else return false;
  }

  return true;
}

const minusCheck = (value) => {
  if(!isNaN(formulaElement.value) && value == '-' && !operFlag) {
    formulaElement.value += value;
    return false;
  }
  return true;
}

const operCheck = (value) => {
  console.log(value);
  if(!operFlag && isNaN(value)) {
    return false;
  }
  return true;
}

const reCacluateCheck = (value) => {
  if(resultFlag && !isNaN(value)) {
    caclulateValue = 0;
    resultFlag = false;
    decimalFlag = false;
    if(operFlag) {
      formulaElement.value = '';
    }
  }
  return true;
}

const handleFormula = (value) => {
  if(value == 0 && isNaN(formulaElement.value.toString().substr(-1))) return;
  // 소수점 한 번만 입력되도록 처리
  if(!decimalCheck(value)) return;

  // 맨 처음 '-' 연산 처리
  if(!minusCheck(value)) return;

  // 연속해서 연산자가 올 경우 그냥 리턴
  if(!operCheck(value)) return;

  // 한 번 연산이 완료되고 연산자가 아닌 숫자가 올 경우
  if(!reCacluateCheck(value)) return;

  // 처음에 계속 0처리하는 부분
  if(!zeroCheck(value)) return;

  if(isNaN(value)) {
    // 입력한 값이 연산자라면 다음 입력을 숫자로 받기 위해 operFlag => false
    operFlag = false;

    if(value != ".") {
      decimalFlag = false;
    }
  } else {
    operFlag = true;
  }
  formulaElement.value += value;
}

const calculate = () => {
  const result = eval(formulaElement.value);
  if(isNaN(result)) return;

  caclulateValue += result;
  formulaElement.value = caclulateValue;
  resultElement.innerHTML = `Ans = ${caclulateValue}`
  resultFlag = true;
  zeroFlag = false;
}

const init = () => {
  Array.from(boxElements).forEach((element) => {
    element.addEventListener('mouseover', handleMouse);
  });

  clearElement.addEventListener('click', handleFormulaClear);
}

init();
```

내가 만들 계산기에 어떤 조건이 있고 이 조건들만 잘 처리하기위한 변수 선언이 가장 핵심이라 생각한다.   

1. calcuateValue : 계산되어지는 값을 축적하는 변수
2. operflag : 연속해서 연산자를 입력받았을때 제어하기 위한 변수
3. resultFlag : 계산이 완료되었는지, 다시 계산을 하는지 체크 하는 변수
4. decimalFlag : 소수점을 처리하기 위한 변수
5. zeroFlag : 0값을 처리하기 위한 변수

`함수 정리`

1. init() : 모든 숫자, 연산자에 해당되는 dom에 마우스오버 속성을 추가시키기 위해 forEach구문으로 핸들러등록 clear Element는 말그대로 모든 값을 초기화시켜야하므로 handleFormulaClear함수 등록

2. calculate() : input태그에 입력된 value값들의 계산된 값을 result변수에 저장시키며 caclulateValue변수에는 이전까지 계산된 값이 축적되며 계속해서 값을 저장시키기 위해 input태그의 value값에는 위에서 축적한 caclulateValue 변수값을, 동시에 결과Element에도 caclulateValue값을 보여주도록 한다.  
   계산이 1번 완료되면 계산이 완료되었다는 flag값인 resultFlag를 true로 zeroFlag는 false로 셋팅
3. handleFormula() : 계산기 어플에서 가장 핵심적인 함수로 주석이 어느정도 달려있지만 중요한 함수인 만큼 부가적으로 설명을 더하려한다. 무엇보다 if절로 체크하는 함수들의 순서가 중요
  3-1. 맨 처음에 0이 계속해서 올때는 input태그에 0000처럼 입력되면 안되므로 조건처리  
  3-2. zeroCheck함수는 클릭한 value값이 0으로 시작되는 값이면 안되므로(015, 043와 같이) 조건처리  
  3-3. decimalCheck함수는 연산자가 나오기 이전의 숫자에 소수점은 한 번만 나와야 하는 조건처리 (3.4.2 와 같은 숫자 X)  
  3-4. minusCheck함수는 다른 연산자와 달리 맨 처음에 올 수 있으므로 조건처리  
  3-5. reCacluateCheck함수는 1번 연산이 완료가되고 바로 연산자가 아니라 숫자가 온 경우 다시 처음 부터 연산을 진행해야하므로 초기화 셋팅 조건처리  
  3-6. 마지막 if조건은 바로 리턴되지 않으며 입력한 값이 연산자인지 아닌지에 따라 처리되는데 만약 연산자라면 다음 입력은 숫자로 받기 위해 operFlag를 false로 셋팅, 또한 소수점이 아닐경우에도 연산자 다음에 소수점을 입력되도록 처리해야하므로 decimalFlag 값 false로 셋팅 조건처리  
  3-7. 위 조건 처리 후 input value값에는 입력한 value가 계속 더해주기  

** 기능은 돌아가지만 뭔가 나도 수정하고 간단하게 만들고 함수단위로 정리해보자 해봤는데도 조건절이랑 flag값들이 많이 남아있다. 이걸 다른사람들이 처음에 위 코드를 봤을때 어떻게하면 조금이나마 더 이해하기 수월할까? 숙제로 남겨야겠다.
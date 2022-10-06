---
permalink: /react/useEffect
title: "useEffect"
categories: React

tags: React Hook useEffect

toc: true
toc_sticky: true


date: 2022-10-06
last_modified_at: 2022-10-06
---

### `useEffect 등장배경`

Effect Hook을 사용하면 함수 컴포넌트에서 side effect를 수행할 수 있는데, 이전 클래스형 컴포넌트 기반과는 달리 관심사에 따라 관리할 수 있게 하는 것이 핵심이다.

**Side Effect는 함수가 실행되면서 함수 외부의 값이나 상태를 변경시키는 것을 의미하는 것으로 대표적으론 데이터를 가져오기 위해서 외부 API를 호출할 때다.**


### `useEffect란 ?`  

잠깐 useEffect에 대해 살펴보기전에 생명주기에 대해 잠깐 짚고 넘어가자.  

리액트에서의 컴포넌트는 기본적으로 마운트 -> 업데이트(반복) -> 언마운트의 생명주기를 가지고 있다.  

마운트 상태 ? 컴포넌트 구조가 HTML DOM에 존재하는 상태로 클래스형 컴포넌트에서는 componentDidMount와 메서드와 같다.
업데이트 상태 ? 컴포넌트 DOM 구조나 내용이 변경될 때 상태로 클래스형 컴포넌트에서는 componentDidUpdate 메서드와 같다.
언마운트 상태 ? 컴포넌트 구조가 HTML DOM에서 제거된 상태로 클래스형 컴포넌트에서는 componentWillUnmount 메서드와 같다.

---

앞서 useEffect가 컴포넌트가 렌더링 될 때마다 Side Effect 로직을 다루는 hook이라고 배웠다.  

근데 왜 굳이 렌더링 이후에 useEffect가 실행되어야 하는걸까? 렌더링 이전에 useEffect를 실행해주면 리액트 성능에 좋지 않기 때문이다. 그 이유는 단순한데, 가상 돔 렌더 과정에 Side Effect가 포함되면 Side Effect가 발생할 때마다 가상 돔을 다시 렌더 해야 하기 때문이다.  

리액트의 핵심인 재조정 알고리즘을 위해서 가상 돔은 `Side Effect를 제외한 순수한 컴포넌트만`을 사용해 렌더링을 한다고 한다.  

그리고 사용자 입장에서 바라봤을때, 비동기적으로 데이터를 가져올때 컴포넌트가 렌더링 되기 전에 useEffect를 수행해야 한다면 하염없이 기다려야 하는데, 지금처럼 렌더링 이후에 처리한다면 데이터가 도착할때까지 해당 영역을 로딩이나 스켈레톤 UI로 가리는 것이 더 UI 적으로 좋다고 생각이 든다.


---

### `useEffect 사용사례`  

```javascript
import { useState, useEffect } from 'react'

function App() {
  const [catData, setCatData] = useState();

  useEffect(() => {
    async function fetchCat() {
      const response = await fetch(`https://aws.random.cat/meow`); // get random cat image
      const data = await response.json();
      console.log(data); // 2번 찍히는 이유?
      if (data) {
        setCatData(data.file);
      }
    }

    fetchCat();
  }, []);

  return (
    <div className="App">
      <img width={500} height={500} src={catData} alt="Cat Image" />
    </div>
  );
}
```

useEffect에 2번째 인자로 빈 배열을 넣어줌으로써 컴포넌트 최초 렌더링 이후에 useEffect 함수가 한 번 실행이 되는데, 이 때 고양이 랜덤 이미지를 가져와서 catData 상태를 바꿔 화면에 고양이 이미지를 뿌려주는 코드를 작성해보았다.  

원래 개발을 할때 간단한 작업에만 콘솔로그를 종종 사용하는 편이다. 근데 개발환경에 따라 useEffect 내에서의 콘솔로그가 2번찍히는데 이 2번찍히는 이유는 다음과 같다.  

Code SandBox 환경에서 로직을 테스트 하고 있는데 App 컴포넌트를 그려줄때 `<React.StrictMode>`로 감싸져 있어서 그렇다. 이 StrictMode는 리액트에서 제공하는 검사 도구인데, 개발 모드일때만 내부적으로 디버그를 하며 애플리케이션 내의 잠재적인 문제를 알아내기 위해 해당 태그로 감싸져 있는 부분은 자손까지 검사한다.  

**단단한 코드를 작성하기 위해서는 StrictMode를 켜놓고 개발하도록 하는게 좋다고 한다.**

useEffect는 2개의 매개변수를 받게되며 항상 DOM 상태 변경, 레이아웃 배치, 화면 그리기가 모두 완료된 이후 첫 번째 인자로 넣어준 effect 함수가 실행된다는 특징을 갖고 있다. 그럼 2개의 인자가 어떤게 있는지 살펴보자.

1. 첫 번째 인자 : 컴포넌트 레이아웃 배치와 화면 그리기가 끝난 후 실행될 함수로 여기 첫번째 인자가 반환하는 함수를 clean-up 함수
2. 두 번째 인자 : 의존성 배열  

이렇게 두 개의 인자로 동작하는 useEffect는 첫번째 인자로 주어진 함수를 실행하기 전에 의존성 배열의 원소가 변경됐는지 확인한다. 이전에 살펴본 useState와 유사하게 `Object.is` 메서드를 통하여 값이 변경되지 않았다면 useEffect는 실행되지 않는다. But, 의존서 배열에 넣어준 값 중에서 하나라도 변경이 된다면, useEffect가 첫 번째 인자로 넣어준 함수가 실행이된다.  

**2번째 인자인 의존 배열에 특정 state를 입력하면 Effect 함수는 컴포넌트 최초 렌더링 및 해당 값이 변경될 때마다 실행되지만 빈 배열을 넣어주면 컴포넌트 최초 렌더링 이후 한 번만 실행된다.**  

`2번쨰 인자로 의존성 배열에 빈 배열을 넣는 경우?`  

1. Kakao Map API 와 같은 DOM을 사용해야 하는 외부 라이브러리를 연동
2. 바로 렌더링 이후에 화면에 뿌려줄 데이터를 비동기 통신으로 서버에 데이터를 요청할 때

`정리(Clean-up)를 이용하지 않는 Effects`  

정리(Clean-up)를 이용하지 않는 Effects라고도 할 수 있다. => 네트워크 리퀘스트, DOM 수동 조작, 로깅 등 정리(clean-up)가 필요 없는 경우들

```javascript
import React, { useState, useEffect } from 'react';

export default function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

`정리(clean-up)를 이용하는 Effects`  

메모리 누수가 발생하지 않도록 정리(clean-up)하는 것은 매우 중요하다.  

```javascript
import { useEffect, useState } from "react";

const Timer = () => {
  const [time, setTime] = useState(0);

  useEffect(() => {
    let currentTime = 0;
    const timer = setInterval(() => {
      currentTime += 1;
      setTime(currentTime);
      console.log(currentTime);
    }, 1000);

    // clean up function
    return () => {
      clearInterval(timer);
    };
  }, []);

  return (
    <div>
      <p>currentTime: {time}</p>
    </div>
  );
};

function App() {
  const [isTimer, setIsTimer] = useState(false);
  const handleToggleTimer = () => {
    setIsTimer(!isTimer);
  };

  return (
    <div className="App">
      <header className="App-header">
        {isTimer && <Timer />}
        <button onClick={handleToggleTimer}>
          {isTimer ? "Stop Timer" : "Start Timer"}
        </button>
      </header>
    </div>
  );
}

export default App;

```

Start Timer 버튼을 클릭하게 되면 타이머가 동작됨과 동시에 카운팅되는 초를 세어줌과 동시에 화면에 노출되는 로직이다.  
만약 Timer 컴포넌트 내에서의 useEffect에서 clean up function으로 clearInterval 함수를 리턴해주지 않았다면... Stop Timer를 눌러도 타이머가 멈추지않고 계속 흘러간다..

![claenup 적용 X](https://user-images.githubusercontent.com/45479309/194263397-67c9f598-8e8d-4589-a839-27c9f535f6f8.gif)  

하지만, clean up function을 리턴함으로써 우리가 원하느느 정상적인 결과를 볼 수가 있다!  

![cleanup 적용 O](https://user-images.githubusercontent.com/45479309/194263498-9bb15677-b262-4f09-8ec0-8b502ec6a39c.gif)

clean-up함수를 리턴해야 하는 주 사례는 아래와 같다.  

1. DOM 이벤트를 제거할 때
2. 위의 예제와 같이 clearTimeout을 통한 타이머 함수를 종료할 때
3. 소켓을 닫아야할때

---

`useLayoutEffect ?`

대부분의 Side Effect는 useEffect를 통한 비동기 처리가 권장 되어진다. 하지만, 사용자 입장에서 DOM의 업데이트로 인하여 화면 깜빡임으로 인해 useLayoutEffect 함수를 사용할 수도 있다.  

useEffect 와는 달리 useLayoutEffect는 컴포넌트가 render 된 후에 동기적으로 실행된다. 그 이후에 마지막으로 paint작업이 실행이 되기 때문에 useLayoutEffect 내부에서 DOM에 영향을 주는 코드가 있어도 화면 깜빡임이 생기지 않는다. 이 useLayoutEffect에 대해서는 내가 팀 프로젝트를 진행하면서 비교한 글이 있는데 해당글을 참고하면 좋을 것 같다.

참고 : https://seokzin.tistory.com/entry/React-useEffect%EC%9D%98-%EC%B2%A0%ED%95%99%EA%B3%BC-Lifecycle#%EB%--%B-%EC%-E%A-%--%EB%B-%B-%EA%B-%BD
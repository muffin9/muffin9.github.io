---
permalink: /react/useLayoutEffect
title: "useLayoutEffect"
categories: React

tags: React Hook useLayoutEffect

toc: true
toc_sticky: true


date: 2022-10-07
last_modified_at: 2022-10-07
---

### `useLayoutEffect`

useEffect를 학습하면서 알게된 개념인 useLayoutEffect. useEffect 와는 달리 useLayoutEffect는 컴포넌트가 render 된 후에 동기적으로 실행되는 개념을 갖고 있다. useLayoutEffect는 내부에서 DOM에 영향을 주는 코드가 있어도 화면 깜빡임이 생기지 않는다고 하는데, 어떤 상황일때 사용할 수 있을지 궁금해서 구글링해보고  적용해보는 시간을 가져봤다.  

### `함수 호출 시점`

![react hook flow](https://raw.githubusercontent.com/donavon/hook-flow/master/hook-flow.png)

1. 리액트 상태 업데이트.
2. React Render Dom
3. useLayoutEffect 발생.
4. Screen Paint 작업
5. useEffect 발생.

>> useLayoutEffect 는 동기적으로 실행하기 때문에 퍼포먼스적으로 useEffect 주로 사용하나, 기능적으로 꼭 필요 할때만 useLayoutEffect를 사용한다. (두 함수의 차이점으로는 브라우저 페인팅 전후에 따른 실행 순서)  

### `useLayoutEffect로 개선`

useLayoutEffect를 사용하여 특정 컴포넌트의 최초 렌더링을 개선해보았다.  

```javascript
import { useLayoutEffect, useEffect, useState } from "react";

const MyPage = () => {
  const [title, setTitle] = useState('빈 값');
  
    useEffect(() => {
        setTitle('지도관리');
    }, []);

  return(
    <>
        <p>{title}</p>
    </>
  )
}
```

![useEffect 사용](https://user-images.githubusercontent.com/45479309/194485737-844c51e6-ea68-435d-b6a9-60b6e47fc763.gif)

새로고침시 useState로 초기에 설정한 값인 빈 값이 텍스트가 보이게 된다.

```javascript
import { useLayoutEffect, useState } from "react";

const MyPage = () => {
  const [title, setTitle] = useState('빈 값');
  
    useLayoutEffect(() => {
        setTitle('지도관리');
    }, []);

  return(
    <>
        <p>{title}</p>
    </>
  )
}
```

![useLayoutEffect 사용](https://user-images.githubusercontent.com/45479309/194485571-1c5cb715-e622-40c2-9f62-f0f756992a74.gif)

지금 당장은 간단한 텍스트 변화만 일어나는 간단한 DOM의 변화였지만 만약 화면이 좀 더 복잡해지고 사용자 입장에서 체감할 수 있는 화면 깜빡임이라면, 그 때는 이 useLayoutEffect 훅을 적절하게 사용하면 되겠다.


### `마무리`

useLayoutEffect 언제 사용하면 좋을까?  

1. DOM을 변경해야 하는 경우
2. 해당 컴포넌트 내에서 관리하고 있는 state 값을 업데이트하여 화면에 표출해야 하는 경우
---
permalink: /react/customHook
title: "customHook"
categories: React

tags: React Hook customHook

toc: true
toc_sticky: true


date: 2022-10-07
last_modified_at: 2022-10-07
---

### `custom Hook?`  


`React Document 에서의 custom hook 은 다음과 같이 설명한다.`  

1. 자신만의 Hook을 만들면 컴포넌트 로직을 함수로 뽑아내어 재사용할 수 있다. 두 개의 자바스크립트 함수에서 같은 로직을 공유하고자 할 때는 또 다른 함수로 분리하는데, 같은 원리로 컴포넌트와 Hook 또한 함수이기 때문에 분리할 수 있다.  

2. 사용자 정의 Hook은 관례상 이름이 use로 시작하는 자바스크립트 함수이다.

3. 사용자 Hook을 사용할 때마다 그 안의 state와 effect는 완전히 독립적이다 => 사용자 정의 Hook을 다른 여러 컴포넌트에서 가져와 사용할때, 다른 여러 컴포넌트 입장에서 해당 Hook의 state, effect는 공유되지 않는다.

**사용자 정의 Hook은 React의 특별한 기능기능이 아닌 Hook의 디자인을 따르는 관습**

### `custom Hook 준비`  

인풋 검색창에 키워드 검색어 입력시, 해당 키워드가 바로 인식되는게 아닌 debounce기법을 적용하여 0.5초 마다 키워드 검색이 되도록 하는 useDebounce Hook을 만들어 보고자한다.  

```javascript
import { useState } from "react";
import "./styles.css";

export default function App() {
  const [searchValue, setSearchValue] = useState("");
  const [movieData, setMovieDatas] = useState([]);

  const handleSearchValueChange = async ({ target }) => {
    const value = target.value;
    setSearchValue(value);
    const url = `https://www.omdbapi.com/?s=${value}&apikey=db99467f`;

    try {
      const response = await fetch(url);
      const movieData = await response.json();
      console.log(movieData);
      setMovieDatas(movieData.Search);
    } catch (err) {
      console.log("error", err);
    }
  };

  return (
    <div className="App">
      <input
        type="search"
        value={searchValue}
        onChange={handleSearchValueChange}
      />
      <h1>Movie API Test</h1>
      {movieData &&
        movieData.map((movie) => {
          console.log(movie);
          return <h2>{movie.Title}</h2>;
        })}
    </div>
  );
}
```

![debounce 적용전](https://user-images.githubusercontent.com/45479309/194572911-028f49d6-6738-4d3f-8082-3f916dd55ee3.gif)


code sandbox에서 테스팅하였으며 Input 검색어창에 영화이름을 입력하면 해당 키워드에 맞는 정보를 영화 api서버로부터 받아서 타이틀들을 찍어주는 로직이다. debounce를 적용하기 이전이라 네트워크탭을 보게되면 키워드를 입력할때마다 api 요청이 전송되는게 보인다.

### `debounce 적용 전`  

그럼 이번에는 debounce를 적용하여 입력할때마다 api 요청을 보내는게 아닌 0.5초 간격으로 api를 전송되도록 수정해보자.  

```javascript
export default function App() {
  const [searchValue, setSearchValue] = useState("");
  const [debouncedValue, setDebouncedValue] = useState("");
  const [movieData, setMovieDatas] = useState([]);

  useEffect(() => {
    const time = setTimeout(() => {
      setDebouncedValue(searchValue);
    }, 500);

    return () => clearTimeout(time);
  }, [searchValue]);

  useEffect(() => {
    console.log(debouncedValue);
    const getFetchMovieData = async () => {
      const url = `https://www.omdbapi.com/?s=${debouncedValue}&apikey=db99467f`;
      const response = await fetch(url);
      const movieData = await response.json();
      console.log(movieData);
      setMovieDatas(movieData.Search);
    };
    getFetchMovieData();
  }, [debouncedValue]);

  ....
}
```

### `debounce 적용 후`  

![dbounce 적용후](https://user-images.githubusercontent.com/45479309/194573167-e70de85f-2f6e-4ff8-99e3-e4e7370ab5b6.gif)

debounce된 값을 저장하는 상태값을 하나두어서 0.5초마다 입력되는 값을 계속 debouncedValue에 저장시켰다.  

그리고 0.5초마다 변경되는 debouncedValue 상태 변화가 일어날때마다 useEffect 함수로 하여금 fetch 요청 api 함수인 getFetchMovieData 호출함으로써 api 요청을 보내도록 하였다. 위 그림과 같이 이전 debounce를 적용하기 이전보다 키워드를 매번 입력할때가 아닌 어느정도 입력이 되었을때 네트워크에 api를 보내어 요청 횟수가 현저히 적어진걸 볼 수가 있다.  

그럼 이제 이 debounce를 검색창이 있는 모든페이지에서 적용할 수 있도록 custom hooks로 빼서 다뤄보자.

### `custom hook - useDebounce`  

#### useDebounce.js
```javascript
import { useState, useEffect } from "react";

export default function useDebouncedValue(input, time = 500) {
  const [debouncedValue, setDebouncedValue] = useState(input);

  useEffect(() => {
    const timeout = setTimeout(() => {
      setDebouncedValue(input);
    }, time);

    return () => {
      clearTimeout(timeout);
    };
  }, [input, time]);

  return debouncedValue;
}
```

useDebouncedValue Hooks로 따로 분리해서 인자로는 input에 입력되는 키워드 값과 debounce를 적용할 시간을 받고 useState로 debouncedValue 값을 다루도록 하였다.

#### App.js

```javascript
import { useState, useEffect } from "react";
import "./styles.css";
import useDebouncedValue from "./useDebouncedValue";

export default function App() {
  const [searchValue, setSearchValue] = useState("");
  const debouncedValue = useDebouncedValue(searchValue, 500);
  const [movieData, setMovieDatas] = useState([]);

  useEffect(() => {
    console.log(debouncedValue);
    const getFetchMovieData = async () => {
      const url = `https://www.omdbapi.com/?s=${debouncedValue}&apikey=db99467f`;
      const response = await fetch(url);
      const movieData = await response.json();
      setMovieDatas(movieData.Search);
    };
    getFetchMovieData();
  }, [debouncedValue]);

  const handleSearchValueChange = async ({ target: { value } }) => {
    setSearchValue(value);
  };

  return (
    <div className="App">
      <input
        type="search"
        value={searchValue}
        onChange={handleSearchValueChange}
      />
      <h1>Movie API Test</h1>
      {movieData &&
        movieData.map((movie) => {
          console.log(movie);
          return <h2>{movie.Title}</h2>;
        })}
    </div>
  );
}
```

![useDebounce](https://user-images.githubusercontent.com/45479309/194573276-fccec649-6390-42df-832b-241a9fd3d6e0.gif)

debouncedValue 값을 Hooks로 분리한 useDebouncedValue에서 불러와서 사용하는 것 말고는 달라진 로직은 없다. 디바운싱 기능을 custom hooks 로 적용해보았는데, 확실히 재사용할 여지가 있거나 독립적인 코드들은 저렇게 따로 분리하는게 훨씬 좋다고 느껴진다.


---

### `마무리`

이처럼 검색창 debounce 로직이나 API를 호출하는 로직을 작성, input 을 관리하는 기능, 애니메이션 관련 기능들도 커스텀 훅으로 만들어서 사용되어진다.


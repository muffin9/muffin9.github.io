---
permalink: /project/wetube-pug
title: "Template Engine - Pug"
date: 2021-11-26T09:11-08:01
tags:
  - project
---

### `Template Engine - Pug`

Pug는 html을 쉽게 생성해주는 도구로 express/js템플릿 언어중 인기가 가장 많다. pug는 상속기능 믹스인, 변수기능 등을 사용할수 있기 때문에 조금만 pug문법을 공부하고 사용한다면 높은 생산성을 발휘할수가 있다.

따라서 Express서버를 구축하고 이번에 View단을 Pug를 사용하여 만들어보았는데 Express기반에서 Pug을 셋팅하는건 매우 간단하다.

express server를 구동하는 소스파일 내에서 아래 2줄만 추가해주면된다.

```javascript
app.set("view engine", "pug");
app.set("views", process.cwd() + "/src/views");
```

위에서 1번째라인은 템플릿 뷰 엔진으로 pug라는 모듈을 사용하겠다는 의미이며

2번째 라인은 경로를 설정해주는 부분인데 저렇게 설정을 안해주면 pacakge.json파일이 src폴더 밖에 있기 때문에 에러가 난다. 뭐 물론 views폴더를 src밖에 위치 시켜줄수도 있지만 위처럼 설정해주면 파일구조가 깔끔해지기 때문이다!

express가 pug파일을 response로 보낼때 pug파일을 html파일로 컴파일하여 사용자는 html을 보게 된다. 개인적으로 이번에 pug를 처음 사용해봤는데 장점은 다음과 같았다.

1. 파일을 상속받아 재사용할수 있는 부분 / block을 사용하여 재사용함과 동시에 각 파일마다 원하는 형태로 내용을 만들수 있는 부분
2. 서버쪽에서 변수들을 전달하여 간편하게 사용할 수 있다는 점이 매우 매력적이였다.

뭐 물론 장점이 있으면 단점도 있기 마련인데.. pug 문법은 파이썬처럼 띄어쓰기 기준으로 작성을 해야하기때문에 가끔씩 pug파일에 내용이 많아질때 헷갈려서 불편함이 있었다.

`pug의 장점`들을 잘 보여주는 샘플 파일이다.

```html
extends base.pug include mixins/video block content h2 Welcome here you will see
the trending videos each video in videos +video(video) else li Sorry nothing
found.
```

모든 페이지에서 공통 레이아웃을 사용되는 부분을 base.pug파일에 작성하여 확장시켰으며 많이 사용되어지는 video부분을 믹스인을 사용하여 재사용하였다.

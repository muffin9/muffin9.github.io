---
permalink: /sideProject/wetube-webpack
title: "Webpack"
date: 2021-12-04T13:13-14:02
categories:
  - SideProject
tags:
  - SideProject
---

## WebPack ?

웹팩은 여러 개의 파일을 하나의 파일로 합쳐주는 모듈 번들러다.

대부분의 프레임워크에는 웹팩이 내장되어 있어서 웹팩을 건드릴 일이 많이 없지만 웹팩은 현재 업계 표준이며 대략적으로 어떻게 돌아가는지는 알아야 하므로 정리를 하였다.

### `웹팩의 핵심 기능`

1. jpg 같은 이미지 파일의 크기를 압축할 수 있다.
2. 어느 버전이든 호환이 가능하도록 오래된 버전의 js, css로 컨버팅 해준다.

> 먼저 webpack을 설정하기 위해선 webpack, webpack-cli 모듈들을 설치해주고 webpack.coonfig.js 파일을 생성한다! 웹팩관련해선 여기서 작업해 주면 된다. (파일명을 다르게 줄 수도 있지만 지금은 기본적으로 웹팩이 인식하도록 하기 위하여 webpack.coonfig.js로 설정 )

참고로 해당 파일은 오래된 js 문법만 인식한다. 최신 문법을 사용하려면 또 따로 설정을 해줘야 하는데 우선 이건.. 난 스킵했다.

### `webpack.config.js`

```javascript
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const path = require("path");

// 보편적으로 entry, output, rules, 변형할 파일 구조로 이루어져있음
module.exports = {
  entry: "./src/client/js/main.js", // 컨버팅할 파일 경로
  plugins: [
    new MiniCssExtractPlugin({
      filename: "css/styles.css",
    }),
  ],
  mode: "development",
  watch: true,
  output: {
    filename: "js/main.js", // 컨버팅되어 나올 파일 이름
    path: path.resolve(__dirname, "assets"), // __dirname : 절대경로 , resolve함수가 뒤의 assets, js를 결합시켜줌
    clean: true, // output folder를 빌드 시작하기전에 clean작업
  },
  module: {
    // rules ? 각각의 파일 종류에 따라 어떤 전환을 할 건지 결정
    // babel-loader를 사용하기 위해 필요한 패키지 : babel-loader, @babel/core, @babel/preset-env webpack
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: "babel-loader",
          options: {
            presets: [["@babel/preset-env", { targets: "defaults" }]],
          },
        },
      },
      {
        // scss-loader : scss -> css 컨버팅
        // css-loader : @import, url를 풀어서 해석
        // MiniCssExtractPlugin : css만 추출해서 DOM에 주입
        // 순서는 역순으로 적어줘야 정상적으로 동작
        test: /\.scss$/,
        use: [MiniCssExtractPlugin.loader, "css-loader", "sass-loader"],
      },
    ],
  },
};
```

주석으로 어느 정도 설명을 적어놨는데 핵심적인 부분을 따로 정리하였다.

### `Entry`

최초 진입 점으로 내가 컨버팅할 파일의 시작점 경로를 지정하는 옵션으로 기본값은 ./src/index.js  
js 파일로 main, videoPlayer 파일들 이름을 각각 지정 후 output의 filename 속성에 [name] 으로 적어주면 각 이름에 맞게 파일들이 컨버팅된다.

### `output`

웹팩으로 인하여 번들링 된 결과물을 어디에 위치할지 명명하는 곳이다. path.resolve 함수는 절대 경로 주소를 시작으로 뒤의 인자들을 결합하여 path를 지정해 준다.

### `loader`

파일들을 해석하고 변환하는 과정을 전체적으로 관여하는데 js, json 파일을 기본 모듈로 보며 css와 같은 다른 모듈을 사용하기 위해선 css 관련 loader를 설정해 줘야 함

\*\* 이제 웹팩을 사용하여 파일을 변환시키면 되는데 터미널에 webpack --config webpack.config.js를 입력하면 assets 폴더 아래에 js, css 파일들이 생성된다!

---

![webpack](/assets/image/wetube/webpack.png)

프론트단에서 실행할 파일들은 client 폴더에서 관리하도록 하였으며 간단한 테스트를 위해 client/js 아래에 main.js 파일을 생성하였다.

### `main.js`

```javascript
import "../scss/styles.scss";
```

main 파일에서는 따로 로직은 없고 scss 파일을 컨버팅하기 위한 import 소스만 들어가있다.

이제 뷰단에서 컨버팅된 js, css 파일을 사용만 하면 된다. 설정은 모든 레이아웃에서 공통으로 사용되어지는 base.pug에 경로를 입력해 주면 된다.

```html
doctype html html(lang="ko") head title #{pageTitle} | Wetube
link(rel="stylesheet",
href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.3/css/all.min.css")
link(rel="stylesheet", href="/static/css/styles.css") // Css 코드 가져오기 body
include partials/header main block content include partials/footer.pug block
scripts
```

static 경로 아래에 접근하도록 되어있는데 웹팩으로 만들어진 css,js는 assets 폴더에 있지만 경로를 static으로 주고 assets 폴더의 파일들에 접근할 수 있도록 express 서버에 아래 코드 추가

### `server.js`

```javascript
app.use("/static", express.static("assets"));
```

---

마지막으로 노드몬으로 인해 서버쪽에서 저장할 시 서버가 재기동되는데 프론트단에서 관리되어지는 소스에서 작업 시에는 서버가 재시동 되면 매우 불편하기 때문에 nodemon.json 파일에 ignore 속성에 무시할 파일 경로와 감지되었을 때 수행할 명령은 exec에 입력

### `nodemon.js`

```json
{
  "ignore": ["webpack.config.js", "fixtures/*", "src/client/*", "assets/*"],
  "exec": "babel-node src/init.js"
}
```

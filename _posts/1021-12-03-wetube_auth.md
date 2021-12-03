---
permalink: /sideProject/wetube-auth
title: "Wetube-Authentication"
date: 2021-12-02T04:12-04:08
categories:
- SideProject
tags:
- SideProject
--- 

### USER AUTHENTICATION

> 이번 글은 유저 로그인과 관련하여 비밀번호 암호화(bcrypt) & 소셜 로그인중 하나인 깃헙로그인 연동 방법에 관련하여 정리하였습니다.  
> 우선 깃헙로그인 연동후 어느 정도 프로젝트가 완성될 때쯤 카카오톡 소셜 로그인을 한 번 붙여볼 생각이다!  
> Video model을 만들어 준거처럼 User model도 똑같이 만들어 주면 된다.

```javascript
import mongoose from "mongoose";
import bcrypt from "bcrypt";

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  avatarUrl: String,
  socialOnly: { type: Boolean, default: false }, // email로 로그인하려는데 password가 없을 때 유용
  username: { type: String, required: true, unique: true },
  password: { type: String },
  name: { type: String, required: true },
  location: String,
});

userSchema.pre("save", async function() {
  console.log("User password : " + this.password); // 그대로 출력
  this.password = await bcrypt.hash(this.password, 5);
  console.log("Hashed password : " + this.password); // 전달받은 패스워드를 5번 해싱함
})

const User = mongoose.model("User", userSchema);
export default User;
```

videoSchema를 만들어서 model를 export 한 거처럼 똑같이 User 또한 model를 만들어준 다음 export 시켰다!  

추가적으로 일 방향 해싱함수인 bcrypt 모듈을 사용하여 유저 객체가 저장되기 이전에 전달받은 암호를 해싱하도록 미들웨어 pre 함수를 세팅하였다.  

---

### Join & Session & Login

#### 1. Join
```javascript
export const postJoin = async (req, res) => {
  const { name, email, username, password, password2, location } = req.body;
  const pageTitle = "Join";
  if(password !== password2) {
    return res.render("join", {
      pageTitle,
      errorMessage : "패스워드 불일치"
    })
  }

  const exists = await User.exists({ $or : [ { email }, { username } ]});
  if(exists) {
    return res.render("join", {
      pageTitle,
      errorMessage: "This username/email is already taken.",
    })
  }

  await User.create({
    name,
    email,
    username,
    password,
    location
  });
  return res.redirect("/login");
};
```

➡️️ 회원가입 부분은 유저를 저장시키기 위한 중복체크 조건이 추가된 거 말고는 비디오를 업로드하는 부분이랑 로직이 거의 비슷한데..  
먼저 뷰단에서 Post 형식으로 데이터를 전달하면 서버 쪽에서 해당 데이터들을 받은 다음 password가 일치하는지 체크하는 부분과 이메일, 닉네임 중복 체크를 통과하게 되면 User 객체가 디비에 저장되도록 하였다.

---

#### 2. Session

Login 쪽 소스를 설명하기 전에 세션이라는 개념을 짚고 넘어가야 하는데 세션은 브라우저와 백엔드 간에 어떤 활동을 했는지 기억하는 개념이다.  

웹은 데이터를 주고받기위하여 HTTP 프로토콜을 사용하는데 이 프로토콜은 요청에 대한 응답을 하면 연결이 끊어지므로(stateless 한성질) 이걸 계속 유지하기 위해서는 세션을 사용해야 한다.  

여기 미들웨어로 express에서 session을 사용하기 위해서 제일 먼저 express-session 모듈을 사용하였다.  

이제 express가 알아서 요청한 브라우저를 위한 세션 id를 만들어 브라우저에게 보내주는데 해당 브라우저는 쿠키에 전달받은 세션 id를 저장하고, express에서도 그 세션을 세션 DB에 저장하게 된다. 주절주절 설명이 길었는데 요약하자면 아래와 같다.  

1. 브라우저 접근 시 → 서버(express)에 접근
2. 서버에서 브라우저에게 쿠키 전달
3. 브라우저에서 서버 쪽에 요청할 때 위에서 받은 쿠키를 함께 보냄
4. 서버는 해당 쿠키를 통하여 사용자 구분

```javascript
export const localsMiddleware = (req, res, next) => {
    res.locals.loggedIn = Boolean(req.session.loggedIn);
    res.locals.loggedInUser = req.session.user;
    next();
}
```

세션을 관리하는 localsMiddleware 파일을 생성하여 세션을 세팅하였다. res.locals의 프로퍼티로 위와 같이 설정하면 Pug 파일에서도 locals라는 속성으로 설정한 loggedIn loggedInUser를 사용할 수 있기 때문!  


### `server.js`

```javascript
import session from "express-session";
import MongoStore from "connect-mongo";
import { localsMiddleware } from "./localsMiddleware";

app.use(
    session({
      secret: process.env.COOKIE_SECRET,
      resave: false,
      saveUninitialized: false, // false일시 세션을 수정할때마다 세션을 DB에 저장
      store: MongoStore.create({ mongoUrl: process.env.DB_URL }), // DB Session에 세션 데이터 저장
    })
  );
app.use(localsMiddleware);
```

>express에서 세션을 사용하기 위하여 위 코드들만 server.js에 따로 추가해 주면 됩니다.  
>세션 관련 option들을 세팅하였는데 secret, store 부분은 다른 사람들이 알면 안 되는 정보로 노출시키지 않기 위해 .env 파일을 따로 만들어서 proccess.env로 설정하였고 라우터를 연결하기 전에 위에서 세팅한 미들웨어를 연결시켰다. 

>서버가 종료되고 다시 시작하여도 로그인이 되기 위해선 디비에 세션을 저장할 필요가 있는데 그러기 위해선 express와 mongodb를 연결시켜주는 connect-mongo 모듈을 npm으로 설치 후 위처럼 MongoStore를 임포트 시키고 session에 설정값으로 store를 세팅해주면 된다. 서버를 재시작하면 몽고 디비에서 sessions 디비가 생성되고 신기하게도 여기서 로그인한 사용자들이 계속해서 관리된다.  

---

#### 3. Login

```javascript
export const postLogin = async (req, res) => {
  const { username, password } = req.body;
  const pageTitle = "Login";
  const user = await User.findOne({username, socialOnly: false});
  if(!user) {
    return res.status(400).render("login", {
      pageTitle,
      errorMessage: "존재하지 않는 회원입니다.",
    })
  }

  const passwordOk = await bcrypt.compare(password, user.password);
  if(!passwordOk) {
    return res.status(400).render("login", {
      pageTitle,
      errorMessage: "Wrong Password!",
    })
  }
  
  req.session.loggedIn = true;
  req.session.user = user;
  return res.redirect("/");
}
```

POST 메서드로 데이터들을 받아온 후에 전달받은 username이 디비에 있는지 찾고 해당 유저가 없다면 로그인 페이지를 다시 그려주고 errorMessage를 보이도록 하였다.  

반대로 user가 있다면 bcrypt.compare 함수로 패스워드를 체크하고 패스워드 불일치 시에도 로그인 페이지를 그려주고 패스워드 일치가 통과되면 session 객체에 loggedIn 값을 true, user 값에는 위에서 뽑은 user 객체 그대로 담도록 하였다.  

---

### GitHub 인증 과정

1. Github에게 이메일 / 패스워드 전달 -> 로그인 과정
2. Github에서 비밀번호, 보안, 이메일 인증 처리
3. 승인 완료 시 token과 함께 웹사이트로 리다이렉트
4. 위에서 받은 toekn으로 내가 필요한 api요청

대략적으로 깃헙 로그인 인증 과정은 위 프로세스 형태로 진행된다.  

>여기서 중요한 점은 깃헙 api가 제공하는 대로 사용을 해야 한다. 깃헙 api를 사용하기 위해서는 먼저 [https://github.com/settings/developers](https://github.com/settings/developers) 링크에서 아래와 같이 Oauth App을 생성해야 한다.

![wetube_github](/assets/image/wetube/github.png){: .width-half}

callbackURL에는 express 서버에서 url 을 따로 만들어줘야 하는데  
(위처럼 설정해도 되고 다른 url을 사용해도 무방하지만 처음에 github 연동을 위해서는 /github/start url를 사용하기 때문에 가급적 이 url과 비슷하게 작성하도록 하자)  
로그인 인증 후 access_token을 받아서 다시 이 토큰으로 User api 정보를 가져오는 과정을 거쳐야 한다. client_id, client_secret 정보는 노출하면 안 되는 정보로 env 파일에서 관리하도록 하자.  

#### `GitHub인증과 관련한 전체적인 소스다.`

```javascript
import fetch from "node-fetch";

export const startGithubLogin = (req, res) => {
  const baseUrl = "https://github.com/login/oauth/authorize";
  const config = {
    client_id: process.env.GH_CLIENT,
    allow_signup: false,
    scope: "read:user user:email", // 필요한 데이터 scope
  };
  const params = new URLSearchParams(config).toString();
  const finalUrl = `${baseUrl}?${params}`;
  return res.redirect(finalUrl);
};

export const finishGithubLogin = async (req, res) => {
  const baseUrl = "https://github.com/login/oauth/access_token";
  const config = {
    client_id: process.env.GH_CLIENT,
    client_secret: process.env.GH_SECRET,
    code: req.query.code,
  };
  const params = new URLSearchParams(config).toString();
  const finalUrl = `${baseUrl}?${params}`;
  // fetch를 통해 데이터 받아옴
  const tokenRequest = await(await fetch(finalUrl, {
    method: "POST",
    headers: {
      Accept: "application/json",
    },
  })).json();

  if ("access_token" in tokenRequest) {
    const { access_token } = tokenRequest;
    const apiUrl = "https://api.github.com";
    // userProfile api
    const userData = await (
      await fetch(`${apiUrl}/user`, {
        headers: {
          Authorization: `token ${access_token}`,
        },
      })
    ).json();
    // userEmail api
    const emailData = await (
      await fetch(`${apiUrl}/user/emails`, {
        headers: {
          Authorization: `token ${access_token}`,
        },
      })
    ).json();
    const emailObj = emailData.find((email)=> email.primary === true && email.verified === true);
    // 깃헙 로그인으로 로그인했을때 이미 디비에 같은 이메일을 가진 유저가 있다면? 해당 유저 로그인처리
    if(!emailObj) {
      return res.redirect("/login");
    }
    let user = await User.findOne({email: emailObj.email});

    if(!user) {
      // db에 github email이 없네?
      user = await User.create({
        avatarUrl: userData.avatar_url,
        name: userData.name,
        username: userData.login,
        email: emailObj.email,
        password: "",
        socialOnly: true,
        location: userData.location,
      })
    }
    req.session.loggedIn = true;
    req.session.user = user;
    return res.redirect("/");
  } else {
    return res.redirect("/login");
  }
};
```

GitHub에서 제공하는 api 요청에 맞게 config를 세팅하고 데이터가 정상적으로 들어왔는지 확인하면 되는데 express 단에서 비동기로 데이터를 요청하기 위해선 node-fetch 모듈을 install 하여 사용해야 한다.  

github api가 요구하는 대로 사용한거라 아마 api 문서를 보는게 더 도움이 될 것이라 생각합니다. [https://docs.github.com/en/rest/guides/getting-started-with-the-rest-api](https://docs.github.com/en/rest/guides/getting-started-with-the-rest-api)

뭐 그래도 강의를 듣고 정리한 소스를 조금 설명하자면..➡️  

1. startGithubLogin 함수에서는 내가 필요로 하는 데이터들을 뽑기 위해 config의 scope 속성으로 설정후 finalUrl, params를 붙여 리다이렉트!  

2. finishGithubLogin 함수는 위 startGithubLogin 함수에서 client_id로 깃헙 계정 인증 후에 호출되는 콜백 함수로 여기선 access_token을 얻기 위하여 그에 맞는 api를 호출한다.  

3. tokenRequest 변수에 access_token이 존재하면 이제 이 토큰은 특정 계정의 토큰이므로 위에서 설정한 scope의 데이터들을 뽑아올 수가 있다.(여기선 로그인을 시도한 유저정보와 이메일 정보!)  

4. 디비에 이미 깃헙 로그인을 시도한 이메일이 존재한다면 로그인을 시키고 없다면 유저 하나를 생성하여 저장시킨 후 마찬가지로 로그인을 시키도록 하였다.



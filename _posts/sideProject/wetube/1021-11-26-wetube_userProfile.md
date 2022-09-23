---
permalink: /wetube/wetube-userProfile
title: "Template Engine - Pug"
categories: wetube
tags: JavaScript Wetube Project

toc: true
toc_sticky: true


date: 2022-02-03
last_modified_at: 2022-02-03
---

### `USERPROFILE`

이번 USER PROFILE에서는 비밀번호 찾기나 유저의 내용들을 업로드하는 기능들은 생략하였는데 왜냐하면 앞에서 나온 router를 설정한 이후에 controller 단에서 기능을 작업하고 pug 파일을 뿌려주는 흐름이 매우 비슷했기 때문!  
➡️ 따라서 보다 더 핵심적이라고 생각되는 다음 아래 기능들을 정리하였다.

✅ 로그인 여부에 따른 접근 권한  
✅ 유저 프로필, 비디오 파일 업로드 기능  
✅ 유저와 비디오 일대다 관계 매핑

### `로그인 여부에 따른 접근 권한`

`localsMiddleware`

```javascript
export const protectorMiddleware = (req, res, next) => {
  if (req.session.loggedIn) {
    return next();
  } else {
    return res.redirect("/login");
  }
};

export const publicOnlyMiddleware = (req, res, next) => {
  if (!req.session.loggedIn) {
    return next();
  } else {
    return res.redirect("/");
  }
};
```

로그인 여부에 따라 접근할 수 있도록 미들웨어를 따로 만들었다. 예를 들어 로그인과 회원가입 같은 경우엔 로그인된 유저가 접근 시 home으로 보내기 위하여 publicOnlyMiddleware 미들웨어를 태웠다.

---

### `파일 업로드 기능`

1. `server.js & middleware.js`

```javascript
app.use("/uploads", express.static("uploads")); // 폴더의 내부를 제공하기 위해서는 static사용
```

먼저 임시적으로 파일을 서버 내 uploads 경로에 저장시키도록 하기 위하여 server.js 파일에 위처럼 설정.

```javascript
export const avatarUpload = multer({
  dest: "uploads/avatars/",
  limits: { fileSize: 3000000 },
});
export const videoUpload = multer({
  dest: "uploads/videos/",
  limits: { fileSize: 300000000 },
});
```

파일 업로드를 위하여 컨트롤러단으로 함수가 타기 전에 미들웨어로 설정을 해줘야 한다. 유저 프로필과 비디오 파일을 추가할 예정이므로 위처럼 저장할 경로(dest), 파일 크기(fileSize)를 지정!

2. `edit-prfoile.pug`

```html
extends base block content img(src="/" + loggedInUser.avatarUrl, width="100",
height="100") form(action="/users/change-image" method="POST",
enctype="multipart/form-data") label(for="avatar") Avatar input(type="file",
id="avatar", name="avatar", accept="image/*") input(type="submit", value="Update
Image") hr if errorMessage span=errorMessage form(method="POST")
input(placeholder="Name", name="name", type="text", required,
value=loggedInUser.name) input(placeholder="Email", name="email", type="email",
required, value=loggedInUser.email) input(placeholder="Username",
name="username", type="text", required, value=loggedInUser.username)
input(placeholder="Location", name="location", type="text", required,
value=loggedInUser.location) input(type="submit", value="Update Profile") if
!loggedInUser.socialOnly hr a(href="change-password") Change password &rarr;
```

유저 프로필을 수정하는 view 파일인데 강의와 다르게 사용자가 이미지만 따로 업로드할 수 있도록 따로 form 태그를 생성하여 작업하였다. 여기서 파일을 백엔드로 보내서 업로드하기 위해서는 form 태그에 enctype 속성 값을 multipart/form-data로 설정해 줘야 한다.

3. `userRouter.js`

```javascript
userRouter.post(
  "/change-image",
  protectorMiddleware,
  avatarUpload.single("avatar"),
  postImageEdit
);
```

change-image url로 post Method에서 파일 업로드 기능이 동작하므로 postImageEdit 컨트롤러로 가기 전에 avatarUplad 함수를 태우도록 하였다. single 함수는 multer 내부에서 지원해 주는 함수로 하나의 파일만 업로드 되도록 설정하는 의미이며 값으로는 edit-profile에서 input의 name에서 지정한 값인(avatar)를 넣어주면 된다!

✨ input으로 avatar 파일을 받아서 uploads 폴더에 저장한 다음 해당 파일 정보를 컨트롤러인 postEdit에 전달해 준다.

4. `userController.js → postEdit`

```javascript
export const postImageEdit = async (req, res) => {
  const {
    session: {
      user: { _id, avatarUrl },
    },
    file,
  } = req;

  const updatedUser = await User.findByIdAndUpdate(
    _id,
    {
      avatarUrl: file ? file.path : avatarUrl,
    },
    { new: true }
  );
  req.session.user = updatedUser;
  return res.redirect("/users/edit");
};
```

이미지만 변경할 수 있도록 postImageEdit를 따로 만들었으며, 여기서 주의할 것이 몇 가지가 있다..

`What ?` 유저가 프로필을 업데이트할 때 파일을 업로드할 수 도 있고 안 할 수도 있기 때문에 이 2가지 경우를 잘 처리해 줘야 한다.

1. 로그인했을 때 session 객체에서 avatarUrl와 req에서 file을 가져온다.
2. 유저를 업데이트할 때 유저가 사진을 재업로드 할 시 file 객체를 전달받는데 이때 해당 file.path가 담긴다.
3. file 객체가 없다는 건 사진은 업로드 안 했다는 것으로 기존 avatarUrl을 사용하도록 설정하였다.
4. 마지막으로는 업데이트된 유저를 다시 세션에 저장시킨다.

---

### `유저와 비디오 일대다 관계 매핑`

우선 유저와 비디오 간 관계를 매핑 한 이유? 자신이 업로드한 비디오를 업로드한 유저만 수정하거나 삭제할 수 있도록 하기 위함 & 유저 프로필에 자신이 업로드한 비디오만 보이도록

유저는 여러 개의 비디오를 가질 수 있으며 비디오는 하나의 유저에만 속하므로 유저:비디오 = 1:다로 관계라 말할 수 있는데 mongoose로 관계를 매핑하기 위해서는 각 스키마에서 타입으로 mongoose.Schema.Types.ObjectId를 설정하고 ref 속성으로 각 model 명을 적어주면 된다.

```javascript
// userSchema
videos: [{ type: mongoose.Schema.Types.ObjectId, ref: "Video" }],

// videoSchema
owner: { type: mongoose.Schema.Types.ObjectId, required: true, ref: "User" }, //
```

위와 같이 각 모델에서 위 코드를 추가해 주면 관계 매핑이 정말 간단하게 설정이 된다. (유저는 비디오를 꼭 가지지 않아도 되므로 required 속성을 제외해 준다.)

다음은 위에서 설정한 관계 매핑을 어떻게 사용하는지 비디오 쪽 예제를 몇 가지 봐보자

`videoController -> watch함수`

```javascript
export const watch = async (req, res) => {
  const id = req.params.id;
  const video = await Video.findById(id).populate("owner");

  if (!video) {
    return res.status(404).render("404", { pageTitle: "Video not found." });
  }
  return res.render("watch", { pageTitle: video.title, video });
};
```

✨ 홈 화면에서 글을 클릭했을 때 동작하는 함수인데 user의 고유 id로 findById 함수를 동작시킬 때 populate 함수를 연결시키는 부분이 중요하다. populate 함수란 몽구스에서 mongoose.SchemaTypes.ObjectId로 접근하여 user 모델에 접근하여 user 객체를 모두 담도록 해준다.

`watch.pug`

```javascript
extends base.pug

block content
    video(src="/" + video.fileUrl, controls)
    div
        p=video.description
        small=video.createdAt
    div
        small Uploaded by
            a(href=`/users/${video.owner._id}`)=video.owner.name
    if String(video.owner._id) === String(loggedInUser._id)
        a(href=`${video.id}/edit`) Edit Video &rarr;
        br
        a(href=`${video.id}/delete`) Delete Video &rarr;
```

찾은 video를 pug 파일에 넘겨줌으로써 watch 파일에서는 해당 video 객체를 사용할 수가 있는데 조건문으로 owner의 \_id가 지금 로그인한 유저의 id일 시 해당 비디오를 업로드한 사람을 의미함으로 수정과 삭제하는 기능을 보여주도록 하였다.

`videoController -> deleteVideo함수`

```javascript
export const deleteVideo = async (req, res) => {
  const id = req.params.id;
  const {
    user: { _id },
  } = req.session;
  const video = await Video.findById(id);
  const user = await User.findById(_id);
  if (!video) {
    return res.status(404).render("404", { pageTitle: "Video not found." });
  }
  if (String(video.owner) !== String(_id)) {
    return res.status(403).redirect("/");
  }
  await Video.findByIdAndDelete(id);
  user.videos.splice(user.videos.indexOf(id), 1);
  user.save();
  return res.redirect("/");
};
```

사용자가 video를 삭제할 때 동작하는 컨트롤러 함수인데 제일 먼저 video와 user를 찾고 해당 video 객체가 존재하는지? 그 비디오의 소유자가 현재 로그인한 사용자인지 체크한다.

위 2가지 조건문을 모두 통과하면 삭제할 수 있다는 의미이므로 Video 모델에서 해당 id를 가진 video를 지우고 동시에 유저에서 매핑되는 데이터도 삭제해야 하기 때문에 user의 videos 객체에 접근하여 indexOf 함수로 그 비디오 id를 찾아 1개만 정확히 삭제하도록 기존의 배열을 수정하는 splice 함수를 사용했다.

이 유저에서 삭제된 비디오도 같이 어떻게 삭제할지 생각하는 게 좀 시간이 많이 소요되었다.  
이젠 splice 함수로 1개가 제거되었으니 다시 save 함수로 유저를 저장시키면 끝!

\*\* 이번 강의를 끝으로 백엔드 부분이 어느 정도 완성되었는데 많은 것을 얻어 갔다고 생각한다! 그중에 가장 매력적인 기능이라고 생각하는 부분은?

1. 서버를 세팅하는데 express 프레임워크를 사용하였는데 이전에도 한 번 사용해 보았지만 정말 간편하다고 생각한다. 간단한 get, post 라우터 설정부터 시작하여 미들웨어와의 결합으로 생산성이 정말 좋다!
2. 로그인 처리 부분을 백단에서 세션으로 관리해서 로그인을 해야 접근할 수 있는 url을 미들웨어로 로그인 여부를 결정하는 점
3. 뷰 템플릿 엔진으로 pug를 사용하였는데 간단하나 설정만으로 서버단에서 뷰단을 생성해 주고 백단에서 변수들을 설정해서 전달해 줄 수 있을 뿐만 아니라 데이터 객체인 locals로 로그인한 유저나 다른 데이터들을 설정해 줄 수 있다는 점이 유용하다 생각한다. 하지만, render 시 변경될 때마다 다시 컴파일을 해야 한다는 점에서 아마 서버 쪽에서 부담이 될 수 있다고 생각이 든다.

✨ 프론트단으로 작업을 진행하기 전에 몇 가지 기능을 추가해 보려 한다.
✅ 소셜 로그인으로 로그인한 유저는 정보 변경 못하도록 안내하도록 구현
✅ 에러 발생 또는 로그인하지 않는 사용자가 로그인해야만 갈 수 있는 url 접근 시 팝업 형태로 안내하도록 구현
✅ 카카오톡 소셜 로그인 구현

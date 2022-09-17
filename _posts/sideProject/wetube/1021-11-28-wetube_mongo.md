---
permalink: /project/wetube-mongo
title: "Wetube-MongoDB"
date: 2021-11-28T00:11-02:11
tags:
  - project
---

### `MongoDB`

데이터베이스에 데이터를 저장하기 위해서 NoSQL 데이터베이스인 MongoDB를 사용하였다. MongoDB는 JSON 형태의 데이터를 저장하는 도큐먼트 지향 데이터베이스로 SQL와 달리 굉장히 유동적인 녀석이다..

여기 강의에서는 MongoDB를 깊게 설명하지는 않고 Express API서버에서 데이터베이스를 연결하는 방법부터 스키마를 생성하고 해당 스키마를 이용하여 데이터를 저장하는 정도로 설명하고 있다.

왜 몽고 디비를 사용하는지 찾아봤는데 성능과 확장성 면에서 NoSQL기반인 MongoDB가 DB로 많이 쓰이는 Mysql과 같은 RDBMS에 비해 좋긴하지만 ACID 트랜잭션같은 작업을 위해서는 RDBMS를 쓰는 편이 더 좋다.

지금은 몽고DB를 한 번 맛보기위함이고 이제 Express서버에 mongodb를 설치해서 연결해보자.

```javascript
import mongoose from "mongoose";

mongoose.connect("mongodb://127.0.0.1:27017/wetube");

const db = mongoose.connection;

const handleOpen = () => console.log("✅ Connected to DB");
const handleError = (error) => console.log("❌ DB Error", error);

db.on("error", handleError);
db.once("open", handleOpen);
```

Express서버에서 몽고 디비를 사용하기 위해서는 mongoose라는 모듈을 설치해줘야한다. mongoose는 자바스크립트를 사용하는 node.js와 mongoDB를 이어주는 다리로 생각하면 편할 것 같다.

handleOpen, handleError 함수로 db가 정상적으로 연결되었는지 on, once함수에 콜백함수로 등록해주었다. on, once함수의 차이는 once함수는 한 번만 실행된다는 점!

`Video Model`

```javascript
import mongoose from "mongoose";

export const formatHashtags = (hashtags) =>
  hashtags.split(",").map((word) => (word.startsWth("#") ? word : `#${word}`));

const videoSchema = new mongoose.Schema({
  title: { type: String, required: true, trim: true, maxLength: 80 },
  description: { type: String, required: true, trim: true, minLength: 20 },
  createdAt: { type: Date, required: true, default: Date.now },
  hashtags: [{ type: String, trim: true }],
  meta: {
    views: { type: Number, default: 0, required: true },
    rating: { type: Number, default: 0, required: true },
  },
});

// model이 생성되기 전에 미들웨어 만들수 있음
// videoSchema.pre("save", async function() {
//   this.hashtags = this.hashtags[0].split(",").map((word) => word.startsWth("#") ? word : `#${word}`);
// });

const Video = mongoose.model("Video", videoSchema);
export default Video;
```

지금까지 더미 데이터를 따로 생성해서 사용했는데 이제는 진짜 데이터를 디비에 저장하기 위해서 위와같이 스키마를 만들어주었다.

Video를 저장하기 위해서 title, description, createdAt, hashtags, meta 데이터를 갖는데 각 데이터에 디테일한 설정들(trim, required 등)을 해줄 수 있으며 또한 타입을 정해주게되면 만약 잘못된 타입이 들어왔을때는 저장이 안되고 에러를 뱉게된다.

또한 model이 생성되기 전에 미들웨어를 만들어 줄 수 있는데 주석 부분처럼 model이 생성되기 전에 미들웨어를 만들려면 pre함수를 통해 생성해줄수 있다.

여기서는 비디오를 생성되기 전 뿐만아니라 수정할때도 해쉬태그를 정상적으로 저장하기 위하여 formatHashtags함수를 따로 만들어서 export하였다.

다음에는 더미 데이터를 사용하는 부분을 진짜 데이터로 저장하기위해서 비디오를 업로드하는 부분을 수정해보자.

```javascript
export const postUpload = async (req, res) => {
  const { title, description, hashtags } = req.body;
  try {
    await Video.create({
      title,
      description,
      hashtags: formatHashtags(hashtags),
    });
    return res.redirect("/");
  } catch (error) {
    return res.render("upload", {
      pageTitle: "Upload Video",
      errorMessage: error._message,
    });
  }
};
```

사용자가 Title, Description, Hastags를 입력하고 Upload 버튼을 눌렀을때 동작되는 API다.

async-await 문법을 사용하여 비동기처리를 동기적으로 처리함으로써 전달받은 데이터들로 Video 모델을 create함수를 호출하게 되면 DB에 저장이 되고 정상적으로 저장이 되면 redirect함수가 발동된다.

만약, create함수에 데이터형식이 안맞거나 다른 에러가 난다면 catch문으로 이동하여 errorMessage에 에러가 담기게되며 화면에는 해당 에러가 보여지게 되도록 설정!

```javascript
export const postEdit = async (req, res) => {
  const id = req.params.id;
  const { title, description, hashtags } = req.body;
  const video = await Video.exist({ _id: id });
  if (!video) {
    return res.render("404", { pageTitle: "Video not found." });
  }
  await Video.findByIdAndUpdate(id, {
    title,
    description,
    hashtags: formatHashtags(hashtags),
  });
  return res.redirect(`/videos/${id}`);
};

export const deleteVideo = async (req, res) => {
  const id = req.params.id;
  await Video.findByIdAndDelete(id);
  return res.redirect("/");
};
```

mongoose에는 create함수 이외에도 model에 대한 여러 함수가 존재하는데 비디오를 수정할때는 exist함수로 전달받은 고유id가 디비에 있는지 체크 이후에 findByIdAndUpdate함수를 사용하였고 어떤 비디오를 삭제할때는 findByIdAndDelete함수를 사용하였다.

데이터를 가공하기위한 다른 함수가 필요하다면 [https://mongoosejs.com/docs/api/model.html](https://mongoosejs.com/docs/api/model.html) 여기서 찾아보자

\*\* MongoDB와 mongoose를 처음 사용해봤는데 Express 서버에 붙이는거도 굉장히 심플하고 디비에 데이터를 접근하는데 mongoose의 여러 api를 사용하여 간편하게 코드를 작성할수 있었고 타입만 지정하면 에러처리도 어느정도 보장된다는게 굉장히 좋다고 느낌..

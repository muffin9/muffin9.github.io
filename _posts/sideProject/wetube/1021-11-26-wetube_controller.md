---
permalink: /project/wetube-controller
title: "Wetube-Controller"
date: 2021-11-26T09:11-08:01
tags:
  - project
---

Express 서버를 구축하면서 Router와 Controller단을 분리하였는데 이번에는 데이터베이스를 사용하기 전에 가짜 객체를 생성하여 Controller를 구축하고 get post 라우터에서 컨트롤러함수를 핸들러로 등록하여 테스트를 진행해보았다.

아래는 강의에서 지금까지 작업한 비디오와 관련한 컨트롤러단쪽 소스다.

```javascript
let videos = [
  {
    title: "First Video",
    rating: 5,
    comments: 2,
    createdAt: "2 minutes ago",
    views: 1,
    id: 1,
  },
  {
    title: "Second Video",
    rating: 5,
    comments: 2,
    createdAt: "2 minutes ago",
    views: 59,
    id: 2,
  },
  {
    title: "Third Video",
    rating: 5,
    comments: 2,
    createdAt: "2 minutes ago",
    views: 59,
    id: 3,
  },
];

export const trending = (req, res) => {
  res.render("home", { pageTitle: "Home", videos });
};
export const search = (req, res) => res.send("Search");

export const see = (req, res) => {
  const id = req.params.id;
  const video = videos[id - 1];
  return res.render("watch", { pageTitle: `Watching: ${video.title}`, video });
};

export const getEdit = (req, res) => {
  const id = req.params.id;
  const video = videos[id - 1];
  return res.render("edit", { pageTitle: `Editing: ${video.title}`, video });
};

export const postEdit = (req, res) => {
  const id = req.params.id;
  const title = req.body.title;
  videos[id - 1].ttile = title;
  return res.redirect(`/videos/${id}`);
};

export const getUpload = (req, res) => {
  return res.render("upload", { pageTitle: "Upload Video" });
};

export const postUpload = (req, res) => {
  const { title } = req.body;
  const newVideo = {
    title,
    rating: 0,
    comments: 0,
    createdAt: "just now",
    views: 0,
    id: videos.length + 1,
  };
  videos.push(newVideo);
  return res.redirect("/");
};

export const deleteVideo = (req, res) => res.send("Delete Video");
```

위 소스에서는 비디오를 업로드하는 postUpload와 가져오는 getUpload 소스만 살펴보면 될 것 같다.

먼저 `getUpload` 핸들러함수를 살펴보자

1. Express router를 생성할때 함수 내부적으로 req, res를 받는다.
2. res의 render함수를 사용하여 첫번째인자에는 pug파일의 upload를 두번째인자에는 upload파일에 넘어가는 여러 변수들을 json형태로 보내게된다.

다음으로 비디오를 업로드 하는 `postupload` 함수

req.body는 form형태에서 post방식으로 input타입의 데이터를 받아올수가 있다.

현재 더미객체를 사용하였기 때문에 새로운 newVideo객체를 생성하여 videos배열에 추가하였고 루트경로로 리다이렉트시키도록 설정하였다.

지금은 컨트롤러단에서 데이터를 가공하고(비즈니스 로직 대부분), view를 뿌려 주는 코드가 모두 들어가있는데 이부분은 클론코딩 강의를 보면서 개인적으로 MVC패턴으로 분리할 생각이다. MVC패턴과 관련해서는 아래 링크에 자세히 정리했습니다.

정말 중요한 개념이니 잘 모르신다면 한 번쯤 관심을 가져보는걸 추천드립니다.

위에서 만든 컨트롤러를 호출하는 라우터쪽 소스다.

```javascript
import express from "express";
import {
  see,
  getEdit,
  postEdit,
  getUpload,
  postUpload,
  deleteVideo,
} from "../controllers/videoController";

const videoRouter = express.Router();

videoRouter.get("/:id(\\d+)", see);
videoRouter.route("/:id(\\d+)/edit").get(getEdit).post(postEdit);
videoRouter.get("/:id(\\d+)/delete", deleteVideo);
videoRouter.route("/upload").get(getUpload).post(postUpload);

export default videoRouter;
```

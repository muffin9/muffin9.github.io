---
permalink: /nextjs/dynamic_routes
title: "Dynamic Routes"
categories: NextJS

tags: NextJS

toc: true
toc_sticky: true


date: 2022-10-17
last_modified_at: 2022-10-17
---

`해당 글은 Next.js Document 의 내용을 참고하여 정리한글입니다.`  

이전에 정적 경로 하나만 두고 블로그 게시물 2개를 가져왔었는데, 이번에는 Next.js의 `getStaticPaths`를 사용하면 동적경로를 생성할 수가 있다고 알고 있어서 이를 한 번 사용하여 각 블로그 게시글로 라우팅이 가능해지도록 해보자.

(Ex. javascript.md, algorithm.md ) 두개의 마크다운 파일이 있다고 가정할 때, /posts/javascript, /posts/algorithm 경로를 만들어보자. (실제로 나는 javascript, algorithm 2개의 마크다운 파일을 생성하였다. 적절하게 title, date 넣어주고 아무 내용이나 입력하였다.)  


`javascript.md`

```md
---
title: 'JavaScript'
date: '2022-10-17'
---

The Document interface describes the common properties and methods for any kind of document. Depending on the document's type (e.g. HTML, XML, SVG, …), a larger API is available: HTML documents, served with the "text/html" content type, also implement the HTMLDocument interface, whereas XML and SVG documents implement the XMLDocument interface.
```

마크다운의 형식 파일들을 내가 원하는 형태로 받아오기 위해 lib 폴더 아래 Posts.ts 파일에 다음과 같이 만들어 주었다.  

`lib/posts.ts`

```javascript
import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';
const postsDirectory = path.join(process.cwd(), 'posts');

export function getAllPostIds() {
  const fileNames = fs.readdirSync(postsDirectory);

  // Returns an array that looks like this:
  // [
  //   {
  //     params: {
  //       id: 'ssg-ssr'
  //     }
  //   },
  //   {
  //     params: {
  //       id: 'pre-rendering'
  //     }
  //   }
  // ]
  return fileNames.map((fileName) => {
    return {
      params: {
        id: fileName.replace(/\.md$/, ''),
      },
    };
  });
}

export function getPostData(id: string) {
  const fullPath = path.join(postsDirectory, `${id}.md`);
  const fileContents = fs.readFileSync(fullPath, 'utf8');

  const matterResult = matter(fileContents);

  return {
    id,
    ...matterResult.data,
  };
}
```  

`getAllPostsIds 함수` : posts 폴더 아래의 파일들을 탐색하여 모든 파일들을 replace 함수로 확장자를 제거한 이후에 파일 이름(name)만 뽑아서 주석이 된 형태와 같이 리턴

`getPostData 함수`: id(파를  이름)를 파라미터로 받아 해당 파일의 메타데이터 정보들을 gray-matter 모듈을 사용하여 데이터 파싱

---


이제 동적 라우팅을 처리하기 위해 /posts/[id].tsx 처럼 최상위 루트 경로에서 posts 폴더 아래 [id].tsx파일 생성

Post 컴포넌트는 위에서 만든 함수들을 사용하여 해당 id(파일이름)과 일치하는 데이터를 가져와 뿌려준다.

title에는 Next.js 에서 제공되어지는 SEO 기능을 위해 Head 메타 태그를 사용하는것도 기억하자.  

```javascript
import Layout from '../../components/layout';
import { getAllPostIds, getPostData } from '../../lib/posts';

export async function getStaticPaths() {
  const paths = getAllPostIds();
  return {
    paths,
    fallback: false,
  };
}

export async function getStaticProps({ params }: {params: { id: string }}) {
  const postData = await getPostData(params.id);
  return {
    props: {
      postData,
    },
  };
}

export default function Post({ postData }: {postData: PostProps}) {
  return (
    <Layout>
      <Head>
      {postData.title}
      </Head>
      <br />
      <h1 className={utilStyles.headingXl}>
        {postData.id}
      </h1>
      <br />
      <div className={utilStyles.lightText}>
        <Date dateString={postData.date} />
      </div>
      <br />
      <div dangerouslySetInnerHTML={{ __html: postData.contentHtml }} />
    </Layout>
  );
}
```  

---

<img width="679" alt="markdown not content" src="https://user-images.githubusercontent.com/45479309/196217781-bf47106e-f830-4aac-a6f9-84b3fc96a907.png">

그런데 지금까지 작업한 내용에서는 메타태그 내용만 가져오지, 마크다운 안에 있는 내용을 가져오지 못하고있다.  

따라서 마크다운 파일 내용도 화면에 뿌려주기 위해 위 2개의 모듈을 다운받아보자.  

```
yarn add remark remark-html
```

```javascript
import { remark } from 'remark';
import html from 'remark-html';

....


export async function getPostData(id: string) {
  const fullPath = path.join(postsDirectory, `${id}.md`);
  const fileContents = fs.readFileSync(fullPath, 'utf8');

  const matterResult = matter(fileContents);

  const processedContent = await remark()
    .use(html)
    .process(matterResult.content);
  const contentHtml = processedContent.toString();

  return {
    id,
    contentHtml,
    ...matterResult.data,
  };
}
```

이제 설치한 마크다운의 데이터를 파싱해주는 부분에서 remark 모듈을 사용하여 마크다운 안에 있는 내용물을 뽑아보도록 하자. remark 모듈은 비동기로 요청해야하기 때문에 getPostData 함수를 async-await 형태로 만들어주어야 한다.  


```javascript
export async function getStaticProps({ params }: {params: { id: string }}) {
  const postData = await getPostData(params.id);
  return {
    props: {
      postData,
    },
  };
}

export default function Post({ postData }: {postData: PostProps}) {
  return (
    <Layout>
      <Head>
      {postData.title}
      </Head>
      <br />
      <h1 className={utilStyles.headingXl}>
        {postData.id}
      </h1>
      <br />
      <div className={utilStyles.lightText}>
        <Date dateString={postData.date} />
      </div>
      <br />
      <div dangerouslySetInnerHTML={{ __html: postData.contentHtml }} />
    </Layout>
  );
}
```  

getStaticProps 함수도 비동기처리를 위해 async-await로 마크다운 데이터를 받아오는 방식으로 수정해줘야 한다.  

Post 컴포넌트에선 마크다운 내용 contentHtml을 뿌려주기 위해 dangerouslySetInnerHTML 속성을 사용하여 렌더링 시켜주자.  

<img width="576" alt="markdown content" src="https://user-images.githubusercontent.com/45479309/196217728-3df6d8cd-8fc5-4c6b-9330-e2a9d131dd6a.png">


>> http://localhost:3000/posts/javascript 접속시 markdown 내용도 이제 보인다!  

출처 : https://nextjs.org/learn/basics/dynamic-routes/setup
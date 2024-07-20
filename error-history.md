# Platform-starter-kit 배포 후 문제 발생 기록

## 2024.07.19

### blog post 500 error 발생
1. localhost에서 blog site 만들고 posts 생성하면 페이지 잘 렌더링 됨
2. vercel에 업데이트 후 posts 생성하면 페이지 렌더링 안됨


``` 
Page changed from static to dynamic at runtime /umjik-blog.umjik.app/test, reason: headers
see more here https://nextjs.org/docs/messages/app-static-to-dynamic-error
```

#### 원인 파악

    edge runtime때문일까? 
    npm run dev 대신 npm run build && npm run start로 실행 후 결과 확인
    동일하게 문제 발생


#### 해결
nextjs에서는 빌드 시에 정적 페이지를 생성하고, 런타임에 동적 페이지로 전환하는 것을 허용하지 않는다. [Docs](https://nextjs.org/docs/messages/app-static-to-dynamic-error)

build 시에 dynamic 렌더링 하도록 설정해야 한다.
```
// [doamin]/posts/[slug]/page.tsx
export const dynamic = 'force-dynamic';
```

## 2024.07.20

### platform을통하여 vercel blob에 업로드된 이미지 요청시 400 error 밣생


```
http status code : 400

INVALID_IMAGE_OPTIMIZE_REQUEST
```
[Docs 에러 링크](https://vercel.com/docs/errors/INVALID_IMAGE_OPTIMIZE_REQUEST)

#### Docs 예상 원인 3가지

    invalid query parameter
    오타
    Review request format

#### 원인 파악
    
* 에러 링크에서 확인한 내용과 다르게 위의 세 가지 원이이 아니었다.

* 해결

next.config.js에서 images/remotePattern 설정을 하지 않아서


### site와 post edit시 자동 업데이트 지속적으로 호출되는 문제

#### 원인 파악

Components/Nav.tsx에서 'useSelectedLayoutSegments'를 호출하고 있었는데, 이 함수가 렌더링 될 때마다 호출되어서 'segments'가 변경이 될때마다 useEffect가 호출되어 post정보 호출한다.


#### 해결
```
// Components/Nav.tsx
...

  const segments = useSelectedLayoutSegments();
  ...

  useEffect(() => {
    // this code make infinite get request to the server
    // useSelectedLayoutSegments called in every render
    if (segments[0] === "post" && id) {
      getSiteFromPostId(id).then((id) => {
        setSiteId(id);
      });
    }
  -- }, [segments, id]);
  ++ }, [segments[0],id]); // 우선적으로 segments[0]만 체크하도록 수정

```
    
#### 추가 확인 필요사항

'useSelectedLayoutSegments' 가 지속적으로 호출되는 이유 확인 필요

-----
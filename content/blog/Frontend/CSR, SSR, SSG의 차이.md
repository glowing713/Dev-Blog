---
title: 'CSR, SSR, SSG의 차이'
date: 2022-06-30
category: 'Frontend'
draft: false
---

> 흔히 말하는 CSR(Client-Side-Rendering)과 SSR(Server-Side-Rendering) 방식의 차이는 분명 알고있었지만, SSG(Static-Site-Generation)라는 키워드도 적잖게 보인다.
> CSR과 SSR, SSG의 차이를 먼저 명확히 이해하고 넘어가려고 한다.

### Client-Side-Rendering

맨아래에 보면 참조한 문서와 영상들이 있는데, 모두 CSR과 SSR의 차이(장단점)를 정말 잘 설명해주셨기 때문에 어느정도 감이 올 것 같다.
내가 개인적으로 이해한 가장 큰 차이점은 CSR 방식은 **브라우저가 요청 결과로 빈 HTML을 받는다**는 것이다.
그리고 HTML을 파싱하는 과정에서 link, script 같은 태그를 만나면 서버로부터 JS 코드들을 받아오게 되고, 브라우저가 웹 앱을 실행하게 된다.
서버로부터 받는 결과는 빈 HTML이기 때문에, JS를 해석하지 못하는 검색 엔진 봇들은 이런 애플리케이션은 빈 문서로 인식하게 되고 그래서 SEO를 다루기가 어렵다고 한다. 물론 이 부분을 해결하기 위해 페이지를 미리 빌드해두고 크롤러에게 전달한다거나, react-helmet 같은 라이브러리를 사용할 수도 있다.
`사용자 네트워크 환경이 좋지 않다면 빈 HTML(또는, 로딩 화면)만 한참 봐야할 수도 있다.`

### Server-Side-Rendering

SSR 방식을 적용한다면 **브라우저는 요청 결과로 렌더링만 하면 되는 완성된 HTML을 받게 된다.**
Multi-Page-Application, 줄여서 MPA라고 할 수도 있겠다. 브라우저가 어떤 페이지를 요청한다면 서버에서는 완성된 형태의 문서를 전달한다.
브라우저는 받은 문서를 렌더링하여 사용자에게 제공하게 되고, 사용자는 JS가 완전히 실행되지 않았더라도 컨텐츠를 볼 수 있게 된다.

취업 준비를 하면서 CSR과 SSR의 차이를 이론으로 많이 공부했었고, SSR을 공부하면서 학부 시절에 접했던 JSP 생각이 많이났다. ~~(최강의 IDE Eclipse에다가 WAS가 필요해서 톰캣 세팅하고 맥에서 지원안하는 OracleDB 쓴다고 도커 이미지 띄우고.. 구구절절했던 기억)~~
JSP에 대한 기억을 더듬어본다면 WAS로 사용했던 톰캣 내부에 Java 엔진이 있었고, JSP 코드(DB에서 데이터 불러오고 Java로 처리하고 HTML에 포함시키는 등의 코드)를 해석하여 완성된 문서를 전달해주었다. JSP가 아니더라도 node나 php 등을 사용해도 동일한 작업을 할 수 있을 것이다.

요청마다 새로운 페이지를 전달받아 새롭게 그리기 때문에 화면이 깜빡이는 "아름다운" 전환 효과를 느낄 수 있다. ~~(윈도우 쓰면 IE 환경에서 페이지 이동할 때 그 특유의 "딸깍"하는 클릭 효과음도 있었슴,,)~~
`사용자 입장에서 컨텐츠는 보이는데 아무런 인터렉션이 없을 수도 있다.`

### Static-Site-Generation

말 그대로 정적인 웹 페이지를 미리 생성하는 것이다. SSR이랑 무슨 차이냐고 물을 수도 있겠다.
SSR 같은 경우 사용자의 요청이 있을 때, 내부 로직을 처리하여 생성한 문서를 전달한다.
SSG의 경우 미리 다 만들어두고 요청이 있을 때 그냥 전달하기만 한다.
예를 들어 컨텐츠가 동적으로 바뀔 일이 없는 블로그 컨텐츠, 자기소개 페이지 등은 SSG 방식으로 Static하게 미리 만들어두고 전달해주기만 하면 훨씬 성능이 좋은 서비스가 될 것이다.

### 결론

Next.js의 경우 CSR 방식인 react를 유지하면서 SSR 방식으로 컨텐츠를 제공할 수 있게 하는 프레임워크이다. 거기에 컴포넌트마다 SSR/SSG를 선택적으로 적용할 수도 있다.(CSR 방식으로 데이터 fetching도 가능)
서버는 그대로 데이터만 전달해주므로써 부하를 줄이면서, 클라이언트는 FE 서버를 두고 SSR 방식으로 컨텐츠를 생성해서 사용자에게 제공한다니.
앞으로 "우리 서비스는 CSR/SSR/SSG 방식이야" 라고 분리해서 생각하기보다 각 방식의 특징을 이해하고 필요에 따라 선택하여 적용하는 것이 중요하겠다는 생각이 든다. 간단한 블로그를 만들더라도 일반 정적 컨텐츠들은 Static하게 만들어두고 요청에 따라 전해주고, 방명록과 같이 동적인 컨텐츠가 있다면 그건 SSR 방식으로 만들어 전달할 수 있다.
물론 아직 사용해보지는 않았지만 하나의 프레임워크 안에서 이런 선택지들을 유연하게 제공하는 Next.js가 상당히 괜찮은 도구일 수도 있겠다는 생각이 들었다.(+ node도 같이 공부하게 되겠지?)

### Reference

- [[10분 테코톡] 🎨 신세한탄의 CSR&SSR](https://youtu.be/YuqB8D6eCKE)
- [SSR과 CSR 이 영상 하나로 끝내기! (SEO 해결 포함)](https://youtu.be/D71ByEIBWEs)
- [토스ㅣSLASH 22 - 잃어버린 유저의 시간을 찾아서 : 100년을 아껴준 SSR 이야기](https://youtu.be/IKyA8BKxpXc)
- [[FE] SSR(Server-Side-Rendering) 그리고 SSG(Static-Site-Generation) (feat. NEXT를 중심으로)](https://velog.io/@longroadhome/FE-SSRServer-Side-Rendering-%EA%B7%B8%EB%A6%AC%EA%B3%A0-SSGStatic-Site-Generation-feat.-NEXT%EB%A5%BC-%EC%A4%91%EC%8B%AC%EC%9C%BC%EB%A1%9C)
- [The Benefits of Server Side Rendering Over Client Side Rendering](https://medium.com/walmartglobaltech/the-benefits-of-server-side-rendering-over-client-side-rendering-5d07ff2cefe8)
- [어서 와, SSR은 처음이지? - 도입 편](https://d2.naver.com/helloworld/7804182)
- [어서 와, SSR은 처음이지? - 개발 편](https://d2.naver.com/helloworld/2177909)

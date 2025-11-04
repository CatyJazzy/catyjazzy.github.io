---
title: CSS: 자식요소의 여백이 왜 부모 요소에 적용되나요?
date: 2024-04-15 10:00:00 +0900
categories: [Frontend]
tags: [HTML/CSS]
---

## 🍊 계기

개인 프로젝트로 두더지 게임을 만들던 중, 아래와 같이 자식 요소의 적용한 여백이 wrapper인 부모 요소의 여백으로 적용되었었다.

해당 현상은 생각보다 자주 겪었던 것 같은데 그때그때 해결만 했을 뿐 정리를 하지 않아서 탐구해보기로 함!

![image.png](CSS%20%EC%9E%90%EC%8B%9D%EC%9A%94%EC%86%8C%EC%9D%98%20%EC%97%AC%EB%B0%B1%EC%9D%B4%20%EC%99%9C%20%EB%B6%80%EB%AA%A8%20%EC%9A%94%EC%86%8C%EC%97%90%20%EC%A0%81%EC%9A%A9%EB%90%98%EB%82%98%EC%9A%94/image.png)

## 🍊 도대체 이 현상은 무엇인가?

부모 태그에 별도의 border, padding, margin이 없고 + max-height, min-height가 설정되어 있지 않을 경우 자식 태그의 margin-top이 부모에게도 적용된다고 한다.

→ 이는 **margin-collapse 현상**의 일종이다. [(출처)](https://velog.io/@heylub/wecode%ED%9A%8C%EA%B3%A0-CSS-westagram-main-layout-%EA%B5%AC%EC%84%B1-%EA%B3%BC%EC%A0%95-%EC%A4%91-%EB%B6%80%EB%AA%A8-%EC%9E%90%EC%8B%9D%EC%9A%94%EC%86%8C-%EA%B4%80%EA%B3%84)

그렇다면 다음과 같은 토픽을 살펴보자.

- 플젝이 아니라 간단한 예제로 다시 재현했을 때도 같은 현상이 나타나는지?
- 해결 방법들에는 어떠한 것들이 있는지?
- margin-collapse란 무엇인지?

### 간단한 예제로 재현했을 때 같은 현상이 나타난다.

```html
<div id="wrapper">
  <div id="top"></div>
</div>
```

```css
body {
  margin: 0;
  padding: 0;
}
#wrapper {
  width: 200px;
  height: 300px;
  background: green;
  margin: 0 auto;
}
#top {
  background: orange;
  height: 100px;
/*   margin-top: 20px; */
}
```

위의 코드에서 margin-top: 20px;을 적용시키면 위의 사진이 아래의의 사진처럼 여백이 생긴다. 즉, *자식 요소인 top에 준 윗여백이 부모 요소인 wrapper에도 적용되었다.*

![](https://blog.kakaocdn.net/dna/cbMiX5/btsGCWoYKwr/AAAAAAAAAAAAAAAAAAAAAJbmnMMlpLg5BllTccUwwVJjj90VlcVyM34BnVT9teSA/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=%2Fx%2BwFqx9E5Scw415AaCx6tlnVsI%3D)

![](https://blog.kakaocdn.net/dna/dNTvft/btsGBitgEG5/AAAAAAAAAAAAAAAAAAAAADb-ro1pqgFfqyGW7x1OPrvAl-kC7MVIoGq5jSOY3wg3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=dqpXpgy5vH41d1wEHhvbigqPe8E%3D)

## 🍊 해결방법으로는 어떠한 것들이 있나?

일단 이러한 현상은 부모의 별도의 border, padding, margin이 없을 때 일어난다고 있기 때문에 각각의 값을 설정해보자.

### [1] 부모 요소에 border 설정해보기 ✅

```css
body {
  margin: 0;
  padding: 0;
}

#wrapper {
  width: 200px;
  height: 300px;
  background: green;
  margin: 0 auto;
  border: 3px solid red;
}
#top {
  background: orange;
  height: 100px;
  margin-top: 20px;
}
```

![](https://blog.kakaocdn.net/dna/bRah5K/btsGBxDHK6Y/AAAAAAAAAAAAAAAAAAAAAJufTTbTi1nRMkrsoCqgz2V4ay1mr_5Cw9iKXJjRyI69/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=O025PevfP0dLQ1ZMCjlr1oC4qf8%3D)

부모 요소에 border를 지정했더니, 해당 현상이 일어나지 않는다.

### [2] 부모 요소에 padding 설정해보기 ✅

```css
body {
  margin: 0;
  padding: 0;
}

#wrapper {
  width: 200px;
  height: 300px;
  background: green;
  margin: 0 auto;
  padding: 10px;
}
#top {
  background: orange;
  height: 100px;
  margin-top: 20px;
}
```

![](https://blog.kakaocdn.net/dna/bhvMqh/btsGDCcLb8p/AAAAAAAAAAAAAAAAAAAAAE0oFqynCNgJbfnX3C16dCJAT4P2k-iT6iAYNjfLXaAG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=Dy8zGQXT%2Brga3xHbsCF4yfOE%2FsA%3D)

부모 요소에 padding을 지정했더니, 마찬가지로 해당 현상이 일어나지 않는다. * 물론 의도했던 자식 요소의 크기와 달라질 수는 있겠다.

### [3] 부모 요소에 margin 설정해보기 ❎

부모 요소의 margin은 이미 0 auto로 해놨기 때문에, *해결책이 아닌 것 같다.* 이미 0px로 지정이 되어 있는 상태에서 해당 현상이 나타난 것이기 때문이다. (* MDN문서를 확인해보니 margin이 0일 때도 여백상쇄는 발생한다고 한다)

### [4] 부모 요소에 max, min-height 설정해보기 ❎

**min-height: 300px;**를 적용해도 해당 현상은 해결되지 않았다. gemini에게 물어보니, min-height는 부모 요소 내 컨텐츠 크기에 따라 의도치 않게 레이아웃을 변형시킬 수 있고 flexbox나 grid레이아웃 시스템과 충돌할 수 있어 추천하지 않는다고 한다.

- 콘텐츠 크기보다 min-height값이 크면, 의도치 않게 콘텐츠를 안에 담는 형태로 강제될 수도 있기 때문이다.

### [5] 자식 요소에 position 지정하기 ✅

생각해보니, 자식 요소가 일반 흐름에서 벗어나면 해결될 것 같아 부모 요소의 position속성을 relative, 자식 요소에 absolute로 지정하였다. 이런 방식으로도 *해당 현상을 해결할 수 있다!*

```css
body {
  margin: 0;
  padding: 0;
}

#wrapper {
  width: 200px;
  height: 300px;
  background: green;
  margin: 0px auto;
  position: relative;
}
#top {
  background: orange;
  width: 100%;
  height: 100px;
  margin-top: 20px;
  position: absolute;
}
```

![](https://blog.kakaocdn.net/dna/0ZqLC/btsGBZz3gRq/AAAAAAAAAAAAAAAAAAAAAIKc6wu2w4gxTq66lPaHAsNo86P_GPJeafV0tYrok4-W/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=y2DYee77aJtCCZwlT6DzkciarVI%3D)

> (😬급깨달음: 해당 현상은 해결되었지만, 일반 흐름에서 벗어난다는 것을 반드시 고려하여 활용해야 한다. 나의 두더지 프로젝트 같은 경우, 차라리 wrapper를 기준으로 모든 자식을 position으로 지정하여 하는 것이 나았을 것 같다. 일반 흐름을 기준으로 쌓이게 하다보니 콘텐츠 크기를 매번 일일이 신경쓰며 해서 오히려 레이아웃이 이상해졌다.)
> 

## 🍊 margin-collapse 여백병합현상은 무엇인가?

이는 ***여러 블록의 위쪽, 아래쪽 여백은 경우에 따라 가장 큰 크기의 단일 여백으로 결합되는 현상***을 말한다. (플로팅 요소나 ‘절대 위치’를 지정한 요소의 여백은 결합되지 않는다)

이러한 현상이 발생하는 상황을 크게 **3가지**로 나눌 수 있다.

(1) 인접 형제 간의 바깥 여백끼리 겹칠 때

(2) 부모와 자손을 분리하는 콘텐츠가 없을 때

(3) 테두리, 안쪽 여백, 인라인 컨텐츠, height, min-height, max-height가 없을 때

가령, 부모와 자손을 구분하는 컨텐츠가 있다면, 아래와 같이 자식의 margin-top(20px)이 의도한 대로 적용된다.

![](https://blog.kakaocdn.net/dna/pdPUg/btsGFmmIwDW/AAAAAAAAAAAAAAAAAAAAAPceU1lBTe9HzHppoirlalQBvblpMTfjTJoxk5EBe3z4/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=wQPzxVZh4QqV3reRj1YkuqKVlto%3D)

MDN문서에서 제공한 **예제**는 다음과 같다.

```html
<p>The bottom margin of this paragraph is collapsed …</p>
<p>
  … with the top margin of this paragraph, yielding a margin of
  <code>1.2rem</code> in between.
</p>

<div>
  This parent element contains two paragraphs!
  <p>
    This paragraph has a <code>.4rem</code> margin between it and the text
    above.
  </p>
  <p>
    My bottom margin collapses with my parent, yielding a bottom margin of
    <code>2rem</code>.
  </p>
</div>

<p>I am <code>2rem</code> below the element above.</p>
```

```css
div {
  margin: 2rem 0;
  background: lavender;
}

p {
  margin: 0.4rem 0 1.2rem 0;
  background: yellow;
}
```

위의 코드는 아래의 화면처럼 나타난다.

![](https://blog.kakaocdn.net/dna/E1glo/btsGBgPMfeK/AAAAAAAAAAAAAAAAAAAAACqDgA8E2BtS-GXLsRBkH0fLGDefhdoQMonUJEfIVDN3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=3xLZE6D36KXao3X1U5s8ZScIDs8%3D)

여백상쇄 현상에 따라 최종결정된 여백 rem들은 다음과 같이 정리해볼 수 있다.

![](https://blog.kakaocdn.net/dna/caB61e/btsGB4878Ks/AAAAAAAAAAAAAAAAAAAAALir9u10-45VLaQuKZP5OLEhYNaTbzAA6A149kcqCsSf/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=1H6L2sUqX7fCVbq1Hf5Tt%2FVjA7g%3D)
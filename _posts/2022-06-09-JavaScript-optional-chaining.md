---
title: JavaScript: 옵셔널 체이닝 ?.
date: 2022-06-09 10:00:00 +0900
categories: [Frontend]
tags: [JavaScript]
---

노션에서 개인적으로만 정리하던 **자바스크립트 토픽들🧡**을 블로그에도 함께 올리려고 한다.

일단 이미 해둔 것들은 조금씩 옮겨놓고, 나중에는 복습하면서 블로그 포스팅으로도 같이 올려야겠다.

강의 듣거나, 개인 프로젝트를 하면서 궁금해진 것들을 코어자바스크립트 문서([https://ko.javascript.info/js](https://ko.javascript.info/js))와 모던자바스크립트 딥다이브책을 참고하며 해결하고 있다.

다른 분들께서 틀린 내용을 지적해주실 수도 있고!

포스팅하다보면 좀더 책임감을 갖고 정확하게 공부하며 정리하려는 태도를 갖출 것 같다!

결론은. 여러 측면에서 블로그 포스팅하면 좋을듯.

![image.png](/assets/img/posts/Javascript-hat.png)

---

## 옵셔널 체이닝 ?.

### 🍊 계기

새로운 문법 소개하는 유튜브 쇼츠에서 봄

### 🍊 노트

옵셔널 체이닝은 “안전하게” 프로퍼티 또는 메서드에 접근할 수 있는 새로운 방식이다. “안전하게”라고 한다면, 의도치 않게 빈값이 들어올 때 에러 발생을 막고 값이 제대로 들어왔는지부터 판단할 수 있다는 것이다.

### 배경)

user.address.street과 같은 방식으로 접근하고자 할 때, 옵셔널 체이닝 등장 전에는 && 연산자를 사용하곤 했다. 앞의 값이 존재할 때만 프로퍼티에 접근하도록 하는 것이나. 그러나 해당 방식은 코드도 길어지고 직관적이지 않다.

### syntax)

```jsx
(평가대상)?.(프로퍼티)
```

평가대상이 undefined 또는 null이면, 평가를 멈추고 undefined를 반환한다.

user.address.street의 경우, user.address가 없다면 undefined의 프로퍼티에 접근할 수 없다는 에러가 날 것이다.

그러나, user.address?.street의 경우, user.address가 있는지 평가하고 값이 있다면 street에 접근하고, 없다면 평가를 멈추고 undefined를 반환한다.

### 활용)

```jsx
user.manager?.admin()
```

user.manager값이 있는지 평가하여 상황에 따라 admin함수를 실행하도록 작성

```jsx
user.manager.admin?.()
```

user.mamager에 admin메서드가 있는지 평가하여 정의되어 있다면 호출하고, 없다면 undefined를 반환한다.

위의 두 케이스는 차이가 있다. 전자는 **특정 값이 있는지 확인**하고 → 있다면 그 값에 있는 메서드를 호출하는 것이고,

후자는 **메서드가 있는지 확인**하고 → 있다면 호출하는 것이다.

```jsx
user?.['address']
```

이렇게도 표현할 수 있다.

```jsx
user?.address?.street
```

중첩하여 작성할 수 있다.

### 유의사항)

남발하지 않기 !!!

옵셔널 체이닝을 남발하면 실제로 에러를 통해서 확인해야 할 어떤 값을 제대로 디버깅하지 못할 수 있다. 가령 위의 예제에서 user는 보통의 케이스라면 비어있으면 안되는 값이므로 옵셔널 체이닝을 활용하기보다는 에러를 통해 특정 조치를 취하도록 하는 것이 낫다.

### 🍊 Reference

[https://ko.javascript.info/optional-chaining](https://ko.javascript.info/optional-chaining)
---
title: 코어자바스크립트 - 데이터타입
date: 2025-08-31 10:00:00 +0900
categories: [Frontend]
tags: [JavaScript]
---

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" />

아래의 내용은, 정재남 강사님의 [인프런 “코어자바스크립트”](https://www.inflearn.com/course/%ED%95%B5%EC%8B%AC%EA%B0%9C%EB%85%90-javascript-flow/dashboard)를 기반으로 학습정리한 내용입니다. 개인적으로 추가 조사/공부한 자료와 섞여있습니다. 

</aside>

## 데이터타입의 종류와 차이

자바스크립트의 자료형은 2가지 타입, (1) Primitive (원시형) (2) Reference (참조형)으로 나눌 수 있다. 

- Primitive Type
    - Number
    - String
    - Boolean
    - null
    - undefined
    - (+) Symbol
- Reference Type
    - Object
        - Array
        - Function
        - RegExp
        - (+) Set / WeakSet
        - (+) Map / WeakMap

둘의 핵심적인 차이는 **“메모리가 저장되는 방식”**에 있다. 

자바스크립트의 메모리 영역을 단순화하여 생각하면, 2가지 영역으로 생각할 수 있다.

(1) **Stack Memory**: 변수. 기본형 데이터. 정적 할당

(2) **Heap Memory**: 참조형 데이터. 동적 할당

조금 더 자세히 이해하기 위해, 아래의 표현을 참고하자.

> **스택**은 *함수*를 실행하는 동안에만 필요한 데이터를 저장하는 데 사용됩니다. 빠르고 *효율적*이지만 *용량이 제한*되어 있습니다. 함수가 호출되면 *자바스크립트 엔진은 함수의 변수와 매개변수를 스택으로 밀어넣고, 함수가 반환되면 다시 스택에서 꺼냅니다.* 스택은 빠른 액세스와 빠른 메모리 관리를 위해 사용됩니다.
> 

> 반면 **힙**은 *애플리케이션의 전체 수명* 동안 필요한 데이터를 저장하는 데 사용됩니다. 스택보다 *조금 느리고 덜 체계적*이지만 용량이 훨씬 큽니다. 힙은 여러 번 액세스해야 하는 *객체*, *배열* 및 기타 *복잡한 데이터 구조*를 저장하는 데 사용됩니다.
> 

*(출처: [https://ykss.netlify.app/translation/javascript_memory_management/](https://ykss.netlify.app/translation/javascript_memory_management/))*

**정적 할당 / 동적 할당**의 차이는 각 변수에게 할당해줘야 하는 메모리의 크기를 알게 되는 ‘시점’이다. 배열과 같이 특정 요소가 추가되거나 삭제될 수 있는 자료형은 `런타임`에 알 수 있지만, 그 외에 기본형 타입은 `컴파일 타임`에 미리 알 수 있다.

이외에도 스택은 “빠른 액세스”를 할 수 있다고 되어있는데 이는 데이터 타입 2가지에 따른 접근 방식의 차이가 있기 때문이다. 해당 부분은 개별 목차로 살펴보겠다.

## 저장 및 접근 방식의 차이?

### (1) 기본형

기본형(원시형) 데이터부터 확인해보자. 간단하다. 

일단, 기본형이든 참조형이든 선언과 할당이 따로 이루어진다. `let a = 1` 처럼, 코드에서 동시에 있다고 하더라도 (1) a에 대한 공간을 먼저 확보하고 (2) 이후 1이라는 값이 할당된다.

이제 메모리에서의 동작을 보자. 아래와 같은 코드가 있을 때, 

```jsx
let msg = "Hello";
```

(1) msg를 위한 메모리 공간이 할당되고, 변수명에 해당하는 이름을 갖는다. 

(2) 특정 공간(@5051)에 “Hello” 값을 저장한다.

(3) msg에 @5051에 저장된 “Hello” 값을 할당한다.

![image.png](/assets/img/posts/js-dataType-1.png)

만약 값이 바뀐다면 어떻게 될까?

```jsx
let msg = "Hello";
msg = "Bye";
```

@5051에 있던 “Hello”값을 바꾸는 것이 아니라, 아예 새로운 공간(@5052)에 필요한 값을 저장하여 주소를 바꾼다. 

![image.png](/assets/img/posts/js-dataType-2.png)

그럼 자연스러운 걱정이 생긴다 … 🤔 저렇게 immutable하게 매번 새롭게 값을 만든다면, 기존값으로 저장되었던 공간들이 **낭비되는 것 아닌가?** 

우리 모두가 알고 있는 **가비지컬렉터(Garbage Collector)**가 이를 해결해준다. 많은 최적화 기법이 있지만, 간단하게는 참조카운팅을 통해 해결한다.

아래의 코드에서 “Hello”를 참조하고 있는 변수의 개수를 주석으로 살펴보자.마지막줄처럼 0개가 된 시점에서, “Hello”는 가비지 컬렉션의 대상이 된다.

```jsx
let msg = "Hello"; // (msg) 1개
let hi = msg; // (msg, hi) 2개
msg = "Bye"; // (hi) 1개
hi = "Hi"; // () 0개
```

### (2) 참조형

하이라이트는 참조형이다. 중첩 객체처럼, depth가 한없이 깊어질 수 있는 이 복잡한 자료형을 어떻게 저장할까?

아래와 같은 코드가 있다. 객체는 프로퍼티(property)로 이루어지므로, 크게 2개의 다리를 거친다. 기본형처럼 바로 변수에 해당하는 값을 바로 참조하는 게 아니라, (1) 프로퍼티의 범위를 먼저 찾고 → (2) 해당 범위에서 각 프로퍼티에 해당하는 실제값들에 접근한다. 

(1) age, name 등을 아우르는 5050~(?) 범위를 담은 (@3001)을 값으로 갖고

(2) 3001에 저장된 @5050~(?)의 범위에서 특정 프로퍼티의 값을 찾는다 
ex. (@5050 → @5053 = ‘jihoo’)

```jsx
let user = {
	name: "jihoo",
	age: 26
}
```

![image.png](/assets/img/posts/js-object-1.png)

**특정 프로퍼티의 값들이 기본형이 아닌, 또다른 객체라면 어떻게 될까?** 같은 원리다. 특정 프로퍼티에 또 간접 참조(그림에서 화살표 2개로 값에 접근하는 방식)가 생기는 것이다.  

```jsx
let user = {
	name: "jihoo",
	car: ["S90", "GV80"]
}
```

car 프로퍼티의 값이 배열이다. 배열도 참조형 데이터이므로, 아래의 사진처럼 5051 공간에서 3002를 가리키고, 3002가 실제 값을 저장할 공간들의 범위를 갖게 된다. 

![image.png](/assets/img/posts/js-object-2.png)

자! 이제 하이라이트다. 참조형 데이터 타입을 잘 이해해야 하는 이유를 말해주는 상황이기도 하다. **만약 위의 user 객체를 그대로, ‘guest’라는 변수에 담았다고 해보자.**

```jsx
let user = {
	name: "jihoo",
	car: ["S90", "GV80"]
}

let guest = user;
guest.car = "QM3";
**console.log(user.car); // ??**
```

그러면 메모리에서는 아래와 같이 복사된다. user가 참조하기 위해 가지고 있던, 주소 @3001이 guest가 참조하는 값으로 복사된다. 

![image.png](/assets/img/posts/js-object-3.png)

```jsx
guest.car = "QM3";
```

그 다음 guest.car에 접근하여 값을 “QM3”로 바꾸었다. 즉, @5051에서 3002를 더이상 참조하지 않고 “QM3”라는 새로운 값을 할당한, @5052를 바라보게 된다.

![image.png](/assets/img/posts/js-object-4.png)

핵심은, 이것이 user에게도 그대로 적용된다는 점이다. 어차피 user, guest 모두 3001을 바라보고 있으므로! guest에서 3001이 참조하던 5051의 값을 바꾸었다면, 그건 user가 참조하던 5051의 값을 바꾼 것이나 마찬가지다. 

따라서, user.car도 ["S90", "GV80"]으로 유지되지 않고, “QM3”로 변경된다. 객체(참조형 데이터)의 이러한 특성 때문에, 객체의 값을 변경하거나 복사할 때 의도한 대로 동작하게 만들기 위해서는 새로운 방법을 사용해야 한다. 

```jsx
let user = {
	name: "jihoo",
	car: ["S90", "GV80"]
}

let guest = user;
guest.car = "QM3";
console.log(user.car); // "QM3"
```

참조하는 ‘주소’를 공유하지 않도록, 완전한 복제본을 할당해야 한다. 이때 중첩된 구조까지 고려하여 복사하는 ‘깊은 복사’와, 그렇지 않은 ‘얕은 복사’가 있다. 

우리가 자주 사용하는 [Object.assign(target, ...sources)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)은 ‘얕은 복사’에 해당한다. 웹 API인 [structuredClone(value)](https://developer.mozilla.org/ko/docs/Web/API/Window/structuredClone)은 ‘깊은 복사’에 해당한다. 또다른 익숙한 방식인 JSON.parse(JSON.stringify())은 ‘깊은 복사’를 해주지만, 객체 관계 내의 순환참조가 있을 때 오류가 발생하며, 함수 프로퍼티는 누락된다는 한계가 있다.

## Reference

- https://ko.javascript.info/object-copy
- https://ykss.netlify.app/translation/javascript_memory_management/
- https://developer.mozilla.org/ko/docs/Glossary/Property/JavaScript
- https://www.daleseo.com/js-objects-clone/
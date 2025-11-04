---
title: JavaScript setTimeout & setInterval
date: 2024-06-05 10:00:00 +0900
categories: [Frontend]
tags: [JavaScript]
---

이전에 정리해놨던 내용인데, 모던 자스 딥다이브 책을 읽다가 마주치니 갑자기 또 헷갈린다... 복습 차원에서 공유..

역시 한 번에 내용들을 흡수한다는 허상을 버리고 한계를 인정해야겠다. 그래야 좋은 학습태도도 가질 수 있을 듯하다.

![](https://blog.kakaocdn.net/dna/bpKux0/btsHPbLWKeO/AAAAAAAAAAAAAAAAAAAAAD7QfFDVBfGAtumcDtjVn3e0h1oMU9v8IXTjlicFfG6z/img.jpg?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=5W0eDjiG34lpALSl%2F2RxUMu9q2c%3D)

https://www.freecodecamp.org/news/explaining-the-best-javascript-meme-i-have-ever-seen/ (출처)

---

## setTimeout과 setInterval

### 🍊 계기

제로초 자스 강의와 두더지 게임 프로젝트에서 setTimeout을 많이 활용하였다. 그런데 미묘한 시간 차이도 있고, 납득할 수 없는 모습으로 동작할 때도 있어서 답답했다. 제대로 알지도 못하고 쓰는 것 같아서 정리하였다.

### 🍊 노트

내가 원하는 시간에 맞춰서, 특정 시간이 지난 시점에 함수 실행을 ‘예약’할 수 있는 것을 **‘호출 스케줄링’**이라고 한다. 여기서 ‘예약’을 해둔다는 것을 기억해야 이해하기가 쉽다.

두 가지 방법이 있다. **setTimeout과 setInterval**이다.

### 1) setTimeout

**let timerId = setTimeout**(함수, 지연시간, 함수인자, 함수인자, …)

→ 함수 실행 시 타이머 아이디를 반환하는데, 브라우저에서는 숫자 형식이다. 실행환경에 따라 형태는 달라질 수 있다.

→ 지연시간은 *밀리초(1000ms = 1s)*를 기준 단위로 한다.

→ 첫 번째 인자의 함수 실행 시 사용할 인자들도 함께 넘겨줄 수 있다.

- 이때 첫 번째 인자로 문자열을 넘겨줄 수 있다. 함수로 평가될 수 있는 값이면 해당 문자열이 정상 실행된다. 추천하진 않음. ex) “alert(’예시’)”

**clearTimeout(타이머 식별자)**

setTimeout실행 시 반환되었던 타이머 아이디(타이머 식별자)를 인자로 넘겨서 해당 스케줄링을 취소할 수 있다.

### 2) setInterval

**let timerId = serInterval**(함수, 지연주기, 함수인자, 함수인자, …)

→ setTimeout과 같은 syntax를 사용한다.

→ 다만 한 번 실행 후 끝내는 게 아니라, 설정한 지연 시간만큼 텀을 두고 주기적으로 함수를 실행하도록 스케줄링한다.

→ clearInterval(timerId)를 통해 중단할 수 있다.

### 3) 1초 간격으로 출력하기 예제

> from에 명시한 숫자부터 to에 명시한 숫자까지 출력해주는 함수 printNumbers(from, to)를 만들어보세요. 숫자는 일 초 간격으로 출력되어야 합니다.
> 

위의 예제는 **2가지의 방식**으로 접근할 수 있다.

- **setTimeout**은 함수를 한 번 실행시키지만, 중첩 구조를 통해 주기적으로 스케줄링 할 수 있다.
- **setInterval**을 통해 구현할 수 있다.

**[1] setTimeout을 활용하여 작성한 코드**

```jsx
const printNumbers = (from, to) => {
  let number = from;

  let timerId = setTimeout(function showNumber() {
    console.log(number);
    number += 1;

    if (number > to) {
      clearTimeout(timerId);
    } else {
      timerId = setTimeout(showNumber, 1000);
    }
  }, 1000);
};
```

재귀적으로 초기에 예약해두었던 함수를 실행할 때, 아직 내가 원하는 만큼 숫자를 출력하지 않았다면 또다시 해당 시점으로부터 1초 후에 스케줄링을 하는 것이다. 이러한 구조의 반복!

**[2] setInterval을 활용하여 작성한 코드**

```jsx
const printNumbers2 = (from, to) => {
  let number = from;
  let timerId = setInterval(function showNumber() {
    console.log(number);
    number += 1;

    if (number > to) {
      clearInterval(timerId);
    }
  }, 1000);
};
```

원하는 만큼 숫자를 다 출력했을 때 setInterval을 적절히 해제해주기만 하면 된다.

### 4) 지연 간격에 대하여

위의 예제에서 활용한 두 방식은 지연 간격에 있어서 차이가 발생한다.

**setInterval**

![](https://blog.kakaocdn.net/dna/8zGdS/btsHO47EGqp/AAAAAAAAAAAAAAAAAAAAAJmftLcG5IzIWJeJqcVVcmjZ4tSmMK7bE-6_z8dUopmt/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=Z4iKJqRnqyouaKDoNBVU1jTY7%2Fc%3D)

**setTimeout**

![](https://blog.kakaocdn.net/dna/GVlwH/btsHOD3Puh9/AAAAAAAAAAAAAAAAAAAAAHpNip_8saRAawIwrhMsS7QQPZQitOwunPnggLXPDVXP/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=5Z04tyPC4diJnKjfm2XsZclpuSI%3D)

**setInterval의 지연 간격에는 함수의 실행을 위해 사용되는 시간도 포함**된다. 따라서, 내가 의도한 지연 간격에서 함수 실행 시간을 제외한 만큼만 지연된다.

반면, 중첩된 **setTimeout을 사용할 경우 내가 지정한 지연 간격이 보장**된다. 그 다음 함수 실행에 대한 스케줄링이 직전 함수 실행이 종료된 후에 진행되기 때문이다.

- **지연시간보다 함수 실행에 소요되는 시간이 더 길다면?**

엔진이 함수 실행까지 기다려주었다가 종료 후 스케줄러를 확인하여, 시간이 오버되었다면 바로 다음 함수를 실행한다.

### 5) 설정한 지연 시간이 0이라면?

순차적으로 실행되고 있는 스크립트 종료 직후에 예약한 함수를 실행시킬 수 있다.

```jsx
setTimeout(() => alert("World!"));
alert("Hello");
```

“Hello” “World!”

Hello → World순으로 alert함수가 실행된다. 그 이유는 setTimeout이 하는 일은, alert(”World!”)함수를 실행시킨 것이 아니라, 예약해둔 것이다. 시간표에 적어둔 것.

alert(”Hello”)를 실행하고 나서야 스케줄러가 할 일을 체크하므로, Hello 직후 실행되는 것이다.

- **브라우저에서 실제 대기 시간이 절대적인 0은 아님. 아래의 코드로 지연시간을 출력해보면, 앞쪽에서는 바로바로 실행되는데 점점 지연 간격이 밀림.**

→ 이는 HTML5표준에서 정한 중첩 타이머 실행 간격 관련 제약을 준수하도록 되어있기 때문. "다섯 번째 중첩 타이머 이후엔 대기 시간을 최소 4밀리초 이상으로 강제해야 한다.”

```jsx
let start = Date.now();
let times = [];

setTimeout(function run() {
  times.push(Date.now() - start);// 이전 호출이 끝난 시점과 현재 호출이 시작된 시점의 시차를 기록if (start + 100 < Date.now()) alert(times);// 지연 간격이 100ms를 넘어가면, array를 얼럿창에 띄워줌else setTimeout(run);// 지연 간격이 100ms를 넘어가지 않으면 재스케줄링함
});

// 출력창 예시:// 1,1,1,1,9,15,20,24,30,35,40,45,50,55,59,64,70,75,80,85,90,95,100
```

Node.js환경에서는 또 다른 듯하다. 이렇게 적다보니 실행 환경에 대한 차이도 궁금하고 신가하당…

![](https://blog.kakaocdn.net/dna/XdPbd/btsHOWIJtj1/AAAAAAAAAAAAAAAAAAAAAJv_q71f5O5TBELRJIi5j0PS0l0ufSWwp2Tu6V5d0tfM/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=aUBMxSu5X88eTvqEB5RftM9UwcU%3D)

### Reference

코어자바스크립트(함수심화학습 - setTimeout과 setInterval을 이용한 호출 스케줄링) [https://ko.javascript.info/settimeout-setinterval](https://ko.javascript.info/settimeout-setinterval)
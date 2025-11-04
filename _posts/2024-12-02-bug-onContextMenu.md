---
title: 우클릭 시 이동모드 활성화되는 오류수정
date: 2024-12-02 10:00:00 +0900
categories: [Frontend]
tags: [React]
---

### 🚨 문제상황: ‣

(테스트해본 결과 Window에서만 의도한 대로 활성화되지 않고, Mac 환경에서만 활성화되고 있는 상태)

![image.png](%EC%9A%B0%ED%81%B4%EB%A6%AD%20%EC%8B%9C%20%EC%9D%B4%EB%8F%99%EB%AA%A8%EB%93%9C%20%ED%99%9C%EC%84%B1%ED%99%94%EB%90%98%EB%8A%94%20%EC%98%A4%EB%A5%98%EC%88%98%EC%A0%95/image.png)

### 🔎 이동활성화 로직 살펴보기

---

Node 컴포넌트에 다음과 같이 핸들러들이 적용되어있다.

```tsx
dragBoundFunc={dragBoundFunc}
**onMouseDown={(e) => move.callbacks.startHold(node, e)}**
onMouseUp={move.callbacks.endHold}
**onTouchStart={(e) => move.callbacks.startHold(node, e)}**
onTouchEnd={move.callbacks.endHold}
**onContextMenu={handleContextMenu}**
```

마우스 관련 이벤트들을 정리해보자

- **`mousedown`** 요소 위에서 마우스 왼쪽 버튼을 누를 때 발생
- **`mouseup`** 마우스 버튼 누르고 있다가 뗄 때 발생
- **`click`**  마우스 왼쪽 버튼을 사용해 동일한 요소 위에서 `mousedown` 이벤트와 `mouseup` 이벤트를 연달아 발생시킬 때 실행됨
- **`contextmenu`** 마우스 오른쪽 버튼을 눌렀을 때 발생

코어자바스크립트의 학습용 블록에서는 아래와 같이 나온다. 

![image.png](%EC%9A%B0%ED%81%B4%EB%A6%AD%20%EC%8B%9C%20%EC%9D%B4%EB%8F%99%EB%AA%A8%EB%93%9C%20%ED%99%9C%EC%84%B1%ED%99%94%EB%90%98%EB%8A%94%20%EC%98%A4%EB%A5%98%EC%88%98%EC%A0%95/image%201.png)

그런데 우리팀 구현코드 핸들러 함수들에서 출력해보면

**[mousedown → contextmenu → ?]**가 끝이다. mouseup이벤트가 발생하지 않는다. Konva에서 지원하는 Mouse events에는 분명히 mouseup이 포함되어있다.

***Window 환경인 팀원에게 부탁하니, 이벤트 발생 순서가 달랐다.***

```tsx
mousedown   button=0
mouseup     button=0
click       button=0
------------------------------
**mousedown   button=2
mouseup     button=2
contextmenu button=2**
```

소름이다. 이렇게 보니 분명, Mac환경에서는 contextmenu가 먼저 발생하는 것이 버그의 원인일 것이다. ***contextmenu 이벤트가 중간에 발생해버리면 mouseup으로 종료되지 않고 이동 모드가 의도치 않게 계속 활성화된 상태로 유지되기 때문이다.***

### ⛑️ Mac에서도 활성화 안되게 고쳐보자!

---

단순하게 우클릭으로 mouse이벤트가 발생했을 때는 아예 이동모드 활성화 시작을 안하면 되지 않을까?

useMoveNode의 startHold에서 button으로 분기처리해보자. 아래와 같이 (2번 버튼 = 마우스의 오른쪽 버튼)이면 아예 함수를 리턴시키도록 변경했다.

```tsx
const startHold = (node: Node, e: KonvaInteractionEvent) => {
    **// 우클릭으로 이벤트가 발생했을 경우 이동모드 활성화 방지
    if (e.evt.button === 2) {
      return;
    }**

    setMoveState((prev) => ({
      ...prev,
      isHolding: true,
      targetNode: node,
      animationEvent: e,
    }));

    setHoldingAnimation(e, true);

    if (holdTimer.current) {
      clearTimeout(holdTimer.current);
    }
    holdTimer.current = setTimeout(() => {
      setMoveState((prev) => ({
        ...prev,
        isMoving: true,
      }));
    }, HOLD_DURATION);
  };
```

이렇게 버그픽스를 하다보니 노드에 대한 이벤트 처리 충돌이 일어날 때마다 하나하나 분기처리를 하는 것이 좋은 방식일지는 모르겠다 🤔 

당장의 상황에서는 간단한 수정이 최선이라고 생각이 들지만, 이벤트 처리에 대한 추상화 단계를 설계과정에서부터 신경 썼으면 좋았을 것 같다.

### Reference

---

- https://ko.javascript.info/mouse-events-basics

## 놀라운 사실  ㄴㅇㄱ …

- https://konvajs.org/api/Konva.Node.html#preventDefault
- https://github.com/konvajs/konva/issues/115

> get/set preventDefault By default all shapes will prevent default behavior of a browser on a pointer move or tap. that will prevent native scrolling when you are trying to drag&drop a node but sometimes you may need to enable default actions in that case you can set the property to false
> 

![image.png](%EC%9A%B0%ED%81%B4%EB%A6%AD%20%EC%8B%9C%20%EC%9D%B4%EB%8F%99%EB%AA%A8%EB%93%9C%20%ED%99%9C%EC%84%B1%ED%99%94%EB%90%98%EB%8A%94%20%EC%98%A4%EB%A5%98%EC%88%98%EC%A0%95/image%202.png)
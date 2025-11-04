---
title: 우여곡절 재탄생한 Gooey
date: 2024-11-21 10:00:00 +0900
categories: [Frontend]
tags: [React]
---

<aside>
📌

**참고
‣** 
‣ 
‣ 

</aside>

- 📒 탐색 과정에서 새롭게 알게된 것들 낙서
    
    MouseEvent의 clientX, clientY
    
    The **`clientX`** read-only property of the [`MouseEvent`](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent) interface provides the horizontal coordinate within the application's [viewport](https://developer.mozilla.org/en-US/docs/Glossary/Viewport) at which the event occurred (as opposed to the coordinate within the page).
    
    `dragstart`, `dragmove`, and `dragend`.
    
    FXXX…
    
    ***커스텀 훅 내부에 작성된 리액트 훅은 각 컴포넌트 별로 "독립적"으로 동작***합니다.
    

## 🐝 구현 기본 아이디어

---

기존에는 두 노드를 필터 처리하고, 해당 부분이 “얼마나 겹치는지” 그 정도에 따라서 끈적이를 표시하는 방식이었다. 하지만, 이제는 단순히 두 노드의 거리를 기반으로 끈적이와 비슷한 **“곡선”을 표현하기로 했다.**

*(인터랙션 구현 방식이 변경된 이유는 위의 [Konva를 통해 Canvas 구조 잡기 링크](https://www.notion.so/Gooey-256d364a49638079a80dcb528c723438?pvs=21) 참고 바람)*

핵심은 두 노드 사이의 적절한 조절점을 두고. 이를 기반으로 한 곡선 2개를 그리면 된다.

이때 [Bezier 곡선](https://ko.javascript.info/bezier-curve)을 활용할 수 있다. 두 원의 중심으로부터 시작점, 끝점, 조절점을 잘 계산해내면 노드가 떨어진 거리에 맞춰서 편리하게 곡선을 그릴 수 있기 때문이다.

![IMG_C8795E5BE4D7-1.jpeg](/assets/img/posts/gooey-overview.jpeg)

## 🐝 구현 흐름과 몇 가지 핵심 이슈

---

![노드생성-phase1.jpeg](/assets/img/posts/gooey-1.jpeg)

![노드생성-phase2.jpeg](/assets/img/posts/gooey-2.jpeg)

![노드생성-phase3.jpeg](/assets/img/posts/gooey-3.jpeg)

![노드생성-phase4.jpeg](/assets/img/posts/gooey-4.jpeg)

기본적으로 노드 생성은 Palette 메뉴에서 타입을 선택하기까지 **위의 4가지 단계**를 거친다.

1. 화면에 렌더링된 노드에서 Drag를 시작
2. 끈적이를 표현할 새로운 가짜 노드(: Gooey노드)가 드래그되는 커서와 함께 이동
3. 두 노드가 겹쳐져 있지 않으면 끈적이 연결부(: GooeyConnection) 표시
4. 새로운 가짜 노드(: Gooey노드)를 드롭하면 해당 좌표에 Palette 메뉴 표시

### Drag 시작한 노드가 드래그되면 안됨!

첫 번째 난관은 **드래그 이벤트**를 핸들링하는 것이었다.

![image.png](/assets/img/posts/gooey-2.jpeg)

기존 노드에서 발생한 Drag 이벤트를 기반으로 핸들러들을 사용하기 위해서는, 노드에 `draggable` 속성을 줘야 했다. *(Konva에서 제공하는 Konva Node에 적용할 수 있는 기본 속성이다)*

그런데 draggable 속성을 주면 실제 노드가 드래그되면서 같이 이동해버린다. 노드 생성과정에서는 Gooey Node만 움직여야 한다.

**여러 가지 선택지**가 있다. draggable 속성을 유지하는 것은 불가피하다.

1. 기존 노드는 dragStart 이벤트만 받고, 이후에는 기존 노드에 drag 이벤트 자체가 발생하는 것을 임의로 막아버린다. 대신 dragMove, dragEnd 등의 이벤트는 Stage로 위임하여 처리한다.
2. Konva에서 제공하는 dragBoundFunc로 드래그 가능 범위를 처음 클릭한 좌표(점 하나)로 제한한다.

**`2번째 방식`을 선택했다.** 이유는 노드 생성, 간선 생성 등 드래그를 기반으로 해서 동작하는 기능들이 많은데 Stage에 이벤트를 넘겼다가 다시 해제하는 등의 로직을 포함할 경우, 태스크를 나눠서 개발하면 복잡해질 수도 있겠다고 생각했다. dragBoundFunc를 사용하면 실제로 노드가 드래그 되어야할 때만 범위 정의 함수를 바꾸면 되기에 간단해보였다.

공식문서의 정의는 다음과 같다.

```jsx
// get drag bound function
var dragBoundFunc = node.dragBoundFunc();

// create vertical drag and drop
node.dragBoundFunc(function(pos){
  // important pos - is absolute position of the node
  // you should return absolute position too
  return {
    **x: this.absolutePosition().x,**
    **y: pos.y**
  };
});
```

중요한 부분은 **“절대 위치”**를 반환해야 한다는 점이었다.

node 데이터들은 화면의 정중앙을 (0,0)으로 가지는 상대좌표가 기본값이므로, 이를 다시 보정하는 작업이 필요했다. 따라서 아래와 같이 `stageSize.xx / 2`를 더해준다.

```jsx
function createDragBoundFunc(node: Node) {
    return function dragBoundFunc() {
      /** 원래 위치로 고정. stage도 draggable하므로 Layer에 적용된 offset을 보정하여 절대 위치로 표시.  */
      return {
        x: node.x + stageSize.width / 2,
        y: node.y + stageSize.height / 2,
      };
    };
  }
```

*11.23 updated* 

화면을 조작하면 stage가 drag되기 때문에 offset을 보정한다고 하더라도 위치가 깨져버린다. 따라서 최종적으로는 뷰포트 기준의 절대위치를 반환하는 방식으로 수정하였다.

```tsx
function dragBoundFunc(this: Konva.Node) {
    return this.absolutePosition();
}
```

### 거리에 따라 부드러운 Bezier 곡선 만들기

### 조절점에 대해

어설픈 그림으로는 티가 안날 수도 있지만, 초기에 구상했던 대로 조절점을 1개만 활용하면 왼쪽과 같이 가파른 곡선이 만들어진다. 특히나 거리가 멀어지면 부드러운 끈적이 표현을 하지 못할 수도 있기에 2개를 사용하기로 했다.

아래와 같이 조절점을 2개를 정의하여 곡률을 좀더 완만하게 만들었다.

**`조절점 1개`**

![조절점1.jpeg](/assets/img/posts/control-p-1.jpeg)

**`조절점 2개`**

![조절점2.jpeg](/assets/img/posts/control-p-2.jpeg)

1개일 때는 단순히 노드 2개 중심점의 중점으로 정의하면 됐었다. 그럼 조절점 2개의 위치는 어떻게 정의하면 좋을까? “부드러운 곡선”을 생각해보면, 조절점이 두 원의 중점에 가까울수록 급격한 곡선이 생긴다.

**`조절점이 노드에서 멀 때`**

![image.png](/assets/img/posts/control-p-3.png)

**`조절점이 노드에서 가까울 때`**

![image.png](/assets/img/posts/control-p-4.png)

이를 위해 조절점의 위치가 너무 멀어지지 않도록 가중치 값을 활용했다. [거리의 절반], [노드 지름] 중 더 작은 값을 택하여 활용한다. (1.3은 직접 곡선을 확인하면서 적절한 곡률을 보이는 값을 찾았다)

```tsx
const controlDistance = Math.min(distance * 0.5, NODE_RADIUS * 2);
...
const ctrl1 = {
    x: startPosition.x + Math.cos(angle) * **controlDistance * 1.3**,
    y: startPosition.y + Math.sin(angle) * **controlDistance * 1.3**,
  };
```

### 시작점.끝점에 대해

베지어 곡선의 시작점.끝점을 정의하는 방식도 변경하였다. 기존 방식에서는 [두 노드의 중심점을 이은 직선]과 [수직을 이루는 두 개의 할선]을 기준으로 정의했다. 이것도 충분했지만 문제는 곡선의 시작이 원의 중간지점에서부터 시작하여 약간의 부자연스러움이 있었다. 

“분열되는 듯한” 느낌을 주기위해서는 곡선이 원으로부터 자연스럽게 파생되는 것처럼 보이는 것이 필요했고, 이후 시작점.끝점을 두원의 접점을 기준으로 수정하였다.

**`BEFORE`**

![image.png](/assets/img/posts/control-p-before.png)

**`AFTER`**

![image.png](/assets/img/posts/control-p-final.png)

## 🍯 돌아보기

이렇게 많은 수학적 연산이 있어도 성능이 확연히 떨어지는 것을 방지할 수 있는 이유는 Canvas 덕분이다. 복잡한 수학 연산이 많더라도 최종적으로는 하나의 Canvas Context에 그려지기 때문이다.

```tsx
typescript
Copy
<Shape
  sceneFunc={(context) => {
// 모든 계산과 그리기가 하나의 Context 내에서 처리됨
    context.beginPath();
    context.bezierCurveTo(ctrl1.x, ctrl1.y, ctrl2.x, ctrl2.y, end1.x, end1.y);
// ...
  }}
/>

```

번외로 svg를 통해서 그렸을 때와 비교해봐도 재밌을 것 같다.
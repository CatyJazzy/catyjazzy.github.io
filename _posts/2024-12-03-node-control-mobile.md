---
title: 모바일을 위한 노드.간선 조작
date: 2024-12-03 10:00:00 +0900
categories: [Frontend]
tags: [React]
---

![모바일우클릭.gif](%EB%AA%A8%EB%B0%94%EC%9D%BC%EC%9D%84%20%EC%9C%84%ED%95%9C%20%EB%85%B8%EB%93%9C%20%EA%B0%84%EC%84%A0%20%EC%A1%B0%EC%9E%91/%25E1%2584%2586%25E1%2585%25A9%25E1%2584%2587%25E1%2585%25A1%25E1%2584%258B%25E1%2585%25B5%25E1%2586%25AF%25E1%2584%258B%25E1%2585%25AE%25E1%2584%258F%25E1%2585%25B3%25E1%2586%25AF%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25A8.gif)

## 🤔 모바일 환경에 대한 고민 배경

---

친구들에게 프로젝트 배포 링크를 자랑하면서 점차 다양한 피드백을 받을 수 있었다. 개발자 친구들은 대부분 PC로 접속했으나, 대부분의 친구들은 핸드폰에서 접속을 했다. 많은 친구들에게 노드 삭제도 못해?라는 말을 했는데, 생각해보니 모바일 환경에서는 우클릭을 통한 컨텍스트 메뉴 표시가 안됐다.

이를 계기로 모바일 환경에서의 사용성을 고민하게 되었다. 나는 노션을 사용할 때도 10번 중의 1번은 모바일에서 쓸 때가 있다. 특정 문서에 바로 접근하기 위함이다. **HoneyFlow에서도 Space 내의 모든 조작이 모바일 환경의 사용성을 고려해야겠다는 생각이 들었다.**

## 🤚 Space 조작 정리해보기

---

조작 지원여부를 아래와 같이 정리해보자. 

|  | **모바일**  | **데스크톱** | **구현에서 활용한 이벤트** |
| --- | --- | --- | --- |
| 노드 생성 | 드래그/드롭 | 드래그/드롭 | onDragStart / onDragMove / onDragEnd |
| 노드 편집 | **X** | 우클릭을 통한 contextmenu 사용 | **onContextMenu** |
| 노드 제거 | **X** | 우클릭을 통한 contextmenu 사용 | **onContextMenu** |
| 노드 이동 | 길게 터치 후 드래그 | 길게 클릭 후 드래그 | onMouseDown-onTouchStart / onDragMove / onDragEnd-*onMouseLeave-onMouseUp-onTouchEnd* |
| 간선 생성 | 드래그/드롭 | 드래그/드롭 | onDragStart / onDragMove / onDragEnd |
| 간선 제거 | **X** | 클릭을 통한 contextmenu 사용 | **onContextMenu** |

문제는 모두 onContextMenu에서 시작되었다. 모바일에서는 우클릭 지원이 안되는데, 편집.제거에 대한 것들이 다 컨텍스트 메뉴에서 가능하므로 방법이 없다.

## 📱 모바일에서도 편집을 가능하게 하려면?

---

### 모바일에서의 onContextMenu 이벤트…

이전에 edge 생성 방식에 대한 예시를 알아보기 위해 아래의 두 가지 케이스를 조사했었다. yjs에서 제공한 예시 외에 다른 그래프뷰 동시편집 데모에서도 edit모드를 활성화하는 버튼이 따로 있는 케이스가 많았다. 

그럼 우리도 ***edit 모드를 따로 구현하여 이를 전환할 수 있는 버튼이 있어야 할까?*** 정답이 있는 것은 아니지만 원했던 방향은 아니라고 생각했다.

HoneyFlow의 가장 큰 강점이자 목표는 쉬운 인터랙션을 통한 지식 구조 관리다. 조작 과정에서 조금이라도 중간단계가 많아지면 이 특징을 해칠 것이다.

**`기존방식`** 우클릭 → 메뉴 → 편집 or 제거

**`edit모드 방식`** 클릭으로 edit모드 활성화 → 편집 대상 선택 → 메뉴 → 편집 or 제거

한 눈에 봐도 과정이 복잡해진다. 편집 대상을 선택하는 것이 번거롭기 때문이다. 그럼, 우클릭 대신 어떤 이벤트를 줘야할까? 조사해본 결과 모바일 환경에서 우클릭(onContextMenu)을 대체하는 것은 주로 **LongClick**이라고 한다. 일정 시간 동안 꾹 누르고 있는 것이다.

그런데 HoneyFlow에서는 이미 0.5초만 노드 위에서 홀드해도 이동 모드가 활성화 된다. 또다른 인터랙션인 노드이동으로 전환되는 것이다. 이로써 또다른 이벤트를 떠올리는 것은 어려워졌다.

### 유레카, 더보기 버튼을 넣자!

문득 점3개의 더보기가 떠올랐다. 위의 react-flow의 데모 예시에서 간선을 삭제할 수 있는 버튼이 있듯이, context-menu를 트리거 할 장치를 만들어주는 것이다!

![image.png](%EB%AA%A8%EB%B0%94%EC%9D%BC%EC%9D%84%20%EC%9C%84%ED%95%9C%20%EB%85%B8%EB%93%9C%20%EA%B0%84%EC%84%A0%20%EC%A1%B0%EC%9E%91/image.png)

노드의 예쁜 외관을 해치니, 더보기 버튼은 모바일 환경에서”만” 넣고 싶었다. 따라서 우리가 구현한 Node 컴포넌트에서 모바일 환경을 식별할 방법이 필요하다.

**🔎 터치 디바이스 식별하기**

---

모바일 환경을 감지할 수 있는 방식은 다양하다.

(1) **navigator.userAgent**를 통해 아래와 같이 확인되는 값으로 식별하는 방법

![image.png](%EB%AA%A8%EB%B0%94%EC%9D%BC%EC%9D%84%20%EC%9C%84%ED%95%9C%20%EB%85%B8%EB%93%9C%20%EA%B0%84%EC%84%A0%20%EC%A1%B0%EC%9E%91/image%201.png)

![image.png](%EB%AA%A8%EB%B0%94%EC%9D%BC%EC%9D%84%20%EC%9C%84%ED%95%9C%20%EB%85%B8%EB%93%9C%20%EA%B0%84%EC%84%A0%20%EC%A1%B0%EC%9E%91/image%202.png)

(2) **터치스크린의 유무**를 통해 식별하는 방법

navigator.maxTouchPoints (navigator 오브젝트의 터치표면에 존재하는 터치포인트 갯수를 반환)

“ontouchstart”가 가능한지 식별 "ontouchstart" in window

(3) window.matchMedia를 통해 **미디어 쿼리 문자열 속 정보를 확인**하는 방법

screen의 max-width 설정값, pointer 설정 등으로 식별

포인팅 장치 **none** | **coarse** (기본 장치가 터치스크린이나 키넥트처럼 정확도는 높지 않지만 포인팅은 가능) | **fine** (기본 장치가 마우스나 터치 패드처럼 정확한 포인팅이 가능)

위의 방식 중 **2번**이 제일 적절하다고 판단했다. 엄밀히 말하면, 우클릭이 불가능한 터치 디바이스인지만 확인하면 되기 때문이다. screen크기나 pointer에 관한 정보 또는 iphone, android 등의 세부적인 식별도 필요없다.

**`Node.MoreButton`**에 아래와 같이 적용하였다!

```tsx
...
const [isTouch, setIsTouch] = useState(false);

  useEffect(() => {
    setIsTouch("ontouchstart" in window || navigator.maxTouchPoints > 0);
  }, []);

  if (!isTouch) return null;
...
```

*(+) Node.MoreButton으로 정의한 이유는 다음과 같이 NoteNode, HeadeNode 등이 아토믹한 Node 컴포넌트들의 조합으로 만들어지기 때문이다. Node 컴포넌트 안에 있는 것들은 Konva의 Group으로 묶이는데, 이 안에 소속되어야 좌표 계산과 배치가 편리하다.*

```tsx
<Node
    id={id}
    x={x}
    y={y}
    onClick={(e) => {
      if (e.evt.button === 0) {
        navigate(`/note/${src}`);
      }
    }}
    onContextMenu={onContextMenu}
    {...rest}
>
  <Node.Circle radius={RADIUS} fill="#FAF9F7" stroke="#DED8D3" />
  <Node.Text width={RADIUS * 2} fontSize={16} content={name} />
  <Node.MoreButton onTap={onContextMenu} content="⋮" />
</Node>
```

## 🚨 더보기 버튼 클릭 시 handleContextMenu 함수를 그대로 활용하려면?

---

산 넘어 산이다. 모바일 환경에서만 더보기 버튼이 표시되도록 하는 것은 쉽게 구현하였는데, 또다른 허들이 있었다.

그동안의 컨텍스트 메뉴에 대한 맥락을 살펴보자.

- 컨텍스트 메뉴는 shadcn을 통해 구현하였다. shadcn은 메뉴 표시를 트리거하는 부분을 <ContextMenu.Trigger>로 감싸줘야 한다. 우리의 구현 맥락에서는 Konva의 Stage를 Trigger로 래핑한 상태다.
- 노드 Group을 기준으로 onContextMenu에 대한 이벤트를 핸들링하고 있다. 이유는 아래와 같이 id를 group에 정의하면, **해당 group에서 이벤트가 발생했을 때 Konva 이벤트의 e.target.attrs을 통해 정의했던 id를 얻을 수 있다.** → 이를 기반으로 노드의 이름이나 타입에 대한 데이터에 접근할 수 있어서 수정요청에 필요한 데이터를 쉽게 얻는다.

```tsx
 head: (node: Node) => (
      <HeadNode
        key={node.id}
        **id={node.id}**
        name={node.name}
        onDragStart={() => drag.handlers.onDragStart(node)}
        onDragMove={drag.handlers.onDragMove}
        onDragEnd={() => drag.handlers.onDragEnd()}
        dragBoundFunc={dragBoundFunc}
      />
    ),
```

> shadcn을 기반으로 구현한 컨텍스트 메뉴 로직과 handleContextMenu 함수를 그대로 활용하기 위해서는 더보기 버튼 클릭 시 우클릭 이벤트를 일부러/임의로(?) 😈 발생시키는 방법밖에 없었다.
> 

다행히도 Konva에서 특정 이벤트를 발생하는 메서드를 제공한다.

![image.png](%EB%AA%A8%EB%B0%94%EC%9D%BC%EC%9D%84%20%EC%9C%84%ED%95%9C%20%EB%85%B8%EB%93%9C%20%EA%B0%84%EC%84%A0%20%EC%A1%B0%EC%9E%91/image%203.png)

KonvaEventObject의 내부 타입을 살펴보면 아래와 같다.

```tsx
export interface KonvaEventObject<EventType, This = Node> {
    type: string;
    **target: Shape | Stage;**
    **evt: EventType;**
    pointerId: number;
    currentTarget: This;
    cancelBubble: boolean;
    child?: Node;
}
```

자 이제 더보기 버튼 Group에서 onTap이벤트가 발생하면, 해당 핸들러에서 임의로 우클릭 이벤트를 발생시켜주면 된다. 기존의 노드 위에서 우클릭하면 target이 해당 노드가 되듯이 더보기 버튼을 눌러도, 해당 버튼이 속한 노드를 찾아줘야 한다. 

Node.moreButton 자체도 Group으로 감싸져 있으므로, Node 컴포넌트의 Group을 얻기 위해서는 두 번의 조상 group을 찾는 과정이 필요하다. 이렇게 찾고 나서, contextmenu 이벤트 타입으로 fire시켜주기만 하면 끝난다!

```tsx
const parentNode = e.target.findAncestor("Group").findAncestor("Group");
if (!parentNode) return;

const contextMenuEvent = new MouseEvent("contextmenu", {
  **button: 2,
  buttons: 2,**
});

parentNode.fire("contextmenu", {
  **evt: contextMenuEvent,
  target: parentNode,**
});
```

이거 하나 보려고 반나절을 쏟은 것 같다. 이래서 마지막 주차에 feature는 하면 안되는데… 아무것도 모르면서 아주 간단할 거라고 생각한 나의 잘못이다… (머리가 안좋으면 몸이 고생한다더니 🥲)

반응 속도가 좀 느린데, 내 아이폰에서 테스트해보기 위해 ios를 업데이트하고 있다 ^^ websocket 관련한 브라우저 이슈가 있어서 테스트도 못한다. 

![모바일우클릭.gif](%EB%AA%A8%EB%B0%94%EC%9D%BC%EC%9D%84%20%EC%9C%84%ED%95%9C%20%EB%85%B8%EB%93%9C%20%EA%B0%84%EC%84%A0%20%EC%A1%B0%EC%9E%91/%25E1%2584%2586%25E1%2585%25A9%25E1%2584%2587%25E1%2585%25A1%25E1%2584%258B%25E1%2585%25B5%25E1%2586%25AF%25E1%2584%258B%25E1%2585%25AE%25E1%2584%258F%25E1%2585%25B3%25E1%2586%25AF%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25A8.gif)

## 🚨 간선 삭제는 어떻게 해야 할까?

---

노드에서는 더보기 버튼을 통해 모바일 편집 문제를 해결했다. 간선은 더욱 까다롭다. 직선으로 얇게 표시된 것에 어떤 형식으로 트리거를 줘야 할까? 노드와 달리 간선 위에서는 LongClick을 활용할 수 있다. 간선에서는 이동모드가 없기 때문이다.

그럼에도 LongClick을 활용하기엔 애매했다. 간선의 영역이 얇고 길기 때문에 이벤트 감지가 원활하지 않을 수도 있겠다고 생각했다. 간선 삭제도 별도의 버튼을 가져가되, 간선의 중점에 표시하도록 했다.

![모바일간선.gif](%EB%AA%A8%EB%B0%94%EC%9D%BC%EC%9D%84%20%EC%9C%84%ED%95%9C%20%EB%85%B8%EB%93%9C%20%EA%B0%84%EC%84%A0%20%EC%A1%B0%EC%9E%91/%25E1%2584%2586%25E1%2585%25A9%25E1%2584%2587%25E1%2585%25A1%25E1%2584%258B%25E1%2585%25B5%25E1%2586%25AF%25E1%2584%2580%25E1%2585%25A1%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A5%25E1%2586%25AB.gif)

## Reference

---

- https://marshallku.com/dev/css%EB%A1%9C-%EA%B8%B0%EA%B8%B0-%ED%8C%8C%EC%95%85%ED%95%98%EA%B8%B0
- https://www.radix-ui.com/primitives/docs/components/context-menu#api-reference
- https://velog.io/@rmaomina/javascript-mobile-detects-User-Agent
- https://konvajs.org/api/Konva.Node.html#findAncestor
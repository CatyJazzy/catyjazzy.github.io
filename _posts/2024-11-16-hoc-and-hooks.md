---
title: 리액트 합성 컴포넌트와 커스텀훅
date: 2024-11-16 10:00:00 +0900
categories: [Frontend]
tags: [React]
---

## 💡 합성 컴포넌트의 의미

---

합성 컴포넌트를 많은 리액트 개발자들이 선택하는 이유를 먼저 생각해보자. 핵심 정의에서부터 시작하자!

> **Sub-컴포넌트들이 Main-컴포넌트 내부의 상태를 공유**하면서 비즈니스 로직과 UI 관련된 부분을 구분하는 리액트 패턴. **여러 개의 작은 컴포넌트들이 각각의 책임을 분담**하여 이를 자유롭게 조합하여 하나의 큰 컴포넌트를 만드는 것. [// from 링크 //](https://velog.io/@hugh0223/React-%ED%95%A9%EC%84%B1-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%99%84%EC%A0%84-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0)
> 

핵심은 **“책임”과 “조합”**이다. 재사용성이 높게 만들기 위해서는, 책임을 컴포넌트들이 나눠가져야 다양한 케이스로 조합할 수 있기 때문이다. 상황에 따라 모양을 다르게 표기하거나, 이벤트 발생시 동작이 달라진다면? 

하나의 컴포넌트에서 조건 분기가 매우 복잡해지거나, 요구사항이 바뀌고 추가할 때마다 props가 쌓이는 불편한 문제가 발생할 것이다. 

## 🐝 HoneyFlow에 적용해보기

---

HoneyFlow에서 재사용될 수 있는 컴포넌트가 어떤 것이 있을까? 일단 기본적인 Node를 생각해보자.

Node의 종류, 형태, 기능을 최대한 나열해보면 다음과 같이 정리할 수 있다.

- **`HeadNode`**
    
    → 스페이스명 & 원 표시
    
    → 노드 생성(드래그) 시작점
    
- **`ContentNode`**
    - `NoteNode`
        
        → 노트명 & 원 표시
        
        → 노드 생성(드래그) 시작점
        
        → 위치이동
        
    - `ImageNode`
        
        → 이미지 & 사각형 표시
        
        → 노드 생성(드래그) 시작점
        
        → 위치 이동
        
    - `UrlNode`
        
        → url & 원 표시
        
        → 노드 생성(드래그) 시작점
        
        → 위치 이동
        
    - `SubspaceNode`
        
        → 스페이스명 & 원 표시
        
        → 노드 생성(드래그) 시작점
        
        → 위치 이동
        

### 1️⃣ 기본 Node

---

- Node는 상대 좌표를 받아서 ***노드의 위치**를 잡는다.*

```tsx
export default function Node({
  x,
  y,
  draggable,
  children,
  ...rest
}: NodeProps) {

  return (
    <Group
      x={x}
      y={y}
      draggable={draggable}
      onDragStart={handleDragStart}
      {...rest}
    >
      {children}
    </Group>
  );
}
```

그럼 형태는 어떻게 잡는가? 여기서부터 합성 컴포넌트 패턴을 적용하여 Node.Circle, Node.Text, Node.Rect 등을 통해 ***실제 화면에 표시할 “형태”의 책임**을 담당한다.*

// **Node.Circle** (반지름, 색을 받는다)

```tsx
Node.Circle = function NodeCircle({ radius, fill }: NodeCircleProps) {
  return <Circle x={0} y={0} radius={radius} fill={fill} />;
};
```

// **Node.Text** (내용, 글씨크기, 폰트, 너비를 받는다. 기본적으로 중앙 정렬)

```tsx
Node.Text = function NodeText({
  content,
  fontSize,
  fontStyle,
  width,
}: NodeTextProps) {
  ...

  return (
    <Text
      ref={ref}
      fontSize={fontSize}
      fontStyle={fontStyle}
      offset={offset}
      text={content}
      width={width}
      align="center"
      wrap="none"
      ellipsis
    />
  );
};
```

### 2️⃣ Node 종류

---

이제 구현 단계에서 로직과 결합된 실제 **Main 컴포넌트들**을 정의할 차례다. 크게 HeadNode와 ContentNode의 종류들인 NoteNode, UrlNode… 등으로 나뉜다.

// **HeadNode** 좌표가 고정되어있으므로, 스페이스명만 받는다.

- Node로 위치 고정
    - Node.Circle로 원의 형태 표시
    - Node.Text로 스페이스명 표시

```tsx
export function HeadNode({ name }: HeadNodeProps) {
  const radius = 64;
  return (
    <Node x={0} y={0}>
      <Node.Circle radius={radius} fill="#FFCC00" />
      <Node.Text
        width={radius * 2}
        fontSize={16}
        fontStyle="700"
        content={name}
      />
    </Node>
  );
}
```

// **NoteNode** 좌표, 노트명만 받는다.

- Node로 위치 고정
    - Node.Circle로 원의 형태 표시
    - Node.Text로 노트명 표시

팀이 초기 설계할 때 원했던 대로 Node.Circle, Node.Text를 자유롭게 조합하여 정의하는 형태가 되었다. 

<aside>
❗ 그럼 **드래그로 파생되는 다양한 이벤트 핸들링**을 어느 곳의 책임으로 둬야 할까?

</aside>

별도의 컴포넌트로 분리하기보다는, 커스텀훅으로 분리하여 사용하기로 했다. 사실 나는 아직 커스텀훅을 많이 사용해본 적이 없다 🥲

데이터 패칭하는 부분만 분리해본 정도. 따라서, 이번 기회로 커스텀훅 Best Practice를 살펴보자.

### 3️⃣ 이벤트 핸들링 로직 커스텀 훅으로 분리하기

---

**📒 기초바탕 쌓기** 

공식 문서의 정의 **“Hook” 정의**는 아래와 같다.

> *훅이란, 컴포넌트의 위계를 변경하지 않으면서도 **state 변화가 들어 있는 로직(stateful logic)을 재사용**하도록 해 준다.*
> 

실제로 적용하기에 유용한 상황을 통해 이해를 더 구체화해보자. 내가 읽은 아티클에서는 다음의 상황을 가정한다.

- 스크린 사이즈에 따라 UI 변경
- 사용자가 브라우저 창을 변경할 때 창의 크기 추적

```tsx
const ScreenDimensions = () => {
	
	// 창의 크기를 상태로 관리 
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined,
  });
  
	// 스크린 사이즈 조정
  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }
    window.addEventListener('resize', handleResize);
    handleResize();
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return (
  	<>
    	<p>Current screen width: {windowSize.width}</p>
        <p>Current screen height: {windowSize.height}</p>
    </>
  )
}
```

이때 스마트폰, 데스크톱용으로 사이즈마다 UI를 다르게 렌더링할 때 어떻게 적용할 수 있을까? 렌더링을 달리하는 두 컴포넌트에 로직을 그대로 복사해서 사용할 수도 있지만, **해당 로직이 변경되었을 때 수정 비용이 커진다.**

따라서 아래와 같이 `useWindowSize`로 재사용되는 로직을 분리해보자.

```tsx
const **useWindowSize** = () => {
  const [windowSize, setWindowSize] = useState({
    width: undefined,
    height: undefined,
  });
  
  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }
    window.addEventListener('resize', handleResize);
    handleResize();
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return windowSize;
}
```

그러면 해당 로직이 사용되는 곳 어디든, 아래와 같이 간단한 형태로 호출하여 활용할 수 있다.

```tsx
const ScreenDimensions = () => {
  **const windowSize = useWindowSize()**
  
  return (
  	<>
    	<p>Current screen width: {windowSize.width}</p>
        <p>Current screen height: {windowSize.height}</p>
    </>
  )
}
```

## Reference

- [https://fe-developers.kakaoent.com/2022/220731-composition-component/](https://fe-developers.kakaoent.com/2022/220731-composition-component/)
- [https://velog.io/@hugh0223/React-합성-컴포넌트-완전-정복하기](https://velog.io/@hugh0223/React-%ED%95%A9%EC%84%B1-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%99%84%EC%A0%84-%EC%A0%95%EB%B3%B5%ED%95%98%EA%B8%B0)
- [https://hj-devlog.vercel.app/blog/컴포넌트 작성에 대한 고민들 (합성 컴포넌트)](https://hj-devlog.vercel.app/blog/%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%20%EC%9E%91%EC%84%B1%EC%97%90%20%EB%8C%80%ED%95%9C%20%EA%B3%A0%EB%AF%BC%EB%93%A4%20(%ED%95%A9%EC%84%B1%20%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8))
- [https://song-dev.vercel.app/blog/React-Compound-Comonent](https://song-dev.vercel.app/blog/React-Compound-Comonent)
- [https://www.freecodecamp.org/korean/news/best-practices-for-react/](https://www.freecodecamp.org/korean/news/best-practices-for-react/)

**📒 HoneyFlow에 적용해서 수정하기**

내가 생각한 수정의 방향을 다이어그램으로 그려달라고 claude에게 부탁했다 ㅎ.ㅎ 
*객체지향 다이어그램 그릴 때처럼, -는 함수 내부에서만 알고, +는 props와 같이 외부와 공유하는 값이다.*

- deprecated
    
    ```mermaid
    classDiagram
        SpaceView --> useNodeDrag
        SpaceView --> HeadNode
        SpaceView --> NoteNode
        SpaceView --> PaletteMenu
        HeadNode --> Node
        NoteNode --> Node
    
        class SpaceView {
            -stageSize
            +render()
        }
    
        class useNodeDrag {
            -dragStartNodeId
            -palettePosition
            +handleDragStart()
            +handleDragEnd()
            +handlePaletteSelect()
        }
    
        class Node {
            +x
            +y
            +draggable
            +children
        }
    
        class HeadNode {
            +name
        }
    
        class NoteNode {
            +x
            +y
            +name
        }
    
        class PaletteMenu {
            +items
            +onSelect
        }
    ```
    
    ```mermaid
    classDiagram
        SpaceView --> useNodeDrag
        SpaceView --> useNodeOperations
        SpaceView --> HeadNode
        SpaceView --> NoteNode
        SpaceView --> PaletteMenu
        HeadNode --> Node
        NoteNode --> Node
        useNodeDrag ..> useNodeOperations : uses operations
        PaletteMenu ..> Node : uses Node["type"]
    
        class SpaceView {
            -stageSize
            -nodes
            -setNodes
            +render()
        }
    
        class useNodeOperations {
            -nodes
            -setNodes
            -stageSize
            +onCreateNode()
        }
    
        class useNodeDrag {
            -dragState
            -nodeOperations
            +handleDragStart()
            +handleDragEnd()
            +handlePaletteSelect()
        }
    
        class Node {
            +x
            +y
            +draggable
            +children
        }
    
        class PaletteMenu {
            +items
            +onSelect
            -buttonConfig
            -getPositionByIndex()
        }
    
        class HeadNode {
            +name
            +onDragStart
        }
    
        class NoteNode {
            +x
            +y
            +name
            +onDragStart
        }
    
        note for SpaceView "상태 관리 및\n컴포넌트 조율"
        note for useNodeOperations "노드 조작 로직"
        note for useNodeDrag "드래그 관련 로직"
    ```
    
    ```mermaid
    classDiagram
        SpaceView --> useNodeDrag
        SpaceView --> useNodeOperations
        SpaceView --> HeadNode
        SpaceView --> NoteNode
        SpaceView --> Edge
        SpaceView --> PaletteMenu
        HeadNode --> Node
        NoteNode --> Node
        useNodeDrag ..> useNodeOperations : uses operations
        PaletteMenu ..> NodeType : uses type
        Edge ..> NodeType : uses for positions
    
        class SpaceView {
            -stageSize
            -nodes
            -edges
            -setNodes()
            -setEdges()
            +render()
        }
    
        class useNodeOperations {
            -setNodes
            -setEdges
            -stageSize
            +onCreateNode()
        }
    
        class useNodeDrag {
            -dragState
            -nodeOperations
            +handleDragStart()
            +handleDragEnd()
            +handlePaletteSelect()
        }
    
        class Node {
            +id
            +type
            +name
            +x
            +y
        }
    
        class PaletteMenu {
            +items
            +onSelect()
            -buttonConfig
            -getPositionByIndex()
        }
    
        class HeadNode {
            +name
            +onDragStart()
        }
    
        class NoteNode {
            +x
            +y
            +name
            +onDragStart()
        }
    
        class Edge {
            +from
            +to
            +nodes
        }
    
        note for SpaceView "스테이지 크기 관리\n노드/엣지 상태 관리"
        note for useNodeOperations "노드/엣지 생성 로직"
        note for useNodeDrag "드래그 시작/종료 처리\n팔레트 선택 처리"
        note for PaletteMenu "노드 타입 선택 UI"
    ```
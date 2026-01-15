# ReactComplexTreeWidget 개발자 가이드

> Mendix Pluggable Widget 개발 스펙 및 비즈니스 로직 지침서
> 주니어 개발자 교육용 문서

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [기술 스택](#2-기술-스택)
3. [프로젝트 구조](#3-프로젝트-구조)
4. [핵심 컴포넌트 상세 설명](#4-핵심-컴포넌트-상세-설명)
5. [비즈니스 로직 및 데이터 흐름](#5-비즈니스-로직-및-데이터-흐름)
6. [위젯 설정 옵션](#6-위젯-설정-옵션)
7. [개발 가이드라인](#7-개발-가이드라인)
8. [빌드 및 배포](#8-빌드-및-배포)
9. [트러블슈팅](#9-트러블슈팅)

---

## 1. 프로젝트 개요

### 1.1 위젯 정보

| 항목 | 내용 |
|------|------|
| **위젯명** | ReactComplexTreeWidget |
| **버전** | 1.3.0 |
| **개발사** | Aiden |
| **라이선스** | Apache 2.0 |
| **플랫폼** | Mendix Pluggable Widget (Web 전용) |

### 1.2 위젯 목적

이 위젯은 Mendix 애플리케이션에서 **복잡한 계층형 트리 구조**를 표시하기 위해 개발되었습니다.

**주요 기능:**
- 계층형 트리 데이터 시각화
- Lazy Loading (지연 로딩) 지원
- Drag & Drop 기능
- 노드 이름 변경
- 파일 타입별 아이콘 표시
- 확장/축소 버튼

### 1.3 핵심 라이브러리

이 위젯은 [react-complex-tree](https://github.com/lukasbach/react-complex-tree) 라이브러리를 기반으로 합니다. 해당 라이브러리는 대용량 트리 데이터를 효율적으로 처리할 수 있는 기능을 제공합니다.

---

## 2. 기술 스택

### 2.1 주요 기술

```
┌─────────────────────────────────────────────────────────────┐
│                    Mendix Platform                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   React     │  │  TypeScript │  │  react-complex-tree │ │
│  │   18.2.0    │  │   (JSX)     │  │       ^2.4.6        │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
├─────────────────────────────────────────────────────────────┤
│           @mendix/pluggable-widgets-tools 10.18.0           │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 의존성 패키지

**런타임 의존성:**
```json
{
  "react-complex-tree": "^2.4.6"
}
```

**개발 의존성:**
```json
{
  "@mendix/pluggable-widgets-tools": "~10.18.0",
  "@prettier/plugin-xml": "^3.4.1"
}
```

### 2.3 Node.js 요구사항

- **최소 버전:** Node.js 16 이상
- **패키지 매니저:** npm 또는 pnpm

---

## 3. 프로젝트 구조

### 3.1 디렉토리 구조

```
reactcomplexttree/
│
├── src/                                # 소스 코드
│   ├── ReactComplexTreeWidget.jsx      # 메인 위젯 컴포넌트
│   ├── ReactComplexTreeWidget.xml      # 위젯 설정 스키마
│   ├── ReactComplexTreeWidget.editorConfig.js  # 에디터 설정
│   ├── package.xml                     # Mendix 패키지 설정
│   │
│   ├── components/                     # React 컴포넌트
│   │   ├── TreeContainer.jsx           # 트리 렌더링 (핵심 컴포넌트)
│   │   └── CustomIcon.jsx              # 파일 아이콘 컴포넌트
│   │
│   ├── utils/                          # 유틸리티
│   │   └── treeDataReducer.js          # 상태 관리 리듀서
│   │
│   └── *.png                           # 위젯 아이콘 이미지
│
├── dist/                               # 빌드 출력
│   └── 1.3.0/
│       └── aiden.ReactComplexTreeWidget.mpk
│
├── package.json                        # npm 설정
├── .eslintrc.js                        # ESLint 설정
├── prettier.config.js                  # Prettier 설정
└── README.md
```

### 3.2 핵심 파일 역할

| 파일 | 역할 | 중요도 |
|------|------|--------|
| `ReactComplexTreeWidget.jsx` | 메인 진입점, 이벤트 핸들러 설정 | ⭐⭐⭐ |
| `TreeContainer.jsx` | 트리 UI 렌더링, 서비스 호출, 상태 관리 | ⭐⭐⭐⭐⭐ |
| `CustomIcon.jsx` | 파일 타입별 아이콘 렌더링 | ⭐⭐ |
| `treeDataReducer.js` | Redux 스타일 상태 관리 | ⭐⭐⭐ |
| `ReactComplexTreeWidget.xml` | 위젯 속성 정의 | ⭐⭐⭐⭐ |

---

## 4. 핵심 컴포넌트 상세 설명

### 4.1 ReactComplexTreeWidget.jsx (메인 컴포넌트)

**위치:** `src/ReactComplexTreeWidget.jsx`

**역할:** Mendix 프레임워크와 위젯을 연결하는 진입점

```jsx
// 주요 구조
export default function ReactComplexTreeWidget(props) {
    // 1. 이벤트 핸들러 정의
    const onSelectionChangedHandler = (nodeIDs) => { ... };
    const onMissingNodesHandler = (nodeIDs) => { ... };
    const onNodeRenamedHandler = (nodeID, newName) => { ... };
    const onDropHandler = (nodeIDs, dropTargetJSON) => { ... };

    // 2. TreeContainer에 props 전달
    return (
        <TreeContainer
            {...props}
            onSelectionChangedHandler={onSelectionChangedHandler}
            onMissingNodesHandler={onMissingNodesHandler}
            onNodeRenamedHandler={onNodeRenamedHandler}
            onDropHandler={onDropHandler}
        />
    );
}
```

**주요 이벤트 핸들러:**

| 핸들러 | 트리거 시점 | 역할 |
|--------|-------------|------|
| `onSelectionChangedHandler` | 노드 선택 시 | 선택된 노드 ID를 Mendix 속성에 저장 |
| `onMissingNodesHandler` | Lazy Loading 시 | 로드되지 않은 자식 노드 요청 |
| `onNodeRenamedHandler` | 노드 이름 변경 시 | 변경된 이름을 Mendix로 전달 |
| `onDropHandler` | Drag & Drop 시 | 드래그한 노드와 대상 정보 전달 |

### 4.2 TreeContainer.jsx (핵심 컴포넌트)

**위치:** `src/components/TreeContainer.jsx`

**역할:** 트리 UI 렌더링 및 모든 로직 처리

**코드 구조 분석:**

```jsx
export default function TreeContainer(props) {
    // ═══════════════════════════════════════════════════════════
    // 1. 상태(State) 정의
    // ═══════════════════════════════════════════════════════════

    // 트리 데이터 상태 (Reducer 사용)
    const [treeData, dispatch] = useReducer(treeDataReducer, initialTreeData);

    // 확장된 노드 목록
    const [expandedItems, setExpandedItems] = useState(["root"]);

    // 선택된 노드 목록
    const [selectedItems, setSelectedItems] = useState([]);

    // 포커스된 노드
    const [focusedItem, setFocusedItem] = useState();

    // 재로드 트리거
    const [triggerReload, setTriggerReload] = useState(false);

    // ═══════════════════════════════════════════════════════════
    // 2. REST 서비스 호출 (useEffect)
    // ═══════════════════════════════════════════════════════════

    useEffect(() => {
        // dataChangeDateAttr가 변경될 때마다 서비스 호출
        const fetchData = async () => {
            const response = await fetch(serviceUrl, {
                headers: {
                    "X-Csrf-Token": mx.session.getConfig("csrftoken")
                }
            });
            const result = await response.json();

            // action에 따라 처리
            switch (result.action) {
                case "reload":
                    dispatch({ type: "reload", data: newData });
                    break;
                case "update":
                    dispatch({ type: "update", data: newData, deletedNodeIDs });
                    break;
                case "focus":
                    // 특정 노드 포커스
                    break;
            }
        };
        fetchData();
    }, [dataChangeDateAttr, triggerReload]);

    // ═══════════════════════════════════════════════════════════
    // 3. 이벤트 핸들러
    // ═══════════════════════════════════════════════════════════

    // 노드 선택 핸들러
    const onSelectionChangedHandler = (items) => {
        setSelectedItems(items);
        props.onSelectionChangedHandler(items.join(","));
    };

    // 노드 확장 핸들러 (Lazy Loading)
    const onExpandItemHandler = (item) => {
        const node = treeData.data[item.index];
        if (node.children.length > 0 && !treeData.data[node.children[0]]) {
            // 자식 노드가 아직 로드되지 않음 - Lazy Loading 필요
            props.onMissingNodesHandler(item.index + "," + node.children.join(","));
        }
        setExpandedItems([...expandedItems, item.index]);
    };

    // Drag & Drop 검증 핸들러
    const canDropAtHandler = (items, target) => {
        // dragType과 acceptDragTypes 비교
        for (const item of items) {
            const dragType = item.data.dragType;
            const acceptTypes = target.data.acceptDragTypes?.split(",");
            if (!acceptTypes?.includes(dragType)) {
                return false;
            }
        }
        return true;
    };

    // ═══════════════════════════════════════════════════════════
    // 4. 렌더링
    // ═══════════════════════════════════════════════════════════

    return (
        <ControlledTreeEnvironment
            items={treeData.data}
            getItemTitle={(item) => item.data.name}
            onSelectItems={onSelectionChangedHandler}
            onExpandItem={onExpandItemHandler}
            canDrag={canDragHandler}
            canDropAt={canDropAtHandler}
            // ... 기타 props
        >
            {/* 버튼 영역 */}
            <div className="button-container">
                <button onClick={onCollapseAllButtonClick}>...</button>
                <button onClick={onExpandAllButtonClick}>...</button>
            </div>

            {/* 트리 영역 */}
            <Tree treeId="main-tree" rootItem="root" />
        </ControlledTreeEnvironment>
    );
}
```

### 4.3 CustomIcon.jsx (아이콘 컴포넌트)

**위치:** `src/components/CustomIcon.jsx`

**역할:** 파일 확장자에 따른 아이콘 표시

**지원 파일 타입:**

| 확장자 | 아이콘 색상 | 설명 |
|--------|-------------|------|
| `.pptx` | 빨강 (#FF5733) | PowerPoint |
| `.xlsx` | 초록 (#2E7D32) | Excel |
| `.docx` | 파랑 (#2196F3) | Word |
| `.pdf` | 빨강 (#F44336) | PDF |
| 폴더 | 노랑 | 폴더 (열림/닫힘) |
| 기타 | 회색 | 기본 파일 |

**코드 예시:**

```jsx
export default function CustomIcon({ item, isExpanded }) {
    const name = item.data?.name || "";
    const extension = name.split(".").pop()?.toLowerCase();

    // 폴더인 경우
    if (item.isFolder) {
        if (item.isEmptyFolder) {
            return <EmptyFolderIcon />;
        }
        return isExpanded ? <OpenFolderIcon /> : <ClosedFolderIcon />;
    }

    // 파일 확장자별 아이콘
    switch (extension) {
        case "pptx":
            return <PptxIcon />;
        case "xlsx":
            return <XlsxIcon />;
        case "docx":
            return <DocxIcon />;
        case "pdf":
            return <PdfIcon />;
        default:
            return <DefaultFileIcon />;
    }
}
```

### 4.4 treeDataReducer.js (상태 관리)

**위치:** `src/utils/treeDataReducer.js`

**역할:** Redux 스타일의 트리 데이터 상태 관리

**액션 타입:**

| 액션 | 설명 | 사용 시점 |
|------|------|-----------|
| `reload` | 전체 트리 데이터 교체 | 초기 로드, 전체 새로고침 |
| `update` | 부분 업데이트 (추가/삭제) | Lazy Loading, 노드 변경 |
| `setDataChangedDate` | 변경 날짜 업데이트 | 서비스 재호출 트리거 |

**코드:**

```javascript
export function treeDataReducer(state, action) {
    switch (action.type) {
        case "reload":
            // 전체 데이터 교체
            return {
                data: action.data,
                dataChangedDate: new Date()
            };

        case "update":
            // 기존 데이터에 새 노드 병합
            const newData = { ...state.data, ...action.data };

            // 삭제된 노드 제거
            if (action.deletedNodeIDs) {
                action.deletedNodeIDs.split(",").forEach(id => {
                    delete newData[id];
                });
            }

            return {
                data: newData,
                dataChangedDate: new Date()
            };

        case "setDataChangedDate":
            return {
                ...state,
                dataChangedDate: action.date
            };

        default:
            return state;
    }
}
```

---

## 5. 비즈니스 로직 및 데이터 흐름

### 5.1 전체 데이터 흐름

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Mendix Application                            │
│  ┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐  │
│  │  Mendix     │────▶│  REST Service    │────▶│  Database       │  │
│  │  Page       │     │  (/rest/...)     │     │                 │  │
│  └─────────────┘     └──────────────────┘     └─────────────────┘  │
│         │                     ▲                                      │
│         ▼                     │                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              ReactComplexTreeWidget                          │   │
│  │  ┌─────────────┐    ┌────────────────┐    ┌──────────────┐  │   │
│  │  │ Main Widget │───▶│ TreeContainer  │───▶│ CustomIcon   │  │   │
│  │  │   (JSX)     │    │    (JSX)       │    │   (JSX)      │  │   │
│  │  └─────────────┘    └────────────────┘    └──────────────┘  │   │
│  │                            │                                  │   │
│  │                            ▼                                  │   │
│  │                     ┌────────────────┐                       │   │
│  │                     │ treeDataReducer│                       │   │
│  │                     │    (State)     │                       │   │
│  │                     └────────────────┘                       │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### 5.2 초기화 흐름

```
┌──────┐      ┌──────────────┐      ┌───────────────┐      ┌──────────┐
│Mendix│      │Main Widget   │      │TreeContainer  │      │REST      │
│Page  │      │              │      │               │      │Service   │
└──┬───┘      └──────┬───────┘      └───────┬───────┘      └────┬─────┘
   │                 │                      │                   │
   │ 1. Widget 로드   │                      │                   │
   │────────────────▶│                      │                   │
   │                 │                      │                   │
   │                 │ 2. Props 전달        │                   │
   │                 │─────────────────────▶│                   │
   │                 │                      │                   │
   │                 │                      │ 3. useEffect 실행 │
   │                 │                      │   (dataChangeDateAttr 변경 감지)
   │                 │                      │                   │
   │                 │                      │ 4. fetch() 호출   │
   │                 │                      │──────────────────▶│
   │                 │                      │                   │
   │                 │                      │ 5. JSON 응답      │
   │                 │                      │◀──────────────────│
   │                 │                      │                   │
   │                 │                      │ 6. dispatch(reload)
   │                 │                      │   - 상태 업데이트  │
   │                 │                      │                   │
   │                 │                      │ 7. 트리 렌더링    │
   │◀─────────────────────────────────────────                  │
```

### 5.3 REST 서비스 응답 포맷

**서버에서 반환해야 하는 JSON 구조:**

```json
{
    "action": "reload | update | focus | none",
    "nodes": [
        {
            "index": "node-1",
            "name": "문서 폴더",
            "children": "node-2,node-3,node-4",
            "isFolder": true,
            "canMove": true,
            "canRename": true,
            "parentID": "root",
            "dragType": "folder",
            "acceptDragTypes": "file,folder"
        },
        {
            "index": "node-2",
            "name": "보고서.xlsx",
            "children": "",
            "isFolder": false,
            "canMove": true,
            "canRename": true,
            "parentID": "node-1",
            "dragType": "file",
            "acceptDragTypes": ""
        }
    ],
    "deletedNodeIDs": "old-node-1,old-node-2",
    "focusNodeID": "node-1",
    "expandItemIDs": "node-1,node-3",
    "resetExpandedItems": false
}
```

**필드 설명:**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `action` | string | O | 처리 방식 (reload/update/focus/none) |
| `nodes` | array | O | 노드 배열 |
| `nodes[].index` | string | O | 노드 고유 ID |
| `nodes[].name` | string | O | 표시 이름 |
| `nodes[].children` | string | O | 자식 노드 ID (쉼표 구분) |
| `nodes[].isFolder` | boolean | O | 폴더 여부 |
| `nodes[].canMove` | boolean | O | 이동 가능 여부 |
| `nodes[].canRename` | boolean | O | 이름 변경 가능 여부 |
| `nodes[].parentID` | string | O | 부모 노드 ID |
| `nodes[].dragType` | string | X | 드래그 타입 |
| `nodes[].acceptDragTypes` | string | X | 허용 드래그 타입 (쉼표 구분) |
| `deletedNodeIDs` | string | X | 삭제할 노드 ID (쉼표 구분) |
| `focusNodeID` | string | X | 포커스할 노드 ID |
| `expandItemIDs` | string | X | 확장할 노드 ID (쉼표 구분) |
| `resetExpandedItems` | boolean | X | 확장 상태 초기화 여부 |

### 5.4 내부 트리 데이터 구조

```javascript
// treeData.data 구조
{
    "root": {
        index: "root",
        isFolder: true,
        isEmptyFolder: false,
        canMove: false,
        canRename: false,
        children: ["node-1", "node-2"],
        data: {
            name: "Root",
            parentID: null,
            dragType: null,
            acceptDragTypes: "folder,file"
        }
    },
    "node-1": {
        index: "node-1",
        isFolder: true,
        isEmptyFolder: false,
        canMove: true,
        canRename: true,
        children: ["node-3", "node-4"],
        data: {
            name: "문서 폴더",
            parentID: "root",
            dragType: "folder",
            acceptDragTypes: "file"
        }
    }
    // ... 추가 노드들
}
```

### 5.5 주요 이벤트 흐름

#### 5.5.1 노드 선택 이벤트

```
[사용자 클릭]
    │
    ▼
TreeContainer.onSelectItems()
    │
    ├─▶ setSelectedItems(items)     // 로컬 상태 업데이트
    │
    └─▶ props.onSelectionChangedHandler(items.join(","))
            │
            ▼
    MainWidget.onSelectionChangedHandler()
            │
            ├─▶ selectedNodeIDsAttr.setValue()  // Mendix 속성에 저장
            │
            └─▶ onSelectionChangedAction?.execute()  // Microflow 실행
```

#### 5.5.2 Lazy Loading 이벤트

```
[노드 확장 클릭]
    │
    ▼
TreeContainer.onExpandItemHandler()
    │
    ├─▶ 자식 노드가 로드됨? ──Yes──▶ setExpandedItems() // 단순 확장
    │
    └─▶ No (미로드)
            │
            ▼
    props.onMissingNodesHandler(nodeIDs)
            │
            ▼
    MainWidget.onMissingNodesHandler()
            │
            ├─▶ missingNodeIDsAttr.setValue()
            │
            └─▶ onMissingNodesAction?.execute()  // Microflow 실행
                    │                              (서버에서 데이터 로드)
                    ▼
            dataChangeDateAttr 변경
                    │
                    ▼
            useEffect 트리거 → REST 서비스 재호출
                    │
                    ▼
            dispatch({ type: "update", data: newNodes })
```

#### 5.5.3 Drag & Drop 이벤트

```
[드래그 시작]
    │
    ▼
canDragHandler(items)
    │
    └─▶ items.every(item => item.canMove)  // 이동 가능 여부 확인
            │
            ▼
[드래그 중 - 드롭 대상 위]
    │
    ▼
canDropAtHandler(items, target)
    │
    └─▶ dragType이 acceptDragTypes에 포함? // 드롭 가능 여부 확인
            │
            ▼
[드롭 실행]
    │
    ▼
onDropHandler(items, target)
    │
    ├─▶ draggedNodeIDsAttr.setValue()   // 드래그한 노드 ID
    │
    ├─▶ dropTargetAttr.setValue()       // 드롭 대상 정보 (JSON)
    │
    └─▶ onDropAction?.execute()         // Microflow 실행
```

**dropTargetAttr JSON 구조:**

```json
{
    "targetType": "item | between-items | root",
    "parentItem": "node-1",
    "childIndex": 2,
    "linearIndex": 5
}
```

---

## 6. 위젯 설정 옵션

### 6.1 속성 그룹 구조

`ReactComplexTreeWidget.xml`에 정의된 속성 그룹:

```xml
<propertyGroup caption="Service call">
    <!-- 서비스 호출 관련 -->
</propertyGroup>

<propertyGroup caption="Collapse all button">
    <!-- 전체 축소 버튼 -->
</propertyGroup>

<propertyGroup caption="Root button">
    <!-- 루트 버튼 -->
</propertyGroup>

<propertyGroup caption="Expand all button">
    <!-- 전체 확장 버튼 -->
</propertyGroup>

<propertyGroup caption="Configuration">
    <!-- 트리 동작 설정 -->
</propertyGroup>

<propertyGroup caption="Rename node">
    <!-- 노드 이름 변경 -->
</propertyGroup>

<propertyGroup caption="Events">
    <!-- 이벤트 핸들링 -->
</propertyGroup>

<propertyGroup caption="Drag and drop">
    <!-- Drag & Drop 설정 -->
</propertyGroup>

<propertyGroup caption="Advanced">
    <!-- 고급 설정 -->
</propertyGroup>
```

### 6.2 상세 속성 설명

#### Service Call 속성

| 속성명 | 타입 | 설명 |
|--------|------|------|
| `dataChangeDateAttr` | DateTime | 이 값이 변경되면 서비스 재호출 |
| `serviceUrl` | Expression (String) | REST 엔드포인트 URL (반드시 `/rest/`로 시작) |

#### Configuration 속성

| 속성명 | 타입 | 기본값 | 설명 |
|--------|------|--------|------|
| `toggleExpandedIconOnly` | Boolean | false | true: 화살표 클릭시만 확장<br>false: 노드 클릭시 확장 |

#### Rename Node 속성

| 속성명 | 타입 | 설명 |
|--------|------|------|
| `allowNodeRename` | Boolean | 이름 변경 기능 활성화 |
| `renamedNodeIDAttr` | String | 변경된 노드 ID 저장 속성 |
| `newNodeNameAttr` | String | 새 이름 저장 속성 |
| `onNodeRenamedAction` | Action | 이름 변경 후 실행할 Microflow |

#### Events 속성

| 속성명 | 타입 | 설명 |
|--------|------|------|
| `selectedNodeIDsAttr` | String | 선택된 노드 ID들 (쉼표 구분) |
| `onSelectionChangedAction` | Action | 선택 변경 시 실행할 Microflow |
| `missingNodeIDsAttr` | String | Lazy Loading에 필요한 노드 ID들 |
| `onMissingNodesAction` | Action | Lazy Loading 시 실행할 Microflow |

#### Drag & Drop 속성

| 속성명 | 타입 | 설명 |
|--------|------|------|
| `allowDragReordering` | Boolean | 같은 부모 내 순서 변경 허용 |
| `allowDragMove` | Boolean | 다른 위치로 이동 허용 |
| `draggedNodeIDsAttr` | String | 드래그한 노드 ID들 |
| `dropTargetAttr` | String | 드롭 대상 정보 (JSON) |
| `onDropAction` | Action | 드롭 후 실행할 Microflow |

#### Advanced 속성

| 속성명 | 타입 | 설명 |
|--------|------|------|
| `logToConsole` | Boolean | 콘솔 로깅 활성화 |
| `dumpServiceResponseInConsole` | Boolean | 서비스 응답 콘솔 출력 |

### 6.3 editorConfig.js 검증 규칙

에디터에서 속성 검증을 수행하는 로직:

```javascript
export function check(values) {
    const errors = [];

    // 이름 변경 기능 활성화 시 필수 속성 검증
    if (values.allowNodeRename) {
        if (!values.renamedNodeIDAttr) {
            errors.push({
                property: "renamedNodeIDAttr",
                message: "Renamed node ID attribute is required"
            });
        }
        if (!values.newNodeNameAttr) {
            errors.push({
                property: "newNodeNameAttr",
                message: "New node name attribute is required"
            });
        }
        if (!values.onNodeRenamedAction) {
            errors.push({
                property: "onNodeRenamedAction",
                message: "On node renamed action is required"
            });
        }
    }

    // Drag & Drop 기능 활성화 시 필수 속성 검증
    if (values.allowDragReordering || values.allowDragMove) {
        if (!values.draggedNodeIDsAttr) {
            errors.push({
                property: "draggedNodeIDsAttr",
                message: "Dragged node IDs attribute is required"
            });
        }
        // ... 추가 검증
    }

    return errors;
}
```

---

## 7. 개발 가이드라인

### 7.1 코드 스타일

**ESLint 설정 (`.eslintrc.js`):**
```javascript
module.exports = {
    extends: ["@mendix/pluggable-widgets-tools/configs/eslint"]
};
```

**Prettier 설정 (`prettier.config.js`):**
```javascript
module.exports = {
    ...require("@mendix/pluggable-widgets-tools/configs/prettier"),
    plugins: ["@prettier/plugin-xml"]
};
```

### 7.2 컴포넌트 작성 규칙

**1. 함수형 컴포넌트 사용:**
```jsx
// Good
export default function MyComponent(props) {
    return <div>...</div>;
}

// Avoid
class MyComponent extends React.Component {
    render() { return <div>...</div>; }
}
```

**2. Props 검증:**
```jsx
// Props 타입은 Mendix XML에서 정의
// 런타임에서 read-only 검증 수행
if (props.myAttribute && !props.myAttribute.readOnly) {
    props.myAttribute.setValue(value);
}
```

**3. 상태 관리:**
```jsx
// 단순 상태: useState
const [selectedItems, setSelectedItems] = useState([]);

// 복잡한 상태: useReducer
const [treeData, dispatch] = useReducer(treeDataReducer, initialState);
```

### 7.3 서비스 호출 패턴

**CSRF 토큰 포함:**
```jsx
const response = await fetch(serviceUrl, {
    headers: {
        "X-Csrf-Token": mx.session.getConfig("csrftoken")
    }
});
```

**Lazy Loading 파라미터:**
```jsx
const url = triggerReload
    ? `${serviceUrl}?fullreload=true`  // 전체 리로드
    : serviceUrl;                       // 일반 호출
```

### 7.4 이벤트 핸들러 패턴

**Mendix 속성 안전하게 설정:**
```jsx
const onSelectionChangedHandler = (nodeIDs) => {
    // 1. 로깅 (디버그용)
    if (logToConsole) {
        console.log("Selection changed:", nodeIDs);
    }

    // 2. 속성 검증 후 설정
    if (selectedNodeIDsAttr && !selectedNodeIDsAttr.readOnly) {
        selectedNodeIDsAttr.setValue(nodeIDs);
    }

    // 3. 액션 실행
    if (onSelectionChangedAction && onSelectionChangedAction.canExecute) {
        onSelectionChangedAction.execute();
    }
};
```

### 7.5 디버깅 팁

**콘솔 로깅 활성화:**
```jsx
// props.logToConsole이 true일 때만 로깅
if (props.logToConsole) {
    console.log("ReactComplexTreeWidget:", message);
    console.log("Performance:", Date.now() - startTime, "ms");
}

// 서비스 응답 덤프
if (props.dumpServiceResponseInConsole) {
    console.log("Service response:", JSON.stringify(result, null, 2));
}
```

---

## 8. 빌드 및 배포

### 8.1 npm 스크립트

| 명령어 | 설명 |
|--------|------|
| `npm start` | Watch 모드 개발 |
| `npm run dev` | 웹 개발 서버 시작 |
| `npm run build` | 프로덕션 빌드 |
| `npm run lint` | ESLint 검사 |
| `npm run lint:fix` | ESLint 자동 수정 |
| `npm run release` | 버전 릴리즈 |

### 8.2 빌드 출력

빌드 실행 시 생성되는 파일:

```
dist/
├── 1.3.0/
│   └── aiden.ReactComplexTreeWidget.mpk    # Mendix 패키지 파일
└── tmp/
    └── widgets/
        ├── aiden/reactcomplextreewidget/
        │   ├── ReactComplexTreeWidget.css   # 스타일
        │   ├── ReactComplexTreeWidget.js    # CommonJS 번들
        │   └── ReactComplexTreeWidget.mjs   # ES Module 번들
        ├── package.xml
        └── ReactComplexTreeWidget.xml
```

### 8.3 Mendix에 배포하기

1. **빌드 실행:**
   ```bash
   npm run build
   ```

2. **mpk 파일 복사:**
   ```
   dist/1.3.0/aiden.ReactComplexTreeWidget.mpk
   ```

3. **Mendix 프로젝트에 추가:**
   - `{MendixProject}/widgets/` 폴더에 mpk 파일 복사
   - Mendix Studio Pro에서 F4 (Synchronize Project Directory)
   - 페이지에서 위젯 사용

### 8.4 버전 업데이트

`package.json`에서 버전 변경:

```json
{
    "name": "reactcomplextreewidget",
    "version": "1.4.0",  // 버전 업데이트
    ...
}
```

---

## 9. 트러블슈팅

### 9.1 일반적인 문제

#### 트리가 표시되지 않음

**원인:** 서비스 응답 포맷 오류

**해결:**
1. `dumpServiceResponseInConsole` 활성화
2. 브라우저 콘솔에서 응답 확인
3. JSON 포맷 검증

```javascript
// 올바른 응답 구조인지 확인
{
    "action": "reload",
    "nodes": [...]  // 배열이어야 함
}
```

#### Lazy Loading이 작동하지 않음

**원인:** `onMissingNodesAction` 미설정

**해결:**
1. 위젯 속성에서 `missingNodeIDsAttr` 설정
2. `onMissingNodesAction` Microflow 연결
3. Microflow에서 `dataChangeDateAttr` 업데이트

#### Drag & Drop 불가

**원인:** `dragType`/`acceptDragTypes` 불일치

**해결:**
```json
// 드래그하는 노드
{ "dragType": "file" }

// 드롭 대상 노드
{ "acceptDragTypes": "file,folder" }  // "file" 포함해야 함
```

### 9.2 성능 최적화

**대용량 트리:**
- Lazy Loading 필수 사용
- 한 번에 로드하는 노드 수 제한 (100개 이하 권장)
- `update` 액션으로 부분 업데이트

**렌더링 최적화:**
- `toggleExpandedIconOnly: true` 설정
- 불필요한 아이콘 렌더링 제거

### 9.3 디버깅 체크리스트

```
□ 브라우저 콘솔에 에러가 있는가?
□ 네트워크 탭에서 REST 호출이 성공하는가?
□ 서비스 응답이 올바른 JSON 포맷인가?
□ 필수 속성이 모두 설정되어 있는가?
□ Mendix 속성이 read-only가 아닌가?
□ Microflow가 올바르게 연결되어 있는가?
```

---

## 부록 A: 참고 자료

### 외부 링크

- [react-complex-tree 문서](https://github.com/lukasbach/react-complex-tree)
- [Mendix Pluggable Widgets 문서](https://docs.mendix.com/apidocs-mxsdk/apidocs/pluggable-widgets/)
- [React 공식 문서](https://react.dev/)

### 파일 위치 요약

| 파일 | 위치 |
|------|------|
| 메인 위젯 | `src/ReactComplexTreeWidget.jsx` |
| 트리 컨테이너 | `src/components/TreeContainer.jsx` |
| 커스텀 아이콘 | `src/components/CustomIcon.jsx` |
| 상태 리듀서 | `src/utils/treeDataReducer.js` |
| 위젯 설정 | `src/ReactComplexTreeWidget.xml` |
| 에디터 설정 | `src/ReactComplexTreeWidget.editorConfig.js` |

---

**문서 버전:** 1.0
**최종 수정:** 2026-01-15
**대상:** ReactComplexTreeWidget v1.3.0

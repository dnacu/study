# 브라우저 렌더링 과정

1. HTML 파싱 -> DOM 트리 구성
2. CSS 파싱 -> CSSOM 트리 구성
3. DOM 트리 + CSSOM 트리 -> Render 트리 구성
4. Layout
5. Update Layer Tree
6. Painting
7. Composite Layer

## reflow, repaint

**reflow**란, 요소들의 레이아웃에 영향을 주는 변화  
**repaint**란, 레이아웃에 영향을 주지 않는 엘리먼트 개별의 스타일 속성 변화

## Layer

레이어를 나눔으로써 하나의 변화에 대해 영향을 받는 엘리먼트들을 찾고, 레이아웃을 계산하는 과정을 줄임. `will-change`나 `translate3d`를 통해 레이어를 분리할 수 있으나 각 레이어를 유지하는데에는 리소스가 들기 때문에 남발하면 안된다.

---

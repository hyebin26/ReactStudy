#Virtual DOM

### 브라우저의 WorkFlow

1. DOM Tree생성 

브라우저가 HTML을 전달받으면, 브라우저의 렌더 엔진이 이를 파싱하고 DOM노드(Node)로 이뤄진 트리를 만들어준다. 각 노드는 각 HTML 엘리먼트들과 연관이 있다.

  2.  Render Tree 생성

그리고 외부 css 파일과 각 엘리먼트들의 inline스타일을 파싱한다. 스타일 정보를 사용하여 DOM트리에 따라 새로운 트리, 렌더트리를 만든다.

  3.  Render Tree 생성- 뒤에서 일어나는 일 

Webkit에서는 노드의 스타일을 처리하는 과정을 "attachment"라고 부른다. DOM트리의 모든 노드들은 attach라는 메소드가 있고 스타일 정보를 계산해서 객체형태로 반환시켜준다.

  4.  Layout(reflow)

렌더 트리가 다 만들어지고 나면 , 레아아웃 과정을 거치고 정확히 어디에 나타나야 할 지 위치가 주어진다

  5.  Painting 

렌더링 된 요소들에 색을 입히는 과정, 트리의 각 노드들을 거쳐가면서 paint()메소드를 호출

---

### Visual DOM

DOM조작의 실제 문제는 각 조직이 레이아웃 변화, 트리변화와 렌더링을 일으킨다는 것이다. 예를 들어 30 개의 노드를 하나 하나 수정하면 그 뜻은 30번의 (잠재적인)레이아웃 재계산과 30번의 (잠재적인)리렌더링을 초래한다는 것이다.

Virtual DOM은 변화가 일어나면 그걸 Virtual DOM트리에 적용시키고 이 DOM트리는 렌더링도 되지 않기 떄문에 연산 비용이 적다. 연산이 끝나면 그 최종적인 변화를 실제 DOM에 던져주는 것이다. 이 DOM트리는 렌더링도 되지 않기 때문에 연산비용이 적다.

이 과정은 Virtual DOM없이도 DOM fragment를 적용하면 위에 과정은 이뤄질 수 있다. Virtual DOM은 DOM fragment처럼 수동으로 관리하는 것이 아니라 자동화하고 추상화하는 것이다.

⇒ 리액트는 DOM보다 빠르지 않다. 최적화 작업을 손수했을 때와 리액트를 사용했을 때를 비교한다면 대부분의 경우 전자가 더 빠르다.

⇒리액트는 DOM보다 빠르지 않고 유지보수가 가능한 어플리케이션만드는 것을 도와주고 그리고 대부분의 경우에 충분히 빠르다.

<a href="https://velopert.com/3236">참고:https://velopert.com/3236</a>
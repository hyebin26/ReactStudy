## Reducers and Immutable Updates

리덕스에 가장 중요한 규칙은 우리의 리듀서는 절대 기존/현재의 state value 복제하는 것을 허락하지 않는다.

```jsx
state.value = 123 // X
```

리덕스에서 state를 안되는 이유는 여러가지가 있다.

- 버그를 발생시킨다. ex) UI가 가장 최근의 value 보여주는 것을 적절하게 업데이트 하지 않을 수 있다.
- 어떻게, 왜 상태가 업그레이드 되었는지 이해하는 것을 어렵게 만든다.
- test하기 어렵게 만든다.
- "time- travel debugging"사용하는 것을 못하게 한다.
- 리덕스에 기본 패턴 그리고 의도된 것에 반대된다.

## Immutable Updates with Immer

Immer은 불변 업데이트 로직의 처리과정을 간편하게 하는 라이브러리이다.

Immer는 원래의 state와 콜백함수 두개의 인자를 받는 함수 produce를 제공한다.   콜백 함수는 그 state에 "draft" 버전으로 주고. 이것은 그 draft value를 복제하는 코드를 작성하는 것은 안전하다. Immer는 draft value를 변경하려는 모든 시도를 추적 한 다음, 안전한 결과를 불변 등가물을 사용하여 복제한다.

## Redux Toolkit and Immer

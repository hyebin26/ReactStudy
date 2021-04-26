## 불변성

리액트에서 불변성을 지켜야 하는 이유는 무엇일까?

리액트가 리렌더링할 때 성능을 위해 얕은 비교를 통해 값이 다르면 리렌더링을 한다.

```jsx
export default function App() {
  const [hyebin, setHyebin] = useState({ name: "hyebin", age: 27 });
  const handleClick = () => {
    hyebin.age += 1;
    setHyebin(hyebin);
  };
  const handleImmuClick = () => {
    setHyebin((state) => ({
      ...state,
      age: state.age + 1
    }));
  };
  return (
    <div>
      <button onClick={handleClick}>떡국</button>
      <button onClick={handleImmuClick}>불변 떡국</button>
      <hr />
      <h2>{hyebin.age}</h2>
    </div>
  );
}
```

예제를 보면 "떡국"을 클릭했을 때는 h2태그에 있는 hyebin.age의 값이 바뀌지 않습니다. 

이유는 리액트에서 얕은 비교를 할 때 원시타입이 아닌 타입은 메모리에 있는 값을 참조하기 때문에 객체에 있는 값이 변경되도 같은 참조값을 유지합니다. 

즉, hyebin.age의 값이 변경되어도 같은 값을 참조하므로, 리렌더링 되지 않습니다. 하지만 handleImmuClick처럼 불변성을 유지해서 새로운 객체를 만들면 같은 값을 참조하지 않기 때문에 리액트가 리렌더링 하게 됩니다.

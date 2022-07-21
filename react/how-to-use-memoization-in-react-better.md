# 효율적인, 효과적인 메모이제이션 in ReactJS

> 메모이제이션(memoization)은 컴퓨터 프로그램이 동일한 계산을 반복해야 할 때, 이전에 계산한 값을 메모리에 저장함으로써 동일한 계산의 반복 수행을 제거하여 프로그램 실행 속도를 빠르게 하는 기술이다. 동적 계획법의 핵심이 되는 기술이다. 메모이제이션이라고도 한다. (- Wikipedia)

React에서 제공하며 함수형 React 컴포넌트를 사용할 때, 주로 우리가 사용하는 메모이제이션 기법은 다음의 세 가지 함수를 사용하여 구현할 수 있습니다.

- `React.memo`
- `React.useMemo`
- `React.useCallback`

1. `React.memo`는 이전에 렌더링한 `Props`와 새로 렌더링할 때의 `Props`가 같다면 이전 렌더링에서 메모이징한 렌더링 결과를 재사용하여 리렌더링을 건너뜁니다. `Props`가 자주 변하지 않거나 렌더링에 많은 리소스가 필요한 컴포넌트에 사용하기 적합합니다.
2. `React.useMemo`는 두 개의 인자를 받습니다. 첫 번째 인자는 함수이며 두번째 인자는 배열입니다. 첫 번째 인자 함수에서 반환하는 값을 `useMemo`의 결과값으로 반환합니다. 단, 두 번째 인자 배열 속 요소들이 이전에 실행했을 때와 같다면 첫 번째 인자 함수를 다시 실행하지 않고, 이전에 실행했던 메모이징된 값을 재사용하여 `useMemo`의 결과값으로 반환합니다.
3. `React.useCallback`도 두 개의 인자를 받습니다. 역시 첫 번째 인자는 함수이며 두 번째 인자는 배열입니다. `useMemo`와 같이 두 번째 인자 배열 속 요소들이 변하지 않았다면 메모이징된 값을 재사용하는 것은 동일하지만, 함수를 실행한 결과값이 아닌 함수 자체를 메모이징합니다.

위에서 언급한 세 가지 함수에서 공통적으로 언급되는 "같다"는 실제로는 "얕은 엄격한 비교"를 사용합니다. 얕은 엄격한 비교는 얕은 비교이면서 엄격한 비교라는 뜻인데, 실제 코드에서는 `===`으로 표현됩니다.

## 엄격한 비교

엄격한 비교가 있다면 엄격하지 않은 비교도 있을까요? 이를 자바스크립트에서는 단순히 "비교" 또는 느슨한 비교라고 하며 비교 연산자 `==`으로 표현합니다. (이 결과값의 부정은 `!=`으로 표현합니다.) 느슨한 비교는 서로 비교하는 값의 타입이 다르다면 타입을 같게 변환하려고 시도하고 변환된 값을 비교합니다. 예를 들어 문자열 `"1"`과 숫자 `1`을 느슨한 비교(`"1" == 1`) 했을 때, 양변의 값을 문자열로 변환한 후에 두 값이 같은지 확인하는데, 이때 변환된 두 값은 문자열 `"1"`로 동일하므로 연산의 결과는 참이 됩니다.

```js
"1" == 1; // true
1 == "1"; // true
0 == false; // true
0 == null; // false
0 == undefined; // false
0 == !!null; // true
0 == !!undefined; // true
null == undefined; // true
```

느슨한 비교와 다르게 엄격한 비교는 엄격합니다. 자바스크립트에서 엄격한 비교는 엄격한 비교 연산자 `===`으로 표현합니다. (이 결과값의 부정은 `!==`으로 표현합니다.) 엄격한 비교는 이름에서 볼 수 있듯 엄격한데, 만약 비교하려는 두 값의 타입이 다르다면 연산의 결과는 거짓이 됩니다. 만약 같은 타입일 때만 두 값이 같은지 확인합니다.

```js
console.log("hello" === "hello"); // true
console.log("hello" === "hola"); // false

console.log(3 === 3); // true
console.log(3 === 4); // false

console.log(true === true); // true
console.log(true === false); // false

console.log(null === null); // true

console.log("3" === 3); // false

console.log(true === 1); // false

console.log(null === undefined); // false
```

그렇다면 다음 코드를 실행했을 때 어떤 값이 출력될까요?

```js
console.log([1, 2, 3] === [1, 2, 3]);
```

쨔잔, `false`입니다. 양변의 타입이 같으며 값이 같은데 왜그럴까요?
이는 두 배열을 비교할 때는 배열이 참조하는 주소가 같은지 비교하는 데,
두 배열을 생성할 때 서로 다른 주소를 포인팅하게 되기 때문입니다.
(배열 리터럴(`[]`)을 사용할 때 마다 새로운 배열이 생성된다고 이해하면 편합니다.)
이때문에 아래와 같이 같은 주소를 참조하는 배열을 비교했을 때의 연산값은 참이 됩니다.

```js
var a = [];
var b = a;

console.log(a === b); // true
```

이와 같은 특징은 **함수, 객체, 배열** 등에서 나타납니다.

## 얕은 비교와 깊은 비교

`[1, 2, 3] === [1, 2, 3]`은 거짓입니다.
근데 우리가 두 배열의 주소가 다르더라도 요소가 전부 같은지 확인하고 싶다면 어떻게 해야할까요?
이때 필요한 것이 깊은 비교이며 이와 다르게 원시 값 또는 주소만 비교하는 것을 얕은 비교라고 합니다.
배열간의 깊은 비교는 다음과 같이 구현해볼 수 있습니다.

```js
function deepCompareArray(a1, a2) {
  for (let index = 0; index < a1.length; index++) {
    // 배열의 모든 값을 순회하며 같은 인덱스를 같는 요소가 모두 같은지 확인합니다.
    // 한번이라도 다르면 false를 리턴합니다.
    if (a1[index] != a2[index]) {
      return false;
    }
  }

  // 모두 같아서 for loop을 탈출했다면 true를 리턴합니다.
  return true;
}
```

이 문서에서는 배열 속 배열 또는 배열 속 객체 등 깊이가 큰 깊은 비교에 대해서는 다루지 않습니다.
[여기를 참고](https://stackoverflow.com/questions/1068834/object-comparison-in-javascript)하여 깊이가 큰 깊은 비교에 대하여 알아볼 수 있습니다.

## React 메모이제이션 함수들은 얕은 엄격한 비교를 한다.

본론으로 돌아와서, 위에서 언급한 React의 세 가지 메모이제이션 함수들은 이전의 값과 현재의 값을 비교할 때 얕은 엄격한 비교를 합니다.
아래와 같이 컴포넌트를 메모이제이션 했다고 가정해봅시다.

```jsx
import { memo } from "react";

const CardImage = ({ cardInfo }) => {
  return (
    <div>
      <h2>{cardInfo.title}</h2>
      <p>{cardInfo.description}</p>
      <img src="/card-image-.png" />
    </div>
  );
};

export default memo(CardImage);
```

그런데 아래와 같이 컴포넌트를 사용했다면 어떨까요?

```jsx
<CardImage
  cardInfo={{
    title: "Flower",
    description: "Color is red.",
  }}
/>
```

실행할 때마다 새로운 `cardInfo` 객체를 생성하기 때문에 이전 렌더링할 때의 `cardInfo`와 새롭게 렌더링할 때의 `cardInfo` 값은 다르므로 `React.memo`를 사용하였지만 의미가 사라져버렸습니다.

이는 두가지 방법으로 해결할 수 있습니다.
첫번째 방법은 `React.memo`의 두번째 인자에 두 Props가 같은지 비교하는 함수를 전달해주는 방법입니다.

```jsx
export default memo(CardImage, (prevProps, nextProps) => {
  // prevProps는 이전 렌더링 때의 props입니다.
  // nextProps는 새로운 렌더링 때의 props입니다.
  return (
    prevProps.cardInfo.title === nextProps.cardInfo.title &&
    prevProps.cardInfo.description === nextProps.cardInfo.description
  );
});
```

두번째 방법은 전달하는 값을 메모이제이션하는 것 입니다.
`useMemo`에 의해 메모이제이션 된 값은 주소도 같기 때문에 이렇게 구현할 수 있습니다.

```jsx
const cardInfo = useMemo(
  () => ({
    title: "Flower",
    description: "Color is red.",
  }),
  []
);

<CardImage cardInfo={cardInfo} />;
```

이러한 특징은 함수, 배열, 객체 등 얕은 비교를 했을 때 주소를 비교하는 값들을 `Props`로 사용했을 때 공통적으로 발생하는 문제입니다.

## 과도한 메모이제이션은 해롭다.

아래와 같이 `useMemo`를 활용했다고 가정해보겠습니다.

```js
const dataExists = useMemo(() => {
  return !loading && data.length != 0;
}, [loading, data.length]);
```

`!loading && data.length != 0;`와 같은 간단한 연산에 `useMemo`를 사용하였습니다.
게다가 결과값은 `true` 혹은 `false`여서 얕은 엄격한 비교의 특징을 고려할 필요도 없습니다.
`useMemo`는 이전의 값을 저장하고, 두번째 인자 배열의 값을 비교하는 등 여러 작업을 통해 메모이제이션을 합니다.
이 과정에서도 연산 리소스가 필요한데, 저렇게 간단한 구문을 메모이징 하는 것은
의미도 적을 뿐더러 `useMemo`를 쓰기 위한 리소스가 불필요하게 추가로 소요됩니다.
결과적으로, 코드를 이해하는 데 난해해지고 성능은 미약하지만 더 안좋아질 것 입니다.
그래서 아래와 같이 단순하게 코드를 수정할 수 있습니다.

```js
const dataExists = !loading && data.length != 0;
```

> Keep it simple, Stupid. (1960년에 미국 해군이 고안한 디자인 원리, KISS 원칙)

물론, 아래의 코드처럼 정말 많은 연산이 필요할 경우에는 충분히 메모이제이션을 사용할 필요가 있습니다.

```js
function factorial(n) {
  if (n == 0 || n == 1) return n;
  return n * factorial(n - 1);
}

const bigNumber = useMemo(() => factorial(500), []);
```

## 이벤트 핸들러도 함수라는 것을 잊지 말자.

```jsx
<div onClick={() => alert("클릭하셨네요?")} />
```

`div` 컴포넌트를 포함한 DOM 컴포넌트의 대부분은 이벤트 핸들러를 위와 같은 형식처럼 함수로 작성합니다.
그런데 `react-dom` 내부적으로도 저 함수의 변화를 리렌더링할 때마다 감지하고 만약 변했다면 이전에 있던 이벤트 핸들러를 unsubscribe하고 새로운 함수를 다시 subscribe합니다.
위의 예시에서는 함수의 실제 로직은 변하지 않지만, 단지 다시 렌더링 될 때 함수가 다시 생성된다는 이유로 비교 연산을 했을 떄 거짓을 결과로 갖습니다.
이러한 불필요한 연산을 줄이기 위해 `useCallback`이 필요합니다.

```jsx
const onClick = useCallback(() => alert("클릭하셨네요?"), []);
return <div onClick={onClick} />;
```

만약 함수안에서 사용하는 변수가 있다면 아래의 코드처럼 그 변수들이 변했을 때만 함수를 새로 생성할 수 있습니다.

```jsx
const [i, setI] = useState(0);
const onClick = useCallback(() => {
  alert(i + "번 클릭하셨네요?");

  // setI는 useState의 반환값 중 하나인데,
  // 이 함수는 변하지 않기 때문에 배열에 넣지 않아도 된다.
}, [i]);

return <div onClick={onClick} />;
```

### 참고자료

- [위키피디아 - 메모이제이션](https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EC%9D%B4%EC%A0%9C%EC%9D%B4%EC%85%98)
- [Toast UI - React.memo() 현명하게 사용하기](https://ui.toast.com/weekly-pick/ko_20190731)
- [mdn web docs - Equality](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Equality)
- [mdn web docs - Strict Equality](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Strict_equality)
- [stack overflow - Why is [] !== [] in JavaScript?](https://stackoverflow.com/questions/40313263/why-is-in-javascript)

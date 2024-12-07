---
title: '리액트 Hook 과 유틸 함수의 차이'
description: '재사용성을 위해 자주 분리시켰던 유틸함수와 훅. 이 둘의 차이는 무엇일까요?'
publishDate: '27 Oct 2024'
coverImage:
  src: 'cover.png'
  alt: 'React post wallpaper'
tags: ['JavaScript', 'React', 'Hook', 'function', 'utils']
ogImage: '/images/react-cover.png'
---

<!-- # React hook 과 유틸 함수의 차이 -->

React에서는 기본으로 제공되는 **Built-in Hook** 과 사용자가 직접 만들 수 있는 **Custom Hook**, 리액트를 베이스로 한 라이브러리에서 제공하는 **Hook** 등 다양한 **"Hook"** 이 등장합니다.

그렇다면 **훅이란 무엇이며, 기존 유틸 함수와 훅의 차이**는 무엇일까요?
리액트 훅의 특징에 대해 알아보며 그 차이를 살펴보겠습니다!

<br>

## [1] React 는 어떤 동작일까요?

먼저 리액트 팀에서 말하는 React 는 어떤 기능일까요?
레거시 리액트 홈페이지의 대문에 이렇게 쓰여있습니다.

> ### Declarative
>
> Design simple views for each state in your application, and React will efficiently update and render just the right components when your data changes.
>
> 애플리케이션의 각 상태에 대해 간단한 뷰를 디자인하면 데이터가 변경될 때 React가 적절한 컴포넌트를 효율적으로 업데이트하고 렌더링합니다.

<aside>

**리액트는 상태(state)에 따라 컴포넌트를 업데이트하고 렌더링 하는 동작**입니다.

</aside>

리액트 홈페이지에서는 리액트가 **반응형 UI**를 쉽게 만들기 위해 설계되었다고 설명합니다. 사용자가 상호작용할 수 있도록 만드는 것이 리액트 동작의 핵심입니다. 그렇기 때문에 리액트에서는 **상태를 관리하고, 업데이트 및 렌더링을 효율적으로 수행하는 것**이 중요한 키워드로 자리 잡습니다.

<br>

이 두가지를 가지고 Hook에 대해 살펴보도록 하겠습니다.

<br>

## [2] Hook 은 무엇일까요?

레거시 리액트 홈페이지를 한 번 더 살펴보겠습니다.

> ### But what is a Hook?
>
> Hooks are functions that let you “hook into” React state and lifecycle features from function components. Hooks don’t work inside classes — they let you use React without classes.
>
> Hook은 함수 컴포넌트에서 React 상태 및 생명주기 기능을 "hook into"할 수 있게 해주는 함수입니다.
> Hook은 클래스 내부에서 작동하지 않으며 클래스 없이 React를 사용할 수 있게 해줍니다.

정리해보자면 내용은 다음과 같습니다.

1. 클래스 없이 React를 사용할 수 있게 해준다. (Hook은 클래스 내부에서 작동하지 않는다.)
2. Hook은 함수 컴포넌트에서 React 상태 및 생명주기 기능을 "hook into"할 수 있게 해준다.

<br>

### **1. 클래스 없이 React를 사용할 수 있게 해준다: 함수 컴포넌트의 한계**

지금은 함수 컴포넌트를 주로 사용하지만, 리액트는 원래 클래스 컴포넌트로 시작되었습니다. 그렇다면 클래스가 없다면 왜 리액트의 기능을 충분히 활용하기 어려울까요? 클래스 컴포넌트를 통해 그 이유를 알아보겠습니다.

<br>

**왜 클래스 컴포넌트로 태어났을까요?**

클래스 컴포넌트는 컴포넌트의 **‘상태’를 캡슐화**하여 관리할 수 있고, 내장 메서드(`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`)로 **생명주기를 제어**할 수 있기 때문에 사용되었습니다.

**상태와 생명주기 관리라는 두 마리 토끼를 동시에 해결**한 좋은 방법이었습니다.

여기, 클래스 컴포넌트의 예시가 있습니다.
컴포넌트 내에 버튼을 누를때 마다 숫자가 증가하고 이를 화면에 표시하는 간단한 동작을 수행합니다.

```jsx
import React, { Component } from 'react'

class MyComponent extends Component {
	state = {
		count: 0
	}

	// 생명주기 메서드 (마운트 시 호출)
	componentDidMount() {
		console.log('컴포넌트가 마운트 됐습니다.')
	}

	// 생명주기 메서드 (업데이트 시 호출)
	componentDidUpdate(prevProps, prevState) {
		if (prevState.count !== this.state.count) {
			console.log('숫자를 업데이트합니다.')
		}
	}

	// 생명주기 메서드 (언마운트 시 호출)
	componentWillUnmount() {
		console.log('컴포넌트가 언마운트 됩니다.')
	}

	// 이벤트 핸들러
	handleIncrement = () => {
		this.setState((prevState) => ({ count: prevState.count + 1 }))
	}

	render() {
		return (
			<div>
				<p>Count: {this.state.count}</p>
				<button onClick={this.handleIncrement}>Increment</button>
			</div>
		)
	}
}

export default MyComponent
```

이 클래스 컴포넌트 예시에서는, 버튼을 클릭할 때마다 `setState`를 통해 **상태값이 업데이트**됩니다. 이 과정에서 `componentDidUpdate`가 호출되어 **상태 변경에 따라 컴포넌트가 업데이트**됩니다.

또한 `componentDidMount`와 `componentWillUnmount` 메서드를 통해 **컴포넌트가 마운트되고 언마운트될 때 실행될 동작을 정의**합니다. 이를 통해 복잡한 상태 변화가 발생하더라도 컴포넌트의 동작을 통제할 수 있습니다.

<br>

**클래스 컴포넌트의 한계점**

하지만 클래스 컴포넌트는 상태가 캡슐화되어 있어 **상태와 로직을 쉽게 분리하거나 재사용하기 어려운** 단점이 있습니다. 또한 코드가 길어질 수 있고, **상태를 사용하려면 `this` 바인딩을 신경 써야 한다**는 점에서 불편함이 따릅니다.

- [📓참고] this 바인딩이 까다로울 수 있는 예시

  ```jsx
  import React, { Component } from 'react';

  class Counter extends Component {
    constructor(props) {
      super(props);
      this.state = {
        count: 0,
      };
      **// 바인딩을 수동으로 설정하지 않으면, this가 undefined가 됩니다.
      this.handleIncrement = this.handleIncrement.bind(this);**
    }

    handleIncrement() {
      this.setState({ count: this.state.count + 1 });
    }

    render() {
      return (
        <div>
          <p>Count: {this.state.count}</p>
          <button onClick={this.handleIncrement}>Increment</button>
        </div>
      );
    }
  }

  export default Counter;

  ```

  <br>

**함수 컴포넌트에게 부족했던 것**

이러한 한계를 극복하기 위해 등장한 **함수 컴포넌트**는 재사용성이 높지만, **상태를 캡슐화할 수 없고** 특정 마운트 단계에서 필요한 동작을 **스스로 정의할 수 없다는** 단점이 있었습니다.

<br>

> 🤷‍♂️: **상태 유지**도 안되고 **생명 주기**에 따른 동작도 정의할 수 없다니, 이래서는 React 를 할 수 없겠는데요…?
>
> 💁‍♀️: 함수도 캡슐화를 할 수있습니다. 바로 “**클로저**”를 통해서요..!

<br>

리액트 팀은 함수 컴포넌트에서도 **‘상태’를 관리하고 ‘생명 주기’를 제어할 수 있도록** 클로저를 활용한 캡슐화된 함수를 만들었습니다. 그것이 바로… **Hook**입니다!

<br>

- [📓참고] “함수형 컴포넌트? 함수 컴포넌트?”

  > **함수형(Functional)이 아니라 함수(Function) 컴포넌트라고 부르는 이유**
  >
  > 함수형 프로그래밍(Functional Programming)에서 '함수형'이라는 용어는순수 함수(pure function),불변성(immutability),사이드 이펙트 없음(side-effect free) 등과 같은 특성을 강조하는데, 리액트 함수 컴포넌트는 순수 함수가 아니고 사이드 이펙트가 일어나는 등과 같은 이유로 리액트는 '함수 컴포넌트'라는 말이 용어가 더 정확합니다.

  <br>

### **2. 함수 컴포넌트에서 React 상태 및 생명주기 기능을 "hook into"할 수 있게 해준다.**

<aside>

**Hooks are functions that let you “hook into” React state and lifecycle features from function components.**

훅은 함수 컴포넌트에서 리액트의 상태와 생명주기 기능을 연결하여 사용할 수 있게 해주는 함수입니다.

</aside>

훅(Hook)은 “hook into”라는 표현에서 유래되었습니다.
여기서 “hook into”는 “연결하다” 또는 “사용하다” 정도로 해석할 수 있습니다.

- [📓참고] Hook into. 사전적 의미

  > : to become connected to (something, such as a computer network or a source of electrical power)
  >
  > (컴퓨터 네트워크 또는 전력 공급원과 같은) 연결할 대상

리액트에서 가장 먼저 접하게 되는 훅 `useState`와 `useEffect`는 이러한 맥락에서 만들어졌습니다.

- `useState`는 상태를 캡슐화하여 컴포넌트가 리렌더링되어도 상태가 유지되도록 합니다.
- `useEffect`는 의존성 배열과 반환값을 통해 생명주기를 제어할 수 있도록 돕습니다.

훅 덕분에 함수형 프로그래밍의 장점을 살리면서,
리액트의 상태 관리와 생명 주기 제어 기능을 충분히 활용할 수 있게 되었습니다.

<br>

## [3] 진짜, Hook 과 유틸 함수의 차이

> 중요한건 Built-in Hook이냐 Custom Hook이냐가 아닙니다.
> 이것은 훅과 유틸함수의 차이를 묻는것과 같습니다.

훅의 탄생배경에서 살펴 보았듯 훅은 함수 컴포넌트의 상태관리와 생애주기 관리를 보완하기 위해 존재합니다.
따라서 훅은 함수 컴포넌트로 이루어진 리액트 환경에서 호출되는 것이 아니면 의미가 없습니다.

### 차이점 1. **호출 조건과 환경**

- **Hook**은 반드시 **React의 함수 컴포넌트 또는 다른 Hook 내부**에서만 호출할 수 있으며, 이 환경이 아니면 동작하지 않습니다. 이는 Hook이 리액트 생명주기와 상태 관리에 깊이 결합된 기능이기 때문입니다.
- **유틸 함수**는 이런 제약이 없습니다. 호출 위치나 사용 환경에 영향을 받지 않으며, 함수 컴포넌트 외부나 클래스 컴포넌트 등에서도 자유롭게 사용할 수 있습니다.

### 차이점 2. **주요 목적**

- **Hook**의 주된 목적은 **컴포넌트의 상태 및 생명주기를 다루는 것**입니다.
- **유틸 함수**는 상태나 생명주기와 무관하게, 특정 로직이나 기능을 재사용할 수 있도록 만든 **일반 함수**입니다.
  유틸 함수는 데이터 가공, 계산, 형식 변환 등 상태와 관계없는 작업을 수행하는 데 주로 사용됩니다.

### 차이점 3. 영향력

- **Hook**은 컴포넌트의 **상태 관리에 관여되**어 있으며, 렌더링에 **영향을 미칩니다**.
- 반면, **유틸 함수는 특정 데이터를 반환하는 순수 함수**로, 호출된 순간에만 작동하고 렌더링에 영향을 주지 않습니다. 같은 인수에 항상 같은 결과를 반환하며, **사이드 이펙트가 없습니다**.

Hook과 유틸 함수는 사용 목적부터 호출 환경까지 다르다는 점에서 구분되며, 각각의 역할이 리액트 컴포넌트에서 필요에 따라 다르게 적용된다는 것을 확인할 수 있습니다.

<br>

## [4] 마치며

코드를 작성할 때마다 “이 코드는 어느 폴더에 위치해야 하는가?”를 고민합니다. 내장 훅은 별다른 고민 없이 사용할 수 있지만, 커스텀 훅을 만들 때는 종종 “이것이 정말 훅인가, 아니면 단순히 `use`라는 이름이 붙은 유틸 함수일 뿐인가?”라는 생각이 듭니다. 특히 코드 리팩토링 시 재사용성이 높아 보이는 코드 블록을 분리할 때 이런 고민이 더 깊어지곤 합니다. 왜 어떤 것은 자연스럽게 훅이라고 느끼고, 어떤 것은 유틸 함수로 여겨지는 걸까요?

유틸 함수는 상황에 따라 순수 함수의 조건을 지키지 못할 수도 있고, 훅을 사용하지 않더라도 커스텀 훅을 만들 수 있지만, 레거시 리액트 문서를 통해 개념과 맥락을 이해하게 되면서 폴더 구조를 더욱 근거 있게 분류할 수 있게 되었습니다.

<br>

---

### 참고자료

[메리엄-웹스터 영영사전](https://www.merriam-webster.com/dictionary/hook%20into) hooki into 의 뜻

[💻용뇽 개발 노트💻:티스토리](https://yong-nyong.tistory.com/78) 함수형 컴포넌트? 함수 컴포넌트?

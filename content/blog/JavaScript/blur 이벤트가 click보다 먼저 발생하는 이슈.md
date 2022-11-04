---
title: 'blur 이벤트가 click보다 먼저 발생하는 이슈'
date: 2022-07-05
category: 'JavaScript'
draft: false
---

> 로그인 폼과 관련된 문제
>
> 1. input 박스에 텍스트를 입력하면 이를 지울 수 있는 x 버튼이 그려진다.
> 2. 사용자가 input 박스 이외의 영역을 클릭해서 focusout(blur) 되면 x 버튼을 지운다.
> 3. input 박스에 텍스트를 입력하는 도중에 x 버튼을 누르면 입력 중인 텍스트를 지우고 x 버튼도 지운다.
>    그런데, 세 번째 룰에 따라 x 버튼을 누르면 텍스트가 지워지고 버튼이 사라져야 하는데, 버튼이 먼저 사라져버리는 문제가 발생했다.
>    두 번째 룰을 보면 focusout이 발생하면 버튼을 삭제하는데, 사용자가 x 버튼을 누르는 순간 포커스가 input에서 button으로 옮겨가면서 focusout 이벤트가 발생하는 것이다.
>    그래서 당연히 브라우저는 x 버튼을 지워버린다.

### 원본 코드

```js
const 컨테이너 = input과button을포함하는div

// 입력이 발생하면 버튼 추가, 텍스트 지우면 버튼도 지우기
input박스.addEventListener('input', ({ target }) => {
  컨테이너.appendChild(button)
  if (target.value.length === 0) 텍스트지우기버튼.remove()
})

// input 박스 활성화되면 버튼 추가(bubbling x)
input박스.addEventListener('focus', () => {
  컨테이너.appendChild(텍스트지우기버튼)
})

// input 박스 비활성화되면 버튼 지우기(bubbling x)
input박스.addEventListener('blur', () => {
  텍스트지우기버튼.remove()
})

// 버튼 누르면 텍스트 지우고 버튼도 지우기
텍스트지우기버튼.addEventListener('click', () => {
  input박스.value = ''
  텍스트지우기버튼.remove()
})
```

### 사용자가 X 버튼을 누르면 발생하는 일

`mousedown -> mousemove(optional) -> mouseup -> click`

사용자가 마우스를 클릭할 때 발생하는 이벤트의 흐름이다.
여기서 **mousedown**을 하면 (마우스를 누르고 움직이거나/떼기 전) 포커스가 해당 엘리먼트로 이동하게 된다.
즉, **이전에 포커스되어있던 엘리먼트는 focusout(blur)이 발생**한다.

문제 해결의 첫 번째 실마리는 이것이었다. 버튼을 클릭하면 기능을 수행하기 전에 blur 이벤트에 의한 버튼 삭제가 수행되고 있었다.
그렇다면 blur와 click 이벤트의 순서를 제어할 수는 없을까?

### blur 이벤트와 click 이벤트의 순서

자료를 찾는 과정에서 대부분의 글은 click 대신 mousedown 이벤트를 사용하라고 한다.
그러면 input 박스가 focusout(blur)되기 전에 로직을 실행할 수 있기 때문이다.

그런데 이 방식에는 한 가지 단점이 있다.

```
> onClick should not be replaced with onMouseDown.
> While this approach somewhat works, the two are fundamentally different events that have different expectations in the eyes of the user. Using onMouseDown instead of onClick will ruin the predictability of your software in this case. Thus, the two events are noninterchangeable.
> To illustrate: when accidentally clicking on a button, users expect to be able to hold down the mouse click, drag the cursor outside of the element, and release the mouse button, ultimately resulting in no action. onClick does this. onMouseDown doesn't allow the user to hold the mouse down, and instead will immediately trigger an action, without any recourse for the user. onClick is the standard by which we expect to trigger actions on a computer.
> In this situation, call event.preventDefault() on the onMouseDown event. onMouseDown will cause a blur event by default, and will not do so when preventDefault is called. Then, onClick will have a chance to be called. A blur event will still happen, only after onClick.
> After all, the onClick event is a combination of onMouseDown and onMouseUp, if and only if they both occur within the same element.
```

stackoverflow에서 가져온 답글인데, 사용자가 클릭을 하다가 커서를 드래그해서 엘리먼트 밖에서 떼는 것으로 취소를 할 때가 있다.
mousedown 이벤트에 로직을 포함시킨다면, 사용자가 클릭을 하면 바로 이벤트 핸들러가 실행되기 때문에 사용자는 액션을 취소할 방법이 없다ㅋㅋㅋ

따라서 click 이벤트를 mousedown으로 대체하기보다는, **mousedown 이벤트의 기본 동작을 취소시키고 click 이벤트를 사용하는 것**을 추천한다.

```js
...
// 기본 동작 취소
텍스트지우기버튼.addEventListener("mousedown", (e) => {
  e.preventDefault();
});

// 버튼 누르면 텍스트 지우고 버튼도 지우기
텍스트지우기버튼.addEventListener("click", () => {
  input박스.value = "";
  텍스트지우기버튼.remove();
});
```

이 방법을 사용한다면 mousedown의 동작이 취소되면서 focusout(blur)이 발생하지 않게 되고, click 이벤트의 로직을 유지할 수 있게 된다.

### Reference

[onclick() and onblur() ordering issue](https://stackoverflow.com/questions/17769005/onclick-and-onblur-ordering-issue)

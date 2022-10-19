---
title: "의식적으로 쓰기 - 선언과 단언"
date: 2022-10-19T22:31:04+09:00
draft: false


categories:
- Typescript
tags:
- conscious
- Typescript
- type assignment
keywords:
- type assignment
- Typescript
- 선언
- 단언
- type 할당
---

# Typescript에선 변수에 type을 어떻게 할당하나요?

여러분이 위와 같은 질문을 받았다면, 어떻게 답변하시겠어요? 저는 Effective Typescript라는 책을 읽으면서, 이 질문을 생각하고 순간 멍해짐을 느꼈습니다. 제겐 너무나도 당연스러운 말인 변수에 type을 할당하는 문장을 풀어서 설명하려니, 적절한 문장이 떠오르지 않았습니다. Typescript에서  변수에 type을 할당하는 것은 도대체 어떤 의미일까요?

## 변수에 type을 할당한다?

저는 변수에 type을 할당한다는 것에 영향을 받는 부분에 집중했습니다. 그래서 변수에 type을 할당한다는 말을 아래와 같이 해석해 보았습니다. 결국 할당으로 영향을 받는 것이 Typescript compiler이기 때문입니다.

> **Typescript compiler에게 어떤 변수가 이런 type이라고 알려주는 과정**
> 

 이 글은, 알지만 막상 설명하려면 막막한, `선언`과 `단언`을 설명합니다. 혹시 놓친 부분이 없는지, 확인하는 느낌으로 읽어보시면 좋을 것 같습니다.

# 선언과 단언은 뭔가요?

 type의 `선언`과 `단언`은 Typescript에서 **변수에 type을 할당하는 방법**입니다. 다음 `apple`이라는 변수를 시작으로 그 이유를  `apple`이라는 변수의 타입 추론은 아래 주석과 같습니다. 정의할 때부터 할당할 때까지, `apple`은 `any` type으로 추론됩니다. 할당 구문 이후, Typescript compiler는 할당된 값 `{color: string}`을 `apple`의 type으로 추론합니다.

```tsx
let apple;                // any
apple = { color: 'red' }; // any

apple;                    // {color: string}
```

만일 변수 `apple`에 `shape`라는 속성을 추가하고 싶으면 어떻게 할까요? 가장 쉬운 방법으로 `color` 와 `shape`를 한번에 할당하는 방법이 있을 것입니다. 그 결과 Typescript compiler에 의해 아래 주석과 같은 type으로 추론될 것입니다. 

```tsx
apple = {color:'red', shape: 'circle'} // {color: string, shape: string}
```

 하지만 만약 `shape`를 초기에 설정하기 싫다면 어떤 방법을 선택할 수 있을까요? `interface`나 `type`이라는 예약어를 사용해 `shape` 속성을 optional type으로 정의하는 방법을 사용할 수 있을 것입니다. 이렇게 정의한 type을 변수에 할당하면, `shape`속성을 나중에 정의해도 Typescript compiler는 오류를 발생하지 않을 것입니다. 이때 우리는 `선언`과 `단언`을 활용합니다.

# 선언과 단언을 적용해보기

`apple`의 type `Fruit`를 아래와 같이 정의해 보았습니다.

```tsx
interface Fruit {
  color: string;
  shape?: string; // shape를 초기에 설정하고 싶지 않아, optional로 정의합니다.
};
```

이제 `선언` 과 `단언` 의 방법으로, 변수에 type을 할당해 봅시다.

### 선언으로 type 할당하기

`선언`은 **변수가 특정 type이라고 할당**하는 방법입니다. 그렇기 때문에, `선언`은 변수를 정의할 때 사용 가능한 방법입니다. 아래 예시처럼 `: Fruit` 라는 표현을 `apple`뒤에 추가해서, `apple`이라는 변수가 `Fruit`라는 type이라고 compiler에게 알려 줍니다. type의 추론은 아래 주석과 같습니다. 간단하죠?

```tsx
let apple: Fruit = {color: 'red'}; // Fruit
apple.shape = 'circle';            // Fruit
```

그렇다면 이제 `단언` 방법으로 할당해 봅시다.

### 단언으로 type 할당하기

`단언`은 변수가 아닌 **값에 type을 할당**하는 방법입니다. `단언`은, Typescript compiler의 동작을 이용해, 결과적으로 변수의 type을 할당합니다. 그렇기 때문에 `단언`은 변수에 값을 할당할 때 사용 가능한 방법입니다. 

아래 예시처럼 `as Fruit` 라는 표현을 값 뒤에 추가해서, `apple`이라는 변수에 할당하는 `object`가 `Fruit` type이라고 할당합니다. Typescript compiler는 할당된 값의 type인 `Fruit` 가 변수 `apple`의 type이라고 추론합니다. 이렇게 `단언으로 변수 `apple`에 `Fruit` type을 할당했습니다.

```tsx
let apple = {color: 'red'} as Fruit; // any
apple.shape = 'circle';              // Fruit
```

# 왜 나눴을까?

우리는 `선언`과 `단언`이라는 2가지 방법으로 변수 `apple`이 `Fruit` type을 할당했습니다. 하지만 달성하고 나니 한가지 궁금증이 떠오릅니다. 왜 굳이 2가지 방법을 따로 분리해 두었을까요? 같은 결과를 위해서라면, 한가지 방법만 만들면 되지 않았을까요?

이 질문에 대해 답변하기 위해,  `Fruit` type에 `taste`라는 필수 속성을 추가해 보았습니다. 그 결과로 발생하는 Typescript compiler의 동작을 통해, 따로 사용하는 이유를 생각해 봅시다.

```tsx
interface Fruit {
	color: string;
  shape?: string;
  taste: string; // 추가, 필수 속성
}
```

### 선언의 경우

 `선언`을 통해 `apple` 이라는 **변수가 `Fruit` type**이라고 할당했다면, Typescript compiler는 이 변수에 할당되는 object를 `Fruit`type이라고 생각하고 validation합니다. 그 결과, 아래와 같이 필수 속성 `taste`가 빠졌다는 validation error를 발생시켜줍니다.

```tsx
/* Fruit type에 필수 속성인 taste가 {color:'red'}에 없습니다 */
let apple: Fruit = {color: 'red'} 
```

 `선언`을 사용해 type을 할당하면, 개발 중에 필수 속성을 쓰지 않는 오류를 줄일 수 있을 것 입니다.

### 단언의 경우

 `단언`은 `apple`이라는 변수에 ‘할당한’ 값인 object의 type이 `Fruit` type이라고 Typescript compiler에게 알려주는 방법입니다. 그 결과, 아무런 error가 발생하지 않습니다. 선언과 다르게, 필수 속성인 `taste`가 없는데도 에러가 발생하지 않는 것입니다.

```tsx
apple = {color:'red'} as Fruit; // any
apple.shape = 'circle';         // Fruit
console.log(apple);             // {color: 'red', shape: 'circle'}
```

 당장은 문제 없지만, 나중에 `apple`의 `taste`속성을 참조할때 문제가 생길 수도 있겠죠?

# 선언과 단언을 구분한 이유

위에서 보았듯, `선언`과 `단언`은 그 의미에서 부터 차이가 발생합니다. `선언`은 변수에, `단언`은 값에 type을 표시하는 작업입니다. 이에 따라, Typescript compiler는 `선언` 했을 땐 할당하는 값의 validation을 수행하고, `단언`은 값 자체를 해당 type으로 validation 없이 처리합니다.

 저는, Typescript의 설계자 입장에서, 개발자들이 이런 차이를 인식하고 사용하기를 바랐다고 생각합니다. 그렇기 때문에 `선언`과 `단언`을 굳이 따로 제공하고 있다고 생각합니다. 이 생각을 바탕으로, 저는 변수에 type을 할당하는 상황에 따라 일관적으로 `선언`과 `단언`을 구분해서 사용하려 노력합니다.

## 저는 이럴때 이 방법을 씁니다.

### 값을 정의할때 - `선언`

저는 평소 변수를 정의할때 `선언`을 사용합니다. 위에서 말했던 compiler의 validation을 사용하고 싶기 때문입니다. 하지만 이것이 불편할 때도 있습니다. HTMLElement나 api response 같이, 저는 알지만 compiler가 추측하기에는 위험한 요소들을 다룰 땐 `선언`이 충분하지 않기도 합니다.

### 확실하고 생산성이 높아질때 - `단언`

위와 같은 상황에서 `단언`을 씁니다. 해당 값의 type을 확신 할 수 있을때, `단언`을 통해 좀더 빠르게 작업을 진행할 수 있다고 판단되면, `단언`을 씁니다. `단언`은 Typescript compiler에게 값의 추론을 강제하도록 지시하는 것과 다름이 없기에, 평소에는 많이 사용하지 않습니다.

# 마무리

Typescript에서 변수의 type을 할당하는 방법인 `단언`과 `선언`을 알아보았습니다. 두 방법의 차이를 알고, 더 좋은 code를 생산하는데 도움이 되었다면 기분이 좋을 것 같습니다. 긴글 읽어주셔서 감사합니다.
---
layout: single
title: '리터럴, 유니언과 교차 타입'
categories: 'TypeScript'
tag: [typescript, types, literal, union, intersection]
toc: true
toc_label: 'Contents'
toc_sticky: true
author_profile: false
sidebar:
  nav: 'sidebar-category'
---

# 리터럴 타입(Literal type)

일반적인 문자열과 숫자 타입 외에도, 타입 위치에서 특정한 문자열과 숫자를 참조할 수 있습니다.

리터럴 타입은 집합 타입의 보다 구체적인 하위 타입입니다. 이것이 의미하는 바는 타입 시스템 안에서 `"Hello World"`는 `string`이지만, `string`은 `"Hello World"`가 아니란 것입니다.

오늘날 TypeScript에는 문자열과 숫자, 두 가지 리터럴 타입이 있는데 이를 사용하면 문자열이나 숫자에 정확한 값을 지정할 수 있습니다.

## 리터럴 타입 좁히기 (Literal Narrowing)

`var` 또는 `let`으로 변수를 선언할 경우 이 변수의 값이 변경될 가능성이 있음을 컴파일러에게 알립니다. 반면, `const`로 변수를 선언하게 되면 TypeScript에게 이 객체는 절대 변경되지 않음을 알립니다.

```tsx
// const를 사용하여 변수 helloWorld가 절대 변경되지 않음을 보장합니다.

// 따라서, TypeScript는 문자열이 아닌 "Bob"로 타입을 정합니다.
const userName1 = 'Bob'

// 반면, let은 변경될 수 있으므로 컴파일러는 문자열(string)이라고 선언할 것입니다.
let userName2 = 'Tom'

userName2 = 3 //에러, 문자열로만 바꿀 수 있다.
```

무한한 수의 잠재적 케이스들 (문자열 값은 경우의 수가 무한대)을 유한한 수의 잠재적 케이스 (`userName1`의 경우: 1개)로 줄여나가는 것을 타입 좁히기 (narrowing)라 한다.

리터럴 타입 자체로는 그다지 가치가 없습니다. 변수가 하나의 값만 가질 수 있다면 그다지 유용하지 않습니다!

그러나 리터럴을 유니언으로 결합함으로써, 훨씬 더 유용한 개념을 표현할 수 있습니다.

## 문자열 리터럴 타입 (String Literal Types)

실제로 문자열 리터럴 타입은 유니언 타입, 타입 가드 그리고 타입 별칭과 잘 결합됩니다. 이런 기능을 함께 사용하여 문자열로 enum과 비슷한 형태를 갖출 수 있습니다.

리터럴을 유니언으로 결합하는 예를 들면, 특정 알려진 값의 집합만 받아들이는 함수 같은 것이 있다.

**예제1.**

```tsx
function printText(s: string, alignment: 'left' | 'right' | 'center') {
  // ...
}
printText('Hello, world', 'left')
printText("G'day, mate", 'centre')
//Argument of type '"centre"' is not assignable to parameter of
//type '"left" | "right" | "center"'.
```

**예제2.**

```tsx
type Job = 'doctor' | 'developer' | 'teacher'

interface User {
  name: string
  job: Job
}

const user: User = {
  name: 'Amy',
  job: 'doctor', //Job에 없는 것을 넣으면 에러 발생
}
```

## 숫자형 리터럴 타입 (Numeric Literal Types)

TypeScript에는 위의 문자열 리터럴과 같은 역할을 하는 숫자형 리터럴 타입도 있습니다.

**예제1.**

```tsx
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1
}
```

**예제2.**

```tsx
interface HighSchoolStudent {
  name: string
  grade: 1 | 2 | 3
}

const student: HighSchoolStudent = {
  name: 'Tom',
  grade: 1,
}
```

# 유니언 타입(Union type)

TypeScript의 타입 시스템은 다양한 연산자를 사용하여 기존의 타입에서 새로운 타입을 구성할 수 있게 해줍니다.
타입을 결합하는 첫 번째 방법은 유니언 타입입니다. 유니언 타입은 두 개 이상의 다른 타입에서 형성된 타입으로, 이 타입들 중 하나일 수 있는 값을 표현합니다. 이러한 타입을 우리는 유니언의 멤버라고 합니다.

문자열이나 숫자에 대해 작동할 수 있는 함수를 작성해봅시다:

```tsx
function printId(id: number | string) {
  console.log('Your ID is: ' + id)
}
// OK
printId(101)
// OK
printId('202')
// Error
printId({ myID: 22342 })
//Argument of type '{ myID: number; }' is not assignable to parameter of type 'string | number'.
```

## 유니언 타입 사용하기

유니언 타입에 맞는 값을 제공하는 것은 간단합니다 - 유니언의 멤버 중 하나와 일치하는 타입을 제공하면 됩니다. 유니언 타입의 값을 가지고 있다면, 어떻게 작업할까요?

TypeScript는 유니언의 모든 멤버에 대해 유효한 경우에만 연산을 허용합니다. 예를 들어, string | number라는 유니언을 가지고 있다면, string에만 사용할 수 있는 메서드는 사용할 수 없습니다:

```tsx
function printId(id: number | string) {
  console.log(id.toUpperCase())
}
//Property 'toUpperCase' does not exist on type 'string | number'.
//Property 'toUpperCase' does not exist on type 'number'.
```

코드로 유니언을 좁히는 것이 해결책입니다, 마치 타입 주석 없이 JavaScript에서 하던 것처럼 말이죠. 좁히기는 TypeScript가 코드의 구조를 기반으로 값에 대한 더 구체적인 타입을 추론할 수 있을 때 발생합니다.

예를 들어, TypeScript는 typeof 값이 "string"일 경우에만 문자열 값이 있을 것이라고 알고 있습니다:

```tsx
function printId(id: number | string) {
  if (typeof id === 'string') {
    // 이 분기에서 id는 'string' 타입입니다.
    console.log(id.toUpperCase())
  } else {
    // 여기서 id는 'number' 타입입니다.
    console.log(id)
  }
}
```

이러한 방식으로 유니언 타입을 사용하여 더 유연하고 안전한 코드를 작성할 수 있습니다.

**예제2.**

```tsx
interface Car {
  name: 'car'
  color: string
  start(): void
}

interface Mobile {
  name: 'mobile'
  color: string
  call(): void
}

function getGift(gift: Car | Mobile) {
  if (gift.name === 'car') {
    gift.start()
  } else {
    gift.call()
  }
}
```

# 교차타입(Intersection Type)

교차 타입은 유니언 타입과 밀접한 관련이 있지만, 사용 방법은 매우 다릅니다. **교차 타입은 여러 타입을 하나로 결합합니다.** 기존 타입을 합쳐 필요한 기능을 모두 가진 단일 타입을 얻을 수 있습니다. 예를 들어, `Person & Serializable & Loggable`은 `Person` *과* `Serializable` *그리고* `Loggable`입니다. 즉, 이 타입의 객체는 세 가지 타입의 모든 멤버를 갖게 됩니다.

예를 들어, 일관된 에러를 다루는 여러 네트워크 요청이 있다면 해당 에러 핸들링을 분리하여 하나의 응답 타입에 대응하는 결합된 자체 타입으로 만들 수 있습니다.

**예제1.**

```tsx
interface ErrorHandling {
  success: boolean
  error?: { message: string }
}

interface ArtworksData {
  artworks: { title: string }[]
}

interface ArtistsData {
  artists: { name: string }[]
}

// 이 인터페이스들은
// 하나의 에러 핸들링과 자체 데이터로 구성됩니다.
type ArtworksResponse = ArtworksData & ErrorHandling
type ArtistsResponse = ArtistsData & ErrorHandling

const handleArtistsResponse = (response: ArtistsResponse) => {
  if (response.error) {
    console.error(response.error.message)
    return
  }
  console.log(response.artists)
}
```

**예제2.**

```tsx
interface Car {
  name: string
  start(): void
}

interface Toy {
  name: string
  color: string
  price: number
}

const toyCar: Car & Toy = {
  //Car 와 Toy를 합쳐서 toyCar를 만들어줄 수 있다.
  name: '타요',
  start() {},
  color: 'pink',
  price: 1000,
}
//모든 속성을 다 기입해줘야 한다. 그렇지 않으면 에러표시
```

### 참조

https://typescript-kr.github.io/pages/literal-types.html  
https://joshua1988.github.io/ts/guide/operator.html#intersection-type  
(타입스크립트 공식문서)[https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types]

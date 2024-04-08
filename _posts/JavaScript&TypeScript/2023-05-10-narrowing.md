---
title: Narrowing & Assertion(타입 확정)
date: 2023-05-10 23:41:00 +09:00
categories: [Frontend, JavaScript & TypeScript]
tags: [TypeScript]
image: /assets/img/post-cover/ts-img.png
---

string | number 같은 union type 에는 일반적으로 조작을 못하게 막아놔서 에러가 발생합니다.

```js
function narrFunction(x: number | string){
    return x + 1; // error
}
```

이를 해결할 수 있는 방법에 대해 알아보겠습니다.

---

### 📌 Narrowing

Narrowing은 실제로 선언된 타입들에 대해 더 **구체적인 타입에 대해 처리하는 것**으로,  
각 타입에 대한 처리를 분명하게 해서 **에러가 나지 않는 안전한 코드를 만드는 것**입니다.

```js
function narrFunction(x: number | string){
    if(typeof x === 'number'){
        return x + 1;
    }else if(typeof x === 'string'){
        return x + 1;
    }else{
        return 0
    };
}
```

조건문 if문과 typeof 를 이용하여 현재 파라미터의 타입을 검사해서 무엇이 실행되어야 하는지 정해주는 방법입니다.  
`typeof`는 타입이 애매하면 안되기 때문에 그것을 막기 위함입니다.

### 📌 Assertion

as를 이용하여 **타입을 덮어쓰는 방법**입니다.  
**어떤 타입이 들어올지 확실할 때**만 assertion을 사용해야합니다.

```js
function narrFunction(x: number | string){
    return (x as number) + 1;
}
console.log(narrFunction(123));
```

`변수명 as number` 라고 쓰면, 변수를 number로 타입을 변경해줍니다.  
union type 같은 복잡한 타입을 하나의 정확한 타입으로 줄이는 역할을 합니다.  
실제 코드 실행 결과는 as가 있을 때나 없을 때나 동일합니다.

---

\[Reference\]

[https://velog.io/@jinhengxi/Typescript-Narrowing-Assertion](https://velog.io/@jinhengxi/Typescript-Narrowing-Assertion)
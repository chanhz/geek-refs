# TypeScript

JavaScript 的超集，带有类型。

## 五种原始类型

1. 布尔值
关键字： `boolean`
```typescript
let isDone: boolean = false;
```

2. 数值
使用 number 定义数值类型：
```typescript
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
```

3. 字符串
使用 string 定义字符串类型：
```typescript
let myName: string = 'Tom';
let myAge: number = 25;

// 模板字符串
let sentence: string = `Hello, my name is ${myName}.
I'll be ${myAge + 1} years old next month.`;
```


其中 ` 用来定义 ES6 中的模板字符串，${expr} 用来在模板字符串中嵌入表达式。


4. 空值
JavaScript 没有空值（Void）的概念，在 TypeScript 中，可以用 void 表示没有任何返回值的函数：

```typescript
function alertName(): void {
    alert('My name is Tom');
}
```

5. Null 和 Undefined

与 void 的区别是，undefined 和 null 是所有类型的子类型。也就是说 undefined 类型的变量，可以赋值给 number 类型的变量：


```typescript
// 这样不会报错
let num: number = undefined;
// 这样也不会报错
let u: undefined;
let num: number = u;
```

## 任意值

any 类型的值，可以被赋值为任意类型。

**未声明类型的变量**
变量如果在声明的时候，未指定其类型，那么它会被识别为任意值类型：
```typescript
let something;
something = 'seven';
something = 7;

something.setName('Tom');
```

**类型推论**
TypeScript 会在没有明确的指定类型的时候推测出一个类型，这就是类型推论。
如果定义的时候没有赋值，不管之后有没有赋值，都会被推断成 any 类型而完全不被类型检查：
```typescript
let myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
//等价于
let myFavoriteNumber: string = 'seven';
myFavoriteNumber = 7;
```


## 联合类型


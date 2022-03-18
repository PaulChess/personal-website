---
title: TS类型体操-模式匹配做提取
date: 2022-3-11T00:00:00.000+00:00
lang: zh
duration: 10min
---

### 模式匹配
比如这样一个 Promise 类型：
```ts
type p = Promise<'guang'>
```

我们想提取value的类型，可以这样做:
```ts
type GetValueType<P> = P extends Promise<infer Value> ? Value : never
```

通过 extends 对传入的类型参数P做模式匹配，其中值的类型是需要提取的，通过infer 声明一个局部变量 Value 来保存，如果匹配，则返回匹配到的 Value, 否则就返回 never 代表没匹配到。
```ts
type GetValueResult = GetValueType<Promise<'guang'>>
// type GetValueResult = "guang"
```

总结:

TS类型的模式匹配是通过extends对类型参数做匹配，结果保存到infer声明的局部变量里，如果匹配就能从该局部变量里拿到提取出的类型。

接下来看下模式匹配在数组、字符串、函数等类型中的应用。

### (1) 数组类型
#### First → 提取数组的第一个元素
```ts
type GetFirst<Arr extends unknown[]> = Arr extends [infer First, ...unknown[]]
  ? First : never
```

通过参数Arr约束只能是数组类型

> any和unknown的区别: any和unknown都代表任意类型，但是unknown只能接收任意类型的值，而any除了可以接收任意类型的值，也可以赋值给任意类型。
类型体操中经常使用unknown接收和匹配任意类型，而很少把任意类型赋值给某个类型变量

```ts
type GetFirstResult = GetFirst<[1, 2, 3]>
// type GetFirstResult = 1

type GetFirstResult = GetFirst<[]>
// type GetFirstResult = never
```

#### Last → 提取数组的最后一个元素
```ts
type GetLast<Arr> = Arr extends [...unknown[], infer Last]
  ? Last : never
```

#### PopArr → 提取剩余的元素
比如取出去掉了最后一个元素的数组:
```ts
type PopArr<Arr extends unknown[]> =
	Arr extends [] ? []
	  : Arr extends [...infer Rest, unknown] ? Rest : never
```

如果是空数组，则直接返回空数组  
否则匹配剩余的元素，放到infer声明的局部变量Rest中，将Rest返回
```ts
type PopResult = PopArr<[1, 2, 3]>
// type PopResult = [1,2]

type PopResult = PopArr<[]>
// type PopResult = []
```

#### ShiftArr
同理
```ts
type ShiftArr<Arr> =
	Arr extends [] ? []
	  : Arr extends [unknown, ...infer Rest] ? Rest : never
```

### (2) 字符串类型
匹配一个模式字符串，把需要提取的部分放到infer声明的局部变量里

#### StartsWith
判断字符串是否以某个前缀开头
```ts
type StartsWith<Str extends string, Prefix extends string> =
	Str extends `${Prefix}${string}` ? true : false
```

模式类型的前缀是Prefix, 后面是任意的string, 如果匹配返回true, 不匹配返回false
```ts
// usage
type StartsWithResult = StartsWith<'guang and dong', 'guang'>
// type StartsWithResult = true;

type StartsWithResult2 = StartsWith<'guang and dong', 'dong'>
// type StartsWithResult2 = false;
```

#### Replace
字符串可以匹配一个模式类型，提取想要的部分，自然也可以用这些再构成一个新的类型。  
比如实现字符串替换:
```ts
type ReplaceStr<
  Str extends string,
  From extends string,
  To extends string,
> = Str extends `${infer Prefix}${From}${infer Suffix}`
  ? `${Prefix}${To}${Suffix}` : Str
```

用Str去匹配模式字符串，模式串由From和之前之后的字符串构成，把之前之后的字符串放到通过infer声明的局部变量Prefix, Suffix里

用Prefix、Suffix加上替换成的字符串To构造成新的字符串返回
```ts
type ReplaceResult = ReplaceStr<'Guangguang\'s bests friend is ?', '?', 'dongdong'>
// type ReplaceResult = "Guangguang's bests friend is dongdong"

// 不匹配时
type ReplaceResult2 = ReplaceStr<'abc', '?', 'dongdong'>
```

#### Trim
由于我们不知道有多少个空白字符，所以只能一个个匹配和去掉，需要递归。

先实现trimRight:
```ts
type TrimStrRight<Str extends string> =
	Str extends `${infer Rest}${' ' | '\n' | '\t'}`
	  ? TrimStrRight<Rest> : Str

type TrimRightResult = TrimStrRight<'guang       '>
// type TrimRightResult = "guang";
```
如果Str匹配字符串+空白字符，那就把字符串放到infer声明的局部变量Rest中，然后递归

trimLeft
```ts
type TrimStrLeft<Str extends string> =
	Str extends `${' ' | '\n' | '\t'}${infer Rest}`
	  ? TrimStrLeft : Str
```

TrimRight和TrimLeft的结合就是Trim:
```ts
type TrimStr<Str extends string> = TrimStrRight<TrimStrLeft<Str>>

type TrimResult = TrimStr<'  dong  '>
// type TrimResult = "dong"
```

### (3) 函数类型
函数也可以做类型匹配，比如提取参数、返回值的类型

#### GetParameters
提取参数类型:
```ts
type GetParameters<Func extends Function> =
	Func extends (...args: infer Args) => unknown ? Args : never
```
Func和模式类型做匹配，参数类型放到用infer声明的局部变量Args里，返回值可以是任何类型，用unknown。

返回提取到的参数类型Args
```ts
type ParametersResult = GetParameters<(name: string, age: number) => string>

// type ParametersResult = [name: string, age: number]
```

#### GetReturnType
提取返回值类型:
```ts
type GetReturnType<Func extends Function> =
	Func extends (...args: unknown[]) => infer ReturnType
	  ? ReturnType : never
```

总结: TS类型的模式匹配是通过类型extends一个模式类型，把需要提取的部分放到通过infer声明的局部变量里，后面可以从这个局部变量拿到类型做各种后续处理。
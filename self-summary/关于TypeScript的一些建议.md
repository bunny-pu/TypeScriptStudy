JavaScript以其灵活的特性为大家所熟知，即使没有参与过JavaScript为语言基础的项目开发，我们也知道，JavaScript的每一个变量，都会在不同的操作下产生不同的类型和结果，也即是，我们很难通过仅仅看代码知道JavaScript代码的最终结果是什么，往往需要运行后才能确定其结果。这就让代码难以维护，bug难以排查。换句话说，在书写代码的时候，开发者很难预测代码最终会造成什么结果。TypeScript解决了JavaScript的这一问题。

> *“TypeScript is like [JavaScript](https://www.altexsoft.com/blog/engineering/javascript-ecosystem-38-tools-for-front-and-back-end-development/) but with no surprises.”*

TypeScript的两大特性帮我们解决了JavaScript中`过于灵活`的问题：

1. 静态类型检查与报错；

2. 类型推断与代码提示；

## 静态类型检查与报错

上面提到，JavaScript可以随时随地得改变变量的类型，使得代码很难看出其运行的结果，很可能我们写了几百行代码后，运行起来才发现是错误的。TypeScript可以让这个错误在书写代码的时期就让开发者感知到。

另外，对于一些场景，比如：

```
const user = {

name: "Daniel",

age: 26,

};

user.location; // returns undefined
```

虽然user对象中没有location变量，但在JavaScript中,运行这种代码并不会报错，而是得到`undefined`.

然而在TypeScript中，书写这种代码会得到<font Color=Red>类型“{ name: string; age: number; }”上不存在属性“location”。ts(2339)</font>的静态报错，并在运行的时候产生运行时异常。

## 类型推断与代码提示

TypeScript的类型推断十分强大，不仅仅针对复制后变量类型的确定。TypeScript能非常准确得得到函数的返回值类型，即使这个函数体中包含了if-else,switch等等复杂的条件判断，TypeScript也能分析出最终的返回类型。

基于强大的类型推断，代码提示让开发者更加便捷得进行开发。无需记住这些变量的类型或者对象的变量，TypeScript能及时提示你应该用哪些变量或者方法。

TypeScript如此强大，但有时候需要我们用正确的方式写代码，才能充分利用其特性。后文是一些建议。

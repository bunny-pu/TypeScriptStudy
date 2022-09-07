# 关于TypeScript的一些建议

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

## 1.非必要不要使用any作为变量的类型

这个可能是每一个关于TypeScript建议为主题的博客都会提到的一点，因为它十分重要。

上文提到JavaScript的灵活性很大程度上归因于类型的灵活。变量可以随意改变类型，对象和类可以随意改变方法或者类型。如果any作为TypeScript的变量类型，就表明，`此变量无需任何类型检查`，那么代码就回归了最原始的JavaScript的写法，偏离了我们写TypeScript的本意。

如果一个变量的类型无法判断或者出现一些特殊的类型断言，你可以使用[unknown](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)

，它可以当作是类型安全的`any`

另外，新手同学可能会写下面的代码

```
callback:() => any
```

如果想表达的是函数不会返回任何值，你可以写成：

```
callback:() => void
```

请记住，你可能永远不需要使用`any`

## 2.类型定义时，`object`, `Function`, `{}`,`[]`这些含义不明确的类型请谨慎使用

object或者{}作为类型时，TypeScript将无法得知其对象拥有的变量名称与类型，因而无法进行对应的代码检查，同样的对于函数的定义。

对于数组、函数、对象的类型定义，请务必注意写清楚其内部变量的类型、参数、返回值等等具体的类型。

## 3. 不要轻易使用Number, String, Boolean, Symbol，Object

`String`, `Number`, and `Boolean` (以大些字母开头的) 是合法的， 但他们极少出现).他们是基本类型的装箱版本。 平时使用 `string`, `number`, 或者 `boolean` 即可.

## 4.谨慎地设计和使用optional的变量

试想一种场景，我们需要定义两种形状类型：圆和方形。并且用半径来描述圆，用边长来描述方形

看看下面类型的定义两种方式：

```
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

```
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;
```

哪一种更好呢？

虽然前者看起来更加简单，但在使用的时候发现用法十分复杂。因为它没有表达出只有圆有半径，方形有边长这一信息。在这种定义下，TypeScript无法帮助我们做到完备的类型检查。比如，在确定了kind为circle的情况下，TypeScript仍然不能确定可以获取到radius的值，它依然可能是undefined的。

反观后者，虽然写起来略复杂，但很清晰得表达出了类型的信息。当确定kind为circle的情况下，radius就一定有值。

## 5.回调函数请勿轻易使用optional参数，除非你十分确定需要使用

理由是显而易见的，回调函数是我们定义需要外部实现的函数，为了外部的实现尽量简单，我们应该避免使用optional参数，除非非常确定。如果有optional参数的场景，或许我们应该新增一个不包含该参数的回调函数定义。

## 6.不要写仅仅参数长度不同的多个方法重载，应该使用optional参数

请不要按照下面的方法写代码：

```
/* WRONG */
interface Example {
  diff(one: string): number;
  diff(one: string, two: string): number;
  diff(one: string, two: string, three: boolean): number;
}
```

可以看到上面的返回值是完全相同的，后面的重载方法仅仅是多了几个变量这里推荐用optional写一个方法：

```
/* OK */
interface Example {
  diff(one: string, two?: string, three?: boolean): number;
}
```

```
function fn(x: (a: string, b: number, c: number) => void) {}
var x: Example = {...};
fn(x.diff);
```

上面的fn(x.diff)在Example使用重载的写法时是可以的，但是当Example是optional参数的写法会报错。但我们希望它报错，因为b和c的类型是不符合的。

Example使用重载的写法时是可以的，原因是TypeScript的方法前面在调用时候可以允许写入多余的参数。

另外，还有一点，optional的参数是可以传入undefined的，但是第一种写法通过方法重载是不可以的。

```
var x: Example;
// 使用方法重载时会报错，因为undefined不能赋值给string类型
// 使用optional时，就不会报错
x.diff("something", true ? undefined : "hour");
```

## 7.泛型的类型定义应该至少出现在两个地方

比如下面的代码

```
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}

greet("world");
```

就没有必要使用泛型，直接这样写就好：

```
function greet(s: string) {
  console.log("Hello, " + s);
}
```

## 8.请勿用一个字母表示泛型类型

我们在最初学习泛型时，可能发现官方的文档也会使用一个字母来表示泛型，但是这个缺点很明显----很难一下子看出类型的含义。回想一下，我们在对变量其起名字的时候，都会避免用`a`,`b`, `c`这样的名字，因为我们知道这样很难看出其含义。

❌

```
function head<T> (arr: T[]): T | undefined {
  return arr[0]
}
```

✅

```
function head<Element> (arr: Element[]): Element | undefined {
  return arr[0]
}
```

## 9.堤防枚举陷阱

目前看来，枚举在我们项目中应用得十分广泛。枚举定义为数字类型时，会出现下面的问题：

```
enum UserResponse {
  No = 0,
  Yes = 1,
}

function respond(recipient: string, message: UserResponse): void {
  // ...
}

respond("Princess Caroline", 2);
```

UserResponse的定义只有`0`和`1`，但是调用的时候传值`2`是不会报错的。使用数字作为枚举的值的时候，需要特别注意这一点。

除了上述需要避免的地方，下面有一些写TypeScript的小tips：

## 1.使用never保证全部的遍历

```
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

任何类型的值都不能赋予给`never`类型，所以我们可以在default中这样写来判断自己是否在switch中列举了所有的case。

## 2.传递组件参数类型

有时候我们需要传递一个组件到另一个组件作为child进行布局,常用的类型有下面两种：

```
interface AppProps {
  children?: React.ReactNode; 
  childrenElement: JSX.Element; 
}
```

总的来说，更加推荐使用`React.ReactNode`。按照Facebook的开发者的[表述](https://github.com/typescript-cheatsheets/react/issues/57)：

`JSX.Element`是`React.createElement`的返回值，但`React.ReactNode`是组件所有可能的返回值组合。

## 3.从类型中创造新的类型

TypeScript的类型系统十分强大，它允许你依据别的类型来创造自己的类型。

最简单的概念是泛型，TypeScript提供了大量的类型操作。我们也可以用已经有的值来表示类型。

通过结合这众多的类型操作符，我们可以进行复杂的类型操作，并且使其简明且可维护。下面的操作均是对已有的类型或者值来进行操作，来描述一个新的类型：

- [泛型](https://www.typescriptlang.org/docs/handbook/2/generics.html) - 可带参数的类型
- [Keyof 类型操作符](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html) - 使用 `keyof` 操作符来创建新的类型
- [Typeof 类型操作符](https://www.typescriptlang.org/docs/handbook/2/typeof-types.html) - 使用 `typeof` 操作类创建新的类型
- [索引访问类型](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html) - 通过 `Type['a']` 语法来创建一系列类型
- [条件类型](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html) - 有条件判断的类型
- [转换类型](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html) - 转换已有的类型来获取新的类型
- [模版字面类型](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html) - 通过模版字符串表示的转换类型

### 4.实用类型

TypeScript提供了一些实用的类型转换，下面举几个例子：

ReturnType<Type>:构造一个由函数的返回类型组成的类型

Omit<Type, Keys>:构造一个Type组成并删除Type中Keys的类型

```
// inside some library - return type { baz: number } is inferred but not exported
function foo(bar: string) {
  return { baz: 1 };
}

//  inside your app, if you need { baz: number }，使用ReturnType
type FooReturn = ReturnType<typeof foo>; // { baz: number }

//use Omit
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
}
 
type TodoPreview = Omit<Todo, "description">;
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
  createdAt: 1615544252770,
};
```

更多的实用操作可以参考[Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)

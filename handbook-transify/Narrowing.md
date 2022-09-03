# 缩小类型范围

假设有一个叫做`padLeft`的方法：

```
function padLeft(padding: number | string, input: string): string {
  throw new Error("Not implemented yet!");
}
```

如果`padding`是`number`类型的，那么这个数字就会被处理为input前面的空格的数目。如果`padding`是一个`string`,那么直接把字符串拼接在input前面展示。我们完成一下这个逻辑：

```
function padLeft(padding: number | string, input: string) {
  return " ".repeat(padding) + input;
}
```

<font color=Red>Argument of type 'string | number' is not assignable to parameter of type 'number'.
 Type 'string' is not assignable to type 'number'.</font>

我们会得到上面的这个报错。TypeScript提示我们不能对`number|string`类型的值进行只适用于`number`类型的操作。也就是说，我们需要先明确`padding`的类型就是`number`，下面才是我们需要的实现：

```
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

上面的代码看起来就跟无趣的JavaScript代码一样.除了我们写的一些类型注释，上面的TypeScript和JavaScript看起来一样。其实对应的观点是，TypeScript的类型系统的目的是在于让大家能尽可能容易得写有类型的JavaScript代码，而不是瞻前顾后的注意类型的安全性。

可能看起来要做的事情并不多，但其实隐藏了很多逻辑。就像TypeScript分析运行时静态类型，它覆盖了像`if/else`, 三元表达式，真假检查等等JavaScript中的运行时控制逻辑，这些都会影响到值的类型。

在`if`检查中，TypeScript注意到了`typeof padding === "number"`并且理解了这样一种特殊的类型保护。TypeScript会顺着所有代码可能执行的路径去检查对应位置的值的可能类型。这种特殊的检查（又称类型保护）和赋值，以及将值确定为更加确切类型的操作叫做*类型收紧*。很多编辑器中会检查这些类型的变化，可以看看下面的例子

```
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;//(parameter) padding: number

  }
  return padding + input;//(parameter) padding: string

}
```

下面有一些对类型收紧不同的理解

## `typeof`类型保护

从上文中可以看到，JavaScript支持在运行时通过`typeof`操作来检查一些基本类型信息。TypeScript在使用时会期望其返回一些特定的字符串：

- `"string"`
- `"number"`
- `"bigint"`
- `"boolean"`
- `"symbol"`
- `"undefined"`
- `"object"`
- `"function"`

跟上面的`padLeft`方法一样，很多JavaScript库经常能看到这样的使用，TypeScript也理解它是怎么收紧类型的。

在TypeScript中，通过`typeOf`的返回值来检查变量的类型叫做类型保护。因为TypeScript对`typeof`如何操作不同的值进行了特殊的处理，因为它知道JavaScript中一些古怪的地方。比如，它注意到上述的列表中，`typeOf`不会返回`null`,看看下面的例子：

```
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
```

报错<font color=Red>Object is possibly 'null'.</font>

接上

```
 console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```

在`printAll`方法中，我们尝试通过检查 `strs`是否是对象来确定它是否是一个数组（这里可以强调一下JavaScript中数组是对象的知识点），但结果在JavaScript中，`typeof null`的返回值是`"object"`!这其实是JavaScript在历史上一个令人依赖的意外。

经验丰富的开发者或许不会觉得意外，但并不是任何人都能很快得在使用JavaScript时反应到这点。但幸好，TypeScript会提示我们，`strs`不仅仅将类型收紧到`string[]|null`， 而不是`string[]`.

这个可能会帮我们很好过渡到`真假`的检查.

## 真假类型收紧

在JavaScript中我们在任意表达式中使用条件语句，`&&`s, `||`s, `if`声明、非（`!`）,还有更多的语句。但是有个情况，`if`语句中的判断并不是总是`boolean`.

```
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

在JavaScript中，类似`if`这样的条件构造会“强迫”语句里的条件转为`boolean`s类型，进而去理解这些语句，然后通过转化后的结果为`true``或者`false`来判断去走哪个分支的逻辑，下面的值：

- `0`
- `NaN`
- `""` (the empty string)
- `0n` (the `bigint` version of zero)
- `null`
- `undefined`

都会被转为`false`,其他的值都会被转为`true`.你也可以通过运行`Boolean`函数和双否定语句让值转为`boolean`s（后者的优势在于，TypeScript会识别为字面类型`true`, 而前者仅仅是类型`boolean`）

```
// both of these result in 'true'
Boolean("hello"); // type: boolean, value: true
!!"world"; // type: true,    value: true
```

这是一种十分受欢迎的操作，特别是在值为`null`或者`undefined`时的类型保护。举个例子，我们看看下面的`printAll`方法：

```
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

可以看到，我们摆脱了上文中的`strs`是否为`真`的提示。这样可以帮我们防止代码运行时出现下面可怕的错误：

> TypeError: null is not iterable

请记住，基本类型的真假性检查可能是错误的。比如，我们这样写`printAll`方法：

```
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  DON'T DO THIS!
  //   KEEP READING
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

上面的代码中，我们所有代码包裹了在`strs`的真假校验中，者会有个缺陷：我们可能不会再正确得去处理这些空字符串（`""`）了。

这里并不是TypeScript的问题，而是写的人对JavaScript不太熟悉。TypeScript通产可以帮你早点找到bugs，但是如果你选择对一个值不做处理，那么这里TypeScript也不会做过分规范化的处理。如果你需要规范化检查，可以使用类似linter这类工具。

最后一个关于真假类型收紧的点，是使用`!`就可以进入否定的分支中运行代码。

```
function multiplyAll(
  values: number[] | undefined,
  factor: number
): number[] | undefined {
  if (!values) {
    return values;
  } else {
    return values.map((x) => x * factor);
  }
}
```

## 相等性收紧

TypeScript也会使用`switch`声明和相等性校验比如`===`,`!==`, `==`以及`!=`去收紧类型，比如：

```
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    //(method) String.toUpperCase(): string
    x.toUpperCase();
    //(method) String.toLowerCase(): string
    y.toLowerCase();
  } else {
    //(parameter) x: string | number
    console.log(x);
    //(parameter) y: string | boolean
    console.log(y);
  }
}
```

当我们进行`x`和`y`的相等性校验时，TypeScript知道他们的类型也一致时才可能相等。再加上`string`是`x`和`y`唯一一个共同的类型, 所以相等时`x`和`y`的类型一定是`string`·

检查每个特定的字面值（相反的方式也一样）同样是有效的。在上文的真假类型收紧中介绍的，我们写了不小心没有处理空字符串的场景的`printAll`方法。其实我们可以用特定的校验去除`null`类型，TypeScript可以做到这点：

```
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      //(parameter) strs: string[]
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      //(parameter) strs: string
      console.log(strs);
    }
  }
}
```

JavaScript的`==`和`!=`校验相对而言比较宽松。如果你不熟悉这一点，检查一个值是否`==null`实际上不仅仅校验它的值是否为`null`----并且校验它是否是`undefined`。这一点和校验是否`==undefined`一样：它们检查是否一个值是`null`或者`undefined`.

```
interface Container {
  value: number | null | undefined;
}

function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
   //(property) Container.value: number
    console.log(container.value);
    // Now we can safely multiply 'container.value'.
    container.value *= factor;
  }
}
```

## 使用`in`操作赋缩小类型范围

JavaScript中有一种操作符来判断一个对象中是否有这个属性：`in`操作赋。TypeScript可以通过这个来缩小可能的类型。

比如，我们看看这种代码：`"value" in x`, 其中`"value"`是一个string的字面值并且`x`是一个联合类型。那么在"true"的代码分支中，`x`的类型中必须有可选（optional）的或者必须有的属性`value`，在"false"的代码分支中，`x`的类型为可选（optional）的或者干脆没有`x`这个属性。

```
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }

  return animal.fly();
}
```

这里重申一下，可选的属性会在类型收紧后依然存在于两个代码分支中，比如一个人他即会有用又会飞行（当然是借助适合的设备）。那么因此下面的代码他会出现在两个分支的逻辑中：

```
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };

function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal;

(parameter) animal: Fish | Human
  } else {
    animal;

(parameter) animal: Bird | Human
  }
}
```

## `instanceof`收紧

JavaScript的操作符用于检查是否一个值是另一个值的`instance`.正确切得说，在JavaScript中`x instanceof Foo`检查x的原型链中是否有`Foo.prototype`.我们在此不会进一步探讨了，后面当我们介绍classes的时候你会了解它更多的特性，它对大部分用`new`构造的的值十分有用。你大概可以猜到，`instanceof`是一种类型保护，也就是TypeScript可以通过`instanceof `s来进行判断来进行不同分支的如果进行类型范围的缩小.

```
function logValue(x: Date | string) {
  if (x instanceof Date) {
   //(parameter) x: Date
    console.log(x.toUTCString());
  } else {
   //(parameter) x: string
    console.log(x.toUpperCase());
  }
}
```

## 赋值操作

我们之前提到过，当我们给任何一个变量赋值，TypeScript都会根据赋值语句右侧的值来合理地限制左侧的类型。

```
//let x: string | number
let x = Math.random() < 0.5 ? 10 : "hello world!";

x = 1;
//let x: number
console.log(x);

x = "goodbye!";
//let x: string
console.log(x)
```

可以看到，两种赋值都是有效的。在第一次赋值的时候,`x`的类型变为`number`,后面的赋值中依然可以把`x`的类型改为`string`。这是应为在声明`x`类型（这是`x`最开始类型）是`string|number`。也就是说，赋值时的类型检查永远针对的是声明时的类型。正因为这个原因，如果我们把 `x`赋值为一个`boolean`类型，我们会看到一个错误提示。

```
x = true;
```

报错 <font color=Red>Type 'boolean' is not assignable to type 'string | number'.</font>

## 流程控制分析

目前为止，我们进行了许多基本的TypeScript特定分支来缩小类型范围的案例分析。TypeScript会浏览所有的变量然后在`if`s, `while`s,等等条件语句中进行类型保护, 但TypeScript做的不仅仅是这些，举个例子：

```
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

`padLeft`方法在第一个`if`块中有一个返回语句。TypeScript会分析这个语句并且查看后续的代码(`return padding+ input;`), TypeScript会了解到当 `padding`是`number`类型的时候，这部分的代码当是不会执行的。得到这个分析结果后，TypeScript会在这个分支移除`padding`的`number`类型（也就是类型从`number|string`缩小到`string`）。

这种代码的可达性分支叫做`流程控制分析`。在遇到类型保护和赋值的时候，TypeScript利用这个流程分析来缩小类型范围。当一个类型被分析的时候，控制流程可能会被重复得进行分割然后组合，通过如此来得到变量在不同代码区域的不同类型。

```
function example() {
  let x: string | number | boolean;

  x = Math.random() < 0.5;
  //let x: boolean
  console.log(x);

  if (Math.random() < 0.5) {
    x = "hello";
   //let x: string
    console.log(x);
  } else {
    x = 100;
    //let x: number
    console.log(x);
  }
  //let x: string | number
  return x;
}
```

## 使用类型断言

到目前为止，我们了解了已经存在于JavaScript中的一些缩小类型范围的操作。如果你想更加直接得更改变量的类型，该如何做呢？

我们需要写一个自定义的类型保护断言，而这只需要简单得通过定义一个返回值为类型断言的方法：

```
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

例子`pet is Fish`就是一个类型判断的返回值。这种判断的格式是`parameterName is Type`, 其中`parameterName`必须是当前方法参数的名字。

每当`isFish`被调用时，如果类型是符合的，TypeScript就会将变量缩小范围到这个特定的类型。

```
// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet();

if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

可以看到，TypeScript不仅仅知道在`if`分支中，`pet` 是`Fish`类型的；它还知道在`else`分支中，`pet`不是`Fish`类型的，而是`Bird`类型的。

你可以利用`isFish`的类型保护在`Fish|Bird`的数组中筛选出所有的`Fish`:

```
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// 或者，同样的
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];

// 当遇到更复杂的情形时
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === "sharkey") return false;
  return isFish(pet);
});
```

除此之外，类可以通过[use `this is Type`](https://www.typescriptlang.org/docs/handbook/2/classes.html#this-based-type-guards)来缩小它的类型范围。

## 区分联合类型成员

目前为止，我们着眼于类似`string`，`boolean`,`number`这些简单的类型。但在JavaScript中，我们大部分情况下都处理的是更为复杂的类型结构。

试想一种场景，我们遇到了两种形状：圆和方形。我们用半径来描述圆，而用边长来描述方形，我们用属性`kind`来描述我们当前处理的形状。根据以上的信息来定义`Shape`:

```
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

可以看到，我们使用字符串字面类型：`"circle"`和`"square"`来描述形状。通过使用

`"circle" | "square"`，我们可以避免后续的拼写错误：

```
function handleShape(shape: Shape) {
  // oops!
  if (shape.kind === "rect") {
//报错
    // ...
  }
}
```

报错<font color=Red>This condition will always return 'false' since the types '"circle" | "square"' and '"rect"' have no overlap.</font>

我们再来定一个`getArea`的方法，区别圆形以及方形来获取面积。我们首先来处理圆形的情况：

```
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
//报错
}
```

报错 <font color=Red>Object is possibly 'undefined'.</font>

配置[`strictNullChecks`](https://www.typescriptlang.org/tsconfig#strictNullChecks)后编译器会报错，因为`radius`可能没有被定义。我们看看检查`kind`属性后的情形：

```
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
//报错
  }
}
```

报错 <font color=Red>Object is possibly 'undefined'.</font>

TypeScript依然具体的情形。因为在这种情况下，我们比TypeScript更加了解变量的意义。如果我们假一个非空的断言（在`shape.radius`后加上`!`）来表明`radius`一定存在：

```
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }
}
```

不会报错了，但是代码看起来并不理想。我们十分强暴得告诉类型检查器，`shape.radius`一定被定义了，但这样十分容易逻辑出错。并且，由于可选属性被强制定义为一定存在，在这种没有[`strictNullChecks`](https://www.typescriptlang.org/tsconfig#strictNullChecks)校验的情况下，我们可以获取所有的属性。我们可以把代码写得更好。

上述代码的问题在于，我们无法通过`kind`属性的判断来得到当前有变量有`radius`还是`sideLength`。我们需要把确切的信息告诉*类型判断*。由此，来看看下面的`Shape`定义：

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

以上，我们把`Shape`分成了两种类型，两种类型拥有不同的`kind`属性，并且`radius`和`sideLength`分别在两者中是必要的属性.

来看看当我们获取直接`Shape`的`radius`属性时会发生什么：

```
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
}
```

报错 <font color=Red>Property 'radius' does not exist on type 'Shape'.
 Property 'radius' does not exist on type 'Square'.</font>

依然会报错，但错误和之前的不同。在前面的代码中，我们把`radius`定义为可选的，我们只是得到了一个可能不存在的错误（当[`strictNullChecks`](https://www.typescriptlang.org/tsconfig#strictNullChecks)打开时）。现在我们把`Shape`定义为一个联合类型，TypeScript可以更确切得告诉我们`Shape`可能是`Square`，并且`Square`没有`radius`属性！所有的报错的提示都是正确的，但只有Shape是联合类型的时候才会提示[`strictNullChecks`](https://www.typescriptlang.org/tsconfig#strictNullChecks)以外的有用信息。

接下来，我们再来尝试检查`kind`类型：

```
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    //(parameter) shape: Circle
    return Math.PI * shape.radius ** 2;
  }
}
```

写到这里，我们终于拜托了报错了！每个联合类型的成员都有共同拥有一个字面类型，TypeScript把它当作区分联合类型成员的标志。

在这个例子中，`kind`就是所有`Shape`共同拥有的属性。检查`kind`属性是否是`"circle"`就可以摆脱其他类型了。由此，`shape`类型缩小范围到`Circle`.

我们同样可以通过`switch`声明来进行检查工作。现在我们可以不用写烦人的`!`，而是优雅得完成`getArea`了：

```
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      //(parameter) shape: Circle
      return Math.PI * shape.radius ** 2;
    case "square":
      //(parameter) shape: Square
      return shape.sideLength ** 2;
  }
}
```

本小结的重点是对`Shape`进行合适的定义。其中至关重要的一点，我们需要告诉TypeScript这个信息：`Circle`和`Square`是同时拥有`kind`属性，但值不同的类型两种类型。通过这样定义可以让我们后续写出跟JavaScript一样的并且类型安全的TypeScript代码。现在，类型系统可以做"正确"得在`switch`声明中推断出类型了。

> 题外话，你可以尝试删除上面例子中的返回关键字，那么你此时可以看到类型检查可以帮你避免这种漏掉switch的错误。

区分联合类型成员不仅仅适用于我们这里讨论的圆和方形。它非常适用于JavaScript中各种形式的信息传递场景，比如发送网络请求（客户端/服务端通信），以及适应状态管理框架中的编码变化。

## `never`类型

在做类型范围缩小时，你移除了联合类型中的所有的可能类型后，此时可以使用TypeScript中的`never`类型，以表示一个不可能存在的状态。

## 穷尽的检查

给任何类型赋予`never`类型，但是，没有类型可以赋予给`never`(除了`never`本身)。你可以利用`never`永远不会出现来在switch语句中进行穷尽的检查。

```
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

如果加了一个新的成员到`Shape`联合类型中，就回出现一个TypeScript报错：

```
interface Triangle {
  kind: "triangle";
  sideLength: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      //报错：Type 'Triangle' is not assignable to type 'never'.
      return _exhaustiveCheck;
  }
}
```

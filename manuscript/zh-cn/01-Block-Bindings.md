# 块绑定（Block Bindings）

有史以来，变量的声明是JavaScript编程中的一个微妙的组成。在很多基于C语言的变成语言里，变量（或者*绑定*）都是在其声明时创建的。然而在JavaScript里并非如此。你的变量在什么时候创建取决于你声明它们的方式。同时，在ECMAScript 6里，它提供了一些选项使得人们能更方便地掌控他们。这一章将述说为什么传统的 `var` 声明会产生歧义，并引入介绍ECMAScript 6中的块绑定概念，最后提供一些使用块绑定的最佳实践方案。

## Var 声明和声明置顶（Hoisting）

通过 `var` 进行变量声明时，其在代码中的位置会被直接无视并将其视为于函数块顶部声明（若在函数块外的话则视为全局声明），这就是*声明置顶（Hoisting）*。下面的代码将演示声明置顶都干了些什么：

```js
function getValue(condition) {
    
    if (condition) {
        var value = "blue";
        
        // other code
        
        return value;        
    } else {
        
        // value exists here with a value of undefined
        
        return null        
    }
    
    // value exists here with a value of undefined
}
```

如果您对JavaScript并不熟悉，那么你会认为变量 `value` 仅在 `condition` 为真的时候才会被创建。实际上，变量 `value` 无论如何都会被创建。在这背后，JavaScript引擎会将函数 `getValue` 视为：

```js
function getValue(condition) {

    var value;

    if (condition) {
        value = "blue";

        // other code

        return value;
    } else {

        return null;
    }
}
``` 

变量 `value` 的声明会被置顶，至于变量赋值操作留在原处。这意味着变量 `value` 实际上是可以在 `else` 分支中被访问的。如果该变量在 `else` 分支中被访问，由于该变量并未被初始化，因此它将会是 `undefined`。

这常常需要花费JavaScript初学者一段时间去适应声明置顶这种特性。并且，误解这种特异行为将最终引入Bug。为此，ECMAScript 6引入了块级别作用域选项从而使得变量生命周期的控制变得更强大。

## 块级别声明（Block-Level Declarations）

块级别声明指的是那些声明变量无法在指定的代码块外访问的声明方式。代码块作用域，也有被称作语意作用域，创建于：

1. 在一个函数体内
1. 在一个代码块内（用 `{` 和 `}` 来限制）

块作用域是很多基于C语言的语言运行的方式，在ECMAScript 6中引入这个块级别声明是为了使JavaScript拥有这一特性的灵活性（以及统一性）。

### Let声明

`let` 声明语法跟 `var`一样。一般来说，你可以用 `let` 来代替 `var` 来声明一个变量，这样声明的变量将会被限制于当前的代码块（这里面还有一些细微的区别将在后续进行说明）。由于 `let` 声明并不会被置顶，因此你或许需要把 `let` 声明代码移到代码块顶部，这样才能使这些变量在整个代码块中有效。举个例子：

```js
function getValue(condition){
    
    if (condition) {
        let value = "blue";
        
        // other code 
        
        return value;
    } else {
        
        // value doesn't exist here 
        
        return null;
    } 
    
    // value doesn't exist here
}
```

这样的 `getValue` 函数将表现得更为贴近那些基于C的语言。由于变量 `value` 是通过 `let` 来声明的，这样的声明不会被置到该函数体，并且，该变量会在 `if` 代码块执行完后不可被访问。如果 `condition` 被判定为false时，那么 `value` 将不会被声明或被初始化。

### 变量不能重定义

如果一个变量标识已经在一个域内被定义，那么在同一个域内通过 `let` 去声明同一个标识将会引起错误并抛出异常。比如：

```js
var count = 30;

// Syntax error
let count = 40;
```

在这个例子中，`count` 被声明了两次：一次通过 `var` 来声明，另一次用 `let`。由于 `let` 不会对同一域中已经存在的标识符进行重定义，这段用 `let` 的声明将会抛出异常。此外，如果通过 `let` 在一个域的子域中定义一个同名的变量则不会产生错误，就如下面代码所示：

```js
var count = 30;

// Does not throw an error
if (condition) {
    
    let count = 40;
    
    // more code    
}
```

这样的 `let` 声明并不会抛出异常，因为它在 `if` 语句内创建了一个名为 `count` 的新变量，而非将 `count` 在外部创建。在 `if` 代码块中，新变量屏蔽掉全局变量 `count`以防止全局的 `count` 在这域内被访问，直到 `if` 代码块执行结束。

### 常量定义

在ECMAScript 6中，你也可以使用 `const` 声明语法来定义变量。使用 `const` 来声明的变量会被认为 *常量（constants）*，意思是这些变量的值一旦被设定将无法更改。因此，所有的 `count` 变量一定要在声明时进行初始化，如下面的例子所示：

```js
// Valid constant
const maxItems = 30;

// Syntax error: missing initialization
const name;
```

`maxItems` 变量是初始化过的，所以它的 `const` 声明不会有问题。然而，对于 `name` 变量，由于它并没有被初始化，在你尝试执行包含这段代码的脚本时，程序会出现一个语法错误。


### 常量与Let声明的对比

常量，跟 `let` 一样， 都是块级别声明。这意味着常量将会在他们定义的域执行完毕并跳出该域后将无法被访问，并且声明不会被置顶。下面是一个例子：

```js
if (condition) {
    const maxItems = 5;
    
    // more code
}

// maxItems isn't accessible here
```

在这段代码中，常量 `maxItems` 声明在 `if` 语句内。当这个语句执行完毕后，`maxItems` 在这个代码块外无发被访问。

另一个跟 `let` 相似的地方是，通过 `const` 去声明一个在同一域内已经存在的标识时会抛出异常。这种情况不管这个变量是通过 `var` （在全局或者一个函数域内）来定义或者通过 `let` （在代码块域内）来定义。举个例子：

```js
var message = "Hello!";
let age = 25;

// Each of these would throw an error.
const message = "Goodbye!";
const age = 30;
```

抛开其它代码来看，这两个 `const` 声明是有效的。但加上在这个案例中之前的 `var` 和 `let` 声明，那两个 `const` 声明都会出错。

除去上述的那些共同点，在 `let` 和 `const` 之间有一个很不一样的地方是需要谨记的：无论是在严格模式还是非严格模式中，当尝试去对一个已经定义了的常量进行赋值将会抛出异常。比如：

```js
const maxItems = 5;

maxItems = 6;   // throws error
```

跟许多在其它语言中的常量一样，`maxItems` 变量在被声明（初始化后）不能被赋予一个新的值。然其不同的是，如果这个常量是一个对象（Object）,它内部的值可以被修过。

#### 将对象声明为常量

一个 `const` 声明会阻止绑定的更改而非值的本身。这意味着 `const` 用于对对象的声明不会阻止对对象值的变更。举个例子：

```js
const person = {
    name: "Nicholas"
};

// works
person.name = "Greg";

// throws an error
person = {
    name: "Greg"    
};
```

这里，`person` 绑定是伴随着一个拥有一个属性的对象的初始化值被创建了。`person.name` 是可以被更改的，因为这只是更改了 `person` 所含有的内容而不是修改这个绑定在 `person` 上的值。当这段代码尝试去给 `person` 赋值（也就是试图修改该绑定），这时将会抛出异常。这大就是为什么 `const` 在与对象协作时很容易误解。只要记住一点：`const` 阻止绑定的修改，而不是所绑定的值的修改。

### 待定盲区（Temporal Dead Zone）

通过 `let` 和 `const` 声明的变量在它声明之前都不能被访问。若试图去访问将会导致一个引用错误，甚至当使用一些正常来说安全的操作方法，比如以下的一个例子：

```js
if (condition) {
    console.log(typeof value);  // ReferenceError!
    let value = "blue";    
}
```

这里，变量 `value` 通过 `let` 来定义并初始化，但这语句从不会被执行因为前一行会抛出异常。问题在于 变量 `value` 存在于JavaScript社区所称的 *待定盲区（temporal dead zone）*（TDZ）里。TDZ并没有在ECMAScript的说明文档中被明确命名，但这一说法（称呼）通常用于表述 `let` 和 `const` 的声明前无法访问性。这节里面涵盖了一些由TDZ引起的声明的巧妙放置方式。尽管在例子里用的都是 `let`，但这些信息同样可以应用到 `const` 上。

当一个JavaScript引擎扫描接下来的一个代码块并寻找一个变量声明时，它要么将其置顶（对于 `var` 而言）或者将其置于TDZ中（对于 `let` 和 `const`而言）。任何试图访问在TDZ中的变量时都会引起运行时错误。这些在TDZ中的变量只会在执行到其声明语句所在位置时才会被移出然后安全地使用。

这是真实的，在你试图在一个通过 `let` 或者 `const` 声明的变量被定义之前使用它的时候。正如之前的例子展示的那样，这甚至会应用在通常来说安全的 `typeof` 运算上。不过，你可以在变量声明的代码块外部对这个变量使用 `typeof` 运算，尽管它不会给你如愿的结果。且看下面这段代码：

```js
console.log(typeof value);      // "undefined"

if (condition) {
    let value = "blue";
}
```

在 `typeof` 运算执行时，由于该运算执行在变量 `value` 声明所在的代码块之外，所以这时变量 `value` 并不在TDZ内。这意味着 `value` 并不存在任何绑定，且 `typeof` 会返回 `undefined`。

TDZ只是块级别绑定中的其中一个独特的方面。另一个独特的地方则是在循环体内。

## 循环体中的块级别绑定

也许开发者最想块级别绑定能作用在变量上的地方就是在 `for` 循环体内了，这样循环计数器就意味着只会在循环中使用。举个例子，在JavaScript中这样的情况并非不常见：

```js
for (var i = 0; i < 10; i++) {
    process(item[i]);
}

// i is still accessible here 
console.log(i);                     // 10
```

在其它的编程语言中，块级别绑定时默认的。这个例子里除了变量 `i` 能在循环体外部被访问这点之外，其它部分都会如常工作。然而，在JavaScript里面，变量 `i` 由于 `var` 的声明置顶，它在循环结束后依然能够被访问。使用 `let` 去替代声明的话，则符合所有要求，如：

```js
for (let i = 0; i < 10; i++) {
    process(items[i]);
}

// i is not accessible here - throws an error
console.log(i);
```

在这个例子里，变量 `i` 仅存在于 `for` 循环内。一旦循环结束，该变量在其它地方不再能被访问。

### 循环体内的函数

由于 `var` 的特点，长期以来在函数体内创建函数都成了问题，因为循环体内的变量能够在外部被访问。且看如下代码：

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push(function () { console.log(i); });
}

funcs.forEach(function (func) {
   func();  // outputs the number "10" ten times 
});
```

按常理来说，你应该期望这段代码会在控制台中输出0到9，但却是在一行内输出十次数字10。那是因为 `i` 在每一次循环迭代中都是被共享的，意味着在函数体内创建的函数都保存这同一变量的引用。变量 `i` 在循环结束后的值时 `10`，所以当 `console.log(i)` 被调用时，每一次都会输出这个值。

为了修复这个问题，开发者通常在一个循环体内使用立即执行函数式（IIFEs）来强制复制创建那个迭代的变量，如以下例子所示：

```js
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push((function (value) {
        return function () {
            console.log(value);
        }        
    }(i)));
}

funcs.forEach(function (func) {
    func();     // outputs 0, then 1, then 2, up to 9 
});
```

这个方案用了函数体内的IIFE。变量 `i` 通过IIFE传递，这样的方式让 `i` 复制并创建一个新的值并存放在 `value` 里。这就是在当且迭代中被函数所使用的值，因此调用每一个函数会如期地返回0到9。幸好，在ECMAScript 6中的 `let` 和 `const` 的块级别绑定能帮你简化这个问题。

### 循环体内的Let声明


# Javascript

`Javascript`是一门弱、动态类型语言

动态类型指不需要指定变量的类型（通常没有类型检查）就可以随意给变量赋予任何类型的值

弱类型指不同类型的变量可以进行操作而不抛出错误（不同类型的变量可以进行相加操作）

![语言的分类](/assets/images/languages.jpg "编程语言的分类")

## Basic

### 继承

#### 原型链继承

```js
function SuperType() {
    this.property = true;
}
SuperType.prototype.getSuperValue = function() {
    return this.property;
}

function SubType() {
    this.subproperty = false;
}
// 重写子类的原型对象，代之以父类的实例，构建起子类实例到父类实例到父类实例原型的原型链
SubType.prototype = new SuperType(); 
SubType.prototype.getSubValue = function() {
    return this.subproperty;
}

var instance = new SubType();
console.log(instance.getSuperValue()); // true
```

缺点：

1. 子类原型被重写，其上的`constructor`属性丢失，随之带来的给子类原型添加方法和原型需要在重写之后
2. 多个实例对父类引用类型的操作会改写该属性

#### 借用构造函数继承

```js
function  SuperType(){
    this.color=["red","green","blue"];
}

function  SubType(){
    // 调用父类构造函数
    SuperType.call(this);
}

var instance1 = new SubType();
instance1.color.push("black");
alert(instance1.color);//"red,green,blue,black"
var instance2 = new SubType();
alert(instance2.color);//"red,green,blue"
```

缺点：

1. 无法继承原型属性和方法
2. 每个子类实例都会有父类函数的副本

#### 组合继承

本质是结合了原型链继承和借用构造函数继承两种方法

```js
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
  alert(this.name);
};

function SubType(name, age){
  SuperType.call(this, name);
  this.age = age;
}
SubType.prototype = new SuperType(); 
// 重写SubType.prototype的constructor属性，指向自己的构造函数SubType
SubType.prototype.constructor = SubType;
// 在重写原型后才能添加原型方法 
SubType.prototype.sayAge = function(){
    alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"
instance1.sayName(); //"Nicholas";
instance1.sayAge(); //29

var instance2 = new SubType("Greg", 27);
alert(instance2.colors); //"red,blue,green"
instance2.sayName(); //"Greg";
instance2.sayAge(); //27
```

缺点：

1. 第一次调用父类构造函数时，给子类原型写入了一次属性
2. 第二次调用父类构造函数时，给实例对象写入了一次属性，导致了属性屏蔽

#### 原型式继承

```js
// 实际上是Object.create方法
function object(obj){
  function F(){}
  // 与原型链继承的核心类似
  F.prototype = obj;
  return new F();
}

var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

console.log(person.friends);   //"Shelby,Court,Van,Rob,Barbie"
```

缺点：

1. 多个实例对父类引用类型的操作会改写该属性
2. 无法传递参数

#### 寄生式继承

```js
function createAnother(original){
  var clone = object(original); // 通过调用 object() 函数创建一个新对象
  clone.sayHi = function(){  // 以某种方式来增强对象
    alert("hi");
  };
  return clone;
}

var person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};
var anotherPerson = createAnother(person);
anotherPerson.sayHi(); //"hi"
```

缺点与原型式继承是一样的，这种方式是在原型式继承上的增强，添加了属性和方法

#### 寄生组合式继承

结合借用构造函数传递参数和寄生模式实现继承

```js
function inheritPrototype(subType, superType){
  // 原型式继承的核心，但是以父类原型作为副本创建新对象
  var prototype = Object.create(superType.prototype);
  prototype.constructor = subType;
  // 原型链继承的核心，但不用实例对象而是使用原型对象，避免多次实例化
  subType.prototype = prototype;
}

function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function(){
  alert(this.name);
};

// 借用构造函数
function SubType(name, age){
  SuperType.call(this, name);
  this.age = age;
}
inheritPrototype(SubType, SuperType);
SubType.prototype.sayAge = function(){
  alert(this.age);
}

var instance1 = new SubType("xyc", 23);
var instance2 = new SubType("lxy", 23);

instance1.colors.push("2"); // ["red", "blue", "green", "2"]
instance2.colors.push("3"); // ["red", "blue", "green", "3"]
```

> es6中的类继承`extends`的核心机制与寄生组合式继承是一致的

```js
function _inherits(subType, superType) {
    // 创建对象，创建父类原型的一个副本
    // 增强对象，弥补因重写原型而失去的默认的constructor 属性
    // 指定对象，将新创建的对象赋值给子类的原型
    subType.prototype = Object.create(superType && superType.prototype, {
        constructor: {
            value: subType,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    
    if (superType) {
        Object.setPrototypeOf 
            ? Object.setPrototypeOf(subType, superType) 
            : subType.__proto__ = superType;
    }
```

### 数据类型

type      | typeof return value | object wrapper
----------|---------------------|---------------
Null      | object              |
Undefined | undefined           |
Boolean   | boolean             | Boolean
Number    | number              | Number
BigInt    | bigint              | BigInt
String    | string              | String
Symbol    | symbol              | Symbol

#### 相等性与类型转换

<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#type_coercion>

<https://dorey.github.io/JavaScript-Equality-Table/>

在`A == B`比较表达式中，类型转换十分常见，当AB为不同类型的值时，需要进行类型转换

```js
function sameValueZero(x,y) {
    if(typeof x === "number" && typeof y === "number") {
        // x和y相等；正负0或者两者都是NaN的情况
        return x === y || (x !== x && y !== y)
    }
    return x === y;
}
```

x         | y         | ==    | ===   | Object.is | sameValueZero
----------|-----------|-------|-------|-----------|--------------
undefined | undefined | true  | true  | true      | true
null      | null      | true  | true  | true      | true
+0        | -0        | true  | true  | false     | true
+0        | 0         | true  | true  | true      | true
-0        | 0         | true  | true  | false     | true
null      | undefined | true  | false | false     | false
NaN       | NaN       | false | false | true      | true

![类型转化](/assets/images/type%20coercion.png "类型转化")

特别的，图中的N表示`ToNumber`操作，即将操作数转换为数字，可理解为`Number()`；P表示`ToPrimitive`操作，即将操作数转化为原始类型的值，这些操作都要优先执行

> `ToPrimitive(obj)`等价于：先计算`obj.valueOf()`，如果结果为原始值，则返回此结果；否则，计算`obj.toString()`，如果结果是原始值，则返回此结果；否则，抛出异常；对于日期类型则相反

* 左边的值都可以理解为存在的值，右边的值可以理解为空的值，当两者进行相等判断时，结果都是`false`
* 对于`null`和`undefined`，因为两者同属空的阵营所以他们的比较是`true`
* 布尔值可以转化为数字0与1，字符串也会转化为数字，可以隐约得出万物皆数字的感觉
* 对于`object`类型的值，首先需要转化为原始类型，再根据原始类型的规则进行判断

### 表达式与运算符

#### 表达式

#### 运算符

##### instanceof

`object instanceof constructor`用于检测`constructor.prototype`是否存在于被检测对象的原型链上

##### new

1. 创建一个空对象
2. 为新创建的对象添加属性`__proto__`，将该属性链接至构造函数的原型对象
3. 这个新对象会绑定到函数调用的`this`
4. 如果该函数没有返回对象，则返回该新对象

```js
function create() {
    // 创建一个空的对象
    var obj = new Object(),
    // 获得构造函数，argument对象的第一项就是构造函数
    Con = [].shift.call(arguments);
    // 链接到原型，obj 可以访问到构造函数原型中的属性；更推荐第二种写法
    // obj.__proto__ = Con.prototype;
    Object.setPrototypeOf(obj, Con.prototype);
    // 绑定 this 实现继承，obj 可以访问到构造函数中的属性
    var ret = Con.apply(obj, arguments);
    // 优先返回构造函数返回的对象
    return ret instanceof Object ? ret : obj;
};
```

##### 赋值表达式

形如`A = B`的表达式，其中A和B都可以是表达式，B可以是任意表达式，A必须是一个**左值**，即可以被赋值的表达式，在标准中用**引用**来描述，A可以是以下几种形式：

* 表达式`foo.bar`
* 变量`email`，特别指当前环境中的引用
* 函数名`func`

执行赋值表达式的步骤：

1. 计算表达式A和B，分别得到引用refA和值valueB
2. 将值赋给引用
3. 返回值

对于连等赋值表达式`A=B=C=D`，根据右结合性，可以看作是`A=(B=(C=D))`，再结合引擎对赋值表达式的执行步骤可以得出以下步骤：

1. 分别计算ABC得到各自的引用，再计算D得到值
2. 将值依次赋给CBA
3. 返回值valueD

可以总结为先解析引用和计算值，再把值从右到左赋值并返回

```js
var a = {n: 1};
var b = a;
// 把a.x换成b.x也是一样的
a.x = a = {n: 2};
alert(a.x); // --> undefined
alert(b.x); // --> {n: 2}
```

> 由于`a.x`优先解析，所以它指向的是对象`{n:1}`，实际是给这个对象的`x`属性赋值，`a = {n: 2}`是当前上下问中的引用`a`被赋值，在输出的时候，输出的是当前环境中的`a`，此时它没有`x`属性，所以结果是`undefined`，而`b.x`是对象`{n:1,x:{n:2}}`的属性，所以结果是`{n:2}`

### Build-in Objects

<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects>

#### Promise

#### Error

### Object

定义一个对象时会自动初始化一组属性，这些属性赋予了对象种种特性，分别是**数据属性**和**访问器属性**

特性         | 数据类型 | 描述                                                                           | 默认值
-------------|----------|------------------------------------------------------------------------------|----------
value        | 任何类型 | 包含这个属性的数据值                                                           | undefined
writable     | Boolean  | 如果该值为`false`，则`value`特性不可以被修改                                    | false
enumerable   | Boolean  | 如果该值为`true`，则可以用`forin`循环来枚举                                     | false
configurable | Boolean  | 如果该值为`false`，则该属性不能被删除，`value`和`writable`以外的特性都不能被修改 | false

特性         | 数据类型            | 描述                                                                          | 默认值
-------------|---------------------|-----------------------------------------------------------------------------|----------
get          | 函数对象或undefined | 包含这个属性的数据该函数使用一个空的参数列表，用于在有权访问的情况下读取属性值 | undefined
set          | 函数对象或undefined | 该函数有一个参数，用于在有访问权的情况下写入属性值                             | false
enumerable   | Boolean             | 如果该值为`true`，则可以用`forin`循环来枚举                                    | false
configurable | Boolean             | 如果该值为`false`，则该属性不能被删除，且不能转换为数据属性                     | false

#### 枚举属性以及查询和遍历方法

查询方法

方式                   | 可枚举的自身属性 | 可枚举的继承属性 | 不可枚举的自身属性 | 不可枚举的继承属性
-----------------------|------------------|------------------|--------------------|----------
propertyIsEnumerable() | true             | false            | false              | false
hasOwnProperty()       | true             | false            | true               | false
Object.hasOwn()        | true             | false            | true               | false
in                     | true             | true             | true               | true

遍历方法

方式                                     | 可枚举的自身属性 | 可枚举的继承属性 | 不可枚举的自身属性 | 不可枚举的继承属性
-----------------------------------------|------------------|------------------|--------------------|----------
Object.keys\Object.values\Object.entries | true(string)     | false            | false              | false
Object.getOwnPropertyNames               | true(string)     | false            | true(string)       | false
Object.getOwnPropertySymbols             | true(symbols)    | false            | true(symbols)      | false
Object.getOwnPropertyDescriptors         | true             | false            | true               | false
Reflect.ownKeys                          | true             | false            | true               | false
for in                                   | true(string)     | true(string)     | false              | false
Object.assign                            | true             | false            | false              | false
...                                      | true             | false            | false              | false

#### Array & Typed Array

类型化数组于数组不同，前者使用`Array.isArray`会返回`false`，并且并不支持所有的数组方法。用于操作原始的二进制数据，其中每一个元素都是原始的二进制数据。它由缓冲（由`ArrayBuffer`实现）和视图两部分组成，在缓存中的数据需要通过视图进行读取和操作

使用类型化数组的浏览器API：

* webgl
* canvas2d imagedata

```js
var imageData = ctx.getImageData(0,0, 200, 100);
var typedArray = imageData.data // data is a Uint8ClampedArray
```

* XMLHttpRequest2

```js
xhr.responseType = 'arraybuffer';
xhr.send();
```

* reader.readAsArrayBuffer(file)
* webworker.webkitPostMessage
* video.webkitSourceAppend
* socket.binaryType = 'arraybuffer'

#### 深浅拷贝

浅拷贝：根据给定对象创建一个副本，这个副本对象有着原对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是地址，当其中一个对象改变了这个地址，就会影响到另一个对象

* 使用`Object.assign()`方法将一个或多个对象的可枚举自身属性赋值到目标对象
* 展开语法
* 数组方法`slice`和`concat`

深拷贝：区别于浅拷贝，当拷贝引用类型时会创建一个新的副本并赋予新的地址，这使得两者互不干扰。

* `JSON.parse(JSON.stringify(object))`，该方案会有以下缺陷，处理`undefined`，`symbol`和函数时会直接省略，处理循环引用会报错，处理`new Date()`时转换结果不准确，处理正则时会转换为`{}`空对象
* 第三方工具库`lodash.cloneDeep`，`jQuery.extend`
* 新的webapi-`structuredClone()`

##### 实现深拷贝

* 深拷贝可以视为浅拷贝和递归的结合，根据属性的性质进行判断，如果是基础类型就浅拷贝，否则就使用递归，依次对该引用类型属性的对象再进行一次相同操作

```js
function deepClone(obj) {
    // 对于基础类型的属性，直接返回即可
    if( obj === null || typeof obj !== 'object' ){
        return obj;
    }

    // 使用构造函数再创建一个同类型的对象
    const temp = new obj.constructor();
    // 如果不需要原型链上的属性则可以使用Object.keys来遍历
    for(let key in obj){
        // 对于引用类型的属性，需要再次进行递归
        temp[key] = deepClone(obj[key])
    }
    return temp;
}
```

* 对于集合类型的数据如`set`和`map`，需要遍历数据并进行拷贝，这类数据使用构造函数进行拷贝，它仍然是浅拷贝；对于正则`regexp`数据，则需要传递原始数据到构造函数中

```js
{
    if(obj instanceof Set){
        const temp = new Set();
        obj.forEach(item => {
            temp.add(deepClone(item))
        })
        return temp;
    }else if(obj instanceof Map){
        const temp = new Map();
        obj.forEach((item,key) => {
            temp.set(key,deepClone(item))
        })
        return temp;
    }else if(obj instanceof RegExp){
        // 传递原始数据到构造函数中
        const temp = new RegExp(obj);
        return temp;
    }else{
        // 接上第一部分的代码 const temp = new obj.constructor();
        // ...
    }
}
```

* 通过保存一份已遍历记录，保存已经遍历过的属性，对每一次遍历都去检测是否已经遍历过以解决循环引用和引用丢失的问题

```js
var obj1 = {};
var obj2 = {a: obj1, b: obj1};
obj2.a === obj2.b; // true
var obj3 = deepClone(obj2);
obj3.a === obj3.b; // false 引用丢失问题

function deepClone(obj) {
    // 省略if判断

    // 使用weakmap维护缓存并保存到函数中
    let cache = null;
    if(!deepClone.cache) {
        deepClone.cache = new WeakMap();
    }
    cache = deepClone.cache;
    // 对于已有的对象则直接返回
    if(cache.has(obj)){
        return obj;
    }

    if(obj instanceof Set){
        // const temp = new Set();
        cache.set(obj,temp)
    }else if(obj instanceof Map){
        // const temp = new Map();
        cache.set(obj,temp)
    }else if(obj instanceof RegExp){
        // 正则不需要
    }else{
        cache.set(obj,temp)
    }
}
```

* 如果对象嵌套层级过深有可能会出现递归爆栈的现象，我们可以使用栈结构和循环解决这个问题

```js
    function deepClone(obj) {
        let root;
        const stack = [{
            parent: null,
            key: undefined,
            data: obj,
        }];

        while (stack.length) {
            const node = stack.pop();
            const parent = node.parent;
            const key = node.key;
            const data = node.data;
            let res = parent;

            if (data === null || typeof data !== 'object') {
                if (key === undefined) {
                    root = data;
                }
                res[key] = data;
            } else {
                if (key === undefined) {
                    res = new data.constructor();
                    root = res;
                } else {
                    res[key] = new data.constructor();
                    res = res[key];
                }
                for (let dataKey in data) {
                    stack.push({
                        parent: res,
                        key: dataKey,
                        data: data[dataKey],
                    });
                }
            }
        }

        return root;
    }
```

#### Prototype

> Javascript是基于原型的语言

对象都拥有一个原型对象，对象以其原型为模板，从原型继承方法和属性，这些属性定义在对象的构造函数的`prototype`属性上，而非对象本身

![原型链](/assets/images/prototype%20chain.jpg "原型链图解")

对象的原型对象可以通过`__proto__`指针查找，原型对象本身也有原型，也可以通过这个指针查找到它的原型对象。通过一层一层的追溯，最终指向`null`从而形成类似链条的形状，我们称之为**原型链**

![原型](/assets/images/prototype%20and%20__proto__%20and%20constructor.jpg "原型图解")

> `__proto__`是实例对象上的属性，而`prototype`是构造函数的属性，它们是不同的，但`p.__proto__`和`P.prototype`指向同一对象

在es6标准中，`__proto__`不推荐使用，推荐使用`Object.getPrototypeOf`和`Object.setPrototypeOf`方法

##### Constructor

`constructor`属性返回对象的构造函数，这个属性的指是对函数本身的引用，而不是一个包含函数名称的字符串

更改基本类型的`constructor`并不会产生任何影响，更改引用类型的`constructor`通常用于解决继承问题，另外修改`constructor`属性不会影响`instanceof`运算符

### Function

函数在js中是一等公民

1. 函数可以作为值赋予变量
2. 函数可以作为函数的参数进行传递
3. 可以从函数中返回函数作为函数的返回值

#### FP

![FP](/assets/images/fp.jpg "FP")

##### currying && partial application

所谓"柯里化"，就是把一个多参数的函数，转化为单参数函数,依次调用，直至传入所有必须参数才会得到最终结果

```js
var curry = function (fn) {
    var limit = fn.length
    return function judgeCurry(...args) {
        if (args.length >= limit) {
            return fn.apply(null, args)
        } else {
            return function (...args2) {
                return judgeCurry.apply(null, args.concat(args2))
            }
        }
    }
}

function add(x, y, c) {
  return x + y + c;
}
let curryAdd = curry(add)
curryAdd(1)(2)(3) // 6
```

偏函数与柯里化类似，都是高阶函数，都将多元函数转换为低元函数，偏函数只返回一次接受剩余参数的函数，柯里化可以看作是自动化的偏函数应用

```js
function add(x, y, c) {
  return x + y + c;
}

// bind就是典型的偏函数
const partialAdd1 = add.bind(null,1);
partialAdd1(2,3); // 6

// 偏导函数
const partial = (fn, ...args) => (...rest) => fn.apply(null, args.concat(rest))
const partialAdd3 = partial(add,3);
partialAdd3(1,1); // 5
```

##### compose & pipe

![组合函数](/assets/images/pipe%20function.jpg "组合函数")

我们将多个函数串流起来，把输入传入第一个函数，第一个函数执行返回的结果作为第二个函数的输入，依次进行直到最后一个函数完成得到结果

我们把接收一系列函数，并将这些函数以某种顺序串联起来执行的函数称为组合或者管道函数

```js
const multiply20 = (price) => price * 20;
const divide100 = (price) => price / 100;
const normalizePrice = (price) => price.toFixed(2);
const addPrefix = (price) => "$" + String(price);

const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((res, fn) => fn(res), x);
const compose =
  (...fns) =>
  (x) =>
    fns.reduceRight((res, fn) => fn(res), x);

// const pipe = (...fns) => fns.reduce((f,g) => (...args) => g(f(...args)))
// const compose = (...fns) => fns.reduce((f,g) => (...args) => f(g(...args)))


const discountPipe = pipe(multiply20, divide100, normalizePrice, addPrefix);
const discountCompose = compose(
  addPrefix,
  normalizePrice,
  divide100,
  multiply20
);

discountPipe(200); // '$40.00'
discountCompose(200); // '$40.00'
```

#### 节流与防抖

![节流和防抖](/assets/images/debounce%20and%20throttle.jpg "节流和防抖")

#### lodash实现分析

* 总体概览：

```js
function debounce(func, wait, options) {
  let lastArgs, // 上一次执行 debounced 的 arguments，
    lastThis, // 保存上一次 this
    maxWait, // 最大等待时间，实现节流效果，保证大于一定时间后一定能执行
    result, // 函数 func 执行后的返回值，多次触发但未满足执行 func 条件时，返回 result
    timerId,
    lastCallTime // 上一次调用 debounce 的时间

  let lastInvokeTime = 0 // 上一次执行 func 的时间
  let leading = false // 是否响应事件刚开始的那次回调
  let maxing = false // 是否有最大等待时间，配合 maxWait 多用于节流相关
  let trailing = true // 是否响应事件结束后的那次回调

  // 不传入wait则使用requestAnimationFrame
  const useRAF = (!wait && wait !== 0 && typeof root.requestAnimationFrame === 'function')

  if (typeof func !== 'function') {
    throw new TypeError('Expected a function')
  }
  
  wait = +wait || 0
  
  if (isObject(options)) {
    leading = !!options.leading
    // options 中是否有 maxWait 属性，节流函数预留
    maxing = 'maxWait' in options
    // maxWait 为设置的 maxWait 和 wait 中最大的，如果 maxWait 小于 wait，那 maxWait 就没有意义了
    maxWait = maxing ? Math.max(+options.maxWait || 0, wait) : maxWait
    trailing = 'trailing' in options ? !!options.trailing : trailing
  }
  
  // 开启定时器
  function startTimer(pendingFunc, wait) {}

  // 取消定时器
  function cancelTimer(id) {}

  // 定时器回调函数，表示定时结束后的操作
  function timerExpired() {}
  
  // 计算仍需等待的时间
  function remainingWait(time) {}
  
  // 执行连续事件刚开始的那次回调
  function leadingEdge(time) {}
  
  // 执行连续事件结束后的那次回调
  function trailingEdge(time) {}

  // 执行 func 函数
  function invokeFunc(time) {}

  // 判断此时是否应该执行 func 函数
  function shouldInvoke(time) {}

  // 取消函数延迟执行
  function cancel() {}
  // 立即执行 func
  function flush() {}
  // 检查当前是否在计时中
  function pending() {}

  // 入口函数
  function debounced(...args) {}
  // 绑定方法
  debounced.cancel = cancel
  debounced.flush = flush
  debounced.pending = pending

  return debounced
}
```

* 入口函数：

```js
function debounced(...args) {
  const time = Date.now()
  // 判断此时是否应该执行 func 函数
  const isInvoking = shouldInvoke(time)

  lastArgs = args
  lastThis = this
  lastCallTime = time

  // 执行
  if (isInvoking) {
    // 无 timerId 的情况有两种：
    // 1、首次调用 
    // 2、trailingEdge 执行过函数(trailingEdge函数会清除timer)
    if (timerId === undefined) {
      return leadingEdge(lastCallTime)
    }
    
    // 定时器存在并且设置了maxing
    // 考虑这种情况：wait为200，maxwait也是200，当前时间为200，上次执行时间为0，上次触发为150，timer存在。在此情况下，满足timeSinceLastInvoke为200的情况，需要立即执行
    if (maxing) {
      // 开启新定时器并执行回调
      timerId = startTimer(timerExpired, wait)
      return invokeFunc(lastCallTime)
    }
  }
  // 一种特殊情况，trailing 设置为 true 时，前一个 wait 的 trailingEdge 已经执行了函数
  // 此时函数被调用时 shouldInvoke 返回 false，所以要开启定时器
  if (timerId === undefined) {
    timerId = startTimer(timerExpired, wait)
  }
  // 不需要执行时，返回结果
  return result
}
```

* startTimer本质就是开启定时器，可以视为`setTimeout`这一个函数；省略cancelTimer函数不进行介绍

```js
function startTimer(pendingFunc, wait) {
  // 使用 RAF
  if (useRAF) {
    root.cancelAnimationFrame(timerId);
    return root.requestAnimationFrame(pendingFunc)
  }
  return setTimeout(pendingFunc, wait)
}
```

* timerExpired函数决定最终回调函数是否执行，在该函数中对于未达到执行条件的情况，直接重启当前定时器，而非取消再创建定时器，节省了浏览器资源，我们在实现自己的节流防抖函数可以使用这个技巧

```js
function timerExpired() {
  const time = Date.now()
  if (shouldInvoke(time)) {
    return trailingEdge(time)
  }
  timerId = startTimer(timerExpired, remainingWait(time))
}
```

* remainingWait函数确定剩余时间，包括节流和防抖两种间隔时间

```js
function remainingWait(time) {
  // 调用 debounce 的时间差
  const timeSinceLastCall = time - lastCallTime
  // 执行 func 的时间差
  const timeSinceLastInvoke = time - lastInvokeTime
  // 剩余等待时间，这个时间是防抖需要的时间
  const timeWaiting = wait - timeSinceLastCall

  // 对于节流：返回「剩余等待时间」和「执行 func 时间差」中的最小值
  return maxing
    ? Math.min(timeWaiting, maxWait - timeSinceLastInvoke)
    : timeWaiting
}
```

* shouldInvoke函数分别确定了四种需要执行回调的情况：第一次调用，超过wait指定的时间，当前时间小于上一次调用时间意味着系统时间做了更改和超过最大等待时间

```js
function shouldInvoke(time) {
  const timeSinceLastCall = time - lastCallTime
  const timeSinceLastInvoke = time - lastInvokeTime
  return (lastCallTime === undefined || (timeSinceLastCall >= wait) ||
          (timeSinceLastCall < 0) || (maxing && timeSinceLastInvoke >= maxWait))
}
```

* leadingEdge函数

```js
function leadingEdge(time) {
  // 记录时间
  lastInvokeTime = time
  // 开启定时器，为了trailingEdge
  timerId = startTimer(timerExpired, wait)
  // 根据配置返回相应结果
  return leading ? invokeFunc(time) : result
}
```

* trailingEdge函数

```js
function trailingEdge(time) {
  // 清空定时器
  timerId = undefined

  // trailing 和 lastArgs 两者同时存在时执行
  // lastArgs 标记位的作用，意味着 debounce 至少执行过一次
  if (trailing && lastArgs) {
    return invokeFunc(time)
  }
  // 清空参数
  lastArgs = lastThis = undefined
  return result
}
```

* invokeFunc函数

```js
function invokeFunc(time) {
  // 获取上一次执行参数并做重置
  const args = lastArgs
  const thisArg = lastThis
  lastArgs = lastThis = undefined
  // 记录时间
  lastInvokeTime = time
  result = func.apply(thisArg, args)
  return result
}
```

### Module

#### 发展史

* 省略刀耕火种时期

* module pattern

```js
// IIFE创建了一个函数scope，在这里可以视为module scope
// 模块名
var module = (function () {

    var exportValue;
    var exportFunction;

    // 导出
    return {
        exportValue,
        exportFunction
    }

})();
```

使用module pattern的这一部分代码是自治的，它向全局环境暴露了模块名，导出了变量拱其它模块使用，通过IIFE的参数传入需要的全局变量的引用以避免不经意间改写全局变量

缺点：该模式并没有根除全局变量污染问题，仍未解决不同代码之间的依赖管理问题

* CommonJS

多用于服务端，模块的加载是同步的，意味着需要等待模块加载完毕，才会进行下一个模块的加载，不适用于浏览器环境

```js
var module1 = require('moudle1');

function fnInsideModule1() {

}

module.exports = {
  fnInsideModule1
}
```

* AMD - asynchronous module definition

为浏览器设计，它是异步的

```js
define(['module1','module2'],
  function (module1Import,module2Import) {
    var module1 = module1Import;
    var module2 = module2Import;

    function fn(){}

    return {
      fn
    }
  }
);
```

* UMD - universal module definition: 它识别代码环境所支持的模块系统类型，然后运行那一种模块类型代码

* ES6 Module

## Javascript Runtime

![运行时](/assets/images/runtime.jpg "运行时")

运行时包含一组执行上下文的集合、执行上下文栈、主线程、一组可能的worker线程、一个任务队列以及一个微任务队列。它是由事件循环进行驱动的

### Javascript Engine

![JIT引擎流程图](/assets/images/engine.jpg "JIT引擎流程")

1. Parse 进行词法分析-lexical analysis，根据keyword将源代码转换为token
2. AST-abstract syntax tree将tokens转化为AST结构
3. Interpreter 解释器将逐行读取和翻译源代码并执行生成的bytecode[^1]
4. Profiler 找出源代码中潜在的可能优化的代码片段并交给JIT compiler
5. Compiler 编译器 先将代码用某种低级语言比如machine code进行重写，在执行的时候直接使用更低级的语言而非原语言并且不需要再次翻译

### Event Loop

![宏任务事件循环](/assets/images/event%20loop%20macro%20queue.jpg "宏任务事件循环")

![事件循环](/assets/images/event%20loop%20micro%20queue.jpg "事件循环")

事件循环负责收集事件（包括用户事件以及其他非用户事件等）、对任务进行调度以便在合适的时候执行回调。然后它执行所有处于等待中的任务（宏任务），然后是微任务[^2]，然后在开始下一次循环之前执行一些必要的渲染和绘制操作

每个宏任务执行完成并且执行上下文为空即call stack为空时，引擎会立即执行微任务队列中的所有任务，**即使中途有微任务被加入到队列中**，然后再执行其他的宏任务，或渲染，或进行其他任何操作

#### 实际应用

##### 拆分CPU密集任务

当浏览器运行CPU密集型的任务时，它就无法处理其它工作，比如用户点击页面上的一些可交互元素，以下代码会使得浏览器挂起一小段事件：

```js
let i = 0;
let start = Date.now();
function count() {
  for (let j = 0; j < 1e9; j++) {
    i++;
  }
  alert("Done in " + (Date.now() - start) + 'ms');
}
count();
```

我们可以使用`settimeout`来拆分这个任务，使得浏览器有处理其它任务的时间：

```js
let i = 0;
let start = Date.now();
function count() {
  // 做任务的一部分
  do {
    i++;
  } while (i % 1e6 != 0);

  if (i == 1e9) {
    alert("Done in " + (Date.now() - start) + 'ms');
  } else {
    setTimeout(count); // 安排新的调用
  }
}
count();
```

对于`settimeout`而言，如果我们事先知道需要调用多次任务，那么我们将任务提前进行调度以缩短任务时间

```js
function count() {
  // 将调度移动到开头
  if (i < 1e9 - 1e6) {
    setTimeout(count);
  }

  do {
    i++;
  } while (i % 1e6 != 0);

  if (i == 1e9) {
    alert("Done in " + (Date.now() - start) + 'ms');
  }
}
```

由于我们提前安排了工作，也就意味着多个嵌套定时器的延迟可以得到优化，使得任务执行更加紧密，缩短了等待的时间

##### 进度指示

浏览器会在当前运行的任务完成后，才会进行绘制操作，如果我们想要对某个任务进行进度追踪，我们也可以使用类似的办法进行拆分

以下代码会在任务完成后更新dom节点中的数字：

```html
<div id="progress"></div>

<script>
  function count() {
    for (let i = 0; i < 1e6; i++) {
      i++;
      progress.innerHTML = i;
    }
  }
  count();
</script>
```

以下代码会在完成一部分任务后更新dom节点中的数字：

```html
<div id="progress"></div>

<script>
  let i = 0;
  function count() {
    do {
      i++;
      progress.innerHTML = i;
    } while (i % 1e3 != 0);

    if (i < 1e7) {
      setTimeout(count);
    }
  }
  count();
</script>
```

##### 保证条件性使用`promises`时的顺序

在条件判断语句中使用`promise`时，可能会带来执行顺序不一致的情况，考虑以下代码：

```js
customElement.prototype.getData = url => {
  if (this.cache[url]) {
    this.data = this.cache[url];
    this.dispatchEvent(new Event("load"));
  } else {
    fetch(url).then(result => result.arrayBuffer()).then(data => {
      this.cache[url] = data;
      this.data = data;
      this.dispatchEvent(new Event("load"));
    )};
  }
};

element.addEventListener("load", () => console.log("Loaded data"));
console.log("Fetching data...");
element.getData();
console.log("Data fetched");
```

连续执行两次这段代码会形成以下结果：

数据未缓存|数据已缓存|
-|-|
Fetching data - Data fetched - Loaded data | Fetching data - Loaded data - Data fetched

由于`then`方法是微任务，所以我们需要在`if`分支也保证同样是微任务：

```js
customElement.prototype.getData = url => {
  if (this.cache[url]) {
    // 使用queueMicrotask调度if分支的任务
    queueMicrotask(() => {
      this.data = this.cache[url];
      this.dispatchEvent(new Event("load"));
    });
  } else {
    fetch(url).then(result => result.arrayBuffer()).then(data => {
      this.cache[url] = data;
      this.data = data;
      this.dispatchEvent(new Event("load"));
    )};
  }
};
```

##### 批量操作

使用微任务将多个来源的请求集中到一个任务进行处理可以避免对同类工作进行多次调用导致的开销

考虑一个消息队列，我们会收集信息并在合适的时候发送消息：

```js
const messageQueue = [];

let sendMessage = message => {
  messageQueue.push(message);

  if (messageQueue.length === 1) {
    queueMicrotask(() => {
      const json = JSON.stringify(messageQueue);
      messageQueue.length = 0;
      fetch("url-of-receiver", json);
    });
  }
};
```

调用`sendMessage`函数，向数组添加消息，当第一条消息进入数组时，马上调度一个微任务。在后续调用`sendMessage`函数时，仍然将消息添加到数组中但不调度新的微任务，一旦执行微任务时，消息数组中会保存着本次事件循环所有推入的消息

### Executing Context

![执行上下文栈](/assets/images/executing%20context%20stack.jpg "执行上下文栈")

执行上下文可以分为全局执行上下文、函数执行上下文和`eval`执行上下文

创建执行上下文包括两个阶段：

1. This Binding、LexicalEnvironment、VariableEnvironment和Hoisting
2. 执行上下文中的代码

#### This

在全局上下文指向全局对象

在函数上下文取决于函数被调用的方式，在严格模式下，如果进入执行环境时没有设置`this`的值，`this`会保持为`undefined`

1. 函数作为对象的方法被调用那么`this`的值就是该对象
2. 原型链方法中的`this`指向调用该方法的对象
3. `getter`和`setter`中的`this`同样绑定到设置或获取属性的对象
4. 作为事件处理函数时，`this`指向触发事件的元素；作为内联处理函数时，`this`指向监听器所在的DOM元素：
5. 在箭头函数中，`this`与封闭的词法环境的`this`保持一致，即在创建时被设置，使用`call`、`apply`和`bind`等方式更改绑定会被忽略掉，但是仍然可以使用这些方式来改变参数并调用
6. 在`class`基类构造函数中，`this`指向正在构造的新对象；**派生类的构造函数没有初始`this`绑定**，需要通过调用`super`函数进行`this`的绑定。在类的方法中，`this`的指向与函数中的`this`类似，取决于被调用的方式

```html
<button onclick="alert(this.tagName);">
    show button
</button>
<button onclick="alert((function(){return this})());">
    show inner this - global window object 
</button>
```

##### this绑定的几种方式

1. 使用`call`和`apply`会立即执行函数，区别在于参数传入的方式不同，前者需要依次传入参数，后者只需要传入一个包含所有参数的数组或类数组对象比如`arguments`
2. 使用`bind`方法产生一个函数副本，该副本具有相同的函数体和作用域，该副本的this被永久设置且无法改变（所以第二次绑定是无效的）

#### Lexical Environment

词法环境由环境（存储变量和函数声明的实际位置）和对外部环境的引用-scope chain（通过该引用可以访问到外部的词法环境）组成

在全局环境中，由于全局环境不存在外部环境所以外部引用为`null`，拥有`global`对象和用户定义的全局变量；在函数环境中，拥有在函数中定义的变量以及`arguments`对象，外部应用可以是全局环境或者其它函数环境

##### Scope Chain

![scope chain](/assets/images/scope%20chain.jpg "作用域链")

当访问一个变量时，解释器会先在当前作用域查找该变量，如果没有找到，则向上一层到父级作用域中查找，直到找到该变量，如果变量不存在则会抛出错误

在浏览器控制台中，可以查看对象的scopes值：
![scope值](/assets/images/internal%20scope%20value.jpg "控制台中的scope值")

##### Closures

> A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment) - MDN

闭包是《表达式的机器执行》论文当中提出的计算机编程概念，它包含环境和控制两部分，在`js`中以函数能够访问其定义时的环境中变量的方式得以实现

闭包定义时，其执行上下文的作用域（scope chain）会将外部环境引用保存下来。当闭包中的外层函数执行上下文从栈中弹出，其执行上下文被摧毁，继续调用内函数时，其作用域链中的外层函数的活动对象仍留存在内存中，这使得内函数仍然能访问到外层函数的活动对象中的变量

#### Variable Environment

变量环境和词法环境相似，它包含词法环境中的所有属性。在`ES6`中，词法环境和变量环境的区别在于前者用于存储函数声明和变量（`let`和`const`）绑定，而后者仅用于存储变量（`var`）绑定

##### Variable object & Activation object

变量对象包含在变量环境中，并不能在代码中直接访问；当进入一个执行上下文中，变量对象被激活，这时我们称变量对象为活动对象，该活动对象里的属性才能够被访问到

#### Hoisting

在全局执行上下文的创建阶段，解释器会将变量、函数和类的定义移动到当前作用域顶层。这个概念不是ECMA规定的标准

##### Temporal Dead Zone

一个代码块从开始到代码执行声明变量的行之前，`let`或`const`声明的变量都处于`TDZ`之中，尝试访问这些变量会抛出引用错误

特别的，该行为的产生取决于执行顺序（时间）而非编写代码的顺序（位置）

```js
{
    // TDZ begin
    const func = () => console.log(letVar);

    let letVar = 3; // TDZ end

    func(); // called outside TDZ
}
```

## Memory

在`js`中，基本类型变量保存在栈中，引用类型的变量保存在堆中，由于引用对象的大小不是固定的，所以将其地址保存在栈中，再由该地址指向堆中的具体位置

### 内存管理

分配内存空间、使用分配到的内存空间和释放不再使用的内存空间是`JavaScript`的内存声明周期

`JavaScript`是一种垃圾回收语言。垃圾回收语言通过周期性地检查先前分配的内存是否可达，帮助开发者管理内存。换言之，垃圾回收语言减轻了“内存仍可用”及“内存仍可达”的问题。两者的区别是微妙而重要的：仅有开发者了解哪些内存在将来仍会使用，而不可达内存通过算法确定和标记，适时被操作系统回收

大部分垃圾回收语言用的算法称之为Mark-and-sweep,算法由以下几步组成：

1. 创建了一个root列表，通常是代码中全局变量的引用。通常是`global`对象，`window`对象总是存在，因此垃圾回收器可以检查它和它的所有子对象是否存在（即不是垃圾）
2. 所有的root被检查和标记为激活（即不是垃圾），所有的子对象也被递归地检查。从 root 开始的所有对象如果是可达的，它就不被当作垃圾
3. 所有未被标记的内存会被当做垃圾，开始释放内存归还给操作系统了

### 内存泄漏

不需要的引用是保留在代码中的变量，它不再需要，却指向一块本该被释放的内存

常见的内存泄露：

* 意外的全局变量

```js
function foo(arg) {
    bar = "this is a hidden global variable";
}

function foo() {
    this.variable = "potential accidental global";
}
foo();
```

* 被遗忘的计时器或回调函数，特别是事件处理函数
* 脱离DOM的引用

```js
var elements = {
    button: document.getElementById('button'),
    image: document.getElementById('image'),
    text: document.getElementById('text')
};
```

* 闭包

[^1]: ![bytecode](/assets/images/language%20for%20humans%20and%20machines.jpg "bytecode")
[^2]: microtask是v8引擎的术语，在ECMA标准中，它的描述是PromiseJobs

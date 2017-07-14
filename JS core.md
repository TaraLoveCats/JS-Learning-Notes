## JavaScript Scope

> 1.    *Scope* : the set of rules that govern how the Engine can look up a variable by its identifier name and find it, either in the current Scope, or in any of the *Nested Scopes（嵌套作用域）* it's contained within.
> 2.    JS中，**定义**函数时会创建函数对象的一个内部属性——*`[[scope]]`*,通常只有JS引擎能访问。这个内部属性包含函数被**定义**时作用域中对象的集合，这个集合被称为函数的作用域链 *（scope chain）*，它决定了哪些数据可以被函数访问。`[[scope]]`最终指向一个全局对象，其中包含了所有的全局变量。
> 3.    执行函数时，会创建一个执行上下文 *（execution context）* 的内部对象，定义了函数执行时的环境。每个执行上下文都有自己的作用域链，用于标识符解析，且被初始化为函数`[[scope]]`所指向的对象。即， **“JS中的函数运行在它们被定义的作用域中，而不是执行的作用域里。”** 这些值按照它们出现在函数中的顺序被复制到执行上下文的作用域链中，组成了一个活动对象 *（activation object）*。活动对象包含函数所有的局部变量、内部函数、命名参数、arguments和this（作为同名属性），然后此对象被推入作用域链的最前端。执行上下文被销毁时，活动对象也随之销毁。
> 4.    **注意**：JS is in fact a compiled language. 在执行**每段**代码前，都会处理var和function定义式。活动对象中的同名局部变量属性，其值在execution时才计算，执行前赋值为undefined。

> *图解及性能优化：[理解JS作用域和作用域链][1]* <br>
> *代码例子：[JS作用域原理][2]*

## JavaScript Closures

> 1.    闭包是一个运行期的概念，当函数a内部的函数b被函数a *外部* 的变量引用（直接赋值或者值（参数）传递）时，闭包就被创建了。闭包是一个段落，它保存了**一份**从上一级作用域取得的变量（键值对），这些变量不会随着上一级函数的执行完毕（返回）而被销毁，可以简单的理解为b依赖a，所以b在内存中时，a也不会被garbage collection回收。一个闭包就是函数返回后，一个没有被释放的栈区。
> 2.    闭包在过程中以环境的形式包含了数据。闭包主要有两个作用：封装变量和延长局部变量的生存期。
> 3.    JS中的garbage collection一般采用标记清楚策略，当变量不再处于环境中且没有其他在环境中的变量引用时，就被视为将要删除变量。但在IE中，BOM和DOM是使用C++以COM对象的方式实现的，COM对象的garbage collection采用的是引用计数。如果两个对象之间有循环引用，就无法释放内存了，造成内存泄漏。这时可以将变量设置为null，切断它与引用对象的联系。使用闭包比较容易造成这种问题（如果闭包的作用域链中包含着DOM节点的话），但本质上这不是闭包的问题。
> 4.    闭包使一些局部变量无法被及时回收，这跟把它们放在全局作用域中一样的。我们可以手动设置null来回收这些变量。这里并不能说成是内存泄漏。

> *看最后2个例子：[学习JS闭包][3]*<br>
> *中间“闭包的微观世界”（详尽的作用域解释）：[JS闭包深入理解][4]*<br>
> *最后5个例子：[理解JS闭包][5]*

## JavaScript This

> 1. this是函数执行时，自动生成的一个对象，只能在函数内部使用。this是运行期绑定的，指向函数**执行时**的当前对象。
> 2. 除了eval()和with语句，this的含义大致分为以下4种：

> * 作为对象的方法调用：this指向该对象
> * 作为普通函数调用：this总是指向global object（在浏览器中通常是window对象）。**在浏览器中setTimeout、setInterval和匿名函数的this均指向全局对象，其实是作为普通函数调用的特殊情况**。*注意匿名函数会用在setTimeout、setInterval和各种回掉函数（事件处理程序等等）之中！* 此时我们可以在这些函数之外用`var that = this;`来保存想要的环境。
> * 作为构造函数调用：this会绑定到新创建的对象（实例）之上。如`var ins = new Func();`
> * Function.prototype.call或Function.prototype.apply调用：this就指向apply或call中的第一个参数，即函数执行环境。Function.prototype.bind() （ES5）也如此。

> 3. eval()：this指向此函数的调用者的执行环境。with语句的this指向即with规定的作用域。
> 4. 函数调用时，会创建一个执行环境，函数所有的行为在此context中发生。创建执行环境的最后一步是根据不同的规则为this变量赋值。

> *“函数执行环境”详尽解释：[深入浅出JS中的this][6]*<br>
> *代码例子：[JS中的this关键字详解][7]*

## JavaScript Prototypes

> 1.    在JS中，一切引用类型都是对象，对象是属性（无序键值对）的集合。JS中的根对象是*Object.prototype*对象，它是一个空对象。JS中的每个对象实际上都是从Object .prototype对象克隆来的，Object.prototype对象就是它们的原型。
> 2.    原型就是一个对象，其他对象可以通过它实现属性继承。任何一个对象都可以成为一个原型。**所有的对象在默认的情况下都有一个原型**，被对象内部的 *`[[prototype]]`* 属性所持有。而每个原型又有自己的原型，直到Object.prototype。
> 3.    JS中创建的每个函数都有一个*prototype*属性，这个属性是一个指针，指向一个对象，这个对象中包含可以由特定类型的所有实例共享的属性和方法。也就是说，prototype就是通过进行此函数的constructor call而创建的实例的原型对象。而此 **原型对象（而不是实例）** 默认情况下就会获得一个 *constructor* 属性，指向拥有prototype属性的函数。
> 4.    因为函数本身也是对象，所以也有自己的原型。**注意**：此原型和prototype属性（原型属性）没有任何关系！
> 5.    所有原生引用类型（Object/ Array/ String等等）都在其构造函数的原型上定义了方法。
> 6.    三个取得对象原型的方法：`var a = {};`
> * Object.getPrototypeOf() *Recommended*
> * a.\_\_proto\_\_ （2条下划线）
> * a.constructor.prototype（前2个方法若不支持，就要从constructor中找它的原型属性）**注意**：a 中其实并没有constructor属性，是沿着原型链到Object.prototype中找到的，指向 [Object Object]。

> 7.原型具有动态性。如果修改构造函数的prototype属性，会立马在实例中体现出来。但是

> * 修改其\_\_proto\_\_则对实例没有任何影响，因为实例跟它根本没有关系。
> * 如果替换prototype属性所指向的对象，并不会影响已经创建了的实例，其仍然引用着原来的原型对象。因为**原型链是建立在实例和原型对象之间的**，建立起来后就和构造函数没有关系了。

> 8.    `a instanceof A`: 如果a的原型在A的原型链中，为`true`
> 9.    Object.prototype.hasOwnProperty(): 判断一个对象是否包含自定义属性。但JS不会防止它被非法占用，因此如果一个对象有一个同名的自定义属性，就要使用外部的hasOwnProperty来获取正确结果。
> 10.    一个小例子：

```js
//example fails in IE
var A = function (name) {
    this.name = name;
};
var a = new A('Tara');

A.prototype === A.__proto__;//false
a.__proto__ === A.prototype;//true
A.__proto__ === A.constructor.prototype;//true
A.__proto__ === Function.prototype;//true
 ```

> *看最后一张图：[JS原型链原理图][8]*<br>
> *笔记大部分来源（函数拓展及更多例子）：[理解JS原型][9]*


## JavaScript Constructor

> 1.    JS中，并没有对constructor和一般函数做任何区分。实际上并没有构造函数，只有函数的constructor call。而任何一个在调用时前面加了关键字 `new`的函数，就都是构造函数。为了更接近面向对象语言中的构造函数，一般首字母大写。

> 2.    constructor属性存在于原型中，而不是实例中。与原型有关的构造函数知识请参考Javascript Prototypes。

> 3.    constructor的作用在于将实例和原型链接起来，它本身和实例没有直接关系。所以constructor不代表"was constructed by"。如下面的例子：

 ```js
function Foo() {...}
//replace prototype
Foo.prototype = {...};
var foo = new Foo();
foo.constructor;// Object
 ```
> * 如果没有替换原型， `foo.constructor === Foo;`因为此时实例的原型中有着constructor属性，指向Foo。而替换原型后，新原型内没有constructor属性，于是一直向上查找到了Object.prototype, 它的constructor指向Object函数。
> * `Object.constructor === Function;`因为所有的函数对象的原型\_\_proto\_\_（不是prototype属性）都是Function.prototype。

## JavaScript OOP & Inheritance

> 创建对象

> 1.    面向对象（OO）的语言有一个标志，就是它们都有类的概念。JS并不是一种真正的面向对象的语言。它的对象与基于类的语言不同。
> 2.    创建对象有不同的模式可以选用，如工厂模式、构造函数模式、原型模式、组合模式、动态原型模式、寄生构造函数模式、稳妥构造函数模式等。
> * 工厂模式：返回一个构建好的对象，问题是这些对象之间没有什么联系。
> * 构造函数模式：使用new操作符。调用后的4个步骤：1）创建一个新对象 2）将构造函数的作用域赋给新对象（this指向这个对象）3）执行构造函数的代码 4）返回新对象。可以将实例标识为特定的类型，问题是每个实例都有自己的方法，不能实现复用。
> * 原型模式：构造函数的prototype属性。我们可以把不变的属性和方法直接定义在prototype对象上。原型的本质是共享。所以如果原型中的属性包括引用类型的值（如数组、对象等），其任何一个实例对它进行操作（因为子对象继承时获得的是一个地址），都会直接改变原型的属性，进而影响到所有的实例。
> * **组合模式**：将构造函数和原型模式组合起来。构造函数模式用于定义实例属性（*私有*），原型模式定义方法的*共享*的属性。
> * **动态原型模式**：在构造函数中初始化原型（仅在必要时），把所有信息封装在构造函数中。注意不能使用字面量重写原型。
> * <s>寄生构造函数模式</s>：不推荐使用。
> * 稳妥构造函数模式：不用this, 不用new。访问不到构造函数中的原始数据成员，适合在某些安全环境下使用。

> 继承

> 1.    继承有两种方式：接口继承和实现继承。JS中没有函数签名，所以只支持实现继承，主要依靠原型链来实现。
> 2.    继承也有多种方式，如原型链继承、借用构造函数继承、组合继承、原型式继承、寄生式继承、寄生组合式继承等。
> * 原型链继承：让原型对象等于另一个对象的实例。实现的本质是重写原型对象，代之以一个新类型的实例。注意现在实例（的原型）的constructor属性已经变化了。只要是原型链中出现的原型，都是该原型链派生的实例的原型。注意：使用原型链继承后，不能再使用对象字面量创建原型方法。因为那样会重写原型，继承就没用了。原型链的问题是子类原型在继承时会成为超类原型的一个实例，即会拥有其属性，这样子类原型的实例就会共享着一个属性，就相当于直接将对象应独有的属性又写到原型中了。再次出现创建对象时原型模式的问题。
> * 借用构造函数继承（constructor stealing）：思想很简单，就是在子类构造函数内部调用类构造函数。这样，就会在子类对象上执行类构造函数中定义的初始化代码了。问题和构造函数模式相似，即方法也在构造函数中定义的话，就不能复用了。
> * **组合继承**：通过原型链实现对原型属性方法的继承（*共享*），通过借用构造函数来实现对实例属性的继承。
> * 原型式继承：要求必须有一个对象可以作为另一个对象的基础，*本质是执行了一次浅复制*。没有严格意义上的构造函数。适用于只想让一个对象与另一个对象保持相似的情况。注意引用类型的属性还是会被共享。
> * 寄生式继承：思路与寄生构造函数模式相似，创建一个仅用于封装继承过程的函数，在该函数内部以某种方式增强对象，最后再返回对象。方法不能复用造成低效率。
> * **寄生组合式继承**：组合模式有个小问题，就是会调用两次类的构造函数，这样会在子类原型上创建多余的属性（其实例上也有且会屏蔽它的属性）。而寄生组合式继承可以解决这个问题，其基本思路是创建超类型原型的一个**副本**再将此副本赋给子类型的原型。

> 代码举例

 ```js
//1.组合模式创建对象
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ['Shelby', 'Court'];
}
Person.prototype = {
    constructor: Person;
    sayName: function () {
        console.log(this.name);
    }
}

//2.动态原型模式
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ['Shelby', 'Court'];
    //方法
    if (!isFunction(this.sayName)) {
        Person.prototype.sayName = function () {
            console.log(this.name);
        }
        Person.prototype.xxx = function () {
            //do something...
        }
    }
}

//1.组合继承
function Super(name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}
Super.prototype.sayName = function () {
    console.log(this.name);
}
function Sub(name, age) {
    Supe.call(this, name);
    //Sub自己的属性
    this.age = age;
}
Sub.prototype = new Super();
Sub.prototype.constructor = Sub;
//Sub自己的方法
Sub.prototype.sayAge = function () {
    console.log(this.age);
}

//2. 原型式继承（不涉及constructor）
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}
var person = {
    name: 'Nick',
    friends: ['Aaron', 'Van']
};
var anotherPerson = object(person);
//ES5的Object.create()可用来替代object(),有可选的第二个参数——>一个对象，包含新对象的属性
var anotherPerson = Object.create(person, {
    //Property description must be an object
    name: {
        value: 'Greg'
    }
});

//3. 寄生组合式继承(最理想)
function Super(name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}
Super.prototype.sayName = function () {
    console.log(this.name);
}
function Sub(name, age) {
    Supe.call(this, name);
    //Sub自己的属性
    this.age = age;
}
Sub.prototype = Object.create(Super.prototype);
//或Sub.prototype = object(Super.prototype);
Sub.prototype.constructor = Sub;

//4.利用深度克隆（cloneObject--util.js 不涉及constructor）
var parent = {
    friends: ['Aaron', 'Van']
};
var child = cloneObject(parent);
```

>  *[JS面向对象编程（一）： 封装][10]*<br>
>  *[JS面向对象编程（二）： 构造函数的继承][11]* （利用空对象作为中介本质和object()一样）<br>
>  *[JS面向对象编程（三）： 非构造函数的继承][12]*


  [1]: http://www.cnblogs.com/lhb25/archive/2011/09/06/javascript-scope-chain.html
  [2]: http://www.laruence.com/2009/05/28/863.html
  [3]: http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html
  [4]: http://www.jb51.net/article/18303.htm
  [5]: http://www.oschina.net/question/28_41112
  [6]: https://www.ibm.com/developerworks/cn/web/1207_wangqf_jsthis/index.html
  [7]: http://www.cnblogs.com/justany/archive/2012/11/01/the_keyword_this_in_javascript.html
  [8]: http://www.jb51.net/article/30750.htm
  [9]: http://blog.jobbole.com/9648/
  [10]: http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_encapsulation.html
  [11]: http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance.html
  [12]: http://www.ruanyifeng.com/blog/2010/05/object-oriented_javascript_inheritance_continued.html

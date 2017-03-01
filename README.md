# You-Dont-Know-JS-Notebook

# 上卷
### 作用域闭包
- 闭包：当函数可以记住并访问所在的**词法作用域(不仅仅是显式的变量)**时，就产生了闭包，即使函数是在当前词法作用域之外执行的。
- let声明可以用来劫持块作用域，如:
```javascript
    for(var i=0;i<10;i++){
       let j=i;    //块作用于变量
        setTimeout(function(){
            console.log(j);
       }) 
   }
```
但若将let声明在for循环头部时，将会有一个特殊行为——每次迭代都会声明一次，并初始化为上一次迭代结束时的值。如：
```javascript
    for(let i=0;i<10;i++){      //每次循环都会声明一个新的i,并且赋值为旧的i+1。
        setTimeout(function(){
            console.log(j);
       }) 
   }
```

### 作用域
- 词法作用域：一套关于引擎如何寻找变量以及会在何处找到变量的规则
- 动态作用域：作用域在运行时被动态确定的形式
- 块作用域的两个小例子
  - with
  - catch
  - let创建块作用域变量

### this
- arguments.callee 可以指向匿名函数自身，书上说**已被废弃**，但在最新的浏览器中仍然有效。
- this在任何情况下都不指向函数的词法作用域。作用域“对象”无法通过代码访问
- 当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、调用方法、传入的参数等信息。**this是记录的其中一个属性，会在函数执行过程中得到。**
- 如何寻找调用位置
  - 调用位置在当前正在执行函数的前一个调用中
  ```javascript
  function baz(){
      // 当前调用栈是:baz
      // 因此当前调用位置是全局作用域
      
      console.log("baz");
      bar();// <-- bar的调用位置
  }
  function bar(){
      // 当前调用栈是baz->bar
      // 因此，当前调用位置在baz中
      console.log("bar");
      foo();  // <-- foo的调用位置
  }
  function foo(){
      // 当前调用栈是baz -> bar -> foo
      // 因此当前调用位置在bar中
      console.log("foo");
  }
  ```
- 严格模式下，全局对象将无法使用默认绑定。
  ```javascript
  function foo() {
      "use strict";
      console.log( this.a );
  }
  var a = 2;
  foo(); // TypeError: this is undefined
  ```
  仅当foo()定义中使用了严格模式，才会使得this无法绑定至全局作用域。
- 作用域绑定
  - 默认绑定
  - 隐式绑定：当函数引用有上下文对象时，**隐式绑定**规则会把函数调用中的this绑定到这个上下文对象。
  - 显示绑定：
      - 硬绑定：使用bind()，call()和apply()显示地强制绑定作用域
  - new绑定
  - 判断this（优先级从上到下）
     1. 函数是否在new 中调用（new 绑定）？如果是的话this 绑定的是新创建的对象。
var bar = new foo()
     2. 函数是否通过call、apply（显式绑定）或者硬绑定调用？如果是的话，this 绑定的是
指定的对象。
var bar = foo.call(obj2)
     3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this 绑定的是那个上
下文对象。
var bar = obj1.foo()
     4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到
全局对象。
var bar = foo()
  - 绑定的例外：
    - call(null),apply(null) 实际应用默认绑定规则
    - 
  ```javascript
  function foo() {
      console.log( this.a );
  }
  var a = 2;
  var o = { a: 3, foo: foo };
  var p = { a: 4 };
  o.foo(); // 3
  (p.foo = o.foo)(); // 2
  ```
  赋值语句p.foo=o.foo返回值是目标函数的引用，因此调用位置是foo()而不是p.foo()或o.foo()
  - 软绑定：给默认绑定指定一个全局对象和undefined以外的值，实现和硬绑定相同的效果，同时保留隐式绑定或显式绑定修改this的能力。
  [关于书上代码的解释](https://segmentfault.com/q/1010000006223479)
  - 箭头函数使用词法作用域，根据外层作用域来决定this，不使用上述四种标准规则。
  
### 对象
- 六个主要类型
  - 简单基本类型：string、boolean、number、null、undefined本身并不是对象
      - 语言本身的一个Bug: typeof null=='object'，但其实null是基本类型
  - 复杂基本类型：object
- 内置对象
  - String,Number,Boolean,Object,Function,Array,Date,RegExp,Error
- ES5 Object相关方法
  - Object.assign方法中，成员属性采用=操作符赋值，并不是深拷贝。
  - Object.preventExtensions(obj) 禁止扩展
  - Object.seal(..) 创建一个"密封"的对象，实际上会在现有对象上调用Object.preventExtensions(..)，并把所有现有属性标记为configurable:false
      - 禁止扩展
      - 禁止配置
  - Object.freeze(..) 创建一个冻结对象，实际上会调用Object.seal(..)并把所有"数据访问"属性标记为writable:false
      - 禁止扩展
      - 禁止配置
      - 禁止修改属性值
- [[Put]]大致会检查下面内容
  1. 属性是否是访问描述符？如果是并且存在setter就调用setter
  2. 属性的数据描述符中writable是否为false?如果是，在非严格模式下静默失败，在严格模式下抛出TypeError异常
  3. 如果都不是，将该值设置为属性的值
- getter和setter的另一种写法
```javascript
var myObj={
    get a(){
      return this.val;
    },
    set a(newVal){
      this.val=newVal;
    }
}
```
- 检查属性是否存在
  - "xx" in Obj:会检查属性是否在对象及其原型链中
  - hasOwnProperty(..)只会检查属性是否在Obj对象中，不会检查原型链
- 枚举
  - “可枚举”相当于“可以出现在对象属性的遍历中”
  - Object.propertyIsEnumerable(..)会检查给定的属性名是否直接存在于对象中(而不是在原型链上)并且满足enumerable:true
  - Object.keys(..)返回一个数组，包含所有可枚举属性。
  - Object.getOwnPropertyNames(..)返回一个数组，包含所有属性，无论它们是否可枚举。
- 遍历
  - ES6新增 for..of循环语法来遍历value而非key
  - 数组对象有迭代器Symbol.iterator，而普通对象没有。可以自行添加一个：
  ```javascript
  var obj={
      a:2,
      b:3
  }
  Object.defineProperty(obj,Symbol.iterator,{
      enumerable:false,
      writable:false,
      configurable:false,
      value:function(){
          var o=this;
          var idx=0;
          var ks=Object.keys(o);
          return{
              next:function(){
                  return {
                      value:o[ks[idx++]],
                      done:idx>ks.length
                  }
              }
          }
      }
  })
  ```
  
### 类 继承
- 类实际上是一种设计模式。
- 寄生继承
```javascript
// “寄生类” Car
function Car() {
     // 首先，car 是一个Vehicle
     var car = new Vehicle();
     // 接着我们对car 进行定制
     car.wheels = 4;
     // 保存到Vehicle::drive() 的特殊引用
     var vehDrive = car.drive;
     // 重写Vehicle::drive()
     car.drive = function() {
     vehDrive.call( this );
     console.log(
          "Rolling on all " + this.wheels + " wheels!"
     );
     return car;
}
```
- 隐式混入
```javascript
var Something = {
    cool: function() {
        this.greeting = "Hello World";
        this.count = this.count ? this.count + 1 : 1;
    }
};
Something.cool();
Something.greeting; // "Hello World"
Something.count; // 1
var Another = {
    cool: function() {
    // 隐式把Something 混入Another
        Something.cool.call( this );
    }
};
Another.cool();
Another.greeting; // "Hello World"
Another.count; // 1 （count 不是共享状态）
```

### 原型
- [[Prototype]]原型链
  - 假设有```obj.foo="foo"```的语句。如果foo属性不直接存在于obj中而是存在于原型链上层时，上述语句会出现三种情况：
    1. 如果在[[Prototype]] 链上层存在名为foo 的普通数据访问属性（参见第3 章）并且没有被标记为只（writable:false），那就会直接在myObject 中添加一个名为foo 的新属性，它是屏蔽属性。
    2. 如果在[[Prototype]] 链上层存在foo，但是它被标记为只读（writable:false），那么无法修改已有属性或在obj上创建屏蔽属性。如果运行在严格模式下，代码会抛出一个错误。否则，这条赋值语句会被忽略。总之，不会发生屏蔽。
    3. 如果在[[Prototype]] 链上层存在foo 并且它是一个setter（参见第3 章），**那就一定会调用这个setter**。foo 不会被添加到（或者说屏蔽于）myObject，也不会重新定义foo 这个setter。
    - 如果希望在上述第二、三种情况下也屏蔽foo，那就需要使用Object.defineProperty来向obj添加foo
- 原型继承
  - 方法一：```newObj.prototype=Object.create(oldObj.prototype)```
  - 方法二：```newObj.prototype=new oldObj()```
  - ES6新增的修改对象原型的API:```Object.setPrototypeOf(obj,other.prototype)```
      - polyfill:
      ```javascript
      Object.setPrototypeOf=Object.setPrototypeOf||function(obj,proto){
          obj.__proto__=proto;
          return obj;
      }
      ```
- 检查"类"关系
  - ```a instanceof Foo``` 返回a的整条原型链中是否有指向Foo.prototype的对象。但仅限对象与函数之间的关系检查。
  - ```obj1.isPrototypeOf(obj2)``` 返回obj1是否出现在obj2的原型链中。
  - ```Object.getPrototypeOf(..)``` 获取原型链(\__proto__)
- class语法
  - ES6中，class无法定义类成员属性，只能定义类成员方法
  - super只能用于constructor()函数中。在子类中，只有在调用super()之后，才能使用this指针。
  
### 行为委托
- 委托行为：在找不到属性或者方法引用时，会把这个请求委托给另一个对象。js中的表现是委托原型链上的对象。
- 使用函数对象的name标识符：
```javascript
var Foo={
    bar:function bar(){}
}
```
好处是方便自我引用和追踪调用栈。
- 自省：检查实例的类型
  - 主要目的：通过创建方式来判断对象的结构和功能
- JS的原型链机制本质上就是行为委托机制。


# 中卷
### 类型
- symbol是ES6新增的第七种类型
- ```typeof null === "object" // true```

### 值
- 数组
  - length 由数组最大的下标决定，之前的所有元素若未赋值，则为undefined。例如:
  ```javascript
  var a=[]
  a[2]='1';
  a.length===3 //true
  ```
- 数字
  - js中没有真正意义上的整数。“整数”即是没有小数的十进制数。
  - .tofixed(..)用来指定小数部分的显示位数
  - .topRecision(..)用来指定有效数位的显示位数
  - 浮点数相加通常不精确。解决方案是设置一个误差范围值，称为"机器精度"。ES6中，该值定义为Number.EPSILON中
  - 能够呈现的最大浮点数：Number.MAX_VALUE;最小浮点数:Number.MIN_VALUE
  - 非自反值：NaN

### 原生函数
- js会自动为基本类型值包装一个封装对象
  - 不需要对诸如.length这样的场景做提前的性能优化，原因是浏览器已经对此做了优化，直接使用封装对象来"提前优化"反而会降低执行效率。
- 优先使用基本类型而非封装对象
- Function.prototype是一个空函数;RegExp.prototype是一个"空"的正则表达式（非空对象）;Array.prototype是一个空数组
- 函数调用的详细过程：
  - 当调用一个函数时，一个新的执行上下文就会被创建。
  1. 创建阶段：执行上下文会分别创建变量对象，建立作用域链，以及确定this的指向。
      - 创建变量对象的过程：
      1. 建立arguments对象。
      2. 检查当前上下文的函数声明，也就是使用function关键字声明的函数。在变量对象中以函数名建立一个属性，属性值为指向该函数所在内存地址的引用。如果函数名的属性已经存在，那么该属性将会被新的引用所覆盖。
      3. 检查当前上下文中的变量声明，每找到一个变量声明，就在变量对象中以变量名建立一个属性，属性值为undefined。如果该变量名的属性已经存在，为了防止同名的函数被修改为undefined，则会直接跳过，原属性值不会被修改。
  2. 代码执行阶段：创建完成之后，开始执行代码。这个时候，会完成变量赋值，函数引用，以及执行其他代码。
  ![](http://upload-images.jianshu.io/upload_images/599584-391af3aad043c028.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  3.执行完毕后出栈，等待被回收

### 强制类型转换
- 值类型转换
  - 发生在运行时(runtime)
  - 隐式强制类型转换 ```var a=42; var b=a+"" ```
  - 显式强制类型转换 ```var c=String(a);```
- 抽象值操作
  - toString
      - 对于普通对象，除非自行定义，否则toString()返回内部属性[[Class]]的值
  - JSON
      - JSON.stringify()可以字符串化所有安全的JSON值
      - 不安全的JSON值：undefined,function,symbol(ES6+)和包含循环引用(对象之间互相引用，形成一个无线循环)的对象。
      - toJSON() 如果对象中定义了toJSON()方法，JSON字符串化时会首先调用该方法，然后用它的值来进行序列化。
  - ToNumber
      - Number()
      - undefined->NaN,null->0
      - 处理失败时返回NaN
      - "","\n"," "等空字符串被ToNumber强制类型转换为0
  - ToBoolean
      - 假值(falsy value):undefined,null,false,+0,-0,NaN,""。转换为false
      - 假值对象(falsy object)
          - 并非封装了假值的对象
          - document.all是一个假值对象。经常通过将其转换为Boolean来判断浏览器是否是老版本IE。
      ```javascript
      if(document.all){ /* it's IE */}
      ```
      - 真值:除""外的其他字符串,[],{},function(){}等
  - 对象的toPrimitive操作:调用valueOf()，若有并返回基本类型值，则使用该值进行强制类型转；若没有则调用toString()。如果两者都没有则产生TypeError错误。
  - 显示强制转换
      - 例:``` var c="3.14"; var d=+c; //3.14```
      - 日期显示转换为数字
          - 获取时间戳的快速方法：```var timestamp=+new Date();```
      - ~运算符
          - 只适用于32位整数。强制操作数使用32位格式
          - 首先将值强制类型转换为32位数字，然后执行字位操作"非"
          - 对负数的处理与Math.floor(..)不同
      ```javascript
      Math.floor(-49.6);  // -50
      ~~-49.6;            // -49
      ```
          - 用途：将字符串转为整数，取整，将值截为32位整数等
  - 显式解析数字字符串
      - parseInt(..)
          - 先将参数强制类型转换为字符串再进行解析。
          - 一些奇怪的行为：
    ```javascript
    parseInt( 0.000008 ); // 0 ("0" 来自于 "0.000008")
parseInt( 0.0000008 ); // 8 ("8" 来自于 "8e-7")
parseInt( false, 16 ); // 250 ("fa" 来自于 "false")
parseInt( parseInt, 16 ); // 15 ("f" 来自于 "function..")
parseInt( "0x10" ); // 16
parseInt( "103", 2 ); // 2
    ```
- 相等比较
  - 对于x==
      - 若type(x)为数字,type(y)为字符串，则返回x==ToNumber(y)的结果
      - 若type(x)为bool,type(y)为数字，则返回toNumber(x)==y的结果
      - 若type(x)为字符串或数字,type(y)为对象，则返回x==ToPrimitive(y)的结果
  - 在==中,null和undefined是一回事。除此之外其他值都不存在这种情况。
  ```javascript
  var a = null;
var b;
a == b; // true
a == null; // true
b == null; // true
a == false; // false
b == false; // false
a == ""; // false
b == ""; // false
a == 0; // false
b == 0; // false
  ```
  - 根据规范,<= 被处理为 >，然后将结果翻转。例如a<=b被处理为a>b，然后将该结反转得到原结果值。
  
### 混合环境javascript
- 全局DOM变量
  - 声明一个全局变量不仅仅是创建了一个全局变量，还会在global对象（浏览器中为window）中创建一个同名属性
  - 由于浏览器演进的历史遗留问题，在创建带有id属性的DOM元素时也会创建同名的全局变量:
  ```javascript
  <div id="foo"></div>
  console.log(foo)  //HTML元素
  ```
- 部分引擎的一些限制：
  - 字符串常量中允许的最大字符数（并非只是针对字符串值）；
  - 可以作为参数传递到函数中的数据大小（也称为栈大小，以字节为单位）；
  - 函数声明中的参数个数；
  - 未经优化的调用栈（例如递归）的最大层数，即函数调用链的最大长度；
  - JavaScript 程序以阻塞方式在浏览器中运行的最长时间（秒）；
  - 变量名的最大长度。

### 异步
- 并发
  - 严格来说，setTimeout(..,0)并不直接把项目插入到事件循环队列。定时器会在“有机会”的时候插入事件。因此两个连续的setTimeout(..,0)调用并不能严格保证其顺序处理。
- 任务
  - ES6引入了任务对列，建立在事件循环队列之上。
      - 挂在事件循环队列的每个tick之后的一个队列
      - 在每次事件循环中要确保任务对列中的事件都做完（实际上用了嵌套循环）

### Promise
- Promise调用then(..)的时候，即使这个Promise已经决议，一同给then(..)的回调也总会被异步调用。
- 一个Promise决议后，这个Promise上所有的通过then(..)注册的回调都会在下一个异步时机点依次被立即调用。
- 不要依赖于Promise间回调的顺序和调度。
- 若把同一个回调注册了不止一次(p.then(f);p.then(f))，那它被调用的次数就会和注册次数相同。
- Promise回调中发生的异常错误，都会导致Promise拒绝(reject)
- 关于Promise从创建到状态改变的执行顺序:
```javascript
let promise = new Promise(function(resolve, reject) {
    console.log('Promise');
    resolve();
});
promise.then(function() {
    console.log('Resolved.');
});
console.log('Hi!');
// Promise
// Hi!
// Resolved
```
- Promise.catch():最好使用catch(..)来捕获error。原因：catch既可以捕捉Promise回调中的错误，也可以捕捉在它之前的then(..)回调中的错误。
- 将多个Promise实例包装成一个新的Promise实例
  - Promise.all([]):仅当所有Promise为resolve，状态才变为resolve；如果有其中一个reject，状态就变为reject
  - Promise.race([]):当其中有一个实例改变了状态，则最终状态发生改变。那个率先改变状态的Promise实例返回的值便传递给then()（或catch()）回调函数。
- Promise.resolve()（Promise.reject()）将现有对象转换成Promise对象。具体请看《ECMAScript6入门》

### 生成器
- 任意一个数组对象的Symbol.iterator方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。
```javascript
let myIterable={};
let it=myIterable[Symbol.iterator]()
it.next()
```
for..of 应用的对象必须有Symbol.iterator方法
- Generator函数就是遍历器生产函数，因此可以把Generator赋值给对象的Symbol.iterator属性
- 若Generator函数中有赋值yield的语句，则next()传入的参数影响到函数内变量的值。例如:
```javascript
function* foo(x) {
    var y = 2 * (yield (x + 1));
    var z = yield (y / 3);
    return (x + y + z);
}
var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}
var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```

### 程序性能
- web worker
  - worker之间以及它们与主程序之间不会共享任何作用域或资源
  - 实例化worker:
  ```javascript
  var w1=new Worker("http://xxx/index.js")  //应该指向一个javascript位置
  ```
  - 共享worker```new SharedWorker(url)```
      - 用于创建一个可以共享的中心Worker，节省资源
      - 调用程序必须使用worker的port对象用于通信：
    ```javascript
    w1.port.addEventListener(...)
    w1.port.postMessage(..)
    ```
- SIMD(单指令多数据)是一种数据并行方式，与Web Worker的任务并行相对。
  - 重点：并行处理数据的多个位
- asm.js 指js中可以高度优化的一个子集

### 性能测试与调优
- **Benchmark.js** 已经实现了许多有效的性能测试逻辑
- 写好测试
  - 不要试图窄化到真实代码的微小片段，以及脱离上下文而只测量这一小部分的性能，因为
包含更大（仍然有意义的）上下文时功能测试和性能测试才会更好。这些测试可能也会运
行得慢一点，这意味着环境中发现的任何差异都更有意义。
- 尾调用优化(TCO)
  - 尾调用：出现在另一个函数“结尾”处的函数调用，并且调用结束后没有其余事情要做了
  ```javascript
  function foo(x){
      return x;
  }
  function bar(y){
      return foo(y+1);  // 尾调用
  }
  function baz(){
      return 1+bar(40); // 非尾调用
  }
  ```
  - 优化：支持TCO的引擎能够意识到foo(y+1)位于尾部，意味着bar(..)基本上已经完成了，那么在调用foo(..)时，就不需要创建一个新的栈帧，而是可以重用已有的bar(..)的栈帧。这样不仅速度快，也更节省内存。

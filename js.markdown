# 线程与进程
JavaScript 单线程指的是浏览器中负责解释和执行 JavaScript 代码的只有一个线程，即为JS引擎线程，但是浏览器的渲染进程是提供多个线程的，如下：

* JS引擎线程
* 事件触发线程
* 定时触发器线程
* 异步http请求线程
* GUI渲染线程  

当遇到计时器、DOM事件监听或者是网络请求的任务时，JS引擎会将它们直接交给 webapi，也就是浏览器提供的相应线程（如定时器线程为setTimeout计时、异步http请求线程处理网络请求）去处理，而JS引擎线程继续后面的其他任务，这样便实现了 异步非阻塞。

定时器触发线程也只是为 setTimeout(..., 1000) 定时而已，时间一到，还会把它对应的回调函数(callback)交给 任务队列 去维护，JS引擎线程会在适当的时候去任务队列取出任务并执行。
## 事件循环与消息队列
JavaScript 通过 事件循环 event loop 的机制来解决什么时候处理队列任务的问题

其实 事件循环 机制和 任务队列 的维护是由事件触发线程控制的。

事件触发线程 同样是浏览器渲染引擎提供的，它会维护一个 任务队列。

JS引擎线程遇到异步（DOM事件监听、网络请求、setTimeout计时器等...），会交给相应的线程单独去维护异步任务，等待某个时机（计时器结束、网络请求成功、用户点击DOM），然后由 事件触发线程 将异步对应的 回调函数 加入到消息队列中，消息队列中的回调函数等待被执行。

同时，JS引擎线程会维护一个 执行栈，同步代码会依次加入执行栈然后执行，结束会退出执行栈。

如果执行栈里的任务执行完成，即执行栈为空的时候（即JS引擎线程空闲），事件触发线程才会从消息队列取出一个任务（即异步的回调函数）放入执行栈中执行。

消息队列是类似队列的数据结构，遵循先入先出(FIFO)的规则

*  1. 所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。
*  2. 主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。
*  3. 一但"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
*  4. 主线程不断重复上面的第三步。

只要主线程空了，就会去读取"任务队列"，这就是JavaScript的运行机制。这个过程会不断重复，这种机制就被称为事件循环（event loop）机制。

[详细参考看引用](https://zhuanlan.zhihu.com/p/139967525)


所有任务分为 macro-task 和 micro-task:

macro-task：主代码块、setTimeout、setInterval等（可以看到，事件队列中的每一个事件都是一个 macro-task，现在称之为宏任务队列）
micro-task：Promise、process.nextTick等
JS引擎线程首先执行主代码块。

每次执行栈执行的代码就是一个宏任务，包括任务队列(宏任务队列)中的，因为执行栈中的宏任务执行完会去取任务队列（宏任务队列）中的任务加入执行栈中，即同样是事件循环的机制。

在执行宏任务时遇到Promise等，会创建微任务（.then()里面的回调），并加入到微任务队列队尾。

micro-task必然是在某个宏任务执行的时候创建的，而在下一个宏任务开始之前，浏览器会对页面重新渲染(task >> 渲染 >> 下一个task(从任务队列中取一个))。同时，在上一个宏任务执行完成后，渲染页面之前，会执行当前微任务队列中的所有微任务。

也就是说，在某一个macro-task执行完后，在重新渲染与开始下一个宏任务之前，就会将在它执行期间产生的所有micro-task都执行完毕（在渲染前）。

这样就可以解释 "promise 1" "promise 2" 在 "timer over" 之前打印了。"promise 1" "promise 2" 做为微任务加入到微任务队列中，而 "timer over" 做为宏任务加入到宏任务队列中，它们同时在等待被执行，但是微任务队列中的所有微任务都会在开始下一个宏任务之前都被执行完。

在node环境下，process.nextTick的优先级高于Promise，也就是说：在宏任务结束后会先执行微任务队列中的nextTickQueue，然后才会执行微任务中的Promise。
执行机制：

* 执行一个宏任务（栈中没有就从事件队列中获取）  
* 执行过程中如果遇到微任务，就将它添加到微任务的任务队列中  
* 宏任务执行完毕后，立即执行当前微任务队列中的所有微任务（依次执行）  
* 当前宏任务执行完毕，开始检查渲染，然后GUI线程接管渲染  
* 渲染完毕后，JS引擎线程继续，开始下一个宏任务（从宏任务队列中获取）  
![alt text](/img/task.png)

宏任务 macro-task(Task)  
一个event loop有一个或者多个task队列。task任务源非常宽泛，比如ajax的onload，click事件，基本上我们经常绑定的各种事件都是task任务源，还有数据库操作（IndexedDB ），需要注意的是setTimeout、setInterval、setImmediate也是task任务源。总结来说task任务源：

script  
setTimeout  
setInterval  
setImmediate  
I/O  
requestAnimationFrame  
UI rendering  

微任务 micro-task(Job)  
microtask 队列和task 队列有些相似，都是先进先出的队列，由指定的任务源去提供任务，不同的是一个 event loop里只有一个microtask 队列。另外microtask执行时机和Macrotasks也有所差异

process.nextTick  
promises  
Object.observe  
MutationObserver  

宏任务和微任务的区别

宏队列可以有多个，微任务队列只有一个,所以每创建一个新的settimeout都是一个新的宏任务队列，执行完一个宏任务队列后，都会去checkpoint 微任务。  
一个事件循环后，微任务队列执行完了，再执行宏任务队列  
一个事件循环中，在执行完一个宏队列之后，就会去check 微任务队列  


# 起步
* js 全称 javascript, 是一门解释型弱类型编程语言，js包含  
    * ECMAScript 
        * ecma-262定义的语言标准, 不同的宿主环境都会基于此标准具体实现
        * 例如 Node环境 web环境 flash环境
    * BOM 
        * 浏览器对象模型, 与浏览器交互的方法和接口 （h5是BOM实现的规范标准）
        * BOM 主要针对浏览器窗口和子窗口（frame）
        * 浏览器相关操作(移动,缩放,弹出,关闭)
        * navigator 提供浏览器相关信息
        * location 页面相关信息
        * screen 屏幕相关信息
        * performance 内存/时间统计/导航行为相关信息
        * 本地存储 cookie/localstorage/sessionstoreage
        * 其他
    * DOM 
        * 文档对象模型,给予可以与文档交互的方法和接口
        * 映射文档结构,视图,事件,CSS,DOM的遍历和范围，SVG(可伸缩矢量图)等

* "use strict"  严格模式, 可以写在全局 也可以写在函数内,提供了更严格的规范
### 将js引入html
* 使用```<script> js代码 </script>```
* 使用```<script src="codeUrl" > js代码 </script>```
    * *如果同时引入脚本和在标签内书写代码 则只会下载并执行脚本文件，从而忽略行内代码。*
    * js代码会被从上到下解释, 解释和下载js会阻塞页面, 通常放在页面末尾
        * defer 和 async 都会将变为异步 不会阻塞页面
        * defer 立即下载，脚本在页面完全加载后按照顺序执行,适用于依赖页面 DOM 结构且需要按照特定顺序执行的脚本
        * async 脚本在加载完成后立即执行，不保证顺序, 不考虑页面的其他部分是否加载完毕. 适用于独立的、不依赖于页面其他部分的脚本，执行时间不确定且不影响页面初始渲染
* 使用```<noscript> 此页面未支持javascript</noscript>```
## js基础
### 变量
* var let const  使用频率推荐 const > let >var
    * var
        * 不初始化的情况下，变量会保存一个特殊值 undefined
        * 如果声明变量省略了操作符 那么会创建一个全局变量  `不推荐这么做`
        * var声明的变量会自动提升到函数作用域的顶部, 全局作用域内声明的会成为window对象的属性
    * let 
        * let声明是块级作用域
        * let声明的变量不会被提升, 声明之前变量被执行的瞬间被称为'暂时性死区'
        * 不会成为window对象的属性
    * const
        * 与let条件相同
        * 声明变量时必须初始化, 尝试修改会报错
        * 如果声明的变量是一个引用类型 修改引用类型的内容不受影响
### 数据类型
* 6种简单数据类型(原始类型): undefined,null,number,string,boolean,symbol
* 1种复杂数据类型 object
* typeof 操作符: 确定数据类型
    * 'undefined' 未定义
        * 未定义的都会被赋值为 undefined
    * 'boolean' 布尔值 包含 true和false, 区分大小写的
        * 在条件语句中(if等)会自动进行布尔转换,布尔值的转换 Boolean(变量)
        *
            |原变量|转为true|转为false|
            |:----:|:----:|:----:|
            |boolean|true|false|
            |string|非空字符串|\"\" (空字符串)|
            |number|非0字符串|0 NaN|
            |object|任意对象|null|
            |undefined|无|undefined|

    
    * 'string' 字符串
        *  除了null和undefined其他几乎所有值都有 toString()方法
        * Number(10)toString(16) 结果为 a  , 参数为数值类型时的进制
        * String(null)
        * 模板字符串 ``` `${a}123132${b}123456` ```
        * 标签函数 
        ``` 
        let a = 6; 
        let b = 9;
        function simpleTag(strings, ...expressions) { 
            console.log(strings);  ["", " + ", " = ", ""] 
            console.log(expressions);  [6,9,15]
            return 'foobar'; 
        }
        let taggedResult = simpleTag`${ a } + ${ b } = ${ a + b }`; 
        console.log(taggedResult);  "foobar"
         ```
    * 'number' 数字
        * 使用 IEEE 754 (双精度) 表示整数和浮点数, 会导致 0.1+0.2 != 0.3的问题: 例如，0.1 的二进制表示是一个无限循环小数0.00011001100110011...，在计算机中只能用有限的位数进行近似表示
        * 07 八进制 0x1 16进制
        * 浮点数的存储空间是整数的两倍,如果是1.0 会被自动调整为 1
        * NaN表示不是一个数, 它不等于任何 甚至不等于自身(NaN == NaN为false), isNaN方法可以传递一个变量, 把不可以变为数字的(字符串的数字,布尔值,数字均可以变为数字)都返回true
        * 非数值转为数值 Number() parseInt() parseFloat()
        * Number()包装后: number返回数值; boolean: true返回1,false返回0;null返回0;undefined 返回NaN;对象，调用 valueOf()方法，并按照上述规则转换返回的值。如果转换结果是 NaN，则调用toString()方法，再按照转换字符串的规则转换;string: '01'会转为1, "0xf" 会转为对于的十进制数, 空串转为0
        * parseInt("10", 2); // 2，按二进制解析
        * parseFloat("22.34.5"); // 22.34 解析到字符串末尾或者解析到一个无效的浮点数值字符为止
    * 'symbol' 符号类型
        * 符号实例是唯一、不可变的。符号的用途是确保对象属性使用唯一标识符，不会发生属性冲突的危险
        * 声明需要 Symbol('此符号的描述')初始化, 可以传递一个参数作为符号的描述
        * ECMAScript 6 也引入了一批常用内置符号（well-known symbol），用于暴露语言内部行为，开发者可以直接访问、重写或模拟这些行为,在提到 ECMAScript 规范时，经常会引用符号在规范中的名称，前缀为@@。比如，@@iterator 指的就是 Symbol.iterator。
        * Symbol.iterator 这个符号表示实现迭代器 API 的函数,可以通过在自定义对象上重新定义Symbol.iterator 的值，来改变 for-of 在迭代该对象时的行为
            ```
            class Emitter { 
                constructor(max) { 
                this.max = max; 
                this.idx = 0; 
                } 
                *[Symbol.iterator]() { 
                    while(this.idx < this.max) { 
                        yield this.idx++; 
                    } 
                } 
            } 
            function count() { 
                let emitter = new Emitter(5); 
                for (const x of emitter) { 
                    console.log(x); 
                } 
            } 
            count(); 
            // 0
            ```
        * instanceof 操作符会使用 Symbol.hasInstance 函数来确定关系。以 Symbol. 
hasInstance 为键的函数会执行同样的操作，只是操作数对调了一下：
            ```
            function Foo() {} 
            let f = new Foo(); 
            console.log(Foo[Symbol.hasInstance](f)); // true

            Foo[Symbol.hasInstance](f) 等同于 f instanceof Foo


            // 可以直接修改调整继承
            class Bar {} 
            class Baz extends Bar { 
            static [Symbol.hasInstance]() { 
            return false; 
            } 
            }

            ```
        * 其他内置symbol:  Symbol.match Symbol.replace Symbol.search Symbol.species  Symbol.split Symbol.toPrimitive Symbol.toStringTag Symbol.unscopables
    * 'function' 函数 (严格说函数也是一个对象, 但是由于有特殊的属性 必须进行区分)
    * 'object' 对象或者null
        * null被认为是一个空对象指针,未来如果要保存对象的话 建议使用null来初始化
        * 对象其实就是一组数据和功能的集合
        * 每个object都有的属性和方法 new Object()
            ```
            let obj = new Object()
            function Person(name, age) {
                this.name = name;
                this.age = age;
                this.constructor = function(newName, newAge) {
                    this.name = newName;
                    this.age = newAge;
                };
            }
            let p1 = new Person(1,2)
            p1.constructor == Person  返回true 创建对象的函数,值就是Object()函数 
            obj.hasOwnProperty(name)  判断当前对象实例（不是原型）上是否有name属性
            isPrototypeOf(obj)  用于判断当前对象是否为另一个对象的原型
            toString()
            valueOf()
            ```
* 位操作符
    * ecma中所有数值都有ieee 754 64位格式存储,但是操作会先转为32位再转为64位
    * 前31位表示数值,32位为符号位表示正负(0正1负)
    * ~ 按位非  按位非的最终效果是对数值取反并减 1
    * & 按位与  都为1才为1
    * | 按位或  一个为1即为1
    * ^ 按位异或  一个为0一个为1时  为 1  ,其他为0
    * << 左移    等于*2的n次方  尽头会循环
    * \>\> 右移    等于/2的n次方  尽头是 0
* 其他
    * +如果和string  1+ "1" = "11" 
    * -如果和string  1-"1" = 0
     * / % **  ! || && 
* 关系操作符 > < >= <=
    * 如果操作数都是数值，则执行数值比较。
    * 如果操作数都是字符串，则逐个比较字符串中对应字符的编码。
    * 如果有任一操作数是数值，则将另一个操作数转换为数值，执行数值比较。
    * 如果有任一操作数是对象，则调用其 valueOf()方法，取得结果后再根据前面的规则执行比较。如果没有 valueOf()操作符，则调用 toString()方法，取得结果后再根据前面的规则执行比较。
    * 如果有任一操作数是布尔值，则将其转换为数值再执行比较。
* 强制类型转换 ==
    * 如果任一操作数是布尔值，则将其转换为数值再比较是否相等。false 转换为 0，true 转换为 1
    * 如果一个操作数是字符串，另一个操作数是数值，则尝试将字符串转换为数值，再比较是否相等。
    * 如果一个操作数是对象，另一个操作数不是，则调用对象的 valueOf()方法取得其原始值，再根据前面的规则进行比较。在进行比较时，这两个操作符会遵循如下规则。
    * null 和 undefined 相等。
    * null 和 undefined 不能转换为其他类型的值再进行比较。
    * 如果有任一操作数是 NaN，则相等操作符返回 false，不相等操作符返回 true。记住：即使两个操作数都是 NaN，相等操作符也返回 false，因为按照规则，NaN 不等于 NaN。
    * 如果两个操作数都是对象，则比较它们是不是同一个对象。如果两个操作数都指向同一个对象，则相等操作符返回 true。否则，两者不相等。
* 语句
    * for-in  :  用于枚举对象中的非符号键属性 for (property in expression) statement
    * for-of  : 严格的迭代语句，用于遍历可迭代对象的元素  for (property of expression) statement
## 变量与作用域与内存
### 引用值
* 引用值是保存在内存中的对象，在操作对象时，实际上操作的是对该对象的引用（reference）而非实际的对象本身
* 只有引用值可以动态添加后面可以使用的属性
* 所有引用值都是 Object 的实例
### 传递参数
* ECMAScript 中所有函数的参数都是按值传递的
* ECMAScript 中函数的参数就是局部变量
### 执行上下文与作用域
* 变量或函数的上下文决定了它们可以访问哪些数据，以及它们的行为。每个上下文都有一个关联的变量对象（variable object），而这个上下文中定义的所有变量和函数都存在于这个对象上。虽然无法通过代码访问变量对象，但后台处理数据会用到它。不同的宿主环境的最外层上下文不一样，在浏览器中，全局上下文就是我们常说的 window 对象，
* 每个函数调用都有自己的上下文。当代码执行流进入函数时，函数的上下文被推到一个上下文栈上。在函数执行完之后，上下文栈会弹出该函数上下文，将控制权返还给之前的执行上下文。ECMAScript程序的执行流就是通过这个上下文栈进行控制的
* 上下文中的代码在执行的时候，会创建变量对象的一个作用域链（scope chain）。这个作用域链决定了各级上下文中的代码在访问变量和函数时的顺序。代码正在执行的上下文的变量对象始终位于作用域链的最前端。如果上下文是函数，则其活动对象（activation object）用作变量对象。活动对象最初只有一个定义变量：arguments。（全局上下文中没有这个变量。）作用域链中的下一个变量对象来自包含上下文，再下一个对象来自再下一个包含上下文。以此类推直至全局上下文；全局上下文的变量对象始终是作用域链的最后一个变量对象
* 函数参数被认为是当前上下文中的变量，因此也跟上下文中的其他变量遵循相同的访问规则
* 作用域增强
    * with语句 对 with 语句来说，会向作用域链前端添加指定的对象 with(xxx){}
    * try catch 
### 垃圾回收
* js使用了自动的垃圾回收：确定哪个变量不会再使用，然后释放它占用的内存。这个过程是周期性的
    * 标记清理
    * 引用计数
### 内存管理
* js内存无需开发者操心
* 如果数据不再必要，那么把它设置为 null，从而释放其引用。这也可以叫作解除引用
* 策略
### 小结
JavaScript 变量可以保存两种类型的值：原始值和引用值。原始值可能是以下 6 种原始数据类型之
一：Undefined、Null、Boolean、Number、String 和 Symbol。原始值和引用值有以下特点。
* 原始值大小固定，因此保存在栈内存上。
* 从一个变量到另一个变量复制原始值会创建该值的第二个副本。
* 引用值是对象，存储在堆内存上。
* 包含引用值的变量实际上只包含指向相应对象的一个指针，而不是对象本身。
* 从一个变量到另一个变量复制引用值只会复制指针，因此结果是两个变量都指向同一个对象。
* typeof 操作符可以确定值的原始类型，而 instanceof 操作符用于确保值的引用类型。
---
任何变量（不管包含的是原始值还是引用值）都存在于某个执行上下文中（也称为作用域）。这个上下文（作用域）决定了变量的生命周期，以及它们可以访问代码的哪些部分。
* 执行上下文分全局上下文、函数上下文和块级上下文。
* 代码执行流每进入一个新上下文，都会创建一个作用域链，用于搜索变量和函数。
* 函数或块的局部上下文不仅可以访问自己作用域内的变量，而且也可以访问任何包含上下文乃
至全局上下文中的变量。
* 全局上下文只能访问全局上下文中的变量和函数，不能直接访问局部上下文中的任何数据。
* 变量的执行上下文用于确定什么时候释放内存。
JavaScript 是使用垃圾回收的编程语言，开发者不需要操心内存分配和回收。JavaScript 的垃圾回收
程序可以总结如下。
* 离开作用域的值会被自动标记为可回收，然后在垃圾回收期间被删除。
* 主流的垃圾回收算法是标记清理，即先给当前不使用的值加上标记，再回来回收它们的内存。
* 引用计数是另一种垃圾回收策略，需要记录值被引用了多少次。JavaScript 引擎不再使用这种算
法，但某些旧版本的 IE 仍然会受这种算法的影响，原因是 JavaScript 会访问非原生 JavaScript 对
象（如 DOM 元素）。
* *引用计数在代码中存在循环引用时会出现问题。
* *解除变量的引用不仅可以消除循环引用，而且对垃圾回收也有帮助。为促进内存回收，全局对
象、全局对象的属性和循环引用都应该在不需要时解除引用。

## 引用类型
* 都有 valueOf()、toLocaleString()和 toString()方法
### 基本引用类型/包装类型
* Date()
* RegExp    
    * let expression = /pattern/flags; new RegExp("pattern", "flags");
    * 主要方法 pattern.exec() test()
        * 每次调用 exec()都会返回字符串中的下一个匹配项，直到搜索到字符串末尾。注意模式的lastIndex 属性每次都会变化。
    * flags
        * g:全局模式，表示查找字符串的全部内容，而不是找到第一个匹配的内容就结束
        * i：不区分大小写，表示在查找匹配时忽略 pattern 和字符串的大小写。
        * m：多行模式，表示查找到一行文本末尾时会继续查找。
        * y：粘附模式，表示只查找从 lastIndex 开始及之后的字符串。
        * u：Unicode 模式，启用 Unicode 匹配。
        * s：dotAll 模式，表示元字符.匹配任何字符（包括\n 或\r）。
* String
    * slice(子字符串开始的位置,子字符串结束的位置). substring(子字符串开始的位置,子字符串结束的置). substr(子字符串开始的位置, 子字符串数量)
    * indexOf(字符, 查询开始位置)  lastIndexOf(字符, 查询开始位置)
    * startsWith(字符, 查询开始位置)  endsWith(字符, 查询开始位置)  includes(字符, 查询开始位置)
    * trim() 清除前后空格
    * repeat(重复数量)
    * padStart(最小填充长度, 填充字符串) padEnd(最小填充长度, 填充字符串)
    * toLowerCase()、toLocaleLowerCase()、toUpperCase()和toLocaleUpperCase()
    * 字符串自己的匹配方法
        * match()
        * search()
        * split(用什么字符串分割)
        * replace()
            ```
            text.replace(正则, 替换的字符串 );
            text.replace(正则, function(match, pos, originalText) { return xx } );
             
            ```
    * 字符串的原型上暴露了一个@@iterator 方法,使之可以迭代
        *```  stringIterator = str[Symbol.iterator](); stringIterator.next()  ```
* Boolean
    * 以读模式访问字符串值的任何时候，后台都会执行以下 3 步：(1) 创建一个 String 类型的实例；(2) 用实例上的特定方法；(3) 销毁实例。
* Number
    * toFixed(2) 保留小数 四舍五入
    * 
### 单例内置对象
* Global
Global 对象是 ECMAScript 中最特别的对象，因为代码不会显式地访问它。ECMA-262 规定 Global对象为一种兜底对象，它所针对的是不属于任何对象的属性和方法 在全局作用域中定义的变量和函数都会变成 Global 对象的属性
    * encodeURI()和 encodeURIComponent()
    * eval() 当解释器发现 eval()调用时，会将参数解释为实际的 ECMAScript 语句，然后将其插入到该位置。通过 eval()执行的代码属于该调用所在上下文，被执行的代码与该上下文拥有相同的作用域链。
        * 通过 eval()定义的任何变量和函数都不会被提升
    * Global 对象属性...
    * window对象
        * 虽然 ECMA-262 没有规定直接访问 Global 对象的方式，但浏览器将 window 对象实现为 Global对象的代理。因此，所有全局作用域中声明的变量和函数都变成了 window 的属性
* Math
    * min()和 max()
    * Math.ceil(向上)、Math.floor(向下)、Math.round(四舍五入) 、Math.fround(最接近的单精度的)
    * random() 始终返回小数 为加密的可以使用 window.crypto.getRandomValues()
    * 其他 abs exp 等等
### 集合引用类型
#### Object 对象
* 如何创建 new关键字 或 对象字面量
    ```
    let person = new Object();
    let person = { 
            name: "Nicholas", 
            age: 29 
        };
    ```
    * 在使用对象字面量表示法定义对象时，并不会实际调用 Object 构造函数。
#### Array 数组
* 如何创建数组, 可以省略new
    ```
        let colors = new Array(20); 创建大小为20的数组
        let colors = new Array("red", "blue", "green");
    ```
    * 在使用数组字面量表示法创建数组不会调用 Array 构造函数
* 方法
    * Array.from(类数组, 增强函数 function(x){  return x }  ，this指向  ) 将类数组结构转换为数组实例
        * 对现有数组执行浅复制
        * 可以使用任何可迭代对象   即包含 ``` *[Symbl.iterator]()```
        * 函数的arguments 也可以
    * of()  将一组参数转换为数组实例
        ```
        console.log(Array.of(1, 2, 3, 4)); // [1, 2, 3, 4] 
        console.log(Array.of(undefined)); // [undefined]
        ```
    * Array.isArray() 检测是不是数组  替代 xxx instanceof Array
    * 迭代器方法  keys()、values()、entries() 索引/值对
    * 批量复制方法 copyWithin(插入位置, 复制起点,复制终点) 填充数组方法 fill()
    * join()：如果不给 join()传入任何参数，或者传入 undefined，则仍然使用逗号作为分隔符  、toLocaleString()、toString()：和不传参的join一样 、valueOf()：返回本身
        * 如果数组中某一项是 null 或 undefined，则返回的结果中会以空字符串表示
    * 栈方法 后入先出   push()尾插入 pop() 尾弹出
    * 队列方法 先入先出 push()尾插入 shift() 首弹出     unshift 首插入
    * 排序 reverse()反转  ； sort(v1 v2 =>  v1在前 返回负数  ) 排序  默认从小到大排 
    * 操作方法
        * concat 连接 :  新数组 = 老数组1.concat(?新的项, 老数组2)
            * 如果传递数组 会打平 如果不需要打平可以重写
            * 在参数数组上指定一个特殊的符号：Symbol.isConcatSpreadable。这个符号能够阻止 concat()打平参数数组。相反，把这个值设置为 true 可以强制打平类数组对象
            ```
                let moreNewColors = { 
                    [Symbol.isConcatSpreadable]: true, 
                    length: 2, 
                    0: "pink", 
                    1: "cyan" 
                }; 
                newColors[Symbol.isConcatSpreadable] = false;  不打平
            
            ```
        * slice 不影响原始数组 创建新数组  slice(开始索引,结束索引) 
        * splice 直接改变原始数组 插入 删除 替换 :  splice(删除元素的位置, 删除元素的数量,....插入的字符串)
    * 严格相等的搜索方法 indexOf()、lastIndexOf()、 includes()
    * 断言方法 find()和 findIndex()  找到匹配项后 不再继续搜索
        ```
        数组.find((element, index, array) => 返回true就停止继续 , this的指向)
        ```
    * 迭代方法 不改变调用它们的数组
        * every()：对数组每一项都运行传入的函数，如果对每一项函数都返回 true，则这个方法返回 true。
        * some()：对数组每一项都运行传入的函数，如果有一项函数返回 true，则这个方法返回 true。
        * filter()：对数组每一项都运行传入的函数，函数返回 true 的项会组成数组之后返回。
        * forEach()：对数组每一项都运行传入的函数，没有返回值。
        * map()：对数组每一项都运行传入的函数，返回由每次函数调用的结果构成的数组。
        ```
         Arr.map(function(数组元素、元素索引,数组本身) =>{ } ,作用域对象 )   
        ```
    * 归并方法 
        * reduce() reduceRight() :迭代数组的所有项，并在此基础上构建一个最终返回值。
            * 归并初始值默认是数组第一项， 数组从第二项开始归并
        ```
        Arr.reduce(function(上一个归并值，当前项，当前项的索引, 数组本身){ return 归并值}, 初始值)
        ```
#### 定型数组
* 定型数组（Typed Arrays）是 JavaScript 中的一种特殊数据结构，主要用于处理二进制数据
* ArrayBuffer 是所有定型数组及视图引用的基本单位。 实际上是分配的一块内存空间
    * 定型数组是另一种形式的 ArrayBuffer 视图, 其特定于一种 ElementType 且遵循系统原生的字节序
    * 设计定型数组的目的就是提高与 WebGL 等原生库交换二进制数据的效率。由于定型数组的二进制表示对操作系统而言是一种容易使用的格式，JavaScript 引擎可以重度优化算术运算、按位运算和其他对定型数组的常见操作，因此使用它们速度极快
    * 创建定型数组的方式包括读取已有的缓冲、使用自有缓冲、填充可迭代结构，以及填充基于任意类
型的定型数组
* 特点
    * 类型明确：定型数组具有明确的数据类型，如 Int8Array（8 位有符号整数）、Uint32Array（32 位无符号整数）、Float64Array（64 位浮点数）等。这使得数据在内存中的存储更加紧凑和高效，并且避免了 JavaScript 中普通数组可能出现的类型不确定性。
    * 高效的内存访问：由于数据类型固定，定型数组可以更高效地进行内存访问和操作。这对于处理大量的数字数据或进行高性能的计算非常有用。
    * 与 WebGL 和其他底层 API 的兼容性：定型数组在与 WebGL（用于在网页上进行图形渲染的 API）等底层技术交互时非常方便。可以直接将定型数组中的数据传递给这些 API，而无需进行额外的类型转换。
* 创建和使用
    * 创建定型数组：
        ```
        // 创建一个包含 5 个 8 位有符号整数的定型数组
        const int8Array = new Int8Array(5);
        // 创建一个长度为 3 的 32 位无符号整数定型数组，并初始化为 [1, 2, 3]
        const uint32Array = new Uint32Array([1, 2, 3]);
        ```
    * 访问和修改元素：
        ```
        int8Array[0] = 10;
        const value = uint32Array[1];
        ```
    * 与普通数组的转换：
        ```
        // 将定型数组转换为普通数组
        const normalArray = Array.from(int8Array);
        // 将普通数组转换为定型数组
        const newInt8Array = new Int8Array(normalArray);
        ```
* 应用场景
    * 图形处理和游戏开发：在处理图像数据、顶点坐标等方面，定型数组可以高效地存储和操作大量的数字数据。
    * 科学计算和数据分析：对于需要进行大规模数值计算的场景，定型数组可以提供更高的性能和内存效率。
    * 网络通信和数据传输：在处理二进制数据的网络通信中，定型数组可以方便地进行数据的打包和解包。
#### Map 映射 有顺序的 键/值存储 
Map 是一种新的集合类型，真正的键/值存储机制, 映射关系
* 初始化
    * 如果想在创建的同时初始化实例，可以给 Map 构造函数传入一个可迭代对象，需要包含键/值对数
组
    ```
    const m = new Map();

    // 使用嵌套数组初始化映射
    const m1 = new Map([ 
    ["key1", "val1"], 
    ["key2", "val2"], 
    ["key3", "val3"] 
    ]); 
    alert(m1.size); // 3 
    // 使用自定义迭代器初始化映射
    const m2 = new Map({ 
    [Symbol.iterator]: function*() { 
    yield ["key1", "val1"]; 
    yield ["key2", "val2"]; 
    yield ["key3", "val3"]; 
    } 
    }); 
    alert(m2.size); // 3
    ```
* 操作: 可以使用 set()方法再添加/修改键/值对。另外，可以使用 get()和 has()进行查询，可以通过 size 属性获取映射中的键/值对的数量，还可以使用 delete()和 clear()删除值
* 说明
    * Map 可以使用任何 JavaScript 数据类型作为键
    * Map 实例会维护键值对的插入顺序，因此可以根据插入顺序执行迭代操作
    * Map 只能通过 set 方法修改,
    ```
    const m = new Map([ 
        ["key1", "val1"], 
        ["key2", "val2"], 
        ["key3", "val3"] 
    ]);
    m.entries === m[Symbol.iterator] true
    ```
#### WeakMap 弱映射 增强的键/值对存储机制
* 使用和Map基本一致，不可迭代
* 弱映射中的键只能是 Object 或者继承自 Object 的类型
* 弱代表 键不会阻止垃圾回收, 避免内存泄漏
* 之所以限制只有对象作为键, 为了保证只有通过键对象的引用才能取得值
* 应用场景：保护私有属性, 当结点被释放, 那么值也被释放

#### Set 集合 有顺序的不重复的
Set相似于只存储key的Map, 值不重复
* 可以使用 add()增加值，使用 has()查询，通过 size 取得元素数量，以及使用 delete()
和 clear()删除元素
* add()和 delete()操作是幂等的。delete()返回一个布尔值，表示集合中是否存在要删除的值：
    ```
    // 使用数组初始化集合 
    const s1 = new Set(["val1", "val2", "val3"]); 
    alert(s1.size); // 3 
    // 使用自定义迭代器初始化集合
    const s2 = new Set({ 
        [Symbol.iterator]: function*() { 
            yield "val1"; 
            yield "val2"; 
            yield "val3"; 
        } 
    }); 
    alert(s2.size); // 3
    ```
* 可以迭代 类似于Map
* 可以自己封装继承，增强Set 功能，但是需要注意的是
    * 某些 Set 操作是有关联性的，因此最好让实现的方法能支持处理任意多个集合实例。
    * Set 保留插入顺序，所有方法返回的集合必须保证顺序。
    * 尽可能高效地使用内存。扩展操作符的语法很简洁，但尽可能避免集合和数组间的相互转换能够节省对象初始化成本。
    * 不要修改已有的集合实例。union(a, b)或 a.union(b)应该返回包含结果的新集合实例。
#### WeakSet 弱集合 有顺序 不可迭代
* 只能用对象作为值，与上面类似
* 应用场景， 例如维护一个已注销集合, 当内容被移除时,会自动缩减,而不用手动判断
    ```
        const disabledElements = new WeakSet(); 
        const loginButton = document.querySelector('#login'); 
        // 通过加入对应集合，给这个节点打上“禁用”标签
        disabledElements.add(loginButton);
        // 只要 WeakSet 中任何元素从 DOM 树中被删除，垃圾回收程序就可以忽略其存在，而立即
        释放其内存（假设没有其他地方引用这个对象）
    ```
#### 扩展操作符
    ```
        let arr1 = [1, 2, 3]; 
        let arr2 = [0, ...arr1, 4, 5]; 
        console.log(arr2); // [0, 1, 2, 3, 4, 5]
    ```
## 迭代器与生成器
迭代的英文“iteration”源自拉丁文 itero，意思是“重复”或“再来”
* 任何实现 Iterable 接口的数据结构都可以被实现 Iterator 接口的结构“消费”（consume）。迭
代器（iterator）是按需创建的一次性对象。每个迭代器都会关联一个可迭代对象，而迭代器会暴露迭代
其关联可迭代对象的 API。迭代器无须了解与其关联的可迭代对象的结构，只需要知道如何取得连续的
值
### 可迭代协议
* 实现 Iterable 接口（可迭代协议）要求同时具备两种能力：支持迭代的自我识别能力和创建实现
Iterator 接口的对象的能力。在 ECMAScript 中，这意味着必须暴露一个属性作为“默认迭代器”，而
且这个属性必须使用特殊的 Symbol.iterator 作为键。这个默认迭代器属性必须引用一个迭代器工厂
函数，调用这个工厂函数必须返回一个新迭代器
    * 很多内置类型都实现了 Iterable 接口
        * 字符串 数组 映射 集合 arguments NodeList 等 DOM 集合类型
    * 检查是否存在迭代器属性
        ```
         obj[Symbol.Iterator]    暴露出迭代器工厂
         obj[Symbol.Iterator]()  调用会生成一个迭代器
        ```
    * 原生语言结构会在后台调用提供的可迭代对象的这个工厂函数，从而创建一个迭代器。接收可迭代对象的原生语言特性包括
        * for-of 循环
        * 数组解构
        * 扩展操作符
        * Array.from()
        * 创建集合
        * 创建映射
        * Promise.all()接收由期约组成的可迭代对象
        * Promise.race()接收由期约组成的可迭代对象
        * yield*操作符，在生成器中使用
### 迭代器协议
* 迭代器是一种一次性使用的对象，用于迭代与其关联的可迭代对象。迭代器 API 使用 next()方法
在可迭代对象中遍历数据。每次成功调用 next()，都会返回一个 IteratorResult 对象，其中包含迭
代器返回的下一个值。若不调用 next()，则无法知道迭代器的当前位置。
* next()方法返回的迭代器对象 IteratorResult 包含两个属性：done 和 value。done 是一个布
尔值，表示是否还可以再次调用 next()取得下一个值；value 包含可迭代对象的下一个值（done 为
false），或者 undefined（done 为 true）。done: true 状态称为“耗尽”。
    ```
        let arr = ['foo']; 
        let iter = arr[Symbol.iterator](); 
        console.log(iter.next()); // { done: false, value: 'foo' } 
        console.log(iter.next()); // { done: true, value: undefined } 
        console.log(iter.next()); // { done: true, value: undefined } 
        console.log(iter.next()); // { done: true, value: undefined }
    ```
    * 迭代器并不与可迭代对象某个时刻的快照绑定，而仅仅是使用游标来记录遍历可迭代对象的历程。
如果可迭代对象在迭代期间被修改了，那么迭代器也会反映相应的变化
    * 迭代器维护着一个指向可迭代对象的引用，因此迭代器会阻止垃圾回收程序回收可
迭代对象。
    * 当包含 return { done: true } 时 提前结束
### 自定义迭代器
```
class myIterator {
    constructor(max){
        this.max = max
    }
    [Symbol.Iterator](){
        let i = 0
        return {
            next: () => {  
                if(i < this.max) return { value:i++, done: false  }
                return { value: this.max, done: true }
             } 
        }
   }
}

let b = {
    max:10,
    [Symbol.Iterator] : function(){
        let i = 0;
        return {
            next: () =>{ 
                if(i < this.max) return { value:i++, done: false  }
                return { value: this.max, done: true }   
            }
        }
    }
}
```
### 生成器 
拥有在一个函数块内暂停和恢复代码执行的能力
* 调用生成器函数会产生一个生成器对象。生成器对象一开始处于暂停执行（suspended）的状态。与
迭代器相似，生成器对象也实现了 Iterator 接口，因此具有 next()方法。调用这个方法会让生成器
开始或恢复执行。
```
// 生成器函数声明
function* generatorFn() {} 
// 生成器函数表达式
let generatorFn = function* () {} 
// 作为对象字面量方法的生成器函数
let foo = { 
 * generatorFn() {} 
} 
// 作为类实例方法的生成器函数
class Foo { 
 * generatorFn() {} 
} 
// 作为类静态方法的生成器函数
class Bar { 
 static * generatorFn() {}
}


function* generatorFn() {}
const g = generatorFn(); 
console.log(g === g[Symbol.iterator]());
```

* yield会暂停程序的执行 暂时的返回一个迭代器 使用 yield 输入和输出
* yield 关键字可以将跟在它后面的可迭代对象序列化为一连串值
```
function* gen() {
    yield 1
    yield 2
    yield 3
    yield 4
    return yield 5
}
let b = gen()
b.next(99)
1 2 3 4 5 99
```
* 可以使用星号增强 yield 的行为，让它能够迭代一个可迭代对象，从而一次产出一个值
```
    function* generatorFn() { 
    yield* [1, 2]; 
    yield *[3, 4]; 
    yield * [5, 6]; 
    } 
    for (const x of generatorFn()) { 
    console.log(x); 
    } 
    // 1 
    // 2 
    // 3 
    // 4 
    // 5 
    // 6
    // 等价的 generatorFn： 
    // function* generatorFn() { 
    // for (const x of [1, 2, 3]) { 
    // yield x; 
    // } 
    // }
```
* 生成器用于递归：非常简洁的深度优先遍历
```
    const visitedNodes = new Set(); 
    function* traverse(nodes) { 
        for (const node of nodes) { 
            if (!visitedNodes.has(node)) { 
                yield node; 
                yield* traverse(node.neighbors); 
            } 
        } 
    } 
    // 取得集合中的第一个节点
    const firstNode = this.nodes[Symbol.iterator]().next().value; 
    // 使用递归生成器迭代每个节点
    for (const node of traverse([firstNode])) { 
        visitedNodes.add(node); 
    } 
    return visitedNodes.size === this.nodes.size;

```
* 强制关闭
```
   const g = generatorFn(); 
    console.log(g); // generatorFn {<suspended>} 
    console.log(g.return(4)); // { done: true, value: 4 } 
    console.log(g); // generatorFn {<closed>}

```

### 生成器作为默认迭代器

```
迭代器写法
class a{
    constructor() { 
        this.values = [1, 2, 3];
    }
    [Symbol.Iterator]() {
        let i = 0
        return {
            next: () =>{
                if(i < this.values.length) { 
                    return { value: this.values[i++], done:false}   
                }
                return { value: undefined, done: true} 

            }
        }
    }

}
相当于
class a{
    constructor() { 
        this.values = [1, 2, 3];
    }
    *[Symbol.Iterator]() {
       yield * this.values
    }

}

```

## 对象和类
* ECMA-262 将对象定义为一组属性的无序集合。严格来说，这意味着对象就是一组没有特定顺序的
值。对象的每个属性或方法都由一个名称来标识，这个名称映射到一个值。正因为如此（以及其他还未
讨论的原因），可以把 ECMAScript 的对象想象成一张散列表，其中的内容就是一组名/值对，值可以是
数据或者函数
### ECMAScript 中原型的本质

```
function Person(){} 
let p1 = new Person
Person.prototype.constructor == Person
p1.constructor == Person
p1.__proto__ == Person.prototype
```
 * 实例只有指向原型的指针，没有指向构造函数的指针

`

无论何时，只要创建一个函数，就会按照特定的规则为这个函数创建一个 prototype 属性（指向
原型对象）。默认情况下，所有原型对象自动获得一个名为 constructor 的属性，指回与之关联的构
造函数。对前面的例子而言，Person.prototype.constructor 指向 Person。然后，因构造函数而
异，可能会给原型对象添加其他属性和方法。
在自定义构造函数时，原型对象默认只会获得 constructor 属性，其他的所有方法都继承自
Object。每次调用构造函数创建一个新实例，这个实例的内部[[Prototype]]指针就会被赋值为构
造函数的原型对象。脚本中没有访问这个[[Prototype]]特性的标准方式，但 Firefox、Safari 和 Chrome
会在每个对象上暴露__proto__属性，通过这个属性可以访问对象的原型。在其他实现中，这个特性
完全被隐藏了。关键在于理解这一点：实例与构造函数原型之间有直接的联系，但实例与构造函数之
间没有。
`
![alt text](/img/prototype.png)
### 创建对象
* 按照惯例，构造函数名称的首字母都是要大写的，非构造函数则以小写字母开头
* 通过 new创建
    ```
        let person = new Object(); 
        person.name = "Nicholas"; 
        person.age = 29; 
        person.job = "Software Engineer"; 
        person.sayName = function() { 
        console.log(this.name); 
        };
    ```
* 字面量创建
    ```
    let person = { 
        name: "Nicholas", 
        age: 29, 
        job: "Software Engineer", 
        sayName() { 
            console.log(this.name); 
        } 
    };
     ```
* 工厂模式
    ```
        function createPerson(name,gender,sayName){
            let o = new Object()
            o.name = name;
            o.gender = gender;
            o.sayName = function (){ this.name  }
            return o
        }

    ```
* 构造函数模式

    `构造函数与普通函数唯一的区别就是调用方式不同。除此之外，构造函数也是函数。并没有把某个
函数定义为构造函数的特殊语法。任何函数只要使用 new 操作符调用就是构造函数，而不使用 new 操
作符调用的函数就是普通函数`

    要创建 Person 的实例，应使用 new 操作符。以这种方式调用构造函数会执行如下操作。
    * 在内存中创建一个新对象。
    * 这个新对象内部的[[Prototype]]特性被赋值为构造函数的 prototype 属性。
    * 构造函数内部的 this 被赋值为这个新对象（即 this 指向新对象）。
    * 执行构造函数内部的代码（给新对象添加属性）。
    * 如果构造函数返回非空对象，则返回该对象；否则，返回刚创建的新对象。
    ```
        function Person(name,gender){
            this.name = name;
            this.gender = gender;
            this.sayName = function(){  console.log(this.name);  }
        }
    let person1 = new Person("Nicholas", 29, "Software Engineer"); 
    let person2 = new Person("Greg", 27, "Doctor"); 
    person1.sayName(); // Nicholas 
    person2.sayName(); // Greg

    ```
* 原型模式

    `每个函数都会创建一个 prototype 属性，这个属性是一个对象，包含应该由特定引用类型的实例
共享的属性和方法。实际上，这个对象就是通过调用构造函数创建的对象的原型。使用原型对象的好处
是，在它上面定义的属性和方法可以被对象实例共享。原来在构造函数中直接赋给对象实例的值，可以
直接赋值给它们的原型`

    ```
        function Person() {} 
        Person.prototype.name = "Nicholas"; 
        Person.prototype.age = 29; 
        Person.prototype.job = "Software Engineer"; 
        Person.prototype.sayName = function() { 
        console.log(this.name); 
        }; 
        let person1 = new Person(); 
        person1.sayName(); // "Nicholas" 
        let person2 = new Person(); 
        person2.sayName(); // "Nicholas" 
        console.log(person1.sayName == person2.sayName); // true

    ```
* Object.create() 创建对象

### 属性
属性分两种：数据属性和访问器属性。
* 数据属性
数据属性包含一个保存数据值的位置。值会从这个位置读取，也会写入到这个位置。数据属性有 4
个特性描述它们的行为。
    * [[Configurable]]：表示属性是否可以通过 delete 删除并重新定义，是否可以修改它的特
    性，以及是否可以把它改为访问器属性。默认情况下，所有直接定义在对象上的属性的这个特
    性都是 true，如前面的例子所示。
    * [[Enumerable]]：表示属性是否可以通过 for-in 循环返回。默认情况下，所有直接定义在对
    象上的属性的这个特性都是 true，如前面的例子所示。
    * [[Writable]]：表示属性的值是否可以被修改。默认情况下，所有直接定义在对象上的属性的
    这个特性都是 true，如前面的例子所示。
    * [[Value]]：包含属性实际的值。这就是前面提到的那个读取和写入属性值的位置。这个特性
    的默认值为 undefined。
    * 要修改属性的默认特性，就必须使用 Object.defineProperty()方法。这个方法接收 3 个参数：
要给其添加属性的对象、属性的名称和一个描述符对象。最后一个参数，即描述符对象上的属性可以包
含：configurable、enumerable、writable 和 value，跟相关特性的名称一一对应。根据要修改
的特性，可以设置其中一个或多个值
        ```
        let person = {}; 
        Object.defineProperty(person, "name", { 
            writable: false, 
            value: "Nicholas" 
        }); 
        console.log(person.name); // "Nicholas" 
        person.name = "Greg"; 
        console.log(person.name); // "Nicholas"
        ```
* 访问器属性
访问器属性不包含数据值。相反，它们包含一个获取（getter）函数和一个设置（setter）函数，不
过这两个函数不是必需的。在读取访问器属性时，会调用获取函数，这个函数的责任就是返回一个有效
的值。在写入访问器属性时，会调用设置函数并传入新值，这个函数必须决定对数据做出什么修改。访
问器属性有 4 个特性描述它们的行为
    * [[Configurable]]：表示属性是否可以通过 delete 删除并重新定义，是否可以修改它的特
性，以及是否可以把它改为数据属性。默认情况下，所有直接定义在对象上的属性的这个特性
都是 true。
    * [[Enumerable]]：表示属性是否可以通过 for-in 循环返回。默认情况下，所有直接定义在对
象上的属性的这个特性都是 true。
    * [[Get]]：获取函数，在读取属性时调用。默认值为 undefined。
    * [[Set]]：设置函数，在写入属性时调用。默认值为 undefined。
    * 访问器属性是不能直接定义的，必须使用 Object.defineProperty()
        ```
            let book ={  year_: 2017, edition: 1  }
            Object.defineProperty(book, "year", { 
                get() { 
                    return this.year_; 
                }, 
                set(newValue) { 
                    if (newValue > 2017) { 
                        this.year_ = newValue; 
                        this.edition += newValue - 2017; 
                    } 
                } 
            }); 
            book.year = 2018; 
            console.log(book.edition); // 2

        ```
### 定义多个属性
* ECMAScript 提供了 Object.defineProperties()方法。这个方法可以通过多个描述符一次性定义多个属性。它接收两个参数：要为之添
加或修改属性的对象和另一个描述符对象，其属性与要添加或修改的属性一一对应
    ```
        let book = {}; 
        Object.defineProperties(book, { 
            year_: { 
            value: 2017 
            }, 
            edition: { 
            value: 1 
            }, 
            year: { 
                get() { 
                    return this.year_; 
                },
                set(newVal) {
                    return  newVal + 1
                }
            }
        })
    ```
* 简单重写
    ```
        let book = {
            value: 2017,
            get value(){
                return 10
            },
            set value(v){
                return 999
            }
        }

    ```
### 获取属性描述
* Object.getOwnPropertyDescriptor(obj,属性)
* Object.getOwnPropertyDescriptors(obj)
### 属性枚举顺序
* for-in 循环、Object.keys()、Object.getOwnPropertyNames()、Object.getOwnPropertySymbols()以及 Object.assign()
### 对象方法
* 合并对象 Object.assign()
    * 这个方法接收一个目标对象和一个
或多个源对象作为参数，然后将每个源对象中可枚举（Object.propertyIsEnumerable()返回 true）
和自有（Object.hasOwnProperty()返回 true）属性复制到目标对象。以字符串和符号为键的属性
会被复制。对每个符合条件的属性，这个方法会使用源对象上的[[Get]]取得属性的值，然后使用目标
对象上的[[Set]]设置属性的值。结果返回目标对象
    * Object.assign(目标对象,...多个源对象)实际上对每个源对象执行的是浅复制。如果多个源对象都有相同的属性，则使用最后一个复制的值。此外，从源对象访问器属性取得的值，比如获取函数，会作为一个静态值赋给目标对象。换句话说，不能在两个对象间转移获取函数和设置函数。

        ```

        ```
* Object.is()  增强判定方法, 相当于 === 但是更加严格 多了边界 例如 NaN 和 NaN 相等 
### 对象解构
* 可以提前声明解构的变量, 可以设置不存在的属性
* 可以设置默认参数
* 解构赋值可以使用嵌套结构，以匹配嵌套的属性
* 可以对函数参数解构，不影响 arguments
```
let personName, personAge; 
let person = { 
    name: 'Matt', 
    age: 27 
}; 
({name: personName, age: personAge, c= 999} = person); 
console.log(personName, personAge,c); // Matt, 27, 999
let person = { 
 name: 'Matt', 
 age: 27 
}; 
function printPerson(foo, {name, age}, bar) { 
 console.log(arguments); 
 console.log(name, age); 
} 
function printPerson2(foo, {name: personName, age: personAge}, bar) { 
 console.log(arguments); 
 console.log(personName, personAge); 
} 
printPerson('1st', person, '2nd'); 
// ['1st', { name: 'Matt', age: 27 }, '2nd'] 
// 'Matt', 27 
printPerson2('1st', person, '2nd'); 
// ['1st', { name: 'Matt', age: 27 }, '2nd'] 
// 'Matt', 27
```
### 继承与原型链
`
* ECMA-262 把原型链定义为 ECMAScript 的主要继承方式。其基本思想就是通过原型继承多个引用
类型的属性和方法。
* 重温一下构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型有
一个属性指回构造函数，而实例有一个内部指针指向原型。如果原型是另一个类型的实例呢？那就意味
着这个原型本身有一个内部指针指向另一个原型，相应地另一个原型也有一个指针指向另一个构造函
数。这样就在实例和原型之间构造了一条原型链。这就是原型链的基本构想。
`
* 任何函数的默认原型都是一个 Object 的实例，这意味着这个实例有一个内部指针指向
Object.prototype

![alt text](/img/inherit.png)

```
function Animal() {}
Animal.prototype.run = function(){ console.log("running") }

function Dog(){}
Dog.prototype = new Animal()

let dog1 = new Dog()
dog1.run()


```
### 类
* 在类块中定义的所有内容都会定义在类的原型上
* 派生类的方法可以通过 super 关键字引用它们的原型。在类构造函数中使用 super 可以调用父类构造函数
* 调用 super()会调用父类构造函数，并将返回的实例赋值给 this。
* 如果在派生类中显式定义了构造函数，则要么必须在其中调用 super()，要么必须在其中返回一个对象。
* 在类构造函数中，不能在调用 super()之前引用 this。
### 类的继承
* extends
### 组合胜过继承（composition over inheritance）
### 小结
对象在代码执行过程中的任何时候都可以被创建和增强，具有极大的动态性，并不是严格定义的实体。下面的模式适用于创建对象。
* 工厂模式就是一个简单的函数，这个函数可以创建对象，为它添加属性和方法，然后返回这个
对象。这个模式在构造函数模式出现后就很少用了。
* 使用构造函数模式可以自定义引用类型，可以使用 new 关键字像创建内置类型实例一样创建自
定义类型的实例。不过，构造函数模式也有不足，主要是其成员无法重用，包括函数。考虑到
函数本身是松散的、弱类型的，没有理由让函数不能在多个对象实例间共享。
* 原型模式解决了成员共享的问题，只要是添加到构造函数 prototype 上的属性和方法就可以共享。而组合构造函数和原型模式通过构造函数定义实例属性，通过原型定义共享的属性和方法。JavaScript 的继承主要通过原型链来实现。原型链涉及把构造函数的原型赋值为另一个类型的实例。这样一来，子类就可以访问父类的所有属性和方法，就像基于类的继承那样。原型链的问题是所有继承的属性和方法都会在对象实例间共享，无法做到实例私有。盗用构造函数模式通过在子类构造函数中调用父类构造函数，可以避免这个问题。这样可以让每个实例继承的属性都是私有的，但要求类型只能通过构造函数模式来定义（因为子类不能访问父类原型上的方法）。目前最流行的继承模式是组合继承，即通过原型链继承共享的属性和方法，通过盗用构造函数继承实例属性。
除上述模式之外，还有以下几种继承模式。
* 原型式继承可以无须明确定义构造函数而实现继承，本质上是对给定对象执行浅复制。这种操
作的结果之后还可以再进一步增强。
* 与原型式继承紧密相关的是寄生式继承，即先基于一个对象创建一个新对象，然后再增强这个
新对象，最后返回新对象。这个模式也被用在组合继承中，用于避免重复调用父类构造函数导
致的浪费。
* 寄生组合继承被认为是实现基于类型继承的最有效方式。
ECMAScript 6 新增的类很大程度上是基于既有原型机制的语法糖。类的语法让开发者可以优雅地定
义向后兼容的类，既可以继承内置类型，也可以继承自定义类型。类有效地跨越了对象实例、对象原型
和对象类之间的鸿沟。
## 代理和反射
`
ECMAScript 6 新增的代理和反射为开发者提供了拦截并向基本操作嵌入额外行为的能力。具体地
说，可以给目标对象定义一个关联的代理对象，而这个代理对象可以作为抽象的目标对象来使用。在对
目标对象的各种操作影响目标对象之前，可以在代理对象中对这些操作加以控制
`
* 代理是目标对象的抽象。从很多方面看，代理类似 C++指针，因为它可以
用作目标对象的替身，但又完全独立于目标对象。目标对象既可以直接被操作，也可以通过代理来操作。
但直接操作会绕过代理施予的行为

### 定义和使用
* 代理是使用 Proxy 构造函数创建的。这个构造函数接收两个参数：目标对象和处理程序对象。缺
少其中任何一个参数都会抛出 TypeError。
* 在代理对象上执行的任何操作实际上都会应用到目标对象。唯一可感知的不同
就是代码中操作的是代理对象
* 使用代理的主要目的是可以定义捕获器（trap）。捕获器就是在处理程序对象中定义的“基本操作的
拦截器”。每个处理程序对象可以包含零个或多个捕获器，每个捕获器都对应一种基本操作，可以直接
或间接在代理对象上调用。每次在代理对象上调用这些基本操作时，代理可以在这些操作传播到目标对
象之前先调用捕获器函数，从而拦截并修改相应的行为。
* 所有捕获器都可以访问相应的参数，基于这些参数可以重建被捕获方法的原始行为。比如，get()
捕获器会接收到目标对象、要查询的属性和代理对象三个参数。
    ```
    const target = { 
    foo: 'bar' 
    }; 
    const handler = { 
    get(trapTarget, property, receiver) { 
    return trapTarget[property]; 
    } 
    }; 
    const proxy = new Proxy(target, handler); 
    console.log(proxy.foo); // bar 
    console.log(target.foo); // bar
    ```
* 所有捕获器都可以基于自己的参数重建原始操作，但并非所有捕获器行为都像 get()那么简单。因
此，通过手动写码如法炮制的想法是不现实的。实际上，开发者并不需要手动重建原始行为，而是可以
通过调用全局 Reflect 对象上（封装了原始行为）的同名方法来轻松重建。
* 处理程序对象中所有可以捕获的方法都有对应的反射（Reflect）API 方法。这些方法与捕获器拦截
的方法具有相同的名称和函数签名，而且也具有与被拦截方法相同的行为。因此，使用反射 API 也可以
像下面这样定义出空代理对象
    ```
    const target = { 
        foo: 'bar' 
    }; 
    const handler = { 
        get() { 
            return Reflect.get(...arguments); 
        } 
        <!-- 也可以更简洁一些 -->
        get: Reflect.get
    }; 
    const proxy = new Proxy(target, handler); 
    console.log(proxy.foo); // bar 
    console.log(target.foo); // bar


    // 如果真想创建一个可以捕获所有方法，然后将每个方法转发给对应反射 API 的空代理，那
么甚至不需要定义处理程序对象
    const proxy = new Proxy(target, Reflect);
    ```
* 反射 API 为开发者准备好了样板代码，在此基础上开发者可以用最少的代码修改捕获的方法
### 捕获器不变式
* 使用捕获器几乎可以改变所有基本方法的行为，但也不是没有限制。根据 ECMAScript 规范，每个
捕获的方法都知道目标对象上下文、捕获函数签名，而捕获处理程序的行为必须遵循“捕获器不变式”
（trap invariant）。捕获器不变式因方法不同而异，但通常都会防止捕获器定义出现过于反常的行为
    ```
      let a = {a:1}
      Object.definedProprety(a,'a', { configurable:false, enumable: false, value: 2   })
      const  handler = new Proxy(a,{
        get() {
            return 'xxx'
        }
      })
        console.log(proxy.a); 
    // TypeError
    ```
### 可以撤销代理
* 有时候可能需要中断代理对象与目标对象之间的联系。对于使用 new Proxy()创建的普通代理来
说，这种联系会在代理对象的生命周期内一直持续存在。
* Proxy 也暴露了 revocable()方法，这个方法支持撤销代理对象与目标对象的关联。撤销代理的
操作是不可逆的。而且，撤销函数（revoke()）是幂等的，调用多少次的结果都一样。撤销代理之后
再调用代理会抛出 TypeError。
撤销函数和代理对象是在实例化时同时生成的
    ```
    const target = {
     foo: 'bar' 
    }; 
    const handler = { 
        get() { 
            return 'intercepted'; 
        } 
    }; 
    const { proxy, revoke } = Proxy.revocable(target, handler); 
    console.log(proxy.foo); // intercepted 
    console.log(target.foo); // bar 
    revoke(); 
    console.log(proxy.foo); // TypeError
    ```
### 实用的反射API
某些情况下应该优先使用反射 API，这是有一些理由的。
 * 反射 API 与对象 API 在使用反射 API 时，要记住：
    * 反射 API 并不限于捕获处理程序；
    * 大多数反射 API 方法在 Object 类型上有对应的方法。
通常，Object 上的方法适用于通用程序，而反射方法适用于细粒度的对象控制与操作。
* 状态标记
    * 很多反射方法返回称作“状态标记”的布尔值，表示意图执行的操作是否成功。有时候，状态标记
比那些返回修改后的对象或者抛出错误（取决于方法）的反射 API 方法更有用。例如，可以使用反射
API 对下面的代码进行重构：
        ```
        // 初始代码 
        const o = {}; 
        try { 
        Object.defineProperty(o, 'foo', 'bar'); 
        console.log('success'); 
        } catch(e) { 
        console.log('failure'); 
        } 
        在定义新属性时如果发生问题，Reflect.defineProperty()会返回 false，而不是抛出错误。
        因此使用这个反射方法可以这样重构上面的代码：
        // 重构后的代码
        const o = {}; 
        if(Reflect.defineProperty(o, 'foo', {value: 'bar'})) { 
        console.log('success'); 
        } else { 
        console.log('failure'); 
        }
        ```
* 以下反射方法都会提供状态标记：
    * *Reflect.defineProperty()
    * *Reflect.preventExtensions()
    * *Reflect.setPrototypeOf()
    * *Reflect.set()
    * *Reflect.deleteProperty()
* 用一等函数替代操作符
以下反射方法提供只有通过操作符才能完成的操作。
    * *Reflect.get()：可以替代对象属性访问操作符。
    * *Reflect.set()：可以替代=赋值操作符。
    * *Reflect.has()：可以替代 in 操作符或 with()。
    * *Reflect.deleteProperty()：可以替代 delete 操作符。
    * *Reflect.construct()：可以替代 new 操作符。
* 安全地应用函数

    `
    在通过 apply 方法调用函数时，被调用的函数可能也定义了自己的 apply 属性（虽然可能性极小）。
    为绕过这个问题，可以使用定义在 Function 原型上的 apply 方法，比如：
    Function.prototype.apply.call(myFunc, thisVal, argumentList); 
    这种可怕的代码完全可以使用 Reflect.apply 来避免：
    Reflect.apply(myFunc, thisVal, argumentsList);
    `
### 代理捕获器与反射方法
代理可以捕获 13 种不同的基本操作。这些操作有各自不同的反射 API 方法、参数、关联 ECMAScript
操作和不变式。
* get()
    
    get()捕获器会在获取属性值的操作中被调用。对应的反射 API 方法为 Reflect.get()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    get(target, property, receiver) { 
    console.log('get()'); 
    return Reflect.get(...arguments) 
    } 
    }); 
    proxy.foo; 
    // get() 
    ```
    * 返回值
    返回值无限制。
    * 拦截的操作
        * *proxy.property
        * *proxy[property]
        * *Object.create(proxy)[property]
        * *Reflect.get(proxy, property, receiver)
    * 捕获器处理程序参数
        * *target：目标对象。
        * *property：引用的目标对象上的字符串键属性。①
        * *receiver：代理对象或继承代理对象的对象。
    * 捕获器不变式
    如果 target.property 不可写且不可配置，则处理程序返回的值必须与 target.property 匹配。
    如果 target.property 不可配置且[[Get]]特性为 undefined，处理程序的返回值也必须是 undefined。

* set()
    set()捕获器会在设置属性值的操作中被调用。对应的反射 API 方法为 Reflect.set()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    set(target, property, value, receiver) { 
    console.log('set()'); 
    return Reflect.set(...arguments) 
    } 
    }); 
    proxy.foo = 'bar'; 
    // set() 
    ```
    1. 返回值
        * 返回 true 表示成功；返回 false 表示失败，严格模式下会抛出 TypeError。
    2. 拦截的操作
        * proxy.property = value
        * proxy[property] = value
        * Object.create(proxy)[property] = value
        * Reflect.set(proxy, property, value, receiver)
    3. 捕获器处理程序参数
        * target：目标对象。
        * property：引用的目标对象上的字符串键属性。
        * value：要赋给属性的值。
        * receiver：接收最初赋值的对象。
    4. 捕获器不变式
        * 如果 target.property 不可写且不可配置，则不能修改目标属性的值。
        * 如果 target.property 不可配置且[[Set]]特性为 undefined，则不能修改目标属性的值。
    在严格模式下，处理程序中返回 false 会抛出 TypeError。
* has()  
    has()捕获器会在 in 操作符中被调用。对应的反射 API 方法为 Reflect.has()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    has(target, property) { 
    console.log('has()'); 
    return Reflect.has(...arguments) 
    } 
    }); 
    'foo' in proxy; 
    // has() 
    ```
    1. 返回值
        * has()必须返回布尔值，表示属性是否存在。返回非布尔值会被转型为布尔值。
    2. 拦截的操作
        * property in proxy
        * property in Object.create(proxy)
        * with(proxy) {(property);}
        * Reflect.has(proxy, property)
    3. 捕获器处理程序参数
        * target：目标对象。
        * property：引用的目标对象上的字符串键属性。
    4. 捕获器不变式
        * 如果 target.property 存在且不可配置，则处理程序必须返回 true。
        * 如果 target.property 存在且目标对象不可扩展，则处理程序必须返回 true。
* defineProperty()  
    defineProperty()捕获器会在 Object.defineProperty()中被调用。对应的反射 API 方法为
    Reflect.defineProperty()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    defineProperty(target, property, descriptor) { 
    console.log('defineProperty()'); 
    return Reflect.defineProperty(...arguments) 
    } 
    }); 
    Object.defineProperty(proxy, 'foo', { value: 'bar' }); 
    // defineProperty() 
    ```
    1. 返回值
        * defineProperty()必须返回布尔值，表示属性是否成功定义。返回非布尔值会被转型为布尔值。
    2. 拦截的操作
        * Object.defineProperty(proxy, property, descriptor)
        * Reflect.defineProperty(proxy, property, descriptor)
    3. 捕获器处理程序参数
        * target：目标对象。
        * property：引用的目标对象上的字符串键属性。
        * descriptor：包含可选的 enumerable、configurable、writable、value、get 和 set
    定义的对象。
    4. 捕获器不变式
        * 如果目标对象不可扩展，则无法定义属性。
        * 如果目标对象有一个可配置的属性，则不能添加同名的不可配置属性。
        * 如果目标对象有一个不可配置的属性，则不能添加同名的可配置属性
* getOwnPropertyDescriptor()  
    getOwnPropertyDescriptor()捕获器会在 Object.getOwnPropertyDescriptor()中被调
    用。对应的反射 API 方法为 Reflect.getOwnPropertyDescriptor()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    getOwnPropertyDescriptor(target, property) { 
    console.log('getOwnPropertyDescriptor()'); 
    return Reflect.getOwnPropertyDescriptor(...arguments) 
    } 
    }); 
    Object.getOwnPropertyDescriptor(proxy, 'foo'); 
    // getOwnPropertyDescriptor() 
    ```
    1. 返回值
        * getOwnPropertyDescriptor()必须返回对象，或者在属性不存在时返回 undefined。
    2. 拦截的操作
        * Object.getOwnPropertyDescriptor(proxy, property)
        * Reflect.getOwnPropertyDescriptor(proxy, property)
    3. 捕获器处理程序参数
        * target：目标对象。
        * property：引用的目标对象上的字符串键属性。
    4. 捕获器不变式
        * 如果自有的 target.property 存在且不可配置，则处理程序必须返回一个表示该属性存在的
    对象。
        * 如果自有的 target.property 存在且可配置，则处理程序必须返回表示该属性可配置的对象。
        * 如果自有的 target.property 存在且 target 不可扩展，则处理程序必须返回一个表示该属性存
    在的对象。
        * 如果 target.property 不存在且 target 不可扩展，则处理程序必须返回 undefined 表示该属
    性不存在。
        * 如果 target.property 不存在，则处理程序不能返回表示该属性可配置的对象。

* deleteProperty()  
    deleteProperty()捕获器会在 delete 操作符中被调用。对应的反射 API 方法为 Reflect. 
    deleteProperty()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    deleteProperty(target, property) { 
    console.log('deleteProperty()'); 
    return Reflect.deleteProperty(...arguments) 
    } 
    }); 
    delete proxy.foo 
    // deleteProperty() 
    ```
    1. 返回值
        * deleteProperty()必须返回布尔值，表示删除属性是否成功。返回非布尔值会被转型为布尔值。
    2. 拦截的操作
        * delete proxy.property
        * delete proxy[property]
        * Reflect.deleteProperty(proxy, property)
    3. 捕获器处理程序参数
        * target：目标对象。
        * property：引用的目标对象上的字符串键属性。
    4. 捕获器不变式
        * 如果自有的 target.property 存在且不可配置，则处理程序不能删除这个属性

* ownKeys()  
    ownKeys()捕获器会在 Object.keys()及类似方法中被调用。对应的反射 API 方法为 Reflect. 
    ownKeys()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    ownKeys(target) { 
    console.log('ownKeys()'); 
    return Reflect.ownKeys(...arguments) 
    } 
    }); 
    Object.keys(proxy); 
    // ownKeys() 
    ```
    1. 返回值
        * ownKeys()必须返回包含字符串或符号的可枚举对象。
    2. 拦截的操作
        * Object.getOwnPropertyNames(proxy)
        * Object.getOwnPropertySymbols(proxy)
        * Object.keys(proxy)
        * Reflect.ownKeys(proxy)
    3. 捕获器处理程序参数
        * target：目标对象。
    4. 捕获器不变式
        * 返回的可枚举对象必须包含 target 的所有不可配置的自有属性。
        * 如果 target 不可扩展，则返回可枚举对象必须准确地包含自有属性键。

* getPrototypeOf()  
    getPrototypeOf()捕获器会在 Object.getPrototypeOf()中被调用。对应的反射 API 方法为
    Reflect.getPrototypeOf()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    getPrototypeOf(target) { 
    console.log('getPrototypeOf()'); 
    return Reflect.getPrototypeOf(...arguments) 
    } 
    }); 
    Object.getPrototypeOf(proxy); 
    // getPrototypeOf() 
    ```
    1. 返回值
        * getPrototypeOf()必须返回对象或 null。
    2. 拦截的操作
        * Object.getPrototypeOf(proxy)
        * Reflect.getPrototypeOf(proxy)
        * proxy.__proto__
        * Object.prototype.isPrototypeOf(proxy)
        * proxy instanceof Object
    3. 捕获器处理程序参数
        * target：目标对象。
    4. 捕获器不变式
        *如果 target 不可扩展，则 Object.getPrototypeOf(proxy)唯一有效的返回值就是 Object. 
    getPrototypeOf(target)的返回值。

* setPrototypeOf()   
    setPrototypeOf()捕获器会在 Object.setPrototypeOf()中被调用。对应的反射 API 方法为
    Reflect.setPrototypeOf()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    setPrototypeOf(target, prototype) { 
    console.log('setPrototypeOf()'); 
    return Reflect.setPrototypeOf(...arguments) 
    } 
    }); 
    Object.setPrototypeOf(proxy, Object); 
    // setPrototypeOf() 
    ```
    1. 返回值
        * setPrototypeOf()必须返回布尔值，表示原型赋值是否成功。返回非布尔值会被转型为布尔值。
    2. 拦截的操作
        * Object.setPrototypeOf(proxy)
        * Reflect.setPrototypeOf(proxy)
    3. 捕获器处理程序参数
        * target：目标对象。
        * prototype：target 的替代原型，如果是顶级原型则为 null。
    4. 捕获器不变式
        * 如果 target 不可扩展，则唯一有效的 prototype 参数就是 Object.getPrototypeOf(target)的返回值
* isExtensible()  
    isExtensible()捕获器会在 Object.isExtensible()中被调用。对应的反射 API 方法为
    Reflect.isExtensible()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    isExtensible(target) { 
    console.log('isExtensible()');
    return Reflect.isExtensible(...arguments) 
    } 
    }); 
    Object.isExtensible(proxy); 
    // isExtensible() 
    ```
    1. 返回值
        * isExtensible()必须返回布尔值，表示 target 是否可扩展。返回非布尔值会被转型为布尔值。
    2. 拦截的操作
        * Object.isExtensible(proxy)
        * Reflect.isExtensible(proxy)
    3. 捕获器处理程序参数
        * target：目标对象。
    4. 捕获器不变式
        * 如果 target 可扩展，则处理程序必须返回 true。
        * 如果 target 不可扩展，则处理程序必须返回 false
* preventExtensions()  
    preventExtensions()捕获器会在 Object.preventExtensions()中被调用。对应的反射 API
    方法为 Reflect.preventExtensions()。
    ```
    const myTarget = {}; 
    const proxy = new Proxy(myTarget, { 
    preventExtensions(target) { 
    console.log('preventExtensions()'); 
    return Reflect.preventExtensions(...arguments) 
    } 
    }); 
    Object.preventExtensions(proxy); 
    // preventExtensions() 
    ```
    1. 返回值
        * preventExtensions()必须返回布尔值，表示 target 是否已经不可扩展。返回非布尔值会被转
    型为布尔值。
    2. 拦截的操作
        * Object.preventExtensions(proxy)
        * Reflect.preventExtensions(proxy)
    3. 捕获器处理程序参数
        * target：目标对象。
    4. 捕获器不变式
        * 如果 Object.isExtensible(proxy)是 false，则处理程序必须返回 true。
* apply()  
    apply()捕获器会在调用函数时中被调用。对应的反射 API 方法为 Reflect.apply()。
    ```
    const myTarget = () => {}; 
    const proxy = new Proxy(myTarget, { 
    apply(target, thisArg, ...argumentsList) { 
    console.log('apply()'); 
    return Reflect.apply(...arguments) 
    } 
    }); 
    proxy(); 
    // apply() 
    ```
    1. 返回值
        * 返回值无限制。
    2. 拦截的操作
        * proxy(...argumentsList)
        * Function.prototype.apply(thisArg, argumentsList)
        * Function.prototype.call(thisArg, ...argumentsList)
        * Reflect.apply(target, thisArgument, argumentsList)
    3. 捕获器处理程序参数
        * target：目标对象。
        * thisArg：调用函数时的 this 参数。
        * argumentsList：调用函数时的参数列表
    4. 捕获器不变式
        * target 必须是一个函数对象。
* construct()  
    construct()捕获器会在 new 操作符中被调用。对应的反射 API 方法为 Reflect.construct()。
    ```
    const myTarget = function() {}; 
    const proxy = new Proxy(myTarget, { 
    construct(target, argumentsList, newTarget) { 
    console.log('construct()'); 
    return Reflect.construct(...arguments) 
    } 
    }); 
    new proxy; 
    // construct() 
    ```
    1. 返回值
        * construct()必须返回一个对象。
    2. 拦截的操作
        * new proxy(...argumentsList)
        * Reflect.construct(target, argumentsList, newTarget)
    3. 捕获器处理程序参数
        * target：目标构造函数。
        * argumentsList：传给目标构造函数的参数列表。
        * newTarget：最初被调用的构造函数。
    4. 捕获器不变式
        * target 必须可以用作构造函数。
### 代理模式的作用  
* 跟踪属性访问  
    * 通过捕获 get、set 和 has 等操作，可以知道对象属性什么时候被访问、被查询。把实现相应捕获
    器的某个对象代理放到应用中，可以监控这个对象何时在何处被访问过：
    ```
    const user = { 
    name: 'Jake' 
    }; 
    const proxy = new Proxy(user, { 
    get(target, property, receiver) { 
    console.log(`Getting ${property}`); 
    return Reflect.get(...arguments); 
    }, 
    set(target, property, value, receiver) { 
    console.log(`Setting ${property}=${value}`); 
    return Reflect.set(...arguments); 
    } 
    }); 
    proxy.name; // Getting name 
    proxy.age = 27; // Setting age=27
    ```
* 隐藏属性
    代理的内部实现对外部代码是不可见的，因此要隐藏目标对象上的属性也轻而易举。比如：
    ```
    const hiddenProperties = ['foo', 'bar']; 
    const targetObject = { 
    foo: 1, 
    bar: 2, 
    baz: 3 
    }; 
    const proxy = new Proxy(targetObject, { 
    get(target, property) { 
    if (hiddenProperties.includes(property)) { 
    return undefined; 
    } else { 
    return Reflect.get(...arguments); 
    } 
    }, 
    has(target, property) {
        if (hiddenProperties.includes(property)) { 
    return false; 
    } else { 
    return Reflect.has(...arguments); 
    } 
    } 
    }); 
    // get() 
    console.log(proxy.foo); // undefined 
    console.log(proxy.bar); // undefined 
    console.log(proxy.baz); // 3 
    // has() 
    console.log('foo' in proxy); // false 
    console.log('bar' in proxy); // false 
    console.log('baz' in proxy); // true
    ```
* 属性验证
    * 因为所有赋值操作都会触发 set()捕获器，所以可以根据所赋的值决定是允许还是拒绝赋值：
    ```
    const target = { 
        onlyNumbersGoHere: 0 
    }; 
    const proxy = new Proxy(target, { 
        set(target, property, value) { 
            if (typeof value !== 'number') { 
                return false; 
            } else { 
                return Reflect.set(...arguments); 
            } 
        } 
    }); 
    proxy.onlyNumbersGoHere = 1; 
    console.log(proxy.onlyNumbersGoHere); // 1 
    proxy.onlyNumbersGoHere = '2'; 
    console.log(proxy.onlyNumbersGoHere); // 1
    ```
* 函数与构造函数参数验证
    * 跟保护和验证对象属性类似，也可对函数和构造函数参数进行审查。比如，可以让函数只接收某种
类型的值：
    ```
    function median(...nums) { 
        return nums.sort()[Math.floor(nums.length / 2)]; 
    } 
    const proxy = new Proxy(median, { 
    apply(target, thisArg, argumentsList) { 
        for (const arg of argumentsList) { 
            if (typeof arg !== 'number') { 
                throw 'Non-number argument provided'; 
            } 
        }
        return Reflect.apply(...arguments); 
        } 
    }); 
    console.log(proxy(4, 7, 1)); // 4 
    console.log(proxy(4, '7', 1)); 
    // Error: Non-number argument provided 
    类似地，可以要求实例化时必须给构造函数传参：
    class User { 
        constructor(id) { 
            this.id_ = id; 
        } 
    } 
    const proxy = new Proxy(User, { 
    construct(target, argumentsList, newTarget) { 
    if (argumentsList[0] === undefined) { 
        throw 'User cannot be instantiated without id'; 
    } else { 
        return Reflect.construct(...arguments); 
    } 
    } 
    }); 
    new proxy(1); 
    new proxy(); 
    // Error: User cannot be instantiated without id
    ```
* 数据绑定与可观察对象
    * 通过代理可以把运行时中原本不相关的部分联系到一起。这样就可以实现各种模式，从而让不同的
    代码互操作。
    比如，可以将被代理的类绑定到一个全局实例集合，让所有创建的实例都被添加到这个集合中：
    ```
    const userList = []; 
    class User { 
        constructor(name) { 
            this.name_ = name; 
        } 
    } 
    const proxy = new Proxy(User, { 
    construct() { 
    const newUser = Reflect.construct(...arguments); 
        userList.push(newUser); 
        return newUser; 
    } 
    }); 
    new proxy('John'); 
    new proxy('Jacob'); 
    new proxy('Jingleheimerschmidt'); 
    console.log(userList); // [User {}, User {}, User{}]
    另外，还可以把集合绑定到一个事件分派程序，每次插入新实例时都会发送消息：
    const userList = []; 
    function emit(newValue) { 
        console.log(newValue); 
    } 
    const proxy = new Proxy(userList, { 
        set(target, property, value, receiver) { 
        const result = Reflect.set(...arguments); 
        if (result) { 
            emit(Reflect.get(target, property, receiver)); 
        } 
        return result; 
    } 
    }); 
    proxy.push('John'); 
    // John 
    proxy.push('Jacob'); 
    // Jacob
    ```
### 总结
* 从宏观上看，代理是真实 JavaScript 对象的透明抽象层。代理可以定义包含捕获器的处理程序对象，
而这些捕获器可以拦截绝大部分 JavaScript 的基本操作和方法。在这个捕获器处理程序中，可以修改任
何基本操作的行为，当然前提是遵从捕获器不变式。
* 与代理如影随形的反射 API，则封装了一整套与捕获器拦截的操作相对应的方法。可以把反射 API
看作一套基本操作，这些操作是绝大部分 JavaScript 对象 API 的基础。
* 代理的应用场景是不可限量的。开发者使用它可以创建出各种编码模式，比如（但远远不限于）跟
踪属性访问、隐藏属性、阻止修改或删除属性、函数参数验证、构造函数参数验证、数据绑定，以及可
观察对象。

### 应用
在 JavaScript 中，Reflect是一个内置的对象，它提供了一组用于拦截 JavaScript 操作的方法，与Proxy对象配合使用，可以实现对对象属性访问、函数调用等操作的拦截和自定义行为。以下是Reflect的一些常用应用：
* 对象属性操作
    ```
    获取属性值
    使用Reflect.get()方法可以获取对象的属性值。
    示例：
    
    
        const obj = { name: 'John', age: 30 };
        const nameValue = Reflect.get(obj, 'name');
        console.log(nameValue); // 输出：John
    ```
    ```
    设置属性值
    Reflect.set()方法用于设置对象的属性值。
    示例：
        const obj = { name: 'John' };
        Reflect.set(obj, 'age', 30);
        console.log(obj); // 输出：{ name: 'John', age: 30 }
    ```
    ```
    判断是否存在属性
    Reflect.has()方法类似于in运算符，用于判断对象是否具有某个属性。
    示例：
        const obj = { name: 'John' };
        console.log(Reflect.has(obj, 'name')); // 输出：true
        console.log(Reflect.has(obj, 'age')); // 输出：false
    ```
* 函数调用
    ```
    调用函数
    Reflect.apply(要调用的函数,调用函数时的 this 值, 一个类数组对象，表示传递给函数的参数列表)方法用于调用一个函数，并指定this值和参数列表。
    示例：
        function add(a, b) {
            return a + b;
        }
        const result = Reflect.apply(add, null, [2, 3]);
        console.log(result); // 输出：5
     ```
     ```
    构造函数调用
    Reflect.construct()方法用于使用指定的构造函数和参数列表创建一个新对象。
    示例：
     class Person {
       constructor(name, age) {
         this.name = name;
         this.age = age;
       }
     }
     const person = Reflect.construct(Person, ['John', 30]);
     console.log(person); // 输出：Person { name: 'John', age: 30 }
    ```
* 元编程和拦截操作
    ```
    与Proxy配合使用
    Reflect的方法可以在Proxy的拦截器中使用，以实现对对象操作的自定义行为。
    示例：
     const target = { name: 'John', age: 30 };
     const handler = {
       get: function(target, property) {
         console.log(`Getting property ${property}`);
         return Reflect.get(target, property);
       },
       set: function(target, property, value) {
         console.log(`Setting property ${property} to ${value}`);
         return Reflect.set(target, property, value);
       }
     };
     const proxy = new Proxy(target, handler);
     console.log(proxy.name); // 输出：Getting property name，然后是 John
     proxy.age = 35; // 输出：Setting property age to 35
     console.log(proxy.age); // 输出：35
     ```
Reflect提供了一种更加标准化和简洁的方式来执行一些常见的对象操作和函数调用，同时也为元编程和拦截操作提供了强大的工具。

## 函数
函数实际上是对象。每个函数都是Function类型的实例，而 Function 也有属性和方法，跟其他引用类型一样。因为函数是对象，所以函数名就是指向函数对象的指针，而且不一定与函数本身紧密绑定

#### 如何定义函数
* 函数声明的方式定义
* 函数表达式，函数表达式与函数声明几乎是等价的
* 箭头函数
* 构造函数
    ```
     函数声明
        function sum(){}         具名函数
        let sum = function(){}   匿名函数
        () => {}                 箭头函数
        let sum = new Function(arg1,arg2, "return num1 + num2")
    ```
不推荐使用构造函数这种语法来定义函数，因为这段代码会被解释两次：第一次是将它当作常规
ECMAScript 代码，第二次是解释传给构造函数的字符串。这显然会影响性能。不过，把函数想象为对
象，把函数名想象为指针是很重要的

#### 箭头函数 请注意
* 箭头函数不能使用 arguments、super 和 new.target，也不能用作构造函数。此外，箭头函数也没有 prototype 属性
#### 函数名
* 函数名就是指向函数的指针，所以它们跟其他包含对象指针的变量具有相同的行为
* ECMAScript 6 的所有函数对象都会暴露一个只读的 name 属性，其中包含关于函数的信息。多数情
况下，这个属性中保存的就是一个函数标识符，或者说是一个字符串化的变量名。即使函数没有名称，
也会如实显示成空字符串。如果它是使用 Function 构造函数创建的，则会标识成"anonymous"
* 如果函数是一个获取函数、设置函数，或者使用 bind()实例化，那么标识符前面会加上一个前缀
```
function foo() {} 
let bar = function() {}; 
let baz = () => {}; 
console.log(foo.name); // foo 
console.log(bar.name); // bar 
console.log(baz.name); // baz 
console.log((() => {}).name); //（空字符串）
console.log((new Function()).name); // anonymous

function foo() {} 
console.log(foo.bind(null).name); // bound foo 
let dog = { 
    years: 1, 
    get age() { 
    return this.years; 
 }, 
 set age(newAge) { 
    this.years = newAge; 
 } 
} 
let propertyDescriptor = Object.getOwnPropertyDescriptor(dog, 'age'); 
console.log(propertyDescriptor.get.name); // get age 
console.log(propertyDescriptor.set.name); // set age
```
#### 参数
ECMAScript 函数的参数在内部表现为一个数组。函数被调用时总会接
收一个数组，但函数并不关心这个数组中包含什么。如果数组中什么也没有，那没问题；如果数组的元
素超出了要求，那也没问题
* ECMAScript 中的所有参数都按值传递的。不可能按引用传递参数。如果把对象作为参数传递，那么传递的值就是这个对象的引用
* 在使用 function 关键字定义（非箭头）函数时，可以在函数内部访问 arguments 对象，从中取得传进来的每个参数值
    * arguments 对象是一个类数组对象（但不是 Array 的实例），因此可以使用中括号语法访问其中的
元素（第一个参数是 arguments[0]，第二个参数是 arguments[1]）。而要确定传进来多少个参数，
可以访问 arguments.length 属性。
        ```
            function sum(a) {
                arguments.length
                argument[0]
                argument[1]
                a === argument[0]
            }
        ```
    * arguments 对象可以跟命名参数一起使用, 可以用于实现函数重载
        ```
        function doAdd(num1, num2) { 
            if (arguments.length === 1) { 
                console.log(num1 + 10); 
            } else if (arguments.length === 2) { 
                console.log(arguments[0] + num2); 
            } 
        }
        ```
    * arguments的值始终会与对应的命名参数同步，始终以调用函数时传入的值为准， 如果只传了一个参数，然后把 arguments[1]设置为某个值，那么这个值并不会反映到第二个命名参数。这是因为 arguments 对象的长度是根据传入的参数个数，而非定义函数时给出的命名参数个数确定的
* 默认参数
    * 默认参数的一种常用方式就是检测某个参数是否等于 undefined，如果是则意味着没有传这个参数，那就给它赋一个值
    * 函数的默认参数只有在函数被调用时才会求值，不会在函数定义时求值。而且，计算默认值的函数
    只有在调用函数但未传相应参数时才会被调用
    * 参数初始化顺序遵循“暂时性死区”规则，即前面定义的参数不能引用后面定义的
        ```
        // 调用时不传第一个参数会报错
        function makeKing(name = numerals, numerals = 'VIII') { 
            return `King ${name} ${numerals}`; 
        } 
        参数也存在于自己的作用域中，它们不能引用函数体的作用域：
        // 调用时不传第二个参数会报错
        function makeKing(name = 'Henry', numerals = defaultNumeral) { 
            let defaultNumeral = 'VIII'; 
            return `King ${name} ${numerals}`; 
        }
        ```
#### 函数重载
* ECMAScript 函数没有签名，因为参数是由包含零个或多个值的数组表示的。没有函数签名，自然也就没有重载
* 检查参数的类型和数量，然后分别执行不同的逻辑来模拟函数重载
#### 参数扩展与收集
扩展操作符最有用
的场景就是函数定义中的参数列表，在这里它可以充分利用这门语言的弱类型及参数长度可变的特点。
扩展操作符既可以用于调用函数时传参，也可以用于定义函数参数
* 扩展
    ```
        function sum() {}
        
        sum(...[1,2,3],4)
        相当于
        sum(1,2,3,4)

    ```
* 收集
    ```
        function sum(...args, end) {  
            args == [1,2,3]
            end == 4
        }
        
        sum(...[1,2,3],4)
        相当于
        sum(1,2,3,4)

    ```
#### 函数声明与函数表达式
JavaScript 引擎在任何代码执行之前，会先读取函数声明，并在执行上下文中
生成函数定义。而函数表达式必须等到代码执行到它那一行，才会在执行上下文中生成函数定义
* 函数声明会在任何代码执行之前先被读取并添加到执行上下文。这个过程叫作函数声明提升（function declaration hoisting）。在执行代码时，JavaScript 引擎会先执行一遍扫描，把发现的函数声明提升到源代码树的顶部。因此即使函数定义出现在调用它们的代码之后，引擎也会把函数声明提升到顶部
    ```
        // 可以提升   函数声明提升了
        function sum(num1, num2) { 
            return num1 + num2; 
        }
        // 提升的是普通变量, 不是函数声明。 会出错
        let sum = function() {}

    ```
#### 函数作为值
 因为函数名在 ECMAScript 中就是变量，所以函数可以用在任何可以使用变量的地方。这意味着不
仅可以把函数作为参数传给另一个函数，而且还可以在一个函数中返回另一个函数
### 函数内部
在 ECMAScript 5 中，函数内部存在两个特殊的对象：arguments 和 this。ECMAScript 6 又新增
了 new.target 属性。
#### arguments

arguments 是一个类数组对象，包含调用函数时传入的所有参数。这
个对象只有以 function 关键字定义函数（相对于使用箭头语法创建函数）时才会有。主要用于包
含函数参数
* arguments对象有arguments[0] length callee

* 使用 arguments.callee 就可以让函数逻辑与函数名解耦. callee是一个指向 arguments 对象所在函数的指针
    ```
        function factorial(num) { 
            if (num <= 1) { 
                return 1; 
            } else { 
                return num * arguments.callee(num - 1); 
                // 类似于  return num * factorial(num - 1);
            } 
        }
        // 这个重写之后的 factorial()函数已经用 arguments.callee 代替了之前硬编码的factorial。这意味着无论函数叫什么名称，都可以引用正确的函数
    ```
#### this
在标准函数中，this 引用的是把函数当成方法调用的上下文对象，这时候通常称其为 this 值
值（在网页的全局上下文中调用函数时，this 指向 windows）。

* 函数被调用时才能确定 this 到底引用哪个对象(this指向哪里)
* 箭头函数中，this引用的是定义箭头函数的上下文
#### caller
这个属性引用的是调用当前函数的函数，或者如果是
在全局作用域中调用的则为 null

```
function outer() {
    inner(); 
} 
function inner() { 
    console.log(inner.caller); 
 // console.log(arguments.callee.caller);
} 
outer();
```
#### new.target

ECMAScript 中的函数始终可以作为构造函数实例化一个新对象，也可以作为普通函数被调用。
ECMAScript 6 新增了检测函数是否使用 new 关键字调用的 new.target 属性

如果函数是正常调用的则 new.target 的值是 undefined；如果是使用 new 关键字调用的，则 new.target 将引用被调用的构造函数

### 函数属性与方法
ECMAScript 中的函数是对象，因此有属性和方法。每个函数都有两个属性：length
和 prototype
* length 属性保存函数定义的命名参数的个数
* prototype 属性 是保存引用类型所有实例方法的地方，这意味着 toString()、valueOf()等方法实际上都保存在 prototype 上，进而由所有实例共享。这个属性在自定义类型时特别重要。prototype 属性是不可枚举的，因此使用 for-in 循环不会返回这个属性
* apply()和 call() 这两个方法都会以指定的 this 值来调用函数，即会设
置调用函数时函数体内 this 对象的值。 两者只有传参方式不同
    * apply()方法接收两个参数： this 的值和一个参数数
组。第二个参数可以是 Array 的实例，但也可以是 arguments 对象
    * call(this指向, args1,args2) 方法接收多个个参数：函数内 this 的值和多个参数
    * 使用 call()或 apply()的好处是可以将任意对象设置为任意函数的作用域，这样对象可以不用关
心方法
* bind()   bind()方法会创建一个新的函数实例，其 this 值会被绑定到传给 bind()的对象
* valueOf()返回函数本身
* toLocaleString()和 toString()始终返回函数的代码

### 尾调用

ECMAScript 6 规范新增了一项内存管理优化机制，让 JavaScript 引擎在满足条件时可以重用栈帧, 即 一个函数的返回值是另一个返回值时 会有性能优化, 即一个函数内部的最后一项是返回另一个函数
```
function outerFunction() { 
    return innerFunction(); // 尾调用
}
```
```
// 无优化：尾调用没有返回 
function outerFunction() { 
    innerFunction(); 
} 
// 无优化：尾调用没有直接返回
function outerFunction() { 
    let innerFunctionResult = innerFunction(); 
    return innerFunctionResult; 
} 
// 无优化：尾调用返回后必须转型为字符串
function outerFunction() { 
    return innerFunction().toString(); 
} 
// 无优化：尾调用是一个闭包
function outerFunction() { 
    let foo = 'bar'; 
    function innerFunction() { return foo; } 
    return innerFunction(); 
} 
下面是几个符合尾调用优化条件的例子：
"use strict"; 
// 有优化：栈帧销毁前执行参数计算
function outerFunction(a, b) { 
    return innerFunction(a + b); 
} 
// 有优化：初始返回值不涉及栈帧
function outerFunction(a, b) { 
    if (a < b) { 
        return a; 
    } 
    return innerFunction(a + b); 
} 
// 有优化：两个内部函数都在尾部
function outerFunction(condition) { 
    return condition ? innerFunctionA() : innerFunctionB(); 
}
```

在 ES6 优化之前，执行这个例子会在内存中发生如下操作。  
    (1) 执行到 outerFunction 函数体，第一个栈帧被推到栈上。  
    (2) 执行 outerFunction 函数体，到 return 语句。计算返回值必须先计算 innerFunction。  
    (3) 执行到 innerFunction 函数体，第二个栈帧被推到栈上。  
    (4) 执行 innerFunction 函数体，计算其返回值。  
    (5) 将返回值传回 outerFunction，然后 outerFunction 再返回值。  
    (6) 将栈帧弹出栈外。  
在 ES6 优化之后，执行这个例子会在内存中发生如下操作。    
    (1) 执行到 outerFunction 函数体，第一个栈帧被推到栈上。  
    (2) 执行 outerFunction 函数体，到达 return 语句。为求值返回语句，必须先求值   innerFunction。  
    (3) 引擎发现把第一个栈帧弹出栈外也没问题，因为 innerFunction 的返回值也是 outerFunction
    的返回值。  
    (4) 弹出 outerFunction 的栈帧。  
    (5) 执行到 innerFunction 函数体，栈帧被推到栈上。  
    (6) 执行 innerFunction 函数体，计算其返回值。  
    (7) 将 innerFunction 的栈帧弹出栈外。  
很明显，第一种情况下每多调用一次嵌套函数，就会多增加一个栈帧。而第二种情况下无论调用多
少次嵌套函数，都只有一个栈帧。这就是 ES6 尾调用优化的关键：如果函数的逻辑允许基于尾调用将其
销毁，则引擎就会那么做。
### 闭包


当一个内部函数在其外部函数中被定义，并可以访问外部函数的变量时，就形成了一个闭包
```
    function compare(value1, value2) { 
        if (value1 < value2) { 
            return -1; 
        } else if (value1 > value2) { 
            return 1; 
        } else { 
            return 0; 
        } 
    } 
    let result = compare(5, 10);
```


函数执行时，每个执行上下文中都会有一个包含其中变量的对象。全局上下文中的叫变量对象，它
会在代码执行期间始终存在。而函数局部上下文中的叫活动对象，只在函数执行期间存在。在定义
compare()函数时，就会为它创建作用域链，预装载全局变量对象，并保存在内部的[[Scope]]中。在
调用这个函数时，会创建相应的执行上下文，然后通过复制函数的[[Scope]]来创建其作用域链。接着
会创建函数的活动对象（用作变量对象）并将其推入作用域链的前端。在这个例子中，这意味着 compare()
函数执行上下文的作用域链中有两个变量对象：局部变量对象和全局变量对象。作用域链其实是一个包含指针的列表，每个指针分别指向一个变量对象，但物理上并不会包含相应的对象

* 注意 this 和 arguments 都是不能直接在内部函数中访问的。如果想访问包含作用域中
的 arguments 对象，则同样需要将其引用先保存到闭包能访问的另一个变量中
    ```
    window.identity = 'The Window'; 
    let object = { 
        identity: 'My Object', 
        getIdentityFunc() { 
            let that = this; 
            return function() { 
                return that.identity; 
            }; 
        } 
    };
    console.log(object.getIdentityFunc()()); // 'My Object'
    ```
### 立即调用的函数表达式
立即调用的匿名函数又被称作立即调用的函数表达式（IIFE，Immediately Invoked Function 
Expression）。它类似于函数声明，但由于被包含在括号中，所以会被解释为函数表达式。紧跟在第一组
括号后面的第二组括号会立即调用前面的函数表达式

使用 IIFE 可以模拟块级作用域，即在一个函数表达式内部声明变量，然后立即调用这个函数。这
样位于函数体作用域的变量就像是在块级作用域中一样

```
// IIFE 
(function () { 
 for (var i = 0; i < count; i++) { 
    console.log(i); 
 } 
})(); 
console.log(i); // 抛出错误
```

在 ECMAScript 6 以后，IIFE 就没有那么必要了，因为块级作用域中的变量 let const 无须 IIFE 就可以实现同
样的隔离

### 小结

函数是 JavaScript 编程中最有用也最通用的工具。ECMAScript 6 新增了更加强大的语法特性，从而
让开发者可以更有效地使用函数。  
* 函数表达式与函数声明是不一样的。函数声明要求写出函数名称，而函数表达式并不需要。没
有名称的函数表达式也被称为匿名函数。
* ES6 新增了类似于函数表达式的箭头函数语法，但两者也有一些重要区别。
* JavaScript 中函数定义与调用时的参数极其灵活。arguments 对象，以及 ES6 新增的扩展操作符，
可以实现函数定义和调用的完全动态化。
* 函数内部也暴露了很多对象和引用，涵盖了函数被谁调用、使用什么调用，以及调用时传入了
什么参数等信息。
* JavaScript 引擎可以优化符合尾调用条件的函数，以节省栈空间。
* 闭包的作用域链中包含自己的一个变量对象，然后是包含函数的变量对象，直到全局上下文的
变量对象。
* 通常，函数作用域及其中的所有变量在函数执行完毕后都会被销毁。
* 闭包在被函数返回之后，其作用域会一直保存在内存中，直到闭包被销毁。
* 函数可以在创建之后立即调用，执行其中代码之后却不留下对函数的引用。
* 立即调用的函数表达式如果不在包含作用域中将返回值赋给一个变量，则其包含的所有变量都
会被销毁。
* 虽然 JavaScript 没有私有对象属性的概念，但可以使用闭包实现公共方法，访问位于包含作用域
中定义的变量。
* 可以访问私有变量的公共方法叫作特权方法。
* 特权方法可以使用构造函数或原型模式通过自定义类型中实现，也可以使用模块模式或模块增
强模式在单例对象上实现。

## 期约 与 异步函数
* 异步类似于系统终端, 期约的处理程序是先添加到消息队列，然后才逐个执行, 但是具体执行时间不可预知, 推入后函数将会结束
    ```
        setTimeout(() =>{} , 1000)
        // 1000毫秒后 将方法推入消息队列中， 等待执行
        // 推到队列之后，回调什么时候出列被执行是未知的
    ```

### promise
*  使用 new操作符 初始化期约  let p  =  new Promise(执行器函数executor)
* 期约状态机
    * 三种状态， 且不可以逆， 只能从 pending到resolved 或 pending到rejected
        * 待定 pending
        * 解决 resolved
        * 拒绝 rejected
    * 期约的状态是私有的，不能直接通过 JavaScript 检测到。这主要是为了避免根据读取到
的期约状态，以同步方式处理期约对象。另外，期约的状态也不能被外部 JavaScript 代码修改。这与不
能读取该状态的原因是一样的：期约故意将异步行为封装起来，从而隔离外部的同步代码
    * 由于期约的状态是私有的，所以只能在内部进行操作。内部操作在期约的执行器函数中完成。执行
器函数主要有两项职责：初始化期约的异步行为和控制状态的最终转换。其中，控制期约状态的转换是
通过调用它的两个函数参数实现的。这两个函数参数通常都命名为 resolve()和 reject()。调用
resolve()会把状态切换为兑现，调用 reject()会把状态切换为拒绝。另外，调用 reject()也会抛
出错误
        ```
            let p1 = new Promise((resolve, reject) => resolve()); 
            setTimeout(console.log, 0, p1); // Promise <resolved> 
            let p2 = new Promise((resolve, reject) => reject()); 
            setTimeout(console.log, 0, p2); // Promise <rejected>
            // Uncaught error (in promise)

            let p = new Promise((resolve, reject) => { 
                resolve();
                reject(); // 没有效果
            });
        ```
    * 执行器函数是同步执行的。这是因为执行器函数是期约的初始化程序
    * 无论 resolve()和 reject()中的哪个被调用，状态转换都不可撤销了
    * 期约并非一开始就必须处于待定状态，然后通过执行器函数才能转换为落定状态Promise.resolve 和 Promise.reject
        * Promise.resolve() 可以把任何值都转换为一个解决的期约, 具有幂等逻辑
        * Promise.reject()会实例化一个拒绝的期约并抛出一个异步错误
        ```
            // 下面两个期约实例实际上是一样的
            let p1 = new Promise((resolve, reject) => resolve()); 
            let p2 = Promise.resolve();

            // 使用Promise.resolve()，实际上可以把任何值都转换为一个解决的期约
            Promise.resolve(3)

            Promise.resolve(p) ===  Promise.resolve(Promise.resolve(p)) // true
        ```
    * promise是同步对象（在同步执行模式中使用），但也是异步执行模式的媒介。 拒绝期约的错误并没有抛到执行同步代码的线程里，而是通过浏览器异步消息队列来处理的。因此，try/catch 块并不能捕获该错误。代码一旦开始以异步模式执行，则唯一与之交互的方式就是使用异步结构——更具体地说，就是期约的方法
    * 在一个解决期约上调用 then()会把 onResolved 处理程序推进消息队列，但这个
处理程序在当前线程上的同步代码执行完成前不会执行。因此，跟在 then()后面的同步代码一定先于
处理程序执行。
* 期约的实例方法  
期约实例的方法是连接外部同步代码与内部异步代码之间的桥梁。这些方法可以访问异步操作返回
的数据，处理期约成功和失败的结果，连续对期约求值，或者添加只有期约进入终止状态时才会执行的
代码
    * then() catch() finally() 都将返回一个promise包装，默认是 Promise.resolve()
    *  实现 Thenable 接口
        ```
            class MyThenable { 
                then() {} 
            }
        ```
    * Promise.prototype.then(onResolved, onRejected)  
        * Promise.prototype.then()是为期约实例添加处理程序的主要方法。这个 then()方法接收最多
两个参数：onResolved 处理程序和 onRejected 处理程序。这两个参数都是可选的，如果提供的话，
则会在期约分别进入“兑现”和“拒绝”状态时执行  
        * 因为期约只能转换为最终状态一次，所以这两个操作一定是互斥的。
        * 传给 then()的任何非函数类型的参数都会被静默忽略
    * Promise.prototype.catch(onRejected)
        * 调用它就相当于调用 Promise.prototype.then(null, onRejected)
    * Promise.prototype.finally()
        * 与状态无关,清除冗余代码，无法知道是否完成或拒绝
        * 大多数情况下它将表现为父期约的传递
    * promise((resolve,reject)=> {   resolve('111') })  resolve(inner)的inner 会传递给then的 onResolved函数;  reject(inner)会传递给then和catch的 onRejected
* promise 的连锁与合成  
连锁就是一个期约接一个期约地拼接，合成则是将多个期约组合为一个期约
    * 连锁:  由于then() catch() finally() 都将返回一个promise包装，所以可以在一个结束之后返回一个新的promise用于连锁
        ```
            let p = new Promise((resolve,reject) =>{ console.log(1); setTimeout(resolve,1000)   })
            p.then(() =>{
                return new Promise((resolve) =>{  console.log(2); setTimeout(resolve,1000)      })
            }).then(() =>{ 
                return 3
            }).then((a) =>{ 

                console.log(a) // 3
            })
        ```
    * 合成： Promise.all()和 Promise.race()
        * Promise.all()静态方法创建的期约会在一组期约全部解决之后再解决。这个静态方法接收一个
可迭代对象，返回一个新期约
            ```
                // 一次拒绝会导致最终期约拒绝 // 第一个拒绝的期约会将自己的理由作为合成期约的拒绝理由
                let p2 = Promise.all([ 
                    Promise.resolve(), 
                    Promise.reject(123456), 
                    Promise.reject(789456), 
                    Promise.resolve() 
                ]); 
                setTimeout(console.log, 0, p2); // Promise <rejected>
                p2.catch((res) =>{  console.log(res)   })  // 123456
                // 全部完成则返回一个包含所有结果的数组
                let p = Promise.all([ 
                    Promise.resolve(3), 
                    Promise.resolve(), 
                    Promise.resolve(4) 
                ]);
                p.then((res) =>{  console.log(res)   })  // [3,undefined,4]

            ```
        * Promise.race()静态方法返回一个包装期约，是一组集合中最先解决或拒绝的期约的镜像。这个
方法接收一个可迭代对象，返回一个新期约
            * 不会对解决或拒绝的期约区别对待。无论是解决还是拒绝，只要是第一个落定的
期约，Promise.race()就会包装其解决值或拒绝理由并返回新期约
            ```
            // 解决先发生，超时后的拒绝被忽略
            let p1 = Promise.race([ 
                Promise.resolve(3), 
                new Promise((resolve, reject) => setTimeout(reject, 1000)) 
            ]); 
            setTimeout(console.log, 0, p1); // Promise <resolved>: 3
            ```
        * 将多个函数用promise串起来
            ``` 
                function addTwo(x) {return x + 2;} 
                function addThree(x) {return x + 3;} 
                function addFive(x) {return x + 5;} 
                function compose(...fns) { 
                return (x) => fns.reduce((promise, fn) => promise.then(fn), Promise.resolve(x)) 
                } 
                let addTen = compose(addTwo, addThree, addFive);
                addTen(8).then(console.log); // 18


            ```
### 异步函数
异步函数，也称为“async/await”（语法关键字）
* async 关键字用于声明异步函数。这个关键字可以用在函数声明、函数表达式、箭头函数和方法
* 异步函数如果使用 return 关键字返回了值（如果没有 return 则会返回 undefined），这
个值会被 Promise.resolve()包装成一个期约对象。异步函数始终返回期约对象，在函数外部调用这这个对象会得到它的返回值
* 用 await关键字可以暂停异步函数代码的执行，等待期约解决
* JavaScript 运行时在碰到 await 关键字时，会记录在哪里暂停执行。``等到 await 右边的值可用了``，JavaScript 运行时会向消息队列中推送一个任务，这个任务会恢复异步函数的执行
* 
### 应用
* 实现sleep
    ```
        async function sleep(delay) {
            return new promise((resolve) => settimeout(resolve,delay))
        }
        async function foo() { 
            const t0 = Date.now(); 
            await sleep(1500); // 暂停约 1500 毫秒
            console.log(Date.now() - t0); 
        } 
        foo();
    ```
* 实现异步并发
    ```
        async function request(id) {
            return new promise((reslove) =>{
                console.log(id, 'start')
                setTimeout(()=>{
                    reslove(id)
                    console.log(id, 'finish')
                }, 1000)
            })
        }

        async function taskdo(tasklist){
            let p1 = request(1)
            let p2 = request(2)
            let p3 = request(3)

            await p1
            await p2
            await p3
            // 返回的结果不一定是按顺序的

        }
        
    ```
## BOM
    浏览器对象模型
### window
* BOM 的核心是 window 对象，表示浏览器的实例。window 对象在浏览器中有两重身份，一个是
ECMAScript 中的 Global 对象，另一个就是浏览器窗口的 JavaScript 接口。这意味着网页中定义的所有
对象、变量和函数都以 window 作为其 Global 对象，都可以访问其上定义的 parseInt()等全局方法,通过 var 声明的所有全局变量和函
数都会变成 window 对象的属性和方法
* screenLeft 和 screenTop 属性，用于表示窗口相对于屏幕左侧和顶部的位置 ，返回值的单位是 CSS 像素
* 像素比: CSS 像素是 Web 开发中使用的统一像素单位
    * 不同像素密度的屏幕下就会有不同的
缩放系数，以便把物理像素（屏幕实际的分辨率）转换为 CSS 像素（浏览器报告的虚拟分辨率）
    * 这个物理像素与 CSS 像素之间的转换比率由 window.devicePixelRatio 属性提供
    * window.devicePixelRatio 实际上与每英寸像素数（DPI，dots per inch）是对应的。DPI 表示单
位像素密度，而 window.devicePixelRatio 表示物理像素与逻辑像素之间的缩放系数。
* 窗口大小：window.
    * innerWidth、innerHeight、outerWidth 和 outerHeight。outerWidth 和 outerHeight 返回浏
览器窗口自身的大小（不管是在最外层 window 上使用，还是在窗格<frame>中使用）。innerWidth
和 innerHeight 返回浏览器窗口中页面视口的大小（不包含浏览器边框和工具栏）。
    * document.documentElement.clientWidth 和 document.documentElement.clientHeight
返回页面视口的宽度和高度
* 视口位置: 浏览器窗口尺寸通常无法满足完整显示整个页面，为此用户可以通过滚动在有限的视口中查看文
档
    * 度量文档相对于视口滚动距离的属性有两对，返回相等的值：window.pageXoffset/window. 
scrollX 和 window.pageYoffset/window.scrollY,
    * 可以使用 scroll()、scrollTo()和 scrollBy()方法滚动页面,可以通过 behavior 属性
告诉浏览器是否平滑滚动
        ```
            // 相对于当前视口向下滚动 100 像素
            window.scrollBy(0, 100); 
            // 相对于当前视口向右滚动 40 像素
            window.scrollBy(40, 0);

            // 正常滚动 
            window.scrollTo({ 
                left: 100, 
                top: 100, 
                behavior: 'auto' 
            }); 
            // 平滑滚动
            window.scrollTo({ 
                left: 100, 
                top: 100, 
                behavior: 'smooth' 
            });
        ```
* 导航与打开新窗口
    * 如果返回null说明open被禁用
    * window.open()方法可以用于导航到指定 URL，也可以用于打开新浏览器窗口
        ``` 
        // 第二个参数是一个已经存在的窗口或窗格（frame）的名字
        // 第三个参数是 特性字符串是一个逗号分隔的设置字符串 例如 "height=400,width=400,top=10,left=10,resizable=yes"
        
        let wroxWin = window.open(要加载的 URL,目标窗口,特性字符串, 表示新窗口在浏览器历史记录中是否替代当前加载页面的布尔值)

        // window.open()方法返回一个对新建窗口的引用。这个对象与普通 window 对象没有区别，只是为控制新窗口提供了方便
        // 缩放
        wroxWin.resizeTo(500, 500); 
        // 移动
        wroxWin.moveTo(100, 100);
        // 关闭
        wroxWin.close();
        // 把 opener 设置为 null 表示新打开的标签页不需要与打开它的标签页通信，因此可以在独立进程中运行
        wroxWin.opener = null;
        ```
* 定时器

JavaScript 在浏览器中是单线程执行的,但允许使用定时器指定在某个时间之后或每隔一段时间就
执行相应的代码,setTimeout()用于指定在一定时间后执行某些代码，而 setInterval()用于指定
每隔一段时间执行某些代码
* 为了调度不同代码的执行，JavaScript 维护了一个任务队列。其中的任务会按照添
加到队列的先后顺序执行。setTimeout()的第二个参数只是告诉 JavaScript 引擎在指定的毫秒数过后
把任务添加到这个队列。如果队列是空的，则会立即执行该代码。如果队列不是空的，则代码必须等待
前面的任务执行完才能执行。
* 调用 setTimeout()时，会返回一个表示该超时排期的数值 ID。这个超时 ID 是被排期执行代码的
唯一标识符，可用于取消该任务。要取消等待中的排期任务，可以调用 clearTimeout()方法并传入超
时 ID

* 系统对话框
    * 使用 alert()、confirm()和 prompt()方法，可以让浏览器调用系统对话框向用户显示消息
    * 详情看文档不过多赘述
### location

* location 是最有用的 BOM 对象之一，提供了当前窗口中加载文档的信息，以及通常的导航功能。
这个对象独特的地方在于，它既是 window 的属性，也是 document 的属性。也就是说，
window.location 和 document.location 指向同一个对象。location 对象不仅保存着当前加载文
档的信息，也保存着把 URL 解析为离散片段后能够通过属性访问的信息
    * 具体参考文档
* 查询字符串：location.search 返回了从问号开始直到 URL 末尾的所有内容
* URLSearchParams
    * URLSearchParams 提供了一组标准 API 方法，通过它们可以检查和修改查询字符串。给
URLSearchParams 构造函数传入一个查询字符串，就可以创建一个实例。这个实例上暴露了 get()、
set()和 delete()等方法，可以对查询字符串执行相应操作
* 操作地址
    ```
    location.assign("http://www.wrox.com");
    window.location = "http://www.wrox.com"; 
    location.href = "http://www.wrox.com";
    location.reload(); // 重新加载，可能是从缓存加载
    location.reload(true); // 重新加载，从服务器加载
    // 假设当前 URL 为 http://www.wrox.com/WileyCDA/ 
    // 把 URL 修改为 http://www.wrox.com/WileyCDA/#section1 
    location.hash = "#section1"; 
    // 把 URL 修改为 http://www.wrox.com/WileyCDA/?q=javascript 
    location.search = "?q=javascript"; 
    // 把 URL 修改为 http://www.somewhere.com/WileyCDA/ 
    location.hostname = "www.somewhere.com"; 
    // 把 URL 修改为 http://www.somewhere.com/mydir/ 
    location.pathname = "mydir"; 
    // 把 URL 修改为 http://www.somewhere.com:8080/WileyCDA/ 
    location.port = 8080; 
    除了 hash 之外，只要修改 location 的一个属性，就会导致页面重新加载新 URL。
    ```
### navigator

客户端标识

* navigator 对象实现了 NavigatorID 、 NavigatorLanguage 、 NavigatorOnLine 、
NavigatorContentUtils 、 NavigatorStorage 、 NavigatorStorageUtils 、 NavigatorConcurrentHardware、NavigatorPlugins 和 NavigatorUserMedia 接口定义的属性和方法
* 检测插件: window.navigator.plugins
### screen

window 的另一个属性 screen 对象，是为数不多的几个在编程中很少用的 JavaScript 对象。这个对
象中保存的纯粹是客户端能力信息，也就是浏览器窗口外面的客户端显示器的信息，比如像素宽度和像
素高度。每个浏览器都会在 screen 对象上暴露不同的属性

### history 对象
history 对象表示当前窗口首次使用以来用户的导航历史记录。因为 history 是 window 的属性，
所以每个 window 都有自己的 history 对象。出于安全考虑，这个对象不会暴露用户访问过的 URL，
但可以通过它在不知道实际 URL 的情况下前进和后退

```
// 后退一页
history.go(-1); 
// 前进一页
history.go(1); 
// 前进两页
history.go(2);
// 后退一页
history.back(); 
// 前进一页
history.forward();

```
## DOM

    文档对象模型（DOM，Document Object Model）是 HTML 和 XML 文档的编程接口。DOM 表示由多层节点构成的文档，通过它开发者可以添加、删除和修改页面的各个部分

* 层级节点
    * Node类型
    * document 对象
        *   向网页输出流中写入内容。这个能力对应 4 个方法：write()、
writeln()、open()和 close()。其中，write()和 writeln()方法都接收一个字符串参数，可以将
这个字符串写入网页中。write()简单地写入文本，而 writeln()还会在字符串末尾追加一个换行符
（\n）
    * Element 类型
        * 对于dom元素的操作 包括不限于 设置属性 获取属性 修改内容
    * Text 类型
    * Comment 类型
    * CDATASection 类型
    * DocumentType 类型
    * DocumentFragment 类型
    * Attr 类型
* DOM编程
* MutationObserver

#### MutationObserver

    MutationObserver是 JavaScript 中的一个接口，用于监视 DOM 树的变化。它可以观察到 DOM 节点的添加、删除、属性变化等操作，并在这些变化发生时触发回调函数


* MutationObserver 接口是出于性能考虑而设计的，其核心是异步回调与记录队列模型。为了在
大量变化事件发生时不影响性能，每次变化的信息（由观察者实例决定）会保存在 MutationRecord
实例中，然后添加到记录队列。这个队列对每个 MutationObserver 实例都是唯一的，是所有 DOM
变化事件的有序列表
* 使用
    * 可以在observe方法中传递一个配置对象来指定要观察的变化类型：
        * childList：观察目标节点的子节点的添加、删除。
        * attributes：观察目标节点的属性变化。
        * characterData：观察目标节点的文本内容变化。
        * subtree：如果设置为true，则观察目标节点及其所有后代节点。
    ```
       const observer = new MutationObserver((mutationsList, observer) => {
            for (const mutation of mutationsList) {
                if (mutation.type === 'childList') {
                    console.log('Child nodes changed.');
                } else if (mutation.type === 'attributes') {
                    console.log(`Attribute ${mutation.attributeName} changed.`);
                }
            }
        });

        const targetNode = document.getElementById('some-element');
        observer.observe(targetNode, { attributes: true, childList: true, subtree: true });

    ```
* 应用 : 当观察的节点发生变化的时候,自动想要
    * 自动保存
    * 主题切换检测
    * 页面结构变化检测
    * 实时表单验证


* 小结  
文档对象模型（DOM，Document Object Model）是语言中立的 HTML 和 XML 文档的 API。DOM Level 1 将 HTML 和 XML 文档定义为一个节点的多层级结构，并暴露出 JavaScript 接口以操作文档的底
层结构和外观。  
DOM 由一系列节点类型构成，主要包括以下几种。  
* Node 是基准节点类型，是文档一个部分的抽象表示，所有其他类型都继承 Node。
* Document 类型表示整个文档，对应树形结构的根节点。在 JavaScript 中，document 对象是
Document 的实例，拥有查询和获取节点的很多方法。
* Element 节点表示文档中所有 HTML 或 XML 元素，可以用来操作它们的内容和属性。
* 其他节点类型分别表示文本内容、注释、文档类型、CDATA 区块和文档片段。  
DOM 编程在多数情况下没什么问题，在涉及<script>和<style>元素时会有一点兼容性问题。因
为这些元素分别包含脚本和样式信息，所以浏览器会将它们与其他元素区别对待。  
要理解 DOM，最关键的一点是知道影响其性能的问题所在。DOM 操作在 JavaScript 代码中是代价
比较高的，NodeList 对象尤其需要注意。NodeList 对象是“实时更新”的，这意味着每次访问它都
会执行一次新的查询。考虑到这些问题，实践中要尽量减少 DOM 操作的数量。  
MutationObserver 是为代替性能不好的 MutationEvent 而问世的。使用它可以有效精准地监控
DOM 变化，而且 API 也相对简单。  


## 事件

JavaScript 与 HTML 的交互是通过事件实现的，事件代表文档或浏览器窗口中某个有意义的时刻。
可以使用仅在事件发生时执行的监听器（也叫处理程序）订阅事件。在传统软件工程领域，这个模型叫
“观察者模式”，其能够做到页面行为（在 JavaScript 中定义）与页面展示（在 HTML 和 CSS 中定义）的分离。

### 事件流

事件流描述了页面接收事件的顺序: 当你点击一个按钮时，实际上不光点击了这个按钮，还点击了它的容器以及整个页面

* 事件冒泡和事件捕获
    * 事件冒泡: 当点击一个按钮 事件会依次按照  Button > div > html > document 传递下去 在经过的每个节点上依次触发，直至到达 document 对象。
    * 捕获与冒泡方向相反 基本不用
* DOM 事件流
    * DOM2 Events 规范规定事件流分为 3 个阶段：事件捕获、到达目标和事件冒泡
* 事件处理程序（或事件监听器）
    * DOM0： 单击（click）、加载（load）、鼠标悬停（mouseover） 等 ： click 事件的处理程序叫作 onclick，而 load 事件的处理程序叫作 onload
    * DOM1： addEventListener(事件名、事件处理函数，布尔值)和 removeEventListener()
        * 布尔值 true 表示在捕获阶段调用事件处理程序，false（默认值）表示在冒泡阶段调用事件处理程序
### 事件对象 event
* event 对象只在事件处理程序执行期间存在，一旦执行完毕，就会被销毁
* 在 DOM 中发生事件时，所有相关信息都会被收集并存储在一个名为 event 的对象中。这个对象包
含了一些基本信息，比如导致事件的元素、发生的事件类型，以及可能与特定事件相关的任何其他数据。
例如，鼠标操作导致的事件会生成鼠标位置信息，而键盘操作导致的事件会生成与被按下的键有关的信
息
    * DOM 事件对象
        ```
        btn.onclick = function(event) { 
            console.log(event.type); // "click" 
        };
        ```
        * event.preventDefault()方法用于阻止特定事件的默认动作。比如，链接的默认行为就是在被单击时导航到 href 属性指定的 URL。如果想阻止这个导航行为，可以在 onclick 事件处理程序中取消
### 事件类型

* DOM3 Events 定义了如下事件类型。
    * 用户界面事件（UIEvent）：涉及与 BOM 交互的通用浏览器事件。
        * DOMActivate：元素被用户通过鼠标或键盘操作激活时触发（比 click 或 keydown 更通用）。这个事件在 DOM3 Events 中已经废弃。因为浏览器实现之间存在差异，所以不要使用它。
        * load：在 window 上当页面加载完成后触发，在窗套（<frameset>）上当所有窗格（<frame>）
        都加载完成后触发，在<img>元素上当图片加载完成后触发，在<object>元素上当相应对象加
        载完成后触发。
        * unload：在 window 上当页面完全卸载后触发，在窗套上当所有窗格都卸载完成后触发，在
        <object>元素上当相应对象卸载完成后触发。
        * abort：在<object>元素上当相应对象加载完成前被用户提前终止下载时触发。
        * error：在 window 上当 JavaScript 报错时触发，在<img>元素上当无法加载指定图片时触发，
        在<object>元素上当无法加载相应对象时触发，在窗套上当一个或多个窗格无法完成加载时
        触发。
        * select：在文本框（<input>或 textarea）上当用户选择了一个或多个字符时触发。
        * resize：在 window 或窗格上当窗口或窗格被缩放时触发。
        * scroll：当用户滚动包含滚动条的元素时在元素上触发。<body>元素包含已加载页面的滚动条。
    * 焦点事件（FocusEvent）：在元素获得和失去焦点时触发。
        * blur：当元素失去焦点时触发。这个事件不冒泡，所有浏览器都支持。
        * DOMFocusIn：当元素获得焦点时触发。这个事件是 focus 的冒泡版。Opera 是唯一支持这个事件的主流浏览器。DOM3 Events 废弃了 DOMFocusIn，推荐 focusin。
        * DOMFocusOut：当元素失去焦点时触发。这个事件是 blur 的通用版。Opera 是唯一支持这个事件的主流浏览器。DOM3 Events 废弃了 DOMFocusOut，推荐 focusout。
        * focus：当元素获得焦点时触发。这个事件不冒泡，所有浏览器都支持。
        * focusin：当元素获得焦点时触发。这个事件是 focus 的冒泡版。
        * focusout：当元素失去焦点时触发。这个事件是 blur 的通用版。 

        焦点事件中的两个主要事件是 focus 和 blur，这两个事件在 JavaScript 早期就得到了浏览器支持。它们最大的问题是不冒泡。这导致 IE后来又增加了 focusin 和 focusout，Opera又增加了 DOMFocusIn 和 DOMFocusOut。IE 新增的这两个事件已经被 DOM3 Events 标准化。  

        当焦点从页面中的一个元素移到另一个元素上时，会依次发生如下事件。  
            1. focuscout 在失去焦点的元素上触发。
            2. focusin 在获得焦点的元素上触发。
            3. blur 在失去焦点的元素上触发。
            4. DOMFocusOut 在失去焦点的元素上触发。
            5. focus 在获得焦点的元素上触发。
            6. DOMFocusIn 在获得焦点的元素上触发。  
        其中，blur、DOMFocusOut 和 focusout 的事件目标是失去焦点的元素，而 focus、DOMFocusIn和 focusin 的事件目标是获得焦点的元素。
    * 鼠标事件（MouseEvent）：使用鼠标在页面上执行某些操作时触发。
    * 滚轮事件（WheelEvent）：使用鼠标滚轮（或类似设备）时触发。
    * 输入事件（InputEvent）：向文档中输入文本时触发。
    * 键盘事件（KeyboardEvent）：使用键盘在页面上执行某些操作时触发。
    * 合成事件（CompositionEvent）：在使用某种 IME（Input Method Editor，输入法编辑器）输入字符时触发。
    * HTML5 事件
        * contextmenu 事件 :上下文菜单
        * beforeunload 事件:给开发者提供阻止页面被卸载的机会
        * DOMContentLoaded 事件: window 的 load 事件会在页面完全加载后触发，因为要等待很多外部资源加载完成，所以会花费较长时间。而 DOMContentLoaded 事件会在 DOM 树构建完成后立即触发，而不用等待图片、JavaScript文件、CSS 文件或其他资源加载完成.相对于 load 事件，DOMContentLoaded 可以让开发者在外部资
源下载的同时就能指定事件处理程序，从而让用户能够更快地与页面交互。
        * readystatechange 事件  文档或元素加载状态的信息
        * pageshow 与 pagehide 事件
        * hashchange 事件:用于在 URL 散列值（URL 最后#后面的部分）发生变化时通知开发者
    * 设备事件
### 事件委托
过多事件处理程序”的解决方案是使用事件委托。事件委托利用事件冒泡，可以只使用一个事件
处理程序来管理一种类型的事件。例如，click 事件冒泡到 document。这意味着可以为整个页面指定
一个 onclick 事件处理程序，而不用为每个可点击元素分别指定事件处理程序
```
<ul id="myLinks"> 
 <li id="goSomewhere">Go somewhere</li> 
 <li id="doSomething">Do something</li> 
 <li id="sayHi">Say hi</li> 
</ul> 
这里的 HTML 包含 3 个列表项，在被点击时应该执行某个操作。对此，通常的做法是像这样指定 3
个事件处理程序：
let item1 = document.getElementById("goSomewhere"); 
let item2 = document.getElementById("doSomething"); 
let item3 = document.getElementById("sayHi"); 
item1.addEventListener("click", (event) => { 
    location.href = "http:// www.wrox.com"; 
}); 
item2.addEventListener("click", (event) => { 
    document.title = "I changed the document's title"; 
}); 
item3.addEventListener("click", (event) => { 
    console.log("hi"); 
});

改进后

let list = document.getElementById("myLinks"); 
list.addEventListener("click", (event) => { 
    let target = event.target; 
    switch(target.id) { 
        case "doSomething": 
            document.title = "I changed the document's title"; 
            break; 
        case "goSomewhere": 
            location.href = "http:// www.wrox.com"; 
            break; 
        case "sayHi": 
            console.log("hi"); 
            break; 
    } 
});
```
### 模拟事件
    * DOM 事件模拟
        任何时候，都可以使用 document.createEvent()方法创建一个 event 对象。这个方法接收一个参数，此参数是一个表示要创建事件类型的字符串。在 DOM2 中，所有这些字符串都是英文复数形式，但在 DOM3 中，又把它们改成了英文单数形式。可用的字符串值是以下值之一。
            * "UIEvents"（DOM3 中是"UIEvent"）：通用用户界面事件（鼠标事件和键盘事件都继承自这
            个事件）。
            * "MouseEvents"（DOM3 中是"MouseEvent"）：通用鼠标事件。
            * "HTMLEvents"（DOM3 中没有）：通用 HTML 事件（HTML 事件已经分散到了其他事件大类中）。
        注意，键盘事件不是在 DOM2 Events 中规定的，而是后来在 DOM3 Events 中增加的。  
        创建 event 对象之后，需要使用事件相关的信息来初始化。每种类型的 event 对象都有特定的方法，可以使用相应数据来完成初始化。方法的名字并不相同，这取决于调用 createEvent()时传入的参数。  
        事件模拟的最后一步是触发事件。为此要使用 dispatchEvent()方法，这个方法存在于所有支持事件的 DOM 节点之上。dispatchEvent()方法接收一个参数，即表示要触发事件的 event 对象。调用 dispatchEvent()方法之后，事件就“转正”了，接着便冒泡并触发事件处理程序执行
        * 模拟鼠标
            ```
                let btn = document.getElementById("myBtn"); 
                // 创建 event 对象
                let event = document.createEvent("MouseEvents"); 
                // 初始化 event 对象
                event.initMouseEvent("click", true, true, document.defaultView, 
                0, 0, 0, 0, 0, false, false, false, false, 0, null); 
                // 触发事件
                btn.dispatchEvent(event);
            ```
        * 模拟键盘 "KeyboardEvent"
        * 模拟其他事件
        * 自定义 DOM 事件
            DOM3 增加了自定义事件的类型。自定义事件不会触发原生 DOM 事件，但可以让开发者定义自己的事件。
### 小结  

事件是 JavaScript 与网页结合的主要方式。最常见的事件是在 DOM3 Events 规范或 HTML5 中定义
的。虽然基本的事件都有规范定义，但很多浏览器在规范之外实现了自己专有的事件，以方便开发者更
好地满足用户交互需求，其中一些专有事件直接与特殊的设备相关。  
围绕着使用事件，需要考虑内存与性能问题。例如：
*  最好限制一个页面中事件处理程序的数量，因为它们会占用过多内存，导致页面响应缓慢；
*  利用事件冒泡，事件委托可以解决限制事件处理程序数量的问题；
*  最好在页面卸载之前删除所有事件处理程序。  
使用 JavaScript 也可以在浏览器中模拟事件。DOM2 Events 和 DOM3 Events 规范提供了模拟方法，
可以模拟所有原生 DOM 事件。键盘事件一定程度上也是可以模拟的，有时候需要组合其他技术。IE8
及更早版本也支持事件模拟，只是接口与 DOM 方式不同。  
事件是 JavaScript 中最重要的主题之一，理解事件的原理及其对性能的影响非常重要。

## 动画与 Canvas 图形

### requestAnimationFrame

requestAnimationFrame是浏览器提供的一个用于在浏览器下一次重绘之前执行特定函数的方法  
这个方法会告诉浏览器要执行动画了，于是浏览器可以通过最优方式确定重绘的时序, 知道何时绘制下一帧是创造平滑动画的关键

* 一般计算机显示器的屏幕刷新率都是 60Hz，基本上意味着每秒需要重绘 60 次。大多数浏览器会限制重绘频率，使其不超出屏幕的刷新率，这是因为超过刷新率，用户也感知不到
* 因此，实现平滑动画最佳的重绘间隔为 1000 毫秒/60，大约 17 毫秒。以这个速度重绘可以实现最平滑的动画，因为这已经是浏览器的极限了。如果同时运行多个动画，可能需要加以限流，以免 17 毫秒
的重绘间隔过快，导致动画过早运行完。
* requestAnimationFrame()方法接收一个参数，此参数是一个要在重绘屏幕前调用的函数, 这个函数就是修改 DOM 样式以反映下一次重绘有什么变化的地方
    * 为了实现动画循环，可以把多个 requestAnimationFrame()调用串联起来，就像以前使用 setTimeout()时一样：
        ```
            function updateProgress() { 
                var div = document.getElementById("status"); 
                div.style.width = (parseInt(div.style.width, 10) + 5) + "%"; 
                if (div.style.left != "100%") { 
                    requestAnimationFrame(updateProgress); 
                } 
            } 
            requestAnimationFrame(updateProgress);
            // 停止
            let requestID = window.requestAnimationFrame(() => { 
                console.log('Repaint!'); 
            }); 
            window.cancelAnimationFrame(requestID);
     ```
    * 因为 requestAnimationFrame()只会调用一次传入的函数，所以每次更新用户界面时需要再手动调用它一次。同样，也需要控制动画何时停止。结果就会得到非常平滑的动画。目前为止，requestAnimationFrame()已经解决了浏览器不知道 JavaScript 动画何时开始的问题，以及最佳间隔是多少的问题，但是，不知道自己的代码何时实际执行的问题呢？这个方案同样也给出了解决方法。
    * 传给 requestAnimationFrame()的函数实际上可以接收一个参数，此参数是一个 DOMHighResTimeStamp 的实例（比如 performance.now()返回的值），表示下次重绘的时间。这一点非常重要：requestAnimationFrame()实际上把重绘任务安排在了未来一个已知的时间点上，而且通过这个参数告诉了开发者。基于这个参数，就可以更好地决定如何调优动画了。

### canvas 2D画布

* 创建<canvas>元素时至少要设置其 width 和 height 属性
* 要在画布上绘制图形，首先要取得绘图上下文。使用 getContext()方法可以获取对绘图上下文的
引用。对于平面图形，需要给这个方法传入参数"2d"，表示要获取 2D 上下文对象
* 可以使用 toDataURL()方法导出<canvas>元素上的图像: 如果画布中的图像是其他域绘制过来的，toDataURL()方法就会抛出错误
```
// <canvas id="drawing" width="200" height="200">A drawing of something.</canvas>
let drawing = document.getElementById("drawing"); 
// 确保浏览器支持<canvas> 
if (drawing.getContext) { 
    // 取得图像的数据 URI 
    let imgURI = drawing.toDataURL("image/png"); 
    // 显示图片
    let image = document.createElement("img"); 
    image.src = imgURI; 
    document.body.appendChild(image); 
}

```
#### 绘制
* 2D 上下文的坐标原点(0, 0)在<canvas>元素的左上角。所有坐标值都相对于该点计算，因此 x 坐标向右增长，y 坐标向下增长。默认情况下，width 和 height 表示两个方向上像素的最大值。
* 填充和描边
* 绘制矩形
* 绘制路径
* 绘制文本
* 变换
* 绘制图像
* 阴影
* 渐变
* 图案
* 图像数据
* 合成
### WebGL  3D画布

## javascript API
* Atomics与sharedArrayBuffer
* 跨上下文消息 XDM （cross-document messaging）
    * 跨上下文消息用于窗口之间通信或工作线程之间通信
    * postMessage(消息,表示目标接收源的字符串,可选的可传输对象的数组)
        ```
            let iframeWindow = document.getElementById("myframe").contentWindow; 
            iframeWindow.postMessage("A secret", "http://www.wrox.com");
            // 最后一行代码尝试向内嵌窗格中发送一条消息，而且指定了源必须是"http://www.wrox.com"。如果源匹配，那么消息将会交付到内嵌窗格；否则，postMessage()什么也不做。这个限制可以保护信息不会因地址改变而泄露。如果不想限制接收目标，则可以给 postMessage()的第二个参数传"*"，但不推荐这么做
        ```
    * 接收到 XDM 消息后，window 对象上会触发 message 事件。这个事件是异步触发的，因此从消息发出到接收到消息（接收窗口触发 message 事件）可能有延迟。传给 onmessage 事件处理程序的 event对象包含以下 3 方面重要信息。
        * data：作为第一个参数传递给 postMessage()的字符串数据。
        * origin：发送消息的文档源，例如"http://www.wrox.com"。
        * source：发送消息的文档中 window 对象的代理。这个代理对象主要用于在发送上一条消息的窗口中执行 postMessage()方法。如果发送窗口有相同的源，那么这个对象应该就是 window对象。
        ```
            window.addEventListener("message", (event) => { 
            // 确保来自预期发送者
            if (event.origin == "http://www.wrox.com") { 
            // 对数据进行一些处理
            processMessage(event.data); 
            // 可选：向来源窗口发送一条消息
            event.source.postMessage("Received!", "http://p2p.wrox.com"); 
            } 
            });
        ```
* Encoding API
* File API 与 Blob API
    * FileReader 类型: FileReader类型表示一种异步文件读取机制。可以把FileReader 想象成类似于XMLHttpRequest，只不过是用于从文件系统读取文件，而不是从服务器读取数据
        * readAsText(file, encoding)：从文件中读取纯文本内容并保存在 result 属性中。第二个参数表示编码，是可选的。
        * readAsDataURL(file)：读取文件并将内容的数据 URI 保存在 result 属性中。
        * readAsBinaryString(file)：读取文件并将每个字符的二进制数据保存在 result 属性中。
        * readAsArrayBuffer(file)：读取文件并将文件内容以 ArrayBuffer 形式保存在 result 属性。  
        这些读取数据的方法为处理文件数据提供了极大的灵活性。例如，为了向用户显示图片，可以将图片读取为数据 URI，而为了解析文件内容，可以将文件读取为文本  
        * 读取的状态 progress(进行中)、error(错误) 和 load(加载中)、 progress 事件每 50 毫秒就会触发一次
        * progress 事件用于跟踪和显示读取文件的进度，而 error 事件用于监控错误
        * 如果想提前结束文件读取，则可以在过程中调用 abort()方法，从而触发 abort 事件
    * Blob 与部分读取：blob 表示二进制大对象（binary larget object），是 JavaScript 对不可修改二进制数据的封装类型  
        某些情况下，可能需要读取部分文件而不是整个文件。为此，File 对象提供了一个名为 slice()的方法。slice()方法接收两个参数：起始字节和要读取的字节数。这个方法返回一个 Blob 的实例，而 Blob 实际上是 File 的超类。  
        * 包含字符串的数组、ArrayBuffers、ArrayBufferViews，甚至其他 Blob 都可以用来创建 blob。Blob构造函数可以接收一个 options 参数，并在其中指定 MIME 类型
        * Blob 对象有一个 size 属性和一个 type 属性，还有一个 slice()方法用于进一步切分数据。另外也可以使用 FileReader 从 Blob 中读取数据
    * 对象 URL 与 Blob  
        对象 URL 有时候也称作 Blob URL，是指引用存储在 File 或 Blob 中数据的 URL。对象 URL 的优点是不用把文件内容读取到 JavaScript 也可以使用文件。只要在适当位置提供对象 URL 即可。要创建对象 URL，可以使用 window.URL.createObjectURL()方法并传入 File 或 Blob 对象。这个函数返回的值是一个指向内存中地址的字符串。因为这个字符串是 URL，所以可以在 DOM 中直接使用  
    * 读取拖放文件
        * 在页面上创建放置目标后，可以从桌面上把文件拖动并放到放置目标。这样会像拖放图片或链接一样触发 drop 事件。 被放置的文件可以通过事件的 event.dataTransfer.files 属性读到，这个属性保存着一组 File 对象，就像文本输入字段一样
        ```
            let droptarget = document.getElementById("droptarget"); 
            function handleEvent(event) { 
                let info = "", output = document.getElementById("output"), files, i, len; 
                event.preventDefault(); 
                if (event.type == "drop") { 
                    files = event.dataTransfer.files; 
                    i = 0; 
                    len = files.length; 
                    while (i < len) { 
                        info += `${files[i].name} (${files[i].type}, ${files[i].size} bytes)<br>`; 
                        i++; 
                    } 
                    output.innerHTML = info; 
                } 
            } 
            droptarget.addEventListener("dragenter", handleEvent); 
            droptarget.addEventListener("dragover", handleEvent); 
            droptarget.addEventListener("drop", handleEvent);
        ```
* 媒体元素
* 原生拖放
    * 拖放事件几乎可以让开发者控制拖放操作的方方面面。关键的部分是确定每个事件是在哪里触发的。有的事件在被拖放元素上触发，有的事件则在放置目标上触发。在某个元素被拖动时，会（按顺序）触发以下事件：
        * dragstart
        * drag
        * dragend
    * 在把元素拖动到一个有效的放置目标上时，会依次触发以下事件：
        * dragenter
        * dragover
        * dragleave 或 drop
    * 可拖动能力 draggable="false"
* Notifications API
* Page Visibility API
    这个 API 本身非常简单，由 3 部分构成。  
    * document.visibilityState 值，表示下面 4 种状态之一。
     页面在后台标签页或浏览器中最小化了。
     页面在前台标签页中。
     实际页面隐藏了，但对页面的预览是可见的（例如在 Windows 7 上，用户鼠标移到任务栏图标
    上会显示网页预览）。
     页面在屏外预渲染。
    * visibilitychange 事件，该事件会在文档从隐藏变可见（或反之）时触发。
    * document.hidden 布尔值，表示页面是否隐藏。这可能意味着页面在后台标签页或浏览器中被最小
    化了。这个值是为了向后兼容才继续被浏览器支持的，应该优先使用 document.visibilityState
    检测页面可见性。  
    要想在页面从可见变为隐藏或从隐藏变为可见时得到通知，需要监听 visibilitychange 事件。  
    document.visibilityState 的值是以下三个字符串之一：  
    * "hidden"
    * "visible"
    * "prerender
* Streams API  
    Streams API 是为了解决一个简单但又基础的问题而生的：Web 应用如何消费有序的小信息块而不是大块信息？这种能力主要有两种应用场景。  
    * 大块数据可能不会一次性都可用。网络请求的响应就是一个典型的例子。网络负载是以连续信
    息包形式交付的，而流式处理可以让应用在数据一到达就能使用，而不必等到所有数据都加载
    完毕。
    * 大块数据可能需要分小部分处理。视频处理、数据压缩、图像编码和 JSON 解析都是可以分成小
    部分进行处理，而不必等到所有数据都在内存中时再处理的例子。
    * Stream API 定义了三种流。
        * 可读流：可以通过某个公共接口读取数据块的流。数据在内部从底层源进入流，然后由消费者（consumer）进行处理。
            * 可读流是对底层数据源的封装。底层数据源可以将数据填充到流中，允许消费者通过流的公共接口读取数据。
        * 可写流：可以通过某个公共接口写入数据块的流。生产者（producer）将数据写入流，数据在内部传入底层数据槽（sink）。
        * 转换流：由两种流组成，可写流用于接收数据（可写端），可读流用于输出数据（可读端）。这两个流之间是转换程序（transformer），可以根据需要检查和修改流内容。
* 计时 API  
    Performance 接口通过 JavaScript API 暴露了浏览器内部的度量指标，允许开发者直接访问这些信息并基于这些信息实现自己想要的功能。这个接口暴露在window.performance 对象上  
    Performance 接口由多个 API 构成：  
        * High Resolution Time API 
        * Performance Timeline API 
        * Navigation Timing API 
        * User Timing API 
        * Resource Timing API 
        * Paint Timing API
* Web 组件
    * 模板html  <template id="bar">my dom</template>
    * 模板脚本 
    * 影子 DOM ：通过它可以将一个完整的 DOM 树作为节点添加到父 DOM 树。这样可以实现 DOM 封装，意味着 CSS 样式和 CSS 选择符可以限制在影子 DOM 子树而不是整个顶级 DOM 树中。
        * 影子 DOM 是通过 attachShadow()方法创建并添加给有效 HTML 元素的。容纳影子 DOM 的元素被称为影子宿主（shadow host）。影子 DOM 的根节点被称为影子根（shadow root）。
        * attachShadow()方法需要一个shadowRootInit 对象，返回影子DOM的实例。shadowRootInit对象必须包含一个 mode 属性，值为"open"或"closed"。对"open"影子 DOM的引用可以通过 shadowRoot属性在 HTML 元素上获得，而对"closed"影子 DOM 的引用无法这样获取
            ```
                document.body.innerHTML = ` 
                <div id="foo"></div> 
                <div id="bar"></div> 
                `; 
                const foo = document.querySelector('#foo'); 
                const bar = document.querySelector('#bar'); 
                const openShadowDOM = foo.attachShadow({ mode: 'open' }); 
                const closedShadowDOM = bar.attachShadow({ mode: 'closed' }); 
                console.log(openShadowDOM); // #shadow-root (open)
                console.log(closedShadowDOM); // #shadow-root (closed)
                console.log(foo.shadowRoot); // #shadow-root (open) 
                console.log(bar.shadowRoot); // null
            ```
    *  自定义组件
        ```
            class FooElement extends HTMLElement { 
                constructor() { 
                    super(); 
                    console.log('x-foo') 
                    // this 引用 Web 组件节点
                    this.attachShadow({ mode: 'open' }); 
                    this.shadowRoot.innerHTML = ` 
                        <p>I'm inside a custom element!</p> 
                    `;
                } 
            } 
                customElements.define('x-foo', FooElement); 
                document.body.innerHTML = ` 
                <x-foo></x-foo> 
                <x-foo></x-foo> 
                <x-foo></x-foo>      
        ```
* Web Cryptography API

## 错误与调试
* throw new refrenceError(‘123’)
* JavaScript 调试器 debugger
## 网络请求
* XHR
    ``` 
    let xhr =  new XmlHttpRequest()
    xhr.onreadystatechange(() =>{

    })
    xhr.open('get', url)
    xhr.send()

    ```
* fetch
    * fetch(url).then(res => res.text()).then(data =>{    })
    * 发送 JSON 数据
        ```
        可以像下面这样发送简单 JSON 字符串：
        let payload = JSON.stringify({ 
            foo: 'bar' 
        }); 
        let jsonHeaders = new Headers({ 
            'Content-Type': 'application/json' 
        }); 
        fetch('/send-me-json', { 
            method: 'POST', // 发送请求体时必须使用一种 HTTP 方法
            body: payload, 
            headers: jsonHeaders 
        });
        ```
    * 在请求体中发送参数
        ```
            let payload = 'foo=bar&baz=qux'; 
            let paramHeaders = new Headers({ 
                'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8' 
            });
            fetch('/send-me-params', { 
                method: 'POST', // 发送请求体时必须使用一种 HTTP 方法
                body: payload, 
                headers: paramHeaders 
            });
        ```
    * 发送文件
        ```
            let imageFormData = new FormData(); 
            let imageInput = document.querySelector("input[type='file'][multiple]"); 
                for (let i = 0; i < imageInput.files.length; ++i) { 
                imageFormData.append('image', imageInput.files[i]); 
            }
            fetch('/img-upload', { 
                method: 'POST', 
                body: imageFormData 
            });
        ```
    * 加载 Blob 文件
        ```
            const imageElement = document.querySelector('img'); 
            fetch('my-image.png') 
            .then((response) => response.blob()) 
            .then((blob) => { 
                imageElement.src = URL.createObjectURL(blob); 
            });
        ```
    * 中断请求
        ```
            let abortController = new AbortController(); 
            fetch('wikipedia.zip', { signal: abortController.signal }) 
            .catch(() => console.log('aborted!'); 
            // 10 毫秒后中断请求
            setTimeout(() => abortController.abort(), 10); 
            // 已经中断
        ```
    * 流式获取数据

### 浏览器存储
* cookie
* web storage
    * localstorage
    * sessionstorage
* indexedDB

### 模块
CommonJS  
* CommonJS 规范概述了同步声明依赖的模块定义。这个规范主要用于在服务器端实现模块化代码组织，但也可用于定义在浏览器中使用的模块依赖。CommonJS 模块语法不能在浏览器中直接运行
* CommonJS 模块定义需要使用 require()指定依赖，而使用 exports 对象定义自己的公共 API。
* 无论一个模块在 require()中被引用多少次，模块永远是单例
* 模块第一次加载后会被缓存，后续加载会取得缓存的模块 模块加载顺序由依赖图决定
* 在 CommonJS 中，模块加载是模块系统执行的同步操作
* CommonJS 依赖几个全局属性如 require 和 module.exports。如果想在浏览器中使用 CommonJS模块，就需要与其非原生的模块语法之间构筑“桥梁”。模块级代码与浏览器运行时之间也需要某种“屏障”，因为没有封装的CommonJS 代码在浏览器中执行会创建全局变量。这显然与模块模式的初衷相悖。常见的解决方案是提前把模块文件打包好，把全局属性转换为原生 JavaScript 结构，将模块代码封装在函数闭包中，最终只提供一个文件。为了以正确的顺序打包模块，需要事先生成全面的依赖图。

#### ES6的  模块
* <script type="module"> 所有模块都会像<script defer>加载的脚本一样按顺序执行
* <script type="module">关联或者通过 import 语句加载的 JavaScript 文件会被认定为模块。
* 模块行为
    * 模块代码只在加载后执行。
    * 模块只能加载一次。
    * 模块是单例。
    * 模块可以定义公共接口，其他模块可以基于这个公共接口观察和交互。
    * 模块可以请求加载其他模块。
    * 支持循环依赖。
    * ES6 模块默认在严格模式下执行。
    * ES6 模块不共享全局命名空间。
    * 模块顶级 this 的值是 undefined（常规脚本中是 window）。
    * 模块中的 var 声明不会添加到 window 对象。
    * ES6 模块是异步加载和执行的。
* 模块导出
    ES6 模块支持两种导出：命名导出和默认导出
    * export 关键字用于声明一个值为命名导出。导出语句必须在模块顶级，不能嵌套在某个块中
    * export 语句与导出值的相对位置或者export 关键字在模块中出现的顺序没有限制
        ```
            const foo = 'foo'; 

            // 等同于 export default foo;  默认导出
            export { foo as default };
            export { foo };   // 允许同时存在
        ```
* 模块导入
    * import 必须出现在模块的顶级 不能嵌套在某个块中
    * import 语句被提升到模块顶部
    * 如果在浏览器中通过标识符原生加载模块，则文件必须带有.js 扩展名，不然可能无法正确解析。不过，如果是通过构建工具或第三方模块加载器打包或解析的 ES6 模块，则可能不需要包含文件扩展名
    * 导入对模块而言是只读的，实际上相当于 const 声明的变量。在使用*执行批量导入时，赋值给别名的命名导出就好像使用 Object.freeze()冻结过一样。直接修改导出的值是不可能的，但可以修改导出对象的属性。同样，也不能给导出的集合添加或删除导出的属性。要修改导出的值，必须使用有内部变量和属性访问权限的导出方法
        ```
            import { foo, bar, baz as myBaz } from './foo.js';
        ```
* 模块转移导出
    * 模块导入的值可以直接通过管道转移到导出
    * export * from './foo.js';

## 工作者线程

* 允许把主线程的工作转嫁给独立的实体，而不会改变现有的单线程模型
* JavaScript 环境实际上是运行在托管操作系统中的虚拟环境。在浏览器中每打开一个页面，就会分
配一个它自己的环境。这样，每个页面都有自己的内存、事件循环、DOM，等等。每个页面就相当于
一个沙盒，不会干扰其他页面。对于浏览器来说，同时管理多个环境是非常简单的，因为所有这些环境
都是并行执行的。
* 使用工作者线程，浏览器可以在原始页面环境之外再分配一个完全独立的二级子环境。这个子环境
不能与依赖单线程交互的 API（如 DOM）互操作，但可以与父环境并行执行代码。
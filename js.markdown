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
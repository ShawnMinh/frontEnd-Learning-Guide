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
    * >> 右移    等于/2的n次方  尽头是 0
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
    

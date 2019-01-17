---
title: JavaScript精粹选摘
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
#### 语句
- 每个\<script>标签都提供一个被编译且立即执行的编译单元。因为缺少链接器，JavaScript把他们一起抛入一个公共的全局名字空间中。
#### 假值，在判断真假的句子中返回false

- NaN
- null
- undefined
- ''
- 0
- null

#### 运算符优先级

1. 属性存取及函数调用 
    >   . []  ()
2. 一元运算符  （注意跟加减的区别）
    > delete new typeof + - ！
3. 乘法、除法、取模
    > \* / %
4. 加（连接）、减
    > \+ \-
5. 不等式运算符
    > '>=', '<=','>','<' 
6. 等式运算符（注意高出逻辑运算符优先级）
    > === !==
7. 逻辑与
    > &&
8. 逻辑或
    > ||
9. 三元运算符
    > ?:

#### 函数字面量

- 函数字面量定义了函数值，有一个可选的名字，用于递归的调用自己。

#### 对象

1. 基本性质
    - JavaScript简单类型包括数字、字符、布尔值、null、undefined，其他的所有值都是对象。
    - 数组、函数、正则表达式都是对象。
    - 对象是属性的容器，属性名可以是包含空字符串在内的**任意字符串**,属性值可以是除了undefined之外的任何值。
    - 通过原型链，允许对象继承另一对象的属性。使用它能减少对象初始化时间和内存消耗。

2. 对象字面量
    - key可用引号也可以不用，但当key的string含有不合法字符串时，必须用引号
3. 对象的检索
    - 当key不是保留字的时候可以使用obj[keyString]也可以用obj.key访问对应值。
4. 对象的原型
    - 每个对象都由隐藏属性\__proto__链接到一个原型对象，并且可以从中继承属性。
    - 所有通过对象字面量创建的对象的\__proto__都指向Object.prototype对象。
    - 原型对象只在检索时有用，在更新对象时不受影响，除非显式修改原型对象：obj.\__proto__ = balabala 或 obj.prototype = balabala
5. 反射
    - 通过反射检查数据的类型，可以通过typeof、hasOwnProperty()等方法。
6. 枚举
    - 对于对象，可以使用for...in枚举，会把包括原型里面的属性都遍历。例如：
        ```
        for(key in obj){
            if(typeof obj.key === 'object'){
                
            }
        }
        ```
    
    - 注意：for(item of items)中，item是值。
7. 删除
    - 可以通过delete关键字删除某个key-value,比如：delete obj.name 或者 delete.\__proto__.name
8. 减小全局变量污染
    - 只创建一个全局变量（对象），其他所有代码都写在这个对象里面
    - 使用闭包（推荐）

#### 函数

1. 函数对象

    - 函数也是一个对象，且其__proto__属性连接到Function.prototype，而Function.prototype的__proto__指向Object.prototype;
    - 每个函数对象在创建时随带一个prototype属性指向其原型对象。默认情况下，prototype对象里面只有一个constructor属性，指向构造函数（该函数对象）。
    - 每个函数在创建时，会有两个附加属性：函数上下文、实现函数行为的代码
2. 函数字面量：function 可选名字(可选形参){可选语句}
3. 函数调用方式：见[this相关问题](http://note.youdao.com/noteshare?id=2e611fb94dbb28e4ae1cd3ebd5ae4ef3)
4. 函数参数: 可以定义一个无参函数，但是在调用时传递一些参数，然后在函数体内通过arguments对象获取
5. 函数返回:
    - 一个函数总会返回一个值，如果没有指定则返回undefined
    - 如果函数在前面加new调用，且返回值不是一个对象，则返回该this
6. 函数执行异常
    - 可以随时在语句中使用throw关键字，后面跟一个自定义的Exception，这个Exception可以是一个简单值，也可以是一个自定义对象。
    - throw语句用来抛出一个用户自定义的异常。**当前函数的执行将被停止**（throw之后的语句将不会执行），并且控制将被传递到调用堆栈中的第一个catch块。**不管throw后面抛出的是啥，一旦执行，且调用者函数中没有catch块，程序将会报错且终止**。
    - 可以通过继承异常基类Error自定义异常类型,可以添加除了Error.prototype.message和Error.prototype.name之外的属性和方法。
7. 模块
    - 我们可以通过使用函数和闭包实现模块，模块是一个提供接口却隐藏状态与是实现的函数或对象。
    - 模块的一般形式为：**一个定义了私有变量和函数的函数，利用闭包创建可以访问私有变量和函数的特权方法，最后返回这个特权函数或者把他们保存到一个可以访问到的地方(比如挂在global对象上)**。
8. 级联
    - 不带有return语句的函数默认返回undefined
    - 对于修改某个对象的状态这样的函数，可以使其返回this而不是undefined，就可以启用级联。也即是链式调用。
9. 套用
    - 函数也是值，从而可以像使用值的方式去操纵函数。
    - 套用允许我们将函数与传递给它的参数相结合产生一个新的函数。
10. 记忆
    - 利用闭包特性，将函数的历史执行结果以（参数：函数返回值）的形式用数组或对象保存在函数内部，下次再输入某个参数时，首先判断该参数是否已经被计算过，如果是则直接返回保存的结果。
    ```
     function wrapper(){
         var result = [];
         return function(i){
             var temp = result[i];
             if(!temp){
                 // 正常执行计算
                 result[i] = 计算结果;
                 return 计算结果;
             }
             return temp；
         }
     }
     
     // 继续抽象，使缓存可以自定义，执行函数体也可以自定义(不支持递归)
     function compute(store, func){
         return function(i){
             if (typeof store[i] !== '某类型'){
                 store[i] = func(i);
                 return store[i];
             }
             return store[i];
         }
     }
     ===============================================================
     // 如果要考虑递归的情况，做如下修改：
     function comupute(store, func){
         var comp = function(i){
              if (typeof store[i] !== '某类型'){
                 store[i] = func(comp, i);
                 return store[i];
             }
             return store[i];
         };
         return comp;
     }
     
     // 求和时的func可以这么定义
     var store = [0,1];
     var func = function(fun1, n){
         return fun1(n-1) + fun1(n-2);
     }
     
     // 求阶乘可以这么定义
     var store = [1,1];
     var func = functin(fun1, n){
         return n*fun1(n-1);
     }
     
    ```
#### 继承

JavaScript是一门弱类型语言，从不需要类型转换，对象的起源是无关紧要的，重要的是这个对象能做什么。在基于类的语言中，对象是类的实例，并且类可以从另一个类继承。JavaScript是一门基于原型的语言，这意味着**对象直接从其他（原型）对象继承**。

- 伪类
    - JavaScript的某些像类的语法隐蔽了他的原型机制。**他不让对象直接直接从其他对象继承，反而插入了一个中间层，从而使构造器函数产生对象**。参考：[new操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
        ```
        PS：不管是不是显式继承，只要是new产生对象，内部都使用了原型机制。
        
        当代码 new Foo(...) 执行时，会发生以下事情：
       
        1、一个继承自 Foo.prototype 的新对象被创建。
           > var that = Object.creat(Foo.prototype); // 到这里的时候，其实已经完成了一个对象继承另一个对象。
        2、使用指定的参数调用构造函数 Foo ，并将 this 绑定到新创建的对象。new Foo 等同于 new Foo()，也就是没有指定参数列表，Foo 不带任何参数调用的情况。
           > var other = Foo.call(that, arguments); // 构造函数的返回值
        3、由构造函数返回的对象就是 new 表达式的结果。如果构造函数没有显式返回一个对象，则使用步骤1创建的对象。（一般情况下，构造函数不返回值，但是用户可以选择主动返回对象，来覆盖正常的对象创建步骤）。
           > return (typeof other === 'object' && other) || that;
        ```
        
    - 当一个函数对象被创建时，Function构造器产生的函数对象会运行类似这样的一些代码：
    > this.prototype = {constructer: this}; 
    // 新函数对象被赋予一个prototype属性，其值是包含一个constructor属性且属性值为该新函数对象。

- 对象说明符
    很多时候，构造器要接受一大堆参数，因此在**编写构造器时使其接受一个简单的对象说明符**可能会更加友好。那个对象包含了将要构建的对象规格说明。
        
        ```
        比如：var Foo = function(a,b,c,d){} 
        可以更改为：var Foo = function({
            param1: a,
            param2: b,
            param3: c,
            param4: d
        }){}
        ```
    如果构造器会聪明的使用默认值，则传入的一些参数可以忽略掉。
- 

#### 数组
数组是一段线性分配的内存，他通过计算偏移访问其中的元素。JavaScript并没有提供数组这种数据结构，而是提供了一种拥有一些类数组特性的对象。

它**把数组的下标转变成字符串，用其作为属性**。他明显比真正的数组慢，但它可以方便的使用。属性的检索和更新方式与对象一模一样，除了**有一个可以用整数作为属性名的特性**外。数组有他们自己的字面量格式，数组也有一套非常有用的内置方法。

- 关于数组长度：
    1. 每个数组都有一个**length属性**，并且length是没有上界的，当你用一个大于或等于length的数字作为下标来保存一个元素，那么length将增大来容纳新元素，不会发生数组越界。
    2. length属性的值是这个数组的最大整数属性名加上1，她不一定等于数组里面的属性的个数。
    3. []运算符会将他的表达式转换成一个字符串，如果该表达式有toString方法，就使用该方法的值。这个字符串将被用作属性名。如果则个字符串看起来像一个大于等于这个数组当前的length且小于4294967295的正整数，那么这个数组的length就会被重新设置为新的下标加1。
    4. 可以直接设置length的值，设置更大的length无须给数组分配更多的空间。更小的length会将下标大于等于新length的属性删除。可以通过把下标指定为一个数组当前的length来附加一个新元素到该数组尾部。push方法也可以。

- 删除数组元素
    1. 由于JavaScript数组其实就是对象，所以可以使用delete运算符从数组中移除元素：delete nums[2];
    2. 然而，delete运算符会在数组中遗留一个空洞，被删除的属性和值都消失不见，但length不变。因为排在他后面的元素保留了他们最初的下标。
    ```
    a = [1,2,3,4];
    delete a[1];
    console.log(a); // [1,empty,3,4]
    ```
    3. 为了解决以上问题，可以使用数组的splice方法，他可以删除某些元素并将他们替换成其他的元素，第一个参数是起始位置下标，第二个参数是要删除的元素个数，删除位后面的元素会被删除并且以一个新的键值重新插入，大型数组效率低；当slice的参数为空时，默认返回一个长度和原数组相同的新数组。 
- 数组的枚举
    因为数组其实就是对象，所以for in语句可以用来遍历一个数组的所有属性。不幸的是，for in无法保证无序，并且还有可能意外得到原型链中的属性。

    解决方案：
    
    1. 使用传统的for循环:
        > for(var i = 0; i < arr.length; i++){ console.log(arr[i]); }
    
    2. for...of语句在[迭代对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)（包括Array，Map，Set，String，TypedArray，**arguments** 对象等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句。这种方法只会迭代迭代器中产生的数据。
    
- 使用数组还是对象
    1. 当属姓名是小而连续的整数时，使用数组，否则使用对象。
    2. 使用typeof array会返回'object',并不能判断出类型，因此使用：
        > typeof arr === 'object' && arr.constructor === Array    
 
- 数组的方法
    Javascript提供了一套用于数组的方法，这些方法是被存储在Array.prototype中的函数。
- 多维数组
    JavaScript没有多维数组，但它支持元素为数组的数组
    ```
    Array.prototype.matrix = function(m, n, initial){
        var a, i, j, mat = [];
        for(i=0;i<m;i++){
            a = [];
            for(j=0;j<n;j++){
                a[j] = initial;
            }
            matrix[i] = a; 
        }
        return matrix;
    }
    ```

#### 正则表达式（对象）

JavaScript同样对正则表达式有很好的支持，**RegExp**是JavaScript中的内置类，通过使用RegExp，用户可以自己定义**模式**来对字符串进行匹配。
JavaScript中的String对象的[replace方法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace)也支持使用正则表达式对串进行匹配。

- 元字符与特殊字符
   ``` 
    元字符  含义
    ^       串的开始
    $       串的结束
    *       零到多次匹配
    +       一到多次匹配
    ?       零或一次匹配
    \b      单词边界
    ```
- 范围及重复

    ```
    范围：
        标志符  含义
        […]     在集合中的任一个字符
        [^…]    不在集合中的任一个字符
        .       除\n 之外的任一个字符
        \w      所有的单字，包括字母，数字及下划线
        \W      不包括所有的单字， \w 的补集
        \s      所有的空白字符，包括空格，制表符
        \S      所有的非空白字符
        \d      所有的数字
        \D      所有的非数字
        \b      退格字符
    
    重复：
        标记    含义
        *       0次或多次
        +       1次或多次
        ?       0次或1次
        {n}     重复 n 次
        {n,}    重复 n 或更多次
        {n,m}   重复至少 n 次，至多 m 次
    ```
- 分组与引用

  括号的3个作用：
  1. 将子表达式表记起来，以区别于其他表达式
  2. 括号用来分组，当正则表达式执行完成之后，与之匹配的文本将会按照规则填入各个分组。
  3. 引用分组内容。即在同一个表达式中，后边的式子可以引用前边匹配的文本。
        ```
        var str = "hello, world";
        var str = 'fair enough';
        均为合法字符，我们可能会设计出这样的表达式来匹配该声明：
        var pattern = /['"][^'"]*['"]/;
        看来没有什么问题，但是如果用户输入：
        var str = 'hello, world";
        var str = "hello, world';
        我们的正则表达式还是可以匹配，注意这两个字符串两侧的引号不匹配！ 我们需要的是，前
        边是单引号，则后边同样是单引号，反之亦然。 因此，我们需要知道前边匹配的到底是“单”
        还是“双”。这里就需要用到引用， JavaScript 中的引用使用斜杠加数字来表示，如\1 表
        示第一个分组(括号中的规则匹配的文本)， \2 表示第二个分组，以此类推。 因此我们就设
        计出了这样的表达式：
        var pattern = /(['"])[^'"]*\1/;
        在我们新设计的这个语言中，为了某种原因，在单引号中我们不允许出现双引号，同样，在
        双引号中也不允许出现单引号，我们可以稍作修改即可完成：
        var pattern = /(['"])[^\1]*\1/;
        ```
- 使用正则表达式
    - 创建正则表达式
        1. 使用字面量：
            > var regex = /pattern/[switchs];
        
        2. 使用RegExp对象：
            > var regex = new RegExp("pattern", switchs)
    
    - switchs有以下三种（可以组合使用）：
        ```
        修饰符  描述
        i       忽略大小写开关
        g       全局搜索开关
        m       多行搜索开关(重定义^与$的意义)
        eg:
            var pattern = /^javascript/;
            print(pattern.test("java\njavascript"));//false
            pattern = /^javascript/m;
            print(pattern.test("java\njavascript"));//true
        ```
    - RegExp对象的方法：
        ```
        方法名  描述
        test()  测试串中是否有合乎模式的匹配，不关心匹配结果，返回boolean
        exec()  对串进行匹配，返回需要分组的信息。
        compile() 编译正则表达式，用来改变表达式的模式，这个过程与重新声明一个正则表达式对象的作用相同
        ```
    - String对象中也有多个方法支持正则表达式操作：
        ```
        方法    作用
        match(pattern)   匹配正则表达式，返回匹配数组
        replace(pattern, newStr)     替换
        split(pattern)   分割,返回一个数组
        search(pattern)  查找，返回首次发现的位置
        ```
#### 方法

JavaScript包含了少量用在标准类型上面的标准方法。

- Array
    - arr.concat(item):
        返回一个新数组。它包含arr的浅复制（值拷贝），并将一个或多个item附加在其后，如果item是数组，则对每个元素使用arr.push(item)方法。
    - arr.join(separator)：
        把一个array构造成一个字符串。首先将每个元素构造成一个字符串，并用一个separator为分隔符把他们连接在一起。默认的分隔符是','。为了实现无分隔符的连接，可以使用空字符串作为separator。

        把大量的需要连接的字符串放到一个数组中，然后使用join方法通常比使用'+'号快。
    - arr.pop / arr.push : 数组像堆栈一样工作
    - arr.shift / arr.push : 数组像队列一样工作
    - arr.unshift : 从头部插入数据
    - arr.reverse : 逆置数组元素，返回当前工作数组
    - arr.slice(start, end) : 浅复制从start(arr[start])到end(不包括arr[end])的元素于一个新的数组中返回。
    - arr.splice(start, deleteCount, item...) : 从arr中**移除一个或多个元素**，并用新的item替换他们,返回一个包含被移除元素的数组。
    - arr.sort(compareFunction):
        JavaScript默认的比较函数会将数组元素转换成字符转后按字典序比较。
        自定义比较函数，接收两个参数，如果这两个参数相等则返回0，如果第一个应该排在前面则返回一个负数，如果第二个应该排在前面则返回一个整数。

- Number
    - number.toFixed(fractionDigits)                                 
        把number转换成一个十进制数形式的字符串，可选参数fraction控制小数点后的数字位数，他的值必须在0-20之间，默认为0。
    - number.toPercision(precision)
        把number转换成一个十进制数形式的字符串，可选参数precision控制有效数组的位数，他的值必须在0-21之间。
    - number.toString(redix)
        把number转换成一个以radix进制形式的字符串，radix必须在2-36之间。

- Object
    - obj.hasOwnProperty(name): 判断对象自身是否包含name属性，不会查找原型链。

- String
    - str.charAt(position): 返回position处的字符(串，因为JavaScript并没有character类型)。不合法的position返回空字符串。
    - str.concat(str...): 连接字符串，很少使用，并没有'+'方便。
    - str.indexOf(searchStr, position): 从可选的指定位置查找字符串，返回第一个匹配字符的位置，否则返回-1。
    - str.lastIndexOf(searchStr, position): 倒着找
    - str.match
    - str.replace
    - str.search(pattern): 同indexOf，忽略g参数 
    - str.slice(start, end): 浅复制
    - str.split(separator, llimit): 把str分割成字符数组，limit为数组的长度。
    - str.subString: 同slice，只是不能处理负参数，使用slice替换它

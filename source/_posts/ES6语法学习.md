---
"title": "ES6语法学习"
"date": "2016/1/12 20:46:25"
categories:
- js
- es6

tags:
- js
- es6
---


### let和const命令
#### let和const命令
1. let只在命令所在的代码块内有效。
2. 不存在变量提升。
3. 暂时性死区
4. 不允许重复声明
#### const命令
1. 声明一个只读常量。一旦声明常量值就不能改变。

注意：es6规定，var命令和function命令声明的全局变量，依旧是全局对象的属性；另一方面规定，let命令，const命令，class命令声明的全局变量，不属于全局对象的属性。也就是说，从ES6开始，全局变量将逐步与全局对象的属性脱钩。

### 变量的解构赋值
#### 数组的解构赋值

1. 结构赋值本质上属于模式比配，只要等号两边的模式相同，左边的变量就会被赋予对应的值。
2. 结构赋值允许指定默认的值。
3. 只要某种数据结构具有Iterator接口，度可以采用数组形式的解构赋值。

#### 对象的解构赋值
1. 对象的解构赋值与数组有一个重要的不同。数据的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取得正确的值。
2. 对象的解构赋值内部机制，是先找到同名属性，然后再赋值给对应的变量。真正被赋值的是后者，而不是前者。

#### 字符串的解构赋值
1. 字符串也可以结构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。

#### 数值和布尔值的解构赋值
1. 解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。
2. 解构赋值的规则是，只要等号右边的值不是对象，就先将其转为对象。由于undefined和null无法转为对象，所以对他们进行解构赋值，都会报错.

#### 函数参数的解构赋值
##### 圆括号问题
解构赋值虽然很方便，但是解析起来并不容易。对于编译器来说，一个式子到底是模式，还是表达式，没有办法从一开始就知道，必须解析到（或解析不到）等号才能知道。

由此带来的问题是，如果模式中出现圆括号怎么处理。ES6的规则是，只要有可能导致解构的歧义，就不得使用圆括号。

但是，这条规则实际上不那么容易辨别，处理起来相当麻烦。因此，建议只要有可能，就不要在模式中放置圆括号。

##### 不能使用圆括号的情况
1. 变量声明语句中不能带有圆括号.
2. 函数参数中,模式不能带有圆括号.
3. 赋值语句中,不能讲整个模式,或嵌套模式中的一层,放在圆括号之中.

##### 可以使用圆括号的情况
1. 可以使用圆括号的情况只有一种:赋值语句的非模式部分,可以使用圆括号。

用途
1. 交换变量的值
2. 从函数返回多个值。
3. 函数参数的定义
4. 提取json数据
5. 函数参数的默认值。
6. 遍历Map结构
7. 输入模块的指定方法

### 字符串的扩展

1. 字符的Unicode表示法。
2. codePointAt()，能够正确处理4个字节存储的字符，返回一个字符的码点。
3. String.fromCodePoint()用于从码点返回对应字符，但是这个方法不能识别32位的UTF-16字符
 注意：fromCodePoint方法定义在String对象上，而codePointAt方法定义在字符串的实例对象上。
4. 字符串的遍历器接口，使得字符串可以被for…of循环遍历。
5. at()字符串实例的at方法，可以识别Unicode编号大于）0xFFFF的字符，返回正确的字符。
6. normalize()，将字符的不同表示方法统一为同样的形式，这称为Unicode的正规化.不过，normalize方法目前不能识别三个火三个以上字符的合成。这种情况下，还是只能用正则表达式，通过Unicode编号区间来判断。
7. includes(),startsWith(),endsWith()
8. repeat()方法返回一个新字符串，表示将原字符串重复n次。
9. padStart(),padEnd()ES7推出了字符串不全长度的功能。如果某个字符串不能指定长度，会在头部和尾部不全。padStart用于头部补全，padEnd用于尾部补全。
10. 模板字符串，是增强版的字符串，用反引号标识。它可以当作普通字符串使用，也可以使用定义多行字符串，或者在字符串中嵌入变量。
11. 标签模板，模板字符串可以紧跟在一个函数名后面吗，该函数将被调用来处理这个模板字符串。这被称为标签模板功能。
12. String.raw()ES6还为原生的String对象，提供了一个raw方法。用来充当模板字符串的处理函数，返回一个斜杠都被转义的字符串，对应于替换变量后的模板字符串。

### 正则的扩展
1. RegExp构造函数，如果RegExp构造函数第一个参数是一个正则对象，那么可以使用第二个参数指定修饰符。而且，返回的正则表达式会忽略原有的正则表达式的修饰符，只使用新指定的修饰符。
2. 字符串的正则表达式，字符串对象共有4个方法，可以使用正则表达式：match(),replace(),search()和split()。ES6将这4个方法，在语言内部全部调用RegExp的实例方法，从而做到所有与正则相关的方法，全部定义在RegExp对象上。
3. U修饰符ES6对正则表达式添加了u修饰符，含义为Unicode模式，用来正确处理大于\uFFFF的Unicode字符。也就是说，会正确处理四个字节的UTF-16编码。
4. ES6新增了y修饰符，叫做粘连sticky修饰符y修饰符的作用与g修饰符类似，也是全局匹配，后一次匹配都从上一次匹配成功的下一个位置开始。不同之处在于，g修饰符只要剩余位置中存在匹配就可，而y修饰符确保匹配必须从剩余的第一个位置开始，这也就是“粘连”的涵义。
5. sticky属性，与y修饰符相匹配，ES6的正则对象多了sticky属性，表示是否设置了y修饰符。
ES6为表达式新增了flags属性，会返回正则表达式的修饰符。
6. RegExp.escape()

### 数值的扩展
1. 二进制和八进制表示法,ES6提供了二进制和八进制数值的新的写法，分别用前缀0B或0b和0o或0O
2. ES6在Number对象上，新提供了Number.isFinite()和Number.isNaN()两个方法。Number.isFinite()用来检查一个数值是否为有限的（finite）。Number.isNaN()用来检查一个值是否为NaN。
3. ES6将全局方法parseInt()和parseFloat()，移植到Number对象上面，行为完全保持不变。
4. Number.isInteger()用来判断一个值是否为整数。需要注意的是，在JavaScript内部，整数和浮点数是同样的储存方法，所以3和3.0被视为同一个值。
5. ES6在Number对象上面，新增一个极小的常量Number.EPSILON。实现误差检测机制。
6. Number.isSafeInteger()则是用来判断一个整数是否落在这个范围之内。
7. 指数运算 **

### 数组的扩展
1. Array.from()方法用于将两类对象转为真正的数组：类数组对象和可遍历的对象(包括ES6新增的数据结构Set和Map)
2. Array.of()方法用于将一组值，转换为数组。
3. 数组实例的copyWithin()方法，在当前数组内部，将指定为哈子的成员复制到其他为哈子，然后返回当前数组。会修改当前数组。
4. 数组实例的find()和findIndex()数组实例的find方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员，然后返回该成员。如果没有符合条件的成员，则返回undefined。数组实例的findIndex方法的用法与find方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。另外，这两个方法都可以发现NaN，弥补了数组的IndexOf方法的不足。
5. 数组实例的fill方法使用给定的值，填充一个数组。
6. 数组实例的entries(),keys()和values()用于遍历数组，他们都返回一个便利器对象，可以用for…of循环进行遍历。
7. Array.prototype.includes方法返回一个布尔值，表示某个数组是否包含给定的值，于字符串的includes方法类似。该方法属于ES7,但Babel转码器已经支持。
8. 数组的空位。ES6明确将空位转为undefined.


### 函数的扩展
1. 函数参数的默认值
2. rest参数
3. 扩展运算符

- 合并数组
- 与解构复制结合
- 函数的返回值
- 字符串
- 实现了Iterator接口的对象
- Map和Set结构，Generator函数

4. 函数的name属性，返回该函数的函数名
5. 箭头函数
6. 函数绑定
7. 尾部调用优化

### 对象的扩展
1. 属性的简写表示法
2. 属性名表达式
3. 方法的name属性，返回对象名。
4. Object.is()比较两个值是否相等
5. Object.assign()方法用于对象的合并。注意（浅拷贝）
6. 属性的可枚举性对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。Object.getOwnPropertyDescriptor方法可以获取该属性的描述对象。
7. 属性的遍历

- for…in 自身和继承可枚举
- Object.keys(obj)自身可枚举
- Object.getOwnPropertyNames(Obj)自身所有属性
- Object.getOwnPropertySybols(obj)包含对象自身的所有Symbol属性
- Reflect.ownKeys(obj) 对象自身所有属性，不管属性名是否是Symbol或字符串，也不管是否可枚举。

8. proto属性，Object.setPrototypeOf(),Object.getPrototypeOf()
9. Object.values(),Object.entries()
10. 对象的扩展运算符

### Symbol
### Proxy和Reflect
1. Proxy支持的拦截操作
- get
- set
- has
- deleteProperty
- ownKeys
- getOwnPropertyNames
- getOwnPropertyDescriptor
- defineProperty
- preventExtensions
- getPrototypeOf
- isExtensible
- setPrototypeOf
- apply
- construct

2. Reflect

- get
- set
- has
- deleteProperty
- ownKeys
- getOwnPropertyNames
- getOwnPropertyDescriptor
- defineProperty
- preventExtensions
- getPrototypeOf
- isExtensible
- setPrototypeOf
- apply
- construct


### 二进制数组
1. ArrayBuffer对象
2. TypeArray视图
3. DataView视图

### Set和Map数据结构
1. Set，类似于数组，但是成员值都是唯一的，没有重复的值。
2. WeakSet,成员只能是对象，而不能是其他类型的值。其次，WeakSet中的对象都是弱引用，即垃圾回收机制不考虑WeakSet对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于WeakSet之中。这个特点意味着，无法引用WeakSet的成员，因此WeakSet是不可遍历的。
3. Map
4. WeakMapWeakMap结构与Map结构基本类似，唯一的区别是它只接受对象作为键名（null除外），不接受其他类型的值作为键名，而且键名所指向的对象，不计入垃圾回收机制。


### Iterator和for…of循环
1. Iterator遍历器
2. 数据结构默认的Iterator接口，ES6规定，默认的Iterator接口部署在数据结构的Symbol.iterator属性，或者说，一个数据结构只要具有Symbol.iterator属性，就可以认为是可便利的iterable。调用Symbol.iterator方法，就会得到当前数据结构默认的遍历器生成函数。Symbol.iterator本身就是一个表达式，返回Symbol对象的iterator属性，这是一个预定义好的，类型为Symbol的特殊值，所以要放在方括号内。

- ES6中，有三类数据结构原生具备Iterator接口：数组，某些类似数组的对象，Set和Map结构。
3. 调用Iterator接口的场合

- 解构赋值
- 扩展运算符
- yield*
- 其他场合

4. 字符串的Iterator接口
5. Iterator接口和Generator函数
6. 遍历器对象的retur(),throw()
7. for…of循环

### Generator函数
1. 应用
- 异步操作的同步化表达
- 控制流管理
- 部署iterator接口
- 作为数据结构

### 异步操作和Async函数
1. Generator函数
2. Thunk函数
3. co模块
4. async函数

### class
1. Class的继承
2. Class的Generaotr方法
3. Class的静态方法
4. Class的静态属性和实例属性
5. new.target属性
6. Mixin模式的实现

### 修饰器
1. 类的修饰
2. 类的方法的修饰
3. Mixin
4. Trait

### Moudle
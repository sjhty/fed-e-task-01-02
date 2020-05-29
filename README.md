## 一.描述引用计数的工作原理和优缺点 ##
- 引用计数的核心思想就是在内部去设置一个引用计数器维护当前对象的引用数，去判断当前引用数是否为0来决定当前对象是否为垃圾对象，当引用数为0的时候GC就开始工作，将其对象所在的空间进行回收和释放

```
const user1 = {name: 'tom'}
const user2 = {name: 'jack'}
cosnt student = [user1.name, user2.name]
//此处student的值指向user1和user2的对象空间，当钱执行完后user1和user2的空间由于student值的引用，它们的对象空间就一直存在所以引用计数不会为0

function fn() {
	num1 = 1 //因为此处的变量是指向全局变量，挂载在window上，这时这些变量引用计数不会为0，所以不会被回收
	cosnt num2 = 3 //此处的变量只能被当前作用域使用，所以当fn()执行完之后，num2的计数器就会变为0，这时就会被GC作为垃圾对象对所在的空间进行回收和释放
}

fn() 
```
- 优点
1. 发现垃圾时立即回收
2. 最大限度减少程序暂停

- 缺点
1. 无法回收循环引用的对象
2. 时间开销大


## 二.描述标记整理算法的工作流程 ##
- 标记整理可以看做是标记清除的增强
- 标记阶段的操作和标记清除一致
- 清除阶段会执行整理，移动对象位置

回收前内存中存在散乱活动对象和非活动对象还有空闲空间，当执行标记操作的时候，标记活动对象进行移动整理，把非活动和未使用的对象进行回收，回收后内存中的空间都是连续的，后续申请使用的时候最大化去使用内存当中所释放出来的空间

## 三.描述V8中新生代存储区垃圾回收的流程 ##
- 回收过程采用复制算法+标记整理
- 新生代内存区分为两个等大小空间
- 使用空间为From，空闲空间为To
- 活动对象存储于From空间
- 标记整理后将活动对象拷贝至To
- From 与 To交换空间完成释放
- 回收细节
1. 如果拷贝过程中某一个变量对象所使用的空间在当前的老生代对象里也存在，这里就出现晋升的情况
2. 晋升就是讲新生代对象移至老生代进行存储
3. 一轮GC还存活的新生代需要晋升
4. To空间的使用率超过25%

## 四.描述增量标记算法在何时使用，及工作原理 ##
- 增量标记算法不会等GC执行完才将控制权交回程序，而是一步一步执行，逐步完成垃圾回收，在程序中穿插进行，极大降低GC的最大暂停时间



五.基于以下代码完成下面四个练习
1. 使用函数组合fp.flowRight()重新实现下面这个函数

```
const isLastInStock = fp.flowRight(fp.prop('in_stock'), fp.last)
console.log(isLastInStock(cars)) //false
```
2. 使用fp.flowRight()、fp.prop()和fp.first()获取第一个car的name

```
const isFirstName = fp.flowRight(fp.prop('name'), fp.first)
console.log(isFirstName(cars)) //Ferrari FF
```
3. 使用帮助函数_average重构averageDollarValue,使用函数组合的方式实现

```
let _average = function(xs) { 
	return fp.reduce(fp.add, 0, xs) / xs.length
}
const dollar_value = v => v.dollar_value

let averageDollarValue = fp.flowRight( _average, fp.map(dollar_value))

console.log(averageDollarValue(cars)) //=> 790700
```
4. 使用flowRight写一个sanitizeNames()函数，返回一个下划线链接的小写字符串，把数组中的name转换为这种形式：例如：sanitizeNames(["Hello World"]) => ["hello_world"]

```
let _underscore = fp.replace(/\s+/g, '_')

const sanitizeNames = fp.flowRight(fp.map(fp.flowRight(_underscore(), fp.toLower)))

console.log(sanitizeNames(["Hello Word"])) //=> ["hello_world"]
```

六.基于以下代码完成下面四个练习
1. 使用fp.add(x,y)和fp.map(f,x)创建一个能让functor里的值增加的函数ex1

```
const fp = require('lodash/fp')
const { Maybe, Container } = require('./support')

let maybe = Maybe.of([5, 6, 1])
let ex1 = maybe.map( x => fp.map(fp.add(1), x)) //[6, 7, 2]
```
2. 实现一个函数ex2，能够使用fp.first获取列表的第一个原素

```
let xs = Container.of(['do', 'ray', 'me', 'fa'])
let ex2 = xs.map( x => fp.first(x)) //do
```
3. 实现一个函数ex3，使用safeProp和fp.first找到user的名字的首字母

- 疑问：这个题safeProp方法没看明白，如果这个方法是为了取到name的值，那传入的参数是x: user.name o:user, 怎么返回一个o[x],麻烦老师帮忙分析一下这个方法
- 
4. 使用Maybe重写ex4，不要有if语句

```
let ex4 = Maybe.of('5').map( x => parseInt(x))
console.log(ex4) //不传值时返回undifined, 有值时如果为中文返回NaN,如果为字符串数字或者数字直接返回int型
```
# 绑定规则
this的绑定规则总结起来总共有四种：

1. 默认绑定
2. 隐式绑定
3. 显式绑定
4. new绑定

## 默认绑定
这条规则是无法应用其它绑定时的默认规则，思考以下代码：

```javascript
    function foo() {
        console.log(this.a)
    }

    var a = 2
    foo() // 2
```

`foo()`是直接使用不加任何修饰的函数引用进行调用的，因此只能应用默认绑定，*在非严格模式下*`this`默认绑定到了全局对象

严格模式下则被绑定到`undefined`

```javascript
    function foo() {
        "use strict"
        console.log(this.a)
    }

    var a = 2
    foo()  // TypeError
```

## 隐式绑定
另一条需要考虑的规则是调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含，

思考下面代码：

```javascript
    function foo() {
        console.log(this.a)
    }

    var obj = {
        a: 2,
        foo: foo
    }

    obj.foo() // 2
```

调用位置`obj.foo()`会使用obj上下文来引用函数，因此可以说函数被调用时`obj`对象"拥有"或者"包含"它，当函数引用有
上下文对象时，*隐式绑定*会把函数调用中的`this`绑定到这个上下文对象。

*对象引用链只有最后一层在调用位置起作用*， 举例来说

```javascript
    function foo() {
        console.log(this.a)
    }

    var obj2 = {
        a: 42,
        foo: foo
    }

    var obj1 = {
        a: 2,
        obj2: obj2
    }

    obj1.obj2.foo()  // 42
```

### 隐式丢失问题
一个常见的问题就是被隐式绑定的函数会丢失绑定对象。

例子：

```javascript
    function foo() {
        console.log(this.a)
    }

    var obj = {
        a: 2,
        foo: foo
    }

    var bar = obj.foo
    var a = "global"

    bar() // global
```

虽然`bar`引用的是`obj.foo`，但是`bar`实际上引用的是`foo`函数本身，因此，此时的`bar()`是一个不带
任何修饰的函数调用，所以`foo`应用了默认绑定。

## 显式绑定

JavaScript中所有的函数都具有的方法`call`和`apply`

通过`bar.call(obj)`可以强制将`foo`的`this`绑定到`obj`上

如果`call`和`apply`被传入了一个原始值(字符串类型，布尔类型或者数字类型)来当做`this`的绑定对象，这个原始值会被转换成
对象的形式(也就是 new String(...)...)，这通常称作"装箱"

### 硬绑定
硬绑定的典型应用场景就是创建一个包裹函数，负责接收函数参数并返回值

```javascript
    ...
    var bar  = function() {
        return foo.apply(obj, arguments)
    }

    bar()  // 2
```

或者构建一个简单的绑定辅助函数

```javascript
    function bind(fn, obj) {
        return function() {
            fn.apply(obj, arguments)
        }
    }
```

#### ES5中内置的`bind`方法
`bind()`会返回一个硬绑定的函数，无论在哪里调用(除了使用new)，这个`this`以及传入的`arguments`都会生效

### API调用上下文

许多函数提供了绑定`this`的参数，比如`forEach(callback, thisArg)`

```javascript
    [1,2,3].forEach(foo, obj)
```

## new绑定

思考下面代码：

```javascript
    function foo(a) {
        this.a = a
    }

    var bar = new foo(2)
    console.log(bar.a) // 2
```
使用`new`来调用`foo`时，构造的新对象会被绑定到`foo(...)`调用中的`this`上

# 优先级

new绑定 > 显式绑定 > 隐式绑定 > 默认绑定

# 绑定例外

## 被忽略的this

如果把`null`, `undefined`,作为`this`的绑定对象传入`call``apply`或者`bind`，这些值在调用时会被
忽略，实际应用的是默认绑定规则。

## 间接引用

**经典的例子**：

```javascript
    var a = 2
    var o = { a: 3, foo: foo }
    var p = { a: 4 }

    o.foo()  // 3
    (p.foo = o.foo)()  //2
```

赋值表达式`p.foo = o.foo`返回的是目标函数的引用，因此调用位置是`foo()`，这里会应用默认绑定。


# 箭头函数

箭头函数不使用`this`的四种标准规则，而是根据外层的作用域来决定this,箭头函数的绑定无法修改，new 也不行。










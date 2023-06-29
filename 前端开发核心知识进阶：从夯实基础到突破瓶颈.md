## this

### 全局環境中的 this

函式在瀏覽器全局環境中被簡單調用，在非嚴格模式下 this 指向 window，在通過 use strict 指明嚴格模式的情況下指向 undefined

```
function f1 () {
    console.log(this)
}

function f2 () {
    'use strict'
    console.log(this)
}

f1() // window
f2() // undefined
```

這裡的 this 仍然指向 window，所以會印出 `window`, `undefined`

```
const foo = {
    bar: 10,
    fn: function () {
        console.log(this)
        console.log(this.bar)
    }
}
var fn1 = foo.fn
fn1()

等價於 =>
console.log(window)
console.log(window.bar)
```

但如果改為以下形式
此時 this 指向的是最後調用它的對象，在 foo.fn() 語句中，this 指向 foo 對象，請記住在執行函式時不考慮顯示綁定，如果函式中的 this 是被上一級的對象所調用，那 this 指向的就是上一級的對象，否則指向全域環境

```
const foo = {
    bar: 10,
    fn: function () {
        console.log(this)
        console.log(this.bar)
    }
}
foo.fn()

// {bar: 10, fn: f}
// 10
```

### 上下文對象調用中的 this

```
const student = {
    name: 'Lucas',
    fn: function () {
        return this
    }
}
console.log(student.fn() === student)   // true
```

this 最後會指向最後調用它的對象，因此輸出會是 Mike

```
const person = {
    name: 'Lucas',
    brother: {
        name: 'Mike',
        fn: function () {
            return this.name
        }
    }
}
console.log(person.brother.fn())    // Mike
```

第一個 console 沒問題，主要是第 2, 3 個
第 2 個 console 中 o2.fn() 最終調用的還是 o1.fn()，因此運行結果還是 o1
第 3 個 console 中 o3.fn() 通過 var fn = o1.fn 的賦值進行了 "裸奔" 調用，因此這裡的 this 指向 window 運行結果當然是 undefined

```
const o1 = {
    text: 'o1',
    fn: function () {
        return this.text
    }
}

const o2 = {
    text: 'o2',
    fn: function () {
        return o1.fn()
    }
}

const o3 = {
    text: 'o3',
    fn: function () {
        var fn = o1.fn
        return fn()
    }
}

console.log(o1.fn())    // o1
console.log(o2.fn())    // o1
console.log(o3.fn())    // undefined
```

那如果我希望 `console.log(o2.fn())` 結果是 `o2`？
(不能用 bind, call, apply 來對 this 的指向進行干預...!)

```
const o1 = {
    text: 'o1'
    fn: function () {
        return this.text
    }
}

const o2 = {
    text: 'o2'
    fn: o1.fn
}

console.log(o2.fn())
```

那如果用 bind, call, apply 的話... (他們都是用來改變相關函數 this 指向的)
差別在 call, apply 是直接進行相關函數調用的，bind 不會執行相關函數而是返回一個新的函數，這個新的函數已經自動綁定了新的 this 指向，開發者可以手動調用它，而 call, apply 之間區別主要體現在參數設定上
下面三段 code 是等價的

```
const target = {}
fn.call(target, 'arg1', 'arg2')

const target = {}
fn.apply(target, ['arg1', 'arg2'])

const target = {}
fn.bind(target, 'arg1', 'arg2')()
```

```
const foo = {
    name: ''lucas',
    logName: function () {
        console.log(this.name)
    }
}
const bar = {
    name: 'mike'
}
console.log(foo.logName.call(bar))    // mike
```

### 構造函數和 this

```
function Foo () {
    this.bar = 'Lucas'
}
const instance = new Foo ()
console.log(instance.bar)    // Lucas
```

new 做了什麼？ 創建一個新對象、將構造函數的 this 指向這個新的對象、為這個對象添加屬性、方法、最終返回新對象

```
var obj = {}
obj.__proto__ = Foo.prototype
Foo.call(obj)
```

如果是顯式 return
1: 將輸出 undefined, 此時 instance 返回的是空對象 o

```
function Foo () {
    this.user = 'Lucas'
    const o = {}
    return o
}
const instance = new Foo()
console.log(instance.user)
```

2: 將輸出 `Lucas`，也就是說 instance 此時返回的是目標對象實例 this

```
function Foo() {
    this.user = 'Lucas'
    return 1
}
const instance = new Foo()
console.log(instance.user)
```

所以如果構造函數中顯式返回一個值，且返回的是一個對象 (複雜類型)，那麼 this 就指向這個返回的對象; 如果返回的不是一個對象 (基本類型)，那麼 this 仍然指向實例

### 箭頭函數中的 this

在箭頭函數中， this 的指向是由外層 (函數或全局) 作用域來決定的

```
const foo = {
    fn: function () {
        setTimeout(function() {
            console.log(this)
        })
    }
}
console.log(foo.fn())
```

如果需要讓 this 指向 foo 這個對象，則可以巧用箭頭函數來解決

```
const foo = {
    fn: function () {
        setTimeout(() => {
            console.log(this)
        })
    }
}
console.log(foo.fn())

// {fn: f}
```

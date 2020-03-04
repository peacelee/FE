ECMAScript
---
### 目录
- <a href="#new">new操作符实现</a>
- <a href="#promise">Promise实现</a>
- <a href="#call&apply">call/apply/bind实现</a>

### <a name="new">new操作符实现</a>
```javascript
// 模拟new操作符
function _new() {
    // 创建新对象
    let obj = new Object();
    // 解析参数
    let [ctr, ...args] = [...arguments];
    // 执行构造函数的原型对象
    obj.__proto__ = ctr.prototype;
    // 调用构造函数
    let result = ctr.apply(obj, args);

    // 引用类型直接返回 
    if(typeof result === 'function' || (typeof result === 'object' && result !== null)) {
        return result
    }

    // 返回新对象
    return obj
}

function Person(name) {
    this.name = name
}

const people = _new(Person, 'leon');
```

### <a name="promise">Promise实现</a>
```javascript
// 手写promise
class Promise {
    constructor(fn) {
        this.state = 'pending';
        this.value = undefined;
        this.reason = undefined;
        this.onFulfilledFns = [];
        this.onRejectedFns = [];

        const resolve = (value) => {
            if (this.state === 'pending') {
                this.value = value;
                this.state = 'fulfilled';
                this.onFulfilledFns.forEach(fn => {
                    fn()
                })
            }
        };
        const reject = (reason) => {
            if (this.state === 'pending') {
                this.reason = reason;
                this.state = 'rejected';
                this.onRejectedFns.forEach(fn => {
                    fn()
                })
            }
        };
        fn(resolve, reject)
    }

    then(onFulfilled, onRejected) {
        const promiseNext = new Promise((resolve, reject) => {
            if (this.state === 'fulfilled') {
                setTimeout(() => {
                    const x = onFulfilled(this.value);
                    resolvePromise(promiseNext, x, resolve, reject);
                }, 0)
            }
            if (this.state === 'rejected') {
                setTimeout(() => {
                    const x = onRejected(this.reason);
                    resolvePromise(promiseNext, x, resolve, reject);
                }, 0)
            }
            if (this.state === 'pending') {
                this.onFulfilledFns.push(() => {
                    setTimeout(() => {
                        const x = onFulfilled(this.value);
                        resolvePromise(promiseNext, x, resolve, reject);
                    }, 0)
                })
                this.onRejectedFns.push(() => {
                    setTimeout(() => {
                        const x = onRejected(this.reason);
                        resolvePromise(promiseNext, x, resolve, reject);
                    }, 0)
                })
            }
        });

        return promiseNext;
    }
}

const resolvePromise = (promiseNext, x, resolve, reject) => {
    if (promiseNext === x) {
        return reject(new TypeError('循环引用'))
    }

    const then = x.then;
    if (typeof then === 'function') {
        then.call(x, r => {
            resolvePromise(promiseNext, r, resolve, reject)
        }, err => {
            reject(err)
        })
    } else {
        resolve(x)
    }
}
```

### <a name="call&apply">call/apply/bind实现</a>
```javascript
// call实现
Function.prototype.call = function() {
    // 解析参数
    let [thisArg, ...args] = [...arguments];
    // 如果没有指定this，则指向window或者global
    if(!thisArg) {
        thisArg = typeof window === 'undefined' ? global : window
    }
    // 挂载方法
    thisArg.func = this;
    // 调用
    let result = thisArg.func(...args);
    // 删除临时方法
    delete thisArg.func;

    return result
}

const objC = {
    h: 'hello'
}

function print(w) {
    console.log(this.h + ' ' + w)
}

print.call(objC, 'world')

// apply实现
Function.prototype.apply = function(thisArg, rest) {
    // 返回结果
    let result;
    // 如果没有指定this，则指向window或者global
    if(!thisArg) {
        thisArg = typeof window === 'undefined' ? global : window
    }
    // 挂载方法
    thisArg.func = this;

    // 调用
    if(!rest) {
        result = thisArg.func();
    }else {
        result = thisArg.func(...rest);
    }

    // 删除临时方法
    delete thisArg.func;

    return result
}

const objA = {
    h: 'hello'
}

function print(w, q) {
    console.log(this.h + ' ' + w + q)
}

print.apply(objA, ['world', '!'])

// bind实现
Function.prototype.bind = function() {
    let fn = this;
    let [thisArg, ...args] = [...arguments]
    return function() {
        fn.apply(thisArg, [...args].concat([...arguments]))
    }
}

const objB = {
    h: 'hello'
}

function print(w, q) {
    console.log(this.h + ' ' + w + q)
}

let printFn = print.bind(objB, 'world');
printFn('!')
```
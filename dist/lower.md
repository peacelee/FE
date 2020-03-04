ECMAScript
---
### 目录
- <a href="#new">new操作符实现</a>
- <a href="#promise">Promise实现</a>

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
没看懂问题的意思是什么。所以这里尝试解释各个部分是什么作用。

```js
new Vue({
    data: {
        name: 'numb',
        message: 'hello'
    }
})
```

1. 在 `defineReactive` 时 `new Dep`，每个实例都有一个唯一的 id
2. `watcher = new Watcher`
3. `watcher.get()`
    * Dep.taget = this
    * this.getter
4. `defineReactive` 中的 get
    * `dep.depend()`
    * 判断指令添加过依赖没（通过 id)
    * 没添加过：dep.subs.push(watcher)
5. this.name = 'sunorry'
    * `defineReactive` 中的 set
    * `dep.nofify()`
    * `watcher.update()`

`dep` 中多个 `watcher`，在添加 `watcher` 的时候，会判断添加过没有，数据变化，会执行 `dep.subs.forEach(el => el.update())`

所以猜测 `watcher` 的作用就是保证添加的时候不会被添加多次，（如 HTML 中有两段用到同一个数据 `{{message}} {{message}}`）和 `update` 方法（里面）

还是有很多疑问，比如 `this.message = 'xxx'`，为什么还会执行一次 `name` 的 `get`。是因为我们对 `data:{}`, `name`, `message` 都 new Dep 了么，还是有点晕。 
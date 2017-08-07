```js
new Vue({
    data: {
        name: 'numb',
        message: 'hello'
    }
})
```

1. 触发 getter 时(new Watcher)，`new Dep`，内部缓存一个 `uid`
2. `depend` 添加依赖，调用 `watcher` 的 `addDep`，根据 `uid` 来判断是否需要添加依赖
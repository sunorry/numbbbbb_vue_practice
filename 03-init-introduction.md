# elegant code

## `instance/state.js`

```js
const sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
}
export function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

这段代码挺有意思的，就不用每次都写老长一段代码了。

`this.name` 其实是 `this_data.name`，是如何做到的呢？

Fn: `initData`

```js
function initData (vm: Component) {
  // data 即 new Vue({ data: { message: 'Hello Vue!' } }) 中的 data
  let data = vm.$options.data
  // vm._data 即 data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  const keys = Object.keys(data)
  let i = keys.length
  while (i--) {
      // key 是 messgae，所以访问 vm.message 的时候访问的是 vm._data.message(看 `proxy` 中的 `get`)
      proxy(vm, `_data`, key)
  }
}
```


## `instance/events.js`

自定义事件
一个小 tip。

```js
// me
if(!vm._events[event]) {
    vm._events[event] = []
}
vm._events[event].push(fn)

// vue
(vm._events[event] || (vm._events[event] = [])).push(fn)
```

`$once` 的实现是，和普通的 `on` 一样，不同的是 `on` 时对它包装一下（trigger 的时候先 `off`，再 fn()）。

# pracitce

## `instance/inject`

看 [provide-ubhect](https://vuejs.org/v2/api/#provide-inject) 才知道做什么用的，大概意思是在父组件中注入一个值，在所有的自组件中都可拿到。但不是 reactive 的，且作者故意这么做的，但是也可改变。

问题

1. 为什么作者故意做成不是 reactive
2. 实战中有什么用

第一个问题很简单，如果是 reactive 的，那么子组建是不是就可以改变父组建的数据，这本来就是合理的。
第二个问题，我想到一种非常有用的情景。
想象页面拥有多个子组建，在子组建中要拿到 `vue-route` 上的参数，渲染或者调接口时用，这时候 `provide-reject` 就非常有用了，减少了我们在父组建向子组建传值这么个过程。可以说非常方便了。

但是其实在子组建是可以改变这个值的，实例化时，也会对属性做一次 Observe，但是改变只针对于自己，其他子组建不会影响，也就是相当于在子组建的 `data` 写了这个属性，并设置了初始值。

```
initInjections(vm) // resolve injections before data/props
initState(vm)
initProvide(vm) // resolve provide after data/props
```

Why do they have this order? I'll let you answer this.

Hint: think from another side, what will happen if you change their order?

掉到坑里了，我看的版本是 `2.3.3`，它就是先 `initProvide` 的。想死。


```js
new Vue({
    provide: {
        name: 'numb'
    },
    inject: ['name']
})
```

`inject` 的做法是从自己往上遍历开始拿 `this.provide` 上与自己 key 一样的，如果先 `initProvide` 那么如果 key 相同，拿到的永远是自己的，而不是从父级传过来。不过我觉得遍历的时候从父级开始也可以避归这个问题。作者专门改了执行顺序，应该不是这个原因，拿到自己 `provide` 的是正常操作了。那就可能是优先级问题了，先 `provide` 的话，会让 `data` 上设置的初始值失效，应该就是这个原因吧。

还望高人指点。
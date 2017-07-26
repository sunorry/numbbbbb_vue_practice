# util functions

## `util/index.js`

Fn: `query`

`typeof el === 'string'`，如果是 dom 的话 `typeof` 会返回 `object`，如果想看具体的类型的话使用 `Object.prototype.toString.call(somedom).slice(8, -1)`。

## `shared/util`

Fn: `makeUp`

接收 `mei,nan,zi` 这样的字符串，包装成一个对象，返回一个 function，类似于下面这样

```js
(function () {
    const map = {
        mei: true,
        nan: true,
        zi: true
    }

    return function(val) {
        return map[val]
    }     
})()

```

有真用呢，比如检查变量是否是在一大堆中保留字中的一个，后面看看作者怎么用的吧。

## `util/compact`
Fn: `shouldDecodeNewlines`
真的是奇技淫巧。大概的意思是 IE 会将 `<div a="\n">` 中的 `\n` 转换成 `&#10;`(换行)。
## 聊一聊如何在JavaScript中实现不可变对象？

实现不可变数据有三种主流的方法

1. 深克隆，但是深克隆的性能非常差，不适合大规模使用
2. Immutable.js，Immutable.js是自成一体的一套数据结构，性能良好，但是需要学习额外的API
3. immer，利用Proxy特性，无需学习额外的api，性能良好
4. Object.preventExtensions() 防止扩展
   此方法可防止向现有对象添加新属性，`preventExtensions()` 是不可逆的操作，我们永远不能再向对象添加额外的属性。

```js
const myTesla = {
 maxSpeed: 155,
 batteryLife: 300,
 weight: 2300
};
Object.isExtensible(myTesla); // true
Object.preventExtensions(myTesla);
Object.isExtensible(myTesla); // false
myTesla.color = 'blue';
console.log(myTesla.color) // undefined
```

1. Object.seal() 密封
   它可以防止添加或删除属性，`seal()` 还可以防止修改属性描述符。

```js
Object.isSealed(myTesla); // false
Object.seal(myTesla);
Object.isSealed(myTesla); // true

myTesla.color = 'blue';
console.log(myTesla.color); // undefined

delete myTesla.batteryLife; // false
console.log(myTesla.batteryLife); // 300

Object.defineProperty(myTesla, 'batteryLife'); // TypeError: Cannot redefine property: batteryLife
```

1. Object.freeze() 冻结
   它的作用与 `Object.seal()` 相同，而且它使属性不可写。

```js
Object.isFrozen(myTesla); // false
Object.freeze(myTesla);
Object.isFrozen(myTesla); // true

myTesla.color = 'blue';
console.log(myTesla.color); // undefined

delete myTesla.batteryLife;
console.log(myTesla.batteryLife); // 300

Object.defineProperty(myTesla, 'batteryLife'); // TypeError: Cannot redefine property: batteryLife

myTesla.batteryLife = 400;
console.log(myTesla.batteryLife); // 300
```
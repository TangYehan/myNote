### set

+ `set`是一种叫做**集合**的数据结构

+ 基本用法：

  ```js
  const set = new Set([1,2,2,3,4,4,5]);
  [...set] //[1,2,3,4,5]
  set.size = 5
  ```

+ 基本方法：`add()`,`delete()`,`has()`,`clear()`
+ 遍历操作：
  + `Set.prototype.keys()`：返回键名的遍历器
  + `Set.prototype.values()`：返回键值的遍历器
  + `Set.prototype.entries()`：返回键值对的遍历器 //[red,red]
  + `Set.prototype.forEach()`：使用回调函数遍历每个成员，第二个参数可以指定`this`指向

+ Array 和 Set 对比：

  + `Array` 的 `indexOf` 方法比 `Set` 的 `has` 方法效率低下

  + `Set` 不含有重复值（可以利用这个特性实现对一个数组的去重）

  + `Set` 通过 `delete` 方法删除某个值，而 `Array` 只能通过 `splice`。两者的使用方便程度前者更优

  + `Array` 的很多新方法 `map`、`filter`、`some`、`every` 等是 `Set` 没有的（但是通过两者可以互相转换来使用）


​    

### WeakSet

+ 与`set`区别：
  + `WeakSet `的成员只能是对象，而不能是其他类型的值
  + `WeakSet `中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用，也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。
  + `WeakSet `不可遍历，没有`size`属性

+ 基本方法：`add()`,`delete()`,`has()`



### 转换

+ `array => set : new Set(array)`
+ `set => array : [...set]`

### map

+ `Map` 是一种叫做**字典**的数据结构。

+ 基本用法：

  ```js
  let map = new Map();
  map.set(key,value);
  map.has(key) //true/false
  map.get(key) //value
  map.delete(key)
  map.clear()
  ```

+ 遍历方法
  + `Map.prototype.keys()`：返回键名的遍历器。
  + `Map.prototype.values()`：返回键值的遍历器。
  + `Map.prototype.entries()`：返回所有成员的遍历器。
  + `Map.prototype.forEach()`：遍历 Map 的所有成员。

+ 与对象对比
  + Map的key相比较普通对象来说更为灵活，普通对象的key只能以基础数据类型作为key值，并且所有传入的key值都会被转化成string类型，而Map的key可以是各种数据类型格式。



### map与其他结构的转换

#### 数组 <=> map

```js
//map转数组；
let map = new Map()
map.set('a',1).set('b',2)
[...map]  //[Array(2), Array(2)]

//数组转map
let arr =[[a,1],[b,2]]
let map = new Map(arr)//Map(2) {1 => 2, 3 => 4}
```

#### 对象 <=> map

```js
//map转对象
function map2Obj(map){
    let obj = {}
    for(let [k,v] of map){
        obj[k] = v
	}
    return obj
}

//对象转map
let map = new Map()

//或者

function obj2Map(obj){
    let map = new Map()
    for(let item of Object.keys(obj)){
        map.set(item,obj[item])
	}
    return map
}
```

#### JSON <=> map

```js
//map => JSON
//若key都为字符串型
let map = new Map()
JSON.stringify(map2Obj(map))

//若key含有非字符串
JSON.stringify([...map])

//JSON => map
//key都为字符串
obj2Map(JSON.parse(jsonStr))

//key含有非字符串
new Map(JSON.parse(jsonStr))
```

 

### weakMap

+ 与map区别：
  + 对象作为键名
  + 没有遍历操作
  + 没有`size`属性
  + 弱引用

#### 应用

+ DOM对象上保存相关数据，方法等。若元素被移除，不会因为`weakmap`的引用而不回收其内存

  ```js
  let wm = new WeakMap(), element = document.querySelector(".element");
  wm.set(element, "data");
  
  let value = wm.get(elemet);
  console.log(value); // data
  
  element.parentNode.removeChild(element);
  element = null;
  ```

+ 数据缓存 （参照深拷贝内对`weakmap`的用法）

+ 私有属性
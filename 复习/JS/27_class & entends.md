+ [参考资料](https://juejin.cn/post/7001025002287923207)

  

### class

```js
/*
	class里用到等号的都是挂载到实例上的
*/
class A {
    static type = "person";
    name = "lala";  //实例上 等同于constructor{this.name = "lala"}
    age = _ => 18; //实例上
    hi(){ console.log("hi");} //原型上
}
```

编译后：

```js
"use strict";

/*
	检测是否将A以构造函数形式调用。
	如果是以构造函数方式调用，let a = new A()，则a.__proto__ = A.prototype
	如果不是 let a = A(),则 a可能根本没有__proto__属性
	因此 es6的class不能当作普通函数调用，否则会报错
*/
function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

/*
	将属性挂载到target上,简单来说就是target.xx = xx
	这里用于将属性挂载到原型链上
*/

function _defineProperties(target, props) {
  for (var i = 0; i < props.length; i++) {
    var descriptor = props[i];
    descriptor.enumerable = descriptor.enumerable || false;
    descriptor.configurable = true;
    if ("value" in descriptor) descriptor.writable = true;
    Object.defineProperty(target, descriptor.key, descriptor);
  }
}

/*
	将static方法挂载到A上
	将非static方法挂载到A的原型上 A.prototype
	此外，将A的prototype 的 writable属性设置为false
*/
function _createClass(Constructor, protoProps, staticProps) {
  if (protoProps) _defineProperties(Constructor.prototype, protoProps);
  if (staticProps) _defineProperties(Constructor, staticProps);
  Object.defineProperty(Constructor, "prototype", { writable: false });
  return Constructor;
}


/*
	将属性挂载到实例或对象上，简单来说就是obj.name = name
*/
function _defineProperty(obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerasble: true,
      configurable: true,
      writable: true,
    });
  } else {
    obj[key] = value;
  }
  return obj;
}

var A = /*#__PURE__*/ (function () {
  function A() {
    _classCallCheck(this, A);
	//挂载到实例上的属性/函数
    _defineProperty(this, "name", "lala");

    _defineProperty(this, "age", function (_) {
      return 18;
    });
  }

    //挂载到原型上的属性/函数
  _createClass(A, [
    {
      key: "hi",
      value: function hi() {
        console.log("hi");
      },
    },
  ]);

  return A;
})();

//静态方法挂载到A上
_defineProperty(A, "type", "person");
```



### extends

```
class A {
   constructor(name){
   		this.name = name
   }
   sayhi(){}
}

class B extends A {
  constructor(name, age) {
    super(name);
    this.name = name;
  }
  getName() {
    return this.name;
  }
}
```

编译后

```js
"use strict";
//...A的编译同上,此处省略

function _typeof(obj) {
  "@babel/helpers - typeof";
  return (
    (_typeof =
      "function" == typeof Symbol && "symbol" == typeof Symbol.iterator
        ? function (obj) {
            return typeof obj;
          }
        : function (obj) {
            return obj &&
              "function" == typeof Symbol &&
              obj.constructor === Symbol &&
              obj !== Symbol.prototype
              ? "symbol"
              : typeof obj;
          }),
    _typeof(obj)
  );
}

/*
	检测继承对象的类型
	然后将 B.prototype = A.prototype , 同时将B.prototype的writable属性设置为false
	B.__proto__ = A
*/
function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function");
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: { value: subClass, writable: true, configurable: true },
  });
  Object.defineProperty(subClass, "prototype", { writable: false });
  if (superClass) _setPrototypeOf(subClass, superClass);
}

/*
	将 B 设置为 A 的实例
	B.__proto__ = A
*/
function _setPrototypeOf(o, p) {
  _setPrototypeOf =
    Object.setPrototypeOf ||
    function _setPrototypeOf(o, p) {
      o.__proto__ = p;
      return o;
    };
  return _setPrototypeOf(o, p);
}


function _createSuper(Derived) {
  var hasNativeReflectConstruct = _isNativeReflectConstruct();
  return function _createSuperInternal() {
    var Super = _getPrototypeOf(Derived),
      result;
    if (hasNativeReflectConstruct) {
      var NewTarget = _getPrototypeOf(this).constructor;
      result = Reflect.construct(Super, arguments, NewTarget);
    } else {
      result = Super.apply(this, arguments);
    }
    return _possibleConstructorReturn(this, result);
  };
}

function _possibleConstructorReturn(self, call) {
  if (call && (_typeof(call) === "object" || typeof call === "function")) {
    return call;
  } else if (call !== void 0) {
    throw new TypeError(
      "Derived constructors may only return object or undefined"
    );
  }
  return _assertThisInitialized(self);
}

function _assertThisInitialized(self) {
  if (self === void 0) {
    throw new ReferenceError(
      "this hasn't been initialised - super() hasn't been called"
    );
  }
  return self;
}

/*
	检测有没有Reflect存在
*/
function _isNativeReflectConstruct() {
  if (typeof Reflect === "undefined" || !Reflect.construct) return false;
  if (Reflect.construct.sham) return false;
  if (typeof Proxy === "function") return true;
  try {
    Boolean.prototype.valueOf.call(
      Reflect.construct(Boolean, [], function () {})
    );
    return true;
  } catch (e) {
    return false;
  }
}

function _getPrototypeOf(o) {
  _getPrototypeOf = Object.setPrototypeOf
    ? Object.getPrototypeOf
    : function _getPrototypeOf(o) {
        return o.__proto__ || Object.getPrototypeOf(o);
      };
  return _getPrototypeOf(o);
}

var B = /*#__PURE__*/ (function (_A) {
  _inherits(B, _A);

  var _super = _createSuper(B);

  function B(name, age) {
    var _this;

    _classCallCheck(this, B);

    _this = _super.call(this, name);
    _this.name = name;
    return _this;
  }

  _createClass(B, [
    {
      key: "getName",
      value: function getName() {
        return this.name;
      },
    },
  ]);

  return B;
})(A);

```


# 属性特征

### 修改/定义属性特征方法
#### 数据属性
```
var person = {};
Object.defineProperty(person, "name", {
    writable: false,
    value: "Nicholas",
    configurable: "false"
});
```
#### 访问器属性
```
var book = {
        _year: 2004,
        edition: 1
    };
Object.defineProperty(book, "year", {
    get: function() {
        return this._year;
    },
    set: function() {
        if (newValue > 2004) {
            this._year = newValue;
            this.edition += newValue - 2004;
        }
    }
});

```
#### 定义多个属性

```
var book = {};
Object.defineProperties(book, {
    _year: {
        value: 2004
    },
    
    edition: {
        value: 1
    },
    
    year: {
        get//etc...
    }
)
```

#### 读取属性的特性

```
var descriptor = Object.getOwnPropertyDescriptor(book, "_year");
```

### 数据属性
属性特征 | 描述 | 直接在对象上定义属性时默认值 | 使用var定义属性时默认值
---|---|---|---
`[[Configurable]]`|能否delete/修改特性/变为访问器|`true`||
`[[Enumerable]]`|能否for-in|`true`||
`[[Writable]]`|能否修改属性值|`true`||
`[[value]]`|包含了属性的属性值|`undefined`||

### 访问器属性

属性特征 | 描述 | 默认值 | 使用var定义属性时默认值
---|---|---|---
`[[Configurable]]`|能否delete/修改特性/变为数据属性|`true`||
`[[Enumerable]]`|能否for-in|`true`||
`[[Get]]`|读取时调用的函数|`undefined`||
`[[Set]]`|写入时调用的函数|`undefined`||


### 递归最佳实践

```
var factorial = (function f(num){
    if (num <= 1){
        return 1;
    } else {
        return num * f(num-1);
    }
});
```

#### 优势
1. 防止最开始的factorial被修改指向无法调用自身
2. 防止了`严格模式`下`arguments.callee`被禁用




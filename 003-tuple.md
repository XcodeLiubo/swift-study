<font size = 4>
# 元组(初识)
### 概念
swift中元组的概念和c++中的tuple一样, 只不过在c++中tuple可以在用户层面实现(<font color = red>可变参数模板</font>), 这个在c++的学习中笔者已经手工实现了一个简单的tuple,并实现了`get<N>`的原理. 也可以C语言中实现简单的tuple, 这里就不多追究了. 在swift中元组更简单, 它是编译器提供的语法

```swift
let t1 = ("tierry", 18)
print(t1.0)             // 取出tierry
print(t1.1)             // 取出18

do {
    let (name, age) = ("tierry", 18)    // 一一对应赋值到对象
    print(name)                         // tierry
    print(age)                          // 18
}

do{
    let (name, _) = ("tierry", 18)    // 忽略第2个对象, 只要第1个
    print(name)                         // tierry
}

do {
    let tuple = (name: "tierry", age: 18)   // 
    print(tulpe.0)
    print(tulpe.name)
    print(tulpe.1)
    print(tulpe.age)
}

do {
    let tuple = (name: "tierry")        // error, 元组必须有2个以上元素的组合
}
```

### tuple作为类型传递
tuple可以作为类型当作参数和返回值传递
```swift
func test(tuple: (a : Int, b : Int)) -> (c : Int, d : Int){
    print(tuple.0)
    print(tuple.a)
    print(tuple.1)
    print(tuple.b)
    return (c : tuple.b, d : tuple.b)
}
print(test(tuple: (2, 10)))
```

声明时可以省略标签[^ann-tag], 调用时也可以省略标签


[^ann-tag]: 现在还未学习到函数, 标签是参数的一部分, 所以调用函数时,必须指定标签

</font>

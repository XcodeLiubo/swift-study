<font size = 4>

# guard
### 语法
guard在swift中表示提前退出, 它的的语法格式: 
1. 接收Bool
2. 没有`true`的执行体
3. 只有else执行体(<font color = red>绝大部分场景下必须退出当前作用域</font>)

```swift
guard true else {
    // 这是一个永远不会进入else的代码, 因为编译器可以直接判断条件是成立的
}

guard false eles {
    // 一定会进入, 编译器可以明确推断出条件是false
    // 必须有退出当前作用域的语句
}

guard let _ = Int("abc") else {
    // Int("abc")返回可选
    // guard使用可选绑定, 编译器会判断为Bool
    // 此guard可能会进入到else, 必须有退出当前作用域的语句
}
```

### 退出当前作用域
整体上来看guard所在的上下文可能是:

|上下文|退出|
|:-|:-|
|函数|return|
|循环|break 或 continue|
|全局|throw `os.abort()` `fatalError()` 等|

```swift
func f1() {
    guard false else {
        return                  // 结束当前函数
    }
}

func f2() {
    guard false else {
        fatalError()            // 直接结束程序
//        fatalError("自定义错误信息")
    }
}


func f3() {
    while true {
        guard false else {
//            break               // 结束当前loop
            continue              // 退出guard, 重新loop, 再进入guard
//            return              // 结束函数
        }
    }
}

func f4() throws {
    guard false else{
//        return                  // 结束当前函数
        throw CancellationError()   // 也相当于退出当前函数
    }
}



func f5() {
    do {
        guard false else {
//            return
            throw CancellationError()
        }
    }catch {

    }
}


do {
    guard false else {
//        return              // error, 在函数体外不能return
        throw CancellationError()       // 可以抛出异常结束, 由上层runtime函数(或main)来处理
        fatalError()         // 结束程序
    }
}
```




</font>

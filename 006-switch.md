<font size = 4>

# switch
### 基本语法
- case:
    - 不允许有大括号
    - case中至少要有一条语句(<font color = red>分号不可以算语句</font>).
    - 相同条件的case用逗号隔开
- break:
    - 可以不写break, 不会穿透. 
    - break算一条有效语句
- fallthrough:
    - 手动指定穿透
- default:
    - 一般情况下不可以省略, 除非条件已经列举完毕
- 其他:
    - 允许模式匹配[^ann-pattern]
    > PS: 即case中可以条件匹配, 不要求一定是字面常量

```swift
do {
    let number = 2
    switch number {
    case 2:                             // 后面不允许有大括号
        print("number is 2")            // 可以不写break, 但也不会穿透
    case 3:                             
        print("number is 3")            
    default:                            // 没有default会报错
        print("number is other")
    }
}


do {
    let winner = true
    switch winner {
    case true :
        print("winner")
    case false:                         // 因为列出了所有的情况, 所以不需要default
        print("losser")
    }
}


do {
    // swift中的标识符可以是任意的Unicode字符
    enum Month {
        case 一月份
        case 二月份
        case 三月份
        case 四月份
        case 五月份
        case 六月份
        case 七月份
        case 八月份
        case 九月份
        case 十月份
        case 十一月份
        case 十二月份
    }
    
    let now = Month.八月份

    switch now {
    case .一月份:           // 对于枚举来说,可以省略类型, 当然也可以全写(Month.一月份)
        fallthrough
    case .三月份:
        fallthrough
    case .五月份:
        fallthrough
    case .七月份:
        fallthrough
    case .八月份:
        fallthrough
    case .十月份:
        fallthrough
    case .十二月份:         // 可以分别列出31天的月份, 然后写上fallthrough
        print("有31天")

    case .四月份, .六月份, .九月份, .十一月份: // 也可以一个case, 后面的月份使用逗号隔开
        print("有30天")

    default:
        print("二月份有多少天看情况")
    }
}
```

### 字符串匹配
switch中的case可以指定字符串(<font color = red> 事实上模式匹配</font>). 这一点比c++中要强大

```swift
let name = "tierry"

switch name {
case "jerry":
    print("jerry")
case "tom":
    print("tom")
default:
    print("tierry")
}
```

因为字符串遵循了Equatable协议, 并实现了`==`运算, 在case进行对比时调用相关的方法得出比较的结果, 事实上模式匹配是以`~=`操作符为准

```swift
// 遵循协议后, 对struct来说可以不实现(则默认是bit比较)
struct string : Equatable{
    init(str: String) {
        self.str = str
    }

    // 实现Equatable(注意参数都是 string)
    // 对于case来说, 第1个参数表示case后面的对象, 第2个参数是switch后面的对象
    static func == (this : Self, other: Self) -> Bool {
        return true
    }

    // 模式匹配, 第1个参数表示case后面的对象(swift中的String).第2个参数的类型是switch后的对象
    static func ~= (this : String, other: Self) -> Bool {
        return other.str.hasSuffix(this)
    }
    let str: String
}

//let str = string(str: "tom")
let str = string(str: "tierry")
switch str {
case string(str: "tom"):        // 匹配 ==
    print("om")
case "rry" :                    // 匹配 ~=
    print("rry")
default:
    print("other")
} 
```


### c++中实现case字符串
在C++11以前case语句无法对字符串进行比较. 因为字符串比较的逻辑是运行时对比2字符串的相同位置的字符. 在C++11以后出现了`constexpr`关键字, 它修饰常量或函数, 主要功能体现在函数, 目的是告诉编译器函数的整个过程中函数内部的值全部是常量可以确定的, 举例来说:

```cpp
constexpr int test(const int a){
    a ^= a^100 + a << 31 + a >> 13
    return a;
}

auto r1 = test(2);        // 常量调用, 编译器会立即计算最后的结果, 假如是 0xff, 则当前位置直接被编译器替换成 auto r1 = static_cast<int>(0xff)

int n = 2;
auto r2 = test(n);          // 不是常量调用, 编译器会调用函数, 在运行时处理. 
```
通过constexpr函数, 可以为字符串字面量定义字符串到数字的转换, 这就是实现case字符串的原理. 下面是一个可行的案例. 用到了高级模板技巧
```cpp
namespace lb = std;

// 不同类型的hash

// 其他class或struct要自己加特化, 否则报错
template<typename T>size_t hash(T& val){    return val;     }

// int类型特化
template<> size_t hash<int>(int& val){  return val; }

// 指针类型特化
template<typename T> size_t hash(T* val){   return val; }

// const 指针特化
template<typename T> size_t hash(const T* val){ return val; }


// 声明成常量函数
constexpr static inline size_t cal_c_str(const char* val){
    size_t result = 0;
    auto ptr = const_cast<char*>(val);
    while(*ptr) result += *ptr++;
    return result;
}

// C字符串特化, 声明成constexpr 是为了case语句
template<> constexpr size_t hash(char* val){return cal_c_str(val);}
template<> constexpr size_t hash(const char* val){return cal_c_str(val);}



template<typename T>
constexpr void hash_combine(size_t& seed, const T& val){    //hash_algorithm
    seed ^= hash(val) + 0x9e3779b9 + (seed << 6) + (seed >> 2);
}
template<typename T>
constexpr void hash_val(size_t& seed, const T& val){    // hash_over
    hash_combine(seed, val);
}
template<typename T, typename... Types>
constexpr void hash_val(size_t& seed, const T& val, const Types&... args){  //hash_handle
    hash_combine(seed, val);
    hash_val(seed, args...);
}

template<typename... Types>
constexpr size_t hash_val(const Types&... args){    // hash_main
    size_t seed = 97;                               // 这个值随便取的, 可以规定一个比较大的质数      
    hash_val(seed, args...);
    return seed;
}


// c++14以上, 字符串字面量的后缀格式, 如: "hello"_code
// _code必须加`_`, 为了和标准库里的用法区分开
constexpr size_t operator "" _code(const char* val, size_t len) noexcept {
    return hash_val(val);
}


int main(int argc, const char * argv[]) {
    char abc[] = {'h', 'e', 'l', 'l', 'o', '\0'};
    char* tmp = abc;
    lb::cout << hash_val("hello") << lb::endl;
    lb::cout << hash_val(abc) << lb::endl;
    lb::cout << hash_val(tmp) << lb::endl;
    lb::cout << hash_val("world") << lb::endl;
    lb::cout << hash_val(lb::string("hello").c_str())<<lb::endl;
    lb::cout << hash_val(100)<<lb::endl;
    lb::cout << "hello"_code << lb::endl;

    switch (hash_val(lb::string("hello").c_str())) {
        case "hello"_code:
            lb::cout << "hello" << lb::endl;
            break;

        default:
            break;
    }
    return 0;
}
```

### tuple与case
同样的case可以接收tuple

```swift
let point = (x: 20.0, y: 10.0)

switch point {
case (0, 0):
    print("原点")
case (0, _):
    print("忽略y, x = 0时匹配")
case (_, 0):
    print("忽略x, y = 0时匹配")
case (0...50, 0...50):
    print("在矩形范围内")
default:
    print("其他")
}
```

### 区间与case
前面学习的区间有成员方法contain和`~=`, 所以case后面也可以是区间
```swift
switch 10{

case ..<20:                     // 实际上调用了 ~=方法, 第1个参数就是case后面该对象, 第2个参数就是switch后面的对象
    print("小于20")
case 20...:
    print("大于等于20")
default:
    print("其他")
}
```

### case值绑定
这个是固定的语法, 与模式匹配没有关系. swift中可以在case后面声明对应类型对象, 可以获取到switch后面对象的值, 配合where实现条件匹配
```swift
do {
    let n = 20
    switch n  {

    case let tmp where tmp > 50:        // 声明对象tmp, 在运行时将被赋值于n, 并且tmp大于50时,才进入到case中
        print("大于50")
    default:
        print("小于等于50")
    }
}


do {
    let xy: (x: Double, y: Double)
    xy.x = 20
    xy.y = -20

    switch xy {
    case (0,0):
        print("原点")

        // 可以对每个tuple成员指定let或var
    case (let x, let y) where x == y:           // 当x == y时来这里 
        print("x == y")

    case (let x, _) where x > 0 && x < 100:     // 忽略y的值, 若x在这个范围则来之里, 所以xy这个对象是符合该条件的
        print("0 < x < 100")

        
        // 可以在最前面使用let或var, 则后面所有的成员就是统一的
    case let(x, y) where x == -y:               // xy对象也符合该条件, 但最后结果是上面那个, 取决于编译器先遇到哪个
        print("x == -y")

    default:
        print("other")
    }
}
```

### where
where是swift中条件判断的关键字, 可以用在任何条件判断相关的地方
1. for
2. case

```swift
for i in 5...10 where i > 7 {           // 这里的where判断条件不成立时, 不会结束循环而是继续循环
    print(i)
} 


let score = 60
switch score {
case let n:                             // 绑定
    if(n < 60){
        print("不及格")
    }

case 60... :                            // 永远不会来这里
    print("及格")

default:
    break
}
// 上述的switch-case不符合逻辑, 使用了n绑定后所有的情况都会汇集到第1个case中, 
// 然后在里面判断少了及格, 所以即使score是60分也执行不到第2个case
// 正确的做法是使用where

switch score {
case let n where n < 60:                // 绑定
    print("不及格")
case 60... :                            
    print("及格")
default:
    break
}
```

[^ann-pattern]: swift中强大的功能, 后面学习

</font>

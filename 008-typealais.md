<font size = 4>

# typealias
它类似于c++中的typedef或using
```swift
typealias Byte = Int8;
typealias Short = Int16;
typealias Long = Int64;

typealias Pair = (key :Any, val: Any)
let pair = Pair(key:"id", val:"120");
print(pair.0);
print(pair.key);
print(pair.1);
print(pair.val);


typealias Fuc = (Int, Int) -> Int;
func test(_ v1 :Int, v2 :Int) -> Int{
    v1 + v2;
}
let myfuc : Fuc;
myfuc = test;
myfuc(20,100); 
```

</font>

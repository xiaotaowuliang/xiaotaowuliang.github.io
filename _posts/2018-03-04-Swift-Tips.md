---
title: "Swift 的 Map、Filter、Reduce等高阶函数，以及$的含义"
header:
  image: /assets/images/unsplash-image-5.jpg
categories:
  - Swift Tips
tags:
  - Swift
---

##### 引言

> 从OC转到Swift已经有接近半年了，但是很多地方都是用的最笨最基础的写法来完成，虽然代码没问题，功能能实现，不过对于想提升自己的你来说，还是得找个途径来优化自己的代码，自己维护也轻松，别人看着也舒适。正所谓不积跬步无以至千里，自己也正在一点一滴的积累，在网上看着大神们的写法，敲一遍不代码记得住，所以写篇文章来记录知识。



在说高阶函数之前，我们需要理解一个符号： __$__

>swift 自动为**闭包**提供参数名缩写功能，可以直接通过```$0```和```$1```，来分别表示闭包中的 __第一个参数和第二个参数__ ，并且对应的参数类型会根据类型来进行判断。

- 不使用```$0```和```$1```这些来代替
```
let numbers = [1,2,5,4,3,6,8,7]
sortNumbers = numbers.sorted(by: { (a, b) -> Bool in
    return a > b
})
print(sortNumbers)  // [8,7,6,5,4,3,2,1]
```

- 使用```$0```和```$1```
```
let numbers = [1,2,5,4,3,6,8,7]
let sortNumber = numbers.sorted(by: {$0 < $1})
print(sortNumber)   // [1,2,3,4,5,6,7,8]
```
可以发现，使用```$0```和```$1```的话，参数类型可以自动判断，并且关键字```in```也可以省略，也就是 __只用写函数体__ 就可以了
***


## Map
>map函数能够被数组调用，它接受一个闭包作为参数，作用于数组中的每个元素。闭包返回一个变换后的元素，接着将所有这些变换后的元素组成一个新的数组
>
>简单来说，就是把数组里面的值给替换了，不用写for循环

##### 1. 比如，现在有一个Int类型的数组，需要将数组内的所有数字与自身相加
```
// 常规写法
let numbers = [1,2,3]
var sumNumbers:Array<Int> = Array() 
for var number in numbers {
    number += number
    sumNumbers.append(number)
}
print(sumNumbers) // [2,4,6]
```
```
// 通过map简化后的写法

// 1.可以看到我们甚至可以不再定义可变的数组直接用不可变的就可以
let sumNumber1 = numbers.map { (number) -> Int in
    return number + number
}
// 2.可以省了return，提出一个问题，在下方解释
let sumNumbers3 = numbers.map { number in number + number }
print(sumNumbers3)

// 3.最终简化写法，可以省略return
let sumNumber4 = numbers.map { $0 + $0 }
print(sumNumber4)  // [2,4,6]
```
这里有个小问题，就是发现在Playground里面执行，1写的法是有return的执行了3次，而2写法没有return却执行了4次，很多人理解为循环了3次或是4次，实则不然，这里的意思为本行代码执行了多少次，不信你可以在2写法的```in```后的代码开始换行，换为正常格式，一目了然

#### 2. Map函数返回数组的元素类型，不一定要与原来数组相同
```
let fruits = ["apple", "banana", "orange", ""]
// 这里数组中存在一个""的字符串 为了后面来比较 map 和 flatMap
let counts = fruits.map { (fruit) -> Int? in
    let strCount = fruit.count
    guard strCount > 0 else {
        return nil
    }
    return strCount
}
print(counts) // [Optional(5), Optional(6), Optional(6), nil]
```
#### 3. Map还能返回判断数组中的元素是否满足某种条件的Bool值数组
```
let array = [1,2,3,4,5,6]
let isEven = array.map { $0 % 2 == 0 }
print(isEven)  // [false, true, false, true, false, true]
```
***
### FlatMap
>flatMap 与 map 不同之处：
>1. flatMap返回后的数组中不存在 nil 同时它会把Optional解包；
>2. flatMap 还能把数组中存有数组的数组 一同打开变成一个新的数组；
>3. flatMap也能把两个不同的数组合并成一个数组 这个合并的数组元素个数是前面两个数组元素个数的乘积。

#### 1. flatMap返回后的数组中不存在nil 同时它会把Optional解包
```
let fruits = ["apple", "banana", "orange", ""]  // 同上面的数组
let counts = fruits. flatMap { (fruit) -> Int? in
    let strCount = fruit.count
    guard strCount > 0 else {
        return nil
    }
    return strCount
}
print(counts) // [5,6,6]
```

#### 2. flatMap 能把数组中存有数组的数组 一同打开变成一个新的数组，看代码秒懂
```
let array = [[1,2,3], [4,5,6], [7,8,9]]

// 如果用map来获取新的数组
let arrayMap = array.map { $0 }
print(arrayMap)  // [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

// 用flatMap
let arrayFlatMap = array.flatMap { $0 }
print(arrayFlatMap)   // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```
#### 3. flatMap也能把两个不同的数组合并成一个数组 这个合并的数组元素个数是前面两个数组元素个数的乘积
```
// 这种情况是把两个不同的数组合并成一个数组 这个合并的数组元素个数是前面两个数组元素个数的乘积
let fruits = ["apple", "banana", "orange"]
let counts = [1, 2, 3]
let fruitCounts = counts.flatMap { count in
    fruits.map { fruit in
        // title + "\(index)"
        // 也可以返回元组
        (fruit, count)
    }
} 
print(fruitCounts) // [("apple", 1), ("banana", 1), ("orange", 1), ("apple", 2), ("banana", 2), ("orange", 2), ("apple", 3), ("banana", 3), ("orange", 3)]
// 这种方法估计用的很少 可以算是一个 flatMap 和 map 的结合使用吧
```
***
### Filter
>filter 可以取出数组中符合条件的元素 重新组成一个新的数组，可以理解为过滤

##### filter可以很好的帮我们把数组中不需要的值都去掉 这个很赞
```
let numbers = [1,2,3,4,5,6]
let evens = numbers.filter { $0 % 2 == 0 }
print(evens)  // [2, 4, 6]
```
##### 取出数组中大于22的值
```
// filter 筛选数组中大于22的元素，返回值为新数组
let intArr2 = [21,23,25]
let filtereArr = intArr2.filter{ $0 > 22 }
print(filtereArr)
```
***
### Reduce
>map,flatMap和filter方法都是通过一个已存在的数组，生成一个新的、经过修改的数组。然而有时候我们需要把所有元素的值合并成一个新的值 那么就用到了Reduce

#### 1. 比如我们要获得一个数组中所有元素的和
```
let numbers = [1,2,3,4,5]
// reduce 函数第一个参数是设定的初始化值，这里设置为0
let sum = numbers.reduce(0, { $0 + $1 })
// 这里我写下完整的格式
let sum1 = numbers.reduce(0) { total, num in
    total + num
}
// 这里还能再次简写
let sum = numbers.reduce(0, +)
print(sum)  // 15
```

#### 2. 合并成的新值不一定跟原数组中元素的类型相同
```
let numbers = [1,5,1,8,8,8,8,8,8,8,8]
// reduce 函数第一个参数是返回值的初始化值
let tel = numbers.reduce("", { "\($0)" + "\($1)" }) 
print(tel)  // 15188888888
```
#### 3. ruduce 还可以实现 map 和 filter 并且时间复杂度，从O(n*n)变为O(n) 
```
extension Array {
    func mMap<U> (transform: Element -> U) -> [U] {
        return reduce([], combine: { $0 + [transform($1)] })
    }
    func mFilter (includeElement: Element -> Bool) -> [Element] {
        return reduce([]) { includeElement($1) ? $0 + [$1] : $0 }
    }
}
```
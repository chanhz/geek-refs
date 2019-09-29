# swift 入门

swift 是 Apple Inc 于 2014 年推出的苹果系统开发语言，用于开发苹果操作系统生态圈的应用。

## 一些资料
- [人人能编程](https://www.apple.com/cn/everyone-can-code/)
- [使用 Swift 开发 App 入门课程](https://www.apple.com/105/media/cn/education/everyone-can-code/2017/bb05cf6f_6591_4502_ba4a_e84e14c35c01/download-swift/intro-to-app-development-with-swift.ibooks)
- [官方swift入门教程·playground·中文版](https://developer.apple.com/sample-code/swift/downloads/intro-app-dev-curriculum-cn.zip)
- [社区中文文档](https://swiftgg.gitbook.io/swift/) 


## 开始

### 1. 变量命名
swift 使用 `let` 关键字来初始化一个变量
```swift
let balance = 1000
```

### 2. 字符串插值
**要点：**
- swift 字符串和 golang 很像
- 插值使用`\(变量表达式)`的方式插入到字符串中
```swift
let goalieName = "艾莉森"
let firstHalfSaves = 3
let secondHalfSaves = 6
let overtimeSaves = 2
let goalieReportString = "At the game yesterday, \(goalieName) had \(firstHalfSaves) saves in the first half, \(secondHalfSaves) in the second half and \(overtimeSaves) saves in overtime, for a total of \(firstHalfSaves + secondHalfSaves + overtimeSaves) saves."
print(goalieReportString)
```
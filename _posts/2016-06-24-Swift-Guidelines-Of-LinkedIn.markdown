---
layout:     post
title:      "LinkedIn官方Swift编码风格(译)"
date:       2016-06-24
author:     "Nov.柒月"
catalog: true
tags:
    - Swift style
---

# 代码格式

1.1 tabs键的空间应该是4个空格的空间

1.2 避免每行代码的长度过长，最好限制在160个字符内（Xcode->Preference->Text Editing->Page guide at column:160）

1.3 确保每个文件的结束都起了一个新行

1.4 确保所有地方都没有多余的空格（Xcode->Preference->Text Editing->Automatically trim trailing whitespace + Including whitespace-only lines)

1.5 不要在新行使用大括号

```swift
class SomeClass {
  func someMethod() {
    if x == y {
      /*...*/
    } else if x == z {
      /*...*/
    } else {
      /*...*/
    }
  }
  /*...*/
}
```

1.6 在声明变量的类型，字典的键，函数参数，协议或者是父类的时候，不要在冒号之前添加空格

```swift
// 指定类型
let pirateViewController: PirateViewController

// 字典语法
let ninjaDictionary: [String: AnyObject] = [
  "fightLikeDairyFarmer": false,
  "disgusting": true
]

// 声明函数
func myFunction<T, U: SomeProtocol where T.RelatedType == U>(firstArgument: U, secondArgument: T) {
  /*...*/
}

// 函数调用
someFunction(someArgument: "Kitten");

// 父类
class PirateViewController: UIViewController {
  /*...*/
}

// 协议
extension PirateViewController: UITableViewDataSource {
  /*...*/
}
```

1.7 通常情况下，逗号后面都会跟着一个空格符

```swift
let myArray = [1, 2, 3, 4, 5]
```

1.8 在类似`+`, `==`或者`->`等二元操作符的前后应该空一格，在`(`之后或者`)`之前都不应该出现空格

```swift
let myValue = 20 + (30 / 2) * 3
if 1 + 1 == 3 {
  fatalError("The universe is broken")
}

func pancake() -> Pancake {
  /*...*/
}
```

1.9 采用Xcode自带的缩进风格

```swift
// 声明函数时Xcode的缩进会将声明分割为数行
func myFunctionWithManyParameters(parameterOne: String,
                                  parameterTwo: String,
                                  parameterThree: String) {
  print("\(parameterOne) \(parameterTwo) \(parameterThree)")
}

// if语句的缩进
if myFirstVariable > (mySecondVariable + myThirdVaribale)
    && myFourthVariable == .SomeEnumValue {
      print("Hello, world!")
}
```

1.10 在调用带多个参数的函数时，每个参数都独立成行

```swift
someFunctionWithManyArguments (
  firstArgument: "Hello, I am a string",
  secondArgument: resultFromSomeFunction()
  thirdArgument: someOtherLocalVariable)
```

1.11 在处理较大的隐式数组或者字典时，可以将`[]`看成函数中的`()`一样，以便将数组或者字典的内容拆分成多行。方法中的闭包也应采用这种做法

```swift
someFunctionWithABunchOfArguments (
  someStringArgument: "Hello I am a string",
  someArrayArgument: [
    "dada",
    "string one"
  ],
  someDictionaryArgument: [
    "dictionary key 1": "value 1",
    "dictioanry key 2": "value 2"
  ],
  someClosure: { parameter in
    print(parameter)
  }
)
```

1.12 使用本地变量或者是其他技巧来尽可能避免多行谓词

```swift
// PREFERRED
let firstCondition = x == firstReallyReallyLongPredicateFunction()
let secondCondition = y == secondReallyReallyLongPredicateFunction()
let thirdCondition = z == thirdReallyReallyLongPredicateFunction()
if firstCondition && secondCondition && thirdCondition {
  // do something
}

// NOT PREFERRED
if x == firstReallyReallyLongPredicateFunction()
  && y == secondReallyReallyLongPredicateFunction()
  && z == thirdReallyReallyLongPredicateFunction() {
    // do something
  }
```

# 命名

2.1 在Swift中没必要使用OC的前缀风格(比方说使用GuybrushThreepwood替代LIGuybrushThreepwood)

2.2 对类型（struct, class, enum, typedef, associatedtype），使用Pascal命名法（每一个单字的首字母都采用大写字母的命名格式）

2.3 对于函数，方法，变量，常量，参数等使用驼峰命名法

2.4 遇到在其他地方为全部大写的名称或者是首字母缩写情况时，如果该词组是在开头处的话，并且是需要以小写字母开头的话，则该词组全部字符需要写成小写

```swift
// ‘HTML’出现在变量名的开头，所以使用小写的html
let htmlBodyContent = "<p>Hello</p>";

// 使用ID不建议使用Id
let profileID = 1

// 使用URLFinder不建议使用UrlFinder
class URLFinder {
  /*...*/
}
```

2.5 使用`k` + Pascal命名法来命名静态常量, 但是不要用在单例上

```swift
class MyClassName {
  static let kSomeConstantHeight: CGFloat = 80.0
  static let kDeleteButtonColor: UIColor.redColor()

  static let sharedInstance = MyClassName()
}
```

2.6 对泛型和关联类型来说，使用一个大写字母或者是Pascal命名法命名的单词来描述即可。如果该单词破坏了所遵循的协议或者是所继承的父类的话，可以添加`Type`作为后缀

```swift
class SomeClass<T> { /*...*/ }
class SomeClass<Model> { /*...*/ }
protocol Modelable {
  associatedtype Model
}
protocol Sequence {
  associatedtype IteratorType: Iterator
}
```

2.7 名称需要是明确的，且带有描述性质的

```swift
// 使用RoundAnimatingButton会更好
class RoundAnimatingButton: UIButton { /*...*/ }
class CustomButton: UIButton { /*...*/ }
```

2.8 不要使用缩写，简称或者是单个字母的名称

```swift
// 推荐使用
class RoundAnimatingButton: UIButton {
  let animationDuration: NSTimeInterval
  func startAnimating() {
    let firstSubview = subviews.first
  }
}

// 不推荐使用
class RoundAnimating: UIButton {
  let aniDur: NSTimeInterval
  func srtAnimating() {
    let v = subviews.first
  }
}
```

2.9 如果看不出对象的类型时，可以在命名时加上类型信息

```swift
// 推荐使用
class ConnectionTableViewCell: UITableViewCell {
  let personImageView: UIImageView

  // 这里不带string的话也没什么问题，因为很明显能看出该变量是个string类型的
  let firstName: String

  // 虽然不推荐，但是使用`Controller`来替代`ViewController`是没问题的
  let popupController: UIViewController
  let popupViewController: UIViewController

  // 在使用如table view controller, collection view controller等view controller时，最好在命名中标识出来
  let popupTableViewController: UITableViewController

  // 使用outlets时，最好在变量名中指出outlet的类型
  @IBOutlet weak var submitButton: UIButton!
  @IBOutlet weak var emailTextField: UITextField!
  @IBOutlet weak var nameLabel: UILabel!
}

// 不推荐使用
class ConnectionTableViewCell: UITableViewCell {
  // 这并不是UIImage,所以不能命名为image
  let personImage: UIImageView

  // 这不是`String`，所以可以用`textLabel`
  let text: UILabel

  // 命名为animation的话，表达不出这是表示时间的变量，所以使用`animationDuration`或者`animationTimeInterval`代替
  let animation: NSTimeInterval

  // 并不是很明显的`String`类型
  // 用`transitionText`或者是`transitionString`代替
  let transition: String

  // 这是一个view controller, 不建议使用一个view
  let popupView: UIViewController

  // 跟之前提到的一样，不要使用缩写
  let popupVC: UIViewController

  // 虽然这是一个UIViewController，但是需要指出我们使用的是一个*Table*
  let popupViewController: UITableViewController

  // 将类型放在结尾不建议使用开头
  @IBOutlet weak var btnSubmit: UIButton!
  @IBOutlet weak var buttonSubmit: UIButton!

  // Outlet的话加上类型
  @IBOutlet weak var firstName: UILabel!
}
```

2.10 给函数参数命名时，确保能够让人读懂函数中每个参数的意义

2.11 根据苹果的API Design Guidelines，每个protocol的命名应该是个能描述其行为的名词，或者是使用`able`,`ible`或者`ing`描述能力的后缀。如果这些条件都不满足的话，也可以在名称后面直接添加`protocol`

```Swift
// 描述protocol行为的名词
protocol TableViewSectionProvider {
  func rowHeight(atRow row: Int) -> CGFloat
  var numberOfRows: Int { get }
  /*...*/
}

// 描述protocol的能力
protocol Loggable {
  func logCurrentState()
}

protocol InputTextViewProtocol {
  func sendTrackingEvent()
  func inputText() -> String
  /*...*/
}
```

# 编码风格

## 基础

3.1.1 在合适的地方使用let, 不建议使用都使用var

3.1.2 使用`map`, `filter`, `reduce`等来转换数组等集合。但是要注意的是不要在这些方法中使用闭包

```swift
// 推荐使用的
let stringOfInts = [1, 2, 3].flatMap { String($0) }
// ["1", "2", "3"]

let evenNumbers = [4, 8, 15, 16, 23, 42].filter { $0 % 2 == 0 }
// [4, 8, 16, 42]

// 不推荐使用
var stringOfInts: [String] = []
for integer in [1, 2, 3] {
  stringOfInts.append(String(integer))
}

var evenNumbers: [Int] = []
for integer in [4, 8, 15, 16, 23, 42] {
  if integer % 2 == 0 {
    evenNumbers(integer)
  }
}
```

3.1.3 如果某个变量或者常量可以被推断出来类型的话，可以不需要声明类型

3.1.4 如果某个函数返回多个不同的值的话，可以返回一个使用`inout`修饰的元组。如果多次使用一个确定类型的元组的话，使用`typealias`来添加类型别名；如果在一个元组中返回3个以上的item时，考虑使用`struct`或者是`class`

```swift
func pirateName() -> (firstName: String, lastName: String) {
  return ("Guybrush", "Threepwood")
}

let name = pirateName()
let firstName = name.firstName
let lastName = name.lastName
```

3.1.5 在创建delegate/protocol时注意循环引用，这些属性应该声明为weak

3.1.6 在闭包中调用self会造成循环，可以使用捕获列表来避免这种情况

```swift
myFunctionWithClosure() { [weak self](error) -> Void in
  self?.doSomething()

  // 或者
  guard let strongSelf = self else {
    return
  }
  strongSelf.doSomething()
}
```

3.1.7 不要使用标签来进行break

3.1.8 不要在控制流中使用括号

```swift
// 推荐使用
if x == y {
  /*...*/
}

// 不建议使用
if (x == y) {
  /*...*/
}
```

3.1.9 避免写出枚举的类型

```swift
// 使用
imageView.setImageWithURL(url, type: .person)
// 不建议使用
imageView.setImageWithURL(url, type: AsyncImageView.Type.person)
```

3.1.10 如果没办法从类方法中推断出上下文的话，那么就不要缩写类方法

```swift
// 使用
imageView.backgroundColor = UIColor.whiteColor()
// 不建议使用
imageView.backgroundColor = .whiteColor()
```

3.1.11 在有必要加上`self.`的时候才加上

3.1.12 在编写函数方法时，要考虑好该函数是否会被重写。如果不需要被重写的话，使用`final`来修饰以避免被修改。因为`final`方法可以减少编译时间，所以适当使用`final`可以加快编译速度。

3.1.13 在使用`else`或者`catch`等时，把这些和处理模块放在同一行中

```swift
if someBoolean {
  // do something
} else {
  // do something else
}

do {
  let fileContents = try readFile("filename.txt")
} catch {
  print(error)
}
```

## 访问修饰符

3.2.1 如果需要访问修饰符的话，将其放在最前

```swift
// 使用
private static let kMyPrivateNumber: Int
// 不建议使用
static private let kMyPrivateNumber: Int
```

3.2.2 访问修饰符不应该独立成行

3.2.3 默认情况下，不要添加`internal`。因为`internal`就是默认修饰，不需要加上

3.2.4 如果某个变量需要在单元测试中用到，可以使用`@testable import ModuleName`来将其设置为`internal`。如果变量是私密类型的，但是在单元测试中又被声明为`internal`，最好就为其添加文件说明.

```swift
/**
  This variable defines the pirate's name
  - warning: Not 'private' for '@testable'
*/
let pirateName = "LeChuck"
```

## 自定义操作符

选择创建方法会比自定义操作符更好。

如果你想要使用一个自定义的操作符的话，一般都是要有一个比较重要的原因来使你做出这个决定。

你可以重写已经存在的操作符来满足需求。

## Switch语句和枚举

3.4.1 对一个有限集合使用switch语句的话，不要添加上`default`.取而代之的是，在底部添加上不需使用的数据并且使用`break`来结束语句

3.4.2 因为swift中的switch语句默认是带上break的，所以如果没必要的话就不要添加上`break`了

3.4.3 case语句应该与switch语句对齐

3.4.4 如果哪个case有个关联值的话，确保其值相对于类型来说有个合适的标签值

```swift
enum Problem {
  case attitude
  case hair
  case hunger(hungerLevel: Int)
}

func handleProblem(problem: Problem) {
  switch problem {
  case .attitude:
      print("At least I don't have a hair problem")
  case .hair:
      print("Your barber didn't know when to stop")
  case .hunger(let hungerLevel):
      print("The hunger level is \(hungerLevel)")
  }
}
```

3.4.5 尽可能使用列表里(e.g: case 1, 2, 3:)不建议使用使用`fallthrough`

3.4.6 如果default条件永远也不会满足到的话，可以添加异常语句

```swift
func handleDigit(digit: Int) throws {
  case 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 :
    print("Yes, \(digit) is a digit")
  default:
    throw Error(message: "The given number was not a digit")
}
```

## Optionals

3.5.1 只有在`@IBOutlet`时，才能使用隐式解包的可选类型。在其他情况下，最好使用非可选类型的或者是正常的可选类型值。虽然在某些情况下确实能够确保该值在使用时非空，但是这也是为了保证安全性和一致性

3.5.2 不要使用`as!`或者`try!`

3.5.3 如果没有想好某个值是不是要用可选类型，但是却需要决定该值是否可能为nil的话，将其与nil进行比较会比使用`if let`来判断更好

```swift
// 使用
if someOptional != nil {
  // do something
}

// 不建议使用
if let _ = someOptional {
  // do something
}
```

3.5.4 不要使用`unowned`. 在隐式解包中，可以认为`unowned`等同于一个`weak`变量（虽然`unowned`在处理引用计数方面有点用）。在不需要隐式解包的情况下，也不需要使用`unowned`

```swift
// 使用
weak var parentViewController: UIViewController?

// 不建议使用
wear var parentViewController: UIViewController!
unowned var parentViewController: UIViewController
```

3.5.5 解包时使用相同名称

```swift
guard let myVariable = myVariable else {
  return
}
```

## Protocols

在实现protocols时，代码应该包含下面两个部分

  1. 使用`//MARK:`来分离代码

  2. 使用extension来与原来的代码进行分离


但是，需要记住在extension中的方法不能被子类重写。

## Properties

3.7.1 创建一个只读的计算属性时，可以省略`get {}`来提供getter方法

```swift
var completedProperty: String {
  if someBool {
    return "I'm a mighty pirate!"
  }
  return "I'm selling these fine leather jackets"
}
```

3.7.2 在使用`get {}`, `set {}`, `willSet`和`didSet`时，缩进这些模块

3.7.3 虽然可以在`willSet`, `didSet`和`set`中创建自定义的名称来表示新旧值，但是一般都使用默认的命名即可

```swift
var completedProperty: String {
  get {
    if someBool {
      return "I'm a mighty pirate!"
    }
    return "I'm selling these fine leather jackets"
  }
  set {
    completedProperty = newValue
  }
  willSet {
    print("will set to \(newValue)")
  }
  didSet {
    print("did set from \(oldValue) to \(newValue)")
  }
}
```

3.7.4 创建类常量时使用`static`

```swift
class MyTableViewCell: UITableViewCell {
  static let kReuseIdentifier = String(MyTableViewCell)
  static let kCellHeight: CGFloat = 80.0
}
```

3.7.5 单例

```swift
class PirateManager {
  static let sharedInstance = PirateManager()
}
/*...*/
```

## 闭包

3.8.1 如果参数的类型比较明显，可以直接省略类型名。

```swift
// 省略类型
doSomethingWithClosure() { response in
  print(response)
}

// 保留类型
doSomethingWithClosure() { response: NSURLResponse in
  print(response)
}

// map语句中使用简写
[1, 2, 3].flatMap { String($0) }
```

3.8.2 在参数列表中，如果使用捕获列表且/或者指定返回的类型，则可以将参数列表封装起来。

```swift
// 捕获列表
doSomethingWithClosure() { [weak self] (response: NSURLResponse) in
  self?.handleResponse(response)
}

// 返回类型
doSomethingWithClosure() { (response: NSURLResponse) -> String in
  return String(response)
}
```

3.8.3 如果将闭包作为类型值的话，则没必要将其封装进括号中, 除非有这个必要

```swift
let completionBlock: (success: Bool) -> Void = {
  print("Success? \(success)")
}

let completionBlock: () -> Void = {
  print("Completed!)")
}

let completionBlock: (() -> Void)? = nil
```

3.8.4 让参数名在写在同一行中

3.8.5 使用尾随闭包的写法

```swift
// 尾随闭包
doSomething(1.0) { parameter in
  print("Parameter 1 is \(parameter)")
}

// 不使用尾随闭包
doSomething(1.0, success: { parameter in
  print("Success with \(parameter)")
}, failure: { parameter in
  print("Failure with \(parameter)")
})
```

## 数组

3.9.1 避免直接使用下标来访问数组。尽可能使用`.first`或者`.last`等可选且不会崩溃的的访问方法。使用`for item in items`的语法来遍历，不建议使用使用`for i in 0..<items.count`

3.9.2 不要使用`+=`或者`+`来的操作符来添加或者集合数组。使用`.append()`或者`.appendContentsOf()`方法进行替代。如果你声明了一个基于其他数组的数组，并且想保持不可变状态的话，使用`let myArray = [arr1, arr2].flatten()`来替代`let myArray = arr1 + arr2`

## 错误处理

假定`myFunction`返回一个`String`对象，且有可能会出现错误。此时应该返回`String?`来表示发生错误时返回nil

```swift
func readFile(withFileName filename: String) -> String? {
  guard let file == openFile(filename) else {
    return nil
  }
  let fileContents = file.read()
  file.close()
  return fileContents
}

func printSomeFile() {
  let filename = "somefile.txt"
  guard let fileContents = readFile(filename) else {
    print("Unable to open file \(filename)")
    return
  }
  print(fileContents)
}
```

如果在发生错误时想要获取失败的原因的话，使用`try/catch`

```swift
struct Error: ErrorType {
  public let file: StaticString
  public let function: StaticString
  public let line: UInt
  public let message: String

  public init(message: String, file: StaticString = #file, function: StaticString = #function, line: UInt = #line) {
    self.file = file
    self.function = function
    self.line = line
    self.message = message
  }
}
```

e.g

```swift
func readFile(withFileName filename: String) throws -> String? {
  guard let file == openFile(filename) else {
    throw Error(message: "Unable to open file name \(filename).")
  }
  let fileContents = file.read()
  file.close()
  return fileContents
}

func printSomeFile() {
  do {
    let fileContents = try readFile(filename)
    print(fileContents)
  } catch {
    print(error)
  }
}
```

## 使用`guard`

3.11.1 使用`guard`可以进行提前返回，并且提高了代码的可阅读性

```swift
// 使用
func eatDoughnut(atIndex index: Int) {
  guard index >= 0 && index < doughnuts else {
    // return early because the index is out of bounds
    return
  }

  let doughnut = doughnuts[index]
  eat(doughnut)
}

// 不建议使用使用
func eatDoughnut(atIndex index: Int) {
  if index >= 0 && index < doughnuts.count {
    let doughnut = doughnuts[index]
    eat(doughnut)
  }
}
```

3.11.2 在可选类型的解包过程中，使用`guard`不建议使用`if`,这样做可以减少嵌套数量

```swift
// 使用
guard let monkeyIsland = monkeyIsland else {
  return
}
bookVacation(onIsland: monkeyIsland)
bragAboutVacation(onIsland: monkeyIsland)

// 不建议使用使用
if let monkeyIsland = monkeyIsland {
  bookVacation(onIsland: monkeyIsland)
  bragAboutVacation(onIsland: monkeyIsland)
}

// 更加不建议使用
if monkeyIsland == nil {
  return
}
bookVacation(onIsland: monkeyIsland)
bragAboutVacation(onIsland: monkeyIsland)
```

3.11.3 如果使用`if`和`guard`看起来没多大差别的话，这时候就要考虑代码的可读性来进行选择。如果考虑上代码的可读性都没法确定的话，那就是用guard

```swift
// if的可读性更高
if operationFailed {
  return
}

// guard的可读性更高
guard isSuccessful else {
  return
}
```

3.11.4 如果出现两个状态的选择的话，用if会比`guard`更好

3.11.5 如果出现哪个错误导致需要离开当前的上下文时，应该使用guard.下面的代码使用`if`来替代`guard`,这是两个判断条件互不相关，没必要使用`guard`来离开上下文

```swift
if let monkeyIsland = monkeyIsland {
  bookVacation(onIsland: monkeyIsland)
}

if let woodchuck = woodchuck where canChuckwood(woodchuck) {
  woodchuck.chuckWood()
}
```

3.11.6 `guard`的经常用法是解包多个可选类型值

```swift
guard let thingOne = thingOne,
    let thingTwo = thingTwo,
    let thingThree = thingThree else {
    return
}
```

3.11.7 不要在同一行中完成整个`guard`语句

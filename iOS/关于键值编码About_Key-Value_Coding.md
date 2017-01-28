#关于键值编码（Key-Value Coding）
键值编码是`NSKeyValueCoding`协议提供的一套机制，它为遵从这个协议的对象提供了一种间接访问属性的方式。当一个对象遵从键值编码时，可以对一套简介规整的接口发送字符串消息来访问属性。这套间接访问的机制补充了通过实例变量直接访问属性或通过其相关的 *存取方法* 来访问两种方式的不足。  

你一般会使用存取方法去访问一个对象的属性。一个获得方法（getter方法）返回这个属性的值。一个储存方法（setter方法）设置这个属性的值。在Objective-C中，你也可以直接访问这些属性背后的实例变量。用上面提到的这些方法去访问属性是很方便的，但是需要使用一些为实例变量定制的方法或变量名。随着属性列表的增多或改变，关系着这些变量的代码也会增多和改变。而遵从键值编码机制的属性会提供一个对所有属性统一的简明访问接口。  

键值编码是构成其他许多Cocoa技术的基础概念之一，比如说KVO（key-value observing），Cocoa绑定（Cocoa bindings），Core Data，以及AppleScript-ability。同样的情况下，键值编码能够精简你的代码。  

##使用遵从键值编码的对象
继承（直接或间接）自`NSObject`的对象一贯会遵从键值编码。他们不仅遵守了`NSKeyValueCoding`协议，并且提供了相关方法的默认实现。「」

- **访问对象属性** 协议会规定一些方法，比如说通用的读取方法（getter）`valueForKey:`和通用的储存方法（setter）`setValue:forKey`。这些函数接受一个字符串参数，字符串内容为属性名或关键字（或者说键/Key）。关于这些方法的默认实现和相关方法如何使用键去定位一个数据并与其交互的详细内容，请阅读[Accessing Object Properties](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueCoding/BasicPrinciples.html#//apple_ref/doc/uid/20002170-BAJEAIEE)。
- **操作集合属性** 对于对象集合属性（比如说`NSArray`)，存取方法的实现方式和其他对象差不多。另外，如果一个对象为一个属性定义了集合存取方法，对于这个集合的内容，我们也可以应用键值编码存取。这一般比直接访问更加便利，并且允许你通过一个标准化的接口来使用自定义的集合对象，像在[Accessing Collection Properties](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107i)描述的那样。
- **在集合对象上调用集合操作符** 当你访问一个遵从键值编码规则的对象中的一个集合属性时，你可以在键字符串中插入一个 *集合操作符（collection operator）*，详情阅读[Using Collection Operators](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueCoding/CollectionOperators.html#//apple_ref/doc/uid/20002176-BAJEAIEE)。集合操作符委托默认的 `NSKeyValueCoding`getter方法实现去对这个集合做点什么，并且可以返回一个整理过滤好的集合版本或者这个属性的一个特征。
- **访问非对象属性** 协议默认这样实现：为了遵从协议的接口，它检测包括纯量和结构体在内的非对象属性，自动地把这些非对象属性封包成对象并在必要的时候拆包（译者注：类似Java的自动装箱），如同[Representing Non-Object Values]()描述的那样。此外，有时会面临程序通过键值编码接口设置了一个`nil`值给非对象属性的情况，为此协议还声明了一个方法，这个方法允许遵从协议的对象面临这种情况时做出合适的行为。
- **通过键路径访问属性** 「当你已知遵从协议的对象的层次结构的时候，有了键路径，你就可以仅通过一次方法调用就能设置或获得层次结构深处的值。」
 
 ##让一个对象采用键值编码
 为了让你自己的对象采用键值编码，你需要确保这些对象已经遵守了`NSKeyValueCoding`非正式协议并实现了其相关的方法，比如通用的getter`valueForKey:`和通用的setter`setValue:forKey`。幸运的是，`NSObject`已经遵守了这个协议并且为这些必要的方法提供了默认实现。因此，如果你的对象是继承自`NSObject`（或者它的其他子类），大部分的工作都已经完成了。  
 
 为了让这些默认的方法正常工作，你需要确保对象的存取方法和实例变量「」。这让默认的实现能够根据键值编码信息找到对象属性。「接下来你也可以额外拓展和通过提供一些用于认证或处理特殊情况的函数来定制键值编码。」  
 
 ##Swift中的键值编码
 在Swift中，继承自`NSObject`或其子类的对象，其属性都是默认采用键值编码的。然而在Objective-C中，一个属性的存取方法以及实例变量必须「」。
 
 



# 一、Kotlin优势

## 1、简洁

数据类：

```kotlin
data class Artist(
    var id: Long,
    var name: String,
    var url: String,
    var mbid: String)
```

这个数据类，它会自动生成所有属性和它们的访问器，以及一些有用的方法，比如，`toString()`

## 2、空安全

**Kotlin的变量没有默认值，必须手动赋值**

```kotlin
// var notNullArtist: Artist = null  这里不能通过编译. Artist 不能是null

// Artist 可以是 null
var artist: Artist? = null
// artist.print() 无法编译, artist可能是null，我们需要进行处理
artist?.print()

// 只有在确保artist不是null的情况下才能这么调用，否则它会抛出异常
artist!!.print()

// 使用Elvis操作符来给定一个在是null的情况下的替代值
val name = artist?.name ?: "empty"
```

## 3、扩展方法

给Fragment增加一个显示 toast 的函数：

```kotlin
fun Fragment.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) { 
    Toast.makeText(getActivity(), message, duration).show()
}
调用
fragment.toast("Hello world!")
```

## 4、函数式支持（Lambdas）

```kotlin
view.setOnClickListener { toast("Hello world!") }
```

## 5、三元运算

所以它又引入了一个运算符“?:”，学名叫做“Elvis 操作符”，叫起来有点拗口，读者可以把它当作是Java三元运算符“变量名=条件语句?取值A:取值B”的缩写。引入“?:”的实现代码如下所示：

 ```java
result?.getBoolean(packageName) ?: false
 ```



# 二、开发环境配置

项目根目录下的 `build.gradle`

```groovy
buildscript {
    👇
    ext.kotlin_version = '1.3.41'
    repositories {
        ...
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.5.0-beta05'
        👇
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
```

app 目录下的 `build.gradle`：

```groovy
apply plugin: 'com.android.application'
👇
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
...
​
android {
    ...
}
​
dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    👇
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    ...
}
```



# 三、基础篇

## 1、变量

### 1.1 变量的声明与赋值

Java里声明一个View变量的写法；

```java
View v;
```

Kotlin 里声明一个变量的格式是这样的：

```kotlin
var v: View
```

这里有几处不同：

- 有一个 `var` 关键字
- 类型和变量名位置互换了
- 中间是用冒号分隔的
- 结尾没有分号（对，Kotlin 里面不需要分号）

看上去只是语法格式有些不同，但如果真这么写，IDE 会报错：

```kotlin
class Sample {
    var v: View
    // 👆这样写 IDE 会报如下错误
    // Property must be initialized or be abstract
}
```

这个提示是在说，属性需要在声明的同时初始化，除非你把它声明成抽象的。

- 那什么是属性呢？这里我们可以简单类比 Java 的 field 来理解 Kotlin 的 Property，虽然它们其实有些不一样，Kotlin 的 Property 功能会多些。
- 变量居然还能声明成抽象的？

属性为什么要求初始化呢？**因为 Kotlin 的变量是没有默认值的**，这点不像 Java，Java 的 field 有默认值：

既然这样，那我们就给它一个默认值 null 吧，遗憾的是你会发现仍然报错。

```kotlin
class Sample {
    var v: View = null
    // 👆这样写 IDE 仍然会报错，Null can not be a value of a non-null type View
}
```

又不行，IDE 告诉我需要赋一个非空的值给它才行，怎么办？**其实这都是 Kotlin 的空安全设计相关的内容**。

### 1.2 Kotlin 的空安全设计

单来说就是通过 IDE 的提示来避免调用 null 对象，从而避免 NullPointerException。Kotlin 语言级别的提供了默认支持，而且提示的级别从 warning 变成了 error（拒绝编译）：

```kotlin
var view: View = null
// 👆IDE 会提示错误，Null can not be a value of a non-null type View
```

**在 Kotlin 里面，所有的变量默认都是不允许为空的，如果你给它赋值 null，就会报错**，像上面那样。

这种有点强硬的要求，其实是很合理的：既然你声明了一个变量，就是要使用它对吧？那你把它赋值为 null 干嘛？要尽量让它有可用的值啊。

不过，还是有些场景，变量的值真的无法保证空与否，比如你要从服务器取一个 JSON 数据，并把它解析成一个 User 对象：

```kotlin
class User {
    var name: String = null // 👈这样写会报错，但该变量无法保证空与否
}
```

这个时候，空值就是有意义的。对于这些可以为空值的变量，你可以在类型右边加一个 `?` 号，解除它的非空限制：

```kotlin
class User {
    var name: String? = null
}
```

**加了问号之后，一个 Kotlin 变量就像 Java 变量一样没有非空的限制，自由自在了。**

你除了在初始化的时候可以给它赋值为空值，在代码里的任何地方也都可以：

```kotlin
var name: String? = "Mike"
...
name = null // 👈原来不是空值，赋值为空值
```

这种类型之后加 `?` 的写法，在 Kotlin 里叫**可空类型**。

不过，当我们使用了可空类型的变量后，会有新的问题：

由于对空引用的调用会导致空指针异常，所以 Kotlin 在可空变量直接调用的时候 IDE 会报错：

```kotlin
var view: View? = null
view.setBackgroundColor(Color.RED)
// 👆这样写会报错，Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type View?
```

「可能为空」的变量，Kotlin 不允许用。那怎么办？我们尝试用之前检查一下，但似乎 IDE 不接受这种做法：

```kotlin
if (view != null) {
    view.setBackgroundColor(Color.RED)
    // 👆这样写会报错，Smart cast to 'View' is impossible, because 'view' is a mutable property that could have been changed by this time
} 
```

这个报错的意思是即使你检查了非空也不能保证下面调用的时候就是非空，因为在多线程情况下，其他线程可能把它再改成空的。

那么 Kotlin 里是这么解决这个问题的呢？它用的不是 `.` 而是 `?.`：

```kotlin
view?.setBackgroundColor(Color.RED)
```

这个写法同样会对变量做一次非空确认之后再调用方法，这是 Kotlin 的写法，并且它可以做到线程安全，因此这种写法叫做「**safe call**」。

另外还有一种双感叹号的用法

```kotlin
view!!.setBackgroundColor(Color.RED)
```

意思是告诉编译器，我保证这里的 view 一定是非空的，编译器你不要帮我做检查了，有什么后果我自己承担。这种「肯定不会为空」的断言式的调用叫做 「**non-null asserted call**」。一旦用了非空断言，实际上和 Java 就没什么两样了，但也就享受不到 Kotlin 的空安全设计带来的好处（在编译时做检查，而不是运行时抛异常）了。

以上就是 Kotlin 的空安全设计。

关于空安全，最重要的是记住一点：**所谓「可空不可空」，关注的全都是使用的时候，即「这个变量在使用时是否可能为空」。**

另外，Kotlin 的这种空安全设计在与 Java 的互相调用上是完全兼容的，这里的兼容指：

Java 里面的 @Nullable 和 @NonNull 注解，在转换成 Kotlin 后对应的就是可空变量和不可空变量，至于怎么将 Java 代码转换为 Kotlin，Android Studio 给我们提供了很方便的工具（但并不完美），

```java
@Nullable
String name;
@NonNull
String value = "hello";
```

```kotlin
var name: String? = null
var value: String = "hello"
```



### 1.3 其它

**延迟初始化**

```kotlin
lateinit var view: View
```

这个 `lateinit` 的意思是：告诉编译器我没法第一时间就初始化，但我肯定会在使用它之前完成初始化的。

它的作用就是让 IDE 不要对这个变量检查初始化和报错。换句话说，加了这个 `lateinit` 关键字，这个变量的初始化就全靠你自己了，编译器不帮你检查了。

然后我们就可以在 onCreate 中进行初始化了：

```kotlin
lateinit var view: View
override fun onCreate(...) {
    ...
    👇
    view = findViewById(R.id.tvContent)
}
```

延迟初始化对变量的赋值次数没有限制，你仍然可以在初始化之后再赋其他的值给 `view`。



**类型推断**

Kotlin 有个很方便的地方是，如果你在声明的时候就赋值，那不写变量类型也行：

```ko
var name: String = "Mike"
👇
var name = "Mike"
```

「动态类型」是指变量的类型在运行时可以改变；而「类型推断」是你在代码里不用写变量类型，编译器在编译的时候会帮你补上。因此，Kotlin 是一门静态语言。



**val 和 var**

val 是 Kotlin 在 Java 的「变量」类型之外，又增加的一种变量类型：只读变量。它只能赋值一次，不能修改。而 var 是一种可读可写变量。

> var 是 variable 的缩写，val 是 value 的缩写。



## 2、函数

Kotlin 除了变量声明外，函数的声明方式也和 Java 的方法不一样。Java 的方法（method）在 Kotlin 里叫函数（function），其实没啥区别，或者说其中的区别我们可以忽略掉。对任何编程语言来讲，变量就是用来存储数据，而函数就是用来处理数据。

### 2.1 函数的声明

Java里的方法是这么写的：

```java
Food cook(String name) {
    ...
}
```

而到了 Kotlin，函数的声明是这样：

```kotlin
👇                      👇
fun cook(name: String): Food {
    ...
}
```

- **以 fun 关键字开头**
- **返回值写在了函数和参数后面**

那如果没有返回值该怎么办？Java 里是返回 void：

```java
void main() {
   ...
}
```

Kotlin 里是返回 Unit，并且可以省略：

```kotlin
           👇
fun main(): Unit {}
// Unit 返回类型可以省略
fun main() {}
```



### 2.2 函数参数的可空控制

函数参数也可以有可空的控制，根据前面说的空安全设计，在传递时需要注意：

```kotlin
// 👇可空变量传给不可空参数，报错
var myName : String? = "rengwuxian"
fun cook(name: String) : Food {}
cook(myName)
  
// 👇可空变量传给可空参数，正常运行
var myName : String? = "rengwuxian"
fun cook(name: String?) : Food {}
cook(myName)
​
// 👇不可空变量传给不可空参数，正常运行
var myName : String = "rengwuxian"
fun cook(name: String) : Food {}
cook(myName)
```

**形参与实参要么都是可空，要么都是不可空，必须一致**



### 2.3 属性的 getter/setter 函数

我们知道，在 Java 里面的 field 经常会带有 getter/setter 函数：

```java
public class User {
    String name;
    public String getName() {
        return this.name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

它们的作用就是可以自定义函数内部实现来达到「钩子」的效果，比如下面这种：

```java
public class User {
    String name;
    public String getName() {
        return this.name + " nb";
    }
    public void setName(String name) {
        this.name = "Cute " + name;
    }
}
```

在 Kotlin 里，这种 getter / setter 是怎么运作的呢？

```kotlin
class User {
    var name = "Mike"
    fun run() {
        name = "Mary"
        // 👆的写法实际上是👇这么调用的
        // setName("Mary")
        // 建议自己试试，IDE 的代码补全功能会在你打出 setn 的时候直接提示 name 而不是 setName
        
        println(name)
        // 👆的写法实际上是👇这么调用的
        // print(getName())
        // IDE 的代码补全功能会在你打出 getn 的时候直接提示 name 而不是 getName
    }
}
```

那么我们如何来操作前面提到的「钩子」呢？看下面这段代码：

```kotlin
class User {
    var name = "Mike"
        👇
        get() {
            return field + " nb"
        }
        👇   👇 
        set(value) {
            field = "Cute " + value
        }
}
```

格式上和 Java 有一些区别：

- getter / setter 函数有了专门的关键字 get 和 set
- getter / setter 函数位于 var 所声明的变量下面
- setter 函数参数是 value

除此之外还多了一个叫 field 的东西。这个东西叫做「**Backing Field**」，中文翻译是**幕后字段**或**后备字段**（马云背后的女人😝）。具体来说，你的这个代码：

```kotlin
class Kotlin {
  var name = "kaixue.io"
}
```

在编译后的字节码大致等价于这样的 Java 代码：

```java
public final class Kotlin {
   @NotNull
   private String name = "kaixue.io";
​
   @NotNull
   public final String getName() {
      return this.name;
   }
​
   public final void setName(@NotNull String name) {
      this.name = name;
   }
}
```



### 2.4 函数简化

#### 2.4.1 使用 `=` 连接返回值

我们已经知道了 Kotlin 中函数的写法：

```kotlin
fun area(width: Int, height: Int): Int {
    return width * height
}

// 其实，这种只有一行代码的函数，还可以这么写：
fun area(width: Int, height: Int): Int = width * height

```

`{}` 和 `return` 没有了，使用 `=` 符号连接返回值。

我们之前讲过，Kotlin 有「类型推断」的特性，那么这里函数的返回类型还可以隐藏掉：

```kotlin
//                               👇省略了返回类型
fun area(width: Int, height: Int) = width * height
```

不过，在实际开发中，还是推荐显式地将返回类型写出来，增加代码可读性。

以上是函数有返回值时的情况，对于没有返回值的情况，可以理解为返回值是 `Unit`：

```kotlin
fun sayHi(name: String) {
    println("Hi " + name)
}

// 因此也可以简化成下面这样：
fun sayHi(name: String) = println("Hi " + name)
```

#### 2.4.2 参数默认值

Java 中，允许在一个类中定义多个名称相同的方法，但是参数的类型或个数必须不同，这就是方法的重载：

```java
public void sayHi(String name) {
    System.out.println("Hi " + name);
}
​
public void sayHi() {
    sayHi("world"); 
}
```

在 Kotlin 中，也可以使用这样的方式进行函数的重载，不过还有一种更简单的方式，那就是「参数默认值」：

```kotlin
fun sayHi(name: String = "world") = println("Hi " + name)

sayHi("kaixue.io")
sayHi() // 使用了默认值 "world"
```

这里的 `world` 是参数 `name` 的默认值，当调用该函数时不传参数，就会使用该默认值。

这就等价于上面 Java 写的重载方法，当调用 `sayHi` 函数时，参数是可选的：



不过参数默认值在调用时也不是完全可以放飞自我的。

来看下面这段代码，这里函数中有默认值的参数在无默认值参数的前面：

```kotlin
fun sayHi(name: String = "world", age: Int) {
    ...
}
// sayHi(10)   这时想使用默认值进行调用，IDE 会报以下两个错误
// The integer literal does not conform to the expected type String
// No value passed for parameter 'age'
```

这个错误就是告诉你参数不匹配，说明我们的「打开方式」不对，其实 Kotlin 里是通过「命名参数」来解决这个问题的。

#### 2.4.3 命名参数

```kotlin
fun sayHi(name: String = "world", age: Int) {
    ...
}
      👇   
sayHi(age = 21)
```

在调用函数时，显式地指定了参数 `age` 的名称，这就是「命名参数」。Kotlin 中的每一个函数参数都可以作为命名参数。

再来看一个有非常多参数的函数的例子：

```kotlin
fun sayHi(name: String = "world", age: Int, isStudent: Boolean = true, isFat: Boolean = true, isTall: Boolean = true) {
    ...
}
```

当函数中有非常多的参数时，调用该函数就会写成这样：

```kotlin
sayHi("world", 21, false, true, false)

// 可读性很差。通过命名参数，我们就可以这么写：
sayHi(name = "wo", age = 21, isStudent = false, isFat = true, isTall = false)
```

与命名参数相对的一个概念被称为「位置参数」，也就是按位置顺序进行参数填写。

当一个函数被调用时，如果混用位置参数与命名参数，那么所有的位置参数都应该放在第一个命名参数之前：

```kotlin
fun sayHi(name: String = "world", age: Int) {
    ...
}

sayHi(name = "wo", 21) // 👈 IDE 会报错，Mixing named and positioned arguments is not allowed
sayHi("wo", age = 21) // 👈 这是正确的写法
```

### 2.5 嵌套函数

```kotlin
fun login(user: String, password: String, illegalStr: String) {
    // 验证 user 是否为空
    if (user.isEmpty()) {
        throw IllegalArgumentException(illegalStr)
    }
    // 验证 password 是否为空
    if (password.isEmpty()) {
        throw IllegalArgumentException(illegalStr)
    }
}
```

优化：

```kotlin
fun login(user: String, password: String, illegalStr: String) {
    fun validate(value: String) {
        if (value.isEmpty()) {
                                              👇
            throw IllegalArgumentException(illegalStr)
        }
    }
    ...
}
```

这里我们将共同的验证逻辑放进了嵌套函数 `validate` 中，并且 `login` 函数之外的其他地方无法访问这个嵌套函数。该嵌套函数内直接使用外层函数 `login` 的参数 `illegalStr`。

```kotlin
// 用到了 lambda 表达式以及 Kotlin 内置的 require 函数
fun login(user: String, password: String, illegalStr: String) {
    require(user.isNotEmpty()) { illegalStr }
    require(password.isNotEmpty()) { illegalStr }
}
```







# 四、类和对象 

## 1、次构造器

```java
public class User {
    int id;
    String name;
      👇   👇
    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```



```kotlin
class User {
    val id: Int
    val name: String
         👇
    constructor(id: Int, name: String) {
 //👆 没有 public
        this.id = id
        this.name = name
    }
}
```

可以发现有两点不同：

- Java 中构造器和类同名，**Kotlin 中使用 `constructor` 表示。**
- Kotlin 中构造器没有 public 修饰，因为默认可见性就是公开的（关于可见性修饰符这里先不展开，后面会讲到）。

### 1.1 主构造器

```kotlin
class User constructor(name: String) {
    //                  👇 这里与构造器中的 name 是同一个
    var name: String = name
}
```

这里有几处不同点：

- `constructor` 构造器移到了类名之后
- 类的属性 `name` 可以引用构造器中的参数 `name`

**这个写法叫「主构造器 primary constructor」。**

**写在类中的构造器被称为「次构造器」。**在 Kotlin 中一个类最多只能有 1 个主构造器（也可以没有），而次构造器是没有个数限制。

主构造器中的参数除了可以在类的属性中使用，还可以在 `init` 代码块中使用：

```kotlin
class User constructor(name: String) {
    var name: String
    init {
        this.name = name
    }
}
```

其中 `init` 代码块是紧跟在主构造器之后执行的，这是因为主构造器本身没有代码体，`init` 代码块就充当了主构造器代码体的功能。

另外，如果类中有主构造器，那么其他的次构造器都需要通过 `this` 关键字调用主构造器，可以直接调用或者通过别的次构造器间接调用。如果不调用 IDE 就会报错：

```kotlin
class User constructor(var name: String) {
    constructor(name: String, id: Int) {
    // 👆这样写会报错，Primary constructor call expected
    }
}
```

为什么当类中有主构造器的时候就强制要求次构造器调用主构造器呢？

我们从主构造器的特性出发，一旦在类中声明了主构造器，就包含两点：

- 必须性：创建类的对象时，不管使用哪个构造器，都需要主构造器的参与
- 第一性：在类的初始化过程中，首先执行的就是主构造器

这也就是主构造器的命名由来。

当一个类中同时有主构造器与次构造器的时候，需要这样写：

```kotlin
class User constructor(var name: String) {
                                   // 👇  👇 直接调用主构造器
    constructor(name: String, id: Int) : this(name) {
    }
                                                // 👇 通过上一个次构造器，间接调用主构造器
    constructor(name: String, id: Int, age: Int) : this(name, id) {
    }
}
```

在使用次构造器创建对象时，`init` 代码块是先于次构造器执行的。如果把主构造器看成身体的头部，那么 `init` 代码块就是颈部，次构造器就相当于身体其余部分。

细心的你也许会发现这里又出现了 `:` 符号，它还在其他场合出现过，例如：

- 变量的声明：`var id: Int`
- 类的继承：`class MainActivity : AppCompatActivity() {}`
- 接口的实现：`class User : Impl {}`
- 匿名类的创建：`object: ViewPager.SimpleOnPageChangeListener() {}`
- 函数的返回值：`fun sum(a: Int, b: Int): Int`

**可以看出 `:` 符号在 Kotlin 中非常高频出现，它其实表示了一种依赖关系，在这里表示依赖于主构造器。**

通常情况下，主构造器中的 `constructor` 关键字可以省略：

```kotlin
class User(name: String) {
    var name: String = name
}
```

但有些场景，`constructor` 是不可以省略的，例如在主构造器上使用「可见性修饰符」或者「注解」：

- 可见性修饰符我们之前已经讲过，它修饰普通函数与修饰构造器的用法是一样的，这里不再详述：

  ```kotlin
  class User private constructor(name: String) {
  //           👆 主构造器被修饰为私有的，外部就无法调用该构造器
  }
  ```



### 1.2 主构造器里声明属性

之前我们讲了主构造器中的参数可以在属性中进行赋值，其实还可以在主构造器中直接声明属性：

```kotlin
class User(var name: String) {
}
// 等价于：
class User(name: String) {
  var name: String = name
}
```

如果在主构造器的参数声明时加上 `var` 或者 `val`，就等价于在类中创建了该名称的属性（property），并且初始值就是主构造器中该参数的值。

**以上讲了所有关于主构造器相关的知识，让我们总结一下类的初始化写法：**

- 首先创建一个 `User` 类：

  ```kotlin
  class User {
  }
  ```

- 添加一个参数为 `name` 与 `id` 的主构造器：

  ```kotlin
  class User(name: String, id: String) {
  }
  ```

- 将主构造器中的 `name` 与 `id` 声明为类的属性

  ```kotlin
  class User(val name: String, val id: String) {
  }
  ```

- 然后在 `init` 代码块中添加一些初始化逻辑：

  ```kotlin
  class User(val name: String, val id: String) {
      init {
          ...
      }
  }
  ```

- 最后再添加其他次构造器：

  ```kotlin
  class User(val name: String, val id: String) {
      init {
          ...
      }
      
      constructor(person: Person) : this(person.name, person.id) {
      }
  }
  ```

  

**init**

除了构造器，Java 里常常配合一起使用的 init 代码块，在 Kotlin 里的写法也有了一点点改变：你需要给它加一个 `init` 前缀。

```java
public class User {
   👇
    {
        // 初始化代码块，先于下面的构造器执行
    }
    public User() {
    }
}
```

Kotlin：

```kotlin
class User {
    👇
    init {
        // 初始化代码块，先于下面的构造器执行
    }
    constructor() {
    }
}
```

正如上面标注的那样，Kotlin 的 init 代码块和 Java 一样，都在实例化时执行，并且执行顺序都在构造器之前。



## 2、final

Java 的类如果不加 final 关键字，默认是可以被继承的，**而 Kotlin 的类默认就是 final 的。**

Kotlin 中的 `val` 和 Java 中的 `final` 类似，表示只读变量，不能修改。这里分别从成员变量、参数和局部变量来和 Java 做对比：

```java
final int final1 = 1;
             👇  
void method(final String final2) {
     👇
    final String final3 = "The parameter is " + final2;
}
```

Kotlin

```kotlin
val fina1 = 1
       // 👇 参数是没有 val 的
fun method(final2: String) {
    👇
    val final3 = "The parameter is " + final2
}
```

可以看到不同点主要有：

- final 变成了 val。
- **Kotlin 函数参数默认是 val 类型**，所以参数前不需要写 val 关键字，**Kotlin 里这样设计的原因是保证了参数不会被修改**，而 Java 的参数可修改（默认没 final 修饰）会增加出错的概率。

上一期说过，`var` 是 variable 的缩写， `val` 是 value 的缩写。



**val  自定义 getter**

不过 `val` 和 `final` 还是有一点区别的，虽然 `val` 修饰的变量不能二次赋值，但可以通过自定义变量的 getter 函数，让变量每次被访问时，返回动态获取的值：

```kotlin
val size: Int
    get() { // 👈 每次获取 size 值时都会执行 items.size
        return items.size
    }
```



## 3、object

### 3.1 static property / function

在 `Kotlin` 里，静态变量和静态方法这两个概念被去除了。

那如果想在 Kotlin 中像 Java 一样通过类直接引用该怎么办呢？Kotlin 的答案是 `companion object`：

```kotlin
class Sample {
    ...
       👇
    companion object {
        val anotherString = "Another String"
    }
}
```

### 3.2 单例

Java 里的 `Object` 在 Kotlin 里不用了。

> Java 中的 `Object` 在 Kotlin 中变成了 `Any`，和 `Object` 作用一样：作为所有类的基类。

而 `object` 不是类，像 `class` 一样在 Kotlin 中属于关键字：

```kotlin
object Sample {
    val name = "A name"
}
```

它的意思很直接：创建一个类，并且创建一个这个类的对象。这个就是 `object` 的意思：对象。

在代码中如果要使用这个对象，直接通过它的类名就可以访问：

```kotlin
Sample.name
```

这不就是单例么，所以在 Kotlin 中创建单例不用像 Java 中那么复杂，只需要把 `class` 换成 `object` 就可以了。

- 单例类

  我们看一个单例的例子，分别用 Java 和 Kotlin 实现：

  - Java 中实现单例类（非线程安全）：

    ```java
    public class A {
        private static A sInstance;
        
        public static A getInstance() {
            if (sInstance == null) {
                sInstance = new A();
            }
            return sInstance;
        }
    ​
        // 👇还有很多模板代码
        ...
    }
    ```

    kotlin

    ```kotlin
    // 👇 class 替换成了 object
    object A {
        val number: Int = 1
        fun method() {
            println("A.method()")
        }
    }    
    ```

    和 Java 相比的不同点有：

    - 和类的定义类似，但是把 `class` 换成了 `object` 。
    - 不需要额外维护一个实例变量 `sInstance`。
    - 不需要「保证实例只创建一次」的 `getInstance()` 方法。

    相比 Java 的实现简单多了。

    > 这种通过 `object` 实现的单例是一个饿汉式的单例，并且实现了线程安全。



### 3.3 继承和实现接口

Kotlin 中不仅类可以继承别的类，可以实现接口，`object` 也可以

```kotlin
open class A {
    open fun method() {
        ...
    }
}
​
interface B {
    fun interfaceMethod()
}
  👇      👇   👇
object C : A(), B {
​
    override fun method() {
        ...
    }
​
    override fun interfaceMethod() {
        ...
    }
}
```

- 为什么 object 可以实现接口呢？简单来讲 object 其实是把两步合并成了一步，既有 class 关键字的功能，又实现了单例，这样就容易理解了。



### 3.4 <span style="color:red">匿名类</span>

Java：

```java
ViewPager.SimpleOnPageChangeListener listener = new ViewPager.SimpleOnPageChangeListener() {
    @Override // 👈
    public void onPageSelected(int position) {
        // override
    }
};
```

Kotlin：

```kotlin
val listener = object: ViewPager.SimpleOnPageChangeListener() {
    override fun onPageSelected(position: Int) {
        // override
    }
}   
```

和 Java 创建匿名类的方式很相似，只不过把 `new` 换成了 `object:`：

- Java 中 `new` 用来创建一个匿名类的对象
- Kotlin 中 `object:` 也可以用来创建匿名类的对象

这里的 `new` 和 `object:` 修饰的都是接口或者抽象类。

### 3.5 companion object

```kotlin
class A {
       👇
    companion object B {
        var c: Int = 0
    }
}
```

把需要静态的变量或函数放在内部对象 B 中，外部可以通过如下的方式调用该静态变量：A.B.c

`companion` 可以理解为伴随、伴生，表示修饰的对象和外部类绑定。

但这里有一个小限制：一个类中最多只可以有一个伴生对象，但可以有多个嵌套对象。就像皇帝后宫佳丽三千，但皇后只有一个。

这样的好处是调用的时候可以省掉对象名：

```kotlin
A.c // 👈 B 没了
```

这就是这节最开始讲到的，Java 静态变量和方法的等价写法：`companion object` 变量和函数。



**静态初始化**

Java 中的静态变量和方法，在 Kotlin 中都放在了 `companion object` 中。因此 Java 中的静态初始化在 Kotlin 中自然也是放在 `companion object` 中的，像类的初始化代码一样，由 `init` 和一对大括号表示：

```kotlin
class Sample {
       👇
    companion object {
         👇
        init {
            ...
        }
    }
}
```

### 3.6 top-level property / function 声明

除了静态函数这种简便的调用方式，Kotlin 还有更方便的东西：「`top-level declaration` 顶层声明」。其实就是把属性和函数的声明不写在 `class` 里面，这个在 Kotlin 里是允许的：

```kotlin
package com.hencoder.plus
​
// 👇 属于 package，不在 class/object 内
fun topLevelFuncion() {
}
```

这样写的属性和函数，不属于任何 `class`，而是直接属于 `package`，它和静态变量、静态函数一样是全局的，但用起来更方便：你在其它地方用的时候，就连类名都不用写：

```kotlin
import com.hencoder.plus.topLevelFunction // 👈 直接 import 函数
​
topLevelFunction()
```



**对比**

那在实际使用中，在 `object`、`companion object` 和 top-level 中该选择哪一个呢？简单来说按照下面这两个原则判断：

- 如果想写工具类的功能，直接创建文件，写 top-level「顶层」函数。
- 如果需要继承别的类或者实现接口，就用 `object` 或 `companion object`。



### 3.7 常量

```kotlin
class Sample {
    companion object {
         👇                  // 👇
        const val CONST_NUMBER = 1
    }
}
​
const val CONST_SECOND_NUMBER = 2
```

发现不同点有：

- Kotlin 的常量必须声明在对象（包括伴生对象）或者「top-level 顶层」中，因为常量是静态的。
- Kotlin 新增了修饰常量的 `const` 关键字。

除此之外还有一个区别：

- Kotlin 中只有基本类型和 String 类型可以声明成常量。

原因是 Kotlin 中的常量指的是 「compile-time constant 编译时常量」，它的意思是「编译器在编译的时候就知道这个东西在每个调用处的实际值」，因此可以在编译时直接把这个值硬编码到代码里使用的地方。

而非基本和 String 类型的变量，可以通过调用对象的方法或变量改变对象内部的值，这样这个变量就不是常量了，



# 五、数组和集合

## 1、Array

声明一个 String 数组：

- Java 中的写法：

  ```java
  String[] strs = {"a", "b", "c"};
  ```

- Kotlin 中的写法：

  ```kotlin
  val strs: Array<String> = arrayOf("a", "b", "c")
              👆              👆
  ```

可以看到 Kotlin 中的数组是一个拥有泛型的类，创建函数也是泛型函数，和集合数据类型一样。

> 针对泛型的知识点，我们在后面的文章会讲，这里就先按照 Java 泛型来理解。

将数组泛型化有什么好处呢？对数组的操作可以像集合一样功能更强大，由于泛型化，Kotlin 可以给数组增加很多有用的工具函数：

- `get() / set()`
- `contains()`
- `first()`
- `find()`

这样数组的实用性就大大增加了。

- 取值和修改

  Kotlin 中获取或者设置数组元素和 Java 一样，可以使用方括号加下标的方式索引：

  ```kotlin
  println(strs[0])
     👇      👆
  strs[1] = "B"
  ```

- 不支持协变

  Kotlin 的数组编译成字节码时使用的仍然是 Java 的数组，但在语言层面是泛型实现，这样会失去协变 (covariance) 特性，就是子类数组对象不能赋值给父类的数组变量：

  - Kotlin

    ```kotlin
    val strs: Array<String> = arrayOf("a", "b", "c")
                      👆
    val anys: Array<Any> = strs // compile-error: Type mismatch
                    👆
    ```

    而这在 Java 中是可以的：

    ```java
    String[] strs = {"a", "b", "c"};
      👆
    Object[] objs = strs; // success
    ```

    

## 2、List



- Java 中创建一个列表：

  ```java
  List<String> strList = new ArrayList<>();
  strList.add("a");
  strList.add("b");
  strList.add("c"); // 👈 添加元素繁琐
  ```

  

- Kotlin 中创建一个列表：

  ```kotlin
  val strList = listOf("a", "b", "c")
  ```

  首先能看到的是 Kotlin 中创建一个 `List` 特别的简单，有点像创建数组的代码。而且 Kotlin 中的 `List` 多了一个特性：支持 covariant（协变）。也就是说，可以把子类的 `List` 赋值给父类的 `List` 变量：

  ```kotlin
  val strs: List<String> = listOf("a", "b", "c")
                  👆
  val anys: List<Any> = strs // success
  ```

  而这在 Java 中是会报错的：

  ```java
  List<String> strList = new ArrayList<>();
         👆
  List<Object> objList = strList; // 👈 compile error: incompatible types
  ```

  对于协变的支持与否，`List` 和数组刚好反过来了。关于协变，这里只需结合例子简单了解下，后面的文章会对它展开讨论。

  - 和数组的区别

    Kotlin 中数组和 MutableList 的 API 是非常像的，主要的区别是数组的元素个数不能变。那在什么时候用数组呢？

    - 这个问题在 Java 中就存在了，数组和 `List` 的功能类似，`List` 的功能更多一些，直觉上应该用 `List` 。但数组也不是没有优势，基本类型 (`int[]`、`float[]`) 的数组不用自动装箱，性能好一点。
    - 在 Kotlin 中也是同样的道理，在一些性能需求比较苛刻的场景，并且元素类型是基本类型时，用数组好一点。不过这里要注意一点，Kotlin 中要用专门的基本类型数组类 (`IntArray` `FloatArray` `LongArray`) 才可以免于装箱。也就是说元素不是基本类型时，相比 `Array`，用 `List` 更方便些。



## 3、Set

```kotlin
val strSet = setOf("a", "b", "c")
```

和 `List` 类似，`Set` 同样具有 covariant（协变）特性。

## 4、Map

Kotlin 中创建一个 `Map`：

```kotlin
val map = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 3)
```

取值与修改：

```kotlin
val value1 = map.get("key1")
// 以使用方括号的方式获取：             👇
val value2 = map["key2"]


val map = mutableMapOf("key1" to 1, "key2" to 2)
map.put("key1", 2)
// Kotlin 中也可以用方括号的方式改变 Map 中键对应的值：
map["key1"] = 2  
```

这里用到了「操作符重载」的知识，实现了和数组一样的「Positional Access Operations」



## 5、可变集合/不可变集合

上面修改 `Map` 值的例子中，创建函数用的是 `mutableMapOf()` 而不是 `mapOf()`，因为只有 `mutableMapOf()` 创建的 `Map` 才可以修改。Kotlin 中集合分为两种类型：只读的和可变的。这里的只读有两层意思：

- 集合的 size 不可变
- 集合中的元素值不可变

以下是三种集合类型创建不可变和可变实例的例子：

- `listOf()` 创建不可变的 `List`，`mutableListOf()` 创建可变的 `List`。
- `setOf()` 创建不可变的 `Set`，`mutableSetOf()` 创建可变的 `Set`。
- `mapOf()` 创建不可变的 `Map`，`mutableMapOf()` 创建可变的 `Map`。

可以看到，有 `mutable` 前缀的函数创建的是可变的集合，没有 `mutbale` 前缀的创建的是不可变的集合，不过不可变的可以通过 `toMutable*()` 系函数转换成可变的集合：

然后就可以对集合进行修改了，这里有一点需要注意下：

- `toMutable*()` 返回的是一个新建的集合，**原有的集合还是不可变的，所以只能对函数返回的集合修改。**



## 6、`Sequence`

除了集合 Kotlin 还引入了一个新的容器类型 `Sequence`，它和 `Iterable` 一样用来遍历一组数据并可以对每个元素进行特定的处理，先来看看如何创建一个 `Sequence`。

**类似 `listOf()` ，使用一组元素创建**

```kotlin
sequenceOf("a", "b", "c")
```

**使用 `Iterable` 创建：**

```kotlin
val list = listOf("a", "b", "c")
list.asSequence()
```

这里的 `List` 实现了 `Iterable` 接口。

**使用 lambda 表达式创建：**

```kotlin
 // 👇 第一个元素
val sequence = generateSequence(0) { it + 1 }
                                  // 👆 lambda 表达式，负责生成第二个及以后的元素，it 表示前一个元素
```





# 六、常用

## 1、单例

1）饿汉式

```kotlin
object SimpleSington {
  fun test() {}
}
//在Kotlin里调用
SimpleSington.test()

//在Java中调用
SimpleSington.INSTANCE.test();
```

2）懒汉式

```java
// 懒汉式加载
class LazySingleton private constructor(){
    companion object {
        val instance: LazySingleton by lazy { LazySingleton() }
    }
}
```

- 显式声明构造方法为private
- companion object用来在class内部声明一个对象
- LazySingleton的实例instance 通过lazy来实现懒汉式加载
- lazy默认情况下是线程安全的，这就可以避免多个线程同时访问生成多个实例的问题

## 2 lateinit变量是否已初始化的方法

```java
class Foo {
    lateinit var lateInitVar: String
    fun checkInit() {
        if(this::lateInitVar.isInitialized){　　//重要，this::前缀是必须的。
　　     　　//如果已经初始化了，返回true 　
　　　　　}
    }
}    
```


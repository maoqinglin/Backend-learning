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
// 这里不能通过编译. Artist 不能是null
var notNullArtist: Artist = null

// Artist 可以是 null
var artist: Artist? = null

// 无法编译, artist可能是null，我们需要进行处理
artist.print()

// 只要在artist != null时才会打印
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



### 5、三元运算

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



# 三、基础语法

## 1、变量

### 1）、变量的声明与赋值

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

属性为什么要求初始化呢？因为 Kotlin 的变量是没有默认值的，这点不像 Java，Java 的 field 有默认值：

既然这样，那我们就给它一个默认值 null 吧，遗憾的是你会发现仍然报错。

```kotlin
class Sample {
    var v: View = null
    // 👆这样写 IDE 仍然会报错，Null can not be a value of a non-null type View
}
```

又不行，IDE 告诉我需要赋一个非空的值给它才行，怎么办？**其实这都是 Kotlin 的空安全设计相关的内容**。



### 2）、Kotlin 的空安全设计

单来说就是通过 IDE 的提示来避免调用 null 对象，从而避免 NullPointerException。Kotlin 语言级别的提供了默认支持，而且提示的级别从 warning 变成了 error（拒绝编译）：

```kotlin
var view: View = null
// 👆IDE 会提示错误，Null can not be a value of a non-null type View
```

在 Kotlin 里面，所有的变量默认都是不允许为空的，如果你给它赋值 null，就会报错，像上面那样。

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

加了问号之后，一个 Kotlin 变量就像 Java 变量一样没有非空的限制，自由自在了。

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



### 3）、延迟初始化

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

### 4）、类型推断

Kotlin 有个很方便的地方是，如果你在声明的时候就赋值，那不写变量类型也行：

```ko
var name: String = "Mike"
👇
var name = "Mike"
```

「动态类型」是指变量的类型在运行时可以改变；而「类型推断」是你在代码里不用写变量类型，编译器在编译的时候会帮你补上。因此，Kotlin 是一门静态语言。

### 5）、val 和 var

val 是 Kotlin 在 Java 的「变量」类型之外，又增加的一种变量类型：只读变量。它只能赋值一次，不能修改。而 var 是一种可读可写变量。

> var 是 variable 的缩写，val 是 value 的缩写。



## 2、函数

Kotlin 除了变量声明外，函数的声明方式也和 Java 的方法不一样。Java 的方法（method）在 Kotlin 里叫函数（function），其实没啥区别，或者说其中的区别我们可以忽略掉。对任何编程语言来讲，变量就是用来存储数据，而函数就是用来处理数据。

### 1）、函数的声明

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



### 2）、函数参数的可空控制

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



### 3）、属性的 getter/setter 函数

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




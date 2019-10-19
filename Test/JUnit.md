# 一、JUnit4.4 概述

JUnit 4.4 主要提供了以下三个大方面的新特性来更好的抓住编程人员的代码意图：

- 提供了新的断言语法（Assertion syntax）——`assertThat`
- 提供了假设机制（Assumption）
- 提供了理论机制（Theory）



# 二、新的断言语法 -- assertThat

## 1. Hamcrest 

是一个测试的框架，它提供了一套通用的匹配符 Matcher，灵活使用这些匹配符定义的规则，程序员可以更加精确的表达自己的测试思想，指定所想设定的测试条件。比如，有时候定义的测试数据范围太精确，往往是若干个固定的确定值，这时会导致测试非常脆弱，因为接下来的测试数据只要稍稍有变化，就可能导致测试失败（比如 `assertEquals( x, 10 );` 只能判断 `x` 是否等于 `10`，如果 `x` 不等于 `10`，测试失败）；有时候指定的测试数据范围又不够太精确，这时有可能会造成某些本该会导致测试不通过的数据，仍然会通过接下来的测试，这样就会降低测试的价值。 Hamcrest 的出现，给程序员编写测试用例提供了一套规则和方法，使用其可以更加精确的表达程序员所期望的测试的行为。

JUnit 4.4 结合 Hamcrest 提供了一个全新的断言语法——`assertThat`。程序员可以只使用 `assertThat` 一个断言语句，结合 Hamcrest 提供的匹配符，就可以表达全部的测试思想。

`assertThat` 的基本语法如下：

```
assertThat( [value], [matcher statement] );
```

- `value` 是接下来想要测试的变量值；
- `matcher statement` 是使用 Hamcrest 匹配符来表达的对前面变量所期望的值的声明，如果 `value` 值与 `matcher statement` 所表达的期望值相符，则测试成功，否则测试失败。



## 2. assertThat 的优点

- **优点 1**：以前 JUnit 提供了很多的 assertion 语句，如：`assertEquals`，`assertNotSame`，`assertFalse`，`assertTrue`，`assertNotNull`，`assertNull` 等，现在有了 JUnit 4.4，一条 `assertThat` 即可以替代所有的 assertion 语句，这样可以在所有的单元测试中只使用一个断言方法，使得编写测试用例变得简单，代码风格变得统一，测试代码也更容易维护。
- **优点 2**：`assertThat` 使用了 Hamcrest 的 Matcher 匹配符，用户可以使用匹配符规定的匹配准则精确的指定一些想设定满足的条件，具有很强的易读性，而且使用起来更加灵活。如清单 2 所示：

**使用匹配符 Matcher 和不使用之间的比较**

```java
想判断某个字符串 s 是否含有子字符串 "developer" 或 "Works" 中间的一个
// JUnit 4.4 以前的版本：
assertTrue(s.indexOf("developer")>-1||s.indexOf("Works")>-1 );

// JUnit 4.4：
assertThat(s, anyOf(containsString("developer"), containsString("Works"))); 
// 匹配符 anyOf 表示任何一个条件满足则成立，类似于逻辑或 "||"， 匹配符 containsString 表示是否含有参数子 
```



- **优点 3**：`assertThat` 不再像 `assertEquals` 那样，使用比较难懂的“谓宾主”语法模式（如：`assertEquals(3, x);`），相反，`assertThat` 使用了类似于“**主谓宾**”的易读语法模式（如：`assertThat(x,is(3));`），使得代码更加直观、易读。
- **优点 4**：可以将这些 Matcher 匹配符联合起来灵活使用，达到更多目的。如清单 3 所示：

```java
// 联合匹配符not和equalTo表示“不等于”
assertThat( something, not( equalTo( "developer" ) ) ); 
// 联合匹配符not和containsString表示“不包含子字符串”
assertThat( something, not( containsString( "Works" ) ) ); 
// 联合匹配符anyOf和containsString表示“包含任何一个子字符串”
assertThat(something, anyOf(containsString("developer"), containsString("Works")));
```

- 优点 5

  ：错误信息更加易懂、可读且具有描述性（descriptive）。

  JUnit 4.4 以前的版本默认出错后不会抛出额外提示信息，如：

  ```java
  assertTrue( s.indexOf("developer") > -1 || s.indexOf("Works") > -1 );
  ```

  如果该断言出错，只会抛出无用的错误信息，如：`junit.framework.AssertionFailedError：null`。

  如果想在出错时想打印出一些有用的提示信息，必须得程序员另外手动写，如：

  ```java
  assertTrue( "Expected a string containing 'developer' or 'Works'",    s.indexOf("developer") > -1 || s.indexOf("Works") > -1 );
  ```

  非常的不方便，而且需要额外代码。

  **JUnit 4.4 会默认自动提供一些可读的描述信息，如清单 4 所示：**

  ```java
  String s = "hello world!"; 
  assertThat( s, anyOf( containsString("developer"), containsString("Works") ) ); 
  // 如果出错后，系统会自动抛出以下提示信息：
  java.lang.AssertionError: 
  Expected: (a string containing "developer" or a string containing "Works") 
  got: "hello world!"
  ```

  

- **优点 6**：开发人员可以通过实现 Matcher 接口，定制自己想要的匹配符。当开发人员发现自己的某些测试代码在不同的测试中重复出现，经常被使用，这时用户就可以自定义匹配符，将这些代码绑定在一个断言语句中，从而可以达到减少重复代码并且更加易读的目的。



## 3. 如何使用 assertThat

### 1）、一般匹配符

```java
// allOf匹配符表明如果接下来的所有条件必须都成立测试才通过，相当于“与”（&&）
assertThat( testedNumber, allOf( greaterThan(8), lessThan(16) ) );
// anyOf匹配符表明如果接下来的所有条件只要有一个成立则测试通过，相当于“或”（||）
assertThat( testedNumber, anyOf( greaterThan(16), lessThan(8) ) );
// anything匹配符表明无论什么条件，永远为true
assertThat( testedNumber, anything() );
// is匹配符表明如果前面待测的object等于后面给出的object，则测试通过
assertThat( testedString, is( "developerWorks" ) );
// not匹配符和is匹配符正好相反，表明如果前面待测的object不等于后面给出的object，则测试通过
assertThat( testedString, not( "developerWorks" ) );

```

### 2）、字符串相关匹配符

```java
// containsString匹配符表明如果测试的字符串testedString包含子字符串"developerWorks"则测试通过
assertThat( testedString, containsString( "developerWorks" ) );
// endsWith匹配符表明如果测试的字符串testedString以子字符串"developerWorks"结尾则测试通过
assertThat( testedString, endsWith( "developerWorks" ) ); 
// startsWith匹配符表明如果测试的字符串testedString以子字符串"developerWorks"开始则测试通过
assertThat( testedString, startsWith( "developerWorks" ) ); 
// equalTo匹配符表明如果测试的testedValue等于expectedValue则测试通过，equalTo可以测试数值之间，字
//符串之间和对象之间是否相等，相当于Object的equals方法
assertThat( testedValue, equalTo( expectedValue ) ); 
// equalToIgnoringCase匹配符表明如果测试的字符串testedString在忽略大小写的情况下等于
//"developerWorks"则测试通过
assertThat( testedString, equalToIgnoringCase( "developerWorks" ) ); 
// equalToIgnoringWhiteSpace匹配符表明如果测试的字符串testedString在忽略头尾的任意个空格的情况下等
//于"developerWorks"则测试通过，注意：字符串中的空格不能被忽略
assertThat( testedString, equalToIgnoringWhiteSpace( "developerWorks" ) );
```



### 3）、数值相关匹配符

```java
// closeTo匹配符表明如果所测试的浮点型数testedDouble在20.0±0.5范围之内则测试通过
assertThat( testedDouble, closeTo( 20.0, 0.5 ) );
// greaterThan匹配符表明如果所测试的数值testedNumber大于16.0则测试通过
assertThat( testedNumber, greaterThan(16.0) );
// lessThan匹配符表明如果所测试的数值testedNumber小于16.0则测试通过
assertThat( testedNumber, lessThan (16.0) );
// greaterThanOrEqualTo匹配符表明如果所测试的数值testedNumber大于等于16.0则测试通过
assertThat( testedNumber, greaterThanOrEqualTo (16.0) );
// lessThanOrEqualTo匹配符表明如果所测试的数值testedNumber小于等于16.0则测试通过
assertThat( testedNumber, lessThanOrEqualTo (16.0) );
```



### 4）、collection相关匹配符

```java
// hasEntry匹配符表明如果测试的Map对象mapObject含有一个键值为"key"对应元素值为"value"的Entry项则
//测试通过
assertThat( mapObject, hasEntry( "key", "value" ) );
// hasItem匹配符表明如果测试的迭代对象iterableObject含有元素“element”项则测试通过
assertThat( iterableObject, hasItem ( "element" ) );
// hasKey匹配符表明如果测试的Map对象mapObject含有键值“key”则测试通过
assertThat( mapObject, hasKey ( "key" ) );
// hasValue匹配符表明如果测试的Map对象mapObject含有元素值“value”则测试通过
assertThat( mapObject, hasValue ( "key" ) );
```







# 三、假设机制 （Assumption）



# 四、理论机制 （Theory）


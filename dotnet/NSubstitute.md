# 基概

* `NSubstitute`是一个用于`.NET`应用程序的模拟库
* 它提供了了一种简单而强大的方式来创建模拟对象`mocks`和存根`stubs`
* **`mocks`是在单元测试中用来替代真实对象的简化版本，它们模拟真实对象的行为，让测试能够在隔离的环境中运行，而不依赖外部资源或复杂的配置**
* **`stubs`主要用于为测试提供间接输入，提供预定的响应，模拟数据库查询的结果、远程服务的响应等**

# Demo

1. 通过`NgGet`安装包

   ```sh
   NSubstitute
   ```

   ```c#
   NSubstitute.Analyzers.CSharp  // <--这个包专门设计用于与`NSubstitute`测试库一起工作，帮助开发者进行单元测试时避免常见的错误和最佳实践偏离，所以不是必须的，但推荐导入的
   ```

2. 定义一个接口

   ```c#
   public interface ICalculator
   {
       int Add(int a, int b);
       string Mode { get; set; }
   }
   ```

3. 模拟使用

   ```c#
   namespace Unit;
   
   public class MyNSubstitute
   {
       [Fact]
       public void Test1()
       {
         var calculator = Substitute.For<ICalculator>();
         calculator.Add(1, 2).Returns(3);  // 这里含义为设置当调用`Add()`参数并且为1，2时，返回值设为3
         Assert.Equal(3, calculator.Add(1, 2),);
       }
   }
   ```

   * 这里配合了`xUnit`框架一起使用

# 使用

## - 创建`substitute`



### · 语法

* 基本语法为：

  ```c#
  var substitute = Substitute.For<T>();
  ```

  * 正常来说，**`T`的类型通常是一个`interface`，实际上也可以是`class`，但不推荐**

* 创建一个带有构造参数的类的`substitute`：

  ```c#
  var someClass = Substitute.For<SomeClassWithCtorArgs>(5, "hello world");
  ```

* 创建一个实现了多个接口的`substitute`

  ```#
  var command = Substitute.For<ICommand, IDisposable, XXX>();
  ```

* 创建一个实现了多个接口和某个类的`sbusttite`

  ```c#
  var substitute = Substitute.For(new[] { typeof(IInterface1), typeof(IInterface2), typeof(BaseClass) }, new object[] { /* 构造器参数 */ });
  ```

  * <b style="color:red">注意：可以含有多个接口，但最多只能包含一个具体的类（这是因为一个类不能直接继承多个类）</b>

* 创建一个委托类型的`substitute`

  ```c#
  public delegate int Calculate(int a, int b);
  ```

  ```c#
  var calculateMock = Substitute.For<Calculate>(); // <---
  // 配置委托的行为
  calculateMock.Invoke(Arg.Any<int>(), Arg.Any<int>()).Returns(42);
  // 使用模拟的委托
  int result = calculateMock(5, 7);
  // 验证委托被调用
  calculateMock.Received().Invoke(Arg.Any<int>(), Arg.Any<int>());
  // 检查结果
  Assert.Equal(42, result);
  ```

  * **使用`Subsitute.For<T>()`来替代委托类型时，这个替代对象不能同时实现其他接口或类**



### · 原理

* 这是因为**`NSubstitute`(和大多数`.NET`模拟库)是通过创建一个目标类或接口的子类实现来模拟工作的，并重写其`virtual`成员来实现模拟行为，以便于模拟特定的行为或返回值**

  * **当模拟一个接口时：我们可以自由地为所有方法定义模拟行为，因为接口只定义了方法的签名，没有具体实现**

    > * 对于`Demo`中的`var calculator = Substitute.For<ICalculator>();  calculator.Add(1, 2).Returns(3); `这两行代码
    > * 可以简单理解为`Nsubstitute`构造了一个实现了`ICalculator`接口的子类
    > * 然后在这个子类中重写了`Add()`，并设置为当参数传进来的值为`(1,2)`时，返回值固定死返回`3`

  * **当模拟一个具体的类时：我们只能为那些标记为`virtual`的方法定义模拟行为，非虚拟方法将执行原有的实现，而不会使用模拟配置的返回值**（这可能会导致测试中出现意料之外的行为，尤其是如果这些非虚拟方法包含了修改状态或执行重要操作的代码）

* 也可以换个角度去理解

  * **方法被`virtual`修饰**：**这意味着方法可以在子类中重写。因此，`Nsubstitute`可以创建一个继承自原始类的模拟类，并重写此方法以返回预定义的值或执行预定义的行为**，这样，当我们**在测试中调用此方法时，调用的实际上是`NSubstitute`创建的模拟类的重写版本，从而允许我们控制方法的行为和输出**
  * **方法没有被`virtual`修饰**：**这意味着方法是固定的，不能在子类中被重写，`NSubstitute`不能通过继承和重写机制来插入自定义的逻辑，从而无法拦截调用或改变其行为（如果测试代码中为这个非`virtual`修饰的方法进行调用配置返回值的话，会直接报错`NSubstitute.Exceptions.CouldNotSetReturnDueToNoLastCallException:Could not find a call to return from.`）因此，当`Nsubstitute`尝试模拟这个类时，默认情况下对于此方法的任何调用都会执行原始类中的实际代码**



## - 参数匹配器

* 举一个常见的例子来示意先

* 比如说对于`calculator.Add(1, 2).Returns(3);`这行代码

* 当调用`Add()`，传入参数必须为`(1,2)`时，才会返回`3`

* 那么现在需求改为当第一个参数小于0，第二个参数等于2时，我都返回3，这时该如何操作？

* 这时候参数匹配器就闪亮登场了

* 顾名思义，**参数匹配器是用来对参数做过滤筛选操作的，它允许我们在设置`substitute`的行为和验证方法调用时，对输入参数应用灵活的匹配规则**

* 常见的参数匹配器有两个

  * **`Arg.Any<T>()`：它的作用在于匹配任何类型为`T`的参数，也就是说使用这个匹配器，不论方法调用时传递的具体参数(需匹配参数位置)是什么，只要参数类型匹配，就视为满足条件**

    ```c#
    calculator.Add(Arg.Any<int>(), Arg.Any<int>()).Returns(3);
    ```

  * **`Arg.Is<T>(Predicate<T>)`：这个参数匹配器允许我们为参数定义一个条件，只有当方法调用时传递的参数满足这个条件时，才视为匹配**（这提供了更细致的控制能力）

    ```c#
    calculator.Add(Arg.Is<int>(x => x < 0), Arg.Any<int>()).Returns(3);
    ```

  

## - 设置返回值

### · 正常情况下返回某个值

* 对于`Method`：调用`Returns()`设置即可

  ```c#
  var calculator = Substitute.For<ICalculator>();
  calculator.Add(1, 2).Returns(3);
  ```

  * **请注意，`calculator.Add(1, 2).Returns(3);`这行代码的配置仅适用于该参数组合，也就是说当`substitute`调用`Add()`，传入的参数必须为`(1,2)`时，返回值才是为`Returns()`中设置的`3`；如果传入的是别的参数的话，不满去此时的设置条件，正常情况下会返回这个方法返回值类型的默认值**，像这里`int`类型的话就是返回`0`

* 对于`Property`：可以依旧使用`Returns()`设置，也可以通过`setter()`(也就是`=`号)赋值

  ```c#
  calculator.Mode.Returns("DEC");   // Returns()
  
  calculator.Mode = "HEX";          //    =
  ```



### · 对于任意参数均返回某个值

* 可以将`Return()`替换成`ReturnsForAnyArgs()`

* 这代表**不管方法调用时的参数是如何，其返回值都是固定这个你配置的值**

  ```c#
  calculator.Add(1, 2).ReturnsForAnyArgs(100); 
  Assert.Equal(100, calculator.Add(1, 2));       // true
  Assert.Equal(100, calculator.Add(-7, 15));     // true
  ```

  * **当使用`ReturnsForAnyArgs()`后，说明`substitute`在调用这个`Add()`时，不管传入的参数值是怎样(需满去类型匹配即可)，均返回你配置的值100**

  * **so实际上`calculator.Add(1, 2).ReturnsForAnyArgs(100); `这里面的参数值在配置时可以填任意值(也可以用`default`关键字)，均等效**

    ```c#
    calculator.Add(1, 2).ReturnsForAnyArgs(100); 
    calculator.Add(default, default).ReturnsForAnyArgs(100); 
    ```

    * 通常对于任意参数均返回某个固定值这种场景，常常使用`default`关键字
    * **`default`关键字在这种情况下表达了一种“这个方法的参数在这个调用配置中并不重要”的意思**
    * 尽管从技术上讲，`default`关键字与调用`ReturnsForAnyAgs()`的功能没有直接联系，但这种写法在提高代码可读性和表达测试意图方面是具有价值的

* 实际上，使用参数匹配器也可以达成一样的效果

  ```c#
  calculator.Add(Arg.Any<int>(), Arg.Any<int>()).Returns(100); // 这里的意思是指在模拟调用这个`Add()`时，对于任意位置的参数，只要传入的参数值符合其类型，返回值均为100
  ```



### · 从函数中返回

* 不管是`Returns()`还是`ReturnsForAnyArgs()`

* 我们在设置值时除了设置为某个固定值，比如说100，也还可以通过`Func<CallInfo,T>`（`Lambda函数`）做更为复杂的逻辑处理

* **其中`CallInfo`是提供对调用所有参数的访问的类型，`T`为此方法的返回类型**

* 在`Func<CallInfo,T>`中常见的获取参数的方式有两种

  * **通过索引来访问**

    ```c#
    calculator.Add(Arg.Any<int>(), Arg.Any<int>())
        .Returns(x => (int)x[0] + (int)x[1]);  
    ```
    
    * **注意：通过索引获取到的参数返回的是`object`类型(这样设计是便于容纳任何类型的值)**
    
  * **通过`T Arg<T>()`或`T ArgAt<T>(int position)`**来访问
  
    * `T Arg<T>()`：**通过类型`T`来获取传递给此调用的参数**
  
      ```c#
      calculator.Add(Arg.Any<string>(), Arg.Any<int>()).Returns(x => (666 + x.Arg<int>());
      ```
  
      * <b style="color:red">注意：如果此方法具有多个同类型的参数，那么使用`T Arg<T>()`的话会直接报错</b>
  
    * `T ArgAt<T>(int position)`: **获取指定位置(从0开始)的传递给此调用的参数**
  
      ```c#
      calculator.Add(Arg.Any<string>(), Arg.Any<int>()).Returns(x => (666 + x.ArgAt<int>(1));
      ```



### · 配置序列的返回值

* `Returns()`和`ReturnsForAnyArgs()`是**可以链式的指定多个返回值的，也就是配置为在多次调用中返回不同的值**

* 如下所示

  ```c#
  calculator.Mode.Returns("DEC", "HEX", "BIN");
  Assert.Equal("DEC", calculator.Mode);    // true
  Assert.Equal("HEX", calculator.Mode);    // true
  Assert.Equal("BIN", calculator.Mode);    // true
  Assert.Equal("BIN", calculator.Mode);    // true
  ```

  * **同样适用对方法的调用**

  * **同样支持使用回调来返回**

    ```c#
    calculator.Mode.Returns(x => "DEC", x => "HEX", x => { throw new Exception(); });
    Assert.AreEqual("DEC", calculator.Mode);
    Assert.AreEqual("HEX", calculator.Mode);
    Assert.Throws<Exception>(() => { var result = calculator.Mode; });												
    ```

  * **这种行为允许我们模拟一个属性或方法随着时间变化的返回值，而在这些值都已经返回一遍后，对于随后的访问，它会稳定在最后一个值**



### · 替换返回值

* **`Method`或`Property`的返回值可以根据需要设置多次的，仅返回最近的配置**

* 如下所示：

  ```c#
          var calculator = Substitute.For<ICalculator>();
          calculator.Mode.Returns("DEC,HEX,OCT");
          calculator.Mode.Returns(x => "???");
          calculator.Mode.Returns("HEX");
          calculator.Mode.Returns("BIN");
          Assert.Equal(calculator.Mode, "BIN");   // true
  
      
          calculator.Add(1, 2).Returns(x => (int)x[0] + (int)x[1]);
          calculator.Add(1, 2).Returns(0);
          calculator.Add(1, 2).Returns(9, 7, 5);
          Assert.Equal(9, calculator.Add(1,2));   // true
          Assert.Equal(7, calculator.Add(1,2));   // true
          Assert.Equal(5, calculator.Add(1,2));   // true
          Assert.Equal(5, calculator.Add(1,2));   // true
  ```

* <b style="color:red">所以可以总结道，如果存在多个配置，最近的配置(就近原则)会被使用，对于这个生效的配置，它可能是返回多个值，则随着`Method`/`Property`调用依次返回，在所有值都访问一遍后最后会停留在最后的那个值上</b>



### · 返回某个类型的所有调用

* 对于某个`substitute`，假设其包含的方法统一都是某个类型的返回值

* 则`NSubstitute`则允许我们通过调用`ReturnsForAll()`来实现统一的返回

  ```c#
  public interface ICalculator
  {
      int Add(int a, int b);
      int Sub(int a, int c);
      int Multi(int a, int b);
  }
                         
   var calculator = Substitute.For<ICalculato>();    
   calculator.ReturnsForAll<int>(1);         
   Assert.Equal(1, calculator.Add(1,2));       // true
   Assert.Equal(1, calculator.Sub(1,2));       // true
   Assert.Equal(1, calculator.Multi(4,9));     // ture
  ```

* **`ReturnsForAll<T>(value)`要求类型必须完全匹配，也就是说假设我们的方法返回类型都是`Animal`,现有`Cat`类实现了`Animal`，那么我们在设置时应该为`ReturnsForAll<Aninaml>(cat实例)`，而不是`ReturnsForAll<Cat>(cat实例)`,这样的话不会按预期工作**

* **注意：`.ReturnsForAll<T>()`仅当`NSubstitute`找不到更为具体的返回值配置时才被使用。可以理解为它的优先级别很低，相当于给所有方法都做了一个默认的调用配置**，假设上下文中已经为特定某个方法配置了返回值，实际调用此方法时，其返回值优先于`.ReturnsForAll<T>()`中配置的默认值



## - `reveived calls`

### · `checking`

* **在某些情况下（特别是对于`void`方法），检查`substitute`是否执行过特定的调用(简单来说就是某个方法)是有用的**

* 这需要使用`Reveived()`扩展方法进行检查

* 如下所示

  ```c#
  public interface ICommand {
      void Execute();
      event EventHandler Executed;
  }
  public class SomethingThatNeedsACommand {
      ICommand command;
      public SomethingThatNeedsACommand(ICommand command) { 
          this.command = command;
      }
      public void DoSomething() { command.Execute(); }
      public void DontDoAnything() { }
  }
  [Test]
  public void Should_execute_command() {
      //Arrange
      var command = Substitute.For<ICommand>();
      var something = new SomethingThatNeedsACommand(command);
      //Act
      something.DoSomething();
      //Assert
      command.Received().Execute(); 
  }
  ```

  * 最后的`command.Received().Execute(); `代码断言`Execute()`已执行，如果实际未执行到的话，将会抛出一个`ReceivedCallsException`异常

* 同理，与`Received()`断言某个方法已被执行过，也有`DidNotReceive()`用来断言某个方法未被执行过

* **`Received()`断言某个成员至少进行了一次调用，它还有个重载方法`Reveived(int requiredNumberOfCalls)`来断言对这个成员已经收到特定数量的调用，数量太多太少的话断言都会失败**

  ```c#
  command.Received(3).Execute(); // << This will fail if 2 or 4 calls were received
  ```

***

* **不管是使用`Reveived()`、`Reveived(int requiredNumberOfCalls)`还是说`DidNotReceive()`,后面加某个某方法断言时，其方法参数同样可以使用参数匹配器进行匹配筛选操作的**

  ```c#
  calculator.Add(1, 2);
  calculator.Add(-100, 100);
  //Check received with second arg of 2 and any first arg:
  calculator.Received().Add(Arg.Any<int>(), 2);
  //Check received with first arg less than 0, and second arg of 100:
  calculator.Received().Add(Arg.Is<int>(x => x < 0), 100);
  //Check did not receive a call where second arg is >= 500 and any first arg:
  calculator.DidNotReceive().Add(Arg.Any<int>(), Arg.Is<int>(x => x >= 500));
  ```

* 也可以去忽略使用的参数，只检查成员是否有无被调用,这需要使用到`ReceivedWithAnyArgs()`和`DidNotReceiveWithAnyArgs()`

  ```c#
  calculator.Add(1, 3);
  
  calculator.ReceivedWithAnyArgs().Add(default, default);
  calculator.DidNotReceiveWithAnyArgs().Subtract(default, default);
  calculator.Received().Add(Arg.Any<int>(), Arg.Any<int>());   // <--这种写法也起到忽略参数的作用
  ```

***

* 也可以对`Property`使用相同的方法验证其`setter`/`getter`

  ```c#
  var config = Substitute.For<IConfiguration>();
  config.Setting.Returns("expected value");
  
  // 使用config对象...
  
  // 验证getter是否被调用
  config.Received().Setting;
  
  // 如果需要，也可以验证setter被调用
  config.Received().Setting = Arg.Any<string>();
  ```

  

### · `clearing`

* **可以通过`ClearReceivedCalls()`使`substitute`清除对成员调用的记录**

  ```c#
  command.ClearReceivedCalls();
  ```

* **注意：`ClearReceivedCalls()`用于清除`substitute`上所有方法的接受调用记录，但它不会影响到`Returns()`设置的返回值配置。**这意味着，即时调用`ClearReceivedCalls()`清除了方法调用的记录，当再次调用该方法时，它仍然返回之前通过`Returns()`配置的返回值



## - 回调

* **如果想每当进行特定调用时执行一些任意代码附加操作(比如说日志记录，修改玩不变量等)，可以使用回调实现**

* 对于有返回值的方法，需配合`.AndDoes()`使用，其允许我们在返回指定值的同时，执行一些自定义操作

  ```c#
  var counter = 0;
  calculator.Add(default, default).ReturnsForAnyArgs(x => 0).AndDoes(x => counter++);
  
  calculator.Add(7, 3);
  calculator.Add(2, 2);
  calculator.Add(11, -3);
  Assert.AreEqual(counter, 3)      // true
  ```

* 对于返回类型为`void`的方法，可以使用`When..Do`结构来指定方法被调用时应该执行的操作

  ```c#
  public interface IFoo {
      void SayHello(string to);
  }
  [Test]
  public void SayHello() {
      var counter = 0;
      var foo = Substitute.For<IFoo>();
      foo.When(x => x.SayHello("World")).Do(x => counter++);
  
      foo.SayHello("World");
      foo.SayHello("World");
      Assert.AreEqual(2, counter);   // true
  }
  ```

  * **`When..Do`结构同样可以用于有返回值的方法，但这样的话就设置不了返回值了**

  * 使用`When..Do`结构时，构建`Callback`可以帮助我们创建更为常见的`Do()`场景

    ```c#
    var sub = Substitute.For<ISomething>();
    
    var calls = new List<string>();
    var counter = 0;
    
    sub
      .When(x => x.Something())
      .Do(
        Callback.First(x => calls.Add("1"))       // 第一次调用执行的逻辑
    	    .Then(x => calls.Add("2"))              // 第二次调用执行的逻辑
    	    .Then(x => calls.Add("3"))              // 第三次调用执行的逻辑
    	    .ThenKeepDoing(x => calls.Add("+"))     // 第四次及之后调用执行的逻辑
    	    .AndAlways(x => counter++)              // 每次执行时都需要执行的逻辑
      );
    
    for (int i = 0; i < 5; i++)
    {
      sub.Something();
    }
    Assert.That(String.Concat(calls), Is.EqualTo("123++"));
    Assert.That(counter, Is.EqualTo(5));
    ```

    

## - 抛出异常

* 可以配置在使用函数作为返回值时或使用`When..Do`结构时，可以直接抛出异常

  ```c#
  calculator.Add(-1, -1).Returns(x => { throw new Exception(); });
  calculator.When(x => x.Add(-2, -2)).Do(x => { throw new Exception(); });
  Assert.Throws<Exception>(() => calculator.Add(-1, -1));
  Assert.Throws<Exception>(() => calculator.Add(-2, -2));
  ```

* 下面讲一个比较特殊的例子

  ```c#
  calculator.Add(Arg.Any<int>(), Arg.Any<int>()).Returns(x => { throw new Exception(); });
  
  calculator.Add(1, 2).Returns(3);  //<----实际上测试执行到这的时候会直接抛异常
  
  Assert.Equal(3, calculator.Add(1, 2));  
  Assert.Throws<Exception>(() => calculator.Add(-2, -2));
  ```

  * 这个比较特殊（实际运行测试时会直接报错），因为之前也说道配置是遵循就近原则的，意味着最后设置的规则有最高的优先级，这个原则在大多数情况下使用

  * 但是，**如果我们首先配置了方法对于某些参数时就已经抛出了异常，后续尝试为相同的方法和相同的参数配置一个正常的返回值，这个配置并不会因为"就近原则"而直接生效**

  * **因为在尝试匹配和应用这个具体配置之前，已经有一个全局的异常抛出规则被触发了**

  * **`.Configure()`的使用就是为了解决这个问题的，使用它可以让我们添加或修改配置，而不会受到之前配置的影响（这是通过创建一个新的配置上下文来实现的）**

    ```c3
    calculator.Configure().Add(1, 2).Returns(3);
    ```

    

## - 自动和递归模拟

* **自动模拟是指`Substitute`能够自动创建模拟对象，而不需要显示地为每个依赖想调用模拟创建函数，这在测试对象依赖于多个其他对象时，且每个依赖对象都需要被模拟时非常有用**

* **递归模拟是指当我们模拟一个对象时，`Substitute`会递归式的模拟该对象的所有属性或方法返回的对象**

* 示例如下；

  ```c#
  var service = Substitute.For<IService>();
  service.Client().Name().Returns("John Doe");
  Assert.AreEqual("John Doe", service.Client().Name());
  ```



## - 设定调用顺序

* 有时候对于多个方法，需要按照一定的顺序进行调用（像这样依赖于调用的时间称为时间耦合），`NSubstitute`提供了`Reveived.InOrder()`来让我们实现这个效果

  ```c#
  [Test]
  public void TestCommandRunWhileConnectionIsOpen() {
    var connection = Substitute.For<IConnection>();
    var command = Substitute.For<ICommand>();
    var subject = new Controller(connection, command);
    subject.DoStuff();
    Received.InOrder(() => {
      connection.Open();
      command.Run(connection);
      connection.Close();
    });
  ```

  * **如果实际以不同的顺序运行，则会引发异常**

* **虽然时间耦合在某些应用场景中是必要的，但可以的话，可以尝试减少代码中的时间耦合，这可以通过重构来实现，比如说将这几个方法统一封装到某个方法中，后续我们只需调用这个方法即可，这样也能降低错误的风险**



## - 阻止`void`方法执行原有逻辑

* 对于有返回值类型的方法，我们通常可以通过像调用`.Returns()`来设置返回值，相当于忽略了方法内部具体执行的模拟操作

* 而对于返回值为`void`的方法，则需要通过`When..DoNotCallBase`来实现这个跳过具体实现逻辑的效果

* 如下所示：

  ```c#
  public class MyService
  {
      public virtual void VoidMethod() => Console.WriteLine("实际方法被调用。");
      public virtual int NonVoidMethod() => 42;
  }
  var substitute = Substitute.ForPartsOf<MyService>();
  substitute.When(x => x.VoidMethod()).DoNotCallBase(); //<--这样在后续模拟调用VoidMethod()时就忽略原有的逻辑了
  ```

  * **这里需要用到`Substitute.ForPartsOf<T>()`来配合使用，因为这里模拟了部分方法实现（也就是`NonVoidMethod()`的调用逻辑依旧不变）**

# 注意事项

* 当在模拟框架中使用条件表达式来匹配方法调用的参数时，这些条件表达式可能会抛出异常。这在使用如`Arg.Is<T>(predicate)`这样的匹配器时尤其常见
  * **异常被吞掉（捕获并忽略）**：这意味着异常不会向调用者抛出，测试执行不会因此中断。这样做是为了避免因参数匹配逻辑中的错误而导致测试失败，使得测试结果更加稳定和可预
  * **参数不匹配**：当条件执行引发异常时，模拟框架将假定该特定的参数匹配失败。也就是说，如果你配置了模拟对象在满足特定条件时返回某个值，但是这个条件评估时抛出了异常，那么框架会认为当前的方法调用不满足这个条件，因此不会应用这个特定的返回值配置，**常会返回该方法返回类型的默认值**

* `Substitute.For<T>()`：**用于创建一个完全的模拟，这意味着所有的方法，无论是`void`还是返回某个值的方法，都不会执行其原始实现，相反，所有方法调用默认都是空操作，将返回返回类型的默认值，直到我们为它们配置特定的行为**
* `Substitute.ForPartsOf<T>()`：**用于创建一个部分替代，允许我们选择性地模拟某些方法，而保留类中其他方法的原始实现。此时如果我们没有为这个方法配置模拟行为，那么调用该方法将执行其原始实现**（也就意味这个`T`应该是一个具体的类）
* **`Nsubstitute`不支持直接被`sealed`修饰的类(强行模拟测试时会直接报错)，这是因为`sealed`类不允许被继承，当涉及到`sealed`类的测试代码时，建议的做法为使用包装器模式来提供一个可以模拟的间接层**
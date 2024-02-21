# 基概

## - 引入

* 想象这样一个应用场景，假设我们现在对引入的第三方类库或者说某个框架的类`A`进行扩展操作的话
* 常用的解决方案无非就是通过继承类`A`，在子类中构造新的方法扩展
* 或者使用装饰器模式进行扩展，更甚至如果可以的话通过改动源代码来扩展
* 但这几种方式都带有一定的局限性，比如说无法改动第三方库的类，此类被`sealed`修饰无法被继承，使用装饰器模式进行扩展时显得有些繁琐等等
* 这时候`Extension Method`就闪亮登场了

## - 介绍

* `Extension Method`是`C#`中的一个强大特性
* **它能够使我们向现有类型"添加"方法，而无需创建新的派生类型、重新编译或以其他方式修改原始类型**
* **扩展方法本质上是一种静态方法，但是它可以像代扩展类型调用其实例方法的形式一样进行调用**
* 对于使用`C#`、`F#`和`Visual Basic`编写的客户端代码，调用扩展方法和调用实例方法并没有明显的区别

# 定义&使用

* 定义扩展方法非常简单，只需遵循以下几个规则：
  * **静态类：扩展方法必须定义在静态类中**
  * **静态方法：扩展方法本身必须是`static`修饰的**
  * **`this`关键字：扩展方法的第一个参数必须为代扩展的类型，且必须用`this`关键字进行修饰（这个参数称为"扩展类型参数"，它指明了该方法可以被哪种类型的实例调用）**

* 下述以扩展`string`类型为例，为`string`类型扩展一个经过`SHA256`哈希的方法

  ```c#
  public static class CryptographyUtil
  {
      public static string ToSha256(this string input)
      {
          var sha256 = SHA256.Create();
  
          return Convert.ToHexString(sha256.ComputeHash(Encoding.UTF8.GetBytes(input)));
      }
  }
  ```

* 使用起来也特别简单，`string`类型的对象像正常调用实例方法的形式即可

  ```c#
  var password = "123456";
  var encryptedPassword = password.ToSha256();
  ```

  * **这里需要引入扩展方法`toSha256()`所在的`namespace`，以至于让编译器知道在哪里查找这些扩展方法**

# 常见使用模式

## - 集合

* **对于`int[]`数组、`List<int>`列表或者其他类型的集合，由于它们统一实现`System.Collections.Generic.IEnumerable<T>`接口，因此我们可以就`System.Collections.Generic.IEnumerable<T>`类型进行扩展，从而达到了`int[]` 数组，`List<int> `列表或其他类型的集合都能共享这同一扩展方法，无需为每种类型单独编写扩展方法**

  ```c#
  // 示例
  var ints = new int[] { 1, 2, 3 };
  var list = new List<int>() { 4, 5, 6 };
  ints.ExtensionForInt();
  list.ExtensionForInt();
  
  public static class Extensions
  {
      public static void ExtensionForInt(this IEnumerable<int> input)
      {
          Console.WriteLine("Extension Method");
      }
  }
  ```

* Ps：元素类型为int，就要对`System.Collections.Generic.IEnumerable<int>`类型进行扩展；元素类型为`string`，就要对`System.Collections.Generic.IEnumerable<string>`类型进行扩展，以此类推	



## - 特定于层

* 在我们的应用程序中，通常存在一组域实体或者数据传输对象，用于在不同层之间传递数据或进行通信

* 比如说`Web MVC`场景中，常见的`Controller`层与`UI`层之间的`DTO`，以及映射数据库表的`entity`

* **这些类通常是简单的数据结构，不包含任何行为或只包含与应用程序的所有层相关的最基本功能（这样做的目的是确保这些类的职责单一，使其更容易被理解和使用）**

* 然而，在应用程序的不同层中，可能需要特定于该层的功能，比如说`Controller`层需要进行数据转化或格式化，在`Service`层需要进行一些业务逻辑处理

* 在这种情况下，可以使用`Extension Method`为这些类进行扩展，添加特定于某一层的功能，而无需将这些方法写入到这些类中（保持了这些类的简单性和职责单一性）

* 同时使得代码更加模块化，可扩展和易于维护

* Demo：

  ```c#
  public class DomainEntity
  {
      public int Id { get; set; }
      public string FirstName { get; set; }
      public string LastName { get; set; }
  }
  
  static class DomainEntityExtensions
  {
      static string FullName(this DomainEntity value)
          => $"{value.FirstName} {value.LastName}";
  }
  ```



## - 扩展`Predefined types`

* `Predefined types`，预定义类型是`.NET`框架中提供的一组数据类型，它包含了值类型和引用类型

* 扩展预定义类型通常是对一些值类型或者受`struct`约束的泛型类型进行扩展

* 这需要通过`ref`关键字一起配合，示例如下：

  ```c3
  public static class IntExtensions
  {
      public static void Increment(ref this int number)
      {
          number++;
      }
  }
  
  class Program
  {
      static void Main(string[] args)
      {
          int num = 5;
          Console.WriteLine("Original value: " + num); // 输出：Original value: 5 
          num.Increment(); // 调用 ref 扩展方法
          Console.WriteLine("After increment: " + num); // 输出：After increment: 6
      }
  ```

  * **`ref`关键字将第一个参数声明为引用参数，使其成为`ref`扩展方法**
  * **`ref`关键字可以放在`this`关键字之前或之后，两者没有语义上的差别**
  * **`ref`关键字将第一个参数标记为按引用传递，这意味着方法内部对该参数的任何更改都会反映到原始变量上**

# 注意事项

* **当类A被`sealed`关键字修饰，也就是说类A不能被子类继承，但这并不影响我们使用`Extension Method`来扩展类A，这也是`Extension Method`的一个重要优势，它提供了一种向不可继承的类添加新方法的灵活方式，这种机制特别适用于为第三方库的类增加功能**
* **扩展方法不能覆盖代扩展类型现有的方法，如果扩展方法和实例方法具有相同的签名，实例方法将具有优先权，此扩展方法将永远不会被调用（这是因为编译器在解析方法调用时，会首先查找该类型的实例方法，找不到才会考虑找扩展方法）--so设计扩展方法时，应避免与现有实例方法签名冲突，以避免造成意外的行为**
* **扩展方法的调用看起来像是对对象实例的实例方法的调用，但在编译时，它们会将处理为静态方法调用**
* **扩展方法可以访问它扩展类型的公共成员，但不能访问私有或受保护的成员**




# 基概



# 使用



# 注意事项









* 表达式树是一种特殊的数据结构，它以树的形式组织和存储表达式的各个组成部分。每个节点代表表达式的一个元素，如一个常量、一个变量、一个运算符或一个方法调用

* *表达式树*是表示一些代码的数据结构。 它不是经过编译且可执行的代码。 如果想要执行由表达式树表示的 .NET 代码，必须将其转换为可执行的 IL 指令
* Lambda 表达式是你可通过转换为可执行的中间语言 (IL) 来执行的唯一表达式类型



* Expression有两个属性
  * NodeType：表示表达式节点的类型，c#一共定义了85种
  * Type：这个表达式的静态类型，也就是其返回类型



* 表达式树可以用于序列化和反序列化表达式：由于表达式树是代码的数据结构表示，你可以将其序列化为二进制或文本格式，然后在另一个上下文（甚至在另一个进程或机器）中反序列化并执行。这对于远程过程调用（RPC）和分布式计算等场景非常有用





* 表达式树中的 lambda 表达式通常需要符合以下要求：
  1. **简单表达式**：lambda 表达式应该是由传入参数和返回参数组成的简单表达式。这意味着它通常不能包含复杂的逻辑，如条件判断（`if`）、循环（`for`、`while` 等）、异常处理（`try-catch`）等。
  2. **不含副作用**：在表达式树中的 lambda 表达式中执行的操作应该是无副作用的。这意味着它不应该修改外部状态，如修改全局变量、执行 I/O 操作等。

​	这些限制的主要原因是表达式树的设计目的是为了在运行时分析和处理表达式。复杂的逻辑和副作用会使得这种分析和处理变得困难。例如，当使用表达式树来构建 LINQ 查询时，查询提供者需要能够将表达式树转换为相应的查询语言（如 SQL），如果 lambda 表达式包含复杂的控制流，这种转换将变得非常困难甚至不可能





* ```
  var ageProperty = Expression.Property(personParameter, nameof(Person.Age));
  ```

  生成了一个 `MemberExpression` 类型的表达式，它代表访问 `personParameter` 参数表达式所代表的 `Person` 类型的 `Age` 属性

* 当你使用 `Expression.Property` 创建一个表示属性访问的表达式时，表达式的类型会自动推断为该属性的类型

  ```c#
  MemberExpression (property)
      |
      |-- ParameterExpression (parameter)
      |       |
      |       |-- Type: Person
      |
      |-- MemberInfo (Age)
              |
              |-- Type: int
  ```

  `MemberInfo`（在这个例子中是 `Age`）不是一个子节点，而是该节点内的一些信息。`MemberExpression` 节点本身代表对成员（属性、字段或方法）的访问，而 `MemberInfo` 提供了关于被访问成员的具体信息，比如成员的名称、类型和所属类型等。



* 用法
  * `Expression.Condition` 方法用于创建一个表示条件表达式（三元运算符 `?:`）的表达式树节点。这个方法接受三个参数：
    1. `test`：一个 `Expression`，表示条件表达式的测试部分。它应该是一个返回布尔值的表达式。
    2. `ifTrue`：一个 `Expression`，表示当 `test` 的结果为 `true` 时要执行的表达式。
    3. `ifFalse`：一个 `Expression`，表示当 `test` 的结果为 `false` 时要执行的表达式
  * 当你使用 `Expression.Add` 方法来创建一个表示加法运算的表达式时，你可以通过提供一个额外的参数（`MethodInfo` 对象）来指定一个自定义的加法逻辑

# ？？？

* 在.NET中，表达式树和反射都可以用来在运行时动态地生成和执行代码。然而，表达式树提供了一种在执行效率和代码清晰度方面更优的选择
* 表达式树的主要优点在于它们可以在运行时生成和编译



* 表达式树构造需要满足什么条件

#  编写点

* 以图结合代码的形式去推理出树
* 以反射为例子作为对比
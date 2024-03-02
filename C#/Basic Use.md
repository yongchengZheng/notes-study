1. 对于`person.Name`和**`person!.Name`**的区别

   ```c#
   Person? person = null;
   ```

   * 因为`person`是可能为null的
   * 那么在**调用`person.Name`时，编译器会警告你`person`可能为`null`从而引发空指针异常**
   * 而在调用`person!.Name时`，则**使用了`null`否定运算符(`!.`)来告诉编译器忽略空检查**，也就是说编译器不会在这里警告你了，**但是实际运行时`person`还是可能为`null`从而引发空指针异常**
   * <b style="color">so所以正常来说当使用这个空合并赋值运算符时，开发者是确定当前上下文环境中不会为`null`，即使从类型的声明来看它可以是为`null`的</b>

2. 对于`person.Name`和**`person?.Name`**的区别

   ```c#
   Person? person = null;
   ```

   * 由于`person`是可能为`null`的
   * 所以**调用`person.Name`，编译器会警告你`person`可能为`null`，运行时若真为`null`时会直接报空指针异常**
   * 而在调用`person?.Name`时，**则使用了`null`条件运算符(`?.`)，这是一种安全地访问可能为`null`的对象成员方式，如果`person`为`null`的话，整个表达式的结果就是为`null`，而不会尝试访问`Name`属性，从而避免了在运行时引发空指针异常**


wrwer









## 重点

* **若数据库表中没有设置主键信息，则Entity Framework Core 将使用默认约定，嘉定第一个属性是主键**

* **删除记录的时候，先要把对应数据给查出来，然后再调用Remove方法去删。（直接通过new一个对象再去删除是报错的，因为EF Core需要再内部跟踪对象的状态）**
  * 删除多个时同理，先根据条件把所有带删除数据显查出去，然后在遍历去删
* 更新操作同理，也是先查出来对应数据，然后改完值后再更新到数据库中去。

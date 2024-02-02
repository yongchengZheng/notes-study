# 基概

* `EFCore`是一个开源的、轻量级、可扩展的对象关系映射(`ORM`)框架

# 使用

* 以使用`MySQL`数据库为例

* 对于不同的数据库，`EFCore`需要使用不同的包，以下是常见的示例

  |         数据库          |                   配置实例                   |                           NuGet包                            |
  | :---------------------: | :------------------------------------------: | :----------------------------------------------------------: |
  | SQL Server 或 Azure SQL |      `.UseSqlServer(connectionString)`       | [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/) |
  |     Azure Cosmos DB     | `.UseCosmos(connectionString, databaseName)` | [Microsoft.EntityFrameworkCore.Cosmos](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Cosmos/) |
  |         SQLite          |        `.UseSqlite(connectionString)`        | [Microsoft.EntityFrameworkCore.Sqlite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite/) |
  |  EF Core 内存中数据库   |     `.UseInMemoryDatabase(databaseName)`     | [Microsoft.EntityFrameworkCore.InMemory](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory/) |
  |       PostgreSQL*       |        `.UseNpgsql(connectionString)`        | [Npgsql.EntityFrameworkCore.PostgreSQL](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL/) |
  |     MySQL/MariaDB*      |        `.UseMySql(connectionString)`         | [Pomelo.EntityFrameworkCore.MySql](https://www.nuget.org/packages/Pomelo.EntityFrameworkCore.MySql/) |
  |         Oracle*         |        `.UseOracle(connectionString)`        | [Oracle.EntityFrameworkCore](https://www.nuget.org/packages/Oracle.EntityFrameworkCore/) |

  

## - 上手Demo

1. 通过`NuGet`导入对应包

   ```c#
   Pomelo.EntityFrameworkCore.MySql
   ```

2. 创建对应实体类`Person`

3. 构建继承自`DbContext`的自定义类

   ```c#
   public class ApplicationContext : DbContext
   {
       public DbSet<Person> Persons { get; set; }
       // 重写这个方法做配置
       protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
       {
           // 配置 MySQL 数据库连接字符串
           optionsBuilder.UseMySql("Server=localhost;Database=demo;User=root;Password=123456;", 
               new MySqlServerVersion(new Version(8, 0, 23)))
               .LogTo(Console.WriteLine, LogLevel.Information)
               .EnableSensitiveDataLogging()
               .EnableDetailedErrors();
       }
   }
   ```

4. 使用

   ```c#
   using (var context = new ApplicationContext())
   {
      // 编写逻辑
   }
   ```

   * **使用`Using`语句是为了使用完`context`后自动释放，这可确保释放所有非托管资源，并注销任何事件或其他挂钩，以防止在实例保持引用时出现内存泄漏**
   * **`DbContext`不是线程安全的， 不要在线程之间共享上下文**



* 在构建自定义`DbContext`时除了重写`OnConfiguring()`做配置这种方式，还支持给构造函数传入一个`DbContextOptions`值来配置，

  * 如下所示

    ```c#
    public class ApplicationContext : DbContext
    {
        public DbSet<Person> Persons { get; set; }
        // 将参数传给基类来使用
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }
    }
    ```

  * 那么在构造`ApplicationContext`时就要传入这个参数，如下所示

    ```c#
    var contextOptions = new DbContextOptionsBuilder<BloggingContext>()
        .UseMySql("Server=localhost;Database=demo;User=root;Password=123456;",
            new MySqlServerVersion(new Version(8, 0, 23))).Options;
    using (var context = new ApplicationContext(contextOptions))
    {
       // 编写逻辑
    }
    ```



## - 整合`ASP.NET Core`

1. 通过`NuGet`导入依赖包

2. 构建实体类`Person`

3. 构造继承`DbContext`的自定义类

   ```c#
   public class PractiseForZeroDbContext : DbContext
   {
     
       public DbSet<Person> Persons { get; set;}
     
       private readonly string _connectString;
       
       public PractiseForZeroDbContext(string connectString)
       {
           _connectString = connectString;
       }
   
       protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
       {
           optionsBuilder.UseMySql(_connectString, new MySqlServerVersion(new Version(8, 0, 23)));
       }
   }
   ```

4. 进行配置

   ```c3
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddDbContextFactory<ApplicationDbContext>(
           options =>
               options.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test"));
   }
   ```

5. 后续即可直接注入`PractiseForZeroDbContext`进行使用

# 配置



## - 配置点

* 对于`DbContext`来说，所有配置的起点都是`DbContextOptionsBuilder`

* 下面是其调用的常用方法示例

  |                             方法                             |                   作用                   |
  | :----------------------------------------------------------: | :--------------------------------------: |
  | [UseQueryTrackingBehavior](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder.usequerytrackingbehavior) |          设置查询的默认跟踪行为          |
  | [LogTo](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder.logto) |     获取 EF Core 日志的一种简单方法      |
  | [UseLoggerFactory](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder.useloggerfactory) | 注册 `Microsoft.Extensions.Logging` 工厂 |
  | [EnableSensitiveDataLogging](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder.enablesensitivedatalogging) |    在异常和日志记录中包括应用程序数据    |
  | [EnableDetailedErrors](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder.enabledetailederrors) |     更详细的查询错误（以性能为代价）     |
  | [ConfigureWarnings](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder.configurewarnings) |         忽略或引发警告和其他事件         |
  | [AddInterceptors](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.dbcontextoptionsbuilder.addinterceptors) |           注册 EF Core 侦听器            |
  | [UseLazyLoadingProxies](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.proxiesextensions.uselazyloadingproxies) |         使用动态代理进行延迟加载         |
  | [UseChangeTrackingProxies](https://learn.microsoft.com/zh-cn/dotnet/api/microsoft.entityframeworkcore.proxiesextensions.usechangetrackingproxies) |         使用动态代理进行更改跟踪         |

  

* 可以通过下面三种方式来获取`DbContextOptionsBuilder`从而进行配置

  * 在继承自`DbContext`的自定义配置类中重写方法`OnConfiguring()`

    ```c#
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            // 相关配置
        }
    ```

  * 通过`new`显示构造

    ```c#
    var contextOptions = new DbContextOptionsBuilder<BloggingContext>()
        .UseMySql("Server=localhost;Database=demo;User=root;Password=123456;",
            new MySqlServerVersion(new Version(8, 0, 23))).Options;
    var context = new BloggingContext(contextOptions);
    ```

  * 容器进行配置时在`AddDbContext()`中获取

    ```c#
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddDbContextFactory<ApplicationDbContext>(
            options =>    // 这个options就是
                options.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Test"));
    }
    ```

    

* 注意：
  * **每个`DbContext`实例都必须配置使用一个且仅一个数据库提供**
  * **而对于不同的实例可用与不同的数据库提供程序**



## - `DbContext`池

* 这里用到的是经典的池化思想，当开启这个特性后

* **`EFCore`会维护一个装载了`DbContext`实例的池子，每次请求`DbContext`实例时，`EFCore`都会尝试从池子里面拿取一个现有的实例而不是每次都创建一个新的实例，这请求用完此实例并释放后，此实例不是被销毁，而是重置放进池子里等待将来重用**

* <b style="color:red">也就是说`DbContext`池允许应用程序在启动时一次性支付池化的成本，而不是后续持续性为每个请求付费</b>

* 配置：

  * 具有依赖关系注入时：

    ```c#
    builder.Services.AddDbContextPool<WeatherForecastContext>(         //  <---指明使用DbContextPool
        o => o.UseSqlServer(builder.Configuration.GetConnectionString("WeatherForecastContext")));
    ```

  * 没有依赖关系注入时：

    ```c#
    var options = new DbContextOptionsBuilder<PooledBloggingContext>()
        .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True")
        .Options;
    var factory = new PooledDbContextFactory<PooledBloggingContext>(options);// <--构建池化工厂
    using (var context = factory.CreateDbContext())
    {
        // 使用
    }
    ```

* **池化思想的目的在于实例复用，`DbContext`实例在多个请求中被重用，后续重置状态返回池中，但最好避免在`DbContext`实例中存储任何夸请求可能变化的状态**



## - 加载



### · 预加载

* **这是一种在查询主实体时同时加载相关实体的数据加载策略**

* 常见的`1-1`、`1-n`关系中，我们是在在主体中额外添加属性来映射着这个关系

* 比如说`Student`可以有多个`Course`，于是我们在`Student`实体表中添加属性`List<Course>`(**这个叫导航属性**)来进行关联

* 而预加载达到的效果就是在单个查询`Student`数据时，其关联的`List<Coures>`数据也会查询出来
* 以避免后续拿着`Student`的`Id`又单独的进行查询一遍`Course`表

* 使用

  * **调用`.Include()`方法预加载单个导航属性**

    ```c#
    _context.Students.Include(s => s.CourseList).ToList()
    ```

  * **调用`.ThenInclude()`方法预加载链式导航属性**(A中有B，B中有C这种嵌套)

    ```c#
    _context.Students.Include(s => s.CourseList).ThenInclude(c = >c.Teacher)
    ```

  * **连续调用`.Include()`方法预加载同一实体的多个导航属性**

    ````c#
    _context.Students.Include(s => s.CourseList).Include(s => s.Adress)
    ````

* 其功能原理为，**在使用预加载时，`EFCore`会生成一个单一的`SQL`查询，这个查询通过`JOIN`语句或子查询来包含主实体和其相关实体的数据**
* <b style="color:red">这种策略相当于把原本多次与数据库的交互合成一次来做，减少与数据库的访问次数和潜在的性能开销，常用来优化数据库访问，尤其是在知道需要访问关联数据的情况下</b>
* 注意：
  * 需小心注意`A`引用`B`，`B`引用`A`这种循环引用情况，避免产生无限循环和性能问题
  * 虽然预加载可以减少与数据库的访问次数，但它也可能导致检索大量不必要的数据，使用方面得结合实际应用场景



### · 显示加载

* 显示加载的模式与我们正常查询数据的逻辑相同

* **在首次查询并加载一个实体时，其相关的导航属性不回自动加载，相反，需要后续编写额外的代码来明确加载这些关联数据**

* 而在`EFCore`中是通过需要使用**特定的`API`调用**来加载与实体相关联的数据

* 使用：

  * **对于`1-1`，调用`.Reference()`方法**

    ```c#
    var student = _context.Students.SingleOrDefault(s => s.Id = xxx);
    // 显示加载
    _context.Entry(student).Reference(s => s.Address).Load();
    ```

  * **对于`1-n`，调用`.Collection()`方法**

    ```c#
    var student = _context.Students.SingleOrDefault(s => s.Id = xxx);
    // 显示加载
    _context.Entry(student).Collection(s => s.CourseList).Load();
    ```

* **显而易见，这种方式符合按需加载的特性，允许更细粒度的控制**

* 但是每次调用`load`方法，都会到数据库中执行一个新的查询，**去往数据库的请求数量可以理解为相当于至少是去往这个接口请求数量的`double`**，使用方面仍需考虑实际应用场景



### · 懒加载

* 懒加载的思想与显示加载相似，同样是在查询实体数据时，不回自动加载关联数据，而在后续**首次**访问导航属性时，`EFCore`才会自动执行必要的数据库查询来加载关联数据

* 它与显示加载的不同之处，**在于显示加载在加载实体数据的导航属性时，需要手动调用`API`来进行加载操作，而懒加载只需要做好配置后即可**

* 其工作原理依赖于实体类导航属性的代理实现，**当使用懒加载时，`EFCore`会创建实例类的代理实例，这些代理覆盖了虚拟导航属性，当首次访问这些属性时就会进行拦截，通过执行数据库查询以加载关联数据**

* 懒加载的启用比较特殊，需要满足以下这些条件

  * **需要安装支持懒加载的代理库**，如

    ```c#
    Microsoft.EntityFrameworkCore.Proxies
    ```

  * **导航属性必须被标记为`virtual`**，如

    ```c#
    public class Student
    {
        public int Id { get; set; }
        public string Name { get; set; }
        // 导航属性标记为 virtual
        public virtual List<Course> CourseList { get; set; }
    }
    ```

  * **在配置`DbContext`时，需要开启懒加载代理**

    ```c#
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseLazyLoadingProxies().UseYourDataProvider(options => ...);
    }
    ```

* **与显示加载同理，懒加载也具有按需加载的特性，并且相较于显示加载，只需配置而无需编写额外的代码来加载相关数据，`EFCore`会自动管理，开发方面要简化很多**
* 也要注意到当实体类具有多个导航属性时，使用懒加载在加载这些关联数据时，每个导航属性的加载都会导致一次与数据库的交互，这时候对于数据库来说是个潜在的小高峰请求隐患



## - 跟踪与非跟踪查询

* **所谓跟踪查询，就是`EFCore`对返回实体类型的查询动作默认时跟踪的**

* **这个跟踪意味着这些实例后续的任何更改在调用`SaveChanges()`方法后多会同步到数据库中保存**

* 而非跟踪查询相反，其查询出来的数据后续的任何更改在调用`SaveChanges()`方法后也不会永久持久化道数据库中

* 与跟踪查询相比，其优势在于其性能，毕竟跟踪查询需要`EFCore`耗费内存来实现此特性

* **因此通常用于只读操作**

* 需要开启非跟踪操作，按以下配置即可

  ```c#
  var blogs = context.Blogs
      .AsNoTracking()         // <---
      .ToList();
  // 或者
  protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
  {
      optionsBuilder
          .UseSqlServer(connectString)
          .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);   // <----
  }
  ```



## - 日志

* 日志记录的配置通常是由配置文件`appsettings.json`以及环境特定版本`appsettings.{Environment}.json`的`Logging`部分提供

* 要记录`SQL`具体情况，需添加`key`为`Microsoft.EntityFrameworkCore.Database.Command`的配置，如下所示

  ```c#
  "logging": {
      "logLevel": {
        "Default": "Information",
        "Microsoft.AspNetCore": "Warning",
        "Microsoft.EntityFrameworkCore.Database.Command": "Debug"
      }
    }
  ```


# 模型



## - 模型配置方式



### ·`fluent API`

* 这是`EFCore`用于配置模型和映射的一种编程风格，**它使用链式方法调用来构建对象和执行操作**

* 对于模型的配置，通常也是位于继承自`DbContext`的自定义类中的`OnModelCreating()`方法来进行配置的

  ```c#
  
  ublc class MyContext : DbContext
  {
      public DbSet<Blog> Blogs { get; set; }
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
        // 通过fluent api进行模型配置
          modelBuilder.Entity<Blog>()
              .Property(b => b.Url)
              .IsRequired();
      }
  }
  ```



* 当模型配置量过多时，可将配置提取到实现`IEntityTypeCOnfiguration<TEntity>`的单独类中进行配置，如下所示

  ```c#
  public class BlogEntityTypeConfiguration : IEntityTypeConfiguration<Blog>
  {
      public void Configure(EntityTypeBuilder<Blog> builder)
      {
          builder
              .Property(b => b.Url)
              .IsRequired();
      }
  }
  ```

* 然后在`OnModelCreating()`中构造实例调用`Configure()`方法进行加载(如有多个，逐一调用即可)

  ```c#
  new BlogEntityTypeConfiguration().Configure(modelBuilder.Entity<Blog>());
  new XXXEntityTypeConfiguration().Configure(modelBuilder.Entity<XXX>());
  ```

* 实际上这种方式无非就是把配置封装到某个类而已，通过参数把对应`EntityTypeBuilder<TEntity>`传进来使用

* 至于为什么还要实现`IEntityTypeCOnfiguration<TEntity>`接口

* 是因为对于多个实现了此接口的类的话，`EFCore`提供了以下方式一次性加载(也就是不用说逐一调用加载)

  ```c#
  modelBuilder.ApplyConfigurationsFromAssembly(typeof(BlogEntityTypeConfiguration).Assembly);
  ```

  * **应用配置的顺序是不确定的，因此仅当顺序不重要时才应使用此方法**



### ·  数据注释

* 实际上，使用数据注释会具有更好的可读性(通常也是采用这种方式)

* 对于刚刚上述实现了`IEntityTypeCOnfiguration<TEntity>`接口的类，可通过直接在实体类上加入注释`[EntityTypeConfiguration]`进行引入配置

  ```c#
  [EntityTypeConfiguration(typeof(BookConfiguration))]
  public class Book
  {
      public int Id { get; set; }
      public string Title { get; set; }
      public string Isbn { get; set; }
  }
  ```

* <b style="color:red">注意：使用`fluent API`方式的配置优先级别会跟高，也就是它会取代数据注释中的同样配置</b>



## - 实体映射

### · 默认情况

* `DbSet`为每个实体集创建一个属性

  * 实体集通常对应于数据库表
  * 一个实体对应于表中的一行

* 默认情况下

  * **在使用`DbSet<T>`属性在`DbContext`类中声明实体时，默认`DbSet<T>`的属性名称用作数据中表的名称**

    ```c#
        public DbSet<Product> Products { get; set; }   // 映射的表名为Prodcut
    ```

    * **如果某个实体类在`DbContext`中没有对应的`DbSet<T>`属性声明，则默认使用实体类的名称作为数据库表的名称**

    * **可以通过属性`[Table("product")]`加到实体类上指定对应的表名，这也是最常用的方式**

    * **也可以通过在配置`DbContext`时通过在`OnModelCreating`方法进行配置指定实体类映射的表名称**

      ```c#
      protected override void OnModelCreating(ModelBuilder modelBuilder)
          {
              modelBuilder.Entity<Product>().ToTable("product");
          }
      ```

    * 以上优先级逐层递增，也就是在`OnModelCreating()`方法中进行配置的优先级别最高

  * **实体属性名称用于列名称**

  * **被命名`ID`或`classnameID`被识别为`PK`**

    * 可通过以下方式显示指定

      ```c#
      public class Car
      {
          [Key]                                        // <---
          public string LicensePlate { get; set; }
      }
      // 或者
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
          modelBuilder.Entity<Car>().HasKey(c => c.LicensePlate);
      }
      ```

    * 对于多个属性组合成的主键，可以通过数据注释`[PrimaryKey(nameof(XXX), nameof(XXXX))]`或者`.HasKey(c => new { c.XXXX, c.XXXX }); `来指定配置

  * **实体框架假定主键值由数据库生成**



### · 使用

* 从模型中排除某一个类型

  * 数据注释方式：

    ```c#
    [NotMapped]
    public class BlogMetadata
    {
        public DateTime LoadedFromDatabase { get; set; }
    }
    ```

  * `Fluent API`方式：

    ```c#
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Ignore<BlogMetadata>();
    }
    ```

* 映射表名

  * 数据注释方式：

    ```c#
    [Table("blogs")]  //  <-----
    public class Blog
    {
        public int BlogId { get; set; }
    }
    ```

  * `Fluent API`方式

    ```c#
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>().ToTable("blogs");
    }
    ```

* 映射列名

  * 数据注释方式

    ```c#
    public class Blog
    {
        [Column("blog_id")]
        public int BlogId { get; set; }
    }
    ```

  * `Fluent API`方式

    ```c#
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>().Property(b => b.BlogId)
            .HasColumnName("blog_id");      // <-----
    }
    ```

* 映射视图：

  ```c#
  modelBuilder.Entity<Blog>().ToView("blogsView", schema: "blogging");
  ```

* 映射表值函数：(也就是说实体类映射到的不是数据库中的表，而是表值函数)

  ```c#
  modelBuilder.Entity<BlogWithMultiplePosts>().HasNoKey().ToFunction("BlogsWithMultiplePosts");
  ```

  * **若要将实体映射到表值函数，函数必须是无参数的**

* 表，列注释：

  * 数据注释方式：

    ```c#
    [Comment("Blogs managed on the website")]
    public class Blog
    {
      [Comment("Blogs Id")]
        public int BlogId { get; set; }
    }
    ```

  * `Fluent API`方式：

    ```c#
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>().ToTable(
            tableBuilder => tableBuilder.HasComment("Blogs managed on the website"));
    }
    ```

* 排除某个属性：**默认情况下，所有具有`Getter`和`Setter`的公共属性都包含在模型中**

  * 数据注释方式：

    ```c#
    public class Blog
    {
        public int BlogId { get; set; }
        [NotMapped]    // <----
        public DateTime LoadedFromDatabase { get; set; }
    }
    ```

  * `Fluent API`方式：

    ```c#
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>().Ignore(b => b.LoadedFromDatabase);
    }
    ```

* 列数据类型：**使用关系型数据库时，数据库提供程序会根据属性的`.NET`类型选择数据类型**，除此之外，它还参考属性的一些元属性

  * 配置类型：

    * 数据注释方式：

      ```c#
      public class Blog
      {
          public int BlogId { get; set; }
          [Column(TypeName = "varchar(200)")]
          public string Url { get; set; }
          [Column(TypeName = "decimal(5, 2)")]
          public decimal Rating { get; set; }
      }
      ```

    * `Fluent API`方式：

      ```c#
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
          modelBuilder.Entity<Blog>(
              eb =>
              {
                  eb.Property(b => b.Url).HasColumnType("varchar(200)");
                  eb.Property(b => b.Rating).HasColumnType("decimal(5, 2)");
              });
      }
      ```

  * 配置最大长度：

    * 数据注释方式：

      ```c#
      public class Blog
      {
          public int BlogId { get; set; }
          [MaxLength(500)]                  //  <-----
          public string Url { get; set; }
      }
      ```

    * `Fluent API`方式：

      ```c#
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
          modelBuilder.Entity<Blog>().Property(b => b.Url)
              .HasMaxLength(500);                     //  <----
      }
      ```

  * 配置精度和小数位数：

    * 数据注释方式：

      ```c#
      public class Blog
      {
          public int BlogId { get; set; }
          [Precision(14, 2)]
          public decimal Score { get; set; }
          [Precision(3)]
          public DateTime LastUpdated { get; set; }
      }
      ```

    * `Fluent API`方式：

      ```c#
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
          modelBuilder.Entity<Blog>().Property(b => b.Score)
              .HasPrecision(14, 2);
          modelBuilder.Entity<Blog>().Property(b => b.LastUpdated)
              .HasPrecision(3);
      }
      ```

  * 配置为必需的:

    * 数据注释方式：

      ```c#
      public class Blog
      {
          public int BlogId { get; set; }
          [Required]
          public string Url { get; set; }
      }
      ```

    * `Fluent API`方式：

      ```c#
      protected override void OnModelCreating(ModelBuilder modelBuilder)
      {
          modelBuilder.Entity<Blog>().Property(b => b.Url)
              .IsRequired();           // <---
      }
      ```

      

### · 索引

* 而`EFCore`提供了一种方式，它允许我们在代码上进行索引的一些配置，从而将这些索引的配置同步到数据库中生效

* 但是正常来说我们的操作都是在创建表时就已经直接指定哪些字段设为属性，或者后续通过`DbUp`框架进行数据库的迁移

* 所以说`EFCore`提供的这个索引配置了解即可

* 使用

  * 数据注释方式：**在实体类上加`[Index]`即可**

    ```c#
    [Index(nameof(Url))]          // 唯一索引的话就是--->.  [[Index(nameof(Url), IsUnique = true)]] 
    															// 复合索引的话就是--->   [Index(nameof(xxx),nameof(xxx))]
    public class Blog
    {
        public int BlogId { get; set; }
        public string Url { get; set; }
    }
    ```

  * `Fluent API`方式

    ```c#
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .HasIndex(b => b.Url);      // 唯一索引的话后续调用 ---> .IsUnique();
      																// 复合索引的话就是--> .HashIndex(p = > new {p.FirstName, p.LastName})
    }
    ```



## - 值转换

* 对于实体类的属性与数据库表中的字段，由于数据库和编程语言对于某种类型的定义可能略有区分，或者他们之间进行存储时我们需要一些额外的处理，所以如何将它们之间的值转换就是值转换器要做的内容

* 在`EFCore`的定义中，实体类的字段叫做`ModelClrType`，数据库表中的字段叫做`ProviderClrType`

* 由于需要一个相互之间的转换，所以在配置时也要做一个反向配置，如下所示

  ```c#
  protected override void OnModelCreating(ModelBuilder modelBuilder)
  {
      modelBuilder.Entity<Rider>().Property(e => e.Mount)
          .HasConversion(
              v => v.ToString(),                                 // 实体类属性-->表字段
              v => (EquineBeast)Enum.Parse(typeof(EquineBeast), v));// 表字段--> 实体类属性
  }
  ```

* 有可能这两个类型之间的相互转换时全局应用程序都应用得到的，这时可以把他们的转换规则抽离出来

  ```c#
  public class CurrencyConverter : ValueConverter<Currency, decimal>
  {
      public CurrencyConverter()
          : base(
              v => v.Amount,
              v => new Currency(v))
      {
      }
  }
  protected override void ConfigureConventions(ModelConfigurationBuilder configurationBuilder)
  {
      configurationBuilder
          .Properties<Currency>()
          .HaveConversion<CurrencyConverter>();
   //后续配置其他实体类时可重用这个转换规则
  }
  ```

* `EFCore`内部留有多个内置转换器，[具体参考](https://learn.microsoft.com/zh-cn/ef/core/modeling/value-conversions?tabs=data-annotations)



## - 跟踪实体

* 在`EFCore`中，**跟踪实体是指在其更改跟踪器中保持对实体的监视，以便能够在调用`SaveChanges()`或`SaveChangesAsync()`方法时，自动持久化到数据库中的更改**

* 实体的状态如下所示

  | EntityState |                             描述                             |
  | :---------: | :----------------------------------------------------------: |
  |  Detached   |                     未被`DbCOntext`跟踪                      |
  |    Added    | 此实体为新实体，尚未插入到数据库中，<br />后续调用`SaveChanges()`插入 |
  |  Unchanged  |           此实体从数据库中查询以来尚未进行任何更改           |
  |  Modified   | 此实体从数据库中查询后进行了更改操作，<br />这意味着后续调用`SaveChanges()`时更新 |
  |   Deleted   | 表示实体已标记为删除，这意味这调用`SaveChanges()`时，数据库对应记录会被删除 |

  * **`EFCore`跟踪属性级别的更改，比如说只修改了单个属性值，则数据库更新将近更改改值**

* 以下情况的实体实例会被跟踪

  * **默认情况下，执行查询操作，返回的实体都会被跟踪**
  * **显示地将一个实体实例添加到上下文中**，无论是调用`Add()`、`Attach()`或者`Update()`
  * **当添加或修改一个跟踪的实体，并且这个实体包含对尚未跟踪的新实体的引用时，这些新实体也会被 EF Core 自动检测并开始跟踪**

* 以下情况的实体实例不再被跟踪

  * **`DbContext`被释放**

    * `DbContext`是负责跟踪的主体，它一被释放或者销毁，这意思这所有实体的跟踪信息也会丢失

  * **清除更改跟踪器**

    * `EFCore`通过`ChangeTracker`来管理上下文中实体的跟踪状态，可通过它调用`Clear()`来显示清楚所有跟踪信息

      ```c#
      context.ChangeTracker.Clear(); // 清除所有跟踪信息
      ```

  * **显式拆离实体**

    * 可通过`DbContext`显示地从上下文中拆离一个实体，使其不在被跟踪

      ```c#
      var blog = context.Blogs.FirstOrDefault();
      context.Entry(blog).State = EntityState.Detached; // 将 blog 的状态设置为 Detached，不再被跟踪
      ```

      

## -`SQL`查询

* `EFCore`除了搭配使用`LINQ`进行查询外，还可以使用原生的`SQL`进行查询

* 可通过`.FromSql()`基于`SQL`查询开始`LINQ`查询

  ```c#
  var blogs = context.Blogs
      .FromSql($"SELECT * FROM product")
      .ToList();
  ```

* 可通过`FromSqlInterpolated()`以及字符串内插来构建查询字符串

  ```c#
  var userId = 1;
  var user = dbContext.Users
      .FromSqlInterpolated($"SELECT * FROM Users WHERE UserId = {userId}")
      .FirstOrDefault();
  
  ```

* 如需构造动态`SQL`，则必须使用`.FromSqlRow()`,并用`SqlParameter`将参数封装起来

  ```c#
  var columnName = "Url";
  var columnValue = new SqlParameter("columnValue", "http://SomeURL");
  var blogs = context.Blogs
      .FromSqlRaw($"SELECT * FROM [Blogs] WHERE {columnName} = @columnValue", columnValue)
      .ToList();
  ```

* **可以使用`LINQ`运算符在初始`SQL`查询的基础上进行组合，`EFCore`会将`SQL`视为子查询**

  ```c#
  var searchTerm = "Lorem ipsum";
  var blogs = context.Blogs
      .FromSql($"SELECT * FROM dbo.SearchBlogs({searchTerm})")
      .Where(b => b.Rating > 3)
      .OrderByDescending(b => b.Rating)
      .ToList();
  // 等价于下面SQL
  SELECT [b].[BlogId], [b].[OwnerId], [b].[Rating], [b].[Url]
  FROM (
      SELECT * FROM dbo.SearchBlogs(@p0)
  ) AS [b]
  WHERE [b].[Rating] > 3
  ORDER BY [b].[Rating] DESC
  ```

* 默认情况下使用`SQL`查询也会跟踪结果，如需禁用，同样也是通过`.AsNoTracking()`来更改

  ```c#
  var searchTerm = "Lorem ipsum";
  var blogs = context.Blogs
      .FromSql($"SELECT * FROM dbo.SearchBlogs({searchTerm})")
      .AsNoTracking()    // <----
      .ToList();
  ```



* 对于非查询`SQL`，则可以通过调用`.ExecuteSqlRaw()`/`.ExecuteSqlINterpolated()`执行

  ```c#
  var userId = 1;
  var commandText = "DELETE FROM Users WHERE UserId = {0}";
  dbContext.Database.ExecuteSqlRaw(commandText, userId);
  // 或者
  var userId = 1;
  dbContext.Database.ExecuteSqlInterpolated($"DELETE FROM Users WHERE UserId = {userId}");
  ```

* 当使用 Entity Framework (EF) 添加一个新的实体到数据库时，EF 需要为那些由数据库生成的属性（如主键的自增 ID、时间戳等）提供一个初始值。由于在调用 `SaveChanges()` 方法并将数据实际保存到数据库之前，数据库生成的真实值还不可用，EF 会为这些属性生成一个临时值。这个临时值仅在 EF 的跟踪上下文中使用，用于在保存更改之前在内存中标识和跟踪实体。

  ### 临时值的使用场景

  - **主键**：对于自增的主键，EF 会在添加新实体时生成一个临时的主键值。这个值在调用 `SaveChanges()` 并且数据被实际插入到数据库后，会被数据库生成的真实值替换。

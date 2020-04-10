## 1.实体类相关的注解

**@Entity、@Id**

@Entity 注解表明该类是一个实体类，并且使用默认的 `ORM` 规则，即类名作为数据库表中表名，类的属性名作为表中的字段名。以下面代码为例：

```java
@Entity
public class Employee implements Serializable {
    private static final long serialVersionUID = 1L;
    
    @Id
    private Long id;
    private String name;
    private int age;
    private String addree;
   // Getters and Setters
}
```

如果没有 `@javax.persistence.Entity` 和 `@javax.persistence.Id `这两个注解的话，它只是一个典型的 POJO 的 Java 类，现在加上这两个注解之后，就可以作为一个实体类与数据库中的表相对应，即ORM框架会自动帮你完成映射。

要使实体类与数据库中的表相对应必须遵守以下规则：

- 实体类必须用 @Entity 进行注解；
- 必须使用 @Id 来注解一个主键；
- 实体类必须拥有一个 public 或者 protected 的无参构造函数，此外实体类还可以拥有其他的构造函数；
- 实体类必须是一个顶级类（top-level class）。一个枚举（enum）或者一个接口（interface）不能被注解为一个实体；
- 如果实体类的一个实例需要用传值的方式调用（例如，远程调用），则这个实体类必须实现java.io.Serializable 接口。

**@Table**

@Table 注解是一个非必须的注解，必须配合@Entity使用。@Table 注解指定了 Entity 所要映射带数据库表名，其中 @Table.name用来指定映射表的表名。声明此对象映射到数据库的数据表，通过它可以为实体指定表名(name)，目录 (catalog) 和 schema。

注：当@Table和@Entity同时指name时，@Table.name的优先级高于@Entity.name

**@GeneratedValue、@GenericGenerator**

@GeneratedValue通用策略生成器，必须配合@Id注解使用，用来指定主键的生成策略。

@GenericGenerator自定义主键生成策略，用于配合@GeneratedValue注解，来生成其他类型的主键。例如，使用UUID作为主键

```java
@Id
@GenericGenerator(name = "uuid2", strategy = "org.hibernate.id.UUIDGenerator")
@GeneratedValue(generator = "uuid2")
private UUID uuid;
```

**@Column**

@Column注解声明了属性与数据库之间的映射关系。例如，该字段是否唯一、是否能为空、长度、字段名等。需要注意的是当有属性不需要与数据库字段映射时，需要使用`@Transitent`修饰，表示此属性与表没有映射关系。

**@CreatedDate、@LastModifiedDate**

从字面意思就可以理解，这两个注解分别表示实体创建日期和实体最后更新的日期。在进行新建和更新操作时，字段会自动设置并插入数据库。

使用方式，需要在实体类上加上监听器注解，并且向相应字段上加上@CreatedDate和@LastModifiedDate注解，最后必须在application类上加上@EnableJpaAuditing注解，如下：

```java
@Entity
@Table(name = "store_source_bind")
@EntityListeners(AuditingEntityListener.class)
public class StoreSourceBind {
    /**
     * 创建时间
     */
    @Column(name = "create_time")
    @CreatedDate
    private Date createTime;
   
    /**
     * 修改时间
     */
    @Column(name = "lastmodified_time")
    @LastModifiedDate
    private Date lastmodifiedTime；
}
```

```java
@SpringBootApplication
@EnableJpaAuditing
public class WalletApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(WalletApplication.class).web(true).run(args);
    }
}
```

**@OneToOne**

JPA使用@OneToOne来表示一对一的映射关系，一对一关系有两种描述方式。

- 外键，一个实体通过将另一个实体的主键作为外键。
- 中间表，通过中间表来保存两个实体的主键。

下面以用户（People）与地址（Address）为例进行说明。

1. 外键

	people 表（**id**，name，**address_id**）

	address 表（**id**，address）

	通过`@OneToOne`和`@JoinColumn`注解在people表中将adress主键作为外键

	**People.java**

	```java
	@Entity（name = "people"）
	public class People {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name = "id", nullable = false)
	    private Long id;
	    
	    @Column(name = "name", nullable = true, length = 20)
	    private String name;
	  
	    @OneToOne(cascade=CascadeType.ALL)//People是关系的维护端，当删除 people，会级联删除 address
	    @JoinColumn(name = "address_id")//默认以address的主键作为外键,如果不想使用主键作为外键可以使用referencedColumnName属性来设置
	    private Address address;
	}
	```

	**Adress.java**

	```java
	@Entity(name = "address")
	public class Address {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name = "id", nullable = false)
	    private Long id;//id
	   
	    @Column(name = "address", nullable = true, length = 100)
	    private String address;//地址
	   	
	    //mappedBy表示Address是关系的被维护端，其值address为Address在People中的属性名，Address无法维护与People之间的关系
	    //optinoal属性表示该属性是否能为空，false表示不能为空
	    //如果不需要根据Address级联查询People，可以去掉这个字段
	  	@OneToOne(mappedBy = "address", cascade = {CascadeType.MERGE, CascadeType.REFRESH}, optional = false)
	    private People people;
	}
	```

2. 中间表

	people 表（**id**，name）

	address 表（**id**，address）

	people_address表 (**people_id**, **address_id**)

	通过`@OneToOne`和`@JoinTable`注解来创建中间表

	**People.java**

	```java
	@Entity（name = "people"）
	public class People {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name = "id", nullable = false)
	    private Long id;
	    
	    @Column(name = "name", nullable = true, length = 20)
	    private String name;
	  
	    @OneToOne(cascade=CascadeType.ALL)//People是关系的维护端，当删除 people，会级联删除 address
	    @JoinTable(name = "people_address",
	            joinColumns = @JoinColumn(name="people_id"),
	            inverseJoinColumns = @JoinColumn(name = "address_id"))
	    private Address address;
	}
	```

	**Adress.java**

	不变

**@OneToMany和@ManyToOne**

使用较多的一对多关系包括单向多对一和双向一对多。这两者的区别主要在于一端是否持有多端的引用，单向一对多只需要在多端使用@ManyToOne注解，在多端的表中将单端的主键作为外键；而双向一对多的表结构和单向一对多的表结构没有区别，差异在于一端是否使用@OneToMany注解并持有多端对象。

1. 单向多对一（只使用@ManyToOne注解）

	下面以订单（order，多端）和客户（customer，一端）为例，进行说明。一个客户可以有多个订单，只需要从订单端找到对应的客户，而不关心客户的所有订单。此时可以使用单向多对一，只需要从一端访问多端。

	*表结构*

	customer表（**id**, name）

	order表（**id**, orderNumber, **customer_id**）

	通过@ManyToOne和@JoinColumu注解设置order表的外键

	**Customer.java**

	```java
	@Entity(name = "customer")
	public class Customer {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name = "id", nullable = false)
	    private Long id;
	    
	    @Column(name = "name", nullable = true, length = 32)
	    private String name;
	}
	```

	**Order.java**

	```java
	@Entity(name = "customer")
	public class Order {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name = "id", nullable = false)
	    private Long id;
	    
	    @Column(name = "name", nullable = true, length = 32)
	    private String orderNumber;
	    
	    @ManyToOne(optional = false)//optional属性表示是否能为空，false表示不能为空
	    @JoinColumn(name = "customer_id") //设置在order表中的关联字段(外键),默认为关联实体的主键
	    private Customer customer;
	}
	```

2. 双向一对多（@ManyToOne和@JoinColumu注解同时使用）

	在JPA规范中，一对多的双向关系由多端来维护。多端为关系维护端，负责关系的增删改查。一端则为关系被维护端，不能维护关系。

	还是以订单（order，多端）和客户（customer，一端）为例，当客户需要知道自己的所有订单时即需要通过客户获得对应的订单，必须使用双向一对多关系。

	*表结构*

	customer表（**id**, name）

	order表（**id**, orderNumber, **customer_id**）

	**Customer.java**

	```java
	@Entity(name = "customer")
	public class Customer {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name = "id", nullable = false)
	    private Long id;
	    
	    @Column(name = "name", nullable = true, length = 32）
	    private String name;
	    
	    @OneToMany(mappedBy = "customer")//用于mappedBy属性的为关系被维护端       
	    private List<Order> orders;
	}
	```

	**Order.java**

	不变

**@ManyToMany**

JPA中使用@ManyToMany注解来映射多对多的关系，必须通过中间表来维护。中间表的表名默认是：主表名+下划线+从表名(主表是指关系维护端对应的表,从表指关系被维护端对应的表)。这个关联表只有两个外键字段，分别指向主表ID和从表ID。字段的名称默认为：主表名+下划线+主表中的主键列名，从表名+下划线+从表中的主键列名。

多对多关系与一对多关系类似，也存在单向多对多和双向多对多关系。

1. 单向多对多（只在一端使用@ManyToMany注解）

	下面以类别（category，关系维护端）和条目（item, 关系被维护端）为例进行说明。一个类别可以有多个条目，一个条目可以属于多个类别，单向多对多中只需要通过类别来找到所有条目，而不需要通过条目获得其所属的所有类别。

	*表结构*

	category 表（**id**, name）

	item表（**id**, name, price）

	category _items表（**category _id**, **item_id**）

	**Category .java**

	```java
	@Entity(name = "category")
	public class Category {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name = "id", nullable = false)
	    private Long id;
	    
	    @Column(name = "name", nullable = false)
	    private String name;
	
	    @ManyToMany
	    @JoinTable(name = "category_item", joinColumns = @JoinColumn(name = "category_id"),
	            inverseJoinColumns = @JoinColumn(name = "item_id"))
	    private List<Item> items;
	}
	```

	关于category有几点需要说明：

	- 关系维护端，负责多对多关系的绑定和解除，即只能由category对象来添加items对象，item无法维护他们的关系。
	- @JoinTable注解的name属性指定关联表的名字；joinColumns指定主表主键的名字，关联到关系维护端category；inverseJoinColumns指定从表主键的名字，关联到关系被维护端(item)
	- @JionTable注解可以省略，由默认规则生成中间表。

	**Item.java**

	```java
	@Entity(name = "item")
	public class Item {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name = "id", nullable = false)
	    private Long id;
	    
	    @Column(name = "name", nullable = false)
	    private String name;
	}
	```

2. 双向多对多（两端都使用@ManyToMany注解）

	双向多对多关系中，可以指定任意一端为关系维护端，指定关系维护端后（非mappedBy属性持有端），两端关系的绑定和解除都只能由维护端来完成。当绑定关系后，无法直接删除关系被维护端，要解除绑定关系后才能删除，但可以直接删除关系维护端，删除时会先解除绑定关系。

	还是以类别（category，关系维护端）和条目（item, 关系被维护端）为例进行说明。当双方都需要通过自身来找到对应的对象时，需要使用双向多对多。

	*表结构*

	category 表（**id**, name）

	item表（**id**, name, price）

	category _items表（**category _id**, **item_id**）

	**Category .java**

	与上文中相同

	**Item.java**

	与上文中实体类相似，但是在Item中增加了category的集合。

	```java
	@Entity(name = "item")
	public class Item {
	    @Id
	    @GeneratedValue(strategy = GenerationType.IDENTITY)
	    @Column(name = "id", nullable = false)
	    private Long id;
	    
	    @Column(name = "name", nullable = false)
	    private String name;
	    
	    @ManyToMany(mappedBy = "items")
	    private List<Category> categories;
	}
	```

<==To be continue...

## 2.在SpringBoot中使用Hibernate Validate校验前台传入的属性

详见[https://www.jianshu.com/p/5ab860e4f25a](https://www.jianshu.com/p/5ab860e4f25a)

## 3.使用SpringDataJPA实现数据库访问（默认规则、继承关系之类）

## 4.JpaSpecificationExecutor接口（动态查询）

## 5.自定义Repositiry并与默认实现整合，封装自己的BaseRepository

## 参考

1. [https://liuyanzhao.com/7913.html](https://liuyanzhao.com/7913.html)
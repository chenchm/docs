## 1.Ovirt中的Dao层测试相关

engine中`BaseDaoTestCase`类是dao层单元测试的基础父类，通过Spring TestContext框架为单元测试类提供了上下文环境（dao层对象、datasource等）并且使用Dbunit从Xml文件中加载初始数据到数据库，使数据库的值达到一个已知状态。

## 1.1**Spring TestContext框架**

Spring测试框架可以在测试类上使用@RunWith注解来指定Runner，并且配合Spring其他注解来使用，`BaseDaoTestCase`类使用了如下注解：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@TestExecutionListeners({ TransactionalTestExecutionListener.class, DependencyInjectionTestExecutionListener.class })
@ContextConfiguration(locations = { "classpath:/test-beans.xml" })
@Transactional
public abstract class BaseDaoTestCase {
    ...
}
```

各注解的作用说明：

- **@RunWith**注解指定了`SpringJUnit4ClassRunner`作为测试运行器，该运行器负责总装Spring TestContext测试框架并将其统一到 JUnit 4.4 框架中。
- **@TestExecutionListeners**注解用于配置测试监听器（TestExecutionListener），测试监听器的作用是在对应测试事件发生时起作用。
	- `DependencyInjectionTestExecutionListener.class`负责解析自动注入相关的注解（@Autowire、@Inject），为依赖注入和测试实例的初始化提供支持。
	- `TransactionalTestExecutionListener.class`负责解析事务相关的注解（@Transactional、@Rollback），确保所有的测试用例在事务内执行，并在事务完成时自动回滚测试。
- **@ContextConfiguration**注解用于加载指定的Spring配置文件。
- **@Transactional**使所有的测试用例在事务中执行。

## 1.2 初始化测试条件

`BaseDaoTestCase`作为所有dao层测试类的父类主要负责初始化工作，包括创建数据源、根据xml文件初始化测试数据库以及使用JNDI绑定数据源。

```java
@BeforeClass
public static void initTestCase() throws Exception {
    if(dataSource == null) {
        try {
            //通过读取properties文件来获得连接参数
            dataSource = createDataSource();
            
            //使用Dbunit来初始化数据库
            final IDataSet dataset = initDataSet();
            DatabaseOperation.CLEAN_INSERT.execute(getConnection(), dataset);
            
            //将dataSource绑定到JDNI
            SimpleNamingContextBuilder builder = new SimpleNamingContextBuilder();
            builder.bind("java:/ENGINEDataSource", dataSource);
            builder.activate();
            initialized = true;
        } catch (Exception e) {
            LoggerFactory.getLogger(BaseDaoTestCase.class).error("Unable to init tests", e);
            throw new AssertionError("Unable to init tests", e);
        }
    }
}

@Before
public void setUp() throws Exception {
}
```

其中需要注意以下几点：

- @BeforeClass注解在整个测试过程中只执行一次，并且被该注解标记的方法只能被**static void**修饰；
- 测试程序初始化和执行的顺序为：执行@BeforeClass注解标记的方法->加载@ContextConfiguration指定文件中的bean对象->执行@Test注解标记的测试用例，所以在@BeforeClass标记的方法中不能使用注入的对象，否则会报空指针异常；
- @Before注解标记的方法将会在每个测试用例执行前运行一次，该方法在`BaseDaoTestCase`为空实现，当子类有需要时可以重写该方法。

## 1.3DbFacade在测试中的初始化

`DbFacade`在engine作为dao层对象的容器，负责提供各种dao对象。现在存在的问题是engine源码中依赖注入使用的是CDI（[JBoss Weld](http://seamframework.org/Weld)），但是测试环境使用的是spring环境而spring并不支持CDI，只支持JSR330（CDI兼容JSR330）。

`DbFacade`在源码中注入dao对象的方式如下：

```java
@Inject
private Instance<Dao> daos;

DbFacade() {
    init();
}

private void init() {
    log.info("Initializing the DbFacade");
    instance = this;
}
```

在CDI中@Inject注解配合`Instance`接口可以动态地将所有实现`Dao`接口的对象一起注入。

由于`Spring`并不支持CDI，所以在测试环境中不能通过以上方式来注入bean，所以只能通过@Configuration注解类来配置daos，提供`DbFacade`所需要的依赖。

```java
@Configuration
class CdiIntegration implements BeanDefinitionRegistryPostProcessor {
    private ConfigurableListableBeanFactory beanFactory;

    @Bean
    public Instance<Dao> daos() {
        //通过spring的beanFactory来获得所有实现Dao接口的对象
        Map<String, Dao> daoMap = beanFactory.getBeansOfType(Dao.class);
        return new InstanceImpl(daoMap.values());
    }

    //实现BeanDefinitionRegistryPostProcessor接口后，spring容器在启动时会该类装配beanFactory
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
    }

    private static class InstanceImpl implements Instance<Dao> {
        private Iterable<Dao> daos;

        public InstanceImpl(Iterable<Dao> daos) {
            super();
            this.daos = daos;
        }

        @Override
        public Dao get() {
            return daos.iterator().next();
        }

        @Override
        public Iterator<Dao> iterator() {
            return daos.iterator();
        }

        ...
    }
}
```

上面的类通过@Bean注解在spring容器中创建了名为daos的bean对象，进而在spring的xml文件中配置`DbFacade`，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>
    ...
    <!-- depends-on属性指定的Bean要先初始化完毕后才初始化当前Bean -->
    <bean id="dbFacade" class="org.ovirt.engine.core.dal.dbbroker.DbFacade" depends-on="daos"/>
    ...
</beans>
```

## 参考

1. [https://blog.csdn.net/beijicy/article/details/47441231?utm_source=blogxgwz2](https://blog.csdn.net/beijicy/article/details/47441231?utm_source=blogxgwz2)

2. [https://blog.csdn.net/weixin_33753003/article/details/87765826](https://blog.csdn.net/weixin_33753003/article/details/87765826)
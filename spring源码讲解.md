# spring源码讲解

## 一、注解及组件注册

### 1、@Configuration

#### 1.1、意义：

​	告诉spring这是一个配置类

#### 1.2、属性值：

```
String value() default "";  //指定该配置类beanName,默认是类名首字母小写
```

#### 1.3、示例：

```
@Configuration()
public class MyConfig {

}
```



### 2、@Bean

#### 2.1、含义：

给容器注册一个Bean；类型为返回值的类型，id默认用方法名作为id,或者添加value指定id值  比如@Bean("beanName")

#### 2.1、属性值

```
@AliasFor("name")
String[] value() default {}; //别名，作用和name一样，可以有多个别名

@AliasFor("value")
String[] name() default {};//作用通value

Autowire autowire() default Autowire.NO;//装配类型(name或者type),默认是不自动装配

String initMethod() default "";//指定初始化方法

String destroyMethod() default AbstractBeanDefinition.INFER_METHOD;//指定摧毁方法
```

#### 2.3、示例代码

```java
@Configuration
public class MyConfig{
	@Bean
	public Person person(){
			return new Person();
	}		
}

public class Test{
	public static void main(String[] args){
		ApplicationContext ac = new AnnotationConfigApplicationContext(MyConfig.class);	
		Person p = ac.getBean("person");
	}
}
```



### 3、@ComponentScan

#### 3.1、含义：

​	指定扫描的包

xml中配置如下：

```xml
<!--包扫描，只要标注了@Controller、@Service、@Repository、@Conponent都会被扫描进spring的bean管理-->
<context:component-scan base-package="com"/>
```

#### 3.2、属性值：

```
@AliasFor("basePackages")
String[] value() default {}; //指定扫描包路径

@AliasFor("value")
String[] basePackages() default {};//同value

Filter[] excludeFilters() default {};//按照过滤规则排除相应的扫描

Filter[] includeFilters() default {};//按照指定规则进行扫描
```

#### 3.3、示例

配置类：

```java
@Configuration()
@ComponentScan(value="com",excludeFilters = {
        @Filter(type= FilterType.ANNOTATION,classes = {Controller.class})
}) //扫描com包，有@Controller注解的不进行扫描
public class MyConfig {

    @Bean(value={"user1","user2"})
    public User user(){
        return new User();
    }

}


```

测试类：

```
@Test
public void test(){
    ApplicationContext applicationContext = new 					AnnotationConfigApplicationContext(MyConfig.class);
    String[] beans = applicationContext.getBeanDefinitionNames();
    for(String bean:beans){
        System.out.println(bean);
    }

}
```



#### 3.4、@Filter扩展

@Filter是@Component的一个内部类

##### 3.4.1、意义：

包扫描的过滤规则

##### 3.4.2属性值

```
FilterType type() default FilterType.ANNOTATION;  //过滤的类型，默认采用注解过滤，详情见             	                                            FilterType枚举类

@AliasFor("classes")
Class<?>[] value() default {}; //过滤的class类型数组

@AliasFor("value")
Class<?>[] classes() default {};//同classes

String[] pattern() default {};//设定的模式，主要用FilterType为ASPECTJ和REGEX两种类型
```

##### 3.4.3、FilterType的类型

```
ANNOTATION   //根据注解过滤

ASSIGNABLE_TYPE  //根据给定的类型

ASPECTJ   //根据AspectJ类型表达式

REGEX   //根据正则表达式

CUSTOM   //自定义类型，需要实现org.springframework.core.type.filter.TypeFilter
```



##### 3.4.4、自定义类型

```
public class MyFilter implements TypeFilter {
	/**
     * 满足要求就返回true
     * @param metadataReader
     * @param metadataReaderFactory
     * @return
     * @throws IOException
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        //获取当前类的当前信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //获取正在扫描包的信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        //获取当前类资源
        Resource resource = metadataReader.getResource();

        return false;
    }
}
```



### 4、@ComponentScans

#### 4.1、含义

​	指定多个@ComponentScan属性值

#### 4.2、属性值

```
ComponentScan[] value();  //包含多个ComponentScan
```

#### 4.3、示例：

```
@ComponentScans(value={
        @ComponentScan(value="com",excludeFilters = {
                @Filter(type= FilterType.ANNOTATION,classes = {Controller.class})
        })
})
```



### 5、@Scope

#### 5.1、含义

指定作用域范围，默认是singleton单实例，单实例在容器创建时创建对象。多实例在获取对象时创建对象

#### 5.2、属性值

```
@AliasFor("scopeName")
String value() default "";//取值 prototype：多实例  singleton：单实例 request：每次请求创建一个  session：每次会话创建一个    后面两种情况针对web环境

@AliasFor("value")
String scopeName() default "";  //同vlaue	
```

#### 5.3、示例

```
public class MyConfig {

    @Bean(value={"user1"})
    @Scope("prototype")
    public User user(){
        System.out.println("创建实例...");
        return new User();
    }

}
```



### 6、@Lazy

#### 6.1、含义

懒加载。容器启动时不创建对象，第一次使用（获取）时创建对象，并初始化；

#### 6.2、属性值

```
boolean value() default true;//是否懒加载
```

#### 6.3、示例

```
@Bean(value={"user1"})
@Lazy
public User user(){
    System.out.println("创建实例...");
    return new User();
}
```



### 7、@Conditional

#### 7.1、含义

按照一定条件，满足条件给容器注册bean

#### 7.2、属性值

```
Class<? extends Condition>[] value(); //一个实现了Condition的类数组
```

#### 7.3、示例

```
@Bean(value={"user1"})
@Conditional({MyCondition.class})
public User user(){
    System.out.println("创建实例...");
    return new User("bill",50,10.00);
}

//实现类
public class MyCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //1、获取ioc使用的beanFactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        //2、获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3、获取当前环境信息
        Environment environment = context.getEnvironment();

        String property = environment.getProperty("os.name");
        if(property.contains("Windows")){
            return true;
        }

        return false;
    }
}
```



### 8、@Import

#### 8.1、含义

导入组件，

使用方式：1、直接输入类名数组，默认bean名为组件的全类名（含包名）

​					2、使用实现了ImportSelector的选择器

​					3、使用实现了ImportBeanDefinitionRegistrar的注册组件自己注册

#### 8.2、属性值

```
Class<?>[] value();  //类型数组
```

#### 8.3、示例

```
1、直接导入组件
@Import(User.class)
public class MyConfig {

    @Bean(value={"user1"})
    public User user(){
        System.out.println("创建实例...");
        return new User("bill",50,10.00);
    }

}

2、采用ImportSelector
@Import(MyImportSelector.class)
public class MyConfig {

    @Bean(value={"user1"})
    public User user(){
        System.out.println("创建实例...");
        return new User("bill",50,10.00);
    }

}

public class MyImportSelector implements ImportSelector {

    /**
     * 返回值：就是导入到容器中的组件全类名
     * AnnotationMetadata:当前标注@Import注解的类的所有注解信息
     * @param importingClassMetadata
     * @return  不要返回null值
     */
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.gaok.User"};
    }
}

3、使用ImportBeanDefinitionRegistrar
@Import(MyImportBeanDefinitionRegistrar.class)
public class MyConfig {

    @Bean(value={"user1"})
    public User user(){
        System.out.println("创建实例...");
        return new User("bill",50,10.00);
    }

}

public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //判断是否包含user名称的bean
        boolean  hasUser = registry.containsBeanDefinition("user");
        //如果没有就将其注入到容器中
        if(!hasUser){
            RootBeanDefinition beanDefinition = new RootBeanDefinition(User.class);
            registry.registerBeanDefinition("user",beanDefinition);
        }
    }
}

```



### 9、利用FactoryBean注册Bean

#### 9.1、示例

```
@Bean
public UserFactoryBean userFactoryBean(){
    return new UserFactoryBean();
}

/**
 * 默认获取到的是工厂bean调用的getObject创建的对象
 * 要获取工厂Bean本身，我们需要给id前面添加一个&
 */
public class UserFactoryBean implements FactoryBean<User> {

    /**
     * 返回一个注册到容器中的对象
     * @return
     * @throws Exception
     */
    @Override
    public User getObject() throws Exception {
        return new User();
    }

    /**
     * 返回对象的类型
     * @return
     */
    @Override
    public Class<?> getObjectType() {
        return User.class;
    }

    /**
     * 是否单实例
     * @return
     */
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```



## 二、bean生命周期
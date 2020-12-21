# java面试

## 1、说一下java类集

* 类集是java实现的数据结构应用，如果只是一个使用，那么类集的操作非常简单，因为类集的核心接口：List、Set、Map、Iterator、Enumeration；

* List子接口：是可以根据索引号取得内容，在List集合常见问题：ArrayList(包装的数组，数组是可变的)、LinkedList（链表的实现，搜索数据的时间复杂度:n）区别

* Set子接口:排序子类、HashSet与hashCode()和equals的关系

  * HashSet：重复的判断依靠的是hashCode()和equals方法，无序的

  * TreeSet：有序，依靠的是Comparable排序；

  * LinkedHashSet：继承了HashSet的特点，但是属于有序的（增加顺序为保存顺序）

* Map接口：Map.Entry、Iterator输出、HashMap、WeakHashMap（弱引用）

## 2、字符串hash相等，equals相等吗？反过来呢

equals相等，hash一定相等，hash相等，equals不一定相等。



## 3、spring的工作原理，控制反转是怎么实现的、自己写过滤器过滤编码怎么实现

* spring的核心组成：IOC&DI(工厂设计)、AOP(代理设计、动态代理设计)；
  *  -spring之中针对于XML的解析处理采用的是DOM4J的实现
  * Annotation的使用必须要求有一个容器；

* 对于编码过滤需要考虑两种情况：
  * Struts 1.x、Spring MVC、JSP+Servlet：都可以直接通过过滤器完成
  * Struts 2.X：必须通过拦截器完成
  * 实现：考虑到可扩展性的配置，所以在配置文件里面设置编码，在程序运行的时候动态取得设置的编码进行的操作，但是需要设置两个操作：请求编码、回应编码

## 4、框架的源码有没有看过

* 不要回答没有，
* 框架的核心思想：反射+XML(annotation)
  * Struts 2.x的设计：请求交由过滤器执行，而后过滤器交给控制器完成，后面由于将所有的跳转路径等信息都写在了配置文件或者是Annotation里面，所以还需要进行这部分内容的加载；
  * SpringMVC：它是基于方法的请求处理，所有的参数都是提交到方法上，本质上还是一个DispacherServlet；
  * Hibernete：就是反射和DOM 4J解析处理流程

## 5、动态代理是怎么实现的
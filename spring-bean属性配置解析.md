#spring-bean属性配置解析
[TOC]
###autowire属性值
byName 根据Bean定义时的“id"属性上指定的别名与Setter名称是否一致进行自动装配
byType 根据PoJo的setXXX()方法所接受的类型判断bean定义文件是否定义有类似的类型对象进行自动装配
constructor Spring容器比对容器中的Bean实例类型及相关的构造方法上的参数类型是否符合进行自动装配
autodetect 先进行constructor自动装配，若缺省，则进行byType自动装配
no不进行自动装配
 
###depends-on
若A depends-on B 意思是实例化A之前必须先实例化B，但A不需要持有B的实例
abstract属性值
false默认
ture表示抽象Bean，ApplicationContext预初始化时忽略所有抽象Bean定义
 
###parent   
表示该Bean为子Bean，其值指向父Bean，重用父Bean已实现的依赖
 
###dependency-check属性值
simple 只检查简单的属性是否完成依赖关系
objects 检查对象类型的属性是否完成依赖关系
all检查全部的属性是否完成依赖关系
none默认值，表示不检查依赖性

###singleton属性值指定此Java Bean是否采用单例（Singleton）模式 
false则通过BeanFactory获取此Java Bean实例时，BeanFactory每次都将创建一个新的实例返回。
true(默认) 则在BeanFactory作用范围内，只维护此Java Bean的一个实例，代码通过BeanFactory获得此Java Bean实例的引用。
 
###init-method 初始化方法，
此方法将在BeanFactory创建JavaBean实例和属性set注入之后，在向应用层返回引用之前执行。一般用于一些资源的初始化工作。
 
###destroy-method 销毁方法
此方法将在BeanFactory销毁的时候执行，一般用于资源释放。
 
###lazy-init属性值
true 延迟加载，也就是IoC 容器将第一次被用到时才开始实例化
false默认
 
###factory-bean
通过实例工厂方法创建bean，class属性必须为空，factory-bean属性必须指定一个bean的名字，这个bean一定要在当前的bean工厂或者父bean工厂中，并包含工厂方法。而工厂方法本身通过factory-method属性设置。
factory-method 定义工厂方法，若是class属性指向工厂类，该工厂类包含的工厂方法须是static
###scope属性值
scope可以取值： 
 * singleton:每次调用getBean的时候返回相同的实例.这个是默认,也就是单实例
 * prototype:每次调用getBean的时候返回不同的实例.这个是多实例

还可取值request、session、global session等（不常用）

具体的用法如下
```
<bean id="userDao" class="com.test.dao.impl.UserDAOImpl" scope="singleton"/>
 <bean id="userService" class="com.test.service.impl.UserServiceImpl" scope="prototype">
  <property name="userDao" ref="userDao"/>
 </bean>
```
这个怎么需要看你具体的时机...如果你的bean是无状态的.那么单实例就可以了...如果Bean是有状态的,那么你最好设置成多实例的.因为这样可以解决线程安全问题.

id: Bean的唯一标识名。它必须是合法的XML ID，在整个XML文档中唯一。 
name: 用来为id创建一个或多个别名。它可以是任意的字母符合。多个别名之间用逗号或空格分开。 
class: 用来定义类的全限定名（包名＋类名）。只有子类Bean不用定义该属性

 
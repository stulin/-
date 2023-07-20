//内嵌的tomcat，神策上的代码是否 配置个WebConfig registrationBean能运行web？

//Bean工厂相关的后处理器，4个；Bean生命周期相关的后处理器，讲了2个？

课程地址：

https://www.bilibili.com/video/BV1P44y1N7QG/?vd_source=8bd5ab544d4cb8d9821752b68ce53b11

#### 问题小结：

- jdk代理的原理？如何自己实现代理？Cglib?? 代理的模拟实现自己手写下？？
- 想体验下微软的面试？看下人家和我之间的差距？
- 我一直在想，为什么cglib jdk代理要直接生成字节码？因为直接生成字节码才能避免反射调用吗？或者直接生成java类定义也是可以的；  ==所以代理的精华在于自动生成class定义/对应class的字节码？估计是先生成class再编译比较麻烦，因为要检测到哪里有自动生成定义的代码，所以干脆直接生成字节码==

看源码技巧

![image-20230130195909028](spring原理-photos/image-20230130195909028.png)

![image-20230130200213405](spring原理-photos/image-20230130200213405.png)

![image-20230130200315981](spring原理-photos/image-20230130200315981.png)

![image-20230130200432114](spring原理-photos/image-20230130200432114.png)

### BeanFactory和ApplicationContext

#### 1.什么是BeanFactory//看类图ctrl alt u; 查看实现：ctrl alt 单击；

![image-20230205170555379](spring原理-photos/image-20230205170555379.png)

启动类的run方法返回值就是spring容器，

ApplicationContext继承了BeanFactory; BeanFactory[里面有SingletonObjects]才是Spring的核心，ApplicationContext的部分功能是组合了BeanFactory的内容实现的[beanFactory是applicationContext的成员变量，可以断点看属性验证]；





![image-20230202201258253](spring原理-photos/image-20230202201258253.png)

#### 2.BeanFactory的功能(要看有哪些方法和实现哪些接口、还有成员变量可以有很多方法)//ctrl+F12看方法？ uml :选中+F4

默认实现类  DefaultListableBeanFactory，可以管理所有Bean; 实现类中的DefaultSignletomBeanRegistry管理单例对象，可以反射查看所有单例；//下面的singletonObjects.get()里面的入参是什么情况？?也是个对象 说明beanFactory也是个单例对象，且保存在singletonObject中；看看 反射查看私有属性的示例代码？？

![image-20230205165940508](spring原理-photos/image-20230205165940508.png)

//只想看某个bean 话还可以过滤；

![image-20230205170207500](spring原理-photos/image-20230205170207500.png)

#### 3.ApplicationContext相对BeanFactory多了四种能力：//log输出日志会给出类名？@Autowired注入环境变量；

![image-20230205164002835](spring原理-photos/image-20230205164002835.png)

处理国际化资源的能力：MessageSource: context.getMessage() ； 依据key找到不同版本翻译结果，一般在messages打头的文件(不同语言的资源信息)；浏览器的请求头提供请求的语言类型；

![image-20230202204706856](spring原理-photos/image-20230202204706856.png)



​	通配符  获取资源（磁盘路径对应的资源）的能力：getResources()   //在jar包里面也查找：calsspath*:....

![image-20230205172101764](spring原理-photos/image-20230205172101764.png)



getEvironment: 获取环境信息（系统环境变量等）

发布事件对象 (本质上是一种解耦方式，如用户注册后 发短信、发邮件等；可以对标下AOP看哪个更优雅？)：pushlishEvent ，入参 事件源要继承ApplicationEcent； 接收时间  参数和入参一直，@EventListener

![image-20230205225338111](spring原理-photos/image-20230205225338111.png)

//单例+发送事件；

![image-20230205230310058](spring原理-photos/image-20230205230310058.png)

![image-20230205230425861](spring原理-photos/image-20230205230425861.png)

### 容器实现

#### BeanFactory实现 //默认实现DefaultListableBeanFactroy 是一个核心的spring容器？  bean的定义，BeanFactory会依据定义创建对象

//容器默认为空，往容器添加也给bean定义(先设置BeanDefinitin--类名、生命周期；然后注册bean--设置bean名字)；

![image-20230205232232567](spring原理-photos/image-20230205232232567.png)

原始的beanFactory并不会去解析注解，添加了后处理器[register扩展功能]并执行[postProcess/addBean....，或者称为建立联系]之后，会去解析注解，

![image-20230205233200331](spring原理-photos/image-20230205233200331.png)

BeanFactory后处理的主要功能，补充了一些bean定义，如@Bean @Configuration注解；



Bean后处理器，针对ean生命周期的各个阶段提供扩展，解析例如@Autowired注解；@Resource注解//javaee的注解

![image-20230206230400978](spring原理-photos/image-20230206230400978.png)



bean创建对象的时机：初始化的时候只会保存bean的定义、描述信息到beanFactory，当第一次用的时候，才会真正创建实例； 单例对象如果希望初始化时创建所有的单例对象，可以使用preInstantiateSingletons()：

![image-20230206231511602](spring原理-photos/image-20230206231511602.png)

applicationContext会把上面这些常用的初始化操作都直接封装好；



beanFactory的排序：

@autowired，bean容器中找实现类；有多个的时候[可以用qualifier指定？]会匹配成员变量名字和类名，匹配上优先；@Resource可以用name属性指定； 



![image-20230206232613301](spring原理-photos/image-20230206232613301.png)

![image-20230206232848214](spring原理-photos/image-20230206232848214.png)

优先级[优先级高的生效]@Autowired > @Resource，可以用比较器控制先后顺序，排序依据实现order接口的getOrder方法的返回值，数字小的排前面//同时使用两个注解时;

![image-20230206234006952](spring原理-photos/image-20230206234006952.png)

为什么sorted之后顺序会变？？和register一样的比较器啊？除非register只是进行了比较器的初始化，并没有把它用于排序，即执行；

#### ApplicationContext的常见实现和用法

- ClassPathXmlApplicationContext  基于xml路径读取配置；通常使用    <context:annotation-config>   标签就会自动加入一些有用的后处理器；
  -  ![image-20230208215603262](spring原理-photos/image-20230208215603262.png)
- FileSystemXmlApplicationContext : 基于文件路径读取配置；//绝对路径、相对路径均可
  - ![image-20230208215718274](spring原理-photos/image-20230208215718274.png)
- ApplicationContext是如何把beanDefination信息加载到beanFactory中的：用的XmlBeanDefinitonReader的 loadBeanDefinitions方法，入参也可以是ClassPathResource()对象；
  - ![image-20230208220500569](spring原理-photos/image-20230208220500569.png)
- AnnotationConfigApplicationContext : 基于配置类的applicationContext
  - ![image-20230212110308969](spring原理-photos/image-20230212110308969.png)

#### 内嵌容器、注册DispatcherServletAnnotationConfigServletWebServerApplicationContext：既支持配置类，又支持内嵌servlet的Web容器---tomcat

- - spring的web服务器的核心是DispatcherServlet； DispatcherServlet要运行在tomcat服务器中；
  - 路径一般配置/，所有请求都经过dispatcherServlet，再到controller
  - ![image-20230212113647701](spring原理-photos/image-20230212113647701.png)
  - ![image-20230212113533870](spring原理-photos/image-20230212113533870.png)
  - 前3步必须的[构建内置tomcat，构建dispatcherServlet，建立dipatcherServlet和tomcat容器之间的关联]，controller1可选，bean名字/开头并实现Controller就可以作为控制器；
  - ![image-20230212113738417](spring原理-photos/image-20230212113738417.png)

#### 

### Spring Bean的生命周期

#### Spring Bean生命周期的各个阶段

![image-20230212223508800](spring原理-photos/image-20230212223508800.png)

 //@Autowired的参数  也会自动注入[值、变量]；

![image-20230212213754091](spring原理-photos/image-20230212213754091.png)



![image-20230212225551281](spring原理-photos/image-20230212225551281.png)

![image-20230212225605205](spring原理-photos/image-20230212225605205.png)

![image-20230212225841324](spring原理-photos/image-20230212225841324.png)



#### 模板设计模式：固定不变的内容+接口调用称为了模板[变化的内容单独封装为接口]；模板方法不需修改，改业务代码即可；

![image-20230212232225441](spring原理-photos/image-20230212232225441.png)

![image-20230212232233475](spring原理-photos/image-20230212232233475.png)

![image-20230212232156415](spring原理-photos/image-20230212232156415.png)

### 第四章

#### 常见的Bean后处理器//变量注入放在方法里面可以打印； Resolver是为了解析@Value值注入；//掌握下每个后处理器 能解析哪几个注解

![image-20230212234621913](spring原理-photos/image-20230212234621913.png)

![image-20230212234601021](spring原理-photos/image-20230212234601021.png)





Spring注解积累，@ConfigrationProperties  SpringBoot的bean的属性和配置文件的键值对 做绑定；







一个单例的bean注入其他scope的bean会有问题，需要加一个Lazy注解。

#### 单例注入多例，scope会失效（解决方案本质上都是获取多例的时候多一层） E（单例）有属性发（多例）；

![image-20230305135529245](spring原理mac.assets/image-20230305135529245.png)

- 添加@Lazy解决  

  - 代理类是原F1的子类，代理对象使用f的方法时 可以控制注入新的f对象；

![image-20230305135609810](spring原理mac.assets/image-20230305135609810.png)

- 要注入的多例对象上添加配置，也是生成代理

![image-20230305140022017](spring原理mac.assets/image-20230305140022017.png)

- 多一层对象工厂

![image-20230305140156490](spring原理mac.assets/image-20230305140156490.png)

- 注入一个ApplicationContext，调用getBean方法；

#### aop之ajc增强

切面的实现方式不止代理，MyAspect监听MyService，拿到的MyService对象名字还是MyService; 编译的时候改写了原有的类  增加了前置增强；

编译结果的位置，==拖拉文件到idea可以反编译！！==

![image-20230305141334462](spring原理mac.assets/image-20230305141334462.png)

本质上，切面是由ajc编译器管理的，所以MyAspect不需要添加@Component注解；

pom.xml中添加插件：

![image-20230305141714908](spring原理mac.assets/image-20230305141714908.png)

![image-20230305141736924](spring原理mac.assets/image-20230305141736924.png)

有时候，idea默认是javac编译器，不使用aspect插件，可以使用maven的compiler强制使用；



相比spring的代理方式实现 aop，使用ajc编译器（修改class文件）可以增强static方法；

#### Aop实现之agent类加载

类加载阶段实现修改字节码 实现增强

![image-20230305145656423](spring原理mac.assets/image-20230305145656423.png)

![image-20230305145721931](spring原理mac.assets/image-20230305145721931.png)

可以突破代理实现aop的限制：一个方法调用另一个方法，被调用的方法无法增强；

![image-20230305145922238](spring原理mac.assets/image-20230305145922238.png)

阿里巴巴的arthas工具：可以实现运行时的反编译（类加载才实现的增强，直接看target中编译结果还是没有增强）

![image-20230305150119707](spring原理mac.assets/image-20230305150119707.png)

![image-20230305150228904](spring原理mac.assets/image-20230305150228904.png)

#### AOP实现之proxy//==简写为lamda快捷键？？==

代理类没有源码，运行时直接生成字节码；

##### jdk只能针对接口代理

- 参数一：classLoader；
- 参数二：要实现的接口可以一次实现多个接口；
- 参数三：invocationHandler，规定被代理方法具体的行为；

- invocationHandler的三个参数：
  - 代理对象
  - 真正执行的方法
  - 方法的参数

- 示例：
  - ![image-20230305152125580](spring原理mac.assets/image-20230305152125580.png)
- 特点：被代理类和代理类是兄弟关系，不能互相强转，且被代理类可以是final;

##### cglib实现代理

- 参数一：被代理类
- 参数二：Callback的子接口，MethodInterceptor
- MethodInterceptor的四个参数：代理对象，当前代理对象中执行的方法，方法的参数，MethodProxy
- 示例
  - ![image-20230305153521286](spring原理mac.assets/image-20230305153521286.png)
  - 使用methodProxy可以避免反射调用，invoke传被代理对象[spring使用这种]，厚着invokeSuper传代理对象
  - ![image-20230305153957518](spring原理mac.assets/image-20230305153957518.png)

- 特点：被代理类和代理类是父子关系，可以相互强转，且被代理类不能是final；父类方法加了final也不能被增强，代理类会重写被代理方法；

#### jdk代理原理

//代理的场景：日志；权限；事务的增强？

- 模拟实现代理，代理对象 增加一个私有成员Invocationhandler，代理方法的具体执行逻辑，放在Invocationhandler的invoke方法中； 在声明代理对象的时候再指定具体的invoke方法的执行内容；

- ![image-20230305155311339](spring原理mac.assets/image-20230305155311339.png)
- 对象有多个方法时，上述代码的invoke方法都调用的是被代理的foo方法，需要改进 所有的方法都调invoke， invoke具体反射调用哪个被代理方法 参数化；
- ![image-20230305155917375](spring原理mac.assets/image-20230305155917375.png)
- 还需要增加返回值处理（invoke是Object），增加代理对象参数，异常处理（检查异常、throwable异常不能直接抛，需要转换下再抛）；
- ![image-20230305161229366](spring原理mac.assets/image-20230305161229366.png)
- 方法对象的获取不需要每次调用都获取一次
- ![image-20230305161510415](spring原理mac.assets/image-20230305161510415.png)
-  jdk的代理会继承Proxy类，内含一个InvocationHandler接口，所以可以直接用
- ![image-20230305163339477](spring原理mac.assets/image-20230305163339477.png)
- arthas工具[powershell]需要知道类名才能反编译；程序要保持运行状态，可以System.in.read()；//直接看jdk代理源码基本看不懂，因为用的asm动态生成代理类的字节码；

- ![image-20230305164327370](spring原理mac.assets/image-20230305164327370.png)
- ![image-20230305164452865](spring原理mac.assets/image-20230305164452865.png)
- ![image-20230305164535950](spring原理mac.assets/image-20230305164535950.png)
- ![image-20230305164609432](spring原理mac.assets/image-20230305164609432.png)

#### jdk代理字节码生成

- jdk代理没有源码，运行期间动态生成字节码，用的asm[spring jdk使用很多]
- 安装idea插件 java源码转换为asm代码，然后可以转换为字节码；但不能很好地在高版本的jdk里面工作；
  - ![image-20230305165206211](spring原理mac.assets/image-20230305165206211.png)
- 编写一个代理类的代码
  - ![image-20230305165934972](spring原理mac.assets/image-20230305165934972.png)
  - 编译， 右键-- show Bytecode outline，会转换为ASMified  即asm代码；拷贝代码、粘贴；需要导下包，导入spring的包即可；
  - ClassWriter类调用生成字节码；cw.visit 就是生成一个代理对象（@Lazy就是做这个事情）；定义类的成员变量、方法;  cw.toByteArray（）得到的数组就是Class字节码； 例如：把byte数组写进ckass文件；
  - ![image-20230308223140701](spring原理mac-photos/image-20230308223140701.png)
  - 直接在内存中使用字节码，defineClass依据字节数组生成类对象；  入参：类名、字节数组、字节数组起始为位置、长度；依据类对象可以创建对象；   
  - ![image-20230308223951386](spring原理mac-photos/image-20230308223951386.png)
  - ![image-20230308224058618](spring原理mac-photos/image-20230308224058618.png)
  - 具体字节码的生成要看asm的api，还要熟悉jvm的指令，成本有点高；

#### jdk反射优化//反射调用一般效率较低

- method.invoke本质上是通过methodAccessor实现类实现；
- 前16次用本地的java native api MethodAccessor，性能低；第17次  换了实现类反射，提高了性能；
- ![image-20230308225422323](spring原理mac-photos/image-20230308225422323.png)
- ![image-20230308225547930](spring原理mac-photos/image-20230308225547930.png)

- Arthas查看得知，第17次直接直接正常调用，为了优化反射调用  直接生成了 代理类，就可以直接调用
  - ![image-20230308225402202](spring原理mac-photos/image-20230308225402202.png)

​			

#### cglib代理

![image-20230319142730188](spring原理mac.assets/image-20230319142730188.png)

- 继承父类

- ==MehodInterceptor.intercept 入参: 代理类对象   当前正在执行的方法   方法实参数组 **mehtodProxy[后面有说明]**==
- jdk和cglib的区别：jdk第17次有优化，针对一个方法产生一个代理类， 反射调用-->直接调用；cglib第一次调用就会产生代理，无需反射调用，一个代理类对应两个FastClass[对应代理对象 和目标对象 ，可能有些场景目标对象比代理对象多了一些功能吧，所以有两种用法？]，每个FastClass类里面可以匹配多个方法，相比jdk生成代理类数量少一些；

![image-20230319143141053](spring原理mac.assets/image-20230319143141053.png)

![image-20230319143558529](spring原理mac.assets/image-20230319143558529.png)

![image-20230319143626105](spring原理mac.assets/image-20230319143626105.png)

![image-20230319143813939](spring原理mac.assets/image-20230319143813939.png)

- ==MehtodProxy.create的五个参数：目标类型  代理类型 “参数、返回类型”  带增强功能的方法名 带原始功能的方法名;==//()V表示入参为空，返回值为void; ()表示入参为int 返回值 void
- ![image-20230319150416493](spring原理mac.assets/image-20230319150416493.png)

![image-20230319145038369](spring原理mac.assets/image-20230319145038369.png)

![image-20230319150845297](spring原理mac.assets/image-20230319150845297.png)

- methdoProxy避免反射调用的原理，使用了FastClass; FastClass也是直接生成字节码，没有Java的源码；==核心的两个方法 getIndex[方法签名转换为整数编号] invoke[依据整数编号调用对用的方法]==； Proxy方法中初始化代码，调用create方法的时候会生成targetClass代理对象；
- methodProxy.invoke(target, args)模拟实现；结合目标对象使用的class
  - ![image-20230319151421709](spring原理mac.assets/image-20230319151421709.png)
  - ![image-20230319155548583](spring原理mac.assets/image-20230319155548583.png)
  - ![image-20230319155732395](spring原理mac.assets/image-20230319155732395.png)
  - ![image-20230319160219530](spring原理mac.assets/image-20230319160219530.png)
  - ![image-20230319161820487](spring原理mac.assets/image-20230319161820487.png)
- methodProxy.invoke()    结合代理对象使用
  - 注意：调的是代理类的原始功能方法  因为使用methodProxy代理之前已经进行过增强了；
  - ![image-20230319162337876](spring原理mac.assets/image-20230319162337876.png)
  - ![image-20230319162908632](spring原理mac.assets/image-20230319162908632.png)
  - ![image-20230319162954823](spring原理mac.assets/image-20230319162954823.png)

#### spring选择代理

**切面：通知+切点**； advisor包含一个通知和切点；

- ![image-20230319164309395](spring原理mac.assets/image-20230319164309395.png)

模拟实现切面

- org.springframework.aop.Pointcut

- ![image-20230319164910490](spring原理mac.assets/image-20230319164910490.png)

- org.aopalliance.intercept.MethodInterceptor

- ![image-20230319165552181](spring原理mac.assets/image-20230319165552181.png)

- ![image-20230319165937659](spring原理mac.assets/image-20230319165937659.png)

- ProxyFactory会依据具体情况选择 cglib或者jdk增强

  - ![image-20230322220034855](spring原理mac-photos/image-20230322220034855.png)
  - ![image-20230322220228401](spring原理mac-photos/image-20230322220228401.png)

  

#### 16讲 切点匹配

- execution(返回值  包名-类名-方法名 )

- @annotation(注解的 包名-类名)

- ![image-20230322221158226](spring原理mac-photos/image-20230322221158226.png)
- @Transactional  可加方法、类[所有方法]、接口[实现该接口的类的所有的接口中的方法]；

- MergeAnnotaions可以获取方法  类上的注解信息；默认只差一层，即仅本类，不会搜父类 接口，设置为SearchStrategy.TYPE_HIERARCHY
  - ![image-20230322223204543](spring原理mac-photos/image-20230322223204543.png)

- ![image-20230322223228071](spring原理mac-photos/image-20230322223228071.png)

#### 17讲 从@Apect到Advisor

- 
- ![image-20230322224530882](spring原理mac-photos/image-20230322224530882.png)
- 转换为lamda快捷键？？？
- ![image-20230322224829058](spring原理mac-photos/image-20230322224829058.png)
- GenericApplicationContext是一个干净的容器；注册配置类  ConfigurationClassPostProcessor后处理器解析配置类内部的@Bean注解

- ![image-20230322225053015](spring原理mac-photos/image-20230322225053015.png)



#### 第17讲 findEligibleAdvisors

bean处理器，一  找到所有的切面，包括高级切面Aspect和低级切面Advisor；二  依据切面创建代理对象；

AnnotationAwareAspectJAutoProxyCreator.class

findEligibleAdvisors ：找到所有有资格的低级切面 List， 高级切面会被转换成低级切面；入参：目标类[查看切点是否和目标类匹配]，bean在容器的名字

wrapIfNecessary:是否有必要创建代理

- ![image-20230329214318042](spring原理mac-photos/image-20230329214318042.png)
- ![image-20230329214751813](spring原理mac-photos/image-20230329214751813.png)
-   ![image-20230329215404758](spring原理mac-photos/image-20230329215404758.png)

#### 第17讲 代理创建时机

//通常是 创建之后  或者 初始化之后 二选一；//==我看后续的视频似乎是还有 依赖注入和初始化之间创建代理????应该理解为之前，如bean2依赖bean1，注入bean1之前会创建代理对象==

- ![image-20230329222446471](spring原理mac-photos/image-20230329222446471.png)
- ![image-20230329221344788](spring原理mac-photos/image-20230329221344788.png)
- ![image-20230329221457507](spring原理mac-photos/image-20230329221457507.png)
- ![image-20230329221649561](spring原理mac-photos/image-20230329221649561.png)
- ![image-20230329221945241](spring原理mac-photos/image-20230329221945241.png)
- 循环依赖的情况，bean1的代理对象在bean1的构造--bean1的初始化之间被创建，因为依赖注入需要  代理对象被提前创建；
- ![image-20230329222321324](spring原理mac-photos/image-20230329222321324.png)

#### 第17讲 切面的生效顺序

- 可以自己设置顺序，高级切面和低级切面的顺序设置方法如下
  - ![image-20230331184739833](spring原理mac-photos/image-20230331184739833.png)
- spring切面顺序控制的缺点
  - 低级切面， @Order 和@Bean共同使用时无效
  - 高级切面，加在方法上无效，故单个类内的不同方法的顺序无法控制



#### 第17讲 高级切面转换为低级切面

- 遍历方法，查看方法的注解，创建切点  不同注解对应的通知类  切面；

- ![image-20230331190751095](spring原理mac-photos/image-20230331190751095.png)
- ![image-20230331190922746](spring原理mac-photos/image-20230331190922746.png)

### 第18讲 静态通知调用

#### 不同通知同意转换为环绕通知，适配器模式体现

- ![image-20230401145634031](spring原理mac-photos/image-20230401145634031.png)
- 最后的效果是想达到下述的 套娃的效果，由外到内进，由内到外出；
- ![image-20230401145543139](spring原理mac-photos/image-20230401145543139.png)
- getInterceptorsAndDynamicInterceptionAdvice会把不同通知转换成环绕通知
- ![image-20230401150606788](spring原理mac-photos/image-20230401150606788.png)
-  适配器模式：把一套接口转换成另一套接口，以便适合某种场景的调用
- ![image-20230401152121979](spring原理mac-photos/image-20230401152121979.png)
- 创建并执行调用链；proceed（）调用所有的环绕通知和目标；还需要准备好methodInvocation，可以最外层环绕通知实现放入当前线程（addAvice(ExposeInvocationInterceptor.INSTANCE)）；
- ![image-20230401161906834](spring原理mac-photos/image-20230401161906834.png)
- ![image-20230401162201172](spring原理mac-photos/image-20230401162201172.png)
- ![image-20230401162211740](spring原理mac-photos/image-20230401162211740.png)
- ![image-20230401162226966](spring原理mac-photos/image-20230401162226966.png)



#### 无参数绑定通知链执行过程，责任链模式体现 

#### 模拟实现MethodInvocation

- 本质就是一个递归调用；
- 责任链模式：链对应调用链，元素对应 通知类；
- ![image-20230401170127067](spring原理mac-photos/image-20230401170127067.png)
- ![image-20230401170144096](spring原理mac-photos/image-20230401170144096.png)
- ![image-20230401170152734](spring原理mac-photos/image-20230401170152734.png)

### 第十九讲 动态通知调用

#### 参数绑定通知链执行过程

![image-20230405095105582](spring原理mac-photos/image-20230405095105582.png)

- proxyCreator()用于将高级切面转为低级切面，同时创建代理对象；
  -  ![image-20230405101001361](spring原理mac-photos/image-20230405101001361.png)
- 测试类代码，interceptorList则是所有通知转换后的环绕通知；最外层的ExposeInvocationInterceptor为其它通知准备好methodInvocatin对象；带参数的切面，最后得到的是InterceptorAndDynamicMethodMatcher对象（内部含切点属性、通知属性）
  - ![image-20230405101150137](spring原理mac-photos/image-20230405101150137.png)
  - ![image-20230405101905348](spring原理mac-photos/image-20230405101905348.png)
- ![image-20230405102240033](spring原理mac-photos/image-20230405102240033.png)
- ![image-20230405102348242](spring原理mac-photos/image-20230405102348242.png)

- 注意，上面的invocation使用了一个语法：new 一个匿名子类，为的是调用受保护的构造；

### 第二十讲：RequestMappingHandlerMapping与RequestMappingHandlerAdapter

#### diapatcherServlet初始化

- AnnotationConfig指支持java配置类方式构建容器；ServletWebServer支持内嵌 web容器（如内嵌tomcat）；
- @ComponentScan默认范围：自己类所在的包及子包；
- 工厂方法的参数支持按类型匹配，接近依赖注入；
- 不和其它servlet路径匹配，默认和/匹配
  - ![image-20230405105013453](spring原理mac-photos/image-20230405105013453.png)

- dispatcherServlet是spring容器创建；但是初始化 ：tomcat容器默认在首次使用dispatcherServlet的时候初始化（走的是servlet的初始化流程）； 如果希望在启动tomcat的时候初始化dipatcherServlet，可以设置loadOnStartup，大于0便会在启动是初始化，具体数值表示多个servlet时的优先级；
  - ![image-20230405110150278](spring原理mac-photos/image-20230405110150278.png)
  - 配置属性可以在配置文件中设置 @PropertySource；ServerPropertires会打包读取配置类中server打头的key
  - ![image-20230405110509882](spring原理mac-photos/image-20230405110509882.png)
  - ![image-20230405110902514](spring原理mac-photos/image-20230405110902514.png)
  - ![image-20230405111145774](spring原理mac-photos/image-20230405111145774.png)
  - ![image-20230405111229135](spring原理mac-photos/image-20230405111229135.png)

#### diapatcherServlet初始化内容

- onRefresh-->initStrategies，会初始化下面的九个组件；
  - ![image-20230405112543985](spring原理mac-photos/image-20230405112543985.png)
  - initMultipartResolver：文件上传成解析器
  - initLocaleResolver：本地化解析器，属于哪一种国家、地区、语言；//有多种实现：accept cookie
  - initThemeResolver：不重要
  - initHandlerMappings  路径映射器，请求下方到controller；
  - initHandlerAdapters   handler：具体处理请求的代码，有多种形式；适配 不同形式的适配器方法，并调用它；
  - initHandlerExceptionResolvers   解析异常
  - 后面三个不重要

- initHandlerMappings代码阅读
  - ![image-20230405113052468](spring原理mac-photos/image-20230405113052468.png)
  - 找到所有的HandlerMapping（detectAllHandlerMapping如果为真如果当前容器没有还会去父容器中找），如果容器中有，优先使用容器中的HandlerMapping；如果容器没有，使用默认的HandlerMapping，在DispatcherServlet.properties中配置；

#### RequestMappingHandlerMapping //流程：请求到handlerMapping映射到控制器，并和拦截器包装成调用链 chain；然后handlerAdapter解析参数；然后执行方法；最后 返回值 处理器对返回值进行解析；

- 解析RequestMapping及派生注解，建立 请求路径----控制器之间的映射关系；
- `初始化的时候`，先到当前容器下找到所有控制器，查看控制器有哪些方法并记录 路径---控制器方法  信息；

- 默认的RequestMappingHandlerMapping 创建的RequestMappingHandlerMapping对象会作为dispatcher的属性，但是不会放入Spring容器中； 可以在WebConfig中，添加定义：
  - ![image-20230405135550465](spring原理mac-photos/image-20230405135550465.png)

- 模拟  路径匹配handlerMethod的过程；HandlerExcecutionChain不仅包含了handlerMethods[即控制器的方法信息]，还包含了拦截器对象；
  - ![image-20230405141610054](spring原理mac-photos/image-20230405141610054.png)

#### RequestMappingHandlerAdapter

- 调用控制器方法
- ![image-20230405142611874](spring原理mac-photos/image-20230405142611874.png)
-  invokeHandlerMethod是protected方法，为了调用可以自己创建一个子类，放大修饰符；测试案例：

- ![image-20230405143357866](spring原理mac-photos/image-20230405143357866.png)

- 如何解析控制器方法的参数、返回值等？？
- ![image-20230405150833366](spring原理mac-photos/image-20230405150833366.png)

- 自定义参数解析器
  - ![image-20230410140805276](spring原理mac-photos/image-20230410140805276.png)
  - ![image-20230410140819593](spring原理mac-photos/image-20230410140819593.png)
  - 加在参数位置上，运行期一直都有效；目标：标注了@Token，就会获取请求投的参数，赋值给token参数；
  - ![image-20230410142447131](spring原理mac-photos/image-20230410142447131.png)
  - 校验参数是否包含@Token注解，不包含（return false）则不继续解析；
  - ![image-20230410142552736](spring原理mac-photos/image-20230410142552736.png)
  - 将自定义的参数解析器，加入到Adapter类中；
  - ![image-20230410142807805](spring原理mac-photos/image-20230410142807805.png)
  - 测试；

- 自定义返回值处理器 //依据返回值类型、方法是否加某个注解进行特殊处理

  - ![image-20230410143225981](spring原理mac-photos/image-20230410143225981.png)
  - ![image-20230410144056540](spring原理mac-photos/image-20230410144056540.png)
  - 第三步是为了省略去spring MVC后续的视图解析等流程；
  - ![image-20230410144140098](spring原理mac-photos/image-20230410144140098.png)
  - ![image-20230410144452656](spring原理mac-photos/image-20230410144452656.png)
  - 测试


### 第二十一讲  参数解析器

#### 常见的参数解析器

- ![image-20230418201704758](spring原理mac-photos/image-20230418201704758.png)

- 没加解析参数默认@RequestParam或者@ModelAttribute
- ![image-20230412211924689](spring原理mac-photos/image-20230412211924689.png)
- 控制器方法封装为HadnlerMethod，然后才能完成 访问路径映射；对象绑定与类型转换，入请求的String转换互为contrller的int入参；
- getMethodParameters可以获取所有的形参，但是参数名还是Null，initParameterNameDiscovery才能解析参数名，
- getParameterAnnotations获取参数上的所有注解名，但是真正解析注解，获取实参需要RequestParamMethodArgumentResolver.resolveArgument；
- ![image-20230412214805728](spring原理mac-photos/image-20230412214805728.png)
- ![image-20230418195126689](spring原理mac-photos/image-20230418195126689.png)

#### 逐个解析器调试@RequestParam

- 模拟请求类定义
- getMethodParameters可以获取所有的形参，但是参数名还是Null，initParameterNameDiscovery才能解析参数名，
- getParameterAnnotations获取参数上的所有注解名，但是真正解析注解，获取实参需要RequestParamMethodArgumentResolver.resolveArgument；
- new RequestParamMethodArguementResolver: beanFactory[用于支持${}解析等]  是否能省略@RequestParam注解 
- resolver.resolveArgument入参：参数；modelAndView容器 暂存中间model结果，spring封装后的请求request，bindFactory[用于类型转换]
- 解析${}需要beanFactory 读取环境变量、控制文件；

- ![image-20230412220715273](spring原理mac-photos/image-20230412220715273.png)
- ![image-20230412220804261](spring原理mac-photos/image-20230412220804261.png)
- ![image-20230418194938944](spring原理mac-photos/image-20230418194938944.png)
- ![image-20230418194755181](spring原理mac-photos/image-20230418194755181.png)
- ![image-20230412222214873](spring原理mac-photos/image-20230412222214873.png)
- 当前问题：没有@RequestParam的其它参数 如带@PathVariable也会被尝试解析；

#### 组合模式

- 需要依次调用每个Resolver.supportsParameter方法，直到找到一个 支持此参数的解析器；//==组合器的设计模式==
- ![image-20230418201104224](spring原理mac-photos/image-20230418201104224.png)
- 解析@PathVariable注解之前，需要handlerMapping将{id}和实参对应起来
- ![image-20230418203043026](spring原理mac-photos/image-20230418203043026.png)
- ![image-20230418203126803](spring原理mac-photos/image-20230418203126803.png)
- @RequestHeader @CookieValue @Value HttpServletRequest ，依次对应如下：
- ![image-20230418204112667](spring原理mac-photos/image-20230418204112667.png)
- ${}是环境参数，#{}是spring的EL表达式；

10-12

- @ModelAttribute， 
  - 参数和javaBean的属性做一个绑定，对应的ServletModelAttributeMethodProcessor可以指定@ModelAttribute是否可以省略，spring中会添加两个；参数解析器的结果作为模型数据存入ModelAndViewContainer[默认modelAttribute中名字为类型名字  ]；  
  - 注意：不需要@ModelAttribute注解的的processor一定要放在最后，不然会尝试省略@ModelAttribute方式解析@RequestBody对应的参数，认为它省略了@ModelAttribute； 多个省略要先对象，后普通类型；
- ![image-20230606105347176](spring原理mac-photos/image-20230606105347176.png)
- @ModelAttribute，@RequestBody依次对应
- ![image-20230425200654484](spring原理mac-photos/image-20230425200654484.png)
- 消息转换器，把JSON数据解析为javaBean；
- dataBinder[类型转换和数据绑定]换一个：![image-20230425195229757](spring原理mac-photos/image-20230425195229757.png)
- ![image-20230425200601463](spring原理mac-photos/image-20230425200601463.png)

//${}对应配置文件；#{}对应EL表达式；

### 第二十二讲  参数解析器

#### 获取参数名(之前用了DefaultParameterNameDiscoverer)

- 编译    反编译 可以发现编译默认是不保留参数名；可以添加-parameters[编译会生成MethodParameter，可以发射获取] ；或者-g[编译会生成LocalVariableTable，反射无法获取，但是可以ASM获取; 只能获取类的参数名，对接口不生效]  //javap -c -v .\Bean1.class
- project structure-----Modules-----show-----dependencies，点击+号，JARs or directories，把Bean2.java所在的外层目录加进来；
- 反射获取
  - ![image-20230607230342537](spring原理mac-photos/image-20230607230342537.png)
- ASM获取，借助工具，底层用的asm；

- spring中结合了两种，勇大 是DefaultParameterNameDiscoverer
  - ![image-20230607230037504](spring原理mac-photos/image-20230607230037504.png)

### 第二十三讲  对象绑定与类型转换

- 底层第一套转换接口与实现
  - ![image-20230607230921675](spring原理mac-photos/image-20230607230921675.png)
- 底层第二套转换接口  //jdk自带
  - ![image-20230607231119585](spring原理mac-photos/image-20230607231119585.png)
- 高层接口实现
  - ![image-20230608001924728](spring原理mac-photos/image-20230608001924728.png)
  - 转换器查找顺序
    - ![image-20230608001940383](spring原理mac-photos/image-20230608001940383.png)
  - 几个接口的附加功能
    - ![image-20230608002014695](spring原理mac-photos/image-20230608002014695.png)
    - spring反射插件bean，不知道有哪些属性，需要批量属性赋值；Property走反射的set get；Field 走反射的成员变量赋值；
    - 绑定配置文件属性和Bean属性，directFieldAccess为真则走Field；
  - 四个实现的基本用法
    - ![image-20230608124859458](spring原理mac-photos/image-20230608124859458.png)
    - bean属性赋值，类型不匹配会自动转换；
    - ![image-20230608124915996](spring原理mac-photos/image-20230608124915996.png)
    - ![image-20230608124944364](spring原理mac-photos/image-20230608124944364.png)
    - ![image-20230608125018307](spring原理mac-photos/image-20230608125018307.png)
    - web环境下推荐的binder和propertyValues
    - ![image-20230608125546669](spring原理mac-photos/image-20230608125546669.png)
- 绑定工厂
  - 不合格式的日期等需要自定义转换器；
  - new ServletRequestDataBinderFactory入参：要新增的自定义的方法、???
  - createBinder入参：封装的request请求、目标对象、对象名[随便起]
  - @initBinder标注方法  入参：WebDataBinder；  factory.createBinder方法调用时，会回调@InitBinder方法，可以在回调中把自定义的formatter方法添加倒WebDataBinder中；
    - 无自定义转换
      - ![image-20230611220516701](spring原理mac-photos/image-20230611220516701.png)
    - @InitBinder注解实现自定义转换，底层用的是PropertyEditorRegistry   PropertyEditor
      - ![image-20230611220729862](spring原理mac-photos/image-20230611220729862.png)
    - ![image-20230611112332517](spring原理mac-photos/image-20230611112332517.png)
    - 用ConversionService+formatter 添加自定义转换方法，需要封装为Intializer；
      - ![image-20230611222627050](spring原理mac-photos/image-20230611222627050.png)
    - 两种都有，@InitBinder优先级更高
      - ![image-20230611223736631](spring原理mac-photos/image-20230611223736631.png)
    - 默认的ConversionService配合@DateTimeFormat指定日期格式；  DefaultFormattingConversionService或ApplicationConversionService；
      - ![image-20230611224506730](spring原理mac-photos/image-20230611224506730.png)

- 绑定工厂
  - getGenericSuperclass获取有泛型信息的父类；有泛型信息 类型是ParameterizedType；
  - resolveTypeArgument入参：子类类型、父类类型
  - 获取父类泛型参数的两种方法
    - ![image-20230611225345879](spring原理mac-photos/image-20230611225345879.png)



### 第24讲 ControllerAdvice

- @InitBinder【添加自定义类型不定期】  @ExceptionHandler  @ModelAttribute
- @InitBinder  用于扩展类型转换器
  - @Controller【只对单个controller生效】、@ControllerAdvice中【全局，对所有控制器生效】
  -  getDataBinderFactory被调用的时候会解析contorller1的 @InitBinder
  - ![image-20230615210510198](spring原理mac-photos/image-20230615210510198.png)
  - ![image-20230615210534354](spring原理mac-photos/image-20230615210534354.png)

### 第25讲 控制器方法执行流程

- ![image-20230618100545004](spring原理mac-photos/image-20230618100545004.png)
- 图2 和 图3是连在一起的；先是图2这边的 准备工作：准备 数据绑定工厂、模型工厂，中间的临时数据保存倒ModelAndViewContainer；然后是图3：ServletInvocableHandlerMethod 完成调用：参数解析、反射调用方法、返回值解析、最后从ModelAndViewContainer中获取最终结果
  - ![image-20230618101521892](spring原理mac-photos/image-20230618101521892.png)
  - ![image-20230618102238378](spring原理mac-photos/image-20230618102238378.png)
  - 参数名解析器、参数解析器、设置数据绑定工厂、加了@RespaonseStataus(HttpStatus.OK)，可以暂时不考虑返回值处理器；     handlerMethod.incokeAndHandle入参：http请求对象、MVCContainer
  - ![image-20230618105410210](spring原理mac-photos/image-20230618105410210.png)
  - ![image-20230618105422817](spring原理mac-photos/image-20230618105422817.png)

### 第26讲 ControllerAdvice之@ModelAttribute

- 加在参数名上  流程：参数解析器[ServletModelAttributeMethodProcessor]调用对象构造方法，用数据绑定工厂 绑定空对象和参数，结果放入MVCContainer；
- 加在方法名上  流程：解析者变为RequestMappingHandlerAdapter，ModelFactory[模型工厂]调用标注了@ModelAttribute方法，并把返回值放入MVCContainer；
  - afterProperties会找到controller中所有带@ModelAtribute注解的方法，并记录；  
  - ==我没想到的是用的handlerMethod对象，绑定的是foo方法，却不影响modelFactory的初始化和反射调用，看来getModelFactory.invoke的反射 真的是只执行了 带@ModelAtribute注解的方法 自动调用，所以只用到了类信息把；modelFactory的initModel( )方法可以为MVCContainer补充模型数据；==
  - ![image-20230618113052400](spring原理mac-photos/image-20230618113052400.png)
  - ![image-20230618113629547](spring原理mac-photos/image-20230618113629547.png)

- 加在controller中方法上 流程：单个控制器中方法调用时都会补充mvc数据；  而ControllerAdvice中方法上则对应所有的的controller中方法；

### 第27讲 返回值处理器

- 准备返回值处理器；渲染：这里结合freeMarker：renderView()方法是自己写好的渲染方法；
  - ![image-20230618123336599](spring原理mac-photos/image-20230618123336599.png)
  - ![image-20230618123500334](spring原理mac-photos/image-20230618123500334.png)
- 7种返回值类型：ModelAndView、String[代表视图的名字]、@ModelAttribute+@RequestMapping[默认试图会取路径]+自定义类型、自定义类型[省略@ModelAttribute]     不走视图渲染的三个方法:【RequestHandler为true】： HttpEntity<T>   HttpHeaders   @ResponseBody
- ModelAndView  //modelAndView也没有定义试图名称啊？？？？ModelAndView种指定试图名了
  - ![image-20230618125215217](spring原理mac-photos/image-20230618125215217.png)
- String和ModelAndView类似  方法名改为method2即可；
- ModelAttribute+@RequestMapping[默认试图会取路径]+自定义类型
  - @RequestMapping[默认试图会取路径]原理：路径解析结果存到request作用域[resolveAndCacheLookupPath]，后续会生成默认的视图名；
  - 没加@RequestMapping注解的话需要自己设置路径到request作用域
  - ![image-20230619234244698](spring原理mac-photos/image-20230619234244698.png)
- HttpEntity<T>  可控制 状态码  响应头  响应体；响应体有值
  - ![image-20230620000522201](spring原理mac-photos/image-20230620000522201.png)
- HttpHeaders：和HttpEntity<T>类似，不同 的是响应头有值
  - ![image-20230620000928682](spring原理mac-photos/image-20230620000928682.png)
- @ResponseBody  和HttpEntity<T>类似，有值的也是响应体，会自动生成部分响应头[有默认值]
  - ![image-20230620000928682](file://C:/Users/lrh/Desktop/code202211/js/summaryAndStudyPlan/spring/spring%E5%8E%9F%E7%90%86mac-photos/image-20230620000928682.png?lastModify=1687191151)

- 被解析返回结果的方法：
  - ![image-20230619234802077](spring原理mac-photos/image-20230619234802077.png)
  - ![image-20230619234821921](spring原理mac-photos/image-20230619234821921.png)
  - ![image-20230620001250060](spring原理mac-photos/image-20230620001250060.png)

小tips:

- //ctrl+alt+b查看实现类； ctrl + alt +v 提取出变量； ctrl+alt+m 抽取成方法； ctrl + alt +上/下箭头  stack trace

- MockHttpServletRequest可以模拟http请求

- MVCContainer中默认的 对象名字：类型名首字母小写；

  

### 第28讲 MessageConverter

- 信息转换器：入参处理器解析requsetBody转换为JSON串、返回值处理器等；

- 消息  javaBean转换示例：

  - ![image-20230622200950643](spring原理mac.assets/image-20230622200950643.png)

  - ![image-20230622201212626](spring原理mac.assets/image-20230622201212626.png)

  - ![image-20230622201224770](spring原理mac.assets/image-20230622201224770.png)

- 多个转换器的执行顺序，若指定请求的response的ContentType/request的Accept头，则以ContentType/Accept头为准；默认List顺序；
  - ![image-20230622202136887](spring原理mac.assets/image-20230622202136887.png)
  - ![image-20230622202210618](spring原理mac.assets/image-20230622202210618.png)

### 第29讲 ControllerAdice之ResponseBodyAdvice

- 对请求体、响应体的增强，例：Result类直接返回，不是则可以自动包装为Result；
- BeforeBodyWrite入参：响应结果、返回值的相关信息如方法名  注解等、contentType、converter等；
- AnnotationUtils.findAnnotation注解会递归查找某个注解，即包含一个该注解的子注解也算
- ![image-20230623152102107](spring原理mac.assets/image-20230623152102107.png)
- ![image-20230623152219896](spring原理mac.assets/image-20230623152219896.png)
- ![image-20230623152143191](spring原理mac.assets/image-20230623152143191.png)

### 第30讲 异常处理

- 之前不是好奇，@Exception注解后，未处理的异常是如何返回给前端的？这一节其实就是讲解这个过程
- dispatcherServlet的doDispatch方法：handlerAdaptor、handle()，如果有异常 会先记录，后续调用processDispatchResult；
- ![image-20230623153458512](spring原理mac.assets/image-20230623153458512.png)
- ExceptionHandlerExceptionResolver:处理带有@Exception；resolver.afterPropertiesSet()会自动设置一些默认的参数解析器、返回值处理器；
- 示例：
  - ![image-20230623154842405](spring原理mac.assets/image-20230623154842405.png)
  - ![image-20230623160015680](spring原理mac.assets/image-20230623160015680.png)
  - ![image-20230623160441568](spring原理mac.assets/image-20230623160441568.png)
  - 嵌套异常信息也能取出；
  - ![image-20230623162806749](spring原理mac.assets/image-20230623162806749.png)
  - ![image-20230623162745418](spring原理mac.assets/image-20230623162745418.png)
  - 例：获取入参
    - ![image-20230623163447886](spring原理mac.assets/image-20230623163447886.png)
    - ![image-20230623163517679](spring原理mac.assets/image-20230623163517679.png)

### 第31讲：ControllerAdvice之@ExceptionHandler

-  全局异常处理，@ControllerAdvice注解类+@ExceptionHandler注解方法，异常会由方法处理
- 会先找抛异常方法上是否有@Exception注解，如果没有的话，会找@ControllerAdvice注解类+@ExceptionHandler注解方法，异常会由方法处理
- 底层实现原理：
  - 初始化的afterPropies方法中会调用initExceptionHandlerAdiceCache()，该方法会 查找context中所有的ControllerAdviceBean，并便利找到其中包含exceptionhandler注解的方法，加入cahce，方便后续从cahce中取异常处理方法并调用；

- ![image-20230623165828547](spring原理mac.assets/image-20230623165828547.png)
- @Bean注解的方法都会自动回调initializeBean
- ![image-20230623165845605](spring原理mac.assets/image-20230623165845605.png)

### 第32讲 tomcat的异常处理

- 控制器的异常可以被ControllerAdvice处理，但是如filter中的异常不会被处理，需要更上层的异常处理者；其实tomcat是自带默认的异常处理器的，会自动返回异常的起因等等；
- tomcat自定义异常处理地址
  - 定义：errorPageRegistrart添加tomcat出错了默认的错误页面地址，可以是静态页面或者自定义的controller的地址[底层是请求转发，浏览器现实的地址不变]；errorPageRegistrarBeanPostProcessor [在创建TomcatServletWebServerFactory的时候会自动回调]用于 回调errorPageRegistrar
  - 其实 方法的入参/最后返回的ErrorPageRegistrar 就是TomcatServletWebServerFactory[是ErrorPageRegistrar的子类]
  - ![image-20230625234214224](spring原理mac-photos/image-20230625234214224.png)
  - tomcat捕获到spring框架外的异常会保存到Request域中；
  - ![image-20230625234424452](spring原理mac-photos/image-20230625234424452.png)
  - ![image-20230623174311022](spring原理mac.assets/image-20230623174311022.png)

- BasicErrorController
  - 支持不同的响应格式[json格式  html格式]
    - ![image-20230625235542579](spring原理mac-photos/image-20230625235542579.png)
  - 返回格式为json[如postMan请求] 入参： ErrotAttributes[要显示的异常内容]   ErrorProperties[要读取的配置文件的键值信息]
    - ![image-20230625235808206](spring原理mac-photos/image-20230625235808206.png)
  - 返回格式为html[如浏览器请求 postman设置Accept为text/html]  需要自定义名为error的视图，这里用bean+视图解析器的方式提供
    - ![image-20230627124607491](spring原理mac-photos/image-20230627124607491.png)
    - ![image-20230627124713159](spring原理mac-photos/image-20230627124713159.png)



### 第33讲  BeanNameUrlHandlerMapping与SimpleControllerHandlerAdapter

- 这么多映射器和适配器，各自有优缺点和适用场景吗？还是做好历史兼容？

- 之前用的RequestMappingHandlerMapping
  - ![image-20230704212839256](spring原理mac-photos/image-20230704212839256.png)
- 路径映射[需求解析@RequestMapping及其派生注解]    调用控制器方法[解析参数  调用 处理返回值]
- BeanNameUrlHandlerMapping   不是去找@RequstMapping注解的方法，而是去找  名字是/ 开头的bean
- SimpleControllerHandlerAdapter   [要求控制器的类必须实现Controller接口]
- ![image-20230704215532370](spring原理mac-photos/image-20230704215532370.png)
- ![image-20230704214047266](spring原理mac-photos/image-20230704214047266.png)
- ![image-20230704214008638](spring原理mac-photos/image-20230704214008638.png)
- 自定义MyHandlerMapping
  -  自己获取容器的bean最简单的方法就是注入ApplicationContext；  需要先找到 容器中所有实现了controller接口的bean，结果保存到collect中；
  - ![image-20230704221620351](spring原理mac-photos/image-20230704221620351.png)
  - ![image-20230704221643227](spring原理mac-photos/image-20230704221643227.png)

- 自定义MyHandlerAdapter
  - getLastModified已经过时；handle返回null表示不视图渲染流程；
  - ![image-20230704221525389](spring原理mac-photos/image-20230704221525389.png)

- RouterFunctionMapping与HandlerFunctionAdapter
  - RouterFunctionMapping初始化时会找到容器中所有的RouterFunction，并添加到？？请求来了，会和所有的RouterFunction的条件进行匹配，匹配上就找到对应的处理函数；最后由adapter反射 调用函数
  - 收集所有的RouterFunction，包括RequestPreficate[请求路径、请求方式等]  HandlerFunction [处理程序]，最后又HandlerFunctionAdapter调用handler
  - 例：get请求、请求路径为/r1 ，由后面的处理器(函数式接口)响应；
  - 和@RequestMapping对比，核心是 映射路径的方式不同，依据 RequestPRedicate方式，参数解析等功能相对少，但是简洁
    - ![image-20230706220402510](spring原理mac-photos/image-20230706220402510.png)

- SimpleUrlHandlerMapping与HttpRequestHandlerAdapter
  - SimpleUrlHandlerMapping映射；ResourceHttpRequestHandler作为处理器处理静态资源；HttpRequestHandlerAdapter调用处理器；
  - SimpleUrlHandlerMapping 没有自动收集返回结果为ResourceHttpRequestHandler的类，需要自己初始化，把所有的类汇总；
  - tomcat三件套初始化略；
  - ![image-20230709104505234](spring原理mac-photos/image-20230709104505234.png)
  - ![image-20230709104437991](spring原理mac-photos/image-20230709104437991.png)
  - ResourceHttpRequestHandler优化//afterPropertiesSet 默认只有一个路径资源的解析器；这里设置为  缓存资源、压缩资源、路径资源；
    - ![image-20230709105430751](spring原理mac-photos/image-20230709105430751.png)
    - 要使用EncodedResourceResolver压缩功能还需要初始化进行html文件压缩
      - ![image-20230709110510576](spring原理mac-photos/image-20230709110510576.png)
  - 欢迎页[静态]   //将访问/路径的请求映射到欢迎页 //springBoot才有
    - 入参：null, context, 欢迎页静态资源[用于判断是否存在]，指定处理器的 路径处理范围？？这里的/**对应前一讲静态资源的路径
    - ![image-20230709111737048](spring原理mac-photos/image-20230709111737048.png)
    - ![image-20230709111703047](spring原理mac-photos/image-20230709111703047.png)
  - 小结：
    - ![image-20230709143751184](spring原理mac-photos/image-20230709143751184.png)

### 36MVC处理流程

- 像一个总结，把前面各个小结的 单个组件 的内容，在这里全部都串联起来了，这里是大纲，前面是细枝末节;   结合每个细节，去前面的章节查看对应的内容//初始化时机：第一次请求来；配置load_on_startUp； 

- ![image-20230709153045566](spring原理mac-photos/image-20230709153045566.png)
- ![image-20230709154354568](spring原理mac-photos/image-20230709154354568.png)
- ![image-20230709154526233](spring原理mac-photos/image-20230709154526233.png)



### 37构建Boot项目骨架

- curl  https://start.spring.io/pom.xml  -d dependencies=mysql, mybatis,web -o pom.xml
- idea64 .\pom.xml
- //help:  curl https://start.spring.io

### 38Boot War项目

- jsp是能打包为war，这里视图用jsp； 

- idea--> new project-->spring initializr-->打包方式改为war, next-->勾选spring Web , finish
- src/main下新建webapp[文件夹名字固定]，创建jsp文件； 
- com.itheima的包下新建controller文件夹，新建controller文件； //字符串返回值会被解析成视图名

- ![image-20230709160006344](spring原理mac-photos/image-20230709160006344.png)
- 设置视图名字的前缀、后缀
  - ![image-20230709160119880](spring原理mac-photos/image-20230709160119880.png)

- //handlerMethod包含了控制器方法对象和控制器对象； preHandler判断请求是否被响应；
- 外置tomcat
  - 运行   配置-->  + -->tomcat server --> local  -->选择tomcat路径--> fix --> ==test5 : war exploded？？？== ,  context建议/ -->apply -->直接运行；
  - 需要创建ServletInitializer类，作为外置tomcat接入springBoot的入口
  - ![image-20230709162929784](spring原理mac-photos/image-20230709162929784.png)
- 内嵌tomcat
  - 没有自带jsp解析器，需要加入jsp解析器
  - ![image-20230709165138427](spring原理mac-photos/image-20230709165138427.png)
  - ![image-20230709165006759](spring原理mac-photos/image-20230709165006759.png)

### 39 Boot启动流程-构造方法

- SpringApplicaion.run --> new SpringApplication(primarySources).run(args);
- 主要内容分为两块，构造方法 做了什么？【准备该做】run方法 做了什么？【真正创建spring容器】
- 构造方法（准备工作，run方法创建spring容器）
  - ![image-20230716142710782](spring原理mac-photos/image-20230716142710782.png)
  - ![image-20230716142514501](spring原理mac-photos/image-20230716142514501.png)
  - 进一步显示bean的来源
    - ![image-20230716144308341](spring原理mac-photos/image-20230716144308341.png)
- 示例：设置BeanDefinition源
  - BeanDefinition源：配置类、xml文件等等；这里主要是指引导类； 
  - ![image-20230711124048783](spring原理mac-photos/image-20230711124048783.png)
- 示例：推断应用类型   //ClassUtils.isPresent 判断类路径下是否存在某个类
  - springBoot支持三种应用类型：非web程序、基于servlet的web程序、reactive的web程序； 基于jar包中关键类判断属于哪一种，创建不同类型的ApplicationContext； 
  - ![image-20230711124448244](spring原理mac-photos/image-20230711124448244.png)
  - ![image-20230711124758864](spring原理mac-photos/image-20230711124758864.png)
- ApplicationContext初始化
  - 初始化器对ApplicationContext添加扩展；
  - initialize的入参就是  容器的初始化器
  - ![image-20230711194220069](spring原理mac-photos/image-20230711194220069.png)
- 监听器与事件
  - 入参event就是生成的事件
  - ![image-20230711195830198](spring原理mac-photos/image-20230711195830198.png)
- 推断主类即运行main方法的类；
  - ![image-20230711200137785](spring原理mac-photos/image-20230711200137785.png)

#### 39 Boot启动流程-run

- ![image-20230716144717300](spring原理mac-photos/image-20230716144717300.png)
- 事件发布
  - SpringApplicationRunListener的实现类只有一个，接口、实现类的对应关系保存在配置文件中（spring-boot-***.META-INF.spring .factories），SpringFactoriesLoader提供相关访问方法； loadFactoryNames入参：接口类型、classLoader
  - 在spring一些重要节点结束之后就发布事件；
  - 反射创建发布器（调的是有参构造），并模拟发送各个事件//关注发布哪些事件即可，不必过于在意每个方法入参
  - ![image-20230711204420913](spring原理mac-photos/image-20230711204420913.png)
  - ![image-20230711204518120](spring原理mac-photos/image-20230711204518120.png)

- 后续步骤
  - 这里bean定义读取，以获取  类定义配置  bean、xml、classPathBeanDefinitionScanner为例；
  
  - 第10不用到的bean的类名 xml位置  扫描路径等本质上是.setResource方法设置的；
  
  - 1？  8 9 10 11设置增强；回调增强；加载bean定义；准备好bean定义才好调用 refresh()方法：准备后处理器，初始化所有单例；
    - ![image-20230716155741282](spring原理mac-photos/image-20230716155741282.png)
    - ![image-20230716155830749](spring原理mac-photos/image-20230716155830749.png)
    
  - 2  12run接口风味两类（入参不同  可以用于预加载数据等）：CommandLineRunner    main传的， ApplicationRunner  封装后的；
    - ![image-20230716155125046](spring原理mac-photos/image-20230716155125046.png)
    - ![image-20230716155615132](spring原理mac-photos/image-20230716155615132.png)
    - 添加参数
      -  ![image-20230716155323268](spring原理mac-photos/image-20230716155323268.png)
    - ![image-20230716155516289](spring原理mac-photos/image-20230716155516289.png)
    
  - 3 4 5 6环境对象有关[配置信息的抽象]    //系统环境变量   properties yaml
  
  - step3：设置env变量；设置命令行变量[暂时没有approperties的来源]；
  
  - ApplicationEnvironment默认两个来源  propertySources：系统属性[VM option]、系统环境[操作系统的环境变量]；有先后查找顺序；
  
  - 添加系统属性
    
    - ![image-20230716223044795](spring原理mac-photos/image-20230716223044795.png)
    
  - approperties、命令行[prgram arguments]  等人工的属性，可以手工添加；通过添加propertySource的方式；
  
    - ![image-20230720192319042](spring原理mac-photos/image-20230720192319042.png)
  
  - step4：为了使得getProperty能自动识别不同的分隔符    -、 _、 驼峰等，需要添加一个特殊的ConfigurationPropertySource；
  
    - ![image-20230720192153496](spring原理mac-photos/image-20230720192153496.png)
  
  - step5：对env进一步增强，补充propertySource[通过后处理器的方式，property对应的源就是在这一步添加]；   spring中是通过监听器读取配置，进行增强[事件发布、监听、增强]；
  
    - ![image-20230720194759521](spring原理mac-photos/image-20230720194759521.png)
  
    - ![image-20230720200059050](spring原理mac-photos/image-20230720200059050.png)
  
  - 补充：绑定env中键值到对象；
  
    - ![image-20230720202642913](spring原理mac-photos/image-20230720202642913.png)
  
  - step6：配置文件中键值绑定到springApplication
  
    - ![image-20230720203049098](spring原理mac-photos/image-20230720203049098.png)
  
  - step7:打印banner，需要借助SpringApplicationBannerPrinter[会把banner转换为文本信息]，可以自己指定banner，不指定会使用默认的banner；版本信息从spring boot jar包获取，manifest.mf中有版本信息;

P138



//tips:.if;  ctrl+alt+v；   a instanceof AClass  aClass.if;   List.of;  ==CTRL+F  chrome不走缓存访问服务器==；F12-->禁用缓存； 接口-->右键-->find usages;  ctrl+alt+左键直接到实现类；  idea右边右键文件，copy path,  reference；

//格式优化：lamda  静态导入；

//todo:mediaType列表；  编码方式列表；

//spring返回值就两种，一种是HttpEntity这种，那MVC就为空；第二种就是MVC响应，那么返回的结果内容存在MVC中；





//蜂蜜、麦片；蒸汽熨斗

//去异味喷雾  蒸汽熨斗；  局部污渍清洁液、小刷子、不掉毛的小帕子[必要时可开水]；[lint free]

棉质--洗衣机：分  白[温水 漂白粉]  浅 深[常温]三色；

丝质：人造丝[洗衣机  衣物清洗带 温和模式 不能阳光直射]  真丝[手洗  常温 专用洗衣液，局部刷  泡15min-30  挤干]

羊毛：羊绒[手洗  常温 专用洗衣液，局部刷  泡15min-30  挤干  平铺晾晒 毛球修剪器]   其它[洗衣机  衣物清洗带 温和模式 低转速 不能阳光直射 平铺晾晒] 



收纳：

挂：外套[宽一家]

叠：弹力、贴身的；

裤子：可叠可挂；



其它：

牛仔裤：尽量不洗；蒸汽熨斗；   洗衣机 洗衣袋；

去静电：特殊喷雾；


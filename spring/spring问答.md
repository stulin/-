如何自己从头实现一个spring?

- 容器与Bean、  AOP、  WEB MVC、  Spring Boot、其他

从自己面试的角度，课程中可以提炼出哪些值得面试的点？

- 如何阅读源码？
  - 先会用；
  - 对代码的全局结构有个大概的了解【官网、或者其他人的解析】
  - 挑几个核心的组件深入研究：先掌握思路，再深入掌握核心代码的细节！
    - 自己能手动写两个单元测试进行验证！！！！试着自己模拟实现组件
  - //有需要的时候，深入阅读组件的其他地方源码



#### 整个课程的每一章讲了什么内容？

- 什么是 BeanFactory和ApplicationContext ？他们之间有什么关系？  //==todo:发布事件可以和最后的P173关联；==
  -  BeanFactory[默认实现类  DefaultListableBeanFactory]才是==Spring的核心==，里面有属性singletonObjects[来自父接口DefaultSingletonBeanRegistry]，是一个map，保存了所有的bean对象，key为 bean名字，vlaue为实例对象
  - ApplicationContext及其子类称为spring容器，是BeanFactory的子类，有个属性beanFactory，容器就是在BeanFactory基础上扩展了些能力；扩展的能力包括：
    - 处理国际化资源的能力  context.getMessage() 
    - 依据通配符  获取磁盘上资源的能力 getResources()
    - 获取环境变量（系统环境变量等）、application.properties等
    - 发布事件对象 
  - fields用法
    - https://vimsky.com/examples/usage/field-get-method-in-java-with-examples.html
    - Field.get(obj)意思是获取obj对象的Field.getName()对应的属性
  
- BeanFactory常见的实现有哪些  底层是什么样的？或者说bean是如何添加到beanFactory的？或者说beanFactory有什么特点？
  - spring中 容器在关联了BeanFactory后处理器、Bean后处理器之后，会去执行对应的后处理方法，从而能够解析如@Autowired  @Resource  @Bean @Configuration等注解，将bean定义添加到BeanFactory中（包括创建bean定义、注册bean两个步骤）
  -  不会主动初始化单例  当第一次用的时候，才会真正创建实例; 如果希望初始化时创建所有的单例对象，可以使用preInstantiateSingletons()
  - 不会主动解析${}与#{}   //==todo:可以和 后面专门讲解析  \$ #的部分练习起来==
  
- ApplicationContext常见的实现有哪些  底层是什么样的？或者说Bean是如何添加到ApplicationContext的？
  - ApplicationContext常见的实现有三种：
    - ClassPathXmlApplicationContext   依据xml文件加载BeanDefinition； 
    - AnnotationConfigApplicationContext[非web环境] : 基于配置类添加beanDefiniton；
    - ==AnnotationConfigServletWebServerApplicationContext[web容器]==：既支持基于配置类添加beanDefiniton，又支持内嵌servlet的Web容器---tomcat
  
- springBean生命周期有哪几个阶段？  自定义的类如何利用后处理器进行增强，如在调用构造方法前执行特定的方法？
  - springBean生命周期  执行顺序：构造，依赖注入，初始化，销毁[单例才调用？其它scope似乎不大一样，关注后面的scope]；
  - 增强的方式：实现后处理器增强接口，重写增强方法即可，增强的时机有——实例化前后、依赖注入、初始化前后、销毁时；
  
- ==todo: 设计模式学习，模板方法==

- @Autowired解析底层原理？如何自己实现解析＠Autowired？
  - @Autowired解析时通过bean后处理器是实现的[AutowiredAnnotationBeanPostProcessor]，主要包括三步，首先，找到所有添加了@Autowired注解的方法，并封装为InjectionMeradata对象[findAutowiringMetadata]；接着调用metadata.inject方法进行依赖注入；
  - 进一步模拟Inject方法的实现，难点在于依据类型从beanFactory中获取bean对象：beanFactory.doResolveDepedency //==doResolveDepedency和geBean的区别是什么？doResolveDepedency是在beanFacotry中还没有的情况下，生成bean对象（很可能还会注入beanFactory），getBean是在beanFacotry中依据有对象的情况下，直接获取，可能会获取到null==
  
- @ComponentScan解析底层原理？如何自己实现解析@ComponentScan？
  - 1. 获取类上的注解对象 AnnotationUtils.findAnnotation(clazz class, clazz annotation); 2 依据注解中的包名获取对应的类资源[class文件]context.getResources(path)； 3 依据class文件获取类的信息和注解的信息 CachingMetadataReaderFactory.getMetadataReader   MeradataReader.getAnnotationMetadata 4.如果类上包含@Component或者类中方法上包含@Component的子注解(hasMetaAnnotation)，则依据类信息生成BeanDefinition[BeanDefinitionBuilder.genericBeanDefinition().getBeanDefinition]并注册到beanFactory[beanFactory.registerBeanDefinition]
  - tips: AnnotationBeanNameGenerator解析@Component获取bean的名字；一个类实现BeanFactoryPostProcessor并且注册到beanFactory，它的postProcessBeanFactory方法会在refresh的时候回调 
  
- @Bean 解析底层原理？如何自己实现解析@Bean？ 
  
  - 1.获取类的元信息 2. 获取类中带有@Bean的方法  3. 基于方法生成对应的BeanDefinition [方法名  自动注入模式  初始化方法名，BeanDefinitionBuilder.genericBeanDefinition().getBeanDefinition] 并将bd注册到beanFactory[beanFactory.registerBeanDefinition]
  
- @MapperScanner 解析底层原理？如何自己实现解析MapperScanner？ 
  - 1.获取对应的类资源[getResources()]，并进一步获取类的信息[getMetaDataReader().getClassMetadata]；2.如果是接口，则生成对应的beanDefinition[类型是MapperFactoryBean.class并设置构造方法入参为接口名、自动装配模式]；3.获取接口对应的beanName；4.注册到beanFactory
  - 说明：和前面较大的不同有两点
    - beanFactory只能管理对象，要添加接口则需要接口转对象——即用MapperFactoryBean<T>封装，并指定sqlSessionFactory即可；
    - beanName获取要注意下，获取接口的beanName不是类的；
    - //setAutowiredMode 设置了自动装配模式后，factory会自动去工厂中找sqlSessionFactory并装配；
  
- Aware接口、InitialzingBean接口有什么用？什么情况下只能用Aware接口、InitialzingBean接口而不能用@Autowired、@PostConstruct？或者说什么情况下@Autowired、@PostConstruct两个注解会失效？
  - 接口功能
    - aware接口：bean名字注入、BeanFactory注入、ApplicationContext容器注入、解析${}等；
    - InitializingBean接口：为bean添加初始化方法；
  - @Autowired、@PostConstruct注解失效的场景：在配置类中存在BeanFactoryPostProcessor的情况，注解失效，只能用上述两个接口
    - 失效原因：注解@Autowired、@PostConstruct依赖容器的后处理器；正常情况下，容器会依次执行  BeanFacotryPostProcessor、BeanPostProcessor、java配置类（java配置类中可能包含@Autowired注解），注解正常解析；但是在配置类中存在BeanFactoryPostProcessor的情况，配置类背提前创建(因为要创建BeanFactoryPostProcessor的缘故)，此时后处理器未准备好，故注解失效；
  
- spring中有哪几种添加初始化或者销毁的方法？

  - 按照优先级顺序，初始化方法依次是 @PostConstruct、实现initialzingBean接口、@Bean(initMethod="XX")
  - 按照优先级顺序，销毁方法依次是 @PreDestroy、实现DisposableBean接口、@Bean(destroyMethod="XX")

- spring中有哪几种scope？单例bean中注入其它scope的bean，scope会失效吗？怎么解决？

  - spring中有5种scope：singleton(容器关闭时销毁),  prototype（自行调用销毁）,  request（存在web的request域中，生命周期同request，==重发请求会销毁==）,  session(会话域，==长时间不发/重新打开浏览器/调用session的invalid方法 请求会销毁==),  application（应用程序域  启动时入servlet context，==似乎spring不会自动销毁 即使你关闭应用==）
  - 单例的bean直接注入其它域的类，注入类的scope会失效，对于单例对象只执行了一次初始化，所以内部属性的注入也只发生了一次；四种解决方法思想上都是一样的：推迟其它scope bean的获取，代理  工厂  applicatioinContext
    - 1.添加@Lazy解决  
      - 加了@Lazy之后注入的是代理对象，代理类虽然不变，但是使用代理对象的方法时会创建新的多例代理对象；
      - 2.使用@Scope中的属性proxyMode解决 ，底层原理也是生成代理
      - 3.(推荐)使用工厂类解决，由工厂创建多例对象
      - 4.(推荐)注入一个ApplicationContext，调用getBean方法解决；

  - AOP的实现方式有哪几种？各有什么优缺点？ //关联题：动态代理的原理

      - jdk代理：自己编写代理类增强；java自带；代理类没有源码，运行时直接生成字节码；只能针对实现了接口的类代理  //思维导图中有示例代码：
      - cglib代理：自己编写代理类增强；cglib是第三方的库代理类没有源码，，运行时直接生成字节码；目标类和代理类是父子关系，目标类不能是final，父类方法加了final也不能被增强，代理类需要重写被代理方法；
      - aspectj：借助apectj插件进行增强；编译时改写class实现增强，可以增强静态方法[代理不能增强静态方法]   
      - agent：类启动时增加VM options: -javaagent；类加载阶段修改class实现增强，可以突破代理实现aop的限制：一个方法调用另一个方法，被调用的方法无法增强(因为另一个方法会通过this调用，不走代理)； //推荐工具 Arthas工具，实现运行时的反编译补充jdk代理和cglib代理在性能优化上的区别：jdk代理，在第17此调用时为提升性能，会针对一个方法产生一个代理类， 反射调用优化为直接调用，提升性能；cglib第一次调用就会产生代理并直接调用，无需反射调用，每个类会生成两个代理类，[一个配合目标对象调用，一个配合代理对象调用]，但是一个类可以有多个方法，不用针对单个方法产生代理类；
      - 扩展jdk代理和cglib代理在性能优化上的区别(详情见后面的课程内容)：jdk代理，在第17此调用时为提升性能，会针对一个方法产生一个代理类， 反射调用优化为直接调用，提升性能；cglib第一次调用就会产生代理并直接调用，无需反射调用，每个类会生成两个代理类，[一个配合目标对象调用，一个配合代理对象调用]，但是一个类可以有多个方法，不用针对单个方法产生代理类；

  - jdk代理原理 // 应用诸如日志、权限、事务增强等；静态方法的反射调用 对象可以传Null;

      - asm运行时动态生成代理类的字节码，直接看jdk代理源码基本看不懂，可以用arthas工具查看反编译代理类的源码；jdk的代理会继承Proxy类，内含一个InvocationHandler属性;

      - ```java
        //模拟实现代理;后续还可以为invoke方法增加入参： 代理对象，method，方法入参；还有增加返回值、异常处理等；
        //梳理下调用链：proxy.foo -> InvocationHandler.invoke -> 在main方法中自己编写的增强方法；
        public interface Foo{
            public void foo();
        }
        static class Target implements Foo{
            @Override //示例中这个override可以省略，不知道为什么，或许java spring版本不同
            public void foo(){
                System.out.println("target foo");
            }
        }
        interface InvocationHandler implements Foo{
            void invoke();
        }
        
        public class $Proxy0 implements Foo{
            pirvate InvacationHandler h;
            public $Proxy0(InvacationHandler h){
                this.h = h;
            }
            @Override
            public void foo(){
        		h.invoke();
            }
        }
        
        public static void main(String[] param){
            Foo proxy = new $Proxy0(new InnvocationHandler(){
                @Override
                public void invoke(){
                    System.out.println("before...");
                    new Target().foo();
                }
            }
            );
            proxy.foo();
        }
        ```

- java是如何提升 jdk代理性能的？
  
  - method.invoke本质上是通过methodAccessor实现类实现；
  - 前16次用本地的java native api， 使用NativeMethodAccessor，性能低；第17次  为了提升性能，生成了一个代理类GeneratedMethodAccessor[Arthas查看得知]，直接直接正常调用即可；
  - //asm插件使用教程略；
  
- cglib代理原理？

  - 模拟实现：在使用mehtod调用目标方法是，和jdk代理基本一致，不重复写了，==唯一的区别是目标类不需要实现接口了，代理类也不需要实现接口而是继承目标类；[视频中的mehtodIntercept直接用了三方包的]==； //梳理下调用链：proxy.save ->methodInterceptor.intercept -> 用了三方包，不知细节，但是理论上和jdk代理类似，main中自己编写的方法即可

  - MethodProxy的使用：使用了三方包的MethodProxy类，提前获取MethodProxy【需要指定一个原始方法，即直接super.foo类似的形式直接调用父类方法即可】，然后作为参数传入methodInterceptor即可； //MethodProxy两个功能：调用目标方法的部分要从反射调整为 直接调用原始方法[直接调的核心是依据method对象知道自己要调目标/代理对象的哪个方法]；要同时支持 目标类、代理类；                                                  调用链梳理： proxy.save ->methodInterceptor.intercept -> 用了三方包，不知细节，但是理论上和jdk代理类似，main中自己编写的方法即可-> MethodProxy中的原始方法？？？

  - 模拟实现MthodProxy：核心的两个方法 getIndex[获取目标方法的编号，方法签名转换为整数编号] invoke[依据整数编号正常调用目标对象的方法]

    - ```java
      //结合目标对象使用
      public abstract class FastClass{
          private Class type;
          protected FastClass(){
              throw new Error("Using the FastClass...")
          }
      }
      public class TargetFastClass{
          static Signature s0 = new Signature("save", "()v");
          static Signature s1 = new Signature("save", "(I)v");
          static Signature s2 = new Signature("save", "(J)v");
      
          public int getIndex(Signature signature){
              if(s0.equals(signature)){
                  return 0;
              }else if(s1.equals(signature)){
                  return 1;
              }else if(s2.equals(signature)){
                  return 2;
              }
          }
         
          public Object invoke(int index, Object target, Object[] args){
              if(index == 0){
                  ((Target) target).save();
                  return null;
              }else if(index == 1){
                  ((Target) target).save((int) args[0]);
                  return null;
              }else if(index ==2){
                  ((Target) target).save((long) args[0];
                  return null;
              }else{
                  throw new RuntimeExceptioin("无此方法");
              }
          }
          public static void main(String[] args){
              TargetFastClasss fastClass = new TargetFastClass();
              int index = fastClass.getIndex(new Signature("save", "(1)v"));
              System.out.println(index);
              fastClass.invoke(index, new Target(), new Object[]{100});
          }
      }
      //结合代理对象使用：类似，不同的是方法编号获取代理类的方法编号；invoke调的是代理类的原始功能方法
                                        
      ```

  - cglib代理 对比 jdk代理????

- 容器、注解、后处理器梳理
  
  - 容器： GenericApplicationContext相比AnnotationConfigApplicationContext  ，很干净，没添加bean后处理器等；refresh()方法会执行工厂后处理器 初始化单例等；
  - bean后处理器及对应注解
    - AutowiredAnnotationBeanPostProcessor 
      - @Autowired 会自动注入[值注入需要配合@Value、bean对象]
      -  @Value 来自环境变量等[后面有Value详解]； //还要额外设置AutowireCandiateResoulver才能通过@Value实现值注入； 内含${}还需要额外 addEmbeddedValueResolver
    - CommandAnnotationBeanPostProcessor
      - @Resource @PostConstruct @PreDestroy
    - ConfigurationPropertiesBindingPostProcesssor 
      - @ConfigrationProperties  SpringBoot的bean的属性和配置文件的键值对 做绑定, 前缀+属性名去配置文件中找环境变量  //[后面有详解]
  - beanFactory后处理器及对应注解
    - ConfigurationClassPostProcessor   @Bean、@Component 、@Import 、@ImportResource
    - MapperScannerConfigurer[springboot自动配置  ]   扫描mybatis的mapper接口；  //SSM架构用的@MapperScanner底层也是MapperScannerConfigurer.class ； @MapperScan
  
- 小tip：
  
  - debug:单例注入多例，抛IllegalAccessException异常
    - 本质上是创建代理对象，打印时最后反射调用Object的toString] ，jdk9以后的版本，反射调用jdk中的类，会抛IllegalAccessException异常；
    - 解决方案：改JDK版本；自己重写 javaBean的toString 方法；运行时添加指定的配置  --add-opens；
  - 测试案例参考 session超时时间配制为10s：但是实际的超时时间是max(超时时间配置，检测session超时的频率)
  - @Autowired结合方法进行注入，可以打印信息查看是否注入成功；
  - @autowired  去找实例对象的依据是什么？@autowired和@Resource区别是什么？哪个优先级更高？//   ==todo:可以和后面@Autowired解析关联==
    - 依据类型[类或接口]匹配，有多个的时候[可以用qualifier指定？]会再匹配名字：即成员变量名字和（首字母小写的）类名，匹配上优先；
    - @Resource 类似，对于多个候选项的情况则可以用name属性指定名字
    - 默认优先级@Autowired > @Resource，依据是后处理器添加的时间(Autowired先)；可以用比较器控制后处理器的优先级顺序[getOrder获取]，数字小的优先



=//想学代理的玩法==

//内嵌的tomcat，神策上的代码是否 配置个WebConfig registrationBean能运行web？

//Bean工厂相关的后处理器，4个；Bean生命周期相关的后处理器，讲了2个？

课程地址：

https://www.bilibili.com/video/BV1P44y1N7QG/?vd_source=8bd5ab544d4cb8d9821752b68ce53b11

梳理的时候多思考？面试的话哪些问题可以提取？   学的内容在自己日常该做中是不是可以有应用？

#### 问题小结：

- jdk代理的原理？如何自己实现代理？Cglib?? 代理的模拟实现自己手写下？？
- 想体验下微软的面试？看下人家和我之间的差距？

看源码技巧


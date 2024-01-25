//内容很多很杂，似乎没有什么重点，是不是可以简单看下哪些==知识点是核心==，==哪些思想是核心==，最后汇总成一个问题：如何自己从头实现一个spring?     问题梳理完还需要继续

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
  
- ==todo: 设计模式学习，模板方法; 责任链模式;组合模式==

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

  - cglib代理 对比 jdk代理???? 参考AOP的实现方式有哪几种？部分的扩展

- 切面、通知、切点、advisor之间是什么关系？spring中AOP是如何选择代理方式的

  - Advisor：单组 通知+切点； 切面：一组或多组的Advisor，即多组 通知+切点。

  - ==proxyFactory==的父类ProxyConfig有一个 ==proxyTargetClass==属性（记忆角度可以理解为是否优先用cglib）；

    -    proxyTargetClass为false且目标 实现  接口，用jdk实现

         proxyTargetClass为false且目标 没有实现 接口，用cglib实现

         proxyTargetClass为true，用cglib实现

  - ```java
    //示例代码(了解下，不是重点)：基于MethodInterceptor等工具类自己模拟实现切面
    //1.备好切点
    AspectJExpressionPointcut pointcut = new AspectJEpressionPointcut();
    pointcut.setExpression("execution(* foo())");
    //2.备好通知  MethodInterceptor本质上是一个环绕通知
    MethodInterceptor advice = invocation -> {
        System.out.println("before...");
        Object result = invocation.proceed();//调用目标
        System.out.println("after...");
        return result;
    }
    //3.备好切面
    DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut, advice);
    //4.创建代理
    Target1 target = new Target1();
    ProxyFactory factory = new ProxyFactory();
    factory.setTarget(target);
    factory.setAdvisor(advisor);
    I1 proxy = (I1) factory.getProxy();
    proxy.foo();
    proxy.bar();
    ```

- spring切点匹配有哪几种方式？底层原理是什么？

  - spring切点支持 表达式匹配、注解匹配两种； 
    - execution(返回值  包名-类名-方法名 ) + [底层是调用]AspectJExpressionPointcut.matches方法； 
    - @annotation(注解的 包名-类名)  + [底层是调用]pointcut.matches方法；
  - 特殊切点：用于判断一个方法或方法所属的类上或父接口上是否有注解 //例如@Transactional
    - 需要自定义一个切点，重写matches方法，推荐使用StaticMethodMatcherPointcut
    - //Tips：MergedAnnotations.from(method) 获取方法上所有注解；  MergedAnnotations.from(targetClass, MergedAnnotations.TYPE_HIERARCHY);  可以自己设置查找策略，默认只找方法、类上，设置为SearchStrategy.TYPE_HIERARCHY【表示还会查找继承树】

- spring AOP底层的原理是什么[advisor层]？spring AOP是怎么实现的[advisor层]？//最底层是代理，这里主要讨论的是代理和高级切面的中间层，==advisor层==

  - AnnotationAwareAspectJAutoProxyCreator.class有两个核心方法：findEligibleAdvisors、wrapIfnecessary
    - findEligibleAdvisors : 找到所有的切面，包括高级切面Aspect[会转换成低级切面]和低级切面Advisor
    - wrapIfNecessary:是否有必要创建代理，内部也是调用findEligibleAdvisors，返回值不为空说明有必要；
  - 扩展 AnnotationAwareAspectJAutoProxyCreator 生效时机：通常生效时间（下发(*)占位的地方）：依赖注入之前   初始化之后 //创建-->  ( * )依赖注入-->初始化(* ) 
    - 正常情况（初始化完成之后创建代理对象）：bean2中注入bean1； bean1先注册到容器，则bean1代理对象在bean1初始化之后生成；
    - bean1、bean2循环依赖（构造和依赖注入之间创建代理对象，且会存入二级缓存）：bean1注入容器时：发现要注入bean2，于是进入bean2的流程【的构造  依赖注入  初始化】，bean2的依赖注入需要bean1代理对象，于是bean1的代理提前注入，最后bean1的依赖注入  初始化；
    - 扩展：依赖注入和初始化不应该增强被增强，仍应该被施加于原始对象；

- spring AOP底层的原理是什么[通知层]？spring AOP是怎么实现的[通知层]？

  - 不同通知最终统一转换为环绕通知 MethodInterceptor，并把转换后的通知添加到调用链MethodInvocation中，最后执行调用链即可【MethodInvocation.proceed()   使用了设计模式中的责任链模式】;// spring提供多个适配器完成转换工作，例如MethodBeforeAdviceAdapter会把@Before对应的  AspectJMethodBeforeAdvice适配为MethodBeforeAdviceInterceptor；getInterceptorsAndDynamicInterceptionAdvice()会把不同通知转换成环绕通知

  - //静态通知、动态通知：整体的思路一样，不同的只是 低级切面中通知的实现类实现类 要包含切点对象

  - //某些通知内部需要用到调用链，需要将MethodInvocation放入当前线程(且需要加在最外层)，通知便可以从当前线程取；

  - 一般适配器包含两个方法：supportAdcice判断是否是指定的Advice； getInterceptor用于进行类型转换； //适配器模式体现：把一套接口转换成另一套接口【转换的类对象成为适配器对象】，以便适合某种场景的调用

  - ==责任链模式  原理：本质上就是一个递归调用——先调用所有的环绕通知，最后调用目标；这里的递归比较隐晦： 责任链的proceed() --> 通知的invoke,同时把自己作为入参 -->责任链的proceed() ;==   核心：入口是责任链的proceed，递归调用methodInterceptorList中的元素方法，Lisit里面的元素回调责任链的proceed

    - ```java
      //责任链  示例代码
      static class MyInvocation implements MethodInvocation{
          ....
          public Object proceed(){
              if(contu > methodInterceptorList.siz()){
                  return method.invoke(targe, args);//调用目标，返回结束递归
              }
              MethodInterceptor methodInterceptor = methodInterceptorList.get(count ++ -1);
              return methodInterceptor.invoke(this);//递归嗲用通知链
          }
          
      }
      static class Advices implements MthodIntercepter{
          public Object invoke(MethodInvocation invocation{
              System.out.println("Advice2.before()");
              Object result = invocation.proceed();
              System.out.println("Advice2.after()");
              return result;
          }
      }
      
      ```

  - 动态通知与静态通知：

      \- 静态通知：通知方法没有入参，即不带参数绑定，性能相对高，执行是不需要切点； 

      \- 动态通知：通知方法有入参，需要参数绑定，执行时需要切点；

- spring生成的代理对象有什么特点？

    - spring自动 依赖注入和初始化使用的是目标对象方法； 生成代理对象之后，切点才会生效；
      - 例：bean1注入bean2，且有一个初始化方法；创建一个切面类切点为bean1所有方法；context中获取bean1对象的时候会发现：spring自动  针对bean2的注依赖入和初始化都没有被增强；
  - 代理对象和目标对象并不公用属性
    - 可以发现代理对象的两个属性均为空值，但是目标对象有值，因为依赖注入和初始化针对目标对象；  代理对象的getBean2()  isInitialized()等方法底层调用的还是目标对象的属性；
  -  static方法  final方法  private方法 无法被增强，只有可以被重写的方法能被增强；

- tomcat、dispatcherServlet之间是什么关系？

  - spring要支持web容器，配置类有三项必须配置：内嵌web容器工厂、DispatcherServelet的Bean定义，注册bean[用于把DispatcherServlet注册到Tomcat]。其中DispatcherServelet是spring容器创建，但是初始化在tomcat完成[tomcat容器默认在首次使用dispatcherServlet的时候初始化，初始化时机可通过loadOnsStartup配置]，是springMVC程序的入口点。

- dispatcherServlet初始化包括哪些组件？各个组件的初始化流程如何？各个组件分别又有什么用处？

  - 代码位置dispatcherServlet-->onRefresh-->initStrategies，初始化下面的九类组件；
      - initMultipartResolver：初始化  文件上传解析器
      - initLocaleResolver：初始化 本地化解析器，属于哪一种国家、地区、语言；//有多种实现：请求头中accept头获取相关信息  从cookie中获取等；
      - //initThemeResolver：不重要
      - initHandlerMappings： 初始化 路径映射器，请求下发到controller；
      - initHandlerAdapters   
        - 适配 不同形式的控制器方法，并调用它；
        - handler：具体处理请求的代码，有多种形式；
      - initHandlerExceptionResolvers   解析异常
      - //后面三个不重要
  - 核心组件的初始化流程 //handlerMapping依据request找到对应的handlerChain，handlerAdapter调用具体的handler（handlerChain的信息作为入参）
      - initHandlerMappings：找到所有的HandlerMapping，依次查找父容器、当前容器，如果容器没有，使用默认的HandlerMapping，在DispatcherServlet.properties中配置；
        - （不足时似乎视频没给出具体的类名、方法名）例如：RequestMappingHandlerMapping  HandlerMappin的常见实现之一；解析@RequestMapping及派生注解，建立 请求路径----控制器方法之间的映射关系；核心方法：getHandler(request)
        - 初始化过程：先到当前容器下找到所有控制器类，查看控制器有@RequestMapping及其派生注解【包括GetMapping等】的方法并记录  路径--->控制器方法  信息【getHandlerMethods方法可以查看】并保存到RequestMappingHandlerMapping     ////默认的RequestMappingHandlerMapping 创建的RequestMappingHandlerMapping对象会作为dispatcher的属性，但是不会放入Spring容器中； 可以在WebConfig中，自己添加bean定义RequestMappingHandlerMapping
      - HandlerAdapters：实现了HandlerAdapter接口，即处理器适配器，作用是调用控制器方法； 核心方法：invokeHandlerMethod【调用handlerMethod】  //初始化内容课程似乎没提及

- spring中如何添加自定义的参数解析器、返回值处理器? ==//课程的两个示例很有参考价值，建议收藏==

  - 自定义参数解析器：

    - \- 自定义注解；

      \- 自定义resolver并实现HandlerMethodArgumentResolver接口，重写两个方法 supportsParameter、 resolveArgument

      \- 将自定义的参数解析器，加入到RequestMappingHandlerAdapter的实现类中；

      //后续使用的时候只需要在方法参数前添加注解即可；

  - 自定义返回值处理器  //依据返回值类型、方法是否加某个注解进行特殊处理
    - 自定义注解；
    - 自定义resolver并实现HandlerMethodReturnValueHandler接口，重写两个方法  supportsReturnType、handleReturnValue
    - 将自定义的参数解析器，加入到RequestMappingHandlerAdapter的实现类中；
    - //后续使用的时候只需要在方法上添加注解即可；

- 常用的参数注解有哪些？spring参数解析的底层原理是什么？//顺便纪录下参数解析相关的核心方法  

  - spring  使用了组合模式【handlerMethodArguments】，需要依次调用每个Resolver.supportsParameter方法，直到找到一个 支持此参数的解析器；
    - 没有@RequestParam的其它参数 如带@PathVariable也会被认为省略了@RequestParam[尝试用RequestParamMethodArguementResolver解析]，会报错；//后面会学到用组合模式+ 可省略/不可省略两个解析器可以解决；
  - 常用的参数注解及解析的内容、对应的解析器： 
    - @RequestParam+String : 普通请求参数param，RequestParamMethodArguementResolver
    -  没加解析参数+String：等价1，省略了@RequestParam；但是要配置支持省略  
    - @RequestParam+类型转换  ：多指定一个类型转换器——dataBindFactory  
    - @RequestParam 注解属性中包含${} ，即从环境变量获取默认值 ：RequestParamMethodArguementResolver的第一个参数要指定beanFactory用于读取环境变量、配置文件
    -  @RequestParam  + MultipartFile 上传文件  ： RequestParamMethodArguementResolver即可
    - @PathVariable   + int // test/{id}   ：RequestParamMethodArguementResolver，请求中需要将请求路径的{id}和实参对应起来，结果放入request作用域[key是固定的]
    - @RequestHeader + String //解析请求头数据，RequestHeaderMethodArgumentResolver  ，需要BeanFactory  
    - @CookieValue + String：//解析Cookie数据，ServletCookieValueMethodArgumentResolver  
    - @Value 注解属性中包含${}/#{}+Stirng   获取Spring中数据 :ExpressionValueMethodArgumentResolver
    -  特殊类型  //包括request response session等；  ServletRequestMethodArgumentResolver
    - @ModelAttribute  + 自定义类型：参数解析器处理结果会保存到MVCContainer  ServletModelAtrributeMethodProcessor
    - 自定义类型：等价上面，省略了@ModelAttribute；但是要配置支持省略   //要放在例2的省略版解析器前面
    -  @RequestBody + 自定义类型: 请求体获取数据    RequestResponseBodyMethodProcessor，需要一个MessageConverter  
  - //比较关键的方法：method----initParameterNameDiscovery是为了解析方法入参的参数名；getParameterAnnotations获取参数上的所有注解名;   resolver----supportsParameter判断时否支持某种参数；resolver.resolveArgument  真正解析参数得到实参；

- spring编译会保留参数名吗？有几种方式可以保留参数名？

  - \- 编译    反编译 可以发现编译默认是不保留参数名；解决方案：

      \- 可以添加-parameters[反编译发现class中有MethodParameter，其中包含参数名称信息，参数可以反射获取] ；

      \- 或者-g[反编译会发现class中有LocalVariableTable，其中包含参数名称信息，参数名 反射无法获取，但是可以ASM获取【LocalVariableTableParameterNameDiscoverer底层用ASM】;  只能获取类的参数名，==对接口不生效==]  //反编译  ： javap -c -v .\Bean1.class 

      \- spring中参数名获取默认是DefaultParameterNameDiscoverer， 结合了两种(MethodParameter+LocalVariableTable)，

  - 其它：idea的src，自动编译会加一些选项，如-p，大坑，吃过大亏！！idea启动正常，然后自动化部署就报错；

- spring中是如何实现数据绑定中的类型转换的？

  - spring中的类型转换接口都实现了TypeConverter，转换时，有一个 TypeConverter Delegate会 视情况委派 ConversionService与PropertyEditorRegistry真正执行转换；优先级依次如下：优先级：PropertyEditor【来自jdk】中自定义的【@InitBinder添加】；ConversionService【来自spring】；PropertyEditor默认的；特殊处理??；
  - TypeConverter 常见的实现有四种： SimpleTypeConverter（仅做类型转换）    BeanWrapperImpl（为bean属性赋值，类型转换走反射的get set）  DirecFieldAccessor（为bean属性赋值，类型转换走反射的成员变量赋值）  ServletRequestDataBinder（绑定配置文件属性和Bean属性，directFieldAccess为真则走Field；） ，web环境下推荐ServletRequestDataBinder且设置directFieldAccess为false？？？

- spring中如何自定义类型转换方法扩展 数据绑定中的类型转换？

  -  ServletRequestDataBinderFactory指定扩展方法 + @InitBinder声明扩展方法；//底层用的是PropertyEditorRegistry   PropertyEditor
  - ServletRequestDataBinderFactory指定initializer + FormattingConcersionService添加自定义扩展转换器并封装入ConfigurableWebBindingIntializer //底层用的时ConversinService Formatter
  - 上述两者同时存在时@InitBinder优先级更高
  - DefaultFormattingConversionService/ApplicationConversionService + @DateTimeFormat指定日期格式
    - 默认的ConversionService[其实内置了对特殊日期格式的解析，会针对注解自己添加转换器]配合@DateTimeFormat指定日期格式；  DefaultFormattingConversionService或ApplicationConversionService[springBoot中]；

- 如何获取一个类的泛型信息？

  - jdk获取：getGenericSuperclass获取有泛型信息的父类；有泛型信息 类型是ParameterizedType；getActualTypeArguments获取泛型参数(可以有多个)；
  - spring获取：resolveTypeArguments入参：子类类型、父类类型（也可以有多个，获取单个对应的方法为resolveTypeArgument）

- spring中@InitBinder是什么时候完成解析的？由谁解析？

  - @InitBinder由RequestMappingHandlerAdapter解析，解析时机视添加的位置而定：@Controller【只对单个controller生效；getDataBinderFactory被调用时完成解析】、@ControllerAdvice中【全局，对所有控制器生效；初始化的时候完成解析】；
  - ////@ControllerAdvice可搭配的注解：@InitBinder【添加自定义类型类型转换器】  @ExceptionHandler【异常处理】  @ModelAttribute【返回值添加到 ModelAndView中】

- spring中控制器方法的执行流程是怎样的？

  - 先是==RequestMappingHandlerAdapter(图2)== 这边的 准备工作：准备 数据绑定工厂(解析Advice中的@InitBinder)、模型工厂(例如 解析Advice中的@ModelAttribute)，[控制器的临时数据会保存到ModelAndViewContainer]；     然后是==ServletInvocableHandlerMethod== 完成调用：准备参数【涉及RequestBodyAdvice等？】（包括 数据绑定&&类型转换；参数名解析；参数解析等）、==反射调用方法==、返回值解析【涉及RespoonseBodyAdvice等?  例如MVC中添加Model数据、处理视图名是否渲染等】、最后从ModelAndViewContainer中获取最终结果；

- @ModelAttribute有几种用法？底层的实现原理是什么？

  - \- 第一种用法：==加在参数名上==  流程：参数解析器[ServletModelAttributeMethodProcessor]   //RequestMappingHandlerAdapter内部就包含了参数解析器；

      \- 参数解析器的解析流程：调用对象构造方法，用数据绑定工厂 绑定空对象和参数，最终的对象放入MVCContainer[默认名称，对象类型首字母小写]；

    \- 第二种用法：==加在controller的某个方法名上==[只对单个controller生效，而不是单个方法！！！！]：解析者变为RequestMappingHandlerAdapter，ModelFactory[模型工厂]在标注了@ModelAttribute方法被调用后，会把返回值放入MVCContainer(默认名字为方法返回值类型首字母小写)； //扩展：加在@ControllerAdvice标注的类的方法上则对所有的方法生效； 

  - //初始化的  afterPropertiesSet()  方法会自动找到所有标注@ModelAttribute的方法并记录；

- spring中返回值有哪几种类型？返回值处理器的底层原理是什么？

  - 返回值有7种类型：@ModelAttribute+@RequestMapping[默认视图会取路径]+自定义类型、自定义类型[省略@ModelAttribute]     不走视图渲染的三个方法（对应的handlerReturnValue中会把setRequestHandler(true) ）:【RequestHandler为true】： HttpEntity<T>   【状态码  响应头  响应体】HttpHeaders【只有响应头】   @ResponseBody【只有响应体】
  - 核心就是两个方法：supportsReturnType[是否支持 输入的  返回值类型]；handlerReturnValue[进行对应的处理；不同的类型对应着不同的处理器 最后用一个组合模式串起来即可 //==渲染的模板哪里来的？MVC有路径，去资源路径下找即可==
  - 可以简单分为两类，前4种为一类，都是 把模型和视图名添加到MVCContainer，然后视图渲染（不同的只是模型和视图名的来源不同）；后3种为一类， 用MessaeConverer把响应体的内容转换成json数据；还可以自己设置状态码、响应头等，最后把结果放到Response中(不同的只是是否有响应体 及是否有默认响应头)。
    -  ModelAndView  //handlerReturnValue会把模型和视图名添加到MVCContainer，然后视图渲染
    - String和ModelAndView类似，string就是视图名  【少了添加MVC到容器】
    - ModelAttribute+自定义类型+@RequestMapping[默认视图会取路径]
        - handlerReturnValue会把模型添加到MVCContainer，视图名则从请求路径获取   //==后面想看下MVCContainer效果；user=User{...}==
      - 省略了@ModelAttribute，和上一种情况一样；
      - HttpEntity<T>   包括：状态码  响应头  响应体；//示例中  handlerReturnValue： 用MessaeConverer把响应体的内容转换成json数据；还可以自己设置状态码、响应头等；
      - @HttpHeaders：和HttpEntity<T>类似，不同 的是只有响应头有值
      - @ResponseBody  和HttpEntity<T>类似，handlerReturnValue： 用MessaeConverer把响应体的内容转换成json数据；//会自动生成部分响应头[有默认值Content-Type-application/json]

- 常见的消息转换器有哪些？sping中哪里用到了消息转换器？spring中如何指定消息转换器？

  - MappingJackson2HttpMessageConverter(对象----JSON 的互相转换)、MappingJackson2XmlHttpMessageConverter(对象转XML)。//canWrite(..)  write  canRead read
  - 消息转换器应用场景：@requsetBody对应的入参处理器解析转换为JSON串、@ResponseBody对应的返回值处理器等；
  - 消息转换器的执行顺序(==针对返回值处理而言，请求参数处理不一定  视频未验证==)：多个转换器的执行顺序，1.看响应中的ContentType(通常可以直接看contreller的@RequestMapping是否指定)若;2.看请求中的Accept是否指定；3.默认messageConverter  List的顺序；

- spring如何对请求体、响应体进行增强？

  - @ControllerAdvice标注类 实现  ResponseBodyAdice<Object>  接口同时重写supprts  beforeBodyWrite方法【supprts方法return true的情况下才会调用beforeBodyWrite方法】
  - 例：Result类直接返回，不是则可以自动包装为Result；

- @Exception 底层原理是什么？ @ExceptionHandler底层的原理是什么？ //==异常处理的层级，自己try catch；方法对应的类有@ExceptionHandler标注的方法；@AdivceController标注的类中有@ExceptionHandler标注的方法；tomcat/BasicErrorController==

  - @Exception：普通的controller请求会调用  dispatcherServlet的doDispatch方法：【获取handlerAdaptor对象、调用handle()，】如果有异常 会先记录，后续调用processDispatchResult时，有异常的话会进一步调用processHandlerException【会调用异常处理器(handlerExceptionResolver)处理异常，常用的异常处理器的实现之一是 ExceptionHandlerExceptionResolver，专门解析@ExceptionHandler】；
  - @ExceptionHandler： 相关的核心类是ExceptionHandlerExceptionResolver，
    - spring初始化阶段 :会为ExceptionHandlerExceptionResolver设置消息转换器；【调用resolver.afterPropertiesSet()会自动】设置一些默认的参数解析器、返回值处理器；
    - ==真正处理异常可以调用resolveException()==        ==【该方法会查看 抛异常方法对应的类中是否有@ExceptionHandler及相关注解的方法，有的话进一步判断注解的异常处理范围与当前捕获的异常是否匹配，匹配则反射调用@ExceptionHandler及相关注解标注的方法{这里就是handle}】==; 
    - 而且可以处理嵌套异常：会被逐层展开  变成一个数组以保证被嵌套的异常也能被处理
  - @ControllerAdvice+@ExceptionHandler组合用法：会先找抛异常方法所在的类内是否有@ExceptionHandler标注的方法，如果没有的话，会找@ControllerAdvice注解类 内 包含@ExceptionHandler注解方法，异常会由方法处理
    - 容器中的ExceptionHandlerExceptionResolver初始化方法afterPropies方法中会调用initExceptionHandlerAdviceCache()，该方法会 查找context中所有的@ControllerAdvice标注的Bean，并遍历找到其中包含exceptionhandler注解的方法，加入cahce，方便后续从cahce中取异常处理方法并调用
  - 什么情况下的异常 @ControllerAdvice+@ExceptionHandler也无法处理？会由谁处理？
    - 制器的异常可以被ControllerAdvice处理，但是如filter中的异常不会被处理，需要更上层的异常处理者；其实tomcat是自带默认的异常处理器的，会自动返回异常的起因等等；
    - 处理springboot抛出的未经处理的异常有两种方式，一是使用tomcat自带的异常处理机制，即ErrorPageRegistrar；二是使用BasicErrorController; spring默认第二种
    - tomcat自定义异常处理地址：errorPageRegistrar 设置tomcat出错时默认的错误页面地址，可以是servlet 、静态页面或者自定义的controller的地址[底层是请求转发，浏览器现实的地址不变]；errorPageRegistrarBeanPostProcessor [在创建TomcatServletWebServerFactory的时候会自动回调]会找到容器中所有errorPageRegistrar()，调用对应的方法来添加errorPage；
      -  //tomcat捕获到spring框架外的异常会保存到Request域中，所以异常信息可以从Request域中获取；
    - BasicErrorController处理 spring未处理的异常，匹配的错误路径：1.(配置文件定义的属性)servler.error.path 2. (配置文件定义的属性)error.path 3. 默认/error；
      - 需要自己添加配置Bean  BasicErrorController，入参： ErrorAttributes[要显示的异常内容,如时间 错误路径等]   ErrorProperties[要读取的配置文件的键值信息] 
      - 支持不同的响应格式[json格式[如postMan请求]   html格式[如浏览器请求] ]  
        - 返回json格式的数据不需要特殊处理；  
        - 返回格式为html[如浏览器请求 postman设置Accept为text/html]，返回ModelAndView需要视图渲染，故需要指定视图名（这里是error）、视图解析器并自定义对应名的视图，这里用bean 定义视图+视图解析器【这里的BeanNameViewResolverl会被交给dispatcherServlet, 可以依据View的名称error找到@Bean名字为error的View】//自定义视图的render方法的model中包含了异常信息，即BasicErrorController的返回值会被添加到MVC

- spring支持哪些 映射器和适配器 组合？各自要怎么用？ //==注意总结各组之间的区别==

  - 之前用的 映射器和适配器 组合，组合一：RequestMappingHandlerMapping + RequestMappingHandlerAdapter; 
    - 作用：路径映射[需要解析@RequestMapping及其派生注解]    调用控制器方法[解析参数  调用 处理返回值]
  - 组合二：BeanNameUrlHandlerMapping   +  SimpleControllerHandlerAdapter  （spring更为早期的实现）
       - BeanNameUrlHandlerMapping   不是去找@RequstMapping注解的方法，而是去找  名字是/ 开头的bean
      - SimpleControllerHandlerAdapter   [要求控制器的类必须实现Controller接口，并重写handleRequest方法（即最终要调用的控制器方法）]
  - 组合三：自定义MyHandlerMapping + 自定义MyHandlerAdapter  //模拟实现组合二的逻辑
      - MyHandlerMapping 实现 HandlerMapping  接口，重写getHandler方法，找到请求对应的controller并封装为HandlerExecutionChain//可以在初始化方法提前准备映射关系：找到所哟实现了controller接口且名字是 / 打头的bean；
      - MyHandlerAdapter  实现 HandlerAdapter  接口并重写三个方法：supports(当前的Adapter是否能处理输入的Handler，这里的例子则要求实现Controller接口)  handle(调用handleRequest)  getLastModified(已经不用了)
          - //getLastModified已经过时；handle返回null表示不视图渲染流程；
  - 组合四：RouterFunctionMapping与HandlerFunctionAdapter (spring5.2才有，处理简单逻辑时相对简洁)
      - 初始化时会找到容器中所有的RouterFunction（包含RequestPredict、HandlerFunction），请求来了，RouterFunctionMapping匹配到对应的handlerFunction(处理函数)，最后由HandlerFunctionAdapter调用handler
      - 对比组合一，区别在于映射的key(这里是RequestPredict, 组合一为RequstMapping ) 和  处理函数的形式（要实现HandlerFunction接口, 组合一为控制器具体方法）; 参数解析、返回值处理等扩展功能相对少，但是简洁
  - 组合五：SimpleUrlHandlerMapping + HttpRequestHandlerAdapter + ResourceHttpRequestHandler//处理静态资源的
      -  SimpleUrlHandlerMapping映射；ResourceHttpRequestHandler 作为处理器处理静态资源[本质就是一个静态资源目录]；HttpRequestHandlerAdapter调用处理器；
      - 对比组合一，区别在于映射的key(ResourceHttpRequestHandler的beanName[一般包含通配符] ) 和  处理函数的形式（静态资源对应的ResourceHttpRequestHandler）
  - 欢迎页映射器 [静态]  WelcomePageHandlerMapping+ ResourceHttpRequestHandler+ SimpleControllerHandlerAdapter    //将访问根路径 / 的请求映射到欢迎页【可以是静态资源或者控制器，这里只讲静态资源】 //springBoot才有
      - WelcomePageHandlerMapping中内置了一个handler，即ParameterizableViewController[内部实现了controller接口，所以后面适配器配合SimpleControllerHandlerAdapter]，作用是依据视图名找视图【视图名固定未forward:index.html】；找视图则用到了 ResourceHttpRequestHandler【第五组有用到，示例中配置的查找路径是static/】
      - 对比组合一，区别在于映射的路径固定为forward:index.html 和  处理函数的形式  跳转到forward:index.html跳转过程进一步用到了处理函数 [静态资源对应的ResourceHttpRequestHandler] ）  //有一点视频没说清楚，需要额外自定义/**  对应的 ResourceHttpRequestHandler，  还是WelcomePageHandlerMapping有内置的声明了？看它设置resource的流程，应该是内置的；

- ==MVC处理流程是怎么样的？==

  - ==课程36总结得很烂，可以先看看javaGuide的流程，结合前面的课程做些扩充，最后结合36讲的总结做个补充；==

- 如何搭建一个spring项目？

  - 略，课程有骨架搭建  war项目搭建的流程；

- 简单叙述springBoot的启动流程？

  - SpringApplicaion.run --> new SpringApplication(primarySources).run(args);
  - 主要内容分为两块，SpringApplication构造，调用run方法 【12大步骤 7大事件】
  - 构造方法（准备工作，run方法创建spring容器）
    - 获取Bean Definition源：配置类、xml文件等等；//引导类也是个配置类； 
    - 推断应用类型：推断应用类型   springBoot支持三种应用类型：非web程序、基于servlet的web程序、reactive的web程序； 基于jar包中关键类判断属于哪一种，创建不同类型的ApplicationContext； //ClassUtils.isPresent 判断类路径下是否存在某个类
    - ApplicationContext初始化器：可以添加初始化器对ApplicationContext添加扩展；//initialize方法的入参就是  applicationContext
    - 监听器与事件：可以添加监听器 监听spring发布的事件 //入参event就是生成的事件
    - 主类推断：推断主类即运行main方法的类；
  - run方法
    - 得到SpringApplicationRunListeners并发布 application starting事件 //名字取得不好，实际是事件发布器；该发布器还会在spring一些重要节点结束之后就发布事件，如开始启动、环境信息准备完毕等；
    - 封装启动args：参数封装为ApplicationArguments【会把参数分为两类：分为选项参数，即--开头的  非选项参数】，第12步runner接口的run()要用到  //默认会[调用new DefausltApplicationArguments()]把main的args封装为ApplicationRunner；
    - 准备Environment：
      - 默认两个来源  propertySources：系统属性[VM option，例-Denv=FAT]、系统环境[操作系统的环境变量]； 
      - approperties[后面添加]、命令行参数[prgram arguments，例如--server.port=7070]  [这里第三步添加]等人工的属性，可以手工添加新的来源；
    - 4 添加ConfigurationPropertySources处理：
      - 为了使得getProperty能自动识别不同的分隔符    -、 _、 驼峰等，需要添加一个特殊的ConfigurationPropertySource；
    - 5.发布application environment已准备事件后，environmentPostProcessorApplicationListener进行env后处理，补充propertySource[通过后处理器的方式，==application.propertiies对应的源、产生随机数的源等==]
    - 6.properties配置文件中spring.main开头的键值绑定到程序的SpringApplicatin.java对象
    - 7.打印banner //可以自己指定banner
    - 8-11:创建容器   准备容器（包括执行 容器的初始化器等）  加载bean定义 （主要包括三种： 配置类定义  bean、xml、包扫描）  refresh容器(beanFactory后处理器  bean后处理器 初始化单例等)
    - 执行runner（ 内容可以自行确定，如可以用于预加载数据等）

- spring结合tomcat的底层是怎么实现的？

  - spring启动时电泳AbstractApplicationContext的onRefresh()方法时，会启动tomcat(在finishRefresh子方法中)；
  - 启动tomcat的过程包括设置虚拟路径、磁盘路径、初始化器、连接器等；然后从springContext获取DispatcherServlet并添加到tomcat的ServletContext中，当请求匹配到dispatcherServlet的路径时，就会走spring后续流程；
  - 补充说明
    - tomcat能直接识别的只有三大组件，经过web.xml配置的 servlet、filter、listener[3.0之后可以不用配置，编程动态添加三大组件]，controller  service只能被三大组件调用；
    -  几个术语的含义实例：context为tomcat中的概念，含义通常为一个应用；applicationContext是spring中的概念，含义通常是spring容器，内含所有的bean等信息；servletContext则是tomcat中的组件，含义是应用中包含的servlet等信息；

- spring自动配置的底层原理是说明？如何自己添加第三方配置类？如何在自己添加第三方配置类的基础上引入条件装配？

  - @Import + 自定义类实现 ImportSelector 接口+SpringFactoriesLoader读取配置文件信息；   
    - 使用@Import注解：spring不仅会自动扫描当前项目的spring.fatories文件、而且会找所有jar包目录的spring.factories的配置，==故要添加新的配置只需要三方包的spring.factories中配置即可，注意@EnableAutoConfiguration获取的配置的key是org.springframework.boot.autoconfigure.EnableAutoConfiguration；==
    -  ImportSelector 接口：【方法返回值就是配置类的类名形成的数组】
  - springBoot：和spring略有不同，@Import+ ==DeferredImportSelector接口==+SpringFactoriesLoader读取配置文件信息+==@ConditioanalOnMissingBean== 
    - 同一个bean不允许重复注册；使用DeferredImportSelector【推迟导入三方配置】故会先解析本项目的配置类；第三方bean添加@ConditioanalOnMissingBean 保证本项目没有时自动配置类第三方bean才生效；
    - 上述三种策略就保证了同一个bean在三方和本项目都有的情况下，本项目生效；
  - 补充说明1：配置类的本质：@Configuration注解修饰的Bean，但这些bean有一定通用性，不同项目都可以引入 ;
  - 自己添加第三方配置类的基础上引入条件装配：上述自动配置 + @Conditianal + 自定义类实现Condition接口进行条件判断（需要重写matches方法）
    - matches方法的入参可以提供一些必要的信息，如通过context获取beanFactory信息；metadata获取类的注解信息；
  - spring自动配置相关的核心类：AopAutoConfiguration、DataSourceAutoConfiguration、MybatisAutoConfiguration、DataSourceTransactionManagerAutoConfiguration、ProxyTransactionManagementConfiguration？？？？？？？
    - 以AopAutoConfiguration源码为例，则使用了大量的@ConditionalOnProperty、@ConditionalOnClass、@EnableAspectJAutoProxy等注解；  默认生效的是cglib代理
    - 补充说明：
      - Enable打头的注解，本质上都是使用@Import注解进行配置导入，功能是编程的方式把bean的beanDefinition加入到容器
      - @EnableAspectJAutoProxy：加入自动代理创建器，默认最终添加的是AnnotationAwareAspectJAutoProxyCreator.class； 

- ----选//spring中如何自定义和工厂类？

  - 首先编写Bean1的类定义，然后在Bean1FactoryBean的类定义上添加注解 @Bean("bean1")
      - 工厂Bean要实现三个方法，getObjectType 返回产品类型[getBean  依据类型获取时用到]； isSingleton  产品是单例还是多例； getObject 提供产品对象； 
      - 产品如果单例不会入beanFactory的单例池singleObjects，会放在factoryBeanObjectCache；
      - factory bean是spring创建的，但是产品时 factory bean调用bean1的构造创建的【不是spring创建的】，所以依赖注入 aware回调 初始化都不生效，但是bean初始化后的后处理器器会生效【代理就是初始化后增强】；

  - spring是如何实现包扫描的？spring是如何提升包扫描的效率的？

      - 编译阶段就实现扫描，减少扫描时间。
      - 编译阶段，去找包含@Indexed注解的类[@Coponent的父注解]，然后添加到spring.components  //需要添加依赖spring-context-indexer
      - spring5.0之后，scan方法在找不到spring.components文件的情况下（找到了就不扫描Jar包和类），才会真正去做包扫描 //target--classes--META-INF，spring.components文件 //spring组件扫描效率很低；

- #### spring AOP零碎知识：

  - 题外话：@SpringBootApplication注解
    - @EnableAutoConfiguration导入自动配置，也是个组合注解
      - @AutoConfigurationPackage：[register的第二个入参]，用来确定扫描范围，是前面记录的引导类的包名；
    - @Component：组件扫描，@Component @Service @Controller  ;
    - @SpringBootConfiguration表明这是个配置类；
  - @EnableConfigurationProperties：注解中属性为 DataSourceProperties.class表示会new一个该对象，并会绑定键值信息——以spring.datasource打头键值绑定到上面创建的对象； 
  - @ConditianalOnSingleCandidate 单一候选者； @AutoConfigureAfter表明了bean注入的先后顺序
    - SqlSessionTemplate：实现了SqlSession，可以生成一个线程绑定的bean，即一个线程共用一个SqlSession;
  - AnnotationUtils.findAnnotation注解会递归查找某个注解，即包含一个该注解的子注解也算【如@RestController同时包含了@Controller和@ResponseBody】；getContainingClass获取包含该 返回结果-对应方法-所在类
  - 一个方法匹配多个切面时如何设置切面的生效顺序？高级切面和低级切面的顺序设置方法如下：
    - 高级切面 @Order； //加在方法上无效，故单个类内的不同方法的顺序无法控制
    - 低级切面：advisor.setOrder   ; //@Bean方法上的 @Order 无法生效；
    - 如何手工讲高级切面转换为低级切面？
        - spring高级切面一共5中，@Before  @After   @Around  @AfterThrowing @AfterReturning ;  不同高级切面 对应的 低级切面的通知 不同，依次为AspectJMehtodBeforeAdvice   AspectJAfterAdvice AspectJAroundAdvice  AspectJAfterReturningAdvice  AspectJAfterThrowingAdvice

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
    - AnnotationAwareAspectJAutoProxyCreator
      - @Aspect
  - beanFactory后处理器及对应注解
    - ConfigurationClassPostProcessor   @Bean、@Component 、@Import 、@ImportResource
    - MapperScannerConfigurer[springboot自动配置  ]   扫描mybatis的mapper接口；  //SSM架构用的@MapperScanner底层也是MapperScannerConfigurer.class ； @MapperScan
  
- 小tip：
  
  - 配置文件属性读取
  
      \- 配置属性可以在配置文件【application.properties】中设置，并通过注解 @PropertySource + @Value/@EnableConfigurationProperties[结合WebMvcProperties.class/WebProperties.class）读取；例如：@EnableConfigurationProperties({ServerPropertires.class})会打包读取application.properties中server打头的key并封装为ServerProperties对象存入容器；
  
  - 注解相关的好用的方法
    - method.getAnnotation(Before.class).value()
    - method.isAnnotationPresent
  
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


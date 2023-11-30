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


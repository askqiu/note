# 数据持久层
作用是在java对象与数据库之间建立映射，也就是说它的作用是将某一个Java类对应到数据库中的一张表。在我们的项目中，就将创建一个实体类User映射到数据库的user表，表中的每个字段对应于实体类的每个属性。而之前配置的JPA的作用就是帮助我们完成类到数据表的映射。
  repository: 存放一些数据访问类（也就是一些能操纵数据库的类）的包，比如存放能对user表进行增删改查的类
  domain：存放实体类的包，比如User类，其作为对应数据库user表的一个实体类

# 业务逻辑层
作用是处理业务逻辑。比如在本项目中，我们就在业务逻辑层实现登录注册的逻辑，像是判断是否有用户名重复，密码是否正确等逻辑
  service: 存放业务逻辑接口的包
  serviceImpl: 存放业务逻辑实现类的包，其中的类实现service中的接口

# 控制层
作用是接收视图层的请求并调用业务逻辑层的方法。比如视图层请求登录并发来了用户的账号和密码，那么控制层就调用业务逻辑层的登录方法，并将账号密码作为参数传入，在将结果返回给视图层。
  controller: 存放控制器的包。比如UserController

# 视图层
作用
是展现数据，

userservice
1、在service包中创建UserService接口
2、添加登录注册需要用到的业务逻辑方法

实现UserServiceImpl
1、在serviceImpl包中创建UserServiceImpl类
2、添加需要实现的方法
3、实现登录业务逻辑

因为要用到UserDao中的方法，所以先通过@Resource注解帮助我们实例化UserDao对象
4、实现注册业务逻辑
5、添加@Service注解

实现UserController
1、在controller包中创建UserController类
2、添加@RestController与@RequestMapping(“/user”)注解，注入UserService
注解
3、实现登录的控制
4、实现注册的控制
@RequestMapping中的"/user"是这个控制器类的基路由
@PostMapping(“/login”)表示处理post请求，路由为/user/login
@PostMapping(“/register”)表示处理post请求，路由为/user/register
### 代码运行原理

CPU 只听得懂**机器码**（由 0 和 1 组成的指令）

CPU 看不懂源代码（比如 `print("你好")`），必须有人"翻译"。负责翻译的工具叫**编译器**。

**编译**是把一种语言（通常是高级语言）翻译成另一种语言（机器码、字节码、汇编等）。





C、C++：源码 → 编译器整体编译成机器码 → 得到可直接运行的程序文件（如 Windows 上的 `.exe`）→ CPU 直接执行

优点就是只用翻译一次，后面运行快，缺点就是不跨平台。在 Windows 编译出的 `.exe` 不能直接在 Linux/Mac 上跑，换平台要重新编译。





现代 Python ：源码（`.py`）→ 编译成字节码（.pyc`）→ Python 虚拟机逐条解释执行字节码

Java：源码（`.java`）→ 编译成字节码（`.class`）→ JVM（Java 虚拟机） 解释执行字节码 →（热点代码）JIT 编译成机器码 → CPU 直接执行

JIT（Just-In-Time Compiler，即时编译器）





虚拟机执行字节码原理：

因为虚拟机（解释器）本身是一个已经编译成机器码的程序

- CPython 的解释器`python.exe`是用 C 写的
- JVM 的解释器主要是 C++ 写的（`java.exe`，JVM 在 `jvm.dll` 里）

所以操作系统可以把 `python.exe` 或 `java.exe` 当成普通的机器码程序加载，让CPU 去执行。字节码只是.exe里读入的变量数据。

```
CPU 执行 python.exe 的机器码：
  循环：
    读取下一条字节码         ← 你的程序，只是数据
    如果是"加法"   → 跳到解释器里"做加法"的那段机器码
    如果是"调用函数" → 跳到解释器里"调用函数"的那段机器码
```





JavaScript：源码（`.js`）→ V8 编译成字节码（在内存中）→ 解释执行 →（热点代码）JIT 编译成机器码 → CPU 直接执行

Typescript：源码（`.ts`）→ TypeScript 编译器（`tsc`）做类型检查并去掉类型，输出 JavaScript（`.js`）→ 之后与上面的 JavaScript 流程完全相同







```
Jakarta EE（原 Java EE）= Java SE + 服务器端规范(Servlet/JPA/JMS…)
└─ Java SE（规范）── JDK 是它的实现
  └─ JDK = 开发工具(javac、调试器…) + JRE
    └─ JRE = JVM + 标准类库
      └─ JVM：运行字节码，屏蔽 Windows/Linux/macOS 差异
```

| 术语       | 一句话                                                       |
| ---------- | ------------------------------------------------------------ |
| JVM        | 运行 `.class` 字节码的虚拟机                                 |
| JRE        | JVM + 标准类库，只能运行程序                                 |
| JDK        | JRE + 编译器/调试器等，用于开发（学习 Java 装它即可）        |
| Java SE    | 标准版平台规范（语言 + 核心库），桌面、命令行、通用程序都用它 |
| Jakarta EE | 在 SE 之上增加服务器端规范，用于网站、后台、企业系统         |

















### 接手项目速通条目清单

A. 语言与运行环境

1. **Java 语法，只学项目里会读到的部分**：类与对象、接口与实现类、`public/private/static/final`、包与 `import`、泛型（`List<Region>`、`IService<T>`）、集合（`List/Map/Set`）、异常（`try/catch/throws`）、Lambda 与 Stream、`Optional`。getter/setter 是仪式代码，只要知道有它们（`BaseEntity`）。
2. **JDK、JVM、编译与运行流程**：`.java` → `javac` → `.class` → JVM。需要弄清 JDK 与 JRE 的关系、`java -jar` 做了什么、为什么项目要求 Java 17 而你装的是 21（高版本向下兼容低版本）。
3. **Maven**：`pom.xml` 相当于 `package.json` 加构建脚本。需要学：坐标（`groupId/artifactId/version`）、依赖与传递依赖、`dependencyManagement` 与 BOM、`<modules>` 多模块、生命周期（`clean/compile/package`）、本地仓库 `.m2`。这就是"为什么要装 Maven、依赖从哪来"的答案（根目录 `pom.xml`）。
4. **VS Code 里的 Java 扩展做了什么**：Extension Pack for Java（语言服务、调试、Maven 集成）和 Spring Boot Extension Pack（启动面板、`application.yml` 补全）。需要弄清它们和 JDK、Maven 的分工：扩展只是工具，不是运行依赖。

B. Web 与 Spring 的基础

1. **Servlet / Jakarta EE 与 Tomcat**：HTTP 请求最终怎么进入 Java 代码。需要学：Servlet 规范、Filter、嵌入式 Tomcat、`javax.*` 与 `jakarta.*` 的区别（Spring Boot 3 用 `jakarta`，代码里能看到 `jakarta.servlet`）。
2. **Spring 核心：IoC、依赖注入、Bean**：`@Component/@Service/@Configuration/@Bean`、构造器注入与 `@Autowired`、Lombok 的 `@RequiredArgsConstructor`（`RegionServiceImpl`）。
3. **注解与 AOP**：注解是元数据加框架反射处理，不是装饰器。需要学：`@Log`、`@PreAuthorize`、`@RepeatSubmit` 这类自定义注解怎么触发切面，以及 AOP 的"切面、切点、通知"。
4. **Spring Boot**：自动配置、starter、`@SpringBootApplication`、`application.yml`、多环境 profile（`devmy/prodmy`）、`@ConfigurationProperties`（`ResvisApplication`、`application.yml`）。
5. **Spring MVC**：`@RestController`、`@RequestMapping/@GetMapping`、`@PathVariable/@RequestBody`、参数绑定、JSON 序列化（Jackson/Fastjson）、统一返回 `AjaxResult`、全局异常处理（`RegionController`）。

C. 数据与缓存

1. **JDBC、连接池与事务**：Java 怎么连数据库、为什么需要 Druid 连接池、`@Transactional`（`DruidConfig`、`application-devmy.yml`）。
2. **MyBatis 与 MyBatis-Plus**：Mapper 接口加 XML、`#{}` 与 `${}` 的区别、动态 SQL（`<if>`、`<where>`）、`BaseMapper`、`IService`、`QueryWrapper`、分页（PageHelper）、`@TableName/@TableId`（`RegionMapper`、`SysUserMapper.xml`）。
3. **Redis**：基本数据类型、`RedisTemplate` 与序列化、项目里用它存什么（登录 token、验证码、限流、字典缓存）、为什么 Redis 没装项目就起不来（`RedisConfig`、`RedisCache`）。
4. **本地 MySQL 与 Redis 脱离内网**：导入 `sql/resvis-mysql.sql`、新建 `application-devlocal.yml` 覆盖数据库和 Redis 地址、Windows 上用 Memurai 代替 Redis。

**D. 安全与若依封装**

1. **Spring Security 与 JWT**：过滤器链、登录认证流程、JWT 的生成与校验、`@PreAuthorize("@ss.hasPermi(...)")` 权限表达式、`@Anonymous` 放行。
2. **若依的分层与基类**：模块职责（`admin/framework/system/common/biz/generator/quartz`）、`BaseController`、`BaseEntity`、数据权限、多数据源切换。
3. **若依代码生成器**：从表结构生成 Controller、Service、Mapper、XML 和前端页面（`resvis-generator`、`service.java.vm`）。这是之后新增业务最省力的方式。

E. 用到时再学的内容

Quartz 定时任务、Magic API 低代码、多租户、OpenAI 对话、Druid 监控页、XSS 过滤、Excel 导入导出。

学习顺序建议：第 1 到 4 条一起学，第 5 到 9 条串成"一次请求怎么走"，第 10 到 12 条串成"数据怎么落库"，最后学 13 到 16 条。




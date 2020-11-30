# 一、日志

## 1.1 日志的概念

* 日志文件是用于记录系统操作事件的文件集合，可分为事件日志和消息日志。具有处理历史数据、诊断 问题的追踪以及理解系统的活动等重要作用。

### 1.1.2 调试日志

软件开发中，我们经常需要去调试程序，做一些信息，状态的输出便于我们查询程序的运行状况。为了 让我们能够更加灵活和方便的控制这些调试的信息，所有我们需要专业的日志技术。java中寻找bug会 需要重现。调试也就是debug 可以在程序运行中暂停程序运行，可以查看程序在运行中的情况。日志主 要是为了更方便的去重现问题。



### 1.1.3 系统日志

系统日志是记录系统中硬件、软件和系统问题的信息，同时还可以监视系统中发生的事件。用户可以通 过它来检查错误发生的原因，或者寻找受到攻击时攻击者留下的痕迹。系统日志包括系统日志、应用程 序日志和安全日志。



* 系统日志的价值
    * 系统日志策略可以在故障刚刚发生时就向你发送警告信息，系统日志帮助你在最短的时间内发现问题。 
    * 系统日志是一种非常关键的组件，因为系统日志可以让你充分了解自己的环境。这种系统日志信息对于 决定故障的根本原因或者缩小系统攻击范围来说是非常关键的，因为系统日志可以让你了解故障或者袭 击发生之前的所有事件。为虚拟化环境制定一套良好的系统日志策略也是至关重要的，因为系统日志需 要和许多不同的外部组件进行关联。良好的系统日志可以防止你从错误的角度分析问题，避免浪费宝贵 的排错时间。另外一种原因是借助于系统日志，管理员很有可能会发现一些之前从未意识到的问题，在 几乎所有刚刚部署系统日志的环境当中。





# 二、JUL

JUL全称Java util Logging是java原生的日志框架，使用时不需要另外引用第三方类库，相对其他日志框 架使用方便，学习简单，能够在小型应用中灵活使用。



## 2.1 架构

![](E:\Study\MyNotes\日志\JUL架构.png)

* Loggers：被称为记录器，应用程序通过获取Logger对象，调用其API来来发布日志信息。Logger 通常时应用程序访问日志系统的入口程序。 
* Appenders：也被称为Handlers，每个Logger都会关联一组Handlers，Logger会将日志交给关联 Handlers处理，由Handlers负责将日志做记录。Handlers在此是一个抽象，其具体的实现决定了 日志记录的位置可以是控制台、文件、网络上的其他日志服务或操作系统日志等。
*  Layouts：也被称为Formatters，它负责对日志事件中的数据进行转换和格式化。Layouts决定了 数据在一条日志记录中的最终形式。 
* Level：每条日志消息都有一个关联的日志级别。该级别粗略指导了日志消息的重要性和紧迫，我 可以将Level和Loggers，Appenders做关联以便于我们过滤消息。 
* Filters：过滤器，根据需要定制哪些信息会被记录，哪些信息会被放过。

用户使用Logger来进行日志记录，Logger持有若干个Handler，日志的输出操作是由Handler完成的。 在Handler在输出日志前，会经过Filter的过滤，判断哪些日志级别过滤放行哪些拦截，Handler会将日 志内容输出到指定位置（日志文件、控制台等）。Handler在输出日志时会使用Layout，将输出内容进 行排版。



## 2.2 日志级别

* java.util.logging.Level中定义了日志的级别：
    * SEVERE（最高值） 
    * WARNING 
    * INFO （默认级别） 
    * CONFIG 
    * FINE 
    * FINER 
    * FINEST（最低值）
* 还有两个特殊的级别：
    * OFF，可用来关闭日志记录。 
    * ALL，启用所有消息的日志记录。



### 自定义日志级别配置

``` java
@Test
public void test1() throws Exception {
    // 1.创建日志记录器对象
    Logger logger = Logger.getLogger("com.qiancijun.JULTest");
    // 一、自定义日志级别
    // a.关闭系统默认配置
    logger.setUseParentHandlers(false);
    // b.创建handler对象
    ConsoleHandler consoleHandler = new ConsoleHandler();
    // c.创建formatter对象
    SimpleFormatter simpleFormatter = new SimpleFormatter();
    // d.进行关联
    consoleHandler.setFormatter(simpleFormatter);
    logger.addHandler(consoleHandler);
    // e.设置日志级别
    logger.setLevel(Level.ALL);
    consoleHandler.setLevel(Level.ALL);
    // 二、输出到日志文件
    FileHandler fileHandler = new FileHandler("d:/logs/jul.log");
    fileHandler.setFormatter(simpleFormatter);
    // 2.日志记录输出
    logger.severe("severe");
    logger.warning("warning");
    logger.info("info");
    logger.config("config");
    logger.fine("fine");
    logger.finer("finer");
    logger.finest("finest");
    
}
```





## 2.3  Logger之间的父子关系

JUL中Logger之间存在父子关系，这种父子关系通过树状结构存储，JUL在初始化时会创建一个顶层 RootLogger作为所有Logger父Logger，存储上作为树状结构的根节点。并父子关系通过路径来关联。

``` java
@Test
public void testLogParent() throws Exception {
    // 日志记录器对象父子关系
    Logger logger1 = Logger.getLogger("com.qiancijun.JULTest");
    Logger logger2 = Logger.getLogger("com.qiancijun");
    System.out.println(logger1.getParent() == logger2);
    // 所有日志记录器对象的顶级父元素 class为java.util.logging.LogManager$RootLogger
    name为""
    System.out.println("logger2 parent:" + logger2.getParent() + "，name：" +
    logger2.getParent().getName());
    // 一、自定义日志级别
    // a.关闭系统默认配置
    logger2.setUseParentHandlers(false);
    // b.创建handler对象
    ConsoleHandler consoleHandler = new ConsoleHandler();
    // c.创建formatter对象
    SimpleFormatter simpleFormatter = new SimpleFormatter();
    // d.进行关联
    consoleHandler.setFormatter(simpleFormatter);
    logger2.addHandler(consoleHandler);
    // e.设置日志级别
    logger2.setLevel(Level.ALL);
    consoleHandler.setLevel(Level.ALL);
    // 测试日志记录器对象父子关系
    logger1.severe("severe");
    logger1.warning("warning");
    logger1.info("info");
    logger1.config("config");
    logger1.fine("fine");
    logger1.finer("finer");
    logger1.finest("finest");
}
```



## 2.4  日志的配置文件

默认配置文件路径$JAVAHOME\jre\lib\logging.properties

``` java
@Test
public void testProperties() throws Exception {
    // 读取自定义配置文件
    InputStream in =
    JULTest.class.getClassLoader().getResourceAsStream("logging.properties");
    // 获取日志管理器对象
    LogManager logManager = LogManager.getLogManager();
    // 通过日志管理器加载配置文件
    logManager.readConfiguration(in);
    Logger logger = Logger.getLogger("com.itheima.log.JULTest");
    logger.severe("severe");
    logger.warning("warning");
    logger.info("info");
    logger.config("config");
    logger.fine("fine");
    logger.finer("finer");
    logger.finest("finest");
}
```

配置文件：

``` properties
## RootLogger使用的处理器（获取时设置）
handlers= java.util.logging.ConsoleHandler
# RootLogger日志等级
.level= INFO
## 控制台处理器
# 输出日志级别
java.util.logging.ConsoleHandler.level = INFO
# 输出日志格式
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter
## 文件处理器
# 输出日志级别
java.util.logging.FileHandler.level=INFO
# 输出日志格式
java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter
# 输出日志文件路径
java.util.logging.FileHandler.pattern = /java%u.log
# 输出日志文件限制大小（50000字节）
java.util.logging.FileHandler.limit = 50000
# 输出日志文件限制个数
java.util.logging.FileHandler.count = 10
# 输出日志文件 是否是追加
java.util.logging.FileHandler.append=true
```





# 三、Log4j

Log4j是Apache下的一款开源的日志框架，通过在项目中使用 Log4J，我们可以控制日志信息输出到控 制台、文件、甚至是数据库中。我们可以控制每一条日志的输出格式，通过定义日志的输出级别，可以 更灵活的控制日志的输出过程。方便项目的调试。



## 3.1 入门使用

``` java
public class Log4jTest {
    @Test
    public void testQuick() throws Exception {
        // 初始化系统配置，不需要配置文件
        BasicConfigurator.configure();
        // 创建日志记录器对象
        Logger logger = Logger.getLogger(Log4jTest.class);
        // 日志记录输出
        logger.info("hello log4j");
        // 日志级别
        logger.fatal("fatal"); // 严重错误，一般会造成系统崩溃和终止运行
        logger.error("error"); // 错误信息，但不会影响系统运行
        logger.warn("warn"); // 警告信息，可能会发生问题
        logger.info("info"); // 程序运行信息，数据库的连接、网络、IO操作等
        logger.debug("debug"); // 调试信息，一般在开发阶段使用，记录程序的变量、参
        数等
        logger.trace("trace"); // 追踪信息，记录程序的所有流程信息
    }
}
```



## 3.2 日志的级别

* fatal 指出每个严重的错误事件将会导致应用程序的退出。
*  error 指出虽然发生错误事件，但仍然不影响系统的继续运行。 
* warn 表明会出现潜在的错误情形。
*  info 一般和在粗粒度级别上，强调应用程序的运行全程。 
* debug 一般用于细粒度级别上，对调试应用程序非常有帮助。 
* trace 是程序追踪，可以用于输出程序运行中的变量，显示执行的流程。



## 3.3  Log4j组件

Log4J 主要由 Loggers (日志记录器)、Appenders（输出端）和 Layout（日志格式化器）组成。其中 Loggers 控制日志的输出级别与日志是否输出；Appenders 指定日志的输出方式（输出到控制台、文件 等）；Layout 控制日志信息的输出格式。



### 3.3.1 Loggers

* 日志记录器，负责收集处理日志记录，实例的命名就是类“XX”的full quailied name（类的全限定名）， Logger的名字大小写敏感，其命名有继承机制：例如：name为org.apache.commons的logger会继承 name为org.apache的logger。
*  Log4J中有一个特殊的logger叫做“root”，他是所有logger的根，也就意味着其他所有的logger都会直接 或者间接地继承自root。root logger可以用Logger.getRootLogger()方法获取。 但是，自log4j 1.2版以来， Logger 类已经取代了 Category 类。对于熟悉早期版本的log4j的人来说， Logger 类可以被视为 Category 类的别名。



### 3.3.2  Appenders

Appender 用来指定日志输出到哪个地方，可以同时指定日志的输出目的地。Log4j 常用的输出目的地 有以下几种：



| 输出端类型               | 作用                                                         |
| ------------------------ | ------------------------------------------------------------ |
| ConsoleAppender          | 将日志输出到控制台                                           |
| FileAppender             | 将日志输出到文件中                                           |
| DailyRollingFileAppender | 将日志输出到一个日志文件，并且每天输出到一个新的文件         |
| RollingFileAppender      | 将日志信息输出到一个日志文件，并且指定文件的尺寸，当文件大 小达到指定尺寸时，会自动把文件改名，同时产生一个新的文件 |
| JDBCAppender             | 把日志信息保存到数据库中                                     |



### 3.3.3 Layouts

布局器 Layouts用于控制日志输出内容的格式，让我们可以使用各种需要的格式输出日志。Log4j常用 的Layouts:



| 格式化器类型  | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| HTMLLayout    | 格式化日志输出为HTML表格形式                                 |
| SimpleLayout  | 简单的日志输出格式化，打印的日志格式为（info - message）     |
| PatternLayout | 最强大的格式化期，可以根据自定义格式输出日志，如果没有指定转换格式， 就是用默认的转换格式 |



* Layout的格式：在 log4j.properties 配置文件中，我们定义了日志输出级别与输出端，在输出端中分别配置日志的输出 格式。
    * %m 输出代码中指定的日志信息 
    * %p 输出优先级，及 DEBUG、INFO 等 
    * %n 换行符（Windows平台的换行符为 "\n"，Unix 平台为 "\n"）
    * %r 输出自应用启动到输出该 log 信息耗费的毫秒数 
    * %c 输出打印语句所属的类的全名 
    * %t 输出产生该日志的线程全名 
    * %d 输出服务器当前时间，默认为 ISO8601，也可以指定格式，如：%d{yyyy年MM月dd日 HH:mm:ss} 
    * %l 输出日志时间发生的位置，包括类名、线程、及在代码中的行数。如： Test.main(Test.java:10) 
    * %F 输出日志消息产生时所在的文件名称
    * %L 输出代码中的行号 
    * %% 输出一个 "%" 字符
*  可以在 % 与字符之间加上修饰符来控制最小宽度、最大宽度和文本的对其方式。如：
    * %5c 输出category名称，最小宽度是5，category<5，默认的情况下右对齐 
    * %-5c 输出category名称，最小宽度是5，category<5，"-"号指定左对齐,会有空格 
    * %.5c 输出category名称，最大宽度是5，category>5，就会将左边多出的字符截掉，<5不 会有空格 
    * %20.30c category名称<20补空格，并且右对齐，>30字符，就从左边交远销出的字符截掉





## 3.4 Appender的输出

``` properties
log4j.rootLogger=trace,console,rollingFile
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.conversionPattern=[%-10p]%r   %l %d{yyyy-MM-dd HH:mm:ss} %m%n


log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.conversionPattern=[%-10p]%r   %l %d{yyyy-MM-dd HH:mm:ss} %m%n
log4j.appender.file.file=src/main/resources/logs/log4j.log
log4j.appender.file.encoding=UTF-8


log4j.appender.rollingFile=org.apache.log4j.RollingFileAppender
log4j.appender.rollingFile.layout=org.apache.log4j.PatternLayout
log4j.appender.rollingFile.layout.conversionPattern=[%-10p]%r   %l %d{yyyy-MM-dd HH:mm:ss} %m%n
log4j.appender.rollingFile.encoding=UTF-8
log4j.appender.rollingFile.file=src/main/resources/logs/log4j2.log
log4j.appender.rollingFile.maxFileSize=200KB
log4j.appender.rollingFile.maxBackupIndex=10
```


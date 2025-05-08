### 一、SpringBoot核心功能
SpringBoot是一个简化Spring初始化搭建以及开发过程的框架。SpringBoot主要的核心功能：

+ 自动装配
+ Starter
+ Actuator

### 二、SpringBoot自动装配机制
SpringBoot自动装配主要是基于注解编程和约定优于配置思想来设计的。我们只需要在SpringBoot启动类上添加添加@ SpringBootApplication注解就能开启自动装配。

1. **自动装配的实现**
    1. 底层是使用@ EnableAutoConfiguration注解来实现的，归纳以下步骤：
        1. 首先启动依赖组件的时候，组件必须包含@ Configuration的配置类，这个配置类中声明@ Bean注解
        2. 如果是第三方包，SpringBoot采用SPI机制，则只需要在/META_INF/目录下增加spring.factories配置文件，SpringBoot会根据约定规则使用SpringFactoriesLoader来加载配置文件内容
        3. 获取到配置文件后，会装配配置文件里面的Bean
2. <font style="color:#DF2A3F;">@ EnableAutoConfiguration注解的原理，利用ImportSelector接口来完成动态加载</font>：
+ @EnableAutoConfiguration -----> @Import(AutoConfigurationImportSelector.class)----->   AutoConfigurationImportSelector 类
+ AutoConfigurationImportSelector这个类实现了ImportSelector接口，主要实现2个逻辑：
    - 扫描并加载配置文件
    - 返回需要装配的Bean数组

### 三、SpringBoot约定优于配置
1. 约定优于配置是一种软件设计规范，核心思想就是减少软件开发人员对配置的维护，更加聚焦在业务逻辑上
2. SpringBoot就是这一理念的产物
3. 比如传统的Spring开发需要做很多配置，现在SpringBoot不再需要去做这些配置，会自动帮我们完成，这就是约定由于配置的提现
4. 其他提现：
    1. 自动装配中，约定扫描路径下的spring.factories
    2. Starter启动依赖，它能帮我们管理所有Jar包
    3. 等
+ 官方组件命名格式统一：spring-boot-starter-xxxx
+ 官方组件不需要spring.factories文件定义，因为统一维护在spring-boot-configuration的jar包里面
+ 第三方命名格式：xxxx-spring-boot-starter
+ 

### 四、SPI机制（Service Provider Interface）
SPI是一种服务发现机制，它提供了一种服务接口与其实现解耦的方式。通过SPI机制，开发者可以动态地发现和加载服务实现类，而无需在代码中硬编码这些实现类的名字或位置。这使得应用程序能够灵活地支持不同的服务提供商。

1. 工作原理
    1. 定义服务接口：首先需要定义一个服务接口，所有服务提供者都必须实现这个接口
    2. 创建服务提供者：不同的服务提供者分别实现该接口，并将其打包为 JAR 文件
    3. 配置服务提供者：
        1. 每个服务提供者都需要包含一个特定的配置文件，位于 META-INF/services/ 目录下，文件名是服务接口的全限定名。
        2. 文件内容则是该服务接口的所有具体实现类的全限定名，每行一个。
    4. 使用 ServiceLoader 加载服务
        1. 当需要使用某个服务时，可以通过 ServiceLoader.load(YourServiceInterface.class) 方法来加载所有的服务提供者实例。


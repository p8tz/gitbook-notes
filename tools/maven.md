## 构建

概念：把Java源文件，配置文件，HTML，资源文件等变为一个可以运行的项目的过程

构建过程中各个环节（大概）

- 清理：清除旧的`.class`，资源等文件
- 编译：`.java --> .class`
- 测试：自动调用`JUnit`程序
- 报告：测试程序的结果
- 打包：打`jar`包
- 安装：`maven`特定的概念，将打包得到的文件复制到仓库中指定的位置
- 部署：部署到`Servlet`容器，使可以运行

## 目录结构

```
project
|---pom.xml
|---src
|---|---main
|---|---|---java
|---|---|---resources
|---|---test
|---|---|---java
|---|---|---resources
```

## 生命周期

分为3套独立的生命周期

- `Clean LifeCycle`：构建之前执行清理工作
- `Default LifeCycle`：核心部分，编译，测试，打包，安装，部署等
- `Site LifeCycle`：生成项目报告，站点，发布站点

生命周期仅仅定义了任务，具体执行需要插件来完成

### Default LifeCycle

截取了部分

- validate  验证工程是否正确，所需的信息是否完整
- initialize  初始化构建平台，例如：设置properties或创建目录
- process-resources 复制并处理资源文件，至目标目录，准备打包
- compile  编译源代码
- test-compile  编译测试源代码
- test  执行单元测试
- package  将工程文件打包为指定的格式，例如JAR，WAR等
- integration-test  集成测试
- verify  检查package是否有效、符合标准
- install  将包安装至本地仓库，以让其它项目依赖。
- deploy  将包发布到远程仓库，以让其它开发人员与项目共享

### 命令

必须在`pom.xml`所在的目录执行，每一个命令会从该生命周期开始执行

```bash
mvn clean  			: 清理 【清除target目录】

mvn compile			: 编译主程序
mvn test-compile	: 编译测试程序
mvn test			: 执行测试程序 【单元测试】
mvn package			: 生成jar包
mvn install			: 安装 【把jar包加入到本地仓库】
mvn deploy			: 发布 【把jar包加入到远程仓库】 可以配置自动部署到服务器

mvn site			: 生成站点 【生成关于项目信息的html页面】
```

## 依赖

### 范围

通过`<scope></scope>`标签设置

|          | 编译（主程序类路径） | 测试（测试程序类路径） | 运行（打包）   |
| -------- | -------------------- | ---------------------- | -------------- |
| compile  | $$\checkmark$$       | $$\checkmark$$         | $$\checkmark$$ |
| test     | $$\times$$           | $$\checkmark$$         | $$\times$$     |
| provided | $$\checkmark$$       | $$\checkmark$$         | $$\times$$     |
| runtime  | $$\times$$           | $$\checkmark$$         | $$\checkmark$$ |

### 传递性

A依赖B，B依赖C，则A会自动依赖C。对于多模块项目，只需要一个父工程依赖所有需要的依赖，然后子类依赖父工程即可

**非`compile`依赖不能传递**

### 排除

A依赖B和C，则D依赖A时会自动依赖B和C，但是D不想要依赖C，这时可以使用`<exclusions></exclusions>`标签排除C

### 原则

作用：为了解决`jar`包冲突问题

```c++
// 下面这种情况, 遵循就近原则, 即A依赖E依赖的D的版本
A-->B-->C-->D
A-->E-->D

// 下面这种情况, 遵循先声明先依赖原则, 即A依赖B依赖的C的版本
A-->B-->C
A-->D-->C
```

### 统一版本管理

```xml
<properties>
    <!-- 自定义标签 -->
    <x.y.z.version>5.0.0</xxx.xxx.version>
</properties>
<dependencies>
    <dependency>
        <groupId>x.y.z</groupId>
        <artifactId>a</artifactId>
        <version>${x.y.z.version}</version>
    </dependency>
    <dependency>
        <groupId>x.y.z</groupId>
        <artifactId>b</artifactId>
        <version>${x.y.z.version}</version>
    </dependency>
</dependencies>
```

### 继承与聚合

指定一个父工程，打包方式为`pom`，子工程声明父工程即可

```xml
<!-- 父工程 -->
<packaging>pom</packaging>
<!-- 聚合子工程 -->
<modules>
    <module>maven-child-a</module>
    <module>maven-child-b</module>
    <module>maven-child-c</module>
</modules>
<groupId>org.example</groupId>
<artifactId>maven-parent</artifactId>
<version>1.0-SNAPSHOT</version>
<!-- 使用该标签统一管理子工程依赖及依赖版本 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.7.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- 子工程 -->
<parent>
    <artifactId>maven-parent</artifactId>
    <groupId>org.example</groupId>
    <version>1.0-SNAPSHOT</version>
</parent>
<!-- gv可以省略 -->
<artifactId>maven-child-a</artifactId>
<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <!-- 不声明版本号, 则使用父工程的版本; 否则使用指定的版本 -->
        <!-- <version>5.7.0</version> -->
    </dependency>
</dependencies>
```


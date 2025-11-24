# Java 完整参考指南

> 范围：JDK安装、项目构建、应用运行、性能监控、部署管理  
> 版本：Java 11+, Maven 3.8+, Gradle 7.0+  
> 最后更新：2025-11-24

---

## 目录

1. [JDK环境设置](#jdk环境设置)
2. [Maven项目管理](#maven项目管理)
3. [Gradle项目管理](#gradle项目管理)
4. [应用运行](#应用运行)
5. [Spring Boot框架](#spring-boot框架)
6. [JVM监控工具](#jvm监控工具)
7. [性能优化](#性能优化)
8. [应用部署](#应用部署)
9. [常见问题](#常见问题)
10. [快速参考](#快速参考)

---

## JDK环境设置

### JDK安装

```bash
# 查看系统中已安装的Java版本
java -version
javac -version

# 更新包列表
sudo apt update

# 安装OpenJDK 11
sudo apt install -y openjdk-11-jdk openjdk-11-jre

# 安装OpenJDK 17（推荐用于新项目）
sudo apt install -y openjdk-17-jdk openjdk-17-jre

# 安装OpenJDK 21（最新LTS）
sudo apt install -y openjdk-21-jdk openjdk-21-jre

# 同时安装多个版本
sudo apt install -y openjdk-11-jdk openjdk-17-jdk openjdk-21-jdk

# 验证安装
java -version
javac -version
jps -version
```

### JDK版本管理

```bash
# 查看所有已安装的Java版本
update-alternatives --list java
update-alternatives --list javac
update-alternatives --list jar

# 设置Java命令的默认版本
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-11-openjdk-amd64/bin/java 1
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-17-openjdk-amd64/bin/java 2
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-21-openjdk-amd64/bin/java 3

# 设置javac命令的默认版本
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-11-openjdk-amd64/bin/javac 1
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-17-openjdk-amd64/bin/javac 2

# 设置jar命令的默认版本
sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/java-11-openjdk-amd64/bin/jar 1
sudo update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/java-17-openjdk-amd64/bin/jar 2

# 交互式选择默认版本
sudo update-alternatives --config java
sudo update-alternatives --config javac
sudo update-alternatives --config jar

# 删除某个版本的java链接
sudo update-alternatives --remove java /usr/lib/jvm/java-11-openjdk-amd64/bin/java

# 查看当前默认版本
alternatives --display java
```

### 设置JAVA_HOME环境变量

```bash
# 查找Java安装位置
which java
readlink -f $(which java)
ls -l /usr/lib/jvm/

# 设置JAVA_HOME临时生效（当前会话）
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

# 验证JAVA_HOME设置
echo $JAVA_HOME

# 永久设置JAVA_HOME（编辑~/.bashrc）
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc

# 使设置立即生效
source ~/.bashrc

# 验证PATH
echo $PATH
java -version
```

### JDK工具验证

```bash
# 查看JDK中包含的所有命令
ls $JAVA_HOME/bin/

# 查看JVM版本信息
java -version
java -fullversion

# 查看JVM系统属性
java -XshowSettings:all -version

# 查看JVM内存配置
java -XshowSettings:vm -version

# 列出所有系统属性
java -XshowSettings:properties -version
```

---

## Maven项目管理

### Maven安装和配置

```bash
# 安装Maven
sudo apt install -y maven

# 验证Maven安装
mvn --version

# 查看Maven主要信息
mvn --help

# 配置Maven本地仓库（~/.m2/settings.xml）
mkdir -p ~/.m2
cat > ~/.m2/settings.xml << 'EOF'
<settings>
  <mirrors>
    <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>Aliyun Maven Repository</name>
      <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
  </mirrors>
</settings>
EOF

# 查看Maven配置
cat ~/.m2/settings.xml

# 查看本地仓库大小
du -sh ~/.m2/repository/

# 清理本地仓库
rm -rf ~/.m2/repository/

# 查看Maven本地仓库位置
mvn help:describe -Dcmd=help:active-profiles
```

### Maven基础命令

```bash
# 验证Maven项目结构
mvn validate

# 生成项目骨架
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=myapp \
  -DarchetypeArtifactId=maven-archetype-quickstart \
  -DinteractiveMode=false

# 编译项目
mvn compile

# 查看编译过程
mvn clean compile -X

# 运行单元测试
mvn test

# 跳过测试进行编译
mvn compile -DskipTests
mvn compile -Dmaven.test.skip=true

# 打包项目（生成JAR或WAR）
mvn package

# 跳过测试打包
mvn clean package -DskipTests

# 查看打包结果
ls -lh target/*.jar

# 显示依赖树
mvn dependency:tree

# 分析依赖
mvn dependency:analyze

# 下载依赖源码
mvn dependency:sources

# 下载依赖javadoc
mvn dependency:resolve -Dclassifier=javadoc

# 检查过期的依赖
mvn versions:display-dependency-updates

# 更新依赖版本
mvn versions:update-properties

# 安装到本地仓库
mvn install

# 部署到远程仓库
mvn deploy

# 清理编译输出
mvn clean

# 完整的清理和编译流程
mvn clean compile test package

# 仅生成项目站点文档
mvn site
```

### Maven调试和故障排查

```bash
# 启用调试输出（显示详细信息）
mvn -X clean package

# 启用调试输出并显示Maven插件版本
mvn -X clean package -v

# 显示有效的pom.xml（所有继承和插件配置）
mvn help:effective-pom

# 显示有效的settings.xml
mvn help:effective-settings

# 执行离线编译（不下载依赖）
mvn -o clean package

# 检查POM文件
mvn pom:effective

# 查看Project信息
mvn help:describe

# 查看特定插件帮助
mvn help:describe -Dplugin=org.apache.maven.plugins:maven-compiler-plugin

# 跳过某个生命周期阶段
mvn clean package -DskipTests

# 执行单个Mojo
mvn org.apache.maven.plugins:maven-shade-plugin:3.2.4:shade

# 显示活跃的profiles
mvn help:active-profiles

# 查看项目信息
mvn project-info-reports:dependencies
```

### Maven Spring Boot特定命令

```bash
# 添加Spring Boot Starter依赖
mvn dependency:add -Dartifact=org.springframework.boot:spring-boot-starter-web:2.7.0

# 启动Spring Boot应用
mvn spring-boot:run

# 指定活跃的profile启动
mvn spring-boot:run -Dspring-boot.run.arguments="--spring.profiles.active=dev"

# 设置应用配置文件
mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8081"

# 启用热部署
mvn -X spring-boot:run -Dspring-boot.run.jvmArguments="-Xmx1024m -Xms256m"

# 生成可执行JAR
mvn clean package -DskipTests

# 生成并运行可执行JAR
mvn clean package -DskipTests && java -jar target/myapp-1.0.0.jar

# 查看Spring Boot依赖
mvn dependency:tree | grep -i spring
```

---

## Gradle项目管理

### Gradle安装和配置

```bash
# 安装Gradle
sudo apt install -y gradle

# 验证Gradle安装
gradle --version

# 查看Gradle帮助
gradle --help

# 初始化Gradle项目
gradle init

# 交互式初始化
gradle init --type java-application
gradle init --type java-library

# 配置Gradle wrapper（推荐用于项目）
gradle wrapper --gradle-version=7.6.1

# 使用wrapper运行Gradle命令
./gradlew build

# 查看Gradle配置
cat gradle.properties

# 查看构建脚本
cat build.gradle
```

### Gradle基础命令

```bash
# 列出所有可用的任务
gradle tasks
gradle tasks --all

# 查看任务详细信息
gradle tasks --all | grep compile

# 构建项目
gradle build

# 清理构建输出
gradle clean

# 完整的清理和构建
gradle clean build

# 编译源码
gradle compileJava

# 编译测试代码
gradle compileTestJava

# 运行单元测试
gradle test

# 跳过测试构建
gradle build -x test

# 只运行特定测试
gradle test --tests com.example.MyTest

# 打包项目
gradle jar

# 生成构建扫描
gradle build --scan

# 查看构建时间
gradle build --profile

# 显示项目属性
gradle properties

# 显示项目依赖
gradle dependencies

# 显示依赖树
gradle dependencies --configuration implementation

# 检查最新的依赖版本
gradle dependencyUpdates

# 运行应用
gradle run

# 指定主类运行
gradle run --args 'arg1 arg2'

# 执行单个任务
gradle clean
gradle build
gradle test

# 执行多个任务
gradle clean build test
```

### Gradle调试和优化

```bash
# 启用调试输出
gradle build --debug

# 启用info日志级别
gradle build --info

# 启用quiet模式（只显示错误）
gradle build --quiet

# 显示堆栈跟踪
gradle build --stacktrace

# 显示完整堆栈跟踪
gradle build --full-stacktrace

# 并行构建（加快速度）
gradle build --parallel

# 启用build cache
gradle build --build-cache

# 查看build cache统计
gradle build --build-cache --verbose

# 刷新dependencies缓存
gradle build --refresh-dependencies

# 强制重新下载
gradle build --write-locks

# 离线构建（使用缓存）
gradle build --offline

# 查看构建性能
gradle build --profile

# 只显示最终输出
gradle build -q

# 在后台运行（守护进程）
gradle --daemon build

# 停止守护进程
gradle --stop

# 禁用守护进程
gradle --no-daemon build
```

### Gradle Spring Boot特定命令

```bash
# 添加Spring Boot插件
cat >> build.gradle << 'EOF'
plugins {
  id 'org.springframework.boot' version '2.7.0'
}
EOF

# 运行Spring Boot应用
gradle bootRun

# 指定参数运行
gradle bootRun --args='--server.port=8081'

# 构建可执行JAR
gradle bootJar

# 查看bootJar任务
gradle bootJar --help

# 生成可执行JAR并运行
gradle bootJar && java -jar build/libs/myapp-1.0.0.jar

# 查看Spring Boot依赖
gradle dependencies | grep -i spring
```

---

## 应用运行

### 直接运行JAR文件

```bash
# 简单运行
java -jar application.jar

# 查看JAR文件内容
jar tf application.jar | head -20

# 查看JAR文件大小
ls -lh application.jar

# 后台运行并保存日志
java -jar application.jar > app.log 2>&1 &

# 后台运行并获取PID
java -jar application.jar > app.log 2>&1 &
echo $! > app.pid

# 使用nohup后台运行
nohup java -jar application.jar > app.log 2>&1 &

# 查看运行中的Java进程
ps aux | grep java

# 获取Java进程PID
pgrep -f "application.jar"

# 停止应用
kill $(cat app.pid)

# 强制停止
kill -9 $(pgrep -f "application.jar")
```

### JVM参数配置

```bash
# 堆内存配置
java -Xms512m -Xmx2g -jar application.jar

# 详解：
# -Xms512m    初始堆大小为512MB
# -Xmx2g      最大堆大小为2GB

# 新生代和老年代比例
java -Xms1g -Xmx2g -XX:NewRatio=3 -jar application.jar
# NewRatio=3 表示老年代:新生代 = 3:1

# 新生代大小设置
java -Xms1g -Xmx2g -Xmn512m -jar application.jar
# 设置新生代为512MB，老年代为剩余部分

# 线程栈大小（解决StackOverflow）
java -Xss256k -jar application.jar

# Survivor区域比例
java -Xms1g -Xmx2g -XX:SurvivorRatio=8 -jar application.jar
```

### 垃圾回收器配置

```bash
# 使用G1GC（推荐）
java -XX:+UseG1GC -Xms1g -Xmx4g -jar application.jar

# 配置G1GC的最大暂停时间
java -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -Xms1g -Xmx4g -jar application.jar

# 使用Parallel GC（多核服务器推荐）
java -XX:+UseParallelGC -XX:+UseParallelOldGC -Xms1g -Xmx4g -jar application.jar

# 设置GC线程数
java -XX:+UseG1GC -XX:ParallelGCThreads=8 -Xms1g -Xmx4g -jar application.jar

# 使用CMS GC（低延迟，已废弃）
java -XX:+UseConcMarkSweepGC -Xms1g -Xmx4g -jar application.jar

# 使用ZGC（超低延迟，需要Java 11+）
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -Xms1g -Xmx4g -jar application.jar

# 启用GC日志
java -Xloggc:gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -jar application.jar

# GC日志输出到stdout
java -XX:+PrintGCDetails -XX:+PrintGCDateStamps -jar application.jar

# 打印GC时间
java -XX:+PrintGCTimeStamps -jar application.jar
```

### 文件和系统配置

```bash
# 指定文件编码
java -Dfile.encoding=UTF-8 -jar application.jar

# 指定时区
java -Duser.timezone=Asia/Shanghai -jar application.jar

# 指定语言
java -Duser.language=zh -Duser.country=CN -jar application.jar

# 指定临时目录
java -Djava.io.tmpdir=/tmp -jar application.jar

# 禁用DNS缓存
java -Dnetworkaddress.cache.ttl=0 -jar application.jar

# 设置DNS缓存时间
java -Dnetworkaddress.cache.ttl=300 -jar application.jar

# 启用系统代理
java -Djava.net.useSystemProxies=true -jar application.jar
```

### 调试和性能分析

```bash
# 启用远程调试
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -jar application.jar

# 启用JFR（Java Flight Recorder）性能录制
java -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -jar application.jar

# 启用性能分析
java -XX:+PrintCompilation -jar application.jar

# 打印所有系统属性
java -XshowSettings:properties -jar application.jar

# 显示JVM设置
java -XshowSettings:vm -version

# 启用类加载日志
java -verbose:class -jar application.jar

# 结合多个参数的典型配置
java -Xms1g -Xmx2g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -Dfile.encoding=UTF-8 \
  -Duser.timezone=Asia/Shanghai \
  -jar application.jar
```

---

## Spring Boot框架

### 创建Spring Boot项目

```bash
# 方式1：使用Maven archetype
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=myapp \
  -DarchetypeArtifactId=maven-archetype-quickstart

# 方式2：使用Spring Boot CLI
spring boot new myapp

# 方式3：使用start.spring.io在线创建

# 方式4：手动创建pom.xml
cat > pom.xml << 'EOF'
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>myapp</artifactId>
  <version>1.0.0</version>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.0</version>
  </parent>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
  </dependencies>
</project>
EOF
```

### Spring Boot开发命令

```bash
# 运行Spring Boot应用（开发模式）
mvn spring-boot:run

# 启用热部署
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xmx1024m"

# 指定活跃的profile
mvn spring-boot:run -Dspring-boot.run.arguments="--spring.profiles.active=dev"

# 打包可执行JAR
mvn clean package -DskipTests

# 运行打包后的JAR
java -jar target/myapp-1.0.0.jar

# 指定服务器端口
mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8081"

# 启用debug模式
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005"

# 查看Spring Boot版本
mvn dependency:tree | grep spring-boot-starter

# 查看活跃的配置
mvn spring-boot:run -Dspring-boot.run.arguments="--debug"
```

### Spring Boot配置

```bash
# application.properties配置示例
cat > src/main/resources/application.properties << 'EOF'
# 服务器配置
server.port=8080
server.servlet.context-path=/api

# 数据库配置
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA配置
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# 日志配置
logging.level.root=INFO
logging.level.com.example=DEBUG
logging.file.name=logs/application.log

# 应用名称
spring.application.name=myapp
EOF

# application.yml配置示例
cat > src/main/resources/application.yml << 'EOF'
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  application:
    name: myapp
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

logging:
  level:
    root: INFO
    com.example: DEBUG
  file:
    name: logs/application.log
EOF
```

### Spring Boot常用依赖

```bash
# 添加Web依赖
mvn dependency:add -Dartifact=org.springframework.boot:spring-boot-starter-web:2.7.0

# 添加数据库依赖
mvn dependency:add -Dartifact=org.springframework.boot:spring-boot-starter-data-jpa:2.7.0

# 添加MySQL驱动
mvn dependency:add -Dartifact=mysql:mysql-connector-java:8.0.29

# 添加Lombok（简化代码）
mvn dependency:add -Dartifact=org.projectlombok:lombok:1.18.24

# 添加测试依赖
mvn dependency:add -Dartifact=org.springframework.boot:spring-boot-starter-test:2.7.0

# 添加Redis依赖
mvn dependency:add -Dartifact=org.springframework.boot:spring-boot-starter-data-redis:2.7.0

# 查看已安装的Spring Boot依赖
mvn dependency:tree | grep -i spring
```

---

## JVM监控工具

### JPS - Java进程状态

```bash
# 查看所有Java进程
jps

# 显示进程ID和类名
jps -l

# 显示进程ID和main方法参数
jps -m

# 显示进程ID和JVM参数
jps -v

# 组合参数
jps -lm
jps -lv

# 查看远程机器的Java进程（需配置RMI）
jps remote_host
jps -l remote_host:port
```

### JSTAT - JVM统计监控工具

```bash
# 查看GC统计（每1000ms输出一次，共10次）
jstat -gc PID 1000 10

# 查看GC统计百分比
jstat -gcutil PID 1000 10

# 查看最后一次GC原因
jstat -gccause PID

# 查看类加载统计
jstat -class PID 1000 10

# 查看编译统计
jstat -compiler PID 1000 10

# 查看新生代大小
jstat -gcnew PID

# 查看老年代大小
jstat -gcold PID

# 查看永久代/元数据空间大小
jstat -gcpermcapacity PID

# 查看GC时间
jstat -gccapacity PID

# 实时监控JVM（推荐用法）
jstat -gc PID 1000    # 每1秒输出一次
jstat -gcutil PID 1000 &  # 后台监控
```

### JMAP - JVM内存映射工具

```bash
# 查看堆信息
jmap -heap PID

# 查看堆内对象统计
jmap -histo PID

# 查看堆内活对象统计
jmap -histo:live PID

# 导出堆文件（dump）
jmap -dump:live,format=b,file=heap.bin PID

# 导出完整堆文件
jmap -dump:format=b,file=heap_full.bin PID

# 查看共享对象统计
jmap -shared PID

# 查看finalizer队列
jmap -finalizerinfo PID

# 导出堆文件后使用MAT工具分析
# 安装MAT: https://www.eclipse.org/mat/downloads.php
# 打开heap.bin文件进行分析
```

### JSTACK - 线程堆栈分析

```bash
# 查看所有线程信息
jstack PID

# 保存到文件
jstack PID > threads.log

# 查看运行中的线程
jstack PID | grep RUNNABLE

# 查看等待中的线程
jstack PID | grep WAITING

# 查看监控线程
jstack PID | grep MONITOR

# 查看特定线程
jstack PID | grep "thread-name"

# 查看死锁信息
jstack PID | grep -A 10 "Found one Java-level deadlock"

# 统计线程数
jstack PID | grep "java.lang.Thread.State" | wc -l

# 查看线程详细状态
jstack -l PID

# 后台定期保存线程信息
while true; do jstack PID >> threads_$(date +%s).log; sleep 10; done
```

### JCMD - Java诊断命令行工具

```bash
# 查看所有可用命令
jcmd PID help

# 查看JVM运行时间
jcmd PID VM.uptime

# 查看系统属性
jcmd PID VM.system_properties

# 查看VM参数
jcmd PID VM.flags

# 查看进程信息
jcmd PID VM.version

# 导出堆文件
jcmd PID GC.heap_dump /tmp/heap.bin

# 执行垃圾回收
jcmd PID GC.run

# 打印所有线程
jcmd PID Thread.print

# 查看类统计
jcmd PID GC.class_histogram

# 启用/禁用诊断选项
jcmd PID VM.unlock_diagnostic_vm_options

# 动态修改日志级别
jcmd PID VM.log what=gc:level=debug
```

### JVisualVM - 可视化监控工具

```bash
# 启动JVisualVM
jvisualvm

# 通过命令行启动并连接到特定进程
jvisualvm --openpid PID

# 查看本地进程
# 双击进程查看详细信息

# 监控项：
# - Heap: 堆内存使用
# - Threads: 线程状态
# - Classes: 类加载情况
# - Sampler: 性能采样

# 使用方式：
# 1. 打开JVisualVM
# 2. 在左侧选择进程
# 3. 点击Profiler标签
# 4. 选择CPU或内存
# 5. 点击Profile按钮开始
# 6. 执行应用代码
# 7. 点击Stop查看结果
```

---

## 性能优化

### JVM参数优化组合

```bash
# 开发环境推荐配置
java -Xms256m -Xmx512m \
  -XX:+UseG1GC \
  -Dfile.encoding=UTF-8 \
  -jar application.jar

# 测试环境推荐配置
java -Xms1g -Xmx2g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -Xloggc:gc.log \
  -XX:+PrintGCDetails \
  -XX:+PrintGCDateStamps \
  -Dfile.encoding=UTF-8 \
  -jar application.jar

# 生产环境推荐配置
java -Xms4g -Xmx4g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=100 \
  -XX:ParallelGCThreads=8 \
  -Xloggc:/var/log/app/gc.log \
  -XX:+PrintGCDetails \
  -XX:+PrintGCDateStamps \
  -XX:+PrintGCApplicationStoppedTime \
  -XX:NumberOfGCLogFiles=10 \
  -XX:GCLogFileSize=1G \
  -XX:+UseStringDeduplication \
  -XX:MaxGCPauseMillis=100 \
  -Dfile.encoding=UTF-8 \
  -Duser.timezone=Asia/Shanghai \
  -jar application.jar

# 高并发场景配置
java -Xms8g -Xmx8g \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=50 \
  -XX:+ParallelRefProcEnabled \
  -XX:+UnlockDiagnosticVMOptions \
  -XX:G1NewCollectionHeuristicWeight=35 \
  -jar application.jar

# 内存受限场景配置
java -Xms256m -Xmx512m \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -XX:CompressedClassSpaceSize=128m \
  -jar application.jar
```

### 垃圾回收优化

```bash
# 查看当前GC配置
java -XX:+PrintFlagsFinal -version | grep -i gc

# 启用GC日志分析
java -Xloggc:gc.log \
  -XX:+PrintGCDetails \
  -XX:+PrintGCDateStamps \
  -XX:+PrintGCApplicationStoppedTime \
  -XX:+PrintSafepointStatistics \
  -jar application.jar

# G1GC特定优化
java -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=100 \
  -XX:+ParallelRefProcEnabled \
  -XX:+AlwaysPreTouch \
  -XX:+UseStringDeduplication \
  -jar application.jar

# 降低GC频率
java -XX:+UseG1GC \
  -XX:InitiatingHeapOccupancyPercent=35 \
  -jar application.jar

# 停止世界时间优化
java -XX:+UseG1GC \
  -XX:+ParallelRefProcEnabled \
  -XX:+PrintGCApplicationConcurrentTime \
  -XX:+PrintGCApplicationStoppedTime \
  -jar application.jar
```

### 内存泄漏检测

```bash
# 启用OOM时自动dump堆
java -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=/tmp/heapdump.bin \
  -jar application.jar

# 启用GC前后的统计
java -XX:+PrintGCDetails \
  -XX:+PrintMemoryUsage \
  -jar application.jar

# 监控内存趋势
while true; do
  jmap -histo:live PID | head -20 >> memory_trend.log
  sleep 60
done

# 使用JConsole连接并监控内存
jconsole &

# 导出堆文件进行分析
jmap -dump:live,format=b,file=heap.bin PID

# 使用Eclipse MAT分析堆文件
# 下载: https://www.eclipse.org/mat/
# 打开heap.bin文件
```

---

## 应用部署

### Systemd服务配置

```bash
# 创建systemd服务文件
sudo tee /etc/systemd/system/myapp.service > /dev/null << 'EOF'
[Unit]
Description=My Java Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp

# JVM配置
Environment="JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64"
Environment="JAVA_OPTS=-Xms1g -Xmx2g -XX:+UseG1GC"

ExecStart=/usr/bin/java ${JAVA_OPTS} \
  -Dfile.encoding=UTF-8 \
  -Duser.timezone=Asia/Shanghai \
  -jar /opt/myapp/application.jar

# 重启策略
Restart=on-failure
RestartSec=10

# 日志
StandardOutput=journal
StandardError=journal
SyslogIdentifier=myapp

# 优雅关闭时间
TimeoutStopSec=30

[Install]
WantedBy=multi-user.target
EOF

# 重新加载systemd配置
sudo systemctl daemon-reload

# 启用服务（开机自启）
sudo systemctl enable myapp

# 启动服务
sudo systemctl start myapp

# 查看状态
sudo systemctl status myapp

# 查看日志
journalctl -u myapp -f

# 停止服务
sudo systemctl stop myapp

# 重启服务
sudo systemctl restart myapp
```

### Docker部署

```dockerfile
# 多阶段构建 - Dockerfile
FROM maven:3.8-openjdk-17 as builder
WORKDIR /build
COPY . .
RUN mvn clean package -DskipTests

# 运行镜像
FROM openjdk:17-jdk-slim
WORKDIR /app

# 复制JAR文件
COPY --from=builder /build/target/application.jar app.jar

# JVM参数
ENV JAVA_OPTS="-Xms512m -Xmx1g -XX:+UseG1GC"

# 暴露端口
EXPOSE 8080

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8080/actuator/health || exit 1

# 启动应用
ENTRYPOINT ["java", "-jar", "app.jar"]
CMD []
```

```bash
# 构建Docker镜像
docker build -t myapp:1.0 .

# 运行容器
docker run -d \
  --name myapp \
  -p 8080:8080 \
  -e "JAVA_OPTS=-Xms512m -Xmx1g" \
  myapp:1.0

# 查看日志
docker logs -f myapp

# 进入容器
docker exec -it myapp /bin/bash
```

### Kubernetes部署

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
        env:
        - name: JAVA_OPTS
          value: "-Xms512m -Xmx1g -XX:+UseG1GC"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

---

## 常见问题

### 问题1: 应用内存不足（OOM）

**症状**: 应用被杀死，日志显示OutOfMemoryError

**诊断**:
```bash
# 查看进程内存
ps aux | grep java
# 查看内存排名
ps aux --sort=-%mem | head

# 导出堆文件
jmap -dump:live,format=b,file=heap.bin PID

# 使用MAT分析heap.bin查找泄漏
```

**解决**:
```bash
# 增加堆大小
java -Xms2g -Xmx4g -jar application.jar

# 使用G1GC
java -XX:+UseG1GC -Xms2g -Xmx4g -jar application.jar

# 启用String去重
java -XX:+UseG1GC -XX:+UseStringDeduplication -Xms2g -Xmx4g -jar application.jar
```

### 问题2: 应用启动缓慢

**症状**: 应用启动需要很长时间

**诊断**:
```bash
# 测量启动时间
time java -jar application.jar

# 查看启动日志
java -Xlog:class+load=info -jar application.jar

# 启用CDS（类数据共享）
java -Xshare:dump
java -Xshare:on -jar application.jar
```

**解决**:
```bash
# 使用CDS加速启动
java -Xshare:on -jar application.jar

# 异步初始化
java -XX:+TieredCompilation -XX:TieredStopAtLevel=4 -jar application.jar

# 预热编译
java -XX:+TieredCompilation -XX:CompileCommand=exclude,*.*,* -XX:TieredStopAtLevel=4 -jar application.jar
```

### 问题3: CPU占用率高

**症状**: Java应用CPU占用率接近100%

**诊断**:
```bash
# 查看哪个线程占用CPU
top -H -p PID

# 导出线程信息
jstack PID > threads.log

# 查看频繁运行的方法
jcmd PID VM.log what=application:level=debug

# 使用jvisualvm采样分析
jvisualvm --openpid PID
```

**解决**:
```bash
# 优化GC
java -XX:+UseG1GC -XX:MaxGCPauseMillis=100 -jar application.jar

# 减少线程数
java -Djava.util.concurrent.ForkJoinPool.common.parallelism=4 -jar application.jar

# 调整应用配置（减少轮询、优化算法等）
```

### 问题4: 垃圾回收停顿时间长

**症状**: 应用出现周期性的长延迟

**诊断**:
```bash
# 分析GC日志
java -Xloggc:gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -jar application.jar

# 使用GCViewer分析
# 下载: https://github.com/chewiebug/GCViewer

# 查看GC频率
jstat -gcutil PID 1000 30
```

**解决**:
```bash
# 使用ZGC（超低延迟）
java -XX:+UnlockExperimentalVMOptions -XX:+UseZGC -jar application.jar

# 调整G1GC参数
java -XX:+UseG1GC -XX:MaxGCPauseMillis=50 -XX:+ParallelRefProcEnabled -jar application.jar

# 扩大堆大小（减少GC频率）
java -Xms4g -Xmx4g -XX:+UseG1GC -jar application.jar
```

---

## 快速参考

### 最常用的10个命令

| 用途 | 命令 |
|------|------|
| 查看版本 | `java -version` |
| 运行应用 | `java -Xms1g -Xmx2g -jar app.jar` |
| Maven打包 | `mvn clean package -DskipTests` |
| Maven运行 | `mvn spring-boot:run` |
| 查看GC | `jstat -gc PID 1000` |
| 导出堆 | `jmap -dump:live,format=b,file=heap.bin PID` |
| 查看线程 | `jstack PID` |
| 查看进程 | `jps -lv` |
| 启动服务 | `systemctl start myapp` |
| 查看日志 | `journalctl -u myapp -f` |

### Maven常用命令速查

```bash
mvn clean compile      # 编译
mvn clean test         # 测试
mvn clean package      # 打包
mvn clean install      # 安装
mvn spring-boot:run    # 运行
mvn dependency:tree    # 查看依赖树
mvn versions:display-dependency-updates  # 检查更新
```

### Gradle常用命令速查

```bash
gradle build           # 构建
gradle test            # 测试
gradle bootRun         # 运行
gradle dependencies    # 查看依赖
gradle tasks           # 列出所有任务
gradle clean build     # 清理后构建
```

### JVM常用参数速查

```bash
-Xms1g                 # 初始堆1GB
-Xmx2g                 # 最大堆2GB
-XX:+UseG1GC           # 使用G1GC
-Dfile.encoding=UTF-8  # 文件编码
-Duser.timezone=Asia/Shanghai  # 时区
```

---

**版本**: 1.0.0  
**最后更新**: 2025-11-24  
**命令数**: 150+  
**配置示例**: 8个
## 用sbt编译spark源码 ##

    ./build/sbt -Pyarn -Phadoop-2.7 package //版本自选

注：网速很慢，笔者大约花了30分钟。采用stan兄的方法应该会快点，将源改为aliyun。

stan兄链接：
[使用阿里云的Maven仓库加速Spark编译过程](https://zhuanlan.zhihu.com/p/25279570)



### 单独编译一个module ###

    $./build/sbt
	>   project core //进入到core子项目
	>   package
	$ # or you can build the spark-core module with sbt directly using:
	$ build/sbt core/package

最后将spark-core_2.11-2.1.0.jar替换到assembly/target/scala-2.11/jars目录下就可以了。



### sbt单独test case ###

1.Spark为了确保代码风格一致规范，在项目引入了Scala-style checker，直接编写test case易陷入坑，建议复制test case然后自己修改。
eg：
```
cd spark/sql/core/src/test/scala/org/apache/spark/sql/
cp SQLQuerySuite.scala SimpleSuite.scala 
```
编写SimpleSuite.scala代码如下：
![](http://i.imgur.com/FgOpkjc.png)

打开IDEA的Terminal窗口,用sbt单独测试SimpleSuite.scala
代码如下：
```
./build/sbt
> project sql
> testOnly *SimpleSuite
 ```
如果顺利会出现如下结果：
![](http://i.imgur.com/892KpgQ.png)

## sbt远程调试 ##
1.在Run/edit Configurations/+ ,新建一个Remote调用，Name自拟，端口默认为5005.
2.进入IDEA的Terminal重新启动sbt，启动时需要添加远程调试参数：`build/sbt -jvm-debug 5005`
此时终端会显示：Listening for transport dt_socket at address: 5005
3.启动sbt调试后，可以用利用IDEA对sbt调试。点击IDEA中右上角的绿色虫子（Debug）
4.给SimpleSuite.scala添加一个断点，然后进入sbt终端执行如下代码：
```
> project sql
> set fork in Test := false //让sbt执行Test时避免fork子进程
> testOnly *SimpleSuite
```
![](http://i.imgur.com/noBih46.png)

如果频繁修改代码，可以执行`~testOnly *SimpleSuite`，sbt会监控文件变化并自动执行测试。

感谢stan兄！！！附上大佬博客：[sbt结合IDEA对Spark进行断点调试开发](https://zhuanlan.zhihu.com/p/26900703)

sbt远程调试参考：
[连城大佬知乎](https://www.zhihu.com/question/24869894)

[官网Useful Developer Tools](http://spark.apache.org/developer-tools.html)



## 用maven编译spark源码 ##

参考csdn博客：[修改maven源加速spark编译过程](http://blog.csdn.net/microsoft2014/article/details/56069376)

## 使用maven单独编译 ##
我们在spark-1.6.1根目录下运行

`mvn -pl examples scala:compile jar:jar`

告诉 maven 只在 examples 这个 submodule 下运行scala:compile和jar:jar两个 maven goals, 能够看到类似输出：

```scala
[damon@localhost spark-2.1.0]$ mvn -pl examples scala:compile jar:jar
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Spark Project Examples 2.1.0
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- scala-maven-plugin:3.2.2:compile (default-cli) @ spark-examples_2.11 ---
[INFO] Using zinc server for incremental compilation
[info] Compiling 1 Scala source to /home/damon/spark-2.1.0/examples/target/scala-2.11/classes...
[info] Compile success at May 17, 2017 8:50:03 PM [0.821s]
[INFO] 
[INFO] --- maven-jar-plugin:2.6:jar (default-cli) @ spark-examples_2.11 ---
[INFO] Building jar: /home/damon/spark-2.1.0/examples/target/scala-2.11/jars/spark-examples_2.11-2.1.0.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.502 s
[INFO] Finished at: 2017-05-17T20:50:04+08:00
[INFO] Final Memory: 49M/637M
[INFO] ------------------------------------------------------------------------

```
## 远程调试 ## 
因为spark job需要通过./bin/spark-submit脚本提交运行，所以断点调试很麻烦，因为我们编写的代码和server端属于不同的process，不能直接调试。用Intellij 这边用debugger远程连过去进行调试，实质上是在localhost上开放一个端口而已。如果调入assembly jars，然后开发application可直接断点调试。
进入Run -> Edit Configurations, 单击左上角+, 新建一个 Remote Configuration, 名字随便改一下, 其余留作默认. 点击OK之前, 复制Configurationtab下的Command line arguments for running remote JVM里面的内容, 之后会用到这个参数, 并做如下更改：

    -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
    改成
    -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005

回到命令行，执行：`mvn -pl examples scala:compile jar:jar`	打包
执行：
```
bin/spark-submit \
--driver-java-options "-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005" \ 
--class org.apache.spark.examples.SparkPi \
examples/target/spark-examples_2.10-1.6.1.jar
```
suspend=y表示启动执行暂停，等待debugger连接之后开始运行（5005端口）
命令行会显示Listening for transport dt_socket at address: 5005
此时只需要在 Intellij 上打开SparkPi.scala文件, 加上断点, 再点Run -> Debug 'Remote', 就能开始单步追踪调试了.
感谢：
[Spark+Intellij 舒服的源码开发环境配置](http://dragonly.github.io/note/2016/05/10/Spark+Intellij-%E8%88%92%E6%9C%8D%E7%9A%84%E6%BA%90%E7%A0%81%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE.html)

[Building Spark官网](http://spark.apache.org/docs/latest/building-spark.html#building-spark)

拓展：有时在linux环境下git版本为1.7.0不适合IDEA的最低要求，需要升级git版本，常规使用命令`yum install git`不管用，需要直接[Installing Git on CentOS from sources](http://opensourceforgeeks.blogspot.in/2013/11/installing-git-on-centos-from-sources.html)。




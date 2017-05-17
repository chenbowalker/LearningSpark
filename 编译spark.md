## 用sbt编译spark源码 ##

    ./build/sbt -Pyarn -Phadoop-2.7 package //版本自选

注：网速很慢，笔者大约花了40分钟。采用stan兄的方法应该会快点，将源改为aliyun。

stan兄链接：[使用阿里云的Maven仓库加速Spark编译过程](https://zhuanlan.zhihu.com/p/25279570)

### 单独编译一个module ###

    $	./build/sbt
	>   project core //进入到core子项目
	>   package
	$ # or you can build the spark-core module with sbt directly using:
	$ build/sbt core/package

最后将spark-core_2.11-2.1.0.jar替换到assembly/target/scala-2.11/jars目录下就可以了。

##用maven编译spark源码##

参考csdn博客：[修改maven源加速spark编译过程](http://blog.csdn.net/microsoft2014/article/details/56069376)

参考：

[Building Spark官网](http://spark.apache.org/docs/latest/building-spark.html#building-spark)




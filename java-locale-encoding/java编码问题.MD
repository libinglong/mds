#### java 默认locale与编码问题

java编码问题其实分为很多种，比如jvm默认locale设置，默认文件编码方式设置，日志编码设置，源文件编码设置，本文档持续更新所有的编码相关问题

* 影响locale，文件编码的jvm属性

  * user.language

  * user.country

  * user.variant

    这个我们一般不关心

  * file.encoding

  这几个属性，首先jvm会通过复杂的逻辑进行初始化，这部分c代码详见openjdk：System.c，java_props_md.c，java_props_macosx.c，该部分逻辑，在不同的平台会有不同，绝不是仅仅通过LANG，LC_ALL，LC_*，LANGUAGE环境变量决定的，由于逻辑较为复杂，而我的clion过期了，因此我也没有debug详细跟踪一下。

  如果不想受系统环境影响，或者不想探究其影响逻辑的话，可以直接在jvm参数中指定上述属性的值，从而在初始化逻辑中覆盖之前得到的值。这其实是最简单高效的方法。

  从这里我们需要认清一点，环境变量是否能够影响程序的locale，完全取决于程序本身有没有读取这几个变量，并按照该变量设置locale，在jvm的实现中，不只是考虑了这些变量的，还有和平台相关的代码。

* macos

  根据首选语言和地区，user.language=zh，user.country=CN，文件编码通过什么影响的我没有找到

  ![image-20211217182748500](images/image-20211217182748500.png)

* Debian/Ubuntu

  Debian/Ubuntu容器环境下，如果在dockerfile中指定下述命令

  ```dockerfile
  # Debian 先安装locales
  RUN apt-get update
  RUN apt-get -y install locales
  
  
  RUN sed -i '/zh_CN.UTF-8/s/^# //g' /etc/locale.gen && \
      locale-gen
  ENV LC_ALL zh_CN.UTF-8
  ```

  得到下述结果，上述命令确实可以完全影响到我们需要的几个属性，但要注意的是，zh_CN.UTF-8这个值必须是locale支持的，不然会得到意想不到的结果，可以查看/etc/locale.gen文件得到支持的值(这个文件中的值是否齐全我不清楚，也没有去搜索相关文档)。

  ```java
  使用下述代码输出
  System.out.println("user.language=" + System.getProperty("user.language"));
  System.out.println("user.country=" + System.getProperty("user.country"));
  System.out.println("file.encoding=" + System.getProperty("file.encoding"));
  System.out.println("default locale=" + Locale.getDefault());
  ```

  ```properties
  user.language=zh
  user.country=CN
  file.encoding=UTF-8
  default locale=zh_CN
  ```

* Centos7

  Centos7容器环境下，则需要在dockerfile中指定下述命令

  ```dockerfile
  RUN localedef -c -f EUC-TW -i zh_TW zh_TW.EUC-TW
  ENV LC_ALL zh_TW.EUC-TW
  ```

  ```properties
  user.language=zh
  user.country=TW
  file.encoding=EUC-TW
  default locale=zh_TW
  ```

#### 网上流言

网上多数的流言大概理论如下，但实际上这些环境变量对jvm的影响是有限的，使用这些环境变量往往得不到预期的结果

https://jarirajari.wordpress.com/2020/11/23/how-to-set-locale-in-linux-for-jvm/

> 1. If the JVM setting `user.language` is set, use it and look no further
> 2. If user language is not set, scan the environment for `LC_ALL`
> 3. If `LC_ALL` does not exist, scan the environment for `LANG`
> 4. If `LANG` is not set, default to `en_US` locale

以上述文章为例，其列举了环境变量的优先级，但这和java应用的实际运行情况显然是不匹配的。

#### 容器内java编码问题排查方法

##### 普通java应用

* 对于运行时的java应用，下载arthas，查看系统属性，看看值是否符合预期，这是最根本的方式。

* 查看环境变量，由于环境变量不可以完全反映出jvm的默认locale和编码，因此作为只能作为一个辅助查询的手段，如果环境变量没有问题，仍然要通过jvm系统属性排查

##### 应用的maven构建过程中出现编码问题

maven也是基于java的应用，因此在构建过程中如果出现编码问题，也可以通过这个方法来排查，当然maven本身的编码相关的选项也是要考虑的

输入mvn -v

```te
jack@jackdeMacBook-Pro mds % mvn -v
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /Users/jack/software/apache-maven-3.6.3
Java version: 1.8.0_301, vendor: Oracle Corporation, runtime: /Library/Java/JavaVirtualMachines/jdk1.8.0_301.jdk/Contents/Home/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "mac os x", version: "10.15.7", arch: "x86_64", family: "mac"
```

#### 默认locale的作用

默认locale只是系统提供的一种默认的语言地区和编码方式，编写的应用程序未必会使用，一般调用三方的类库，在不指定locale的情况下会用默认的locale，三方类库一般是遵循这种约定俗成的习惯的，但如果是自己开发的类库，就不一定了，因此即使我们指定了默认值，但最终使用的是不是默认值，还得看程序怎么写。




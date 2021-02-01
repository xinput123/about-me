### 转自 [Java代码如何查看字节码及汇编码](https://www.ituring.com.cn/article/515229)

我们知道一个Java类要想被Java虚拟机加载，必须先生成相应的二进制文件，即class文件，而一个方法或一段代码在运行期要么被解释执行，要么被编译器编译成汇编码，编译执行，那如何去查看这些转换后的代码呢？

命令 javap

```
javap -v/-c F.class
```

javap 可以用命令拓展在ide工具里，如：

eclipse Run/External Tools 然后点击要查看的文件，Run/External Tools/javap

解释下配置项Name: 命名，如javap

Location： jdk中javap命令的绝对路径

Working Directory：

工作目录，可以在Variables里选择，

${project_loc} 表示当前所选择的resource所在的project或正构建的project或所选择的的绝对路径

如果当前project已经加入到eclipse的workspace中，也可用 ${workspace_loc}/${project_name}

Arguments：

当前执行文件相对于Working Directory的路径

-classpath target/classes： classpath路径

-v： Javap命令的

${java_type_name}：即当前Java文件对于的class文件，Variables里选择

eclipse拓展Javap命令 也可以在idea File/Settings/Tools/External Tools中配置：

idea拓展Javap命令 配置完后，右键需要查看的文件，选择External Tools/javap -v

查看汇编码

首先要下载hsdis-amd64.dll文件，并将文件放置在%JAVA_HOME%/jre/bin的server和client目录下

放置完后，可以 命令测试 java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -version

如果输出PrintAssembly is enabled，说明hsdis-amd64安装成功

hsdis-amd64安装 注：有些jdk版本没有client目录

然后配置VM

```
-server -Xcomp -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -XX:CompileCommand=compileonly,*Test.add
```

idea中查看汇编码 解释下配置项：

```
java -version -server -Xcomp
```

虚拟机一般有两种模式运行执行代码，即解释模式和编译模式，默认采用解释器与其中一个编译器直接配合的方式工作:混合模式，可以看到Java -version 输出内容有mixed mode

使用参数“-Xint”强制虚拟机运行于“解释模式”（Interpreted Mode）

使用参数“-Xcomp”强制虚拟机运行于“编译模式”（CompiledMode）

而HotSpot虚拟机中内置了两个即时编译器

C1 ClientCompiler

C2 Server Compiler

从Java -version 输出 Java HotSpot(TM) 64-Bit Server VM可以得知，运行于Server模式

-XX:+UnlockDiagnosticVMOptions

打开虚拟机的诊断模式，一些参数（如PrintAssembly）在诊断模式下才能使用

-XX:+PrintAssembly

打印即时编译后的二进制信息，即汇编码

-XX:CompileCommand=compileonly,*Test.add

编译时接受的指令，这里compileonly,*Test.add 是指编译add方法

全能查看，即可查看代码，同时也可以对照字节码和汇编码

在GitHub中下载jitwatch，点击其目录下的launchUI，打开图形窗口，

config中配置Java文件所在的项目路径及classpath路径

jitwatch Config配置 idea VM 参数

-server -Xcomp -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -XX:CompileCommand=compileonly,*Test.add -XX:+LogCompilation -XX:LogFile=jit1.log

运行后会在classpath下生成jit1.log日志文件，点击jitwatch窗口的Open Log导入日志文件，再Staart运行，可以查看执行的Source，Bytecode,Assembiy

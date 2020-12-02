### 一、注册github账号
使用github托管源代码

### 二、注册Sonatype的账户
Sonatype通过JIRA来管理OSSRH仓库。JIRA是一个项目管理服务，类似于国内的Teambition。

注册地址 ： https://issues.sonatype.org/secure/Signup!default.jspa

![0c1ad0911b0508504c198bd656d573cc.png](https://github.com/xinput123/about-me/blob/main/Maven/maven04.jpg)
- Username  登录名称
- Password 密码

**注册完成，首次登录会让选择语言，这里我选择了中文。**

### 三、项目的发布申请
#### 1、首次登陆会直接进入新建页面，之后登录创建发布项目申请需要点击导航栏的“新建”进行项目或问题的创建发布。
![bdb6fa2bcdfbf4b01c3826f7550508b7.png](https://github.com/xinput123/about-me/blob/main/Maven/maven15.jpg)

#### 2、通过在JIRA上创建issue来申请发布新的jar包，Sonatype的工作人员会进行审核，一般按照要求填写不会有问题。

![ddc37d7cfbc5450c71dcb5a853a2341e.png](https://github.com/xinput123/about-me/blob/main/Maven/maven05.jpg)

![d70ddeda631cf512916f5a3f4b8a90bd.png](https://github.com/xinput123/about-me/blob/main/Maven/maven06.jpg)

![ac93cb093515898b439826e3bceb4830.png](https://github.com/xinput123/about-me/blob/main/Maven/maven07.jpg)

- 上图问中文版本时填写信息，其中项目要选择“Community Support”项，对应的问题类型选择“New Project”。按照上述选择，才会在下面展示出对应GroupId和Project信息。

- Project URL便是项目的URL地址，也就是你访问到GitHub项目时的浏览器URL。比如：https://github.com/xinput123/xinput-springboot-parent

- SCM url是基于Https形式访问源代码的链接。我们知道获取GitHub的源代码可以有多种形式，下载zip包、通过“Use ssh”，“Use Https”等形式下载，这里的SCM URL便是Https的url地址，比如：https://github.com/xinput123/xinput-springboot-parent.git

其他非必填信息根据需要选择填写即可。

提交完成之后，会创建一个Issues，内容显示如下： 
![388ee3e982e2c6b81c9f1cd425ffa323.png](https://github.com/xinput123/about-me/blob/main/Maven/maven08.jpg)


很显然，目前处于“待解决”状态，等审核。很快便收到官方审核人员的回复“Waiting for Response”。同时，在Issues下方会出现对应的提示注释信息。
![0699ad45ea3929fc16536c5db763c500.png](https://github.com/xinput123/about-me/blob/main/Maven/maven09.jpg)


这里主要是为验证上面的GroupId，来确定对应的域名是否是你所拥有的。

平台为了验证是否拥有GitHub的账户权限，因此需要申请者在GitHub上创建一个名称为“OSSRH-59503”的项目。在GitHub上创建这么一个空项目，然后在评论区回复即可。

```
Hello, the public repo has created.
https://github.com/xinput123/OSSRH-59503
```

回复完成之后，稍微等待一下，便审核完成。

整个上述过程用大概40分钟，官方回复的还比较及时，由于是下午四五点进行操作的，不确定大家在操作时是否会遇到时差问题。大家可在主面板上查看一下最近其他人提交的Issues的回复情况来确认是否等待。

### 四、安装并配置GPG
发布到Maven仓库中的所有文件都要使用GPG签名，以保障完整性。因此，我们需要在本地安装并配置GPG。

本人采用Mac操作系统，关于其他操作系统的安装大家自行搜索。

MacBook安装GPG非常简单，下载并安装GPG Suite即可：https://gpgtools.org/

安装完成可进入创建GPG密钥对的操作界面，Mac下安装完成弹出如下页面：
![64474a4d13e4140a072f5259d427711f.png](https://github.com/xinput123/about-me/blob/main/Maven/maven10.jpg)

生成密钥时将需要输入name、email以及password。秘钥password在之后的步骤需要用到，请记下来。

公钥创建完成，会自动弹出上传到公共的密钥服务器，这样其他人才可以通过公钥来验证jar包的完整性。
![9747600e2ee338232adfc7fcf015f15c.png](https://github.com/xinput123/about-me/blob/main/Maven/maven11.jpg)

如果忘记了公钥信息可执行gpg --list-keys命令查看本地公钥信息
![28694288f83d5dafb179e8af582f0800.png](https://github.com/xinput123/about-me/blob/main/Maven/maven12.jpg)

也可以通过如下形式将公钥信息上传到服务器：
```
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys 4C1379151A9A49B4CA5608C99FA4AB8ADE435E58
```

其中 4C1379151A9A49B4CA5608C99FA4AB8ADE435E58 便是上面查询到的。

如果是Windows操作系统，同样安装完软件，打开cmd命令行，执行gpg --gen-key生成key，执行gpg --list-keys查看key列表，执行上述命令上传key。

### 五、配置Maven的setting.xml
setting.xml为Maven的全局配置文件，路径为$MAVEN_HOME/conf/settings.xml，需要注册Sonatype的账户时配置的Username和Password添加到servers标签中，这样才能将jar包部署到Sonatype OSSRH仓库：

```
<server>
  <id>sonatype-nexus-snapshots</id>
  <username>Sonatype账号</username>
  <password>Sonatype密码</password>
</server>
```

### 六、配置项目的pom.xml
根据Sonatype OSSRH的要求，以下信息都必须配置：
- **Supply Javadoc and Sources**
- **Sign Files with GPG/PGP**
- **Sufficient Metadata**
- **Correct Coordinates**
- **Project Name, Description and URL**
- **License Information**
- **Developer Information**
- **SCM Information**

##### 6-1、增加开源许可协议，SCM信息，开发者信息等待根据自己信息填写即可。
```
    <licenses>
        <license>
            <name>BSD 3-Clause</name>
            <url>https://spdx.org/licenses/BSD-3-Clause.html</url>
        </license>
    </licenses>

    <scm>
        <connection>https://github.com/xinput123/xinput-springboot-parent.git</connection>
        <url>https://github.com/xinput123/xinput-springboot-parent</url>
    </scm>

    <developers>
        <developer>
            <name>xinput</name>
            <email>xinput.xx@gmail.com</email>
            <roles>
                <role>Developer</role>
            </roles>
            <timezone>+8</timezone>
        </developer>
    </developers>
```

#### 6-2、如果发布Release版本，需要添加Release的相关profile配置，distributionManagement节和maven-compiler-plugin节的配置信息根据自己的实际情况做修改。
```
    <profiles>
        <profile>
            <id>release</id>
            <build>
                <plugins>
                    <!-- Source -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>2.2.1</version>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!-- Javadoc -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>2.9.1</version>
                        <configuration>
                                <additionalparam>-Xdoclint:none</additionalparam>
                        </configuration>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!-- GPG -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.5</version>
                        <executions>
                            <execution>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!--Compiler -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <version>3.0</version>
                        <configuration>
                            <source>1.8</source>
                            <target>1.8</target>
                            <fork>true</fork>
                            <verbose>true</verbose>
                            <encoding>UTF-8</encoding>
                            <showWarnings>false</showWarnings>
                        </configuration>
                    </plugin>
                    <!--Release -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-release-plugin</artifactId>
                        <version>2.5.3</version>
                    </plugin>
                </plugins>
            </build>
            <distributionManagement>
                <snapshotRepository>
                    <id>sonatype-nexus-snapshots</id>
                    <name>Sonatype Nexus Snapshots</name>
                    <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
                </snapshotRepository>
                <repository>
                    <id>sonatype-nexus-snapshots</id>
                    <name>Nexus Release Repository</name>
                    <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
                </repository>
            </distributionManagement>
        </profile>
    </profiles>
```
- 其中snapshotRepository便是在setting.xml中定义的server的id。

### 七、发布jar包
完成上述配置，则可通过命令进行打包上传，即可将jar包发布到Sonatype OSSRH仓库。

```
mvn clean deploy -P release
```

执行上述命令，打包并上传。release便是上面配置的profile元素的id。

在执行的过程中需要输入上面设置的key的密码。执行成功控制台显示情况如图。

![image.png](https://github.com/xinput123/about-me/blob/main/Maven/maven13.jpg)

【友情提示】如果打包过程中出现了401类的错误，可能是因为Maven的配置文件中Server节点配置的用户名和密码不正确，或者Issue还未审核通过。

此时访问上面的任何一个链接，便查询对应的信息。比如将url中的具体文件去掉，只留如下路径：https://oss.sonatype.org/content/repositories/snapshots/com/github/secbr/xinput-springboot-parent/1.0.0-SNAPSHOT/

访问上述路径，便可查看到所有上传的文件信息。

### 八、查看发布jar包
此时进入https://oss.sonatype.org/#stagingRepositories查看发布好的构件，点击左侧的Staging Repositories，可以使用Group Id或其他信息搜索自己的项目。

如果弹出用户名或密码，则输入注册sonatype时对应的用户和密码信息。

此时需注意，如果项目中版本信息为1.0.0-SNAPSHOT，即SNAPSHOT为后缀，则发布的项目位于Snapshots目录下。在左上角的Artifact Search中能够搜到。

如果是Release后缀，则可直接在Staging Repositories中看到（有可能要稍等一下，等待平台处理）。

选中对于的repository之后，点击的close，close时会检查发布的构件是否符合要求。若符合要求，则close成功，成功之后点击箭头所指的release，即可正式将jar包发布到Sonatype OSSRH仓库。

![image.png](https://github.com/xinput123/about-me/blob/main/Maven/maven14.jpg)

当点击Release之后，邮件中会收到Issues变化的信息，提示同步已经被激活，正常10分钟就可以更新同步。

然后回到JIRA中你的Issue，写个comment，我写的是`Component has been successfully issued.`告诉工作人员我发布完成了，等待他们审核。审核通过后我们就可以在中央库搜索到我们的构件了！搜索的地址是： [http://search.maven.org/](http://search.maven.org/)

release成功大概2个小时之后，该构件就会同步到Maven中央仓库，届时会有邮件通知。

实践过程中发现十分钟之内已经成功同步到https://repo1.maven.org/的中央仓库当中。 

### 收尾
当发布到Maven中央仓库完成，可以看到对应的Jar包时，可以对自己提交的Issue增加Comment，留言致谢并表示发布已经完成，请工作人员关闭Issue。有始有终。
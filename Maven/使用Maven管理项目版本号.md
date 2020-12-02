在一个有很多子模块的项目中，修改版本号很麻烦
![maven01.png](https://github.com/xinput123/about-me/blob/main/Maven/maven01.png)

## 规范
- 1、同一项目中所有模块版本保持一致 
- 2、子模块统一继承父模块的版本 
- 3、统一在顶层模块Pom的节中定义所有子模块的依赖版本号，子模块中添加依赖时不要添加版本号 
- 4、开发测试阶段使用SNAPSHOT 
- 5、生产发布使用RELEASE 
- 6、新版本迭代只修改顶层POM中的版本 
如下两图，图一表示根目录pom文件，图二表示web模块pom文件，如果手动修改，则每次都需要修改父pom和各个子pom文件中版本号，很是复杂。

![maven02.png](https://github.com/xinput123/about-me/blob/main/Maven/maven02.png)

![maven03.png](https://github.com/xinput123/about-me/blob/main/Maven/maven03.png)

### 使用maven插件 versions-maven-plugin
```
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>versions-maven-plugin</artifactId>
            <version>2.3</version>
            <configuration>
                <generateBackupPoms>false</generateBackupPoms>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 执行命令
```
mvn versions:set -DnewVersion=2.3-SNAPSHOP
```
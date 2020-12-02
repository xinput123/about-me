## parent标签找不到
```
<parent>
    <groupId>com.xinput</groupId>
    <artifactId>xinput-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>
```

```
 Non-resolvable parent POM for com.xinput:springboot-base-case:1.0-SNAPSHOT: Could not find artifact com.xinput:xinput-parent:pom:1.0-SNAPSHOT in nexus (https://repository.xxxcom/content/groups/public/) and 'parent.relativePath' points at no local POM @ line 5, column 13 -> [Help 2]
```

- 当设置了<parent/>时，是需要有一个relative path来查找父项目的pom.xml的，而如果没有，那么默认的父项目pom.xml就在该pom.xml的父附录中。如果没有,就会根据groupId和artifactId在本地和远端maven仓库中找。很不幸，这个父pom.xml对应的不是外层项目，所以我这里就会报错。
- 所以，添加**空标签<relativePath></relativePath>**就会直接去仓库找了。

## MAVEN插件打包SNAPSHOT包MANIFEST.MF中Class-Path带时间戳的问题
当用maven的maven-jar-plugin插件打包依赖的SNAPSHOT的jar包就会表现为MANIFEST.MF中的Class-Path: lib/model-1.0-20200822.093945-1.jar，但是打包到../lib/model-1.0-SNAPSHOT.jar下面包,这样就会出现找不到类的情况。如下加上**<useUniqueVersions>false</useUniqueVersions>**就可以

强制的把MANIFEST.MF中的Class-Path: lib/model-1.0-20200822.093945-1.jar转化成Class-Path:/lib/model-1.0-SNAPSHOT.jar

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
        <configuration>
            <archive>
                <manifest>
                    <mainClass>com.xinput.App</mainClass>
                    <addClasspath>true</addClasspath>
                    <classpathPrefix>lib/</classpathPrefix>
                    <!-- 如果不加这一句则依赖的SNAPSHOT的jar包就会表现为MANIFEST.MF中的 Class-Path: lib/model-1.0-20200822.093945-1.jar 但是打包到../lib/model-1.0-SNAPSHOT.jar下面包,这样就会出现找不到类的情况 -->
                    <useUniqueVersions>false</useUniqueVersions>
                </manifest>
            </archive>
        <classesDirectory></classesDirectory>
    </configuration>
</plugin>
```
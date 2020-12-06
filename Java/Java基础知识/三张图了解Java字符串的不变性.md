#### 1、定义一个字符串
```
String s = "abcd";
```
![01](https://github.com/xinput123/about-me/blob/main/Java/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/image/01.png)

s 中保存了string对象的引用。下面的箭头可以理解为"存储他的引用"。

#### 2、使用变量来赋值变量
```
String s2 = s;
```
![02](https://github.com/xinput123/about-me/blob/main/Java/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/image/02.png)

s2保存了相同的引用值，因为他们代表同一个对象。

#### 3、字符串连接
```
s = s.concat("ef");
```
![03](https://github.com/xinput123/about-me/blob/main/Java/Java%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/image/03.png)

s中保存的是一个重新创建出来的string对象的引用。
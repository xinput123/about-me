## 前言
typora 没法输入

## 问题
在官方的 issue 有人提到了这个问题

- [Some problems #1215](https://github.com/typora/typora-issues/issues/1215)
- [Can't open any file in macOS 10.15 newest version #2923](https://github.com/typora/typora-issues/issues/2923)

还有官方网站也特别备注了。。

- [support.typora.io/Trouble-Sho…](https://support.typora.io/Trouble-Shooting/)

## 解决。 彻底移除 typora 相关的文件
用管理员的方式执行删除相关文件，
会把主程序也顺带给干掉
然后重新安装即可

```
sudo find / -iname "*typora*" | xargs rm -rf
```

## 快速安装相关软件
```
brew cask install typora appcleaner
```

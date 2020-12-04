### 1、克隆、下载项目
```
git clone git@github.com:xinput123/demo.git 
```

### 2、配置上游仓库地址，可以先使用 git remote -v 查看上游库,指定完以后，可以继续使用这个命令进行验证。
```
git remote add upstream [https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git](https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git)
```

### 3、拉取上游仓库的修改，合并到自己的仓库中，并提交到自己的github上，这样就可以保持和远程上游代码库的一致性了。
```
git fetch upstream 

git merge upstream/master

git push -u origin master
```

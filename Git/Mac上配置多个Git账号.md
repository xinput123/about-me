### 1、查看已配置的git列表
```
git config --list
```

### 2、清空默认的用户名和邮箱
```
git config --global --unset user.name
git config --global --unset user.email
```

### 3、先将电脑上有的配置删除. ~/.ssh目录下

### 4、给不同的git账号生成ssh-key: github和gitlab
```
### github 账号
ssh-keygen -t rsa -f ~/.ssh/id_rsa_github -C "yuan_lai1234@163.com"

### gitlab 账号
ssh-keygen -t rsa -f ~/.ssh/id_rsa_gitlab -C "yuan_lai1234@163.com"
```
- github账号：默认如果不设置名字的话就是id_rsa
- gitlab账号：指定了生成路径，和之前的区分

### 5、添加到ssh-agent信任列表
```
# 添加github的到信任列表
ssh-add ~/.ssh/id_rsa_github

# 添加gitlab的到信任列表
ssh-add ~/.ssh/id_rsa_gitlab
```

### 6、将公钥文件添加对github和gitlab中

### 7、在config文件配置多个ssh-key
```
Host github.com
    User github
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa

# gitlab
Host gitlab.pgxcloud.com
    User gitlab
    HostName gitlab.pgxcloud.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_gitlab
```
- config文件在 /Users/Mac的名称/.ssh/config ，如果没有，创建一个
- Host：随意写，但是最好和HostName保持一致，具体原因是关于一个信任的问题
- Hostname：必须写正确，就是git的公有地址
- IdentityFile：必须写正确，rsa的具体地址
- User：随意写，建议好区分，如使用host的前面部分

### 8、测试连接
```
ssh -T git@github.com
```

### 9、添加配置，看需要
```
git config --global http.postBuffer 524288000
```

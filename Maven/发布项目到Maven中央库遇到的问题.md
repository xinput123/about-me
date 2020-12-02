## 一、No public key: 
No public key: Key with id: (9fa4ab8ade435e58) was not able to be located on <ahref="http://keyserver.ubuntu.com:11371/"> http://keyserver.ubuntu.com:11371/</a>. Upload your public key and try the operation again.
这是因为使用命令上传公钥总是出现该问题，所以我们自选择在wen端直接上传。

#### 1、导出公钥文件
```
gpg --export -a youpublickey > youpublickey.key
```

#### 2、上传公钥到服务器。打开链接[https://keys.openpgp.org/](https://keys.openpgp.org/)
- 选择刚导出的公钥上传
- 直接命令行上传公钥总是出现 该问题的错误，还是直接文件上传靠谱

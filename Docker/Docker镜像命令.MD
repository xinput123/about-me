### Docker批量删除REPOSITORY、TAG为none的镜像
```
docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
```

### 获取所有容器ID
```
docker ps -a -q
```

### 停止所有容器
```
docker stop $(docker ps -a -q)
```

### 删除所有停止的容器
```
docker rm $(docker ps -a -q)
```

### 删除所有的镜像
```
docker rmi $(docker images -q)
```

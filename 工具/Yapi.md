yapi是一个很好用的在线接口文档，并且有一个很好用的插件可以帮助我们开发接口文档。

[YapiIdeaUploadPlugin](https://github.com/diwand/YapiIdeaUploadPlugin)

## 使用方式
### 1、在github disk 中下载最新的jar包 (或者在idea 插件库中搜索 安装）
 
### 2、 如果是下载的jar 需要 打开idea preferneces->plugins-> install plugin from disk,导入jar 包后(install)，重启 

### 3、 配置信息 
#### 3.1、配置信息位置, 在项目目录下，.idea 文件夹下，找到misc.xml (如果找不到.idea 请查看是否被折叠或被隐藏) 如果是 .ipr 模式创建的 就找到 项目名.ipr 

##### 3.2、单模块配置
```
<component name="yapi">
    <option name="projectToken">yapi 中项目token</option>
    <option name="projectId">yapi 中项目id</option>
    <option name="yapiUrl">http://127.0.0.1:3000</option>
    <option name="projectType">api</option>
    <option name="attachUploadUrl">http://localhost/fileupload</option>
    <option name="returnClass">com.project.Response(1.7.4 及之后才支持,按需配置)</option>
</component>
```

#### 3.3、多模块配置
1.7.4 及之后的配置

```
<component name="yapi">
    <option name="moduleList">moduleName1,moduleName2</option>
  </component>

  <component name="moduleName1">
      <option name="moduleName1.projectToken">yapi 中项目token</option>
      <option name="moduleName1.projectId">yapi 中项目id</option>
      <option name="moduleName1.yapiUrl">http://127.0.0.1:3000</option>
      <option name="moduleName1.projectType">api</option>
      <option name="moduleName1.attachUploadUrl">http://localhost/fileupload</option>
      <option name="moduleName1.returnClass">com.project.Response</option>
  </component>
```

1.7.4 之前的配置

```
<component name="yapi">
    <option name="moduleList">moduleName1,moduleName2</option>
  </component>

  <component name="moduleName1">
      <opt
ion name="moduleName1.Token">yapi 中项目token</option>
      <option name="moduleName1.Id">yapi 中项目id</option>
      <option name="moduleName1.Url">http://127.0.0.1:3000</option>
      <option name="moduleName1.Type">api</option>
      <option name="moduleName1.AttachUploadUrl">http://localhost/fileupload</option>
  </component>
```

#### 3.4、各个参数获取
- **token 获取方式：** 打开yapi ->具体项目->设置->token 配置
- **项目id 获取方式：** 点击项目，查看url 中project 后面的数字为项目id [http://127.0.0.1:3000/project/72/interface/api](http://127.0.0.1:3000/project/72/interface/api)
- **yapiUrl 获取方式：** 部署的yapi 地址
- **projectType 填写方式：** 根据你要上传的接口类型决定，如果为dubbo 接口就填dubbo ，如果是api 接口就填api
- **attachUploadUrl 填写方式:** 上传java 类zip 的url,如果要用请实现[http://localhost/fileupload](http://localhost/fileupload) 接口 接口请求参数为 file 文件类型。(可不填)
- **returnClass 填写方式** :统一返回对象的全路径（1.7.4及之后支持,之前版本可不配)
- **moduleList 获取方式：** 模块名称，用 "," 分割 ，不支持父节点和子模块名称一样的情况

#### 3.5、上传
- 如果是dubbo 项目，选中dubbo interface 文件中的一个方法（要选中方法名称），右击YapiUpload(alt+u 快捷键)
- 如果是api 项目，选中controller 类中的方法名称或类名（要选中方法名称，或类名，选中类名为当前类所有接口都上传），右击YapiUpload(alt+u快捷键)


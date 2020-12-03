## 一、在调用其他服务时，使用实体对象作为参数。按照以下配置。
#### 1、feign接口。
```
@RequestMapping(value = "/user", method = RequestMethod.POST, consumes = "application/json")
String getUserId(@RequestBody User user);
```
> - 1、**consumes**： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
> - 2、**produces**：  指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；
> - 3、或者 = MediaType.APPLICATION_JSON_VALUE
> - 4、正常来说，在FeignClient的接口中，也不需要在参数上注解@RequestBody ，只需要在实现类上添加@RequestBody 注解即可。

#### 2、其他服务实现。
```
@RequestMapping(value = "/user", method = RequestMethod.POST)
public String getUserId(@RequestBody User user){
    if("a".equals(user.getUserName())){
        return "1001";
    }
    return "1003";
}
```
> - 1、feign对应的服务实现上的参数@RequestBody 注解必不可少，否则接受到参数可能为null。

#### 二、如果以上依然实现不了实体对象的传输，可以增加以下方法修改。以下配置是参考网上别人所写，具体是谁我给忘记了，如果知道，请告诉我一声，我会备注的。
#### 1、修改调用@FeignClient 接口的pom文件。
```
feign:
  httpclient:
    enabled: true
```
	 
#### 2、修改pom文件。添加以下配置：
```
<!-- 使用Apache HttpClient替换Feign原生httpclient -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.3</version>
</dependency>
<dependency>
    <groupId>com.netflix.feign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>8.18.0</version>
</dependency>
```



#### 3、为什么要替换？主要是为了使用GET方法时也可以传入实体作为参数。否则，如果使用实体作为参数传值，默认会改为POST请求方式。看源码。
```
private synchronized OutputStream getOutputStream0() throws IOException {
  try {
      if(!this.doOutput) {
            throw new ProtocolException("cannot write to a URLConnection if doOutput=false - call setDoOutput(true)");
 } else {
      if(this.method.equals("GET")) {
           this.method = "POST";
 }
```
> Feign在默认情况下使用的是JDK原生的URLConnection发送HTTP请求，没有连接池，但是对每个地址会保持一个长连接，即利用HTTP的persistence connection 。我们可以用Apache的HTTP Client替换Feign原始的http client, 从而获取连接池、超时时间等与性能息息相关的控制能力。Spring Cloud从Brixtion.SR5版本开始支持这种替换，首先在项目中声明Apache HTTP Client和feign-httpclient依赖。

#### 4、看一个简单的示例。
```
// portal层
@RequestMapping(value = "/teacher/teacher", method = RequestMethod.GET)
public Teacher getTeacher(){
    Student student = new Student();
    student.setStudentId(1);
    student.setStudentName("王小二");
    student.setAge(12);

    return teacherFeign.getTeacher(student);
}


// feign api层
@FeignClient(value = "feign-server")
public interface TeacherFeign {

    @RequestMapping(value = "/teacher/teacher", method = RequestMethod.GET,consumes = MediaType.APPLICATION_JSON_VALUE)
    Teacher getTeacher( Student student);

}

// feign-server层
@RestController
public class TeacherRest implements TeacherFeign {
    @Override
    public Teacher getTeacher(@RequestBody Student student) {
        System.out.println(student.toString());
        return teacherList().get(0);
    }
}
```


## 三、使用表单模式传输数据，相当于GET方式直接接受实体参数。方式一，构建k1=v1&k2=v2
```
/**
* 在旧系统中，使用了原始的表单提交，并且这也是一个微服务
*/
@RequestMapping(value = "/patients/{id}", method = RequestMethod.PUT)
public APIResult UpdatePatientInfo(@PathVariable(value = "id") String id,PatientParameter patientParameter){...}
```

所以我的feign接口如下
#### 3-1、Feign接口
```
@FeignClient("xx-server")
public interface PatientLegacyAdapterFeign {

/**
 * 编辑客户
 * @param patientId
 * @param patientRequest 编辑客户请求数据
 * @return
 */
@RequestMapping(value = "/patients/{id}",method = RequestMethod.PUT)
Map<String,Object> editPatientInfo(@PathVariable("id")String patientId, PatientParameter patientParameter);
```
> 虽然这样可以请求过去，但是参数中的值并不会去过去，patientParameter结果为null。

&nbsp;
#### 3-2、 分析
```
Content-type=application/x-www-form-urlencoded
```
参数上不能增加@RequestBody的注解。

> - 1、因为使用的表单模式提交，所以这个提交方式是application/x-www-form-urlencoded。
> - 2、采用这种类型：提交到服务端实际上是一个MultiValueMap，如果value中的对象也是一个对象，那么在构建这个参数时就非常困难。采用key1=value1&key2=value2这种形式将所有参数拼接起来；value要进行编码，编码之后的对调试者不友好；value是复杂对象的情况更加糟糕，一般只能通过程序来序列化得到参数，想手写基本不可能。

#### 3-3、解决方案
```
@FeignClient("xx-server")
public interface OldPatientFeign {

/**
 * 编辑患者
 * @param id
 * @param patientParameter
 * @return
 */
@RequestMapping(value = "/patient/{id}", method = RequestMethod.PUT)
Map<String,Object> updatePatientInfo(@PathVariable(value = "id") String id, MultiValueMap patientParameter);
}
```

因为我们知道表单提交时，服务端是一个MultiValueMap，所以我们直接使用这个对象去传。

```
// 调用前使用MultiValueMap 。
LinkedMultiValueMap m = new LinkedMultiValueMap();
m.setAll( BeanUtils.describe(getPatientParameterByPatientRequest(patientRequest)));
return oldPatientFeign.updatePatientInfo(patientId,m);
```

这样就可以完美的使用表单格式提交了。

## 四、Feign实现Form表单提交,方式二
#### 4-1、添加依赖
```
<dependency>
  <groupId>io.github.openfeign.form</groupId>
  <artifactId>feign-form</artifactId>
  <version>3.2.2</version>
</dependency>

<dependency>
  <groupId>io.github.openfeign.form</groupId>
  <artifactId>feign-form-spring</artifactId>
  <version>3.2.2</version>
</dependency>
```

#### 4-2、Feign Client示例：
```
@FeignClient(name = "xxx", url = "http://www.itmuch.com/", configuration = TestFeignClient.FormSupportConfig.class)
public interface TestFeignClient {
    @PostMapping(value = "/test",            
        consumes = {MediaType.APPLICATION_FORM_URLENCODED_VALUE},            
        produces = {MediaType.APPLICATION_JSON_UTF8_VALUE}            )    
    void post(Map<String, ?> queryParam);    

    class FormSupportConfig {
        @Autowired        
        private ObjectFactory<HttpMessageConverters> messageConverters;        
            // new一个form编码器，实现支持form表单提交        
            @Bean        
            public Encoder feignFormEncoder() {            
                return new SpringFormEncoder(new SpringEncoder(messageConverters));        
            }        
            // 开启Feign的日志        
            @Bean        
            public Logger.Level logger() {
                        return Logger.Level.FULL;        
            }    
    }
}
```

#### 4-3、调用示例：
```
@GetMapping("/user/{id}")
public User findById(@PathVariable Long id) {
  HashMap<String, String> param = Maps.newHashMap();
  param.put("username","zhangsan");
  param.put("password","pwd");  
  this.testFeignClient.post(param);  
  return new User();
}
```
##### 1、pom中引入Hystrix依赖
```
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-hystrix</artifactId>  
</dependency>
```

##### 2、编写Controller类
```
@RestController
public class DataImportDoctorRest {

    /** 使用Feign来消费Restful服务 */
    @Autowired
    private DataImportingDoctorFeign dataImportingDoctorFeign;

    @RequestMapping(value = "/1.0/hopsitals/{hospitalId}/doctors", method = RequestMethod.POST)
    @HystrixCommand(fallbackMethod="findByIdFallback") // 使用HystrixCommand注解，在fallbackMethod属性中指定fallback的方法  
    public Map saveDoctor(@PathVariable(value = "hospitalId")String hospitalId, @RequestBody ThirdPartyDoctor doctor){
        Map<String,Object> map = dataImportingDoctorFeign.saveThirdPartyDoctorInfo(doctor);
        return map;	
    }

    /**
    * 覆写fallbackMethod中指定的方法，注意，此方法的返回值，参数必须与原方法一致  
    */
    public Map findByIdFallback(@PathVariable(value = "hospitalId")String hospitalId, @RequestBody ThirdPartyDoctor doctor){
        Map<String,Object> map = new HashMap<>(2);
        map.put("code",1001);
        map.put("message","服务出现问题");
        return map;
    }
}
```

##### 3、在启动类中添加Hystrix支持。
```
@EnableCircuitBreaker  
```

##### 4、配置文件修改
```
# 为了测试Hystrix的fallback效果，此处将超时时间设置成1毫秒  
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds: 1 
```
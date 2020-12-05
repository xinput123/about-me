postman断言是JavaScript语言编写的，在postman客户端的test区域编写即可，断言会在请求返回之后，运行，并根据断言的pass\fail情况体现在最终测试结果中。
具体断言如下所示：

#### 1、设置环境变量--Setting an environment variable
```
postman.setEnvironmentVariable("key", "value");
```

#### 2、获取环境变量--Setting an environment variable
```
postman.getEnvironmentVariable("key");
```

#### 3、设置全局变量--Set a global variable
```
postman.setGlobalVariable("key", "value");
```

#### 4、获取全局变量--Set a global variable
```
postman.getGlobalVariable("key");
```

#### 5、检查响应中包含string--Check if response body contains a string
```
tests["Body matches string"] = responseBody.has("string_you_want_to_search");
```

#### 6、转化XML格式的响应成JSON对象---Convert XML body to a JSON object
```
var jsonObject = xml2Json(responseBody);
```

#### 7、检查响应body中等于指定string--Check if response body is equal to a string
```
tests["Body is correct"] = responseBody === "response_body_string";
```

#### 8、检查JSON某字段值--Check for a JSON value
```
var data = JSON.parse(responseBody);
tests["Your test name"] = data.value === 100;
```

#### 9、检查Content-Type是否包含在header返回（大小写不敏感） --Content-Type is
```
present (Case-insensitive checking)
tests["Content-Type is present"] = postman.getResponseHeader("Content-Type");
// Note: the getResponseHeader() method returns the header value, if it exists.
```

#### 10、检查Content-Type是否包含在header返回（大小写敏感） --Content-Type is
```
present (Case-sensitive)
tests["Content-Type is present"] = responseHeaders.hasOwnProperty("ContentType");
```

#### 11、检查请求耗时时间小于200ms--Response time is less than 200ms
```
tests["Response time is less than 200ms"] = responseTime < 200;
```

#### 12、检查Status code为200--Status code is 200
```
tests["Status code is 200"] = responseCode.code === 200;
```

#### 13、检查Code name包含指定string
```
tests["Status code name has string"] = responseCode.name.has("Created");
```

##### 14、检查成功post的请求status code
```
tests["Successful POST request"] = responseCode.code === 201 || responseCode.code === 202;
```

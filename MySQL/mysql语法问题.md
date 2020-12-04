## 1、逻辑and
```
UPDATE `t_service_info` i 
SET i.`record_state` = 1 and update_time='2018-08-06 14:20:18' 
WHERE i.`team_id`= '5b0393847b75903a27b48688' 
AND i.`doctor_id` in('5b0393737b75903a27b486c4') 
AND i.`record_state` = 0;
```
> 今天同事问了我一条sql，其实是关于修改两个字段的值，但是不小心把逗号写成了 **and**，但是为什么这样也能执行呢？

>- 其实上面的语句确实是可以执行的，但是语句会被解析成 UPDATE `t_service_info` i SET i.`record_state` = ( 1 and update_time='2018-08-06 14:20:18' ),一个**布尔值**。
> 
>- **逻辑AND** 当所有操作数均为非零值、并且不为NULL时，计算所得结果为  1 ，当一个或多个操作数为0 时，所得结果为 0 ，其余情况返回值为 NULL 。
#### 1、NULL问题
oracle中where条件中如果写 where a.name=b.name 如果此时a.name为null,b.name为null  这个并不会返回true而是返回false
 
#### 2、存储过程函数的输入参数问题
oracle的存储过程中 输入参数最好在过程开始时全部赋值给全局变量，因为有些时候过程中取输入参数的值会出现莫名奇妙的问题，重新赋值用新变量就没有此问题。

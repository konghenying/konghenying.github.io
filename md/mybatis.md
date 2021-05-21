# mybatis 映射文件中if标签判断字符串相等的两种方式



```xml
<if test="fullid=='0'.toString()">
<if test = 'fullid== "0"'>
注意：不能使用
<if test="fullid=='0'">
and full_id=#{fullId}
</if>
```

mybatis会把0转换为数字.当full_id 存在层级的时候,即为字符串的时候会报NumberFormatException.


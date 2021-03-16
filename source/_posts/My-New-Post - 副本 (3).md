---
title: My New Post
date: 2020-10-30 15:04:17
tags:
---

特斯特3

<img src="/images/test7.png">



```java

    @RequestMapping("/chandao/getData")
    @ResponseBody
    public ResultDto getData(@RequestParam Map<String, Object> param){
        return chandaoService.getData(param);
    }
```

```xml

    <select id="getData" parameterType="map" resultType="map">
        select b.realname as username, min(a.date) date, #{endTime} as deadline, Round(sum(a.consumed),1) as submitWorkTime from  zt_user b
        LEFT JOIN zt_taskestimate a  on   b.account = a.account
        JOIN zt_task c on a.task = c.id
        where  b.dept  =1 
        <if test="startTime != null and startTime != ''">
            and a.date >= #{startTime}
        </if>
        <if test="endTime != null and endTime != ''">
            and  a.date  &lt;= #{endTime}
        </if>
        <if test="username != null and username != ''">
           and b.realname like concat(concat('%',#{username},'%'))
        </if>
        group by b.realname
    </select>
```


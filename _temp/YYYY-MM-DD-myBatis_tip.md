---
title: mybatis
tags: 태그1 태그2
---

### forEach

- 

~~~SQL
<select id="selectAnswer" resultType="String" parameterType="Map">
  select *
  from table
  where
        <foreach collection="questnArr" item="qestn" open="(" close=")" separator=" or ">
        QESTN = #{qestn}
      </foreach>
</select>
~~~

~~~SQL
SELECT *
FROM table
where
~~~

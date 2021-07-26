# Mysql+mybatis对为null的值赋默认值（ifnull）


<!--more-->

查询两张表 并且对为空的值做默认值 

```sql
select 
    u.User_pictureUrl,
    u.User_phone,u.ID as userId,
    u.user_nickname,
    IFNULL(f.userFriend_status,3)as status ,
    IFNULL(f.userFriend_comment,' ')as userFriend_comment,
    IFNULL(f.userFriend_address,' ')as userFriend_address
from 
    tab_userinfo u 
    LEFT JOIN tab_userfriend f ON  u.ID=f.userFriend_user_id  
where 
    u.ID=#{userId}
```

mybatis中插入

```sql
<insert id="insertForeach" parameterType="java.util.List" >
        insert into user_message
        (
        skip_id,user_id
        )
        values
        <foreach collection="list" item="userMessage" index="index" separator=",">
            (
            ifnull(#{userMessage.skipId},"0"),
            #{userMessage.userId}
            )
        </foreach>
</insert>
```

等同于：

```sql
<if test="userMessage.skipId !=null and userMessage.skipId!=''">
	#{userMessage.skipId},
</if>
<if test="userMessage.skipSonId !=null and userMessage.skipSonId!=''">
	#{userMessage.skipSonId},
</if>
<if test="userMessage.jumpType !=null and userMessage.jumpType!=''">
	#{userMessage.jumpType},
</if>
 
 
<if test="userMessage.skipId ==null or userMessage.skipId ==''">
	"0",
</if>
<if test="userMessage.skipSonId ==null or userMessage.skipSonId ==''">
	"0",
</if>
<if test="userMessage.jumpType ==null or userMessage.jumpType ==''">
	0,
</if>
```



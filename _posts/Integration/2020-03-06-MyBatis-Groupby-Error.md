---
layout: post
title:  "MyBatis에서 Group by 에러가 발생하는 경우 해결 방법"
date:   2020-03-06
categories: [Integration]
tags: [MyBatis, iBatis, SQL, PLSQL, Oracle, Error, Group by, Group, 에러, choose, when]
---


원래 시리즈로 글을 쓰려고 했는데, 작업하는 도중에 MyBatis 관련해서 막혔던 부분이 있어서 해결하게 되다 보니 팁으로 올리면 어떨까 싶어서 써봅니다.

MyBatis에서 XML로 SQL Query를 작성하다가 Group by에서 에러가 발생하는 경우가 있었습니다.

{% highlight JSON %}
{"timestamp":"2020-03-06T00:19:44.270+0000",
 "status":500,
 "error":"Internal Server Error",
 "message":"\r\n### Error querying database.  Cause: java.sql.SQLSyntaxErrorException: ORA-00979: not a GROUP BY expression
   ### The error may exist in file [xxxMapper.xml]\r\n
   ### The error may involve xxxMapper.prodbydate-Inline\r\n
   ### The error occurred while setting parameters\r\n
   ### SQL: select  trunc(time_stamp, ?)
   from sample_table 
   group by trunc(time_stamp, ?)\r\n
   ### Cause: java.sql.SQLSyntaxErrorException: ORA-00979: not a GROUP BY expression\n\n; bad SQL grammar []; 
   nested exception is java.sql.SQLSyntaxErrorException: ORA-00979: not a GROUP BY expression,
 "path":"/sample"}
{% endhighlight %}

원문 XML은 다음과 같습니다.

{% highlight XML %}
<select id="sampledate" resultType="map" parameterType="map">
    <![CDATA[select trunc(time_stamp,#{flag}) time_stamp
  from sample_table
 group by trunc(time_stamp,#{flag})]]>
</select>
{% endhighlight %}

보다시피, #{flag} 변수를 넣는 식으로 했지만 group by로 변수를 넣는 식으로 하면 ORA-00079: not a GROUP BY expression이 나옵니다.
그래서 인터넷을 다 찾아봤는데, 해결이 되지 않아서 제가 직접 찾아낸 방법으로 공유합니다.

방법은 <pre><choose><when></pre> 태그를 사용하는 것입니다.
물론 이 방법은 변수에 들어가는 값이 특정 값으로 제한되어 있을 때에만 가능하다는 단점이 있지만, 반대로 group by 구문에서 변수를 사용하는 경우 자체가
수 많은 유동 값이 들어가는 경우보다는 특정 값 중에 선택해서 넣는 경우가 많기 때문에 유용할 것으로 생각됩니다.

위 예제도 보면, trunc() 함수의 자르는 범위를 특정 값이 아니라 여러 개 값으로 정하는 것이기 때문에 이 경우에도 역시 choose 태그를 사용하는 것이 맞겠죠.

이를 응용한 예제는 다음과 같습니다.

{% highlight XML %}
<select id="sampledate" resultType="map" parameterType="map">
    select
    <choose>
    <when test="flag == 'ww'">
    trunc(jdoh.time_stamp, 'ww')
    </when>
    <when test="flag == 'mm'">
    trunc(jdoh.time_stamp, 'mm')
    </when>
    <when test="flag == 'yy'">
    trunc(jdoh.time_stamp, 'yy')
    </when>
    <otherwise>
    trunc(jdoh.time_stamp, 'dd')
    </otherwise>
    </choose>
    time stamp
    <![CDATA[from sample_table]]>
    <choose>
    <when test="flag == 'ww'">
    group by trunc(jdoh.time_stamp, 'ww')
    </when>
    <when test="flag == 'mm'">
    group by trunc(jdoh.time_stamp, 'mm')
    </when>
    <when test="flag == 'yy'">
    group by trunc(jdoh.time_stamp, 'yy')
    </when>
    <otherwise>
    group by trunc(jdoh.time_stamp, 'dd')
    </otherwise>
    </choose>
</select>
{% endhighlight %}

위와 같이 하면, 변수에 따른 분기를 choose-when에서 담당하고, 실제 들어가는 값은 고정된 값이 들어가기 때문에 올바르게 SQL Group by가 작동합니다.

단 주의해야 할 점은, Group by로 들어가는 구문만 choose-when을 할 것이 아니라, Query에서 보여주는 값 역시 choose-when을 해 줘야 합니다.
그렇게 하지 않으면 동일한 에러가 발생하기 때문에 이 점을 유의해야 합니다.

MyBatis로 Group by 구문에 변수가 들어가는 문장을 만들 때 참고하시기 바랍니다.
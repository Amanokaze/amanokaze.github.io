---
layout: post
title:  "Django Queryset에서 MySQL DB의 Group by SQL문을 표현하는 방법 "
date:   2019-11-11
categories: [Python]
tags: [Python, 파이썬, Django, Queryset, MySQL, Group, Groupby, SQL, Implementation]
---

이번 글에서는 Django 웹 페이지에서 MySQL DB를 사용하여 Group by 형태의 SQL문을 불러오는 방법에 대해서 다루겠습니다.

Django Framework로 웹페이지를 개발할 경우, DB Table의 값을 가져오기 위해서는 models.py 파일의 QuerySet을 사용하여 불러오는 것이 일반적인 방법이죠. 하지만 상황에 따라 Group by 형태의 SQL문을 불러올 때 QuerySet으로 어떻게 표현하면 좋을까요. 

Django 공식 문서에서는 Group by 와 관련하여 다음과 같이 내용을 다루고 있습니다.

<https://docs.djangoproject.com/en/2.2/ref/models/querysets/>

<b>An aggregate within a values() clause is applied before other arguments within the same values() clause. If you need to group by another value, add it to an earlier values() clause instead. For example:</b>

{% highlight Python %}
>>> from django.db.models import Count
>>> Blog.objects.values('entry__authors', entries=Count('entry'))
<QuerySet [{'entry__authors': 1, 'entries': 20}, {'entry__authors': 1, 'entries': 13}]>
>>> Blog.objects.values('entry__authors').annotate(entries=Count('entry'))
<QuerySet [{'entry__authors': 1, 'entries': 33}]>
{% endhighlight %}

요약하자면, values() 함수와 annotate() 함수의 Count() 함수를 사용하여 값을 정의하는 방법입니다.

참고로 덧붙여 말씀드리면, distinct()를 사용하는 방법도 있다고는 하지만 이 함수는 PostgreSQL에서만 작동하고 MySQL에서는 작동하지 않는다고 하니 그 부분이 아쉽긴 하지만 어쩔 수 없는 것 같습니다. 이번에 개발 중인 Django 3.0 버전에서도 MySQL에서 distinct나 group by는 별도로 적용하지 않는다고 하였습니다.


그러면 이제 한번 실제로 적용되는 예를 보도록 하겠습니다.

이런 테이블이 있다고 가정합시다.

#### Table: TB

| id | collection_id | category_seq |
|:---:|:---:|:---:|
| 1 | 1 | 1 |
| 2 | 1 | 2 |
| 3 | 1 | 3 |
| 4 | 1 | 4 |
| 5 | 2 | 1 |
| 6 | 2 | 2 |

그래서 위와 같이 value()와 annotate()를 사용합니다.

{% highlight Python %}
categories = TB.objects.values('category_seq').annotate(Count('category_seq'))
print(categories)
{% endhighlight %}

그랬더니 진짜로 결과가 아래와 같이 나왔습니다.

{% highlight Python %}
<QuerySet [{'category_seq': 1, 'category_seq__count': 4}, {'category_seq': 2, 'category_seq__count': 2}]>
{% endhighlight %}

이것이 과연 올바른 결과일까요? 결과적으로는 '예'. 그렇습니다.

진짜로 위와 같이 했더니 원하는 결과가 나왔습니다.

하지만 이 방법이 만능은 아닙니다. 사용할 때에 주의해야 할 점이 있습니다.

{% highlight Python %}
categories = TB.objects.order_by('category_seq').values('category_seq').annotate(Count('category_seq'))
print(categories)
{% endhighlight %}

이런 식으로, QuerySets 결과를 먼저 order_by 등과 같이 정렬을 한 후 values - annotate(Count)를 하면 어떻게 될까요.

{% highlight Python %}
<QuerySet [{'category_seq': 1}, {'category_seq': 1}, {'category_seq': 1}, {'category_seq': 1}, {'category_seq': 1}, {'category_seq': 1}, {'category_seq': 1}, {'category_seq': 1}, {'category_seq': 2}]>
{% endhighlight %}

여러분들께서는 절대로 원하는 결과를 쉽게 얻어내지 못할 것입니다. 분명히 한계가 있는 구문이죠.

하지만 위와 같은 식의 혼용만 주의한다면 MySQL에서 Group by를 통해서 그룹핑된 결과를 얻어낼 수는 있을 것이니 너무 큰 걱정은 하지 않으셔도 될 것입니다.

그리고 한 가지 더. 가장 쉬운 방법이 있긴 합니다. 바로 DB에서 View를 생성하는 방법입니다.

{% highlight SQL %}
create or replace view TBCategories_v as
    select category_seq
      from TB
     group by category_seq;
{% endhighlight %}

그리고 TBCategories_v 를 models.py에 추가시키는 것이죠.

굉장히 간단하죠?

쉽게 표현할 수 있는 API를 제공하지 않는다면 다른 방법을 통해서 사용하는 것도 하나의 방법이며, 정답은 한 개가 아니라 여러 개일 수 있습니다.

이 점을 참고하셔서 특정 값을 호출한다면 더욱 원활한 QuerySet 이용이 가능할 것입니다.

이상 글 마치겠습니다.
---
layout: post
title:  "Django Subquery와 파일, 이미지 필드 다루기"
date:   2019-11-12
categories: [Python]
tags: [Python, 파이썬, Django, Queryset, MySQL, SubQuery, 서브쿼리, FileField, ImageField, SQL]
---

Django QuerySet에서는 파일 및 이미지를 다루는 FileField와 ImageField가 있습니다. 이들 필드를 Subquery로 가져올 수 있을지 이 글에서 한번 살펴보겠습니다.

결론부터 말하자면, 안타깝게도 가져올 수 없습니다.왜 그런 지를 한 번 살펴보도록 하겠습니다.

일반적인 다른 테이블(Model)의 값을 가져오는 Django Subquery 사용법은 다음과 같습니다.

{% highlight Python %}
from django.db.models import OuterRef, Subquery

image_qs = ImageTable.objects.filter(data_id=OuterRef('pk')))
data = DataTable.objects.annotate(image=Subquery(image_qs.values('image')[:1]))
{% endhighlight %}

그런데 여기에서 중요한 것은 다름아닌 values() 메소드입니다.

image_qs 변수에서는 ImageTable의 값을 가져오는 구문이 포함되어 있으며, data 변수에서는 annotate()를 사용하여 Subquery로 image_qs의 values()를 사용합니다. 여기까지만 보면 굉장히 일반적인 내용이죠.

여기서 image 필드의 값이 IntegerField나 CharField 등이라면 가져오는 데 아무 문제가 없습니다. 하지만 FileField나 ImageField를 사용한다면 이야기가 달라집니다.

실제 values()에서는 FileField나 ImageField의 값을 가져오더라도 결국은 String으로 인식됩니다. 즉 SubQuery를 사용해서는 가져올 수 없겠죠.

그렇다면 어떻게 해결해야 할까요. 방법은 간단합니다.

<b>SubQuery가 포함된 View를 만드세요.</b>

현재 Django에서는 그것 외에는 별다른 표현방법이 없지 싶습니다.

{% highlight SQL %}
create view datatable_v as
select dt.*,
       (select image from imagetable where data_id = dt.id ) image
  from datatable dt;
{% endhighlight %}

그리고 View를 Model로 불러와서 쓰면, Image 테이블의 값을 아주 잘 불러올 수 있습니다.

QuerySet의 values() 메소드 사용에 따른 한계이니 참고하시기 바랍니다.

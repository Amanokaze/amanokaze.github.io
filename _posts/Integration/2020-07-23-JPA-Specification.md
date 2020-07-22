---
layout: post
title:  "JPA Specification"
date:   2020-07-23
image: 'img025_01.png'
categories: [Integration]
tags: [Java,Spring,Framework,JPA,Specification,Model,Repository]
---

바로 전날에는 Front-end 쪽인 Vue.js와 관련된 글이면, 이번에 쓸 글은 Back-end 쪽인 JPA Specification에 대해서 써 보겠습니다.

이번 프로젝트에서는 Spring Framework에 PostgreSQL을 연동하여 시스템을 개발을 하게 되었는데요.
PostgreSQL 연동을 위해서는 기본적인 데이터는 JPA를 사용하고, 그렇지 않고 Stored Function이나 복잡한 쿼리 등은 MyBatis에서 처리하도록 하였습니다.

하지만 만약에, 테이블 1개를 대상으로 조회를 하고 싶은데, 검색조건만 단순히 다양하다? 그럴 경우에는 JPA를 쓰는 것이 조금 더 낫다고 볼 수 있습니다.
그리고 이럴 때 사용하는 것이 JPA Specification으로도 볼 수 있겠죠.

#### 1. 준비

무엇부터 준비해야 할까요.

JPA를 조금이라도 다루셨던 분들은 알겠지만, Model하고 Repository는 기본으로 있어야 합니다.
어느 테이블의 데이터를(Model) 어떻게 갖고 오겠다(Repository)는 반드시 있어야겠죠?

그러면 위 2가지가 모두 있다는 가정 하에서 Specification을 사용하겠습니다.

Test.java - 모델 파일
TestRepository.java - Repository 파일


#### 2. Specification

Specification을 만드는 법은 Spring Boot 문서에도 명시되어 있고, 다른 곳에도 좋은 자료들이 많이 있습니다만,
그냥 간단하게 Specification 파일 하나 새로 만든다고 생각하면 간단합니다.

TestSpecification.java - Specification 파일

인터넷이나 다른데 뒤지고 그러면 Specification을 쓰라는 말은 있는데 도대체 어떻게 쓰라는건지 처음 하시는 분들께는 굉장히 생소할 수 있어요.
그래서 애를 많이 먹기도 하고요.

하지만 크게 걱정할 것 없습니다. 그냥 파일 새로 하나 만들면 됩니다.

{% highlight Java %}
public class TestSpecification {
	public static Specification<Test> searchWith(Map<String, Object> params) {
		return (Specification<Test>) ((root, query, builder) -> {
			List<Predicate> predicate = getPredicateWithKeyword(params, root, builder);
			return builder.and(predicate.toArray(new Predicate[0]));
		});
	}
	
	public static List<Predicate> getPredicateWithKeyword(Map<String, Object> params, Root<Test> root, CriteriaBuilder builder) {
		List<Predicate> predicate = new ArrayList<>();

		if (params.get("key1") != null && !params.get("key1").equals("")) {
			predicate.add(builder.like(root.get("key1"), "%"+(String)params.get("key1")+"%"));
		}

		if (params.get("key1") != null && !params.get("key1").equals("")) {
			predicate.add(builder.like(root.get("key2"), "%"+(String)params.get("key2")+"%"));
		}

		return predicate;
	}
}
{% endhighlight %}

저 코드를 일일히 설명해야 할 필요는 없습니다.
어차피 저 코드는 그대로 복사해서 쓰시고, 모델명만 참조 잘 해주시고요. (실제 쓰면 Test가 아니라 다른 Model Class명이 되겠죠)

중요한 것은 getPredicateWithKeyword() 메소드입니다.
여기에서 여러 가지 조건 선언을 할 수 있습니다.

조건을 추가하는 방법은, predicate.add() 함수를 사용하며, 파라미터로는 builder에서 사용하는 함수를 선언합니다..

builder에서 사용하는 함수는 일반적으로 인자가 2개로, 다음과 같습니다.

* 첫 번째는 Model에서 사용하고 있는 Key 값
* 두 번째는 실제 조건문에 들어가기 위한 Parameter값

이렇게 됩니다.

그렇다면 유사 조건(like), 동일 조건(equal), 같거나 크거나(gte) 등등 조건 연산자에 해당되는 부분은 무엇일까요.
바로 위의 builder.like() 메소드가 정답이 됩니다.

여기에서는 like() 메소드를 선언했기 때문에 유사 조건이겠죠?

builder에서는 거의 모든 유형의 비교 연산자를 메소드로 구현하였기 때문에 맞춰서 사용하면 됩니다.

그러면 이제 Specification 만든걸 써먹어야겠죠?
먼저 Repository로 갑니다.


{% highlight Java %}
@Repository
public interface TestRepository extends JpaRepository<Test, Long>, JpaSpecificationExecutor<Test> {
	Page<Test> findAll(Specification<Test> spec, Pageable pageable);
}
{% endhighlight %}

일반적으로는 findAll() 함수를 오버라이딩하여 사용합니다. 왜냐하면 먼저 모든 조건에서 검색하고 그 속에서 Specification을 사용하기 때문입니다.
그러므로 함수 파라미터의 인자는 첫 번째로 Specification으로 선언한 객체가 되겠고, 두 번째부터는 사실상 추가 옵션이므로 생략합니다.

그 다음으로는 서비스에서 TestRepository.findAll()을 사용해서 검색하면 되겠죠?
JPA Specification을 선언한 후에 자기 입맛에 필요한 조건을 추가해주시고, Repository에 등록해서 가져다 쓰면 끕납니다.

이상입니다.
사실 Specification이 따지고 보면 개념잡기가 어려운데, 개녑만 잡히면 쉽게 사용할 수 있으니 개발에 참고 바라겠습니다.


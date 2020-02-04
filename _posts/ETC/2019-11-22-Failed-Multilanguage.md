---
layout: post
title:  "Jekyll-Polyglot 다국어 지원 & Github Pages 호환 문제"
date:   2019-11-22
image: 'assets/img/img016_01.png'
categories: [ETC]
tags: [Jekyll, Plugin, Github, Pages, 호환, 문제, 제킬, 플러그인, 깃헙, 페이지, 블로그, Polyglot, Jekyll-Polyglot, 다국어, 지원, Multilanguage, Support]
---

최근 3일간 블로그에 다국어화 및 검색엔진 검색 개선을 위한 플러그인을 설치하려고 여러 가지 시도를 했었습니다.

이와 관련하여 찾아보고 알아본 자료만 수십 곳은 되었고, Github Pages, 즉 이 블로그에 적용하려고 수많은 노력을 했지만, 결과는 실패로 돌아갔네요.

해당 플러그인은 다음과 같습니다.

{% highlight Shell %}
jekyll-polyglot # 다국어 지원 플러그인
jekyll-seo-tag  # 검색 개선
{% endhighlight %}

jekyll-seo-tag 플러그인은 한번에 적용이 제대로 잘 됐습니다. 이건 아무 문제가 없었습니다.

그러나 jekyll-polyglot 플러그인이 문제였습니다. 여러 자료를 찾아보려고 했지만 완전히 실패로 돌아갔죠.

대표적으로, jekyll-polyglot에서 지원하는 Liquid Tag인 &#123;% I18n-Headers %&#123; 를 삽입하였을 때, Localhost에서는 아주 잘 불러왔지만 Github Pages로 Push했을 때 웹사이트에서 보여지는 결과는 처참했습니다.

![Result]({{ '/assets/img/img016_01.JPG' | prepend: site.baseurl }})

I18n_Headers Liquid Tag가 Github Page에서 인식되지 않는다는 내용입니다. 즉 jekyll-polyglot 플러그인이 적용되지 않았다는 뜻입니다.

왜 이런 일이 생긴걸까요? 이유는 의외로 간단했습니다.

Jekyll 공식 블로그에 가면, 이런 부분이 있습니다.

![Document]({{ '/assets/img/img016_02.jpg' | prepend: site.baseurl }})

Whitelisted Plugins를 제외한 나머지 플러그인은 Github Page에서 작동하지 않는다고. 실제 확인해 보니 jekyll-seo-tag는 Whitelisted Plugins에 있는 반면, jekyll-polyglot은 없었습니다. 그렇다면 _plugins 디렉토리에 jekyll-polyglot.rb 파일을 넣는다고 해결이 되었을까요? 천만의 말씀입니다. 해결되기는 커녕 로컬에서조차 동작하지 않았습니다.

결국 실패한거죠.

Jekyll 3.5 버전 이하에서는 아마 잘 작동할 지도 모릅니다. 하지만 다국어 지원 때문에 Jekyll을 다운그레이드한다? 차라리 그냥 포기하고 말겠습니다. 검색엔진 플러그인 적용으로 그냥 감사하렵니다.

결과적으로 요약하자면, Jekyll 높은 버전(3.8 이상)에서는 Whitelisted Plugin 외 다른 플러그인은 Github Pages에 적용되지 않으며, 적용되더라도 여러 가지 문제가 있을 것으로 생각합니다. 괜히 이거저거 해보려고 시간낭비할 시간에 포기하는 것이 차라리 편할 수도 있습니다.

다만 3일동안 삽질한 시간동안 아무것도 얻은 것이 없었냐. 그것은 다행히도 아닙니다. 평소에 쓸 줄도 몰랐던 Ruby Gem 사용 및 적용 방식에 대해서 충분한 이해가 있었고, WSL을 설치해서 Ruby Gem을 가동시키는 부분까지 여러 가지를 많이 배웠던 것 같습니다.

나중에 다 쓸 곳이 있겟죠.

제가 이 글을 쓰는 이유는 간단합니다. 인터넷 블로그나 관련 자료에 있는 모든 글 내용을 종합하고 실제로 따라하더라도 안될 것은 결국 안된다. 이것이 결론입니다. 혹시나 다른 피드백이 있지 않을까 검색을 좀 못한 것일까라고 생각하는 분들도 계시겠지만, 3일동안 수십개(사실 100개가 다 되어가네요)의 웹페이지를 검색해도 결과가 나오지 않는 것은, 공식홈페이지나 그런 곳에 물어본다고 마땅한 해답을 제공하지는 않을 것이라는 것이죠. 즉 저처럼 무식하게 밀어붙이는 분들이 더이상 나오지 않고 포기할땐 포기하라는 뜻으로 쓴 글입니다.

Jekyll 다국어 지원 플러그인을 구현하고자 하시는분들께 참고가 되었으면 좋겠습니다.

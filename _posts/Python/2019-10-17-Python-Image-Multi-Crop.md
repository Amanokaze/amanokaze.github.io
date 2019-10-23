---
layout: post
title:  "이미지 일괄 자르기(Crop)"
date:   2019-10-17
categories: [Python]
tags: [Python, 파이썬, Image, 이미지, Crop, 크롭, 일괄, Multi]
---

여러 개의 같은 사이즈를 가진 이미지를 특정 영역을 한꺼번에 자르고 싶을 때 어떻게 잘라야 할까 고민하다가 글을 쓰게 되었습니다.

 

인터넷에서 찾으면 JPEGCrop 인가 하는 프로그램도 있고, 여러 개의 이미지를 일괄로 잘라주는 사이트도 있습니다. 하지만 현실은 녹록치 않더라고요.

 

JPEGCrop인가? 하는 프로그램은 옛날에 나온건데, 높이(H) x 너비(W) 비율이 안바뀌게 설정되어 있고요. 사이트는 들어가보니까 여러개를 하려면 프리미엄 가입해야 하고, 한개씩만 무료로 자르기가 되더라고요.

 

근데 한개씩 이미지를 자를거면 그냥 그림판쓰지 뭣하러 웹에서 할까요. 아무의미 없겠죠. 그래서 기왕 이럴거 그냥 파이썬으로 만들자 싶었습니다.

 

1. 먼저 여러 장의 이미지를 특정 폴더를 만들어서 몰아넣습니다.
2. 이미지 파일을 다 읽습니다.
3. 하나씩 읽어서 같은 크기로 자릅니다.
4. 자른 이미지를 저장합니다.

 

이렇게 하면 되곘죠?

 
{% highlight Python %}
import os
from PIL import Image

for root, dirs, files in os.walk('./'):
    for idx, file in enumerate(files):
        fname, ext = os.path.splitext(file)
        if ext in ['.jpg','.png','.gif']:
            im = Image.open(file)
            width, height = im.size
            
            crop_image = im.crop((500, 0, 875, height))
            crop_image.save('Img' + str(idx) + '.jpg')            
{% endhighlight %}

이게 다입니다. 별거 없어요.

 

첫번째 for문을 사용해서 폴더 내의 모든 파일을 가져옵니다.두번째 for문을 사용해서 파일 하나하나씩 읽고요. if 문을 사용해서 확장자 확인한 다음에, 이미지 열고 CROP 한 다음에 저장하면 됩니다.

 

여기서 가장 중요한 코드는 당연히 이 부분입니다.

 
{% highlight Python %}
crop_image = im.crop((500, 0, 875, height))
#crop((left, top, right, bottom))
{% endhighlight %}

좌우상하 위치 지정해주시면 그 위치 내에 있는 이미지로 CROP 되므로 참고해주시고, For 문 쓰셔서 한꺼번에 다 자르면 됩니다.

 

괜히 엄한 프로그램이나 웹에 있는거 쓰지 마시고 Python 코드 만들어서 쉽게 잘라주세요. 절차도 간단하고 위와 같이 코드도 몇 줄 안되니까 어렵지 않습니다.

 

생성 결과는 아래와 같습니다.

 
![Image Crop Result](/assets/img/img007_01.png)


enumerate() 함수를 사용해서 for 문의 Index를 추출해서 파일 이름으로 붙였으니 구분도 쉽겠죠?

 

**편의 기능을 쓰고 싶을 때 이거저거 검색해서 나오는 프로그램 괜히 쓰지 마시고 Python 사용해서 간단하게 몇 줄로 짜시면 됩니다.**

 

감사합니다.
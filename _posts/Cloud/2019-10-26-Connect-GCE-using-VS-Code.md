---
layout: post
title:  "VS Code를 사용하여 Google Compute Engine에 연결하기"
date:   2019-10-17
image: '/assets/img/img014_01.png'
categories: [Cloud]
tags: [Google, Cloud, SDK, 구글, 클라우드, VS, Visual, Studio, Code, 비주얼, 스튜디오, 코드, GCE, Compute, Engine, VM, Instance, 인스턴스, 연결, 접속]
---

 
이번 글은 Visual Studio Code(VS Code)를 사용하여 Google Compute Engine의 VM Instance에 연결하는 방법에 대해서 다루도록 하겠습니다.

블로그를 Tistory에서 Github로 이전한 후 쓰는 첫 번째 글이 되겠네요. 그 동안 썼던 글은 Tistory 블로그의 글을 Github로 이관했던 글인 반면, 지금 쓰는 글부터는 순수하게 Github에서 작성한 글입니다. 첫 번째 글인 만큼 코멘트를 달았으니 가벼운 참고 바라겠습니다.

VS Code를 사용해서 Google Compute Engine에 접속하기 위해서는 몇 가지 전제 조건이 있습니다.

* VS Code가 설치되어 있어야 할 것
* Google Cloud에 Compute Engine의 VM Instance가 있어야 할 것
* Puttygen이 설치되어 있어야 할 것
* Windows 버전은 Windows 10이어야 할 것

만약 셋 중에 하나라도 없으면, 세 가지를 앞서 진행해 주시기 바랍니다. 혹시 Windows 버전이 Windows 10이 아닐 경우에는 아래 설명에 따라 진행하면서 유사한 부분은 맞춰서 진행해주시기 바랍니다.

그럼 시작하겠습니다.
<br/><br/>

### 1. SSH Key File 생성

Google Compute Engine(이하 GCE)를 외부에서 접속하기 위해서는 먼저 SSH Key 파일이 있어야 합니다. 

AWS에서는 Key 파일을 Management Console에서 생성한 후 다운로드를 받는 형식이라면, GCE는 반대로 사용자가 임의의 Key 파일을 생성한 후 GCE에 등록하는 절차로 구성되어 있습니다. 그러므로 GCE 접속을 위해서 Key 파일을 먼저 생성하도록 합니다. VS Code를 사용한다면 운영체제는 아마 Windows일 것입니다. 그러므로 운영체제는 Windows인 것을 전제로 하겠습니다.

Windows에서 Key 파일을 생성하는 가장 일반적인 방법은 Puttygen을 사용하는 방법입니다. Puttygen에서 키 파일을 생성하는 방법은 다음과 같습니다.

![Execute Puttygen]({{ '/assets/img/img014_01.png' | prepend: site.baseurl }})

위 사진과 같이 Key - Generate Key Pair를 선택합니다.

![Generate Key Pair]({{ '/assets/img/img014_02.png' | prepend: site.baseurl }})

여기서 멍때리고 가만히 있지 마세요. 가만히 있으면 아무 일도 일어나지 않습니다. 마우스 커서를 빈 공간에 계속 움직여주셔야 진행되니, 게이지가 모두 찰 때까지 마우스를 움직여주시기 바랍니다.

![Input a key information]({{ '/assets/img/img014_03.png' | prepend: site.baseurl }})

생성이 완료되면 다음과 같이 나옵니다. 여기서 다음과 같이 입력합니다.

* Key Comment: 계정명@gmail.com 으로 반드시 입력해주시기 바랍니다. 계정명은 GCE에 접속하는 계정 이름을 뜻합니다.
* Key passphrase: 비밀번호입니다. 입력 안해도 상관없습니다. 물론 예제에서는 입력하겠습니다.

이제 다음 절차입니다. 매우 중요합니다.

![Export OpenSSH Key]({{ '/assets/img/img014_04.png' | prepend: site.baseurl }})

Conversions - Export OpenSSH Key 메뉴로 이동해주시기 바랍니다. 

일반적으로는 Save private key를 눌러서 Private Key를 저장하지만, 여기서 생성되는 ppk 파일로 Putty 등의 터미널 프로그램 접속은 가능하지만, VS Code에서는 사용할 수 없습니다. 그러므로 OpenSSH Key 형태로 저장해야 합니다.

![Save OpenSSH Key]({{ '/assets/img/img014_05.png' | prepend: site.baseurl }})

이름은 임의로 입력해 주시고, 확장자는 없어도 됩니다. 물론 붙이고 싶으면 붙이셔도 됩니다.

다음은 Private Key 저장입니다. Private Key도 같이 저장하는 것이 좋은데, 이는 VS Code만 순수하게 사용할 경우면 상관이 없으나, Putty 등 터미널 프로그램 접속도 필요할 수 있으므로 터미널 접속을 위한 Private Key도 같이 저장합니다. 여기서 저장하는 파일은 자동으로 PPK 확장자로 저장되니 참고 바랍니다.

![Save Private Key]({{ '/assets/img/img014_06.png' | prepend: site.baseurl }})

다음은 Public Key 부분입니다. 

위의 본문 영역을 긁어서 Ctrl + C 를 눌러서 복사해야합니다. 이 부분은 GCE에 Key를 등록할 때 사용되는 부분이므로, 미리 참고 바랍니다.

![Copy Public Key]({{ '/assets/img/img014_07.png' | prepend: site.baseurl }})

이제 Puttygen에서 Key를 생성하는 작업은 모두 끝났습니다. GCE에 등록하기 위하여 이동하겠습니다.


### 2. GCE Key 등록

바로 위에서는 Puttygen에서 생성한 키의 Public Key를 Ctrl + C 해서 복사했습니다. 이제 이 값을 GCE에 등록하겠습니다. 

Google Cloud Console에 접속하신 후, GCE를 선택한 후 세부 메뉴를 보면 Metadata 항목이 있습니다.

![Metadata Menu]({{ '/assets/img/img014_08.png' | prepend: site.baseurl }})

다음으로 SSH Keys 탭을 선택합니다.

정확한 위치는 다음과 같습니다.

<b>Compute Engine - Metadata - SSH Keys</b>

![SSH Keys]({{ '/assets/img/img014_09.png' | prepend: site.baseurl }})

위 사진에서 'Edit'를 눌러줍니다.

화면 이동이 완료된 후, 하단의 'Add Items'를 누르면 빈 칸이 나옵니다.

![Add Items]({{ '/assets/img/img014_09.png' | prepend: site.baseurl }})

여기에서 위에서 복사했던 Public Key를 붙여넣기 합니다.

![Paste Public Key]({{ '/assets/img/img014_11.png' | prepend: site.baseurl }})

그리고 가장 하단의 'Save'를 눌러서 저장합니다.

여기까지 하면 GCE에 Key 등록도 모두 마치게 됩니다. 이제 VS Code에서 연결하기 위한 SSH 설정을 하겠습니다.


### 3. VS Code에서 GCE 연결을 위한 SSH 설정

VS Code에서 GCE를 연결하려면 SSH를 설치해야 합니다. Marketplace를 가서 설치합니다. Marketplace는 왼쪽 상단 아이콘 중 다섯번째 작은네모 4개 있는걸 누르면 됩니다.

![VS Code Marketplace]({{ '/assets/img/img014_12.png' | prepend: site.baseurl }})

SSH를 설치하려면 위 사진과 같이 'remote'로 검색합니다. 그러면 위와 같은 결과가 나옵니다.

여기서 설치해야 할 게 많은데, 고민하지 마세요. 그냥 위에 표시된 '
<span style='color: #F00'>Remote Development</span>
' 를 설치해주세요. 이것 설치하면 위에 표시된 WSL, SSH, Containers 등 전부 다 자동으로 설치됩니다. 그러므로 'Remote Development'만 설치하면 됩니다.

설치가 완료되면 왼쪽 상단 네 번째 아이콘인 모니터 아이콘(Remote Explorer)을 눌러줍시다.

![VS Code Remote Explorer]({{ '/assets/img/img014_13.png' | prepend: site.baseurl }})

Remote Explorer에 들어가면 아무것도 없을 것입니다. 이제 만들어야 되겠죠? 우측 상단 'Add New'라고 써진 '+' 기호를 눌러줍니다.

![Enter SSH Connection Command]({{ '/assets/img/img014_14.png' | prepend: site.baseurl }})

누르면 가운데 상단에 위와 같이 'Enter SSH Connection Command'라는 부분이 나옵니다. 이제 여기에서 ssh 커맨드를 입력합니다. 입력 방식은 다음과 같습니다.

{% highlight Shell %}
ssh -i [OpenSSH Key File 경로] [사용자@GCE 외부 IP주소]
예제: ssh -i "C:\key\gkey" aaa@12.34.56.78
{% endhighlight %}

입력 후 엔터를 치면 다음과 같은 화면이 나옵니다.

![Select SSH Configuration File]({{ '/assets/img/img014_15.png' | prepend: site.baseurl }})

일반적으로는 첫 번째 경로인 사용자/.ssh/config 이 파일에 저장합니다. 그러므로 첫 번째를 선택합니다.

![Host Added]({{ '/assets/img/img014_16.png' | prepend: site.baseurl }})

선택하면 우측 하단에 위와 같은 메시지가 나타납니다. 'Open Config'를 눌러서 한 번 확인합니다. 확인했을 때 결과는 아래와 같습니다.

{% highlight Shell %}
Host 12.34.56.78
  HostName 12.34.56.78
  IdentityFile C:\key\gkey
  User aaa
{% endhighlight %}

여기까지 진행하면 VS Code에서 GCE 접속을 위한 SSH 설정을 마쳤습니다. 하지만 여기서 그대로 접속하면 접속이 되지 않으니 한 가지 더 설정을 해 주셔야 합니다.


### 4. key 파일 권한 설정

앞서 위에서 생성했던 Key 파일은 일반적으로 모든 유저가 읽고 사용할 수 있습니다. 하지만 GCE에서 VS Code를 사용해서 접속할 때에는 권한 설정이 되어있지 않으면 접속을 할 수 없습니다. Linux에서는 chmod 600 등을 통해서 변경할 수 있지만, 지금 사용하는 운영체제는 Windows이므로, 특별한 권한 관리를 해야 합니다.

사실 Key 파일은 위에서 생성했기 때문에 위에서 설정해야 하지 않을까 생각할 수 있습니다만, Public Key를 복사하고 GCE에 등록하는 절차는 연속적으로 이루어져야 하는 것이 보다 자연스럽다고 생각했습니다. 이에 따라 의식의 흐름에 맞추어서 Key 파일 권한 설정은 맨 뒤로 이동하였으니 참고 바랍니다.

Key 파일 권한 설정 방법은 다음과 같습니다. 탐색기에서 해당 파일을 찾은 후 우클릭 - 속성(R)을 눌러주시고, '보안' 탭으로 이동합니다.

![File Authentication]({{ '/assets/img/img014_17.png' | prepend: site.baseurl }})

여러 유저의 권한이 나올 것입니다. 아래의 '고급'으로 이동합니다.

![No Inheritance]({{ '/assets/img/img014_18.png' | prepend: site.baseurl }})

하단의 '상속 사용 안합'을 눌러줍니다. 이걸 눌러야 권한 변경이 가능합니다. 누르면 다음과 같은 창이 나옵니다.

![No Inheritance 2]({{ '/assets/img/img014_19.png' | prepend: site.baseurl }})

첫 번째 것을 선택합니다. 그런 다음 '확인'을 눌러줍니다.

다음에 '편집'으로 들어갑니다.

![Edit Authentication]({{ '/assets/img/img014_20.png' | prepend: site.baseurl }})

편집을 들어가면 권한이 나옵니다. '제거'를 4번 연속 눌러서 다 지워줍니다.

![Delete Users]({{ '/assets/img/img014_21.png' | prepend: site.baseurl }})

모두 지웠으면 추가를 합니다. 

![Add Users]({{ '/assets/img/img014_22.png' | prepend: site.baseurl }})

추가할 때에는 현재 접속 중인 Windows의 계정명을 입력해줍니다. 그리고 '이름 확인'을 눌러줍니다.

![Add a current User]({{ '/assets/img/img014_23.png' | prepend: site.baseurl }})

그러면 개체 이름이 아래와 같이 변경됩니다. 변경된 것 확인 후 '확인'을 눌러줍니다.

![Add a current User 2]({{ '/assets/img/img014_24.png' | prepend: site.baseurl }})

이제 모든 권한을 허용한 후, 확인을 눌러줍니다. 현재 사용자만 권한이 있으므로 VS Code에서도 정상적인 권한을 가진 키 파일로 인식됩니다.

![Set an Authority]({{ '/assets/img/img014_25.png' | prepend: site.baseurl }})

계속 확인을 눌러서 모두 마쳐줍니다.


### 5. VS Code를 사용하여 GCE 접속

이제 모든 준비를 다 마쳤습니다. VS Code에서 바로 접속하겠습니다.

![Connect GCE]({{ '/assets/img/img014_26.png' | prepend: site.baseurl }})

접속하면 가운데 상단과 같이 passphrase를 입력하라고 나옵니다. 비밀번호를 입력합니다. 비밀번호는 계정 비밀번호가 아니라, Puttygen에서 생성했던 Passphrase 비밀번호입니다. 

접속이 완료되면 아무런 메시지가 나오지 않고, 반대로 접속에 실패했을 경우에 메시지가 나오니 참고 바라고요. 아무 메시지도 안나온다면 정상 접속된 것입니다. 이제 탐색기를 가보도록 하겠습니다. 탐색기는 첫 번째 아이콘입니다.

![Connected GCE]({{ '/assets/img/img014_27.png' | prepend: site.baseurl }})

위 사진과 같이 나오면 정상 연결된 것이며, '폴더 열기'를 눌러서 파일을 탐색합니다. 탐색하면 passphrase를 입력하라는 창이 다시 나오는데, 다시 입력합니다.

![Connected GCE 2]({{ '/assets/img/img014_28.png' | prepend: site.baseurl }})

위와 같이 나오면 VS Code에서의 GCE 연결이 모두 완료되었습니다.

생각보다 거쳐야 할 절차가 많지만, 순서대로 하나하나씩 하면 쉽게 연결할 수 있으므로 참고하시기 바랍니다.


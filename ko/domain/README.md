# 도메인 만들기

PythonAnywhere 는 무료 도메인을 제공하지만 블로그 주소 끝에 있는 ".pythonanywhere.com" 는 원하지 않을 수도 있을 거예요. 아마도 이런 이름을 원할지도 몰라요. "www.infinite-kitten-pictures.org" 혹은 "www.3d-printed-steam-engine-parts.com" 혹은 "www.antique-buttons.com" 혹은 "www.mutant-unicornz.net" 무엇이든 이런 이름으로요.

여기에서는 PythonAnywhere 에 있는 웹앱을 도메인에 연결하는 방법에 관해 설명해 볼거에요. 어쨌든, 대부분 도메인은 비용이 들어가요. PythonAnywhere 도 도메인명에 대한 사용 비용을 부과할 거에요. --여러분이 원할 때 큰 비용이 아닌 선에서요.


## 도메인은 어디에서 등록할까요?

일반적으로 도메인 비용은 연간 약 15달러 정도가 들어요. 공급자에 따라 좀 더 저렴하거나 비싼 옵션도 있어요. 도메인을 구할 수 있는 회사는 많이 있어요. 간단한 [Google 검색](https://www.google.com/search?q=register%20domain)을 통해 수백개의 옵션을 얻을 수 있어요.

우리가 가장 선호하는 곳은 [I want my name](https://iwantmyname.com/) 라는 곳이에요. 여기는 "고통 없는 도메인 관리"라는 문구로 홍보하는 데 정말 고통스럽지 않아요.

[dot.tk](http://www.dot.tk)은 역시나 도메인을 무료로 얻을 수 있는 곳이에요. 비용을 내고 살 수도 있지만, 이 정도면 저렴한 편이에요. -- 만약 전문적인 비즈니스를 하려고 한다면 아마도 `.com` 으로 끝나는 "적정한" 도메인을 구입해야 할거에요. 

## PythonAnywhere 에서 도메인을 가리키는 방법

*iwantmyname.com* 사이트를 통해 도메인을 구했다면 `Domains` 메뉴를 클릭하고 새로 구입한 도메인을 선택하세요. 그런 다음 `manage DNS records` 링크를 클릭하세요.:

![](images/4.png)

이제, 이 폼을 찾아야 해요.:

![](images/5.png)

그리고 아래 정보를 채워야 해요.:
- Hostname: www
- Type: CNAME
- Value: PythonAnywhere의 도메인 (예를 들어, djangogirls.pythonanywhere.com)
- TTL: 60

![](images/6.png)

'Add' 버튼을 클릭하고 변경사항을 저장해 주세요.

> **Note** 다른 도메인 업체를 사용한다면 DNS / CNAME 설정을 찾는데 필요한 UI는 다르지만, 우리가 해야 할 일은 같아요. `yourusername.pythonanywhere.com`이 새로운 도메인을 가리키도록 CNAME 을 설정하는 거예요.

도메인이 동작하려면 몇 분 정도 걸릴 수도 있어요. 인내심을 갖고 기다려봐요!

## PythonAnywhere 의 웹 앱을 통해 도메인 구성을 설정

PythonAnywhere 에도 커스텀 도메인을 사용할 거라고 알려줘야 해요.

[PythonAnywhere Accounts page](https://www.pythonanywhere.com/account/) 로 가서 계정을 업그레이드하세요. 가장 저렴한 옵션("Hacker")으로 시작해도 괜찮아요. 나중에 블로그가 유명해 지고 수백만건의 조회수 있다면 언제든 업그레이드 할 수 있어요.

다음으로 [Web tab](https://www.pythonanywhere.com/web_app_setup/) 으로 가서 다음 2가지를 메모해 주세요.:

* **가상 환경 경로**를 적어두고 안전한 곳에 보관해 주세요.
* **wsgi 설정 파일**을 클릭해서 내용을 복사해서 붙여넣기 해두고 안전한 곳에 보관해 주세요.

그런 다음, 이전 웹앱을 **삭제**해 주세요. 걱정하지 마세요. 이 과정은 어떠한 코드도 삭제하지 않고 *yourusername.pythonanywhere.com* 도메인을 끄기만 할거에요. 다음으로 새로운 웹 앱으로 가서 다음 단계와 같이 해주세요.:

* 새 도메인 이름을 입력하세요.
* "manual configuration"을 선택하세요.
* Python 3.4를 선택하세요.
* 이제 되었어요!

이제 웹탭으로 돌아갈 때예요.

* 앞에서 저장해 두었던 가상 환경 경로에 붙여넣기를 하세요.
* wsgi 설정 파일을 클릭하고 기존 설정 파일의 내용을 붙여넣기를 하세요.

웹 앱을 다시 재실행(새로 고침)해 보세요. 새로운 도메인으로 사이트를 찾아볼 수 있을 거예요!

만약 문제가 있다면 PythonAnywhere의 "Send feedback" 링크를 통해 문의해 보세요. 친절한 관리자가 시간을 내어 도움을 줄 거예요.


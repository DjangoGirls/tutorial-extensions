# 헤로쿠(Heroku)에 배포하기

항상 서로 다른 두 가지 배포 옵션을 사용하는 것이 좋아요. 헤로쿠(Heroku)와 PythonAnywhere에 사이트를 배포해 볼까요?

[헤로쿠](http://heroku.com/) 역시 방문자 수가 적은 소규모 애플리케이션에는 무료에요. 하지만 배포과정이 다소 까다로워요.

우리는 이 [튜토리얼](https://devcenter.heroku.com/articles/getting-started-with-django)을 따라 해볼 거에요. https://devcenter.heroku.com/articles/getting-started-with-django 다소 복잡해 보일지 몰라도 붙여넣기를 사용해서 쉽게 해볼 거에요.


## 우선 `requirements.txt` 파일이 필요해요.

`requirements.txt` 파일을 만들지 않았다면 이번에 만들어볼 거에요. 이 파일은 우리 서버에 어떤 파이썬 패키지들을 설치해야 하는지 알려 줍니다. 

하지만 첫째로, 헤로쿠를 사용하려면 몇 가지 새로운 패키지를 설치 해야 해요. `virtualenv`를 활성화시킨 상태에서 콘솔로 가서 다음과 같이 입력해 보세요.:

    (myvenv) $ pip install dj-database-url gunicorn whitenoise

설치가 끝났다면 `djangogirls` 디렉토리로 가서 다음 명령어를 실행해 보세요.:

    (myvenv) $ pip freeze > requirements.txt

위 명령어를 실행했다면 `requirements.txt`라는 파일이 생성되었을 거예요. 이 파일 안에는 지금까지 사용했던 패키지들의 리스트가 들어있어요(Django를 비롯한 파이썬 라이브러리가 있을 거예요.).

> __Note__: `pip freeze` 는 가상환경(virtualenv)에 설치된 파이썬 라이브러리 목록을 출력해줘요. 그리고 `>` 는 `pip freeze` 의 실행결과를 파일에 담아주는 역할을 해요.  어떤일이 일어나는지 `> requirements.txt` 없이 `pip freeze` 를 실행해 보세요.

이 파일을 열고 맨 마지막 줄에 다음 내용을 추가해요.:

    psycopg2==2.6.2

이 줄은 헤로쿠에서 우리가 만든 애플리케이션이 실행될 수 있게 해줄 거예요.

## Procfile

헤로쿠는 Procfile 도 필요해요. 이것은 헤로쿠에게 우리의 웹사이트를 시작시키기 위해 실행되어야 할 명령어의 순서를 알려줘요. 코드 편집기를 열고 `djangogirls` 디렉토리 로 가서 `Procfile`이라 불리는 파일을 생성하고 다음 줄을 추가해 보세요.: 

    web: gunicorn mysite.wsgi --log-file -

이 줄은 우리가 `web` 애플리케이션을 배포할 때 `gunicorn mysite.wsgi`(`gunicorn`은 더 강력한 버전의 `runserver` 명령어에요.) 명령을 실행하는 것을 의미해요.

그리고 저장해요. 자 됐어요!

## `runtime.txt` 파일

우리는 헤로쿠에 어떤 버전의 파이썬을 사용하고 있는지 알려줘야 해요. `djangogirls` 폴더에서 여러분이 사용하고 있는 에디터의 "새로운 파일 만들기" 기능을 사용해서 `runtime.txt`파일을 만들어 주세요. 그리고 파일에 다음 한 줄을 추가해 주세요. 이게 다예요.:

    python-3.5.2

## `mysite/local_settings.py`

헤로쿠는 PythonAnywhere 보다 제한적이라 로컬 컴퓨터(지금 사용하고 있는 컴퓨터)와 다른 설정을 해야 해요. 
우리의 예제는 SQLite로 되어있지만 헤로쿠는 Postgres 를 사용하려고 할거에요. 그래서 로컬 환경에서만 필요한 별도의 설정 파일을 만들어야 해요.

그럼 바로 `mysite/local_settings.py` 파일을 만들어 봐요. 이 파일에서 `DATABASE` 설정을 만들어 줄 텐데요. `mysite/settings.py`에서 했던 것처럼 만들어 주세요.:

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(__file__))

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

DEBUG = True
```

자, 그리고 저장해요! :)

## mysite/settings.py

그리고 다음으로 해줄 것은 우리의 웹 사이트 `settings.py` 파일을 수정해 줄 거에요. 에디터를 열고 `mysite/settings.py` 파일을 불러와서 다음 줄을 변경/추가해주세요.:

```python
import dj_database_url

...

DEBUG = False

ALLOWED_HOSTS = ['127.0.0.1', '.herokuapp.com']

...

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'djangogirls',
        'USER': 'name',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}

...

db_from_env = dj_database_url.config(conn_max_age=500)
DATABASES['default'].update(db_from_env)
```

이 코드는 헤로쿠에 필요한 설정을 구성할거예요.

이제 저장해 주세요.

## mysite/wsgi.py


`mysite/wsgi.py` 파일을 열고 파일의 끝에 다음 라인을 추가해줘요.:

```python
from whitenoise.django import DjangoWhiteNoise
application = DjangoWhiteNoise(application)
```

좋아요!

## 헤로쿠 계정

헤로쿠 toolbelt 를 설치해줘야 하는데요. 여기에서 찾을 수 있어요(이미 설치했다면 다음으로 넘어갈게요.): https://toolbelt.heroku.com/

> 윈도우에서 헤로쿠 toolbelt 설치 프로그램을 실행할 때 설치할 구성 요소를 묻는 메시지가 나타나면 "사용자 지정 설치"를 선택해 주세요. 그리고 "Git and SSH" 앞에 있는 체크박스를 선택해 주세요.

> 윈도우에서 명령 프롬프트를 열고 Git과 SSH를 사용하기 위해 다음의 명령어를 실행해 주세요. `PATH`: `setx PATH "%PATH%;C:\Program Files\Git\bin"` 변경된 내용을 확인해 보기 위해 명령 프롬프트를 재시작해주세요.
 
> 명령 프롬프트를 재시작 하고 난 뒤에는 `djangogirls` 폴더로 돌아가서 가상환경을 활성화하는 것을 잊지 마세요! (Hint: [Check the Django installation chapter](http://tutorial.djangogirls.org/en/django_installation/index.html#working-with-virtualenv))

여기에서 헤로쿠 무료 계정을 만들고 오세요: https://id.heroku.com/signup/www-home-top

다음 명령어로 헤로쿠 계정을 인증받을 수 있어요: 

    $ heroku login

이 명령은 SSH 키가 없을 때 자동으로 SSH 키를 만들어요. SSH 는 헤로쿠에 코드를 푸시하기 위해 필요해요.

## Git 커밋

헤로쿠는 배포를 위해 git을 사용해요. PythonAnywhere 와는 달리 Github를 거치지 않고 헤로쿠에 바로 올릴 수 있어요. 그 전에 두 가지를 미리 조정해야해요.

`djangogirls` 폴더로 가서 `.gitignore` 파일을 열어요. 그리고 `local_settings.py` 를 마지막 줄에 추가해요. git이 `local_settings` 파일을 무시하고 헤로쿠에 올라가지 않고 로컬 컴퓨터에 남아있길 원해서예요.

    *.pyc
    db.sqlite3
    myvenv
    __pycache__
    local_settings.py


그리고 변경사항을 git으로 커밋해봐요.

    $ git status
    $ git add -A .
    $ git commit -m "additional files and changes for Heroku"애플리케이션


## 애플리케이션 이름을 정해요.

웹상에서 `[내가 만든 블로그 이름].herokuapp.com` 주소를 통해 직접 만든 블로그에 들어갈 수 있게 될 거에요. 그래서 누구도 사용하지 않는 이름으로 만들어야 해요. 이 이름은 꼭 장고 `blog` 앱 이나 `mysite` 일 필요는 없어요. 이름은 원하는 대로 지을 수는 있지만 헤로쿠는 사용할 수 있는 문자에 대해 조금 제한적이에요. 소문자, 숫자, 대시(`-`)만 사용할 수 있어요.

이름(아마도 이름이나 별명)을 생각해두고 다음 명령어를 실행해 줘요. `djangogirlsblog` 로 되어 있는 이름을 나만의 이름으로 바꿔주세요.:

    $ heroku create djangogirlsblog

> __Note__: `djangogirlsblog` 를 헤로쿠에 있는 나만의 애플리케이션 이름으로 바꾸는 것을 잊지마세요.

이름을 짓는 건 정말 어려운 일이에요. 딱히 떠오르는 이름이 없다면 다음과 같이 실행해 보세요.

    $ heroku create

헤로쿠가 우리를 위해 아무도 사용하고 있지 않은 이름을 골라줄 거예요(아마도 `enigmatic-cove-2527` 이런 형태의 이름으로).

If you ever feel like changing the name of your Heroku application, you can do so at any time with this command (replace `the-new-name` with the new name you want to use):
헤로쿠에서 만든 이름을 변경하고 싶다면 언제든지 아래 처럼 명령어를 실행해서 바꿔줄 수 있을 거예요(`the-new-name` 을 변경할 이름으로 바꿔주세요.):

    $ heroku apps:rename the-new-name

> __Note__: 애플리케이션 명을 변경하면 꼭 `[the-new-name].herokuapp.com` 와 같이 변경된 이름으로 사이트를 방문해서 확인해 보세요.

## 헤로쿠에 배포하기!

설정과 설치과정이 조금 많았죠? 하지만 이 일은 한 번만 해주면 돼요! 이제 배포를 할 수 있어요!

`heroku create`을 실행했을 때 헤로쿠의 remote를 자동으로 설정했어요. git push 로 우리의 애플리케이션을 배포해 보세요.:

    $ git push heroku master

> __Note__: 헤로쿠는 psycopg 를 컴파일하고 설치하기 때문에 처음 실행 될 때는 아마도 *많은* 출력이 있을 거예요. 이 출력 끝에서 `https://yourapplicationname.herokuapp.com/ deployed to Heroku` 이런 출력을 보았다면 아마도 배포가 성공했을 거예요.

## 애플리케이션에 접속해 보세요.

헤로쿠에 코드를 배포하고 `Procfile`로 프로세스 유형을 지정했어요(`web` 프로세스로요).
이제 헤로쿠에 `web process` 시작하라고 얘기해 줄 수 있어요.

이 명령어를 실행하려면 다음과 같이 해보세요.:

    $ heroku ps:scale web=1

이건 헤로쿠에게 `web` 프로세스의 인스턴스 하나만 실행하도록 해요. 우리의 블로그 애플리케이션은 꽤 단순하기 때문에 많은 전력을 필요하지 않아요. 그래서 하나의 프로세스만으로도 충분할 거에요. 헤로쿠에게 더 많은 프로세스를 실행하도록 요청할 수도 있어요(단, 헤로쿠는 이 프로세스를 "Dynos"라고 불러요. 이 단어를 보게 된다면 놀라지 마세요.) 하지만 여러 프로세스를 실행하게 된다면 더 이상 무료가 아니에요.

브라우저에서 `heroku open`을 통해 앱에 들어갈 수 있어요.

    $ heroku open

> __Note__: 아마도 오류 페이지가 보일 거예요! 1분만 이것에 대해 얘기해 볼게요.

브라우저에서 [https://djangogirlsblog.herokuapp.com/]() 우리의 블로그 애플리케이션 주소로 접속하면 아마도 오류 페이지가 나올 거에요.

이 오류는 헤로쿠에 배포를 할 때 비어있는 새로운 데이터베이스를 만들어주었기 때문이에요. PythonAnywhere 에서 했던 것처럼 `migrate` 와 `createsuperuser` 명령어를 실행해주세요. 이번에는 헤로쿠를 위한 특별한 명령어를 실행할게요. 이렇게요. `heroku run`: 

    $ heroku run python manage.py migrate

    $ heroku run python manage.py createsuperuser

이 명령 프롬프트는 사용자명과 비밀번호를 다시 물어볼 거예요. 웹사이트의 어드민 페이지에서 사용할 로그인 정보를 입력해주세요.

브라우저를 새로고침하고 다시 가보세요! 이제 다른 두 플랫폼에 어떻게 배포하는지 알게 되었어요. 마음에 드는 플랫폼을 선택하세요 :)
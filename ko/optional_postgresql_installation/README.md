# 옵션: PostgreSQL 설치하기

> **Note** 이 장의 일부는 [Geek Girls Carrots](http://django.carrots.pl/) 튜토리얼과 [django-marcador
  tutorial](http://django-marcador.keimlink.de/) 튜토리얼을 기초로 작성되었습니다. django-marcador 튜토리얼 저작권은 Markus Zapke-Gründemann et al이 소유하고 있습니다. 

## 윈도우

다음 링크에서 Postgres 설치 프로그램을 다운로드하세요. : http://www.enterprisedb.com/products-services-training/pgdownload#windows

운영체제에 맞는 최신 버전의 설치 프로그램을 다운받은 후 다음 가이드를 따라 설치하세요. : http://www.postgresqltutorial.com/install-postgresql/. 설치 경로가 필요하니 주의깊게 봐두세요. (특히 `C:\Program Files\PostgreSQL\9.3` 부분입니다.)

## 맥 OS X

다음 링크에서 Postgres App 설치 프로그램을 다운로드하세요. : http://postgresapp.com/

다운받은 애플리케이션을 응용 프로그램 디렉터리로 드래그한 후 더블클릭하면 됩니다!

`PATH` 변수에 Postgres 커맨드 라인을 추가해야 합니다. 자세한 내용은 [링크](http://postgresapp.com/documentation/cli-tools.html)를 확인하세요.

## 리눅스

리눅스는 배포판에 따라 설치방법이 매우 다양합니다. 여기서는 우분투 리눅스, 페도라 리눅스를 기준으로 설명하겠습니다. 다른 배포판은 [PostgresSQL 공식 문서 중 리눅스 설치 내용](https://wiki.postgresql.org/wiki/Detailed_installation_guides#General_Linux)을 참고하세요.
                                                  
### 우분투

아래 명령을 실행하세요.

    sudo apt-get install postgresql postgresql-contrib

### 페도라

아래 명령을 실행하세요.

    sudo yum install postgresql93-server

# 데이터베이스 생성하기

이제 데이터베이스를 생성하고, 그 데이터베이스에 접근할 유저계정을 생성하겠습니다. PostgreSQL에서는 원하는 만큼의 데이터베이스와 유저계정을 생성할 수 있어요. 그래서 다수의 사이트를 운영할 경우, 각 사이트마다 데이터베이스를 생성할 수 있습니다.

## 윈도우

윈도우에서 몇 가지 설정작업이 더 필요해요. 지금 다루는 설정을 모두 이해할 필요는 없어요. 하지만 각 설정들이 어떤 역할을 하는지 궁금하다면, 코치들에게 자유롭게 물어보세요.

1. 명령프롬프트를 실행하세요. (시작 → 모든 프로그램 → 보조 프로그램 → 명령 프롬프트)
2. 다음 명령을 입력하고 엔터키를 입력해주세요: `setx PATH "%PATH%;C:\Program Files\PostgreSQL\9.3\bin"`.프롬프트에서 코드를 붙여넣기하려면, 마우스 오른쪽 버튼를 클릭한 후 메뉴에서 `붙여넣기를 선택하세요. 경로 끝에 `\bin` 이 있는지 확인하세요. `SUCCESS: Specified value was saved.` 라는 메세지가 보여야해요.
3. 명령 프롬프트 창을 닫고 다시 열어주세요.


## 데이터베이스 생성하기

먼저, `psql` 명령을 입력해서 Postgres 콘솔을 실행하겠습니다. 콘솔을 어떻게 실행시키는지 모두 기억하고 있죠?

>맥 OS X 에서는 터미널 애플리케이션을 실행하세요. (애플리케이션 → 유틸리티 안에 있어요) 리눅스에서는 애플리케이션(Applications) → 보조 프로그램(Accessories) → 터미널(Command) 안에 있습니다. 윈도우에서는 시작 메뉴 → 모든 프로그램 → 악세사리 → 명령 프롬프트 에 있습니다. 간혹 윈도우에서는 `psql` 실행 시에, Postgres 설치 시에 선택했던 username과 password입력을 요구할 수 있습니다. 만약 psql에서 암호를 입력했는데 인증되지 않는다면, `psql -U <username> -W` 명령을 실행한 후 암호를 입력해보세요.

    $ psql
    psql (9.3.4)
    Type "help" for help.
    #

프롬프트가 `$` 에서 `#` 으로 바뀌었습니다. 이제 PostgreSQL로 명령을 보낼 수 있다는 뜻이에요. `CREATE USER name;` 명령을 실행해 유저 계정을 생성합시다.

    # CREATE USER name;

 `name` 부분을 로그인 아이디(혹은 자기 이름)로 변경해주세요. 악센트가 들어간 글자나 공백문자는 쓸 수 없습니다. (예를 들어 `bożena maria`일 경우 유효하지 않습니다. `bozena_maria`로 변경해주세요.) 성공하면 콘솔에서 `CREATE ROLE` 응답이 보일 거에요.

이제 Django 프로젝트에서 쓸 데이터베이스를 생성해봐요.

    # CREATE DATABASE djangogirls OWNER name;

`name`에는 위 ROLE 에서 입력했던 `name`을 써야해요. (예: `bozena_maria`). 이제 현재 프로젝트에서 사용할 수있는 빈 데이터베이스가 만들어졌어요. 성공하면 콘솔에서 `CREATE ROLE` 응답이 보일 거에요. 

잘했어요. 이 데이터베이스는 모두 정렬된 데이터베이스랍니다!

# 프로젝트 설정 수정하기

`mysite/settings.py` 파일에서 `DATABASES` 부분을 찾아서

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

아래와 같이 수정해주세요.

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'djangogirls',
        'USER': 'name',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

`USER`와 `NAME`에는 방금 전 생성했던 `name` 을 입력해주세요.


# PostgreSQL 파이썬 패키지 설치하기

먼저 https://toolbelt.heroku.com 에서 Heroku Toolbelt을 다운받아 설치해주세요. 이 Toolbelt에는 Git과 더불어 배포에 필요한 모든 유틸리티들이 들어있어요.

다음으로 PostgreSQL 패키지를 설치해야합니다. 이 패키지 이름은 `psycopg2` 입니다. 설치방법은 윈도우와 리눅스/맥 마다 조금 달라요.



## 윈도우

윈도우에서는 http://www.stickpeople.com/projects/python/win-psycopg/ 에서 바이너리 패키지를 받으세요.

현재 설치되어있는 파이썬 버전과 운영체제(왼쪽 칸 32비트, 오른쪽 칸 64비트)에 맞는 패키지를 다운로드합니다.

다운받은 파일을 `C:\` 경로로 옮겨주세요. `C:\psycopg2.exe` 위치에 있어야 합니다.

터미널에서 `virtualenv` 을 활성화하고, 아래 명령을 실행하세요.

    easy_install C:\psycopg2.exe

## 리눅스 / 맥 OS X

터미널에서 아래 명령을 실행하세요.


    (myvenv) ~/djangogirls$ pip install psycopg2

문제가 없다면 아래와 같은 메시지가 보일 거에요.

    Downloading/unpacking psycopg2
    Installing collected packages: psycopg2
    Successfully installed psycopg2
    Cleaning up...

---

완료되면 python -c "import psycopg2"` 명령을 실행하세요. 모든 오류가 없다면 성공적으로 설치가 끝난 거에요.

# 마이그레이션 적용과 슈퍼 유저 생성하기

웹사이트 프로젝트에서 새로 생성된 데이터베이스를 사용하려면 모든 마이그레이션을 적용해야해요. 가상 환경에서 다음 코드를 실행합니다.

    (myvenv) ~/djangogirls$ python manage.py migrate

블로그에 새 게시물을 추가하려면 다음 코드를 실행하여 수퍼 유저를 만들어야해요.

    (myvenv) ~/djangogirls$ python manage.py createsuperuser --username name 
    
`name`을 사용자 이름으로 바꾸는 것을 잊지 마세요. 이메일과 비밀번호를 묻는 메시지가 나타날 거에요.

이제 다시 서버를 실행하고, 수퍼 유저 계정으로 애플리케이션에 로그인을 한 다음 새 데이터베이스에 게시물을 추가할 수 있어요.

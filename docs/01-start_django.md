# 01. Start Django



## 프로젝트 생성하기

```bash
$ django-admin startproject {mysite}
```



### 프로젝트 구조

```
mysite/                 	-- 최상위 디렉토리 (Root)
	mysite/            	-- 프로젝트 중심 앱
		__init__.py     -- 디렉토리를 파이썬 패키지로 인식
    	settings.py
		url.py
		wsgi.py         -- WebServer Gateway Interface
		asgi.py         -- Asynchronous Server Gateway Interface

	manage.py
	db.sqlite3
```

- **manage.py** : 프로젝트 관리를 위한 명령 유틸리티

  - 앱 생성, 데이터베이스 관련 기능 수행, 개발 서버 실행 등

  - **django-admin과 manage.py** : [공식문서](https://docs.djangoproject.com/en/3.2/ref/django-admin/)

    : 프로젝트 생성 시 django-admin, 다른 모든 기능은 manage.py 사용

- settings.py : [공식문서](https://docs.djangoproject.com/en/3.2/ref/settings/)



### 서버 실행하기

```bash
$ python manage.py runserver
$ python manage.py runserver {ip:port}
```

- 개발 서버 : http://127.0.0.1:8000/


## 앱 생성하기

- **프로젝트** : 웹 서비스 전체
- **앱** : 하나의 기능 단위

```bash
$ python manage.py startapp {appname}
```



### 앱 구조

```
mysite/
	mysite/
    polls/
        __init__.py
        admin.py          -- 관리자 연동 설정 파일
        apps.py           -- 앱 설정
        migrations/       -- 데이터베이스 변경 사항 히스토리 누적
            __init__.py
        models.py         -- 데이터 구조 정의
        tests.py      
        urls.py 
        views.py          -- 메인 로직 처리

	manage.py
	db.sqlite3
```

- apps.py : [공식문서](https://docs.djangoproject.com/en/3.2/ref/applications/)



### url 모듈 연결

```python
# polls/urls.py

from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

```python
# mysite/urls.py

from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

- `include()` : 다른 URL 패턴을 포함할 때 참조할 수 있도록 도와주는 함수
- `path()` 의 인수 `route()` : URL 패턴을 가진 문자열
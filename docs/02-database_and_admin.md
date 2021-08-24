# 02. Database and Admin

데이터 베이스 설치 및 관리자 사이트 생성

## 데이터베이스 설치

- django는 기본적으로 SQLite : `mysite/settings.py`
  ```python
  DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
  }
  ```
- 다른 데이터베이스 사용 시 [공식 문서](https://docs.djangoproject.com/ko/3.2/ref/settings/#std:setting-DATABASES) 참고
  ```python
  DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'mydatabase',
        'USER': 'mydatabaseuser',
        'PASSWORD': 'mypassword',
        'HOST': '127.0.0.1',
        'PORT': '5432',
    }
  }
  ```

- `TIME_ZONE` 설정
  ```python
  TIME_ZONE = 'Asia/Seoul'
  ```

<br>

- `INSTALLED_APPS` 확인
  ```python
  INSTALLED_APPS = [
    'django.contrib.admin',         # 관리용 사이트
    'django.contrib.auth',          # 인증 시스템
    'django.contrib.contenttypes',  # 컨텐츠 타입을 위한 프레임워크
    'django.contrib.sessions',      # 세션 프레임워크
    'django.contrib.messages',      # 메세징 프레임워크
    'django.contrib.staticfiles',   # 정적 파일을 관리하는 프레임워크
  ]
  ```
  
- 데이터베이스 설치 (`INSTALLED_APPS`에 대해서만)
  ```bash
  $ python manage.py migrate
  ```

<br>

## 모델 만들기

- **모델** : 부가적인 메타데이터를 가진 데이터베이스의 구조 (layout)
- 우리의 `polls`앱에서는 `Question` 과 `Choice`라는 두 가지 모델을 생성할 것.

<br>

- `polls/models.py`
  ```python
  from django.db import models


  class Question(models.Model):
      question_text = models.CharField(max_length=200)
      pub_date = models.DateTimeField('date Published')

  class Choice(models.Model):
      question = models.ForeignKey(Question, on_delete=models.CASCADE)
      choice_text = models.CharField(max_length=200)
      votes = models.IntegerField(default=0)
  ```
  
- 각 데이터베이스 필드를 `Field` 클래스의 인스턴스로 표현
- `ForeignKey()` : 외래키 참조

<br>

- `INSTALLED_APPS`에 `polls.apps.PollsConfig` 추가
```bash
$ python manage.py makemigrations polls
Migrations for 'polls':
  polls\migrations\0001_initial.py
    - Create model Question
    - Create model Choice
```

- `polls/migrations/0001_initial.py` 파일 생성 확인
- SQL 명령어 확인
  - app 이름과 모델명을 이용해 테이블 이름 자동 생성됨.
  - 필드명, 필드 타입 관례에 따라 자동 생성
```bash
$ python manage.py sqlmigrate polls 0001
BEGIN;
--
-- Create model Question
--
CREATE TABLE "polls_question" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "question_text" varchar(200) NOT NULL,
    "pub_date" datetime NOT NULL);
--
-- Create model Choice
--
CREATE TABLE "polls_choice" (
    "id" integer NOT NULL PRIMARY KEY AUTOINCREMENT,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL,
    "question_id" bigint NOT NULL REFERENCES
    "polls_question" ("id") DEFERRABLE INITIALLY DEFERRED);
CREATE INDEX "polls_choice_question_id_c5b4b260" ON "polls_choice" ("question_id");
COMMIT;
```

- 데이터베이스 모델 관련 테이블 생성하기 (변경사항 데이터베이스에 적용)
```bash
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
```

<br>

## 데이터베이스 API 사용하기

```bash
$ python manage.py shell
Python 3.9.6 (tags/v3.9.6:db3ff76, Jun 28 2021, 15:26:21) [MSC v.1929 64 bit (AMD64)]
Type 'copyright', 'credits' or 'license' for more information
IPython 7.25.0 -- An enhanced Interactive Python. Type '?' for help.
```
```python
In [1]: from polls.models import Choice, Question

In [2]: Question.objects.all()
Out[2]: <QuerySet []>

In [3]: from django.utils import timezone

In [4]: q = Question(question_text="What's new?", pub_date=timezone.now())

In [5]: q.save()

In [6]: q.id
Out[6]: 1

In [7]: q.question_text
Out[7]: "What's new?"

In [8]: q.pub_date
Out[8]: datetime.datetime(2021, 8, 24, 8, 12, 43, 552040, tzinfo=<UTC>)

In [9]: q.question_text = "What's up?"

In [10]: q.save()

In [11]: Question.objects.all()
Out[11]: <QuerySet [<Question: Question object (1)>]>
```

- timezone을 `Asia/Seoul`로 설정했음에도 `UTC`로 표시됨 발견
- `settings.py`에서 `USE_TZ = False`로 변경 → 해결됨

```python
In [8]: q.pub_date
Out[8]: datetime.datetime(2021, 8, 24, 17, 49, 58, 504533)
```

- 객체 표현법 변경
```python
# polls/models.py

# Question
def __str__(self):
    return self.question_text

# Choice
def __str__(self):
    return self.choice_text
```
- 커스텀 메서드 추가
```python
def was_published_recently(self):
    return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
``` 

<br>

- 커스텀 메서드 확인 및 api 다루기
- 이중 밑줄 (`__`)을 이용한 필드 조회

```bash
$ python manage.py shell
Python 3.9.6 (tags/v3.9.6:db3ff76, Jun 28 2021, 15:26:21) [MSC v.1929 64 bit (AMD64)]
Type 'copyright', 'credits' or 'license' for more information
IPython 7.25.0 -- An enhanced Interactive Python. Type '?' for help.
```
```python
In [1]: from polls.models import Choice, Question

In [2]: Question.objects.all()
Out[2]: <QuerySet [<Question: What's up?>]>

In [3]: Question.objects.filter(id=1)
Out[3]: <QuerySet [<Question: What's up?>]>

In [4]: Question.objects.filter(question_text__startswith='What')
Out[4]: <QuerySet [<Question: What's up?>]>

In [5]: from django.utils import timezone

In [6]: current_year = timezone.now().year

In [7]: Question.objects.get(pub_date__year=current_year)
Out[7]: <Question: What's up?>

In [8]: Question.objects.get(pk=1)  # primary key
Out[8]: <Question: What's up?>

In [9]: q = Question.objects.get(pk=1)

In [10]: q.was_published_recently()
Out[10]: True
```

- choice 추가하기
```python
In [11]: q.choice_set.all()
Out[11]: <QuerySet []>

In [12]: q.choice_set.create(choice_text='Not much', votes=0)
Out[12]: <Choice: Not much>

In [13]: q.choice_set.create(choice_text='The sky', votes=0)
Out[13]: <Choice: The sky>

In [14]: c = q.choice_set.create(choice_text='Just hacking again', votes=0)

In [15]: c.question
Out[15]: <Question: What's up?>

In [16]: q.choice_set.all()
Out[16]: <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

In [17]: q.choice_set.count()
Out[17]: 3

In [18]: Choice.objects.filter(question__pub_date__year=current_year)
Out[18]: <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

In [19]: c = q.choice_set.filter(choice_text__startswith='Just hacking')

In [20]: c.delete()
Out[20]: (1, {'polls.Choice': 1})
```

## 관리자 생성하기
```bash
$ python manage.py createsuperuser
Username (leave blank to use 'user'): admin
Email address: chaeyeonhee@kakao.com
Password:
Password (again):
Superuser created successfully.
```

<br>

- 개발 서버 : http://127.0.0.1:8000/admin/
```bash
- $ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
August 24, 2021 - 19:56:57
Django version 3.2.6, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
[24/Aug/2021 19:57:21] "GET /admin/ HTTP/1.1" 302 0
...
```

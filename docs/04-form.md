# Form

## Form 작성하기
- `polls/templetes/polls/detail.html`
```html
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend><h1>{{ question.question_text }}</h1></legend>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="Vote">
</form>
```
- 답변 저장을 위한 url : `{% url 'polls:vote' question.id %}`
- form으로 전송한 데이터가 실제 웹페이지에서 작성한 데이터인지 판단 : `{% csrf_token %}`
> csrf_token 사용을 위해서는 CsrfViewMiddleware 미들웨어가 필요한데 이 미들웨어는 settings.py의 MIDDLEWARE 항목에 디폴트로 추가되어 있으므로 별도의 설정은 필요 없다.
>
> `settings.py` - `MIDDLEWARE` - `django.middleware.csrf.CsrfViewMiddleware` 확인
- 라디오 버튼을 포함한 선택 항목

<br>

- `polls/views.py` : `vote()` 함수 구현
```python
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

<br>

- `polls/views.py` : 설문조사 결과 페이지로 리다이렉트 뷰 작성
```python
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

<br>

- `polls/templates/polls/results.html` : 웹 페이지 템플릿
```python
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```
- http://127.0.0.1:8000/polls/1/ 페이지에서 확인

<br>

## 제네릭 뷰 사용하기

- `polls/urls.py` : URLconf 수정

```python
# 변경
app_name = 'polls'
urlpatterns = [
    # ex: /polls/
    # path('', views.index, name='index'),
    path('', views.IndexView.as_view(), name='index'),

    # ex: /polls/5/
    # path('<int:question_id>/', views.detail, name='detail'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),

    # ex: /polls/5/results/
    # path('<int:question_id>/results/', views.results, name='results'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),

    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

<br>

- `polls/views.py` : view 수정 (장고의 일반적인 뷰 사용)
```python
# 변경
from django.views import generic

from .models import Choice, Question

"""
def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    # HttpResponse 객체와 자주 쓰는 표현 단축 : render
    return render(request, 'polls/index.html', context)
"""

class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


"""
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
"""
    
class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'

"""
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
"""

class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.
```
- 제네릭 뷰 [공식문서](https://docs.djangoproject.com/ko/3.2/topics/class-based-views/)

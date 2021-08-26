# 06. Style

- `polls/static/polls/style.css`
```css
li a {
    color: green;
}

body {
    background: white url("images/loopy.png") no-repeat;
}
```

- `polls/templates/polls/index.html`
```html
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

<br>

- tailwind css를 이용한 커스텀
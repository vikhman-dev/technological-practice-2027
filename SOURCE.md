# Исходный код четырёх программ

Все файлы приведены без изменения. Для запуска скачайте и распакуйте `project-source.zip`.

## case_01_birthday/birthday.py

```python
"""Проверка даты рождения и вывод цифр звёздочками. Python 3.12+."""
import argparse
from datetime import date

WEEKDAYS = ('понедельник', 'вторник', 'среда', 'четверг',
            'пятница', 'суббота', 'воскресенье')
DIGITS = {
    '0': ('***', '* *', '* *', '* *', '***'),
    '1': (' * ', '** ', ' * ', ' * ', '***'),
    '2': ('***', '  *', '***', '*  ', '***'),
    '3': ('***', '  *', '***', '  *', '***'),
    '4': ('* *', '* *', '***', '  *', '  *'),
    '5': ('***', '*  ', '***', '  *', '***'),
    '6': ('***', '*  ', '***', '* *', '***'),
    '7': ('***', '  *', '  *', '  *', '  *'),
    '8': ('***', '* *', '***', '* *', '***'),
    '9': ('***', '* *', '***', '  *', '***'),
    ' ': ('   ',) * 5,
}

def is_leap_year(year: int) -> bool:
    if not 1 <= year <= 9999:
        raise ValueError('Год должен быть от 1 до 9999.')
    return year % 400 == 0 or (year % 4 == 0 and year % 100 != 0)

def make_birth_date(day: int, month: int, year: int,
                    today: date | None = None) -> date:
    try:
        birth = date(year, month, day)
    except ValueError as exc:
        raise ValueError('Такой календарной даты не существует.') from exc
    if birth > (today or date.today()):
        raise ValueError('Дата рождения не может быть в будущем.')
    return birth

def weekday_name(birth: date) -> str:
    return WEEKDAYS[birth.weekday()]

def age_on(birth: date, today: date | None = None) -> int:
    """Для 29 февраля в невисокосный год возраст меняется 1 марта."""
    current = today or date.today()
    if birth > current:
        raise ValueError('Дата рождения не может быть в будущем.')
    return current.year - birth.year - (
        (current.month, current.day) < (birth.month, birth.day)
    )

def render_date(birth: date) -> str:
    text = f'{birth.day:02d} {birth.month:02d} {birth.year:04d}'
    return '\n'.join(' '.join(DIGITS[ch][row] for ch in text)
                     for row in range(5))

def read_integer(prompt: str, lower: int, upper: int) -> int:
    while True:
        raw = input(prompt).strip()
        try:
            value = int(raw)
        except ValueError:
            print('Ошибка: введите целое число.')
            continue
        if lower <= value <= upper:
            return value
        print(f'Ошибка: допустимы значения от {lower} до {upper}.')

def main() -> None:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument('--on', type=date.fromisoformat,
                        help='контрольная дата YYYY-MM-DD; по умолчанию сегодня')
    args = parser.parse_args()
    current = args.on or date.today()
    print('Дата рождения')
    while True:
        day = read_integer('День: ', 1, 31)
        month = read_integer('Месяц: ', 1, 12)
        year = read_integer('Год: ', 1, 9999)
        try:
            birth = make_birth_date(day, month, year, current)
            break
        except ValueError as exc:
            print(f'Ошибка: {exc} Повторите ввод даты.')
    print(f'День недели: {weekday_name(birth)}')
    print(f'Високосный год: {"да" if is_leap_year(birth.year) else "нет"}')
    print(f'Полных лет на {current:%d.%m.%Y}: {age_on(birth, current)}')
    print(f'Дата: {birth.day:02d} {birth.month:02d} {birth.year:04d}')
    print(render_date(birth))

if __name__ == '__main__':
    try:
        main()
    except (EOFError, KeyboardInterrupt):
        print('\nВвод завершён.')

```

## case_01_birthday/test_birthday.py

```python
import unittest
from datetime import date
from birthday import age_on, is_leap_year, make_birth_date, render_date, weekday_name

class BirthdayTests(unittest.TestCase):
    def test_leap_years(self):
        for year, expected in [(1900, False), (2000, True), (2004, True),
                               (2027, False), (2100, False), (2400, True)]:
            with self.subTest(year=year):
                self.assertEqual(is_leap_year(year), expected)

    def test_invalid_year(self):
        for year in (0, 10000):
            with self.subTest(year=year), self.assertRaises(ValueError):
                is_leap_year(year)

    def test_known_weekday(self):
        self.assertEqual(weekday_name(date(2004, 2, 29)), 'воскресенье')

    def test_invalid_calendar_dates(self):
        for args in [(31, 4, 2000), (29, 2, 1900), (0, 1, 2000),
                     (1, 13, 2000), (1, 1, 0)]:
            with self.subTest(args=args), self.assertRaises(ValueError):
                make_birth_date(*args, today=date(2027, 7, 26))

    def test_future_birth(self):
        with self.assertRaises(ValueError):
            make_birth_date(27, 7, 2027, date(2027, 7, 26))

    def test_age_boundary(self):
        birth = date(2000, 7, 26)
        self.assertEqual(age_on(birth, date(2027, 7, 25)), 26)
        self.assertEqual(age_on(birth, date(2027, 7, 26)), 27)

    def test_leap_birthday_policy(self):
        birth = date(2004, 2, 29)
        self.assertEqual(age_on(birth, date(2027, 2, 28)), 22)
        self.assertEqual(age_on(birth, date(2027, 3, 1)), 23)
        self.assertEqual(age_on(birth, date(2028, 2, 29)), 24)

    def test_newborn(self):
        self.assertEqual(age_on(date(2027, 7, 26), date(2027, 7, 26)), 0)

    def test_age_rejects_future(self):
        with self.assertRaises(ValueError):
            age_on(date(2027, 7, 27), date(2027, 7, 26))

    def test_display(self):
        rows = render_date(date(2004, 2, 9)).splitlines()
        self.assertEqual(len(rows), 5)
        self.assertEqual({len(row) for row in rows}, {39})
        self.assertLessEqual(set(''.join(rows)), {'*', ' '})

if __name__ == '__main__':
    unittest.main(verbosity=2)

```

## case_02_counter/counter.js

```javascript
"use strict";
const MIN = -10;
const MAX = 10;
let value = 0;
const result = document.getElementById("result");
const plus = document.getElementById("plus");
const minus = document.getElementById("minus");
const message = document.getElementById("limit-message");

function render() {
  result.textContent = String(value);
  result.className = "result " + (value > 0 ? "positive" : value < 0 ? "negative" : "zero");
  plus.disabled = value === MAX;
  minus.disabled = value === MIN;
  message.textContent = Math.abs(value) === MAX ? "вы достигли экстремального значения" : "";
}

function changeValue(step) {
  value = Math.min(MAX, Math.max(MIN, value + step));
  render();
}

plus.addEventListener("click", () => changeValue(1));
minus.addEventListener("click", () => changeValue(-1));
render();

```

## case_02_counter/index.html

```html
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Счётчик от минус 10 до 10</title>
  <link rel="stylesheet" href="styles.css">
  <script src="counter.js" defer></script>
</head>
<body>
  <main class="card">
    <p class="eyebrow">Кейс 02 · JavaScript</p>
    <h1>Счётчик</h1>
    <p class="hint">Изменяйте значение в диапазоне от −10 до 10.</p>
    <output id="result" class="result zero" aria-live="polite" aria-atomic="true" aria-label="Значение счётчика">0</output>
    <div class="controls">
      <button id="minus" type="button" aria-label="Уменьшить на один">−</button>
      <button id="plus" type="button" aria-label="Увеличить на один">+</button>
    </div>
    <p id="limit-message" class="notice" role="status" aria-live="polite"></p>
    <p class="legend">Жёлтый · больше нуля<br>Зелёный · меньше нуля<br>Красный · ноль</p>
  </main>
</body>
</html>

```

## case_02_counter/styles.css

```css
* { box-sizing: border-box; }
body { margin: 0; min-height: 100vh; display: grid; place-items: center;
  padding: 24px; background: linear-gradient(135deg,#eaf1f9,#d6e4ef);
  color: #182c43; font: 16px/1.55 system-ui,sans-serif; }
.card { width: min(100%,460px); padding: 36px; border-radius: 18px;
  background: #fff; box-shadow: 0 18px 55px #203e5515; text-align: center; }
.eyebrow { margin: 0; font-size: 12px; letter-spacing: .12em; color: #52708c; }
h1 { margin: 10px 0 8px; font-size: 38px; line-height: 1.15; }
.hint { margin: 0 0 24px; color: #526579; }
.result { display: grid; place-items: center; width: 160px; height: 104px;
  margin: 0 auto 24px; border-radius: 10px; font-size: 52px; font-weight: 700;
  font-variant-numeric: tabular-nums; color: #132a3e; }
.positive { background: #f6d966; }
.negative { background: #8ad4b2; }
.zero { background: #f0a5a5; }
.controls { display: flex; justify-content: center; gap: 16px; }
button { width: 100px; min-height: 52px; border: 0; border-radius: 5px;
  background: linear-gradient(135deg,#284d78,#416c9c); color: #fff;
  font: 700 30px system-ui,sans-serif; cursor: pointer; }
button:disabled { opacity: .4; cursor: not-allowed; }
button:focus-visible { outline: 3px solid #c78512; outline-offset: 4px; }
.notice { min-height: 48px; margin: 18px 0 0; color: #7c3a19; font-weight: 600; }
.legend { margin: 12px 0 0; font-size: 13px; line-height: 1.7; color: #526579; }

```

## case_03_django/config/__init__.py

```python

```

## case_03_django/config/settings.py

```python
"""Локальные настройки. Для публикации обязательно задайте переменные окружения."""
import os
import secrets
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent
DEBUG = os.environ.get('DJANGO_DEBUG', '1') == '1'
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
if not SECRET_KEY:
    if not DEBUG:
        raise RuntimeError('Укажите DJANGO_SECRET_KEY при DJANGO_DEBUG=0.')
    SECRET_KEY = secrets.token_urlsafe(50)
ALLOWED_HOSTS = os.environ.get('DJANGO_ALLOWED_HOSTS', 'localhost,127.0.0.1,[::1]').split(',')
INSTALLED_APPS = ['django.contrib.staticfiles', 'greetings']
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
ROOT_URLCONF = 'config.urls'
TEMPLATES = [{
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    'DIRS': [], 'APP_DIRS': True,
    'OPTIONS': {'context_processors': ['django.template.context_processors.request']},
}]
WSGI_APPLICATION = 'config.wsgi.application'
DATABASES = {'default': {
    'ENGINE': 'django.db.backends.sqlite3',
    'NAME': os.environ.get('DJANGO_DB_PATH', str(BASE_DIR / 'db.sqlite3')),
}}
LANGUAGE_CODE = 'ru-ru'
TIME_ZONE = 'Europe/Moscow'
USE_I18N = True
USE_TZ = True
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'
SESSION_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Lax'
CSRF_COOKIE_SAMESITE = 'Lax'
DATA_UPLOAD_MAX_MEMORY_SIZE = 16384
X_FRAME_OPTIONS = 'DENY'
SECURE_CONTENT_TYPE_NOSNIFF = True
if not DEBUG:
    SESSION_COOKIE_SECURE = True
    CSRF_COOKIE_SECURE = True
    SECURE_SSL_REDIRECT = True

```

## case_03_django/config/urls.py

```python
from django.urls import path
from greetings.views import home

urlpatterns = [path('', home, name='home')]

```

## case_03_django/config/wsgi.py

```python
import os
from django.core.wsgi import get_wsgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
application = get_wsgi_application()

```

## case_03_django/greetings/__init__.py

```python

```

## case_03_django/greetings/forms.py

```python
from django import forms
from .models import Visitor

class NameForm(forms.ModelForm):
    class Meta:
        model = Visitor
        fields = ['name']
        widgets = {'name': forms.TextInput(attrs={
            'placeholder': 'Например, Владимир', 'autocomplete': 'given-name',
            'maxlength': 80, 'aria-describedby': 'name-help',
        })}
        error_messages = {'name': {
            'required': 'Введите имя.',
            'max_length': 'Имя должно содержать не более 80 символов.',
        }}

    def clean_name(self):
        name = ' '.join(self.cleaned_data['name'].split())
        if not name:
            raise forms.ValidationError('Введите имя.')
        if not any(ch.isalpha() for ch in name) or not all(
            ch.isalpha() or ch in " '-’" for ch in name
        ):
            raise forms.ValidationError('Используйте буквы, пробел, дефис или апостроф.')
        return name

```

## case_03_django/greetings/migrations/0001_initial.py

```python
from django.db import migrations, models

class Migration(migrations.Migration):
    initial = True
    dependencies = []
    operations = [migrations.CreateModel(
        name='Visitor',
        fields=[
            ('id', models.BigAutoField(auto_created=True, primary_key=True,
                                       serialize=False, verbose_name='ID')),
            ('name', models.CharField(max_length=80, verbose_name='Имя')),
        ],
    )]

```

## case_03_django/greetings/migrations/__init__.py

```python

```

## case_03_django/greetings/models.py

```python
from django.db import models

class Visitor(models.Model):
    name = models.CharField('Имя', max_length=80)

    def __str__(self):
        return self.name

```

## case_03_django/greetings/static/greetings/styles.css

```css
* { box-sizing: border-box; }
body { margin: 0; min-height: 100vh; display: grid; place-items: center; padding: 24px;
  background: linear-gradient(140deg,#edf3f8,#d8e6ef); color: #183047;
  font: 16px/1.55 system-ui,sans-serif; }
.card { width: min(100%,500px); padding: 38px; border-radius: 18px;
  background: #fff; box-shadow: 0 18px 55px #203e5515; }
.eyebrow { margin: 0; font-size: 12px; letter-spacing: .12em; color: #52708c; }
h1 { margin: 10px 0 16px; font-size: 34px; line-height: 1.2; }
.intro { color: #526579; }
.greeting { font-size: 23px; font-weight: 600; color: #255a46; }
label { display: block; margin: 24px 0 6px; font-size: 14px; font-weight: 600; }
input { width: 100%; padding: 13px; border: 1px solid #9baec0; border-radius: 5px;
  font: 18px system-ui,sans-serif; color: #183047; }
.help,.footer { color: #526579; font-size: 13px; }
.help { margin: 8px 0 20px; }
button { width: 100%; min-height: 48px; border: 0; border-radius: 5px;
  background: linear-gradient(135deg,#284d78,#416c9c); color: #fff;
  font: 600 16px system-ui,sans-serif; cursor: pointer; }
input:focus-visible,button:focus-visible { outline: 3px solid #c78512; outline-offset: 3px; }
.errors { color: #962c2c; font-size: 14px; margin: 10px 0 16px; }
.errors ul { padding-left: 20px; }
.footer { margin: 20px 0 0; }
@media(max-width:420px) { .card { padding: 26px; } h1 { font-size: 29px; } }

```

## case_03_django/greetings/templates/greetings/home.html

```html
{% load static %}
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Персональное приветствие</title>
  <link rel="stylesheet" href="{% static 'greetings/styles.css' %}">
</head>
<body>
  <main class="card">
    <p class="eyebrow">Кейс 03 · Django</p>
    <h1>Рады знакомству</h1>
    {% if visitor %}
      <p class="greeting" role="status">Здравствуйте, {{ visitor.name }}!</p>
    {% else %}
      <p class="intro">Как к вам обращаться?</p>
    {% endif %}
    <form method="post" action="{% url 'home' %}" novalidate>
      {% csrf_token %}
      <label for="{{ form.name.id_for_label }}">Ваше имя</label>
      {{ form.name }}
      <p id="name-help" class="help">До 80 символов. Можно использовать дефис и апостроф.</p>
      {% if form.name.errors %}
        <div class="errors" role="alert">{{ form.name.errors }}</div>
      {% endif %}
      <button type="submit">Submit</button>
    </form>
    <p class="footer">Имя сохраняется в базе после проверки формы.</p>
  </main>
</body>
</html>

```

## case_03_django/greetings/tests.py

```python
from django.test import Client, TestCase
from django.test.utils import CaptureQueriesContext
from django.db import connection
from .models import Visitor

class GreetingTests(TestCase):
    def test_get_form(self):
        response = self.client.get('/')
        self.assertContains(response, 'csrfmiddlewaretoken')
        self.assertContains(response, 'Submit')
        self.assertEqual(Visitor.objects.count(), 0)

    def test_valid_name_saved(self):
        response = self.client.post('/', {'name': 'Владимир'}, follow=True)
        self.assertContains(response, 'Здравствуйте, Владимир!')
        self.assertEqual(Visitor.objects.get().name, 'Владимир')
        self.assertEqual(response.redirect_chain, [('/', 302)])

    def test_empty_and_whitespace(self):
        for value in ['', '   ', '\t\n']:
            with self.subTest(value=value):
                response = self.client.post('/', {'name': value})
                self.assertContains(response, 'Введите имя.')
        self.assertEqual(Visitor.objects.count(), 0)

    def test_missing_field(self):
        self.assertContains(self.client.post('/', {}), 'Введите имя.')
        self.assertEqual(Visitor.objects.count(), 0)

    def test_name_too_long(self):
        response = self.client.post('/', {'name': 'А' * 81})
        self.assertContains(response, 'не более 80')
        self.assertEqual(Visitor.objects.count(), 0)

    def test_exact_maximum(self):
        self.client.post('/', {'name': 'А' * 80})
        self.assertEqual(len(Visitor.objects.get().name), 80)

    def test_normalization_and_hyphen(self):
        self.client.post('/', {'name': '  Анна-Мария   Ольга  '})
        self.assertEqual(Visitor.objects.get().name, 'Анна-Мария Ольга')

    def test_apostrophe(self):
        self.client.post('/', {'name': "О'Коннор"})
        self.assertEqual(Visitor.objects.get().name, "О'Коннор")

    def test_markup_and_digits_rejected(self):
        for value in ['<script>alert(1)</script>', 'Иван42', '---']:
            with self.subTest(value=value):
                response = self.client.post('/', {'name': value})
                self.assertEqual(response.status_code, 200)
        self.assertEqual(Visitor.objects.count(), 0)

    def test_invalid_value_is_escaped(self):
        response = self.client.post('/', {'name': '<img src=x onerror=alert(1)>'})
        self.assertNotContains(response, '<img src=x')
        self.assertContains(response, '&lt;img')

    def test_csrf_required(self):
        client = Client(enforce_csrf_checks=True)
        response = client.post('/', {'name': 'Владимир'})
        self.assertEqual(response.status_code, 403)
        self.assertEqual(Visitor.objects.count(), 0)

    def test_csrf_valid(self):
        client = Client(enforce_csrf_checks=True)
        client.get('/')
        token = client.cookies['csrftoken'].value
        response = client.post('/', {'name': 'Владимир', 'csrfmiddlewaretoken': token})
        self.assertEqual(response.status_code, 302)
        self.assertEqual(Visitor.objects.count(), 1)

    def test_refresh_does_not_duplicate(self):
        self.client.post('/', {'name': 'Владимир'})
        self.client.get('/')
        self.client.get('/')
        self.assertEqual(Visitor.objects.count(), 1)

    def test_clients_do_not_share_greeting(self):
        self.client.post('/', {'name': 'Владимир'})
        response = Client().get('/')
        self.assertNotContains(response, 'Здравствуйте, Владимир!')

    def test_unknown_visitor_id(self):
        session = self.client.session
        session['visitor_id'] = 9999
        session.save()
        self.client.cookies['sessionid'] = session.session_key
        self.assertEqual(self.client.get('/').status_code, 200)

    def test_unsupported_method(self):
        self.assertEqual(self.client.put('/').status_code, 405)

    def test_one_lookup_for_greeting(self):
        self.client.post('/', {'name': 'Владимир'})
        with CaptureQueriesContext(connection) as queries:
            response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(queries), 1)

    def test_only_name_business_field(self):
        business_fields = [f.name for f in Visitor._meta.fields if not f.primary_key]
        self.assertEqual(business_fields, ['name'])

```

## case_03_django/greetings/views.py

```python
from django.shortcuts import redirect, render
from django.views.decorators.http import require_http_methods
from .forms import NameForm
from .models import Visitor

@require_http_methods(['GET', 'POST'])
def home(request):
    form = NameForm(request.POST if request.method == 'POST' else None)
    if request.method == 'POST' and form.is_valid():
        visitor = form.save()
        request.session['visitor_id'] = visitor.pk
        return redirect('home')
    visitor_id = request.session.get('visitor_id')
    visitor = Visitor.objects.filter(pk=visitor_id).first() if visitor_id else None
    return render(request, 'greetings/home.html', {'form': form, 'visitor': visitor})

```

## case_03_django/manage.py

```python
import os
import sys

if __name__ == '__main__':
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings')
    from django.core.management import execute_from_command_line
    execute_from_command_line(sys.argv)

```

## case_03_django/requirements.txt

```text
Django==5.2.19
asgiref==3.13.0
sqlparse==0.6.1

```

## case_04_calculator/calculator.js

```javascript
"use strict";
const first = document.getElementById("first");
const second = document.getElementById("second");
const result = document.getElementById("result");
const NUMBER_PATTERN = /^[+-]?(?:\d+(?:[.,]\d*)?|[.,]\d+)(?:e[+-]?\d+)?$/i;

function sum(a, b) { return a + b; }
function difference(a, b) { return a - b; }
function product(a, b) { return a * b; }
function division(a, b) {
  if (b === 0) throw new Error("Деление на ноль невозможно.");
  return a / b;
}
const operations = { sum, difference, product, division };

function parseNumber(input, label) {
  const text = input.value.trim();
  input.removeAttribute("aria-invalid");
  if (text === "" || !NUMBER_PATTERN.test(text)) {
    input.setAttribute("aria-invalid", "true");
    throw new Error(`${label}: введите число.`);
  }
  const value = Number(text.replace(",", "."));
  if (!Number.isFinite(value)) {
    input.setAttribute("aria-invalid", "true");
    throw new Error(`${label}: число вне допустимого диапазона.`);
  }
  return value;
}

function calculate(operation) {
  first.removeAttribute("aria-invalid");
  second.removeAttribute("aria-invalid");
  try {
    const a = parseNumber(first, "Первое поле");
    const b = parseNumber(second, "Второе поле");
    const value = operations[operation](a, b);
    if (!Number.isFinite(value)) throw new Error("Результат вне допустимого диапазона.");
    result.classList.remove("error");
    result.textContent = new Intl.NumberFormat("ru-RU", {
      maximumSignificantDigits: 12, useGrouping: false
    }).format(Object.is(value, -0) ? 0 : value);
  } catch (error) {
    result.classList.add("error");
    result.textContent = error.message;
  }
}

document.querySelectorAll("[data-operation]").forEach(button => {
  button.addEventListener("click", () => calculate(button.dataset.operation));
});

```

## case_04_calculator/index.html

```html
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Калькулятор двух чисел</title>
  <link rel="stylesheet" href="styles.css">
  <script src="calculator.js" defer></script>
</head>
<body>
  <main class="card">
    <p class="eyebrow">Кейс 04 · JavaScript</p>
    <h1>Калькулятор</h1>
    <p class="hint">Два числа и четыре арифметических действия.</p>
    <div class="inputs">
      <label for="first">Первое число
        <input id="first" type="text" inputmode="decimal" autocomplete="off" placeholder="Например, 12,5">
      </label>
      <label for="second">Второе число
        <input id="second" type="text" inputmode="decimal" autocomplete="off" placeholder="Например, 2">
      </label>
    </div>
    <div class="operations">
      <button type="button" data-operation="sum">сумма</button>
      <button type="button" data-operation="difference">разность</button>
      <button type="button" data-operation="product">произведение</button>
      <button type="button" data-operation="division">деление</button>
    </div>
    <p class="result-label">Результат</p>
    <output id="result" aria-live="polite" aria-atomic="true"></output>
    <p class="note">Десятичный разделитель — точка или запятая.</p>
  </main>
</body>
</html>

```

## case_04_calculator/styles.css

```css
* { box-sizing: border-box; }
body { margin: 0; min-height: 100vh; display: grid; place-items: center; padding: 24px;
  background: linear-gradient(130deg,#edf3f8,#d8e6ef); color: #183047;
  font: 16px/1.5 system-ui,sans-serif; }
.card { width: min(100%,600px); background: #fff; padding: 36px; border-radius: 18px;
  box-shadow: 0 18px 55px #203e5515; }
.eyebrow { font-size: 12px; letter-spacing: .12em; margin: 0; color: #52708c; }
h1 { font-size: 34px; margin: 10px 0 8px; line-height: 1.2; }
.hint { margin: 0 0 24px; color: #526579; }
.inputs { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; }
label { font-weight: 600; font-size: 14px; }
input { width: 100%; padding: 12px; margin-top: 6px; border: 1px solid #9baec0;
  border-radius: 5px; font: 18px system-ui,sans-serif; color: #183047; }
input[aria-invalid="true"] { border-color: #a32929; }
.operations { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin: 22px 0; }
button { border: 0; border-radius: 5px; min-height: 46px; padding: 10px;
  background: linear-gradient(135deg,#284d78,#416c9c); color: #fff;
  font: 600 15px system-ui,sans-serif; cursor: pointer; }
input:focus-visible,button:focus-visible { outline: 3px solid #c78512; outline-offset: 3px; }
.result-label { font-weight: 600; margin: 0 0 6px; font-size: 14px; }
output { display: block; min-height: 72px; padding: 16px; border-radius: 6px;
  background: #edf4fa; font-size: 26px; overflow-wrap: anywhere; }
output.error { color: #982525; background: #fff0ee; font-size: 17px; }
.note { color: #526579; font-size: 13px; margin-bottom: 0; }
@media (max-width:440px) { .card { padding: 24px; } .inputs { grid-template-columns:1fr; } }

```

## package.json

```json
{
  "name": "project-technology-practice",
  "version": "1.0.0",
  "private": true,
  "scripts": { "test": "node tests/js_checks.cjs" },
  "devDependencies": { "jsdom": "26.1.0" }
}

```

## tests/benchmark_django.py

```python
"""Локальная оценка обработки GET тестовым клиентом Django без сети."""
import json
import os
from pathlib import Path
import platform
import secrets
import statistics
import subprocess
import sys
import tempfile
import time

ROOT = Path(__file__).resolve().parents[1]

def worker(size):
    sys.path.insert(0, str(ROOT / 'case_03_django'))
    os.environ['DJANGO_SETTINGS_MODULE'] = 'config.settings'
    os.environ['DJANGO_ALLOWED_HOSTS'] = 'testserver,localhost,127.0.0.1'
    import django
    django.setup()
    from django.core.management import call_command
    from django.test import Client
    from django.db import connection
    from django.test.utils import CaptureQueriesContext
    from greetings.models import Visitor
    call_command('migrate', verbosity=0)
    Visitor.objects.bulk_create([Visitor(name='Участник') for _ in range(size)], batch_size=500)
    client = Client()
    client.post('/', {'name': 'Владимир'})
    for _ in range(20):
        client.get('/')
    with CaptureQueriesContext(connection) as queries:
        assert client.get('/').status_code == 200
    durations = []
    for _ in range(200):
        start = time.perf_counter_ns()
        response = client.get('/')
        assert response.status_code == 200
        durations.append((time.perf_counter_ns() - start) / 1_000_000)
    ordered = sorted(durations)
    return {
        'rows': Visitor.objects.count(), 'warmup': 20, 'requests': 200,
        'median_ms': round(statistics.median(durations), 3),
        'p95_ms': round(ordered[189], 3), 'query_count': len(queries),
        'min_ms': round(min(durations), 3), 'max_ms': round(max(durations), 3),
    }

def main():
    if len(sys.argv) == 3 and sys.argv[1] == '--worker':
        print(json.dumps(worker(int(sys.argv[2]))))
        return
    results = []
    with tempfile.TemporaryDirectory() as folder:
        for size in (100, 10000):
            env = os.environ.copy()
            env['DJANGO_DB_PATH'] = str(Path(folder) / f'{size}.sqlite3')
            env['DJANGO_SECRET_KEY'] = secrets.token_urlsafe(50)
            process = subprocess.run([sys.executable, __file__, '--worker', str(size)],
                                     env=env, capture_output=True, text=True, check=True)
            results.append(json.loads(process.stdout))
    output = {
        'method': 'Django Client GET /; sequential; no HTTP network; SQLite',
        'python': platform.python_version(), 'platform': platform.system(),
        'results': results,
    }
    evidence = ROOT / 'evidence'
    evidence.mkdir(exist_ok=True)
    (evidence / 'benchmark.json').write_text(json.dumps(output, ensure_ascii=False, indent=2), encoding='utf-8')
    print(json.dumps(output, ensure_ascii=False, indent=2))

if __name__ == '__main__':
    main()

```

## tests/browser_checks.py

```python
"""Проверки интерфейсов в Chromium с временной базой и локальными серверами."""
import json
import os
from pathlib import Path
import secrets
import shutil
import socket
import subprocess
import sys
import tempfile
import time
from urllib.error import URLError
from urllib.request import urlopen

from playwright.sync_api import sync_playwright

ROOT = Path(__file__).resolve().parents[1]
OUT = ROOT / 'evidence'
OUT.mkdir(exist_ok=True)
checks = []

def record(name, actual, expected):
    ok = actual == expected
    checks.append({'name': name, 'actual': actual, 'expected': expected, 'passed': ok})
    if not ok:
        raise AssertionError(f'{name}: {actual!r} != {expected!r}')

def free_port():
    with socket.socket() as sock:
        sock.bind(('127.0.0.1', 0))
        return sock.getsockname()[1]

def wait_ready(url):
    deadline = time.monotonic() + 15
    while time.monotonic() < deadline:
        try:
            with urlopen(url, timeout=1) as response:
                if response.status == 200:
                    return
        except (URLError, OSError):
            time.sleep(.1)
    raise RuntimeError(f'Сервер не запустился: {url}')

def main():
    web_port, django_port = free_port(), free_port()
    with tempfile.TemporaryDirectory() as temporary, open(os.devnull, 'w') as null:
        env = os.environ.copy()
        env['DJANGO_DB_PATH'] = str(Path(temporary) / 'browser.sqlite3')
        env['DJANGO_SECRET_KEY'] = secrets.token_urlsafe(50)
        subprocess.run([sys.executable, 'manage.py', 'migrate', '--noinput'],
                       cwd=ROOT / 'case_03_django', env=env, stdout=null, check=True)
        servers = []
        try:
            servers.append(subprocess.Popen(
                [sys.executable, '-m', 'http.server', str(web_port), '--bind', '127.0.0.1'],
                cwd=ROOT, stdout=null, stderr=null))
            servers.append(subprocess.Popen(
                [sys.executable, 'manage.py', 'runserver', f'127.0.0.1:{django_port}', '--noreload'],
                cwd=ROOT / 'case_03_django', env=env, stdout=null, stderr=null))
            static_url = f'http://127.0.0.1:{web_port}'
            django_url = f'http://127.0.0.1:{django_port}'
            wait_ready(static_url)
            wait_ready(django_url)
            with sync_playwright() as playwright:
                executable = os.environ.get('BROWSER_EXECUTABLE') or shutil.which('chromium')
                args = {'headless': True}
                if executable:
                    args['executable_path'] = executable
                browser = playwright.chromium.launch(**args)
                page = browser.new_page(viewport={'width': 1024, 'height': 780}, locale='ru-RU')
                errors = []
                page.on('pageerror', lambda err: errors.append(str(err)))
                page.goto(static_url + '/case_02_counter/')
                result = page.locator('#result')
                plus, minus = page.locator('#plus'), page.locator('#minus')
                record('C01 Начальный ноль', result.inner_text(), '0')
                record('C02 Красный фон нуля', result.evaluate('el=>getComputedStyle(el).backgroundColor'), 'rgb(240, 165, 165)')
                record('C03 Скругление кнопки', plus.evaluate('el=>getComputedStyle(el).borderRadius'), '5px')
                record('C04 Градиент фона и кнопки', page.locator('body').evaluate('el=>getComputedStyle(el).backgroundImage.includes("linear-gradient")') and plus.evaluate('el=>getComputedStyle(el).backgroundImage.includes("linear-gradient")'), True)
                plus.click()
                record('C05 Увеличение', result.inner_text(), '1')
                record('C06 Жёлтый положительный', result.evaluate('el=>getComputedStyle(el).backgroundColor'), 'rgb(246, 217, 102)')
                for _ in range(9):
                    plus.click()
                record('C07 Верхняя граница', [result.inner_text(), plus.is_disabled(), minus.is_enabled()], ['10', True, True])
                record('C08 Сообщение на границе', page.locator('#limit-message').inner_text(), 'вы достигли экстремального значения')
                page.locator('main').screenshot(path=str(OUT / 'counter_10.png'))
                minus.click()
                record('C09 Возврат с границы', [result.inner_text(), plus.is_enabled(), page.locator('#limit-message').inner_text()], ['9', True, ''])
                for _ in range(19):
                    minus.click()
                record('C10 Нижняя граница', [result.inner_text(), minus.is_disabled(), plus.is_enabled()], ['-10', True, True])
                record('C11 Зелёный отрицательный', result.evaluate('el=>getComputedStyle(el).backgroundColor'), 'rgb(138, 212, 178)')
                page.locator('main').screenshot(path=str(OUT / 'counter_minus10.png'))
                plus.click()
                record('C12 Возврат с нижней границы', [result.inner_text(), minus.is_enabled()], ['-9', True])
                page.set_viewport_size({'width': 375, 'height': 812})
                record('C13 Нет горизонтальной прокрутки 375px', page.evaluate('document.documentElement.scrollWidth <= window.innerWidth'), True)
                page.set_viewport_size({'width': 1024, 'height': 780})
                page.goto(static_url + '/case_04_calculator/')
                record('A01 Результат изначально пуст', page.locator('#result').inner_text(), '')
                def calculate(a, b, operation):
                    page.locator('#first').fill(a)
                    page.locator('#second').fill(b)
                    page.locator(f'[data-operation="{operation}"]').click()
                    return page.locator('#result').inner_text()
                for name, a, b, operation, expected in [
                    ('A02 Сумма', '12.5', '2', 'sum', '14,5'),
                    ('A03 Разность', '5', '8', 'difference', '-3'),
                    ('A04 Произведение', '-2.5', '4', 'product', '-10'),
                    ('A05 Деление', '7', '2', 'division', '3,5'),
                    ('A06 Запятая', '1,5', '2,25', 'sum', '3,75'),
                    ('A07 Нулевое число допустимо', '0', '5', 'sum', '5'),
                    ('A08 Десятичное отображение', '0.1', '0.2', 'sum', '0,3'),
                    ('A09 Экспонента', '1e3', '2', 'product', '2000'),
                    ('A10 Пустой ввод', '', '2', 'sum', 'Первое поле: введите число.'),
                    ('A11 Буквы', 'abc', '2', 'sum', 'Первое поле: введите число.'),
                    ('A12 Частично числовая строка', '12abc', '2', 'sum', 'Первое поле: введите число.'),
                    ('A13 Деление на ноль', '7', '0', 'division', 'Деление на ноль невозможно.'),
                    ('A14 Переполнение результата', '1e308', '1e308', 'product', 'Результат вне допустимого диапазона.'),
                    ('A15 Бесконечный ввод', '1e309', '1', 'sum', 'Первое поле: число вне допустимого диапазона.'),
                    ('A16 Ошибка второго поля', '2', 'text', 'sum', 'Второе поле: введите число.'),
                    ('A17 Лишние разделители', '1,2,3', '1', 'sum', 'Первое поле: введите число.'),
                    ('A18 Пробелы по краям', '  3  ', ' 4 ', 'sum', '7'),
                    ('A19 Отрицательный ноль', '-0', '1', 'product', '0'),
                ]:
                    record(name, calculate(a, b, operation), expected)
                calculate('12,5', '2', 'product')
                page.locator('main').screenshot(path=str(OUT / 'calculator_result.png'))
                calculate('7', '0', 'division')
                page.locator('main').screenshot(path=str(OUT / 'calculator_error.png'))
                calculate('8', '2', 'division')
                record('A20 Восстановление после ошибки', [page.locator('#result').inner_text(), page.locator('#result').get_attribute('class')], ['4', ''])
                page.set_viewport_size({'width': 375, 'height': 812})
                record('A21 Нет горизонтальной прокрутки 375px', page.evaluate('document.documentElement.scrollWidth <= window.innerWidth'), True)
                page.set_viewport_size({'width': 1024, 'height': 780})
                page.goto(django_url)
                page.locator('button[type=submit]').click()
                record('D01 Пустое имя в браузере', page.locator('.errors').inner_text(), 'Введите имя.')
                page.locator('main').screenshot(path=str(OUT / 'django_error.png'))
                page.locator('#id_name').fill('Владимир')
                page.locator('button[type=submit]').click()
                record('D02 Приветствие', page.locator('.greeting').inner_text(), 'Здравствуйте, Владимир!')
                page.locator('main').screenshot(path=str(OUT / 'django_greeting.png'))
                page.reload()
                record('D03 Приветствие после обновления', page.locator('.greeting').inner_text(), 'Здравствуйте, Владимир!')
                page.set_viewport_size({'width': 375, 'height': 812})
                record('D04 Нет горизонтальной прокрутки 375px', page.evaluate('document.documentElement.scrollWidth <= window.innerWidth'), True)
                record('D05 Нет ошибок JavaScript', errors, [])
                browser.close()
        finally:
            for server in servers:
                server.terminate()
            for server in servers:
                server.wait(timeout=10)
    (OUT / 'browser_results.json').write_text(json.dumps(checks, ensure_ascii=False, indent=2), encoding='utf-8')
    print(f'{len(checks)} browser checks passed')

if __name__ == '__main__':
    main()

```

## tests/js_checks.cjs

```javascript
"use strict";
// Проверки DOM и обработчиков событий. Геометрия реального браузера не измеряется.
const fs = require("node:fs");
const path = require("node:path");
const assert = require("node:assert/strict");
const { JSDOM } = require("jsdom");
const root = path.resolve(__dirname, "..");
const results = [];
function check(name, actual, expected) {
  assert.deepEqual(actual, expected, name);
  results.push({ name, actual, expected, passed: true });
}
function load(directory, script) {
  const html = fs.readFileSync(path.join(root, directory, "index.html"), "utf8");
  const dom = new JSDOM(html, { runScripts: "outside-only" });
  const style = dom.window.document.createElement("style");
  style.textContent = fs.readFileSync(path.join(root, directory, "styles.css"), "utf8");
  dom.window.document.head.append(style);
  dom.window.eval(fs.readFileSync(path.join(root, directory, script), "utf8"));
  return dom;
}
const counter = load("case_02_counter", "counter.js");
const doc = counter.window.document;
const result = doc.getElementById("result");
const plus = doc.getElementById("plus");
const minus = doc.getElementById("minus");
const notice = doc.getElementById("limit-message");
check("C01 initial zero", result.textContent, "0");
check("C02 zero class", result.classList.contains("zero"), true);
check("C03 zero background", counter.window.getComputedStyle(result).backgroundColor, "rgb(240, 165, 165)");
check("C04 radius 5px", counter.window.getComputedStyle(plus).borderRadius, "5px");
check("C05 exactly two buttons", doc.querySelectorAll("button").length, 2);
plus.click();
check("C06 plus one", result.textContent, "1");
check("C07 positive yellow", counter.window.getComputedStyle(result).backgroundColor, "rgb(246, 217, 102)");
for (let i = 0; i < 9; i++) plus.click();
check("C08 upper boundary", [result.textContent, plus.disabled, minus.disabled], ["10", true, false]);
check("C09 extreme message", notice.textContent, "вы достигли экстремального значения");
for (let i = 0; i < 100; i++) plus.click();
check("C10 cannot exceed maximum", result.textContent, "10");
minus.click();
check("C11 plus reenabled", [result.textContent, plus.disabled, notice.textContent], ["9", false, ""]);
for (let i = 0; i < 19; i++) minus.click();
check("C12 lower boundary", [result.textContent, plus.disabled, minus.disabled], ["-10", false, true]);
check("C13 negative green", counter.window.getComputedStyle(result).backgroundColor, "rgb(138, 212, 178)");
for (let i = 0; i < 100; i++) minus.click();
check("C14 cannot exceed minimum", result.textContent, "-10");
plus.click();
check("C15 minus reenabled", [result.textContent, minus.disabled, notice.textContent], ["-9", false, ""]);
for (let i = 0; i < 9; i++) plus.click();
check("C16 return to zero", [result.textContent, result.classList.contains("zero")], ["0", true]);
const calc = load("case_04_calculator", "calculator.js");
const cd = calc.window.document;
check("A01 initial empty result", cd.getElementById("result").textContent, "");
check("A02 exactly two inputs and four buttons", [cd.querySelectorAll("input").length, cd.querySelectorAll("button").length], [2, 4]);
function calculate(a, b, operation) {
  cd.getElementById("first").value = a;
  cd.getElementById("second").value = b;
  cd.querySelector(`[data-operation="${operation}"]`).click();
  return cd.getElementById("result").textContent;
}
for (const [name, a, b, operation, expected] of [
  ["A03 sum", "12.5", "2", "sum", "14,5"],
  ["A04 difference", "5", "8", "difference", "-3"],
  ["A05 product", "-2.5", "4", "product", "-10"],
  ["A06 division", "7", "2", "division", "3,5"],
  ["A07 decimal comma", "1,5", "2,25", "sum", "3,75"],
  ["A08 zero input", "0", "5", "sum", "5"],
  ["A09 decimal display", "0.1", "0.2", "sum", "0,3"],
  ["A10 exponent", "1e3", "2", "product", "2000"],
  ["A11 empty input", "", "2", "sum", "Первое поле: введите число."],
  ["A12 letters", "abc", "2", "sum", "Первое поле: введите число."],
  ["A13 numeric prefix", "12abc", "2", "sum", "Первое поле: введите число."],
  ["A14 zero divisor", "7", "0", "division", "Деление на ноль невозможно."],
  ["A15 overflow", "1e308", "1e308", "product", "Результат вне допустимого диапазона."],
  ["A16 infinite input", "1e309", "1", "sum", "Первое поле: число вне допустимого диапазона."],
  ["A17 second input error", "2", "text", "sum", "Второе поле: введите число."],
  ["A18 multiple separators", "1,2,3", "1", "sum", "Первое поле: введите число."],
  ["A19 trimmed whitespace", " 3 ", " 4 ", "sum", "7"],
  ["A20 negative zero", "-0", "1", "product", "0"],
  ["A21 incomplete decimal", ".5", "2", "product", "1"],
  ["A22 hexadecimal rejected", "0x10", "2", "sum", "Первое поле: введите число."],
  ["A23 Infinity rejected", "Infinity", "2", "sum", "Первое поле: введите число."],
  ["A24 minus zero divisor", "1", "-0", "division", "Деление на ноль невозможно."],
]) check(name, calculate(a, b, operation), expected);
calculate("8", "2", "division");
check("A25 recovery", [cd.getElementById("result").textContent, cd.getElementById("result").classList.contains("error")], ["4", false]);
check("A26 invalid attribute cleared", [cd.getElementById("first").hasAttribute("aria-invalid"), cd.getElementById("second").hasAttribute("aria-invalid")], [false, false]);
check("A27 result is live region", cd.getElementById("result").getAttribute("aria-live"), "polite");
fs.mkdirSync(path.join(root, "evidence"), { recursive: true });
fs.writeFileSync(path.join(root, "evidence", "javascript_results.json"), JSON.stringify(results, null, 2));
console.log(`${results.length} DOM and JavaScript checks passed`);

```

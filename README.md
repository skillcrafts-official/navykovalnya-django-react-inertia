# navykovalnya-django-react-inertia
Личный продакшн-проект в связке Django, React JS, Vite, Inertia.js

## Инструкция по настройке связки Django + React + Inertia.js
### Используемые пакеты
- Python 3.14
- Django 4.2.30
- django-inertia 1.3.0
- Node.js 16+
- React 18 (устанавливается через `react` и `react-dom`)
- @inertiajs/react 3.3.1
- Vite (через `vite` и `@vitejs/plugin-react`)

### Создание backend-части
1. Виртуальное окружение
   ```bash
   uv init
   uv venv
   source .venv/bin/activate
   ```
2. Установите Django и django-inertia
   ```bash
   uv pip install django django-inertia
   uv pip freeze > requirements.txt
   uv add -r requirements.txt
   ```
3. Создайте Django-проект
   ```bash
   django-admin startproject navykovalnya .
   python manage.py migrate
   ```
4. navykovalnya.settings.py
   ```python
   INSTALLED_APPS = [
       # ... стандартные приложения Django ...
       'django_inertia',          # <-- добавить
   ]

   MIDDLEWARE = [
       'django.contrib.sessions.middleware.SessionMiddleware',
       'django.contrib.auth.middleware.AuthenticationMiddleware',
       'django_inertia.middleware.InertiaMiddleware',   # <-- добавить
       'django.middleware.common.CommonMiddleware',
       # ...
   ]

   TEMPLATES = [
       {
           # ...
           'DIRS': [BASE_DIR / 'templates'],
           # ...
       },
   ]

   INERTIA_LAYOUT = 'app.html'
   STATIC_URL = '/static/'
   STATICFILES_DIRS = [BASE_DIR / 'static']
   ```
5. Создание и настройка приложения
   ```bash
   mkdir -p backend/myapp
   python manage.py startapp myapp
   ```
   Стандартно настраиваем `myapp/apps.py`, `urls.py`.
   Пример настройки рендера inertia-шаблонов
   ```python
   from django_inertia import Inertia

   def index(request):
       return Inertia().render(
           request,
           'Index',     # имя React-компонента
           props={
               'message': 'Привет от Django!',
               'count': 42,
           }
       )
   ```
   > **Важно**: `Inertia` — это класс, поэтому необходимо создавать экземпляр: `Inertia().render(...)`. Аргументы: `request`, имя компонента (строка), `props` (словарь).
6. Создание базового шаблона `templates/app.html`:
   ```django
   {% load inertia_tags %}
   {% load static %}
   <!DOCTYPE html>
   <html lang="ru">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Inertia + Django + React</title>
   </head>
   <body>
     {% inertia %}
     <script src="{% static 'js/app.js' %}"></script>
   </body>
   </html>
   ```
   Тег `{% inertia %}` автоматически подставит `<div id="app" data-page='...'>` с корректным JSON из контекста `page__`.

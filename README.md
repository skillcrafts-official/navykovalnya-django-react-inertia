# navykovalnya-django-react-inertia
Личный продакшн-проект в связке Django, React JS, Vite, Inertia.js

## Структура проекта
```text
django_react_project/
├── myproject/             # Django проект (настройки, urls)
│   ├── settings.py
│   ├── urls.py
│   ├── templates/
│   │   └── base.html      # Главный шаблон-обертка
│   └── ...
├── backend/               # Папка с модулями приложения
│   ├── core/              # Модуль для общих утилит (layouts, shared components)
│   ├── customers/         # Модуль "Покупатели"
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── models.py      # Модели (только для этого модуля)
│   │   ├── urls.py        # URL-маршруты (только для этого модуля)
│   │   ├── views.py       # Inertia-представления (только для этого модуля)
│   │   └── tests.py       # Тесты для модуля
│   ├── products/          # Модуль "Товары"
│   │   └── ...            # (аналогичная структура)
│   └── ...                # Другие модули
├── frontend/              # Корень фронтенда (React + Vite)
│   ├── src/
│   │   ├── pages/         # Компоненты-страницы, которые импортируются в views.py
│   │   │   ├── Customers/
│   │   │   │   └── Index.jsx
│   │   │   └── Products/
│   │   │       └── Index.jsx
│   │   ├── layouts/       # Компоненты-обертки (могут подключаться на бэке)
│   │   ├── components/    # Переиспользуемые React-компоненты
│   │   ├── main.jsx       # Точка входа (инициализация Inertia)
│   │   └── app.css        # Стили
│   ├── index.html
│   ├── vite.config.js
│   ├── package.json
│   └── ...
├── manage.py
├── requirements.txt
└── ...
```

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

### Создание frontend-части
1. Инициализация packege.json и установка зависимостей.
   ```bash
   mkdir frontend && cd frontend
   npm init -y
   npm install react react-dom @inertiajs/react
   npm install -D vite @vitejs/plugin-react
   ```
2. Создание структуры исходников (см. общую структуру в начале файла)
   ```bash
   mkdir -p frontend/src/pages
   ```
3. Точка входа React/Inertia - файл `frontend/src/app.jsx`
   ```jsx
   import React from 'react';
   import { createRoot } from 'react-dom/client';
   import { createInertiaApp } from '@inertiajs/react';

   const page = JSON.parse(document.getElementById('app').dataset.page);

   createInertiaApp({
      page,
      resolve: (name) => {
         const pages = import.meta.glob('./pages/*.jsx', { eager: true });
         return pages[`./pages/${name}.jsx`];
      },
      setup({ el, App, props }) {
      createRoot(el).render(<App {...props} />);
      },
   });
   ```
   > Критично: передача `page` (начальных данных) обязательна в `@inertiajs/react v3`.
4. Создание тестового React-компонента в `frontend/src/pages/Test.jsx`
   ```jsx
   import React from 'react';

   export default function Index({ message, count }) {
      return (
         <div>
         <h1>{message}</h1>
         <p>Счётчик: {count}</p>
         </div>
      );
   }
   ```
5. Настройка Vite - файл `frontend/vite.config.js`
   ```js
   import { defineConfig } from 'vite';
   import react from '@vitejs/plugin-react';

   export default defineConfig({
      plugins: [react()],
      build: {
         outDir: 'static',
         rollupOptions: {
            input: 'resources/js/app.jsx',
            output: {
               entryFileNames: 'js/app.js',
               chunkFileNames: 'js/[name].js',
               assetFileNames: 'assets/[name].[ext]',
            },
         },
      },
   });
   ```
6. Сборка фронтенда
   ```bash
   npx vite build
   ```
   В папке `static/src/` появится `app.js`

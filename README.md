# Лабораторная работа 6. Проектирование и реализация комплексной микросервисной системы для автоматизации бизнес-процесса с использованием Docker Compose
# Цель
Научиться разворачивать многокомпонентные приложения, понимать взаимодействие между контейнерами (Business Logic + Database) и модифицировать параметры системы под конкретные бизнес-задачи.
# Вариант 3
## Описание проекта
Данный проект реализует микросервисную систему для начисления баллов с использованием Flask и Redis. Особенностью варианта 3 является реализация функции "Начисление баллов": при каждом обращении к сервису счетчик увеличивается сразу на 10 единиц.
## Бизнес-кейс "Начисление баллов"
Система ведет учёт начисленных баллов. Каждое посещение пользователем приводит к увеличению счётчика на 10 баллов. Результат отображается в виде сообщения: "Баллов начислено: X".
## Архитектура
Система построена на микросервисной архитектуре с использованием Docker Compose для оркестрации контейнеров.
<img width="1163" height="1045" alt="image" src="https://github.com/user-attachments/assets/f8eb18c7-d622-47db-9fc2-5616e01201e6" />


## Компоненты системы
### Web Service (Flask):
* Принимает HTTP-запросы от пользователей
* Реализует бизнес-логику: увеличение счётчика на 10
* Подключается к Redis для хранения текущего значения счётчика

### Redis Service:
* Хранит значение счётчика в памяти
* Имеет заданное имя контейнера: my-business-db
* Настроен на надёжную работу в рамках docker-compose

### Docker Compose:
* Оркестрирует запуск сервисов
* Устанавливает имя контейнера Redis
* Обеспечивает сетевое взаимодействие между сервисами

## Стек технологий
### Backend
* Python 3.9 — язык программирования
* Flask 2.0.1 — веб-фреймворк
* Redis 4.6.0 — клиент для взаимодействия с хранилищем
  
### Инфраструктура и DevOps
* Docker — контейнеризация приложений
* Docker Compose 3.9 — оркестрация многоконтейнерных приложений
* Alpine Linux — легковесный базовый образ
  
### База данных
* Redis (alpine) — in-memory key-value хранилище
  
### Сетевые протоколы
* HTTP/1.1 — для клиент-серверного взаимодействия
* Redis Protocol — для взаимодействия Flask ↔ Redis

# Вариант 3
### 1. Бизнес-логика (app.py)
* Счётчик увеличивается не на 1, а на 10 при каждом обращении
* Формат ответа: "Баллов начислено: X"
  
### 2. Инфраструктура (docker-compose.yml)
* Установлено имя контейнера Redis: container_name: my-business-db

### 3. Среда сборки (Dockerfile)
* Изменена рабочая директория внутри контейнера на: /app

## Инструкция по запуску
### Предварительные требования
* Docker Engine установлен и запущен
* Docker Compose V2 доступен (docker compose)

## Ход работы
### 1. Создаю папку и перехожу в нее
```
mkdir lab6 && cd lab6
```
### 2. Создаем файл зависимостей (requirements.txt) с библиотеками:
```
nano requirements.txt
```
Заполняем его следующим:
```
Flask==2.0.1
Werkzeug==2.3.7
redis==4.6.0
```
### 3. Создаем файл бизнес-логики (app.py):
```
nano app.py
```
Заполняем следующим:
```
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incrby('hits', 10)  # Увеличиваем на 10
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return '<h1 style="color:blue">Бизнес-стенд "Инновации"</h1><p>Начислено баллов: <strong>{}</strong></p>'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```
### 4. Создаем инструкцию по сборке образа (Dockerfile):
```
nano Dockerfile
```
Заполняем следующим:
```
FROM python:3.9-alpine
WORKDIR /app
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```
### 5. Создаем файл связки сервисов (docker-compose.yml):
```
nano docker-compose.yml
```
Заполняем следующим:
```
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:5000"
    depends_on:
      - redis
  redis:
    image: "redis:alpine"
    container_name: my-business-db
```
### 6. Запускаем сервисы
```
docker compose up -d
```
<img width="592" height="410" alt="image" src="https://github.com/user-attachments/assets/8d139490-8074-4a9c-90d2-6b385712775e" />
<img width="577" height="87" alt="image" src="https://github.com/user-attachments/assets/ef290314-6ecf-4b80-b2f8-fbe1b03456cf" />

### 7. Проверяем запущенные контейнеры проекта
```
docker compose ps
```
<img width="597" height="166" alt="image" src="https://github.com/user-attachments/assets/b68d6e2c-abba-48c0-a04a-0f618a5ba5fb" />

### Задача 1. Открываем браузер по адресу http://localhost:8000 и проверяем работу счетчика
При каждом обновлении страницы счетчик увеличивается.
<img width="730" height="136" alt="image" src="https://github.com/user-attachments/assets/6fd65d1f-857b-4bba-bfd7-935e2235c6db" />
<img width="732" height="138" alt="image" src="https://github.com/user-attachments/assets/bd60f1ff-779a-4fc9-9324-66f0f3503356" />
<img width="727" height="141" alt="image" src="https://github.com/user-attachments/assets/16889fe6-ead4-4ea5-a646-04b9327b079d" />

### Задача 2. Проверяем имя контейнера Redis:
<img width="592" height="141" alt="image" src="https://github.com/user-attachments/assets/dca48896-07d6-48fd-91ee-4e60ead21ad6" />

### Задача 3. Проверяем изменение рабочей директории внутри на /app.
<img width="591" height="86" alt="image" src="https://github.com/user-attachments/assets/099a34c9-b1d1-4a50-9258-b068e5eb6146" />

# Вывод
В ходе лабораторной работы мы научились разворачивать многокомпонентные приложения, понимать взаимодействие между контейнерами (Business Logic + Database) и модифицировать параметры системы под конкретные бизнес-задачи.













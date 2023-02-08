# Проект **YaMDb** 

![example workflow](https://github.com/Shkitskiy94/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)

![python version](https://img.shields.io/badge/Python-3.9-green)
![django version](https://img.shields.io/badge/Django-2.2-green)
![Docker version](https://img.shields.io/badge/Docker-4.15-green)
![Djangorestframework version](https://img.shields.io/badge/Djangorestframework-3.12-green)
![PyJWT version](https://img.shields.io/badge/PyJWT-2.1-green)
![gunicornversion](https://img.shields.io/badge/gunicorn-12.7-green)



#### Проект доступен по эл.адресу:

[51.250.93.203/api/v1/](http://51.250.93.203/api/v1/)



<details>
<summary>
Настройка CI and CD
</summary>

для Linux-систем все команды необходимо выполнять от имени администратора
- Склонировать репозиторий
```bash
git clone https://github.com/Shkitskiy94/yamdb_final
```
- Выполнить вход на удаленный сервер
- Установить docker на сервер:
```bash
apt install docker.io 
```
- Установить docker-compose на сервер:
```bash
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
- Локально отредактировать файл infra/nginx.conf, обязательно в строке server_name вписать IP-адрес сервера
- Скопировать файлы docker-compose.yml и nginx.conf из директории infra на сервер:
```bash
scp docker-compose.yml <username>@<host>:/home/<username>/docker-compose.yml
scp nginx.conf <username>@<host>:/home/<username>/nginx.conf
```
- Создать .env файл по предлагаемому выше шаблону. Обязательно изменить значения POSTGRES_USER и POSTGRES_PASSWORD
- Для работы с Workflow добавить в Secrets GitHub переменные окружения для работы:
    ```
    DB_ENGINE=<django.db.backends.postgresql>
    DB_NAME=<имя базы данных postgres>
    DB_USER=<пользователь бд>
    DB_PASSWORD=<пароль>
    DB_HOST=<db>
    DB_PORT=<5432>
    
    DOCKER_PASSWORD=<пароль от DockerHub>
    DOCKER_USERNAME=<имя пользователя>
    
    SECRET_KEY=<секретный ключ проекта django>

    USER=<username для подключения к серверу>
    HOST=<IP сервера>
    PASSPHRASE=<пароль для сервера, если он установлен>
    SSH_KEY=<ваш SSH ключ (для получения команда: cat ~/.ssh/id_rsa)>

    TELEGRAM_TO=<ID чата, в который придет сообщение>
    TELEGRAM_TOKEN=<токен вашего бота>
    ```
    Workflow состоит из четырёх шагов:
     - Проверка кода на соответствие PEP8
     - Сборка и публикация образа бекенда на DockerHub.
     - Автоматический деплой на удаленный сервер.
     - Отправка уведомления в телеграм-чат.
- собрать и запустить контейнеры на сервере:
```bash
docker-compose up -d --build
```
- После успешной сборки выполнить следующие действия (только при первом деплое):
    * провести миграции внутри контейнеров:
    ```bash
    docker-compose exec web python manage.py migrate
</details>
<br />
<br />

## Описание:
 Api собирает отзывы пользователей на различные произведения.
<br />
<br />
## Как запустить проект чeрез Docker:
Должен быть уставлен Docker https://www.docker.com

 Перейти в папку infra

```

cd infra
```

 Создать файл ".env" :

```

DB_ENGINE=django.db.backends.postgresql # указываем, что работаем с postgresql
DB_NAME=postgres # имя базы данных
POSTGRES_USER=postgres # логин для подключения к базе данных
POSTGRES_PASSWORD=postgres # пароль для подключения к БД (установите свой)
DB_HOST=db # название сервиса (контейнера)
DB_PORT=5432 # порт для подключения к БД

```



Собрать проект
```
docker-compose up -d --build
```


<details>
<summary>
памятка у какого mac os c m1
</summary>
При установки зависимостей возможна ошибка при установки psycopg2-binary==2.8.6
пришлость делать сборку на полной версии python 3.9 FROM python:3.9 ,если у вас 
не mac os c m1 можете поставить более легкую версию в Dockerfile




</details>




Cделать миграции, создать суперпользователя и собрать статику

```
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
docker-compose exec web python manage.py collectstatic --no-input
docker-compose exec web python manage.py loaddata fixtures.json # загрузить данные в бд.
```

проект доступен по адресу 

```
http://localhost/api/v1/
```


# Регистрация нового пользователя
```
POST http://localhost/api/v1/auth/signup/

{
  "email": "string",
  "username": "string"
}
```
**Получение JWT-токена в обмен на username и confirmation code(Из письма).**

Так как почтый сервер еще не настроен, нужно достать письмо из котейнера.

```
docker cp ид_контейнера:/app/sent_emails ./email
```

Либо самостоятельно прогуляться до папки /app/sent_emails

```
#попасть в терминал
docker exec -it ид_контейнера /bin/bash
```

```
POST http://localhost/api/v1/auth/token/

{
  "username": "string",
  "confirmation_code": "string"
}
```
**Редактирование, удаление и создание категорий жанров и произведений доступно только Администратору**
# Примеры запросов
## Получение списка всех категорий

```
GET http://localhost/api/v1/categories/
```
## Добавление новой категории

```
POST http://localhost/api/v1/categories/

{
  "name": "string",
  "slug": "string"
}
```

## Получение списка всех жанров

```
GET http://localhost/api/v1/genres/
```
## Добавление жанра

```
POST http://localhost/api/v1/genres/

{
  "name": "string",
  "slug": "string"
}
```

## Получение списка всех произведений
```
GET http://localhost/api/v1/titles/
```
## Добавление произведения
```
POST http://localhost/api/v1/titles/

{
"name": "string",
"year": 0,
"description": "string",
"genre": [
"string"
],
"category": "string"
}
```
## Получение информации о произведении
```
GET http://localhost/api/v1/titles/{titles_id}/
```
## Получение списка всех отзывов
```
GET http://localhost/api/v1/titles/{title_id}/reviews/
```
## Добавление нового отзыва
```
POST http://localhost/api/v1/titles/{title_id}/reviews/

{
  "text": "string",
  "score": 1
}
```
## Полуение отзыва по id
```
GET http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/
```
## Получение списка всех комментариев к отзыву
```
GET http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/comments/
```
## Добавление комментария к отзыву
```
POST http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/comments/

{
  "text": "string"
}
```
## Получение комментария к отзыву
```
GET http://localhost/api/v1/titles/{title_id}/reviews/{review_id}/comments/{comment_id}/
```
## Получение данных своей учетной записи
```
GET http://localhost/api/v1/users/me/
```
## Изменение данных своей учетной записи
```
PATCH http://localhost/api/v1/users/me/

{
  "username": "string",
  "email": "user@example.com",
  "first_name": "string",
  "last_name": "string",
  "bio": "string"
}
```

# Работа с USERS (Права доступа: **Администратор**)
## Получение списка всех пользователей
```
GET http://localhost/api/v1/users/
```
## Добавление пользователя
```
POST http://localhost/api/v1/users/

{
  "username": "string",
  "email": "user@example.com",
  "first_name": "string",
  "last_name": "string",
  "bio": "string",
  "role": "user"
}
```
## Получение пользователя по username
```
GET http://localhost/api/v1/users/{username}/
```

### **Автор**
[Юрий Шкитский](https://github.com/Shkitskiy94)
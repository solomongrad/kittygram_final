![example branch parameter](https://github.com/github/docs/actions/workflows/main.yml/badge.svg?branch=main)

### Описание проекта 
Kittygram - сервис для любителей котиков.

Что умеет проект:

- Добавлять, редактировать и удалять своих котиков.
- Добавлять новые и присваивать уже существующие достижения. 
- Просматривать чужих котов и их достижения.

## Установка 

1. Клонируйте репозиторий на свой компьютер:

    ```bash
    git clone git@github.com:solomongrad/kittygram_final.git
    ```
    ```bash
    cd kittygram_final
    ```

2. Создайте файл .env и заполните его своими данными. Перечень данных указан в корневой директории проекта в файле .env.example.


### Создание Docker-образов

1.  Замените username на ваш логин на DockerHub:

    ```bash
    cd frontend
    docker build -t <username>/kittygram_frontend .
    cd ../backend
    docker build -t <username>/kittygram_backend .
    cd ../nginx
    docker build -t <username>/kittygram_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```bash
    docker push <username>/kittygram_frontend
    docker push <username>/kittygram_backend
    docker push <username>/kittygram_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```bash
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 
    ```

2. Создайте на сервере директорию kittygram через терминал

    ```bash
    mkdir kittygram
    ```

3. Установка docker compose на сервер:

    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

4. В директорию kittygram/ скопируйте файл docker-compose.production.yml:

#### Это можно сделать командой ниже, либо же вручную.

    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml \
    username@server_ip:/home/username/kittygram/docker-compose.production.yml
    ```
Где:
    * path_to_SSH — путь к файлу с SSH-ключом;
    * SSH_name — имя файла с SSH-ключом (без расширения);
    * username — ваше имя пользователя на сервере;
    * server_ip — IP вашего сервера.

### Также в директорию kittygram/ скопируйте файл .env


5. Запустите docker compose в режиме демона:

    ```bash
    sudo docker compose -f docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /static/static/:

    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /static/static/
    ```

7. На сервере в редакторе nano откройте конфиг Nginx:

    ```bash
    sudo nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки nginx:

    ```bash
     server {
         server_name <ДОМЕННОЕ ИМЯ СЕРВЕРА>;

         location / {
         proxy_set_header Host $http_host;
         proxy_pass http://127.0.0.1:9000;
         client_max_body_size 20M;
    }
    ```

9. Проверьте работоспособность конфига Nginx:

    ```bash
    sudo nginx -t
    ```
    Если ответ в терминале такой, значит, ошибок нет:
    ```bash
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

10. Получите SSL-сертификат:
    Шаг 1. Установка certbot.
    ```bash
    sudo apt install snapd
    ```
    Далее сервер, скорее всего, попросит вас перезагрузить операционную систему. Сделайте это, а потом последовательно выполните команды:
    ```bash
    sudo snap install core; sudo snap refresh core
    ```
    ```bash
    sudo snap install --classic certbot
    ```
    ```bash
    sudo ln -s /snap/bin/certbot /usr/bin/certbot
    ```
    Шаг 2. Запускаем certbot и получаем SSL-сертификат.
    ```bash
    sudo certbot --nginx
    ```
    В процессе оформления сертификата ответьте на несколько вопросов. Почта нужна для предупреждений, что сертификат пора обновить.
    Далее система оповестит вас о том, что учётная запись зарегистрирована и попросит указать имена, для которых вы хотели бы активировать HTTPS
    Шаг 3. Перезапустите конфигурацию nginx.
    ```bash
    sudo systemctl reload nginx
    ```

11. Ещё раз проверьте работоспособность конфига Nginx:

    ```bash
    sudo nginx -t
    ```
    Если ответ в терминале такой, значит, ошибок нет:
    ```bash
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

12. Перезапускаем Nginx
    ```bash
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow уже написан. Он находится в директории

    ```bash
    kittygram_final/.github/workflows/main.yml
    ```

2. Для адаптации его на своем сервере добавьте секреты в GitHub Actions:

    ```bash
    DOCKER_USERNAME - имя пользователя в DockerHub
    DOCKER_PASSWORD - пароль пользователя в DockerHub
    HOST - ip_address сервера
    USER - имя пользователя
    SSH_KEY - приватный ssh-ключ
    SSH_PASSPHRASE - кодовая фраза для ssh-ключа
    TELEGRAM_TO - id телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
    TELEGRAM_TOKEN - токен бота (получить токен можно у @BotFather, /token, имя бота)
    ```


###  Как работать с репозиторием финального задания

#### Что нужно сделать

Настроить запуск проекта Kittygram в контейнерах и CI/CD с помощью GitHub Actions

#### Как проверить работу с помощью автотестов

В корне репозитория создайте файл tests.yml со следующим содержимым:
```yaml
repo_owner: ваш_логин_на_гитхабе
kittygram_domain: полная ссылка (https://доменное_имя) на ваш проект Kittygram
taski_domain: полная ссылка (https://доменное_имя) на ваш проект Taski
dockerhub_username: ваш_логин_на_докерхабе
```

Скопируйте содержимое файла `.github/workflows/main.yml` в файл `kittygram_workflow.yml` в корневой директории проекта.

Для локального запуска тестов создайте виртуальное окружение, установите в него зависимости из backend/requirements.txt и запустите в корневой директории проекта `pytest`.

### Автор
[Роман Куделин](https://github.com/solomongrad)
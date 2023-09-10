Киттиграмма
Блог для любителей котиков! Можно добавлять фоты, достижения для Ваших пушистых повелителей, а также рассказать всем, как зовут Его/Её Величество, какого они цвета и когда родились :)

Технологии:
Python 3.x
node.js 9.x.x
frontend: React
backend: Django
nginx
gunicorn

Установка
Все примеры указаны для Linux

1. Склоинровать репозиторий

git clone git@github.com:koyote92/infra_sprint1.git

2. Создать и активировать виртуальное окружение

python3 -m venv venv
source venv/bin/activate

3. Установить зависимости для Python

pip install -r requirements.txt 

4. Перейти в папку backend и выполнить миграции

cd infra_sprint1/backend/
python manage.py migrate

5. Создать суперюзера

python manage.py createsuperuser

6. В файле .env прописать ваш SECRET_KEY, ALLOWED_HOSTS по примеру в .env.example

7. В этом же файле поменять значение переменной DEBUG с True на False

DEBUG = False

8. Установить node.js

curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs

9. Перейти в infra_sprint1/frontend и установить зависимости для frontend-приложения

npm i

10. Для backend-приложения установить WSGI-сервер gunicorn

pip install gunicorn

11. Создать файл конфигурации для автозапуска WSGi-сервера

sudo nano /etc/systemd/system/gunicorn.service

12. Внести в него следующие настройки

[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=<имя-пользователя-в-системе>  # (!) заменить на собственное
WorkingDirectory=/home/<имя-пользователя>/infra_sprint1/backend/
ExecStart=/home/<имя-пользователя>/infra_sprint1/venv/bin/gunicorn --bind 0.0.0.0:8000 backend.wsgi

[Install]
WantedBy=multi-user.target 

13. Запустить созданную службу и внести её в автозапуск

sudo systemctl start gunicorn
sudo systemctl enable gunicorn

14. Установить веб-сервер и запустить его

sudo apt install nginx -y
sudo systemctl start nginx 

15. Перейти в infra_sprint1/frontend/, скомпилировать frontend-приложение и копировать его в системную директорию веб-сервера (необходимо для корректной работы статики)

npm run build
sudo cp -r <имя_пользователя>/infra_sprint1/frontend/build/. /var/www/infra_sprint1/

16. Перейти в infra_sprint1/backend/, собрать статику для админки Django и копировать его в системную директорию веб-сервера (ВАЖНО! Необходимо изменить название папки со статикой во избежание конфликта имён)

infra_sprint1/backend/kittygram_backend/settings.py поменять значение переменной STATIC_URL и добавить новую STATIC_ROOT

STATIC_URL = 'static_backend'
STATIC_ROOT = BASE_DIR / 'static_backend' 
python manage.py collectstatic
sudo cp -r infra_sprint1/backend/static_backend/ /var/www/infra_sprint1/

17. Создать папку для медиафайлов в директории веб-сервера, изменить права доступа

cd /var/www/infra_sprint1/
mkdir media
sudo chown -R <имя_пользователя> /var/www/infra_sprint1/media/

18. Перейти в файл конфигурации веб-сервера и изменить его настройки на следующие (для доступа нужны права root)

sudo nano /etc/nginx/sites-enabled/default 
server {

    server_name server_name <публичный-IP-адрес> <доменное-имя>;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /media/ {
        alias /var/www/infra_sprint1/media/;
    }

    location / {
    root    /var/www/infra_sprint1;
    index   index.html index.htm;
    try_files  $uri /index.html;
    }

}

19. Проверить файл конфигурации веб-сервера, перезагрузить его конфиг в случае успеха

sudo nginx -t
sudo systemctl reload nginx

20. (Опционально) Получить SSL-сертификат для вашего доменного имени при помощи certbot

sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot 
sudo certbot --nginx

Авторы
Гиржу Николай

Шаблон Телеграмм бота с Web Apps

### Для того чтобы запустить бота вам нужно:
1. Задать параметры в файле .env
2. Подключить доменное имя к IP.
3. Получить ssl сертификат на ваш IP.
4. Установить Docker
4. Установить Nginx



### Получение и привязка доменного именеи:
Зарегистрируйтесь на сайте [No-IP:](http://www.noip.com/)
<ul>
    <li>введите адрес электронной почты;</li>
    <li>придумайте пароль;</li>
    <li>поставьте галочку напротив пункта “Create my hostname later” (англ. — «создать доменное имя позже»): создадим доменное имя после регистрации — в личном кабинете;</li>
    <li>нажмите кнопку Free Sign Up;</li>
</ul>
После регистрации на указанную почту придёт письмо со ссылкой на активацию аккаунта — активируйте аккаунт и перейдите в личный кабинет.

Далее в поле Hostname введите доменное имя для вашего сайта, в поле Domain выберите домен и нажмите кнопку Add Hostname:

После привязки доменного имени укажите его в файлах `nginx ➡️ default.conf`, `.env`:<br>

Файл `.env`.
```python
APP_BASE_URL=<Ваш домен>
# Пример:
APP_BASE_URL=https://telegram-web.ddnsking.com
```

Файл `nginx ➡️ default.conf`.
```python
server_name <публичный IP сервера> <Ваш домен> *.<Ваш домен>;
# Пример:
server_name 194.67.87.140 telegram-web.ddnsking.com *.telegram-web.ddnsking.com;

ssl_certificate /etc/letsencrypt/live/<Ваш домен>/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/<Ваш домен>/privkey.pem;
ssl_trusted_certificate /etc/letsencrypt/live/<Ваш домен>/chain.pem;

#Пример:
ssl_certificate /etc/letsencrypt/live/telegram-web.ddnsking.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/telegram-web.ddnsking.com/privkey.pem;
ssl_trusted_certificate /etc/letsencrypt/live/telegram-web.ddnsking.com/chain.pem;

if ($host = <Ваш домен>) {
        return 301 https://$host$request_uri;
    }

#Пример:
if ($host = telegram-web.ddnsking.com) {
        return 301 https://$host$request_uri;
    }


server_name <публичный IP сервера> <Ваш домен> *.<Ваш домен>;
# Пример:
server_name 194.67.87.140 telegram-web.ddnsking.com *.telegram-web.ddnsking.com;
```

### Получение и привязка SSL сертификата:
Для начала установитте Nginx.
Команда для установки Nginx:
```bash
sudo apt install nginx -y 
```

Разрешите запросы по протоколам HTTP, HTTPS и SSH, выполнив команды::
```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH 
```

Теперь включите файрвол:
```bash
sudo ufw enable 
```

Теперь включите файрвол:
```bash
sudo systemctl start nginx 
```

Откройте в браузере IP-адрес или домен своего сервера без указания порта.<br>
Если высветилась такая картинка значит все сделали правильно
![alt text](https://i.imgur.com/hMLoGBj.png)

Получение и настройка SSL-сертификата:<br>
SSL сертификат будем получать с помощью специального пакета — certbot.

Установка certbot.
```bash
sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot 
```

Получение сертификата.<br>
Чтобы начать процесс получения сертификата, введите команду:
```bash
sudo certbot --nginx 
```
В процессе оформления сертификата вам нужно будет указать свою электронную почту и ответить на несколько вопросов.
<ul>
    <li>Enter email address” (англ. «введите почту»). Почта нужна для предупреждений, что сертификат пора обновить</li>
    <li>Please read the Terms of Service...” (англ. «прочитайте правила сервиса»). Прочитайте правила по ссылке, введите "y" и нажмите Enter.</li>
    <li>Would you be willing to share your email address with the Electronic Frontier Foundation?” (англ. «хотите ли вы поделиться своей почтой с Фондом электронных рубежей»). Отметьте на своё усмотрение "y" (да) или "n" (нет) и нажмите Enter</li>
    <li>Please enter the domain name(s) you would like on your certificate” (англ. «пожалуйста, введите доменные имена, которые вы хотели бы видеть в своем сертификате»). Введите добавленное в проект доменное имя, нажмите Enter</li>
</ul>

Перезапустите nginx:
```bash
sudo systemctl reload nginx 
```

Обновление сертификата:<br>

Бесплатный сертификат нужно обновлять минимум раз в три месяца. Certbot делает это по умолчанию, если вы не меняли стандартных настроек. Убедиться, что всё обновляется, можно с помощью команды:
```bash
sudo certbot renew --dry-run 
```
Остановите nginx он нам больше не понадобится:
```bash
sudo systemctl stop nginx 
```
### Установка Docker:
```bash
sudo apt-get install docker docker-compose -y
```

### Укажите настройки в файле .env:
```python
# Имя контейнера бота
BOT_CONTAINER_NAME=bot_container_name
# Имя образа бота
BOT_IMAGE_NAME=botimage_name
# Имя бота
BOT_NAME=mybotname
# Токен бота
BOT_TOKEN=<Ваш токен>
```

### Примечание по проекту:
Файлы веб приложения находятся по пути `tgbot ➡️ web_app`<br>

Подключение шаблонов осуществляется в файле `tgbot ➡️ web_app ➡️ routes.py`<br>

Регистрация шаблонов `bot.py`<br>
```python
from tgbot.web_app.routes import demo_handler, check_data_handler, send_message_handler

def main():
    ...
    app.router.add_get("/demo", demo_handler)
    app.router.add_post("/demo/checkData", check_data_handler)
    app.router.add_post("/demo/sendMessage", send_message_handler)
    ...
```

Подключение статики и javasript:<br>
`nginx ➡️ default.conf`

```python
    location /static/ {
            root /srv/tg-bot/src;
            autoindex off;
    }

    location /images/ {
        root /srv/tg-bot/src;
        autoindex off;
    }
```
В папке `tgbot ➡️ web_app ➡️ src`, лежат статические файлы и java script<br>
Пример подключение в html шаблоне

```html
    <link rel="stylesheet" href="./static/style.css" />
    <script src="./static/dummyData.js"></script>
    <script src="./static/main.js"></script>
```

В папке `tgbot ➡️ web_app ➡️ image`, лежат картинки<br>
Пример подключение в html шаблоне

```html
    <link rel="icon" type="image/x-icon" href="./images/favicon.ico" />
```

### Запуск:
Перейдите в корневой каталог проекта и выполните команду:
```bash
sudo docker-compose up
```
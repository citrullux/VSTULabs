# Лабораторная работа 6: Веб сервер Nginx в реверс-прокси режиме. Использование Let's Encrypt.

<blockquote>

Для выполнения этой работы желательно выполнитьь 5 лабораторную, так как данный материал будет использовать настроенные компоненты.

</blockquote>

## Шаг 1. Установка nginx

После 5 лабораторной установка Nginx - дело максимально простое.

### Установка

Обновляем список репозиториев

`Debian: $ sudo apt update`

`RHEL: $ sudo yum update`

Устанавливаем nginx

`Debian: $ sudo apt install -y nginx`

`RHEL: $ sudo yum install -y nginx`

### Проверка демона

На Debian подобных достаточно проверить что Nginx запустился

`Debian: $ sudo systemctl status nginx.service`

Если демон в состоянии `Active(running)` и имеет в `Loaded:` `enabled` то всё настроенно правильно.

Если нет, то стоит выполнить общие рекомендации для Debian и RHEL.

`$ sudo systemctl start nginx.service`

`$ sudo systemctl enable nginx.service`

`$ sudo systemctl status nginx.service`

> Тут уж точно должно быть Active, но если нет - пишите в Issues этого репозитория, рассмотрим что могло пойти не так.

### Проверка работоспособности

Теперь перейдя в браузере просто на

`localhost`

Вам ответит nginx! Ответ будет вида:

```
Welcome to nginx!

If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.
```

## Шаг 2. Получение сертификата

<blockquote>

### LetsEncrypt (не сработает)

> Данный пункт оставлен здесь в качестве примера того как это делается.

Для того чтобы получить сертификат, нам потребуется утилита certbot

`Debian: $ sudo apt install certbot`

`RHEL: $ sudo yum install certbot`

И компонент интеграции с nginx

`Debian: $ sudo apt install python3-certbot-nginx`

`RHEL: $ sudo yum install python3-certbot-nginx`

Мы будем использовать сертификат Let's Encrypt, в последующем примере требуется сменить на свой адрес почты 

`-m *ваш адрес почты*`

и прописать своё собственное доменное имя (запомните его)

`-d *ваш домен*`

Получится примерно вот такая команда:

`$ sudo certbot --nginx -n --agree-tos -m mymail@gmail.com -d mydomain`

</blockquote>



## Шаг 3. Настройка nginx.
# Лабораторная работа 5: Применение Node Exporter/Prometheus и Grafana для мониторинга ресурсов.

## Шаг 1. Установка Node Exporter и первоначальная настройка.

На всякий случай для начала проверим есть ли у нас установленные пакеты `wget` и `tar` и если их нет, то установим:

`Debian: $ sudo apt install wget tar -y`

`RHEL: $ sudo yum install wget tar -y`

Cкачиваем архив с утилитой node exporter при помощи утилиты `wget`:

`$ wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz`

<blockquote>

Если это не сработало, то попробуйте найти текущую версию пакета на данном сайте:

https://prometheus.io/download/#node_exporter

И обновите команду скопировав ссылку с файла скачивания для архитектуры `linux-amd64`. 

> Hint: Копировать ссылку можно нажав на название файла на сайте правой кнопкой

Если эта команда и после изменений ничего не скачает, то можно написать в Issues данного репозитория

</blockquote>

Распакуем полученный архив при помощи `tar`:

`$ tar xvfz node_exporter-*.*-amd64.tar.gz`

Заходим в дирректорию с распакованным `node_exporter`:

`$ cd node_exporter-*.*-amd64`

И пробуем запустить прямо в консоли.

`$ ./node_exporter`

Если среди последних выдачи вы видите слова `msg="Listening on"` то скорее всего вы всё сделали правильно.

Теперь можно проверить действительно ли данные мониторинга отправляются с узла, для этого заходим в браузер и переходим на страницу:

`localhost:9100/metrics`

<blockquote>

Либо открываем ещё одну консоль и в ней вводим:

`$ curl localhost:9100/metrics`

После выполнения команды можно сделать скриншот и закрыть только что открытую консоль.

</blockquote>

В изначальной консоли останавливаем работу `node_exporter` нажимая `Ctrl+C`

## Шаг 2. Создание демона Node Exporter.

Переместим приложение Node Exporter в новое место для создания демона.

`$ sudo mv node_exporter /usr/local/bin`

И собственно заполним содержимое нового демона

`sudo nano /etc/systemd/system/node_exporter.service`

```
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Для созданного демона потребуется пользователь

`Debian: $ sudo adduser --system --no-create-home --group --shell /sbin/nologin node_exporter`

`RHEL: $ sudo adduser -M -r -s /sbin/nologin node_exporter`

> Тут я не полностью уверен в том что команда для Debian написана оптимально, но это работает

После создания и заполнения файла перезагрузим список доступных демонов:

`sudo systemctl daemon-reload`

Запустим демона 

`sudo systemctl start node_exporter`

Включим демон в автозагрузку

`sudo systemctl enable node_exporter`

И проверим что демон работает

`sudo systemctl status node_exporter`

Теперь можно проверить действительно ли данные мониторинга отправляются от демона, для этого заходим в браузер и переходим на страницу:

`localhost:9100/metrics`

<blockquote>

Либо открываем ещё одну консоль и в ней вводим:

`$ curl localhost:9100/metrics`

После выполнения команды можно сделать скриншот и закрыть только что открытую консоль.

</blockquote>



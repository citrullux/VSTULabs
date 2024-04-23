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

`$ cd node_exporter-*/`

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

Теперь можно удалить более ненужные файлы

`$ cd ~`

`$ rm -rf node_exporter*`

> Даже если вы тут капитально сделаете что-то не так, система вам не даст удалить слишком многого, так как вы действуете не от sudo.

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

> Тут я не полностью уверен в том что команда для Debian написана оптимально, но это работает, если есть идеи по оптимизации - отпишитесь в Issues

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

## Шаг 3.  Установка Prometheus и первоначальная настройка.

Cкачиваем архив с утилитой Prometheus при помощи утилиты `wget`:

`$ wget https://github.com/prometheus/prometheus/releases/download/v2.51.2/prometheus-2.51.2.linux-amd64.tar.gz`

<blockquote>

Если это не сработало, то попробуйте найти текущую версию пакета на данном сайте:

https://prometheus.io/download/#prometheus

И обновите команду скопировав ссылку с файла скачивания для архитектуры `linux-amd64`. 

> Hint: Копировать ссылку можно нажав на название файла на сайте правой кнопкой

Если эта команда и после изменений ничего не скачает, то можно написать в Issues данного репозитория

</blockquote>

Распакуем полученный архив при помощи `tar`:

`$ tar xvf prometheus-*.*-amd64.tar.gz`

### Конфигурация файлов

Заходим в дирректорию с распакованным `Prometheus`:

`$ cd prometheus-*/`

Создадим 2 дирректории

`$ sudo mkdir /etc/prometheus`

`$ sudo mkdir /var/lib/prometheus`

Скопируем конфигурацию Prometheus

`$ sudo cp prometheus.yml /etc/prometheus/`

Скопируем исполняемые файлы

`$ sudo cp prometheus /usr/local/bin/`

`$ sudo cp promtool /usr/local/bin/`

Скопируем библиотеки и консоли

`$ sudo cp -r consoles/ /etc/prometheus`

`$ sudo cp -r console_libraries/ /etc/prometheus`

### Инициализация конфигурации

Откроем файл основной конфигурации prometheus и заполним контентом указанным ниже:

`$ sudo nano /etc/prometheus/prometheus.yml`

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

#=================================================
# OK DONT TOUCH!!!!!!!!!! THATS MAIN CONFIG!
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
# DONT TOUCH ENDS
#=================================================

  - job_name: 'node'
    scrape_interval: 30s
    static_configs:
      - targets: ['localhost:9100']
```

### Добавление пользователя и смена прав

Для смены прав на дирректории и создания демона потребуется новый пользователь

`Debian: $ sudo adduser --system --no-create-home --group --shell /sbin/nologin prometheus`

`RHEL: $ sudo adduser -M -r -s /sbin/nologin prometheus`

Теперь можно сменить права на дирректории с файлами Prometheus которые мы переносили ранее.

`$ sudo chown prometheus:prometheus /etc/prometheus`

`$ sudo chown prometheus:prometheus /var/lib/prometheus`

## Шаг 4. Создание демона Prometheus.


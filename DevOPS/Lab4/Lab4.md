# Лабораторная работа 4: Ansible. Примеры использования.

## Шаг 1. Установка Ansible и первоначальная настройка.

Для установки Ansible следует применить пакетный менеджер, в котором требуется обновить список репозиториев.

`Debian: $ sudo apt-get update`

`RHEL: $ sudo yum update`

Теперь можно установить Ansible

`Debian: $ sudo apt-get install ansible -y`

`RHEL: $ sudo yum install ansible -y`

Установим ssh-сервер

`Debian: $ sudo apt-get install openssh-server -y`

`RHEL: $ sudo yum install openssh-server -y`

В данном случае использование флага `–y` позволяет избегать вопросов при установке так как система будет на них автоматически отвечать утвердительно.

Проверить установку можно применив команду

`$ ansible-playbook -h`

Также необходимо узнать о конфигурации сетевого интерфейса.

`$ ip addr`

Нам нужен IP адрес, который не будет иметь название lo. К примеру:

```
eth0:
inet 172.26.196.88/20
```

Соответственно нужный нам IP адрес - 172.26.196.88

## Шаг 2. Создание файлов конфигурации

Перейдём в домашнюю директорию

`$ cd ~`

Создадим рабочую директорию и перейдём в неё

`$ mkdir Ansible`

`$ cd Ansible`

Создадим 3 файла конфигураций

`$ touch init.sh`

`$ touch playbook_init.yml`

`$ touch hosts`

Добавим прав на исполнение исполняемому файлу

`$ chmod +x init.sh`

Команда показа файлов подтвердит их успешное создание и права доступа

`$ ls -la`

Можно также попробовать использовать

`$ ll`

## Шаг 3. Написание конфигурации

Изменим содержимое основного файла

`$ nano init.sh`

Записав в него следующую конфигурацию

```
#!/usr/bin/bash
ansible-playbook playbook_init.yml -i ./hosts
```

<blockquote>

Если редактор nano отсутствует, то установите его посредством команд:

`$ sudo apt-get install nano -y`

`$ sudo yum install nano -y`

</blockquote>

Теперь перейдём в файл hosts и запишем в него компьютеры для которых будет выполняться действие:

`$ nano hosts`

IP представленный далее взять из 1 шага, не забудьте заменить на свой
```
[Nodes]
172.26.196.88
```

Остаётся указать для ansible что требуется сделать
`$ nano playbook_init.yml`

```
---
- hosts: Nodes
  tasks:
  - name: Touch
    shell: /usr/bin/touch ~/GERONIMO
    args:
       executable: /bin/bash
```

## Шаг 4: Вопросы конфигурации и запуск

Для того чтобы наш скрипт смог сработать мы должны иметь возможность перейти на узел. Для этого требуется установить аутентификацию по ключу для ssh соединения.

`$ ssh-keygen`

(потребует нажать Enter несколько раз)

Желательно на время эксперимента отключить фаерволл

`$ sudo systemctl stop firewalld`

<blockquote>

Пользователям RHEL–подобных систем также следует проверить

`RHEL: $ sestatus`

</blockquote>

Требуется положить новый ключ на сервер, для этого выполняем

> Не забудьте что здесь и далее нужен ваш IP записанный заранее!!!

`$ ssh-copy-id 172.26.196.88`

И перейдите по ssh на узел и наберите на клавиатуре yes и enter

`$ ssh 172.26.196.88`

И произведите выход с узла комбинацией `Ctrl+D` или

`$ exit`

Теперь можно попробовать выполнить ansible скрипт

`$ ./init.sh`

После чего проверим наличие файла GERONIMO в домашней директории

`$ ls ~/`

<blockquote>

## Пояснение к выдаче Ansible в шаге 4.

```
TASK [Touch]*****************************************************************
[WARNING]: Consider using the file module with state=touch rather than running 'touch'.  If you need to use command because file is insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.
changed: [172.26.196.88]
```

Ansible как утилита содержит в себе набор готовых модулей для выполнения определённых задач, в рамках нашего скрипта мы использовали консольную команду для создания файла:

`shell: /usr/bin/touch ~/GERONIMO`

Что создало файл используя системный инструмент для этого предназначенный, но это не лучший метод написать это в ansible.

В шаге 5 и далее будут рассматриваться встроенные методы ansible для исполнения некоторых базовых задач, начиная с создания файла.

</blockquote>

## Шаг 5: Создание файла через встроенные механизмы

Перепишем основную конфигурацию

`$ nano playbook_init.yml`
```
---
- hosts: Nodes
  tasks:
  - name: Touch file, set the permissions
    ansible.builtin.file:
      path: ~/GERONIMO2
      state: touch
      mode: u=rw,g=r,o=r
```

Выполним ansible скрипт

`$ ./init.sh`

Теперь выполним

`$ ls –la ~/`

Скорее всего права для файлов GERONIMO и GEROMIMO2 немного отличаются, так как touch действует по умолчанию, тогда как для этой команды были определены права.

## Шаг 6: Установка приложений и повышение прав

Перепишем основную конфигурацию

`$ nano playbook_init.yml`
```
---
- hosts: Nodes
  become: True
  become_user: root

  tasks:
  - name: InstallScreen
    ansible.builtin.apt:
      name: screen
      state: present
```

Обратите внимание, что в Debian-подобных системах будет использоваться модуль

`Debian: ansible.builtin.apt`

тогда как в RHEL-подобных будет использоваться модуль

`RHEL: ansible.builtin.yum`.

Немного дополним настройки системы. Перейдём в пользователя с правами root.

`$ sudo su`

Создадим ключ корневого пользователя (на все вопросы команды жмём Enter)

`# ssh-keygen`

Далее нужно переместить ключ на целевой узел, поскольку это наш текущий узел и из-за особенностей работы системы, проще всего это сделать следующим образом.

`# cd ~/`

`# cd .ssh`

`# cp id_rsa.pub authorized_keys`

`# chmod 600 authorized_keys`

Дальше перейдите ваш узел оставаясь в пользователе рута и наберите на клавиатуре yes и enter

`# ssh 172.26.196.88`

После этого можно ввести с клавиатуры комбинацию Ctrl+D (дважды) до возвращения в основную пользовательскую сессию
Теперь можно исполнить команду

`$ sudo ./init.sh`

И команду

`$ screen -h`

Которая покажет опции установленного пакета.

## Индивидуальные задания

> Cоответствует остатку от операции деления вашего номера зачётной книжки на 5, где получение результата 0, обозначает что у вас 5 вариант

1.	Напишите ansible скрипт создающий файл и записывающий в него содержимое вывода в консоль команды ps -aux
2.	Напишите ansible скрипт устанавливающий время в системе на определённое, либо синхронизирующий со временем сети. (время проверяется при помощи команды date)
3.	Напишите ansible скрипт удаляющий содержимое определённой директории
4.	Напишите ansible скрипт копирующий файл из одной директории в другую изменяя его имя.
5.	Напишите ansible скрипт создающий файл и записывающий в него содержимое вывода в консоль команды ls -la

> За авторством Абдрахманова Дмитрия, 2024 год
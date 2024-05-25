
# Домашняя работа: Vagrant-стенд c PAM

Цель работы: Научиться создавать пользователей и добавлять им ограничения

Что нужно сделать?

- Создать виртуальную машину
- Запретить всем пользователям кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников
``` * дать конкретному пользователю права работать с докером и возможность перезапускать докер сервис
```
# Выполнение

## Создаём виртуальную машину

Создаю в домашней директории Vagrantfile, в тело данного файла копирую содержимое из прилагаемой методички.
 
Собираю стенд командой:

``` [nur@test hw-15-PAM]$ vagrant up ```

Подключаюсь к стенду:

``` [nur@test hw-15-pam]$ vagrant ssh ```

Переходим в root-пользователя:

``` vagrant@pam:~$ sudo -i ```

Создаём пользователя otusadm и otus:
``` root@pam:~# sudo useradd otusadm && sudo useradd otus ```

Создаём пользователям одинаковые пароли "Otus2022!":
``` root@pam:~# passwd otus
New password: 
Retype new password: 
passwd: password updated successfully
root@pam:~# passwd otusadm
New password: 
Retype new password: 
passwd: password updated successfully
```
Создаём группу admin  и добавляем пользователей vagrant,root и otusadm в группу admin:
``` root@pam:~# groupadd -f admin
root@pam:~# usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
```
Проверяем возможность подключения по ssh c хостовой машины к гостевой для пользователей "otus" и "otusadm":

``` [nur@test hw-15-pam]$ ssh otus@192.168.56.13
otus@192.168.56.13's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-102-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Sat May 25 09:57:10 AM UTC 2024

  System load:  0.0                Processes:             142
  Usage of /:   13.2% of 30.34GB   Users logged in:       0
  Memory usage: 29%                IPv4 address for eth0: 10.0.2.15
  Swap usage:   0%                 IPv4 address for eth1: 192.168.56.13
```
Подключение успешно.

## Запрещаем всем пользователям кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников

Проверим что пользователи root, vagrant и otusadm есть в группе admin:

``` root@pam:~# cat /etc/group | grep admin
admin:x:1003:otusadm,root,vagrant
```
Далее настроим правило, по которому все пользователи кроме тех, что указаны в группе admin не смогут подключаться в выходные дни, для этого
создадим файл-скрипт по пути: /usr/local/bin/login.sh для использования его модулем pam_exec, пример скрипта берём из методички.

Добавим права на исполнение файла:
 
``` root@pam:~# chmod +x /usr/local/bin/login.sh ```

Укажем в файле /etc/pam.d/sshd модуль pam_exec и наш скрипт:

``` root@pam:~# vi /etc/pam.d/sshd
# PAM configuration for the Secure Shell service

# Standard Un*x authentication.
@include common-auth

# Disallow non-root logins when /etc/nologin exists.
account    required     pam_nologin.so
auth required pam_exec.so debug /usr/local/bin/login.sh
# Uncomment and edit /etc/security/access.conf if you need to set complex
# access limits that are hard to express in sshd_config.
# account  required     pam_access.so
```
Проверим работу скрипта:

``` [nur@test hw-15-pam]$ ssh otus@192.168.56.13
otus@192.168.56.13's password: 
Permission denied, please try again.
```
Подключение производилось в субботу, для пользователя "otus" доступ запрещён - скрипт отработал.













